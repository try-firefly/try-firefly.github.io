---
title: 'Firefly case study'
date: 2022-04-18T17:12:09-07:00
draft: true
---

# Firefly Case Study {#case_study}

---

## 1. Introduction {#introduction}

Serverless functions have become a key part of distributed architectures. Unfortunately, due to their stateless, ephemeral nature they are difficult to observe with standard observability approaches. They can behave like a black box in your system. Firefly is an open-source observability framework for serverless functions. It provides key insights into serverless function health through the use of metrics and traces--illuminating the black box.

{{< figure src="/assets/firefly-introduction.png" caption="Figure 1.0 Firefly introduction" >}}

---

## 2. Microservices & Serverless {#microservices_and_serverless}

### 2.1 Microservices

With the rise of cloud compute, the ability to build a microservice architecture has become more attainable – providing an alternative to monolith architectures. Microservices are now commonplace and represent a distributed architectural approach in which a single application is composed of a number of loosely coupled, independently deployable and scalable services. Each service has a clearly defined role, executes its own processes, and communicates via an API.

{{< figure src="/assets/microservice-architecture.png" caption="Figure 2.1 Mircoservice architecture" >}}

### 2.2 Serverless

Serverless is a development approach that allows for applications to be created and deployed without worrying about the underlying infrastructure of the system. In this model, the control in provisioning, maintaining and scaling the server is handed off to a third-party cloud provider for a fee. Developers only have to provide application code which will be packaged and deployed by the third-party provider.

A microservices architecture can be composed of services that maintain their own servers, as well as services which are serverless. Serverless microservices are often built with one of more serverless functions–small pieces of code designed to be asynchronously triggered based on events.

{{< figure src="/assets/serverless-microservice-architecture.png" caption="Figure 2.2 Serverless mircoservice architecture" >}}

### 2.3 Functions as a Service

Functions-as-a-Service (FaaS) describe the cloud-based service that hosts and manages serverless functions. To deploy a serverless function to a FaaS provider, a developer writes code that fulfills a particular purpose in the application and defines an event that will trigger the function. Once the event is triggered, the service provider starts a new instance of the function, work is performed and the result is returned.

As more events are triggered, the FaaS provider is responsible for automating the provisioning of computation resources and scaling function instances in response to demand; this eliminates the need for frequent hands-on management. Providers charge with usage-based pricing, resulting in no costs accumulated when a serverless function is sitting idle. This allows applications to use and pay for more bandwidth only when computing needs increase without making prior reservations, as in a traditional setup.

{{< figure src="/assets/single-server.png" caption="Figure 2.3 Single server" >}}
{{< figure src="/assets/faas-system.png" caption="Figure 2.4 FaaS system" >}}


This auto-scale, auto-provision, pay-per-use model allows developers to save time and money due to being free from implementing, troubleshooting, and maintaining servers. Not only can they respond to customers’ needs faster, but also focus on what matters—business and application goals.

{{< figure src="/assets/function-cycle.png" caption="Figure 2.5 Serverless function cycle" >}}

Unfortunately, the inability to access or modify the cloud provider’s environment and infrastructure makes it challenging for a developer to collect data about serverless functions; a different set of strategies is required. 

---

## 3. What is Observability? {#what_is_observability}

### 3.1 Observability overview

> “[Observability is the] measure of how well you can understand and explain any state your application can get into, no matter how novel, or bizarre.” - Majors et al., Observability Engineering

A software system — when reduced to its most basic purpose — should take an input, perform some form of computation and produce an output. Observability is concerned with the data generated during this process. Using this data, we are able to gain a better understanding of the internal state of the system from the outside. The inner workings of the system do not have to be known to ask and answer questions about it. Tracking down potential issues and onboarding new engineers becomes easier the more observable a system is.

Data generated by the system, pertaining to its behaviour, is referred to as telemetry data. In order to generate this data, a system must be instrumented. Instrumentation is the process of adding code to your system in order for it to generate and emit telemetry data. Observability focuses on three main telemetry data types: metrics, traces and logs.

{{< figure src="/assets/three-pillars.png" caption="Figure 3.1 Three pillars of observability" >}}

### 3.2 Metrics

Metrics are numeric, quantifiable measurements that reflect the health of the infrastructure aggregated over a defined period of time. They are key performance indicators for a system, enabling engineers to identify patterns and anomalies. While they are able to quickly indicate the presence of an issue, they are not sufficient for pinpointing the exact source of the problem within a program. Examples of metrics include request rate, error rate and latency.

### 3.3 Logs

A log is a time stamped message which can be either structured or unstructured. It is a record of a specific, pre-configured event that happened at a particular time during the request lifecycle. This allows them to provide granular insights that a metric cannot; however their volume of generation causes them to be difficult to manage. Logs aren’t useful for tracking a request through its lifecycle but come in handy once a greater amount of granularity is required. Firefly does not currently incorporate logs.

### 3.4 Traces

Distributing tracing is the process of tracking a single request as it makes its way through the different parts of an application. A trace is the term used to describe the visual representation of this tracked request. Due to the nature of distributed systems, reproducing failures locally can be overly complex. Tracing negates this complexity and makes debugging considerably easier as we are able to break down each step in a given request.  Let’s look at the system below for a clearer picture:

{{< figure src="/assets/distributed-application.png" caption="Figure 3.4 Distributed application" >}}

Here we see the components that make up the system, but we aren’t able to see how a request would travel through it, or which point in the request’s life cycle causes the request to fail. This is where distributed tracing comes in.

{{< figure src="/assets/spans.png" caption="Figure 3.5 Distributed trace" >}}

The above displays a trace in its entirety. Each component is referred to as a span. A span represents a unit of work and the time taken to complete that work. The first span in a trace is the root span — it generally represents a request from start to finish. So in this case the root span is the web application. The spans underneath are child spans; each child represents a unit of work within that request’s journey. The request flow for the above trace is as follows:

* `Web Application` calls service `A`
* Service `A` calls service `B` and waits for a response
* Service `A` calls service `C` and `D`

In order for spans to become a distributed trace, certain metadata representing a shared state (context) needs to be passed at each part of the request execution. This can then be used to correlate the spans into a single trace. This mechanism is known as context propagation. Requests carry this information throughout their lifecycle, allowing each generated span to have access to the shared context. Context propagation is what makes distributed tracing possible across a distributed system.

{{< figure src="/assets/context-propagation.png" caption="Figure 3.6 Context propagation" >}}

---

## 4. Observability for serverless functions {#observability_for_serverless_functions}

### 4.1 Why is it important?

Observing serverless functions is essential if we want to get an overall picture of a system’s health. For an entire system to be observable, each individual part must be observable. In other words, each part must emit telemetry data. 

Due to the ubiquity of serverless functions in distributed architectures, observing them is necessary if we are to have an accurate idea of how an entire system is performing. Failing to do so would make distributed traces lose context, function errors go undetected, and key metrics remain unseen.  Ultimately, this would make tracking down issues harder and lead to false positives for issues in downstream services.

{{< figure src="/assets/complete-observability.png" caption="Figure 4.1 Complete observability" >}}
{{< figure src="/assets/partial-observability.png" caption="Figure 4.2 Partial observability" >}}

### 4.2 Why is it challenging?

While the case for serverless function observability seems clear, their unique nature makes increasing observability via tried-and-true approaches challenging.

Many current solutions for adding observability to services use agents to collect and emit telemetry data.  Agents are processes which run independently of the service but within the same environment.  It collects and temporarily stores telemetry data generated by the service and its environment, emitting it to the observability backend.

The inability to access the function’s environment in FaaS results in the lack of ability to install an agent. Furthermore, the ephemeral nature of the serverless function results in any temporarily stored telemetry data to be lost when the cloud provider destroys the instance. Working around these issues requires difficult manual instrumentation and attempting to tap into the cloud provider’s ecosystem for relevant environment-related data.

The serverless function’s event-based invocations make tracing and context propagation difficult compared to microservices which communicate via HTTP requests.  The events which invoke the functions must carry context in a way that is accessible to the instrumentation of that function.  The result which is returned by the function must also carry context in a way that is readable by downstream functions or services.  Both scenarios in FaaS are vendor-specific, and context propagation would need to account for this specificity to be able to create an unbroken trace across the lifecycle of the request.  In a system which may span multiple cloud ecosystems, a vendor-agnostic approach must be found.

---

## 5. Existing solutions {#existing_solutions}

### 5.1 Software as a Service (SaaS)

Software-as-a-service solutions usually require minimal setup to use. Once you have an account they will generally provide code and processes to manually instrument your functions or set up auto-instrumentation. These vendors normally have an extensive feature set, encompassed in a central UI, that goes well beyond serverless functions. They will also manage the data pipeline, so scaling for increased load does not have to be taken into account.

While these vendors provide many benefits to their users, there are some downsides. The first is cost, which may have a greater impact for smaller companies and projects. Observability platforms are known to have unpredictable and large costs; it's not uncommon for them to be second in cost to your cloud provider. Another downside is the loss of data control and ownership, as it is all sent to a third party–a deal breaker for certain industries and businesses, such as banks and hospitals.

### 5.2 In-house vendor-specific (AWS CloudWatch)

AWS provides CloudWatch as its native observability solution for its serverless functions. The advantage of using CloudWatch is that there is no setup required. Metrics and logs are collected by default, and tracing (via AWS X-Ray) can be enabled through the AWS console for each function you wish to collect trace data for.

Despite being easy to setup, CloudWatch has numerous disadvantages. Like SaaS solutions, billing can be unpredictable and confusing, with separate billing for queries, events, dashboards, data volume, alarms, among others. It can quickly become difficult to track exactly how much you will be subsequently charged and where those charges have stemmed from.

CloudWatch offers no global overview of your telemetry data, with each telemetry type being separated into its own dashboard. Being able to see links between your telemetry data makes it considerably easier to diagnose potential problems; having to switch between dashboards adds unnecessary friction to this process.

Using CloudWatch also means your data is siloed. Add this to the fact that it can only be used to observe AWS services and a number of issues start to arise. To get a picture of your overall system health each component needs to be observable, having one solution that is able to incorporate your entire architecture starts to become paramount.

### 5.3 DIY

A do-it-yourself solution can alleviate many of the issues seen with a SaaS provider. Open-source tools such as OpenTelemetry can be used to receive, process, and emit telemetry data. Using a DIY solution provides the greatest amount of freedom, you can choose exactly how you want to setup your system and have full control over the data emitted. This not only keeps costs down but provides much better control. The one major downside of a DIY approach is the sheer amount of work it takes to setup. There is a considerable amount of prior research that needs to be undertaken. Then, once done, there could be weeks or months of work required to perfect your chosen setup.

---

## 6. Introducing Firefly {#introducing_firefly}

Firefly is an open-source observability framework for serverless functions. It provides key insights into serverless function health through the use of metrics and traces. Firefly automates function instrumentation and the deployment of a telemetry pipeline, ultimately presenting your function data in an easy-to-use dashboard.

Firefly is designed for small companies and projects looking to observe their serverless functions without committing to a commercially available observability solution. It aims to strike a balance between the ease-of-use of the SaaS solution, with the low cost of the DIY solution.

{{< figure src="/assets/solution-comparison.png" caption="Figure 6.0 Solution comparison" >}}

### Setup

Firefly’s setup involves two main components:

1. Deployment of the backend infrastructure via Docker to a host of your choice
2. Automated instrumentation through a command line interface.

SETUP GIFs TO GO HERE

### Usage

MAIN DASHBOARD TO GO HERE

Upon loading Grafana you will be greeted with Firefly's main dashboard. This provides a general overview of all your serverless functions through highlighting key metrics such as invocations, errors and duration.

INDIVIDUAL DASHBOARD TO GO HERE

Clicking on an individual function will lead you to Firefly's function dashboard. This allows you to dive into a specific function’s metrics and traces in much greater depth.

SHOW HOW METRICS AND TRACES ARE LINKED

Scrolling down the dashboard you will be able to see traces for individual invocations and any associated errors.

SHOW TRACE VIEW

Clicking on an individual trace will take you to the trace view, where you can further inspect each span and see the entire request lifecycle.

SHOW CLICKING ON SPAN WITH ERROR

Clicking on a span will show detailed information about that particular point in the request’s journey, enabling you to diagnose potential errors with greater ease.

### Firefly's limitations

Although Firefly is an ideal solution for small companies and projects looking to increase observability of their serverless functions, it does have its limitations. Firefly currently only supports Node.js AWS Lambda functions and does not incorporate logs, concentrating on metrics and traces. It should also be noted, Firefly is not a complete observability tool covering your entire stack. It is narrowly focused on serverless functions, with the extensibility to grow into something much more encompassing if desired.

---

## 7. Architecture {#architecture}

### 7.1 OpenTelemetry

OpenTelemetry is a free and open-source collection of tools, APIs, and SDKs used to instrument, generate, collect and export telemetry data. It is an industry standard for observability, created in response to increasing vendor variability in instrumentation resulting in vendor lock-in.  We chose to incorporate OpenTelemetry because of its vendor neutrality and strong community.

The key piece we use within our architecture is the OpenTelemetry collector. It receives and exports data in multiple formats. We use a stripped down collector to send trace data from the serverless function, but we also use a collector as a gateway to our backend. This was because we needed to receive data in two different formats as well as being able to batch and export this data to our database. The diagram below shows a high-level overview of how the gateway collector is configured.

{{< figure src="/assets/otel-collector.png" caption="Figure 7.1 OpenTelemetry collector" >}}

### 7.2 AWS Lambda

As the popularity of serverless grows, so do the list of available serverless providers and services. Among them, one of the most popular functions-as-a-service is the AWS Lambda. Firefly currently only supports Node.js Lambda functions. We chose to support AWS first due to a large proportion of developers already being familiar with the platform and having existing infrastructure there; this being a direct result of AWS's market share.

### 7.3 What is a telemetry system architecture?

{{< figure src="/assets/telemetry-system-architecture.png" caption="Figure 7.2 Telemetry system architecture" >}}

A telemetry system architecture or pipeline consists of an emitting, shipping, and presentation phase.

The emitting phase is where the application code is instrumented to generate telemetry data used by the pipeline. This phase determines the format and content of the telemetry data before entering the shipping phase of the pipeline.

The shipping phase processes, transforms, and stores telemetry data. This data can be stored in a relational database, a NoSQL document store, a SaaS provider, or an object store, which will be used to subsequently query the data in the presentation phase.

The presentation phase is where telemetry data is presented in charts, graphs, and tables. This aids the developer in getting an overview of their system health and helps them pinpoint failures in their system.

{{< figure src="/assets/firefly-architecture.png" caption="Figure 7.3 Firefly architecture" >}}

### 7.4 Emit phase

#### Traces

In the emitting phase, Firefly generates and collects traces and sends this data to the shipping phase.

Handling trace emission was a two step process. We used middleware provided by AWS Distro for OpenTelemetry in order to instrument the Lambda functions for tracing. The AWS Distro for OpenTelemetry (ADOT) is an AWS-supported distribution of the OpenTelemetry project, designed to provide auto-instrumentation of AWS resources and services. We then also created a new Lambda handler–a wrapper for the main function code–to better handle context propagation.

AWS Lambda Layers are used to deploy the wrapper and middleware to users’ Lambdas. Lambda Layers allow code and dependencies to be easily deployed and shared across multiple Lambdas, allowing for easier management and updates. This abstraction allows for user-friendly auto-instrumentation.

{{< figure src="/assets/lambda-layers.png" caption="Figure 7.4 Lambda layers" >}}

When a Lambda function is sent an event, the request first traverses through the code deployed by the layers. The applied code allows spans to be emitted from the function to the backend in OpenTelemetry format. It also ensures context propagation, which allows multiple spans to share the same context. This is later used to correlate the spans into complete traces.

{{< figure src="/assets/lambda-layers-plus-event.png" caption="Figure 7.5 Lambda layers receiving event" >}}

#### Metrics

AWS Lambdas automatically report metrics to Amazon CloudWatch, with additional Lambda-specific metrics available to be parsed from CloudWatch logs. Using a combination of CloudWatch’s metric stream, AWS Kinesis Firehose and AWS Lambda Insights services, Firefly is able to send key serverless metrics data in JSON format to the telemetry backend for shipping and presentation via HTTPS.

{{< figure src="/assets/metric-emission.png" caption="Figure 7.6 Metric emission" >}}

### 7.5 Shipping phase

The [OpenTelemetry collector](https://opentelemetry.io/docs/concepts/components/) is the gateway to Firefly’s observability backend for both metric and trace data sent during the emit phase. The collector then sends this data to Promscale. Promscale acts as a connector between the collector and TimescaleDB, a timeseries database powered by PostgreSQL. It accepts the Prometheus data format for metrics, due to it currently being the most widespread, and the OpenTelemetry format for traces. Once the data is stored in the timeseries database it can be easily queried via SQL, PromQL or Jaeger query.

While trace data in the OpenTelemetry format is emitted from the function and can ultimately be received by Promscale in the same format, metrics data needs to be transformed. Luckily, via multiple open-source, OpenTelemetry supported components, the collector can easily receive, process and export telemetry data in one format to another and send it to backend endpoints. This allows for metrics data to pass to the collector, be transformed from JSON to Prometheus metrics format and be sent to Promscale for storage.

{{< figure src="/assets/otel-collector-gateway.png" caption="Figure 7.7 Gateway OpenTelemetry collector" >}}

### 7.6 Presentation phase

Firefly uses Grafana – an open-source monitoring solution and visualization tool – in order to present the collected metric and trace data. A number of dashboards specifically focused on serverless functions are pre-built and loaded at start up. While Firefly's dashboards provide all the necessary data to hunt down a serverless function issue, Grafana allows users to easily add to these dashboards, build their own, or perform ad-hoc queries.

Grafana has the ability to query multiple data types and sources, with different query interfaces for each.  The three most applicable to Firefly are the metrics (Prometheus), traces (Jaeger) and PostgreSQL sources. Firefly pre-loads Grafana with the appropriate configuration for all three sources. This allows users who are more familiar with querying their telemetry data in a certain format to do so. With this, users are able to use the full capabilities of Grafana without sacrificing the convenience of Firefly.

{{< figure src="/assets/presentation-phase.png" caption="Figure 7.8 Presentation phase" >}}

---

## 8. Challenges & Tradeoffs {#challenges_and_tradeoffs}

### 8.1 Emission and shipment of metrics

Firefly made the decision to use OpenTelemetry's tooling to build out the necessary infrastructure to emit and receive telemetry data. The reason for this was twofold: it's open source and rapidly becoming the industry standard. We therefore intended to build a Lambda layer that would emit both trace and metric data with OpenTelemetry specifications. Unfortunately, OpenTelemetry's Lambda support for instrumentation of metrics in Node.js is still a work in progress and proved too difficult to manage within the scope of this project.

Despite this, we did not want to drop OpenTelemetry. By using the OpenTelemetry collector as the gateway to our backend infrastructure we ensured the extensibility of our system. Users with existing components instrumented using OpenTelemetry would have a far easier time plugging them into Firefly. Alternatively, they could use Firefly's command line interface to automatically instrument their functions and set up the necessary AWS infrastructure to send telemetry data to their existing OpenTelemetry-compliant observability backend.

We therefore had to find a solution that would enable us to emit metric data from AWS while also being able to receive it using an OpenTelemetry collector. Two mechanisms were available to us: push versus pull.

In a pull based mechanism, sources emitting metrics need to expose an endpoint from which data can be “pulled” by a scraper. In a push based mechanism, the data sources actively push metrics out to a backend receiver once data is generated and available. The OpenTelemetry collector has built-in capability to handle both scenarios, acting as a receiver or scraper that can then send the collected data to the backend for storage. As Lambda metrics were automatically generated and sent to Amazon CloudWatch, attempting to either push or pull from CloudWatch became a clear choice.

{{< figure src="/assets/pull-method.png" caption="Figure 8.1 Pull method" >}}

A key hurdle with the pull mechanism was finding an HTTP endpoint exposing metrics in a format understandable to the collector. Since no such endpoint natively existed, an intermediary exporter needed to be used to fetch and transform the data. This newly formatted information then needed to be made available via an endpoint. We identified the [yet-another-cloudwatch-exporter](https://github.com/nerdswords/yet-another-cloudwatch-exporter) (YACE) to be a good option.

The main advantage of YACE was how specific it allowed us to be; each metric we wished to include could be listed in a configuration file. There were, however, a number of downsides. Using YACE resulted in another node being added to our pipeline, as well as having to poll an API to receive metric data. These downsides caused a domino effect, resulting in increased latency and the requirement to manage polling intervals. The management of polling intervals is an important balancing act. Too many calls is wasteful and causes API throttling; making too few adds unnecessary latency.

Looking towards other observability solutions, we determined a push based approach would be the better option. Amazon CloudWatch’s metric stream service in conjunction with Amazon Firehose has the ability to aggregate and send metrics to an HTTPS endpoint on a per minute basis only when new data is generated. This ensures we quickly receive data when activity is high, and eliminate wasteful API calls when there is no activity. While there is little ability to specify exactly which Lambda metrics we wanted to receive and a nominal cost is incurred per metric update, this was the better option.

{{< figure src="/assets/push-method.png" caption="Figure 8.2 Push method" >}}

Using a receiver built by the OpenTelemetry community we were also able to receive this data using the OpenTelemetry collector. While the receiver is still in alpha stage, it proved to be stable enough for our purposes with little issues. The OpenTelemetry collector is battle-tested and has extensive functionality, with over 70 receivers in various stages of development. We therefore chose this extensibility over initial robustness.

### 8.2 Context propagation

Distributed tracing is the act of tracking a request across a system. Context propagation is the means by which this is achieved. As a request goes from service to service, a context object is passed along with it. This is often performed using dedicated HTTP headers embedded in the request. The headers contain formatted information that usually includes a trace ID as well as parent span’s ID. Downstream services can parse this information to appropriately assign the correct trace and hierarchy as it generates spans.

{{< figure src="/assets/context-passing.png" caption="Figure 8.3 Context passed from service to service" >}}

Several different protocols exist for context propagation; each protocol has its own convention for headers and formatting for associated values.  Services communicating with each other must ensure they share the same protocol for propagation to be successful.

#### How it should work:

The AWS Distro for OpenTelemetry (ADOT) layer within Firefly’s architecture acts as the middleware for tracing and context propagation for the Lambda. Using OpenTelemetry API components, the layer should parse context information from incoming requests, create spans with the appropriate context and then inject the context in outgoing requests to downstream services.

{{< figure src="/assets/adot-layer.png" caption="Figure 8.4 ADOT layer" >}}

#### What actually happens:

By default, AWS Lambdas and the ADOT layer default to using [AWS X-Ray's context propagation method](https://docs.aws.amazon.com/xray/latest/devguide/xray-concepts.html) with AWS specific headers; however, those headers are not natively supported by OpenTelemetry. This creates a problem: with the ADOT layer, Lambdas can only parse OpenTelemetry supported context formats while injecting AWS formatted context to outgoing requests. Upon research, various GitHub issues suggested the former scenario may be an oversight, while the latter may have been a design choice to allow for AWS’s propagation method to be the default.

{{< figure src="/assets/default-adot-layer.png" caption="Figure 8.5 Default ADOT layer behaviour" >}}

#### The fix

Fortunately, ADOT allows users to switch from using X-Ray's context propagation method to OpenTelemetry supported [W3C TraceContext propagation](https://www.w3.org/TR/trace-context/).

{{< figure src="/assets/w3c-adot-layer.png" caption="Figure 8.6 ADOT layer using W3C headers" >}}

The W3C protocol uses a header called `traceparent` containing the trace ID and parent span’s ID to pass context.

{{< figure src="/assets/traceparent-header.png" caption="Figure 8.7 traceparent header" >}}

By switching propagation methods, OpenTelemetry’s context propagation API is able to look for the presence of the `traceparent` header to parse context appropriately, as well as injecting a new `traceparent` into the header for outgoing requests. Because it is supported by OpenTelemetry and is vendor neutral, following the protocol was a logical choice for Firefly.

{{< figure src="/assets/adot-layer-detailed.png" caption="Figure 8.8 ADOT layer detailed" >}}

Lambdas are typically invoked through function URLs, HTTP requests to API gateways, the AWS SDK or SQS/SNS messages. They may similarly use those methods to pass events off to other services.

OpenTelemetry's context propagation API expects the `traceparent` header to be in the `headers` object of a request. When a request follows the standard HTTP format, the `traceparent` header can be found without issue. This is the case for functions invoked via a URL or an API gateway.

{{< figure src="/assets/traceparent-header-response.png" caption="Figure 8.9 traceparent header in response (source: https://uptrace.dev/opentelemetry/opentelemetry-traceparent.html)" >}}

Functions invoked through the AWS SDK or via SQS/SNS messages on the other hand require additional instrumentation to extract the header. In SQS/SNS messages, the `traceparent` header is injected in a non-standard location. This results in the context extractor not being able to find the header, therefore using a new context to create the span. The span is assigned a new trace ID and therefore context propagation is lost.

{{< figure src="/assets/adot-layer-invocation.png" caption="Figure 8.10 ADOT layer responding to invocation through AWS SDK or SQS/SNS message" >}}

Firefly solves this by wrapping the user's function and intercepting any incoming requests. From there, the `traceparent` is parsed and the span created by the instrumentation is reassigned to the correct context. The firefly wrapper then invokes the user's function as intended.

{{< figure src="/assets/firefly-layer.png" caption="Figure 8.11 Firefly layer" >}}

With AWS SDK invocations, OpenTelemetry propagators are able to inject the `traceparent` header into the request. Unfortunately, the invoked function does not receive the full request but instead only a designated payload that doesn’t contain the header. We decided to solve this by creating a custom wrapper function for the AWS SDK Lambda Invoke function, which injects the `traceparent` into the payload that the function receives. Users wishing to invoke their Lambda via SDK will need to use the wrapper function to ensure context is propagated.

This introduces limitations and potential areas of future work:
* It is likely other AWS resource invocations via the SDK will have the same issue: injection of context in an unparsable location. If context needs to be propagated to those other services, work will need to be done to wrap those applicable SDK functions.
* The Lambda invoking function is only available within the Firefly Lambda layer. Non-Lambda services, instrumented with OpenTelemetry, still need access to this function if invoked via the AWS SDK while passing context. Therefore the wrapper function should be made importable via a separate library.
* If a code base already widely uses the SDK to invoke Lambdas, updating it to use the wrapper function will be tedious and resource intensive. Using a mechanism to hook into the SDK module would be a more user-friendly approach to explore.

### 8.3 Database options

The list of options for potential databases for telemetry data is vast; however, the majority focus on storing either metric or trace data–not both. We therefore had a choice, opt to have two databases, or find one that could store both.

{{< figure src="/assets/database-options.png" caption="Figure 8.12 Database options" >}}

We looked at some of the most popular projects such as Thanos, Cortex, VictoriaMetrics, and Jaeger. All excelled in their particular niche and would have been fantastic options if we chose to focus on one telemetry data type. In the end, we decided to choose a database that could accommodate both metric and trace data. This was to reduce complexity, centralise data collection, and also to provide a uniform interface in which users could use to build their own queries for both data types.

We narrowed the field to three options: Elasticsearch, Amazon Timestream, and Promscale. Elasticsearch and Timestream were both viable candidates, but ultimately Promscale was chosen due to fitting our use case best. Promscale is a connector–built by Timescale–which facilitates the storage of metric and trace data in TimescaleDB.

Elasticsearch was a little too heavyweight for our needs, with most of its features simply not being necessary for our use case.  While it accepted both trace and metric data as well as supporting OpenTelemetry, it was less an observability backend and more geared towards a fullstack solution.  Integration of Elasticsearch would have required more work to set up multiple components to achieve the same results as other alternatives out-of-the-box.

Timestream fell by the wayside in a few key areas in comparison to Promscale. TimescaleDB had 5-175x the query speed, 6000x higher inserts and was also up to 220x cheaper, it was therefore a simple choice. We also wanted to avoid having our pipeline dependent on the AWS cloud. Allowing users to host and own their data anywhere they wanted would allow Firefly to easily expand to other serverless function platforms without lock-in to a specific cloud provider.

The query language used was also a major deciding factor. Promscale supports queries in SQL, PromQL and Jaeger query. SQL's continued pervasiveness in the industry meant we could support as many engineers as possible right out of the box.

{{< figure src="/assets/language-breakdown.png" caption="Figure 8.13 Most popular programming languages (source: https://survey.stackoverflow.co/2022/#technology-most-popular-technologies)" >}}

There were a number of tradeoffs we made in choosing Promscale, such as project maturity, TimescaleDB being more CPU hungry, query performance underperforming databases focused solely on one metric type, among others. However, we felt it more than made up for these areas with its refined focus, ease-of-use, ability to store both metric and trace data types, and the strong team surrounding the project.

---

## 9. Future work {#future_work}

Firefly currently works well for AWS Lambda functions using Node.js, we do however have a number of areas in which we would like to expand Firefly's capabilities, these are:

* Extend language support to Python, Java and Go
* Provide observability to other serverless function providers such as Microsoft, Google Cloud and Cloudflare
* Implement metric emission through firefly's lambda layer once OpenTelemetry matures further

---

## 10. References {#references}

https://martinfowler.com/articles/microservices.html
https://about.gitlab.com/topics/serverless/
https://www.sumologic.com/blog/microservices-vs-serverless-architecture/
https://about.gitlab.com/topics/serverless/
https://www.datadoghq.com/knowledge-center/serverless-architecture/serverless-microservices/
https://stackify.com/telemetry-tutorial/
https://mediatemple.net/blog/cloud-hosting/serverless-benefits-and-challenges/
https://www.oreilly.com/library/view/what-is-serverless/9781491984178/ch04.html
https://opentelemetry.io/docs/concepts/what-is-opentelemetry/
https://www.timescale.com/blog/timescaledb-vs-amazon-timestream-6000x-higher-inserts-175x-faster-queries-220x-cheaper/
https://docs.victoriametrics.com/FAQ.html

---

## Team {#team}