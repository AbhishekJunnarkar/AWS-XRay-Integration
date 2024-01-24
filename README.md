# Understanding the power of AWS X-Ray: Simplifying Application performance monitoring
![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+) **Introduction**

In todays fast paced digital landscape, delivering seamless and high performing applications is crucial for businesses to stay competitive. As orgniazations transition to cloud based infrastructures, the need for robust monitoring and debugging tools becomes paramount. AWS X-Ray emerges as a game-changer in this realm, offering a comprehensive solution for tracing, analyzing, and troubleshooting applications on Amazon Web Services(AWS).

![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+)  **What is AWS X-Ray?**

AWS X-Ray is a powerful service that provides end-to-end visibility into distributed applications, allowing developers to identify performance bottlenecks, pinpoint errors, and gain deep insights into application behavior. By tracing requests as they travel through various components of a distributed system, X-Ray enables developers to understand the dependencies and performance of each component.

![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+) **Key Features and Benefits**

1. **End-to-End Tracing**: X-Ray traces requests across AWS services and microservices, offering a detailed view of how requests flow through an application.

2. **Performance Insights**: It provides real-time insights into application performance, highlighting areas that require optimization, thus enhancing overall efficiency.

3. **Debugging Capabilities**: Developers can quickly identify issues by analyzing traces, allowing for faster debugging and resolution of errors.

4. **Compatibility and Integration**: X-Ray seamlessly integrates with popular AWS services and supports various programming languages, ensuring flexibility and ease of use.

![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+) **How does it work?**

AWS X-Ray operates by instrumenting the code within applications, capturing data and generating a visual representation of the application's components and their interactions. This data is presented in a trace map, displaying a detailed view of each component's performance metrics and dependencies. Developers can leverage this information to diagnose performance issues, optimize resource utilization, and enhance user experiences.

## **Use case for Serverless computing**
To effectively debug distributed appplications in production environment, there is no need to go through complex logs or do filtering or indexing on them when visual analysis is possible with AWS X-Ray.

The AWS X-Ray SDK does not send trace data directly to AWS X-Ray. To avoid calling the service every time your application serves a request, the SDK sends the trace data to a daemon, which collects segments for multiple requests and uploads them in batches. Use a script to run the daemon alongside your application.

To properly instrument your applications in Amazon ECS, you have to create a Docker image that runs the X-Ray daemon, upload it to a Docker image repository, and then deploy it to your Amazon ECS cluster. You can use port mappings and network mode settings in your task definition file to allow your application to communicate with the daemon container.

The AWS X-Ray daemon is a software application that listens for traffic on UDP port 2000, gathers raw segment data, and relays it to the AWS X-Ray API. The daemon works in conjunction with the AWS X-Ray SDKs and must be running so that data sent by the SDKs can reach the X-Ray service.

- The correct steps to properly instrument the application is to create a Docker image that runs the X-Ray daemon, upload it to a Docker image repository, and then deploy it to your Amazon ECS cluster.
- In addition, you also have to configure the port mappings and network mode settings in your task definition file to allow traffic on UDP port 2000.

## What is a Segment and Subsegment

A segment document conveys information about a segment to X-Ray. A segment document can be up to 64 kB and contain a whole segment with subsegments, a fragment of a segment that indicates that a request is in progress, or a single subsegment that is sent separately. You can send segment documents directly to X-Ray by using the PutTraceSegments API.

X-Ray compiles and processes segment documents to generate queryable trace summaries and full traces that you can access by using the GetTraceSummaries and BatchGetTraces APIs, respectively. In addition to the segments and subsegments that you send to X-Ray, the service uses information in subsegments to generate inferred segments and adds them to the full trace. Inferred segments represent downstream services and resources in the service map.

A subset of segment fields is indexed by X-Ray for use with filter expressions. For example, if you set the user field on a segment to a unique identifier, you can search for segments associated with specific users in the X-Ray console or by using the GetTraceSummaries API.

A segment can break down the data about the work done into subsegments. Subsegments provide more granular timing information and details about downstream calls that your application made to fulfill the original request. A subsegment can contain additional details about a call to an AWS service, an external HTTP API, or an SQL database. You can even define arbitrary subsegments to instrument specific functions or lines of code in your application.

Subsegments represent your application's view of a downstream call as a client. If the downstream service is also instrumented, the segment that it sends replaces the inferred segment generated from the upstream client's subsegment. The node on the service graph always uses information from the service's segment, if it's available, while the edge between the two nodes uses the upstream service's subsegment.

- Segments and subsegments can include a metadata object containing one or more fields with values of any type, including objects and arrays.

- Segments and subsegments can include an annotations object containing one or more fields that X-Ray indexes for use with filter expressions.

Below are the optional subsegment fields:

- namespace - aws for AWS SDK calls; remote for other downstream calls.

- http - http object with information about an outgoing HTTP call.

- aws - aws object with information about the downstream AWS resource that your application called.

- error, throttle, fault, and cause - error fields that indicate an error occurred and that include information about the exception that caused the error.

- annotations - annotations object with key-value pairs that you want X-Ray to index for search.

- metadata - metadata object with any additional data that you want to store in the segment.

- subsegments - array of subsegment objects.

- precursor_ids - array of subsegment IDs that identify subsegments with the same parent that was completed prior to this subsegment.

You can use the "metadata" field in the segment section to add custom data for your tracing. If you want to trace all the AWS SDK calls of your application, then you can add a subsegment and set the "namespace" field to "AWS". Alternatively, you can set the "namespace" value to "remote" if you want to trace other downstream calls.

## How AWS Lambda communicates with Amazon X-Ray
AWS Lambda uses environment variables to facilitate communication with the X-Ray daemon and configure the X-Ray SDK.

- _X_AMZN_TRACE_ID: Contains the tracing header, which includes the sampling decision, trace ID, and parent segment ID. If Lambda receives a tracing header when your function is invoked, that header will be used to populate the _X_AMZN_TRACE_ID environment variable. If a tracing header was not received, Lambda will generate one for you.

- AWS_XRAY_CONTEXT_MISSING: The X-Ray SDK uses this variable to determine its behavior in the event that your function tries to record X-Ray data, but a tracing header is not available. Lambda sets this value to LOG_ERROR by default.

- AWS_XRAY_DAEMON_ADDRESS: This environment variable exposes the X-Ray daemon's address in the following format: IP_ADDRESS:PORT. You can use the X-Ray daemon's address to send trace data to the X-Ray daemon directly without using the X-Ray SDK.

- AWS_XRAY_DEBUG_MODE is used to configure the SDK to output logs to the console without using a logging library.

- AUTO_INSTRUMENT is primarily used in X-Ray SDK for Django Framework only. This allows the recording of subsegments for built-in database and template rendering operations.

## Annotations

Annotations are simple key-value pairs that are indexed for use with filter expressions. Use annotations to record data that you want to use to group traces in the console, or when calling the GetTraceSummaries API.
X-Ray indexes up to 50 annotations per trace.

## Metadata 
Metadata are key-value pairs with values of any type, including objects and lists, but that is not indexed. Use metadata to record data you want to store in the trace but don't need to use for searching traces.

## Segments 
The computing resources running your application logic send data about their work as segments. A segment provides the resource's name, details about the request, and details about the work done.

## Sampling 
To ensure efficient tracing and provide a representative sample of the requests that your application serves, the X-Ray SDK applies a sampling algorithm to determine which requests get traced. By default, the X-Ray SDK records the first request each second, and five percent of any additional requests.

## **Conclusion**
AWS X-Ray empowers developers and DevOps teams to proactively monitor and optimize their applications, ultimately improving reliability and delivering better user experiences. By providing a detailed view of application performance, it enables businesses to swiftly identify and address issues, ensuring that their applications run smoothly even under high loads.

In an era where user expectations for application performance are at an all-time high, AWS X-Ray stands as a fundamental tool in the arsenal of modern cloud-based applications, enabling businesses to stay agile, competitive, and customer-focused.

## Cover
One of the most revolutionary but underutilized AWS service is AWS X-Ray.

Refere
https://tutorialsdojo.com/instrumenting-your-application-with-aws-x-ray/?src=udemy#instrumenting-your-java-app
