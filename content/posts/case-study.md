---
title: "Firefly case study"
date: 2022-04-18T17:12:09-07:00
draft: true
---

# Firefly Case Study {#case_study}

---

## 1. Introduction {#introduction}

Serverless functions have become a key part of distributed architectures. Unfortunately, due to their stateless, ephemeral nature they are difficult to observe with standard observability approaches. They can behave like a black box in your system. Firefly is an open-source observability framework for serverless functions. It provides key insights into serverless function health through the use of metrics and traces--illuminating the black box.

{{< figure src="/assets/firefly-introduction.png" alt="Figure 1.0 Firefly introduction" caption="Figure 1.0 Firefly introduction" >}}

---

## 2. Microservices & Serverless {#microservices_and_serverless}

### 2.1 Microservices

The ability to build a microservice architecture has become more attainable with the rise of cloud compute–due to the abstraction of deployment and management of physical infrastructure away from the end-user–offering an alternative to monolith architectures. **Microservices** are now commonplace and represent a distributed architectural approach in which a single application is composed of a number of loosely coupled, independently deployable and scalable services. Each service has a clearly defined role, executes its own processes, and often communicates via an API. <sup>[1](#ref_1)</sup>

{{< figure src="/assets/microservice-architecture.png" alt="Figure 2.1 Mircoservice architecture" caption="Figure 2.1 Mircoservice architecture" >}}

### 2.2 Serverless

**Serverless** is a development approach that allows for applications to be created and deployed without worrying about the underlying infrastructure of the system<sup>[2](#ref_2)</sup>. In this model, the control in provisioning, maintaining, and scaling the server is typically handed off to a third-party cloud provider for a fee. Developers only have to provide application code which will be packaged and deployed by the third-party provider.

A microservices architecture can be composed of services deployed on their own servers, as well as services which are serverless. Serverless microservices are often built with one or more **serverless functions**–small pieces of code designed to be asynchronously triggered based on events.

{{< figure src="/assets/serverless-microservice-architecture.png" alt="Figure 2.2 Serverless mircoservice architecture" caption="Figure 2.2 Serverless mircoservice architecture" >}}

### 2.3 Functions-as-a-Service

**Functions-as-a-Service (FaaS)** describe the cloud-based service that hosts and manages serverless functions. To deploy a serverless function to a FaaS provider, a developer writes code that fulfills a particular purpose in the application and defines an event that will trigger the function<sup>[3](#ref_3)</sup>. Once the event is triggered, the service provider starts a new instance of the function, work is performed, and the result is returned.

{{< figure src="/assets/faas-system.gif" alt="Figure 2.3 FaaS system" caption="Figure 2.3 FaaS system" >}}

As more events are triggered, the FaaS provider is responsible for **automating the provisioning** of computation resources and **horizontally scaling** function instances in response to demand; this eliminates the need for frequent hands-on management.

This is contrary to a system where a service is deployed on a server which the developer has to manage. The server is kept continuously running, ready to respond to any incoming requests. If demand increases more than the server can handle, requests are ultimately refused and scaling will need to happen manually.

{{< figure src="/assets/single-server.gif" alt="Figure 2.4 Single server system" caption="Figure 2.4 Single server system" >}}

FaaS providers only allocate compute resources when required. When demand decreases and the function sits idle, resources are deallocated and functions do not use compute. Serverless functions do not need to reserve and hold computational resources in advance, as in a traditional set up.

{{< figure src="/assets/function-cycle.png" alt="Figure 2.5 Serverless function cycle" caption="Figure 2.5 Serverless function cycle" >}}

This auto-scale, auto-provision, pay-per-use model allows developers to save time due to being free from implementing, troubleshooting, and maintaining servers.

Unfortunately, the inability to access or modify the cloud provider’s environment and infrastructure makes it challenging for a developer to collect data about serverless functions compared to applications running on traditional servers; a different set of strategies is required.

---

## 3. What is Observability? {#what_is_observability}

### 3.1 Observability Overview

> “[**Observability** is the] measure of how well you can understand and explain any state your application can get into, no matter how novel, or bizarre.” - Majors et al., Observability Engineering<sup>[4](#ref_4)</sup>

A software system—when reduced to its most basic purpose—should take an input, perform some form of computation and produce an output. Observability is concerned with the data generated during this process. Using this data, we are able to gain a better understanding of the internal state of the system from the outside. Tracking down potential issues and onboarding new engineers becomes easier the more observable a system is.

Data generated by the system, pertaining to its behaviour, is referred to as **telemetry data**. In order to generate this data, a system must be instrumented. **Instrumentation** is the process of adding code to your system in order for it to generate and emit telemetry data. Observability focuses on three main telemetry data types: metrics, traces and logs.

{{< figure src="/assets/three-pillars.png" alt="Figure 3.1 Three pillars of observability" caption="Figure 3.1 Three pillars of observability" >}}

### 3.2 Metrics

**Metrics** are numeric, quantifiable measurements that reflect the health of the infrastructure aggregated over a defined period of time. They are key performance indicators for a system, enabling engineers to identify patterns and anomalies. While they are able to quickly indicate the presence of an issue, they are not sufficient for pinpointing the exact source of the problem within a program. Examples of metrics include invocations, errors and duration.

{{< figure src="/assets/main-dashboard.png" alt="Figure 3.2 Firefly's main dashboard" caption="Figure 3.2 Firefly's main dashboard" >}}

### 3.3 Logs

A **log** is a time stamped message which can be either structured or unstructured. It is a record of a specific, pre-configured event that happened at a particular time during the request lifecycle. This allows them to provide granular insights that a metric cannot; however their volume of generation causes them to be difficult to manage. Logs aren’t useful for tracking a request through its lifecycle but come in handy once a greater amount of granularity is required.

{{< figure src="/assets/logs-1.png" alt="Figure 3.3 An example of logs from AWS CloudWatch" caption="Figure 3.3 An example of logs from AWS CloudWatch" >}}

### 3.4 Traces

**Distributed tracing** is the process of tracking a single request as it makes its way through the different parts of an application. A **trace** is the term used to describe the visual representation of this tracked request. Due to the nature of distributed systems, reproducing failures locally can be overly complex. Tracing significantly reduces this complexity and makes debugging considerably easier as we are able to break down each step in a given request. Let’s look at the system below for a clearer picture:

{{< figure src="/assets/distributed-application.png" alt="Figure 3.4 Distributed application" caption="Figure 3.4 Distributed application" >}}

Here we see the components that make up the system, but we aren’t able to see how a request would travel through it, or which point in the request’s life cycle causes the request to fail. This is where distributed tracing comes in.

{{< figure src="/assets/distributed-trace.png" alt="Figure 3.5 Distributed trace" caption="Figure 3.5 Distributed trace" >}}

The above displays a trace in its entirety. Each component is referred to as a span. A **span** represents a unit of work and the time taken to complete that work. The first span in a trace is the **root span**–it generally represents a request from start to finish. So in this case the root span is the web application. The spans underneath are **child spans**; each child represents a unit of work within that request’s journey.

By combining each service's span into a distributed trace we are able to see the request's progression through the system. The request flow for the above trace is as follows:

- `Web Application` calls service `A`
- Service `A` calls service `B` and waits for a response
- Service `A` calls service `C` and `D`

In order for spans to become a distributed trace, certain metadata representing a shared state (**context**) needs to be passed at each part of the request execution. This can then be used to correlate the spans into a single trace. This mechanism is known as **context propagation**. Requests carry this information throughout their lifecycle, allowing each generated span to have access to the shared context. Context propagation is what makes distributed tracing possible across a distributed system.

{{< figure src="/assets/context-propagation.png" alt="Figure 3.6 Context propagation" caption="Figure 3.6 Context propagation" >}}

---

## 4. Observability for Serverless Functions {#observability_for_serverless_functions}

### 4.1 Why is it Important?

Observing serverless functions is essential if we want to get an overall picture of a system’s health. For an entire system to be observable, each individual part must be observable. In other words, each part must emit telemetry data.

{{< figure src="/assets/complete-observability.png" alt="Figure 4.1 Complete observability" caption="Figure 4.1 Complete observability. When an issue occurs in Service A, the error can be identified in Function 2." >}}

Due to the ubiquity of serverless functions in distributed architectures, increasing their observability is necessary if we are to have an accurate idea of how an entire system is performing. Using observability tools can help identify slow-running functions, allowing for optimization. Failing to instrument functions would make distributed traces lose context, errors go undetected, and key metrics remain unseen. Ultimately, this would make tracking down issues harder and lead to false positives for issues in downstream services.

{{< figure src="/assets/partial-observability.png" alt="Figure 4.2 Partial observability" caption="Figure 4.2 Partial observability. When an issue occurs, it is hard to track down the source of the error as we have no insight into the serverless functions of Service A." >}}

### 4.2 Why is it Challenging?

While the case for serverless function observability seems clear, their unique nature makes increasing observability via tried-and-true approaches challenging<sup>[5](#ref_5), [6](#ref_6), [7](#ref_7)</sup>.

Many current solutions for adding observability to services use agents to collect and emit telemetry data. Agents are processes which run independently of the service but within the same environment. It collects and temporarily stores telemetry data generated by the service and its environment, emitting it to the observability system.

The inability to access the function’s environment in FaaS results in the lack of ability to install an agent. Furthermore, the ephemeral nature of the serverless function results in any temporarily stored telemetry data to be lost when the cloud provider destroys the function instance. Working around these issues requires difficult manual instrumentation and attempting to tap into the cloud provider’s ecosystem for relevant environment-related data.

In addition, serverless functions’ event-based invocations make tracing and context propagation difficult compared to microservices which often communicate via HTTP requests. HTTP provides a well-established standard for communication and how information should be transferred from one service to the next.

{{< figure src="/assets/microservice-communication.png" alt="Figure 4.3 Microservice communication through HTTP" caption="Figure 4.3 Microservice communication through HTTP" >}}
{{< figure src="/assets/event-invocation.png" alt="Figure 4.4 Event based invocation of a serverless function" caption="Figure 4.4 Event based invocation of a serverless function" >}}

On the other hand, the events which invoke the functions must carry context in a way that is accessible to the instrumentation of that function. The result which is returned by the function must also carry context in a way that is readable by downstream functions or services. Both scenarios in FaaS are vendor-specific, and context propagation would need to account for this specificity to be able to create an unbroken trace across the lifecycle of the request. In a system which may span multiple cloud ecosystems, a vendor-agnostic approach must be found.

---

## 5. Existing Solutions {#existing_solutions}

Developers wishing to increase observability of their serverless functions typically have three main options:

- Software-as-a-Service solutions
- In-house, vendor-specific solution from FaaS provider
  - We will be discussing specifically AWS CloudWatch as Firefly currently [only supports AWS Lambdas](#firefly_focus)
- Do-it-yourself (DIY)

### 5.1 Software-as-a-Service

Software-as-a-service (SaaS) solutions usually require minimal setup to use. Once you have an account they will generally provide code and processes to manually instrument your functions or set up auto-instrumentation. These vendors normally have an extensive feature set, encompassed in a central UI, that goes well beyond serverless functions. They will also manage the data pipeline, so scaling for increased load does not have to be taken into account.

{{< figure class="center" src="/assets/saas-logos.png" alt="Figure 5.0 Logos of existing Software-as-a-Service solutions" caption="Figure 5.0 Logos of existing Software-as-a-Service solutions" >}}

While these vendors provide many benefits to their users, there are some downsides. The first is cost, which may have a greater impact for smaller companies and projects. The lack of visibility and transparency in their pricing can give observability platforms unpredictable and large costs<sup>[8](#ref_8)</sup>; sometimes stretching up to 50% of your total infrastructure costs<sup>[9](#ref_9)</sup>. Another downside is the loss of data control and ownership, as it is all sent to a third party–a deal breaker for businesses that place importance on data privacy and vendor flexibility.

### 5.2 In-House Vendor-Specific (AWS CloudWatch)

{{< figure class="inline" src="/assets/cloudwatch-logo.png" alt="Figure 5.1 Amazon CloudWatch" caption="Figure 5.1 Amazon CloudWatch" width="100px" >}}

Cloud providers are no strangers to the problem of observability. Many provide in-house solutions for users to conveniently gain insight to services deployed within their ecosystem.

AWS provides [CloudWatch](https://aws.amazon.com/cloudwatch/) as its native observability solution for its serverless functions. The advantage of using CloudWatch is that there is no setup required. Metrics and logs are collected by default, and tracing (via AWS X-Ray) can be enabled through the AWS console for each function you wish to collect trace data for.

Despite being easy to set up, CloudWatch has numerous disadvantages. Like SaaS solutions, billing can be unpredictable and confusing, with separate billing for a variety of things such as queries, events, dashboards, data volume, and alarms. It can therefore quickly become difficult to track exactly how much you will be subsequently charged and where those charges have stemmed from.

CloudWatch offers no global overview of your telemetry data, with each telemetry type being separated into its own dashboard. Being able to make associations between different types of telemetry data make it easier to diagnose potential problems; using CloudWatch means you have to switch between dashboards to do this, adding unnecessary friction.

Using CloudWatch also means your observability data is siloed within AWS, which makes extracting it more difficult as you do not have full control over it. This is problematic if a system also contains non-AWS hosted services, as observability data across the whole system cannot be cohesively accessed from a single source. To get a picture of your overall system health each component needs to be observable, having one solution that is able to incorporate your entire architecture starts to become paramount.

### 5.3 DIY

A do-it-yourself solution can alleviate many of the issues seen with a SaaS provider. Open-source tools such as OpenTelemetry can be used to receive, process, and emit telemetry data. Using a DIY solution provides the greatest amount of freedom, as you can choose exactly how you want to setup your system and have full control over the data emitted. This not only keeps usage costs down but provides much better control. The one major downside of a DIY approach is the sheer amount of work it takes to setup. There is a considerable amount of prior research that needs to be undertaken. Then, once done, there could be weeks or months of work required to perfect your chosen setup.

{{< figure class="center" src="/assets/back-door-lockin.png" alt="Figure 5.2 DIY solution cartoon" caption="Figure 5.2 DIY solution cartoon ([source: Good Tech Things](https://www.goodtechthings.com/backdoor-lock/))" width="500px">}}

---

## 6. Introducing Firefly {#introducing_firefly}

Firefly is an open-source observability framework for serverless functions. It provides key insights into serverless function health through the use of metrics and traces. Firefly automates function instrumentation and the deployment of a telemetry pipeline, ultimately presenting your function data in an easy-to-use dashboard.

Firefly is designed for small companies and projects looking to observe their serverless functions without committing to a commercially available observability solution. It aims to strike a balance between the ease-of-use of the SaaS solution, with the low cost of the DIY solution.

{{< figure src="/assets/solution-comparison.png" alt="Figure 6.0 Solution comparison" caption="Figure 6.0 Solution comparison" >}}

### 6.1 Firefly's Focus {#firefly_focus}

Firefly focuses on a few key areas:

<div class="indent">

#### Serverless Functions

Firefly is not a complete observability tool covering your entire stack. We have, however, used open-source, vendor-neutral tools such as OpenTelemetry to ensure Firefly remains extensible, so if a user wished to grow the scope of the project to include other components they could.

#### Node.js AWS Lambda Functions

With AWS being the largest cloud provider we felt it made sense to build out support for its serverless functions first. Node.js was chosen due its popularity, with the plan to extend support to other languages in the future.

#### Metrics and Traces

Metrics and traces provide a comprehensive overview of serverless function health. We determined logs to be too granular for our current use case.

</div>

### 6.2 Setup

Firefly’s setup involves two main components:

1. Deploy the telemetry pipeline to a host of your choice
2. Run Firefly’s command line interface

See our [GitHub page](https://github.com/try-firefly) for more information.

### 6.3 Usage

Upon loading Grafana you will be greeted with Firefly's main dashboard. This provides a general overview of all your serverless functions through highlighting key metrics such as invocations, errors and duration.

{{< figure src="/assets/main-dashboard.png" alt="Figure 6.1 Main dashboard" caption="Figure 6.1 Main dashboard" >}}

Clicking on an individual function will lead you to Firefly's function dashboard. This allows you to dive into a specific function’s metrics and traces in much greater depth.

{{< figure src="/assets/function-dashboard.png" alt="Figure 6.2 Function dashboard" caption="Figure 6.2 Function dashboard" >}}

Scrolling down the dashboard you will be able to see data for concurrent executions and throttles, as well as traces for individual invocations. If there is an error, it will be highlighted on the right-hand side of the trace table in the exception column.

{{< figure src="/assets/function-exceptions.png" alt="Figure 6.3 Function traces and exceptions" caption="Figure 6.3 Function traces and exceptions" >}}

Clicking on an individual trace will take you to the trace view, where you can further inspect each span and see the entire request lifecycle.

{{< figure src="/assets/trace-view.png" alt="Figure 6.4 Trace view" caption="Figure 6.4 Trace view" >}}

Clicking on a span will show detailed information about that particular point in the request’s journey, enabling you to diagnose potential errors with greater ease.

{{< figure src="/assets/span-error-expanded.png" alt="Figure 6.5 Span error expanded" caption="Figure 6.5 Span error expanded" >}}

---

## 7. Architecture {#architecture}

Telemetry systems, such as Firefly, have three main phases within their architecture: emitting, shipping and presentation<sup>[10](#ref_10)</sup>.

The **emitting** phase handles the instrumentation of application code so telemetry data can be generated. The data will have its format and content determined by this phase before it is sent to the shipping phase.

The **shipping** phase receives data sent from the emitting phase, processes and transforms it, then stores it within a database.

The final phase is the **presentation** phase. This is where telemetry data is retrieved from storage and displayed to the end user in charts, graphs and tables. This enables the user to get an overview of their system health and help them pinpoint failures.

Firefly includes each of these individual phases within its architecture, as can be seen below.

{{< figure src="/assets/firefly-architecture.png" alt="Figure 7.0 Firefly architecture" caption="Figure 7.0 Firefly architecture" >}}

### 7.1 Emit Phase: Traces

The emit phase is split into two different parts, as metrics and traces are not collected and emitted in the same manner.

Trace emission is a two step process:

1. The function must be instrumented to create and emit spans
2. Spans must be assigned the correct context from upstream services to ensure they can be joined together into a distributed trace–otherwise known as context propagation.

For instrumentation Firefly uses [OpenTelemetry](https://opentelemetry.io/docs/concepts/what-is-opentelemetry/). **OpenTelemetry** is a free and open-source collection of tools, APIs, and SDKs used to instrument, generate, collect and export telemetry data. It is an industry standard for observability, created in response to increasing vendor variability in instrumentation resulting in vendor lock-in. We chose to incorporate OpenTelemetry because of its vendor neutrality and strong community.

The key piece we use within our architecture is the **OpenTelemetry collector**. It receives, processes and exports data in multiple formats. A general overview of the collectors composition can be seen below.

{{< figure src="/assets/otel-collector.png" alt="Figure 7.1 OpenTelemetry collector" caption="Figure 7.1 OpenTelemetry collector" >}}

We use the collector in two locations within Firefly’s architecture.

{{< figure src="/assets/firefly-arch-collectors.png" alt="Figure 7.2 OpenTelemetry collectors within architecture" caption="Figure 7.2 OpenTelemetry collectors within architecture" >}}

Within the emit phase, the OpenTelemetry collector is deployed via the AWS Distro for OpenTelemetry (ADOT). ADOT acts as a form of middleware to instrument the Lambda function for tracing. It is an AWS supported distribution of the OpenTelemetry project, designed to provide auto-instrumentation of AWS resources and services.

Within ADOT is a version of the OpenTelemetry collector and additional code which allows for auto-instrumentation. The ADOT collector only supports a select few receivers, processors and exporters to better align to the limited storage space within a Lambda deployment. This component is ultimately responsible for emitting spans from the Lambda to the shipping phase in OpenTelemetry format via HTTP.

The ADOT is deployed as a Lambda layer. This terminology is specific to AWS, but a Lambda layer is just a way to package code that you can use with your Lambda functions. They can contain a number of different things such as libraries, custom runtimes, data or even configuration files.

For context propagation, Firefly uses a custom Lambda layer. The code within this layer acts as a wrapper to the function code to better handle context propagation, as the instrumentation provided by the ADOT layer wasn't able to do so.

{{< figure src="/assets/lambda-layers.png" alt="Figure 7.3 Lambda layers" caption="Figure 7.3 Lambda layers" >}}

### 7.2 Emit Phase: Metrics

AWS Lambdas automatically report metrics to Amazon CloudWatch–AWS's native observability solution. In order to extract metrics from CloudWatch we use two different AWS services: the Amazon CloudWatch Metric Stream and Amazon Kinesis Data Firehose.

The Amazon CloudWatch Metric Stream is a feature designed to emit metric updates as they occur. It was introduced to provide an alternative to intermittent polling of CloudWatch’s API for metric data. As soon as CloudWatch receives an update, the metric stream kicks into action and emits the data to an Amazon S3 bucket or, in Firefly’s case, an Amazon Firehose.

The Firehose is a service used in conjunction with the Metric Stream to receive stream data, transform it, and then send it to a desired endpoint–not too dissimilar from what the OpenTelemetry collector does. Using the Firehose, Firefly can direct the metric stream data to the shipping phase in JSON format via HTTPS.

{{< figure src="/assets/metric-emission.png" alt="Figure 7.4 Metric emission" caption="Figure 7.4 Metric emission" >}}

### 7.3 Shipping Phase

As mentioned, we use two OpenTelemetry collectors within our architecture. The second collector is a custom OpenTelemetry collector built to include specific components for Firefly's use case. It acts as the gateway to the shipping phase.

{{< figure src="/assets/shipping-phase.png" alt="Figure 7.5 Shipping phase" caption="Figure 7.5 Shipping phase" >}}

The collector sends the telemetry data it receives to the database. Firefly uses a single database, Promscale, for both metrics and traces. Promscale acts as a connector built on top of a timeseries database, built for long-term storage of telemetry data.

Using the OpenTelemetry collector as the gateway to the shipping phase ensures compatibility between telemetry data from the emit phase and Promscale. The emit phase has two emit sources: the ADOT layer (OpenTelemetry format) and the Amazon Firehose (JSON format). Promscale only accepts metric data in the Prometheus format, therefore the data sent from the Firehose needs to be transformed from JSON to Prometheus. The collector allows Firefly to perform this transformation and send the appropriately formatted data to Promscale for storage.

{{< figure src="/assets/otel-collector-gateway.png" alt="Figure 7.6 Gateway OpenTelemetry collector" caption="Figure 7.6 Gateway OpenTelemetry collector" >}}

### 7.4 Presentation Phase

The presentation phase has one aim: retrieve and display data. Firefly uses Grafana–an open-source monitoring solution and visualization tool–in order to present the collected metric and trace data. A number of dashboards specifically focused on serverless functions are pre-built and loaded at start up. While Firefly's dashboards provide all the necessary data to hunt down a serverless function issue, Grafana allows users to add to these dashboards, build their own, or perform ad-hoc queries.

Grafana has the ability to query multiple data types and sources, with different query interfaces for each. The three most applicable to Firefly are the metrics (Prometheus), traces (Jaeger) and PostgreSQL sources. Firefly pre-loads Grafana with the appropriate configuration for all three sources. This allows users who are more familiar with querying their telemetry data in a certain format to do so. With this, users are able to use the full capabilities of Grafana without sacrificing the convenience of Firefly.

{{< figure src="/assets/presentation-phase.png" alt="Figure 7.7 Presentation phase" caption="Figure 7.7 Presentation phase" >}}

### 7.5 Overall Firefly Architecture

{{< figure src="/assets/firefly-architecture.png" alt="Figure 7.8 Firefly Architecture" caption="Figure 7.8 Firefly Architecture" >}}

Combining all phases creates a complete telemetry system. Each phase had its own challenges and tradeoffs, some of which are discussed in greater detail below.

---

## 8. Technical Challenges & Tradeoffs {#challenges_and_tradeoffs}

### 8.1 Emission and Shipment of Metrics

Firefly made the decision to use OpenTelemetry's tooling to build out the necessary infrastructure to emit and receive telemetry data. The reason for this was twofold: it's open source and the industry standard. We therefore intended to build a Lambda layer that would emit both trace and metric data with OpenTelemetry specifications. Unfortunately, OpenTelemetry's Lambda support for instrumentation of metrics in Node.js was still experimental at the time and proved difficult to manage within the scope of the project.

Despite this, we did not want to drop OpenTelemetry. By using the OpenTelemetry collector as the gateway to the shipping phase of our observability system we ensured extensibility. Users with existing components instrumented using OpenTelemetry would have a far easier time plugging them into Firefly. Alternatively, they could use Firefly's command line interface to automatically instrument their functions and set up the necessary AWS infrastructure to send telemetry data to their existing OpenTelemetry-compliant observability system.

We therefore had to find a solution that would enable us to emit metric data from AWS while also being able to receive it using an OpenTelemetry collector. Two mechanisms were available to us: push versus pull.

In a pull based mechanism, sources emitting metrics need to expose an endpoint from which data can be “pulled” by a scraper. In a push based mechanism, the data sources actively push metrics out to a receiver once data is generated and available. The OpenTelemetry collector has built-in capability to handle both scenarios, acting as a receiver or scraper that can then send the collected data to the backend for storage. As Lambda metrics were automatically generated and sent to Amazon CloudWatch, attempting to either push or pull from CloudWatch became a clear choice.

{{< figure src="/assets/pull-method.png" alt="Figure 8.1 Pull method" caption="Figure 8.1 Pull method" >}}

A key hurdle with the pull mechanism was the lack of an HTTP endpoint exposing metrics from CloudWatch. Since no such endpoint natively existed, an intermediary exporter needed to be used to fetch and transform the data. This information then needed to be made available via an HTTP endpoint. We identified the [yet-another-cloudwatch-exporter](https://github.com/nerdswords/yet-another-cloudwatch-exporter) (YACE) to be a good option.

The main advantage of YACE was how specific it allowed us to be; each metric we wished to include could be listed in a configuration file. There were, however, a number of downsides. Using YACE resulted in having to poll an API to receive metric data. This polling caused a domino effect, resulting in increased latency in sending metrics to the collector and the need to manage polling intervals. The management of polling intervals is an important balancing act. Too many calls is wasteful and causes API throttling; making too few adds unnecessary latency.

Looking towards other observability solutions<sup>[11](#ref_11)</sup>, we determined a push based approach would be the better option. Amazon CloudWatch’s metric stream service in conjunction with Amazon Firehose has the ability to aggregate and send metrics to an HTTPS endpoint on a per minute basis only when new data is generated. This ensures we quickly receive data when activity is high, and eliminate wasteful API calls when there is no activity. While there is little ability to specify exactly which Lambda metrics we wanted to receive and a nominal cost is incurred per metric update, this was the better option.

{{< figure src="/assets/push-method.png" alt="Figure 8.2 Push method" caption="Figure 8.2 Push method" >}}

Using a receiver built by the OpenTelemetry community we were also able to receive this data using the OpenTelemetry collector. While the receiver is still in alpha stage, it proved to be stable enough for our purposes with little issues. The OpenTelemetry collector is battle-tested and has extensive functionality, with over 70 receivers in various stages of development. We therefore chose this extensibility over initial robustness.

### 8.2 Context Propagation

Distributed tracing is the act of tracking a request across a system. Context propagation is the means by which this is achieved. As a request goes from service to service, a context object is passed along with it. This is often performed using dedicated HTTP headers embedded in the request. The headers contain formatted information that usually includes a trace ID as well as the parent span’s ID. Downstream services can parse this information to appropriately assign the correct trace and hierarchy as it generates spans.

{{< figure src="/assets/context-passing.png" alt="Figure 8.3 Context passed from service to service" caption="Figure 8.3 Context passed from service to service" >}}

Several different protocols exist for context propagation; each protocol has its own convention for headers and formatting for associated values. Services communicating with each other must ensure they share the same protocol for propagation to be successful.

#### How it Should Work:

The AWS Distro for OpenTelemetry (ADOT) layer within Firefly’s architecture acts as the middleware for tracing and context propagation for the Lambda. Using OpenTelemetry API components, the layer should parse context information from incoming requests, create spans with the appropriate context and then inject the context in outgoing requests to downstream services.

{{< figure src="/assets/adot-layer.png" alt="Figure 8.4 ADOT layer" caption="Figure 8.4 ADOT layer" >}}

#### What Actually Happens:

By default, AWS Lambdas and the ADOT layer default to using [AWS X-Ray's context propagation method](https://docs.aws.amazon.com/xray/latest/devguide/xray-concepts.html) with AWS specific headers; however, those headers are not natively supported by OpenTelemetry. This creates a problem: with the ADOT layer, Lambdas can only parse OpenTelemetry supported context formats while injecting AWS formatted context to outgoing requests.

{{< figure src="/assets/adot-layer-aws-headers.png" alt="Figure 8.5 Default ADOT layer behaviour" caption="Figure 8.5 Default ADOT layer behaviour" >}}

#### How Firefly Overcame the Issue:

Fortunately, ADOT allows users to switch from using X-Ray's context propagation method to OpenTelemetry supported [W3C TraceContext propagation](https://www.w3.org/TR/trace-context/).

{{< figure src="/assets/adot-layer-w3c-headers.png" alt="Figure 8.6 ADOT layer using W3C headers" caption="Figure 8.6 ADOT layer using W3C headers" >}}

The W3C protocol uses a header called `traceparent` containing the trace ID and parent span’s ID to pass context.

{{< figure src="/assets/traceparent-header.png" alt="Figure 8.7 traceparent header" caption="Figure 8.7 traceparent header" >}}

By switching propagation methods, OpenTelemetry’s context propagation API is able to look for the presence of the `traceparent` header to parse context appropriately, as well as injecting a new `traceparent` header for outgoing requests. Because it is supported by OpenTelemetry and is vendor neutral, following the protocol was a logical choice for Firefly.

Lambdas are typically invoked through function URLs, HTTP requests to API gateways, the AWS SDK or SQS/SNS messages. They may similarly use those methods to pass events off to other services.

OpenTelemetry's context propagation API expects the `traceparent` header to be in the `headers` object of a request. When a request follows the standard HTTP format, the `traceparent` header can be found without issue. This is the case for functions invoked via a URL or an API gateway.

{{< figure src="/assets/traceparent-header-response.png" alt="Figure 8.8 traceparent header in response" caption="Figure 8.8 traceparent header in response ([source: uptrace.dev](https://uptrace.dev/opentelemetry/opentelemetry-traceparent.html))" >}}

Functions invoked through the AWS SDK or via SQS/SNS messages on the other hand require additional instrumentation to extract the header. In SQS/SNS messages, the `traceparent` header is injected in a non-standard location. This results in the context extractor not being able to find the header, therefore using a new context to create the span. The span is assigned a new trace ID and therefore context propagation is lost.

{{< figure src="/assets/adot-layer-invocation.png" alt="Figure 8.9 ADOT layer responding to invocation through AWS SDK or SQS/SNS message" caption="Figure 8.9 ADOT layer responding to invocation through AWS SDK or SQS/SNS message" >}}

Firefly solves this by wrapping the user's function and intercepting any incoming requests. From there, the `traceparent` is parsed and the span created by the instrumentation is reassigned to the correct context. The firefly wrapper then invokes the user's function as intended.

{{< figure src="/assets/firefly-layer.png" alt="Figure 8.10 Firefly layer" caption="Figure 8.10 Firefly layer" >}}

With AWS SDK invocations, OpenTelemetry propagators are able to inject the `traceparent` header into the request. Unfortunately, the invoked function does not receive the full request but instead only a designated payload that doesn’t contain the header. We decided to solve this by creating a custom wrapper function for the AWS SDK Lambda Invoke function, which injects the `traceparent` into the payload that the function receives. Users wishing to invoke their Lambda via SDK will need to use the wrapper function to ensure context is propagated.

### 8.3 Database options

The list of options for potential databases for telemetry data is vast; however, the majority focus on storing either metric or trace data–not both. We therefore had a choice, opt to have two databases, or find one that could store both.

{{< figure src="/assets/two-databases.png" alt="Figure 8.11 Two databases to handle metrics and traces separately" caption="Figure 8.11 Two databases to handle metrics and traces separately" >}}
{{< figure src="/assets/single-database.png" alt="Figure 8.12 Single database to handle both metrics and traces" caption="Figure 8.12 Single database to handle both metrics and traces" >}}

We looked at some of the most popular projects such as Thanos, Cortex, VictoriaMetrics, and Jaeger. All excelled in their particular niche and would have been fantastic options if we chose to focus on one telemetry data type. In the end, we decided to choose a database that could accommodate both metric and trace data. This was to reduce complexity, centralise data collection, and provide a single source of truth from which users could build their own queries for both telemetry data types.

{{< figure src="/assets/custom-query.png" alt="Figure 8.13 Custom query" caption="Figure 8.13 Custom query" >}}

We narrowed the field to three options: Elasticsearch, Amazon Timestream, and Promscale. Elasticsearch and Timestream were both viable candidates, but ultimately Promscale was chosen due to fitting our use case best. Promscale is a connector–built by Timescale–which facilitates the storage of metric and trace data in TimescaleDB.

Elasticsearch was a little too heavyweight for our needs, with most of its features simply not being necessary for our use case. While it accepted both trace and metric data as well as supporting OpenTelemetry, it was too broad in its feature set while lacking the specificity we needed. Integration of Elasticsearch would have required more work to set up multiple components to achieve the same results as other alternatives out-of-the-box.

Timestream fell by the wayside in a few key areas in comparison to Promscale. TimescaleDB had 5-175 times the query speed, 6000 times higher inserts and was also up to 220 times cheaper, it was therefore a simple choice<sup>[12](#ref_12)</sup>. We also wanted to avoid having our pipeline dependent on the AWS cloud. Allowing users to host and own their data anywhere they wanted would allow Firefly to expand to other serverless function platforms without lock-in to a specific cloud provider.

The query language used was also a major deciding factor. Promscale supports queries in SQL, PromQL and Jaeger query. SQL's continued pervasiveness in the industry meant we could support as many engineers as possible right out of the box.

There were a number of tradeoffs we made in choosing Promscale, such as project maturity, TimescaleDB being more CPU hungry, query performance underperforming databases focused solely on one metric type, among others<sup>[13](#ref_13)</sup>. However, we felt it more than made up for these areas with its refined focus, ease-of-use, ability to store both metric and trace data types, and the strong team surrounding the project.

---

## 9. Future work {#future_work}

Firefly currently works well for AWS Lambda functions using Node.js, we do however have a number of areas in which we would like to expand Firefly's capabilities, these are:

- Extend language support to Python, Java and Go
- Provide observability to other serverless function providers such as Microsoft, Google Cloud and Cloudflare
- Implement metric emission through firefly's lambda layer once OpenTelemetry matures further
- Improvements to context propagation via an importable library and more user-friendly auto-instrumentation
- Enable customization of emitted telemetry data

---

## 10. References {#references}

- https://stackify.com/telemetry-tutorial/

<div class="references">

<span class="refnum">1</span> <a name="ref_1" /> https://martinfowler.com/articles/microservices.html

<span class="refnum">2</span> <a name="ref_2" /> https://about.gitlab.com/topics/serverless/

<span class="refnum">3</span> <a name="ref_3" /> https://www.sumologic.com/blog/microservices-vs-serverless-architecture/

<span class="refnum">4</span> <a name="ref_4" /> https://info.honeycomb.io/observability-engineering-oreilly-book-2022

<span class="refnum">5</span> <a name="ref_5" /> https://lumigo.io/aws-serverless-ecosystem/understanding-serverless-observability/

<span class="refnum">6</span> <a name="ref_6" /> https://www.datadoghq.com/knowledge-center/serverless-architecture/serverless-microservices/

<span class="refnum">7</span> <a name="ref_7" /> https://mediatemple.net/blog/cloud-hosting/serverless-benefits-and-challenges/

<span class="refnum">8</span> <a name="ref_8" /> https://www.argonaut.dev/blog/observability-comparison-2022

<span class="refnum">9</span> <a name="ref_9" /> https://www.honeycomb.io/blog/how-much-should-my-observability-stack-cost

<span class="refnum">10</span> <a name="ref_10" /> https://livebook.manning.com/book/software-telemetry/part-1/v-5

<span class="refnum">11</span> <a name="ref_11" /> https://newrelic.com/topics/how-to-monitor-prometheus

<span class="refnum">12</span> <a name="ref_12" /> https://www.timescale.com/blog/timescaledb-vs-amazon-timestream-6000x-higher-inserts-175x-faster-queries-220x-cheaper/

<span class="refnum">13</span> <a name="ref_13" /> https://docs.victoriametrics.com/FAQ.html

</div>

---

## Team {#team}
