## CloudFront
### CloudFront functions vs Lambda Edge
- **Lambda Edge**: are lambda functions that override the behaviour of request and responses 
- **CloudFront functions**: Lightweight edge functions for high-scale, latency-sensitive CDN customizations. CloudFront Functions are cheaper, faster, but more limited than Lambda edge functions.

## CloudFront Functions

- **Programming languages**: JavaScript (ECMAScript 5.1 compliant)
- **Event sources**:
    - Viewer request/response
- **Scale**: 10,000,000 requests per second or more
- **Function duration**: Submillisecond
- **Maximum memory**: 2 MB
- **Maximum size function code + libs**: 10 KB
- **Network, File System, and Request Body access**: No
- **Access to geolocation and device data**: Yes
- **Can build/test entirely within CloudFront**: Yes
- **Function logging and metrics**: Yes
- **Pricing**: Free tier available; charged per request
## Lambda Edge
- **Programming languages**: Node.js and Python
- **Event sources**:
    - Viewer request/response
    - Origin request/response
- **Scale**: Up to 10,000 requests per second per Region
- **Function duration**:
    - Up to 5 seconds (viewer request and response)
    - Up to 30 seconds (origin request and origin response)
- **Maximum memory**: 128–3,008 MB
- **Maximum size function code + libs**:
    - 1 MB (viewer request and viewer response)
    - 50 MB (origin request and origin response)
- **Network, File System, and Request Body access**: Yes
- **Access to geolocation and device data**:
    - No (viewer request)
    - Yes (origin request/response, and viewer response)
- **Can build/test entirely within CloudFront**: No
- **Function logging and metrics**: Yes
- **Pricing**: No free tier; charged per request, function duration
Note:CloudFront uses **Edge Locations**. Lambda edge uses **Edge Locations** and **Regional Edge Caches**. 

## EBS
![[Pasted image 20241001174016.png]]

## Amazon MSK (low priority)
Amazon Managed Streaming for Apache Kafka is a fully managed service that enables you to build and run application that use Apache Kafka to process streaming data. 
Amzon utilizes Zookeper servers.

Amazon MSK:
- Provisioned: you managed the broker instances 
- Serverless: you pay for what you use, and you don't have to manage instances

**Bootstrap brokers** refers to list of brokers endpoints that an Apache Kafka client use as a starting point to connect to the cluster.

**Amazon MSK Connect:** is a feature of Amazon MSK that makes it easy for developers to stream data to and from their apache kafka clusters.

- **Apache kafka**: Open source streaming platform to create high performance data pipelines, streaming analytics, data integration, and mission critical applications. (Developed by Linkedin). based on Consumer/Producer architecture.

## Keyspace
![[Pasted image 20241003125003.png]]


## Step Function
- Step Function Inputs and Outputs: receive JSON event data as input and pass JSON as output
	- You can manipulate JSON payload 
		- InputhPath: allows us to select what we plan to pass to current step
		- Parameters: allows to construct key paris. When you want to use JSONPATH for parameters you  need to add ".$" to the key name
		- ResultSelector: lets you create a collection of key value pairs, where the values are static or selected from the state's result.
		- ResultPath: Let's you decide to
			- Use only the output from a task
			- Use the input as the output
			- Use the output and add or have it replace an existing key in the input and have that as the output
		- OutputPath:
	- JSONPath is a query languafe for JSON, similar to Xpath for XML
## AWS Compute Optimizer
Analyzes the current configuration of your AWS Compute resources, and their utilization metrics from Amazon CloudWatch over a period of the last 14 days. 
It will reccomend specific configuration changes

![[Pasted image 20241004120413.png]]

**Schedule Expression**
You can create EventBridge Rules that trigger on a schedule. You can think of it as Serverless Cron Jobs. Use UTC time zone. **support cron and rate expression**:
- Cron expression: Very fine Grain control
- Rate expression: Easy to set, not as fine grained

**CloudTrail Event**:
Event Pattern: are used to filter what events should be used to pass along to a target. You can filter events by providing the same fields and values

**EventBridge Event Pattern**: Prefix Matching, Anything-but matching, Numeric matching, IP address matching, exists matching, empty value matching.

**EventBridge Rules:** Up to 5 target, you may have additional fields to select target, you can specify what gets passed along with Configure Input.

**EventBridge Partner:** A list of third pary service providers can be integrated

**Schema registry:** allows you to create, discover and manage OpenAPI Schema for events on EventBridge. You can set schema in order to see if the sctructure of the events have changed over time

**Amazon CloudWatch Alarms:** Alarms are used to trigger notifications for any metric 
- Use example:
	- **Auto Scaling**: increase or decrease EC2 instances “desired” count 
	- **EC2 Actions**: stop, terminate, reboot or recover an EC2 instance 
	- **SNS notifications**: send a notification into an SNS topic 
		- You can create a billing alarm • **Billing data metric is stored in CloudWatch us-east-1** Billing data are for overall worldwide AWS costs • It’s for actual cost, not for projected costs
- **Conditions:**
	![[Pasted image 20241004123829.png]]
- CloudWatch are alarms that watch other alarms. Using composite alarms can help you reduce alarm noise.

### VPC Lattice 
Is a fully managed application networking service that you use to connect, secure, and monitor the services for your application -> Turn your AWS resources into services for a micro services architecture

- Traffic Mirroring: sends a copy network traffic from a source ENI to target ENI, or UDP-enabled NLB or GWLB
- Route 53 Resolver DNS Firewall: Firewall that protect against DNS exfiltration of your data
	- **DNS exfiltration**: DNS data exifiltration is a method used by hackers to steal data from an IT system or network by exploiting the DNS protocol by hiding malicious code within DNS packets.
 ![[Pasted image 20241001111926.png]]
- AWS Network firewall: is a stateful, managed, network firewall and IDS/IPS for VPCs (use open source Suricata )
## Audit Manager
Continually audit your AWS usage to simplify risk and compliance assessment. 
AWS Audit Manager contains: Framework Library, Control Library.
You can create assesments to review the evidence collected and generate an assesment report.

## AWS ECS
### Components
- **Execution role**: used to prepare or manage the contianer
- **Task role:** is the role that is used by the running compute of the container (when the container is running)
- **ECS capacity providers** manage the scaling of the infracture for tasks in your clusters.
- **Task Definition:**
	- Family, Execution role, Task role, Network Mode, CPU and Memory, Requires compatibilities, Container definition
	- Container Definition
		- Name, Dockerhub, Essential, Health Chekcs, port mapping, log configuration, environment, secrets
- **ECS Exec**: allows you to direct interact with containers without needing to first interact with the host conatiner operating system, open inbound ports, or manage SSH Keys. cannot be used with AWS console.
- **Log configuration**: You can set log driver that tells where the container should log. There are other third-party log driver. AWS log is blocked by default you need to configure it to enable it.
- **ECS Service Connect**: makes it easy to setup a service mesh for service-to-service communication. Evolution of AppMesh.
	- Service Connect will deploy a sidecar proxy container. You can use the service discovery name to easily talk to other service
- **ECS Optimized AMIs** are preconfigured with the requirements and reccomnadations to run your container, they are used by default.
- **ECS Anywhere**: allows you to register external VMS residing from you on-premises network to your cluster
	- $0.01025 per hour for each managed ECS Anywhere on-premises instance
	- You can register an external instance to a single cluster
	- External instances require an IAM role that allows them to communicate with AWS APIs
	- Service load balancing isn't supported.
	- EFS volumes aren't supported