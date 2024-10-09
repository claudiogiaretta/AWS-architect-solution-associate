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