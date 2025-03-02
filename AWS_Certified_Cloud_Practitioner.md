^^ = da guardare video per capirne un po' di più
IOPS = Input/output operations per second


# Lista Servizi

## EC2 and other

1. **IAM** (Identity Access Management): Servizio **globale**, per la gestione degli account delle autorizzazioni date a ciascuno di essi.
2. **EC2 (Elastic Compute Cloud)**: Servizio per affittare macchine virtuali
	1. Type of Instances
		- **Compute Optimized:** Great for compute-intensive tasks that require high performance
		- **Memory Optimized:** Great for compute-intensive tasks that require high performance (Use case: cache, database)
		- **Storage Optimized:** Storage optimized instances are designed for workloads that require high, sequential read and write access to very large data sets on local storage. They are optimized to deliver tens of thousands of low-latency, random I/O operations per second (IOPS) to applications.
	2. Type of purchasing option:
		- **On-Demand Instances** : Has **high cost but no upfront payment**, Recommended for short-term and un-interrupted workloads commitment
		- **Reserved (1 & 3 years)**: up to 72% save, long workloads/ long workloads with flexible instances
		- **Savings Plans (1 & 3 years)** :commitment to an amount of usage, long workload, up to 72%
		- **Spot Instances** : short workloads, cheap, can lose instances (less reliable), up to 90%
		- **Dedicated Hosts** : **The most expensive**, book an entire physical server, control instance placement, let you use your software license
		- **Dedicated Instances** : no other customers will share your hardware
		- **Capacity Reservations** : reserve capacity in a specific AZ for any duration also if you don't use what you payed for
	3. **EC2 Instance Connect:** • Connect to your EC2 instance within your browser via **ssh**, temporary ssh key is uploaded onto EC2 by AWS every acces don't need to load yours


3. **EC2 Image Builder**: Used to automate the creation of Virtual Machines. Also with scheduling. **Free service** (only pay for the underlying resources)
4. **EBS (Elastic Block Store)**: Non global service, allows you to create "EBS volume" which are **network drive** you can attach to your instances while they run. **They can only be mounted to one instance at a time.** **EBS volume is not multi Availability Zone**.
	1. **EBS Snapshot Archive:** Move a Snapshot to an ”archive tier” that is 75% cheaper
	2. **Recycle Bin for EBS Snapshots**: Specify retention (from 1 day to 1 year)
5. **EFS (Elastic Files System)**: Managed NFS (network file system) that can be mounted on 100s of EC2 • EFS works with **Linux** EC2 instances in multi-AZ • Highly available, scalable, expensive (3x gp2), pay per use, no capacity planning
6. **EC2 Instance Store**: **high-performance hardware disk, use EC2 Instance Store**, Better I/O performance. Used as cache, Instance Store lose their storage if they’re stopped
7. **EFS Infrequent Access (EFS-IA)** : cost-optimized for files not accessed every day • Up to 92% lower cost compared to EFS Standard • EFS will automatically move your files to EFS-IA based on the last time they were accessed
8. **FSx**: Launch 3rd party high-performance file systems on AWS. Alternatives, if you don't want to use **s3** or **efs**.
9. **ELB (Elastic Load Balancer)**: is a managed load balancer, a tool to distribute traffic through different servers. Doesn't scale, **High availability across zones**, Expose a single point of access (DNS) to your application
	- **Application Load Balancer** : An ALB operates on OSI layer 7 and allows for application-level traffic manipulation and routing. HTTP / HTTPS / gRPC • HTTP Routing features
	- **Network Load Balancer**: An NLB operates on layer 4 for network-level traffic management based on ports and IP addresses. High Performance: millions of request per seconds
	- **Gateway Load Balancer** : A GLB works across layers 3 and 7, providing balancing and routing services at the network level along with gateway functionality.
10. **Auto Scaling Group (ASG)**: Creation of instances based on amount of workload. **High Availability**
	1. **Manual Scaling:**
	2. **Dynamic Scaling:**
		1. **Simple Scaling:** When a **CloudWatch alarm** is triggered(example CPU > 70%), then add 2 units or/and (example CPU < 30%), then remove 1
		2. **Step Scaling:** Also scale wehn CloudWatch alarm but more rfficient when cloudWatch allarm is **repetedly trigger**
		3. **Target Tracking Scaling:** Example: I want the average ASG CPU to stay at around 40%
		5. **Predictive Scaling**: Triggers scaling by analyzing historical load data to detect daily or weeklu patterns in traffic flows

## S3 storage and Database

1. **S3**: (**not global and SERVERLESS**) Amazon Simple Storage Service (Amazon S3) is an object storage service offering industry-leading scalability, data availability, security, and performance. Buckets are defined at the **region level**. Object are encrypted at server-side level.
	- **Storage Classes:**
		- **S3 Standard:** S3 Standard offers high durability, availability, and low latency object storage for frequently accessed data
		- **Amazon S3 Standard-Infrequent Access (IA)** : S3 Standard-IA is for data that is accessed less frequently, **but requires rapid access when needed.** Use cases: Disaster Recovery, backups 
		- **Amazon S3 One Zone-Infrequent Access** : Use Cases: Storing secondary backup copies of on-premise data, or data you can recreate. **Costs 20% less** than S3 Standard-IA
		- **Amazon Glacier:** Low-cost object storage meant for archiving / backup, **Serverless**
			- **Amazon S3 Glacier Instant Retrieval** :Amazon S3 Glacier Instant Retrieval is an archive storage class that delivers **low cost storage** for long-lived data that is rarely accessed and requires retrieval in milliseconds
			- **Amazon S3 Glacier Flexible Retrieval** : S3 Glacier Flexible Retrieval delivers low-cost storage, up to 10% lower cost (than S3 Glacier Instant Retrieval), for archive data that is accessed 1—2 times per year and is retrieved asynchronously. Expedited (1 to 5 minutes), Standard (3 to 5 hours), Bulk (5 to 12 hours)
			- **Amazon S3 Glacier Deep Archive** : S3 Glacier Deep Archive is Amazon S3’s **lowest-cost storage** class and supports long-term retention and digital preservation for data that may be accessed once or twice in a year. Standard (12 hours), Bulk (48 hours)
		- **Amazon S3 Intelligent Tiering** :
		- **Amazon S3 Express One Zone** :Amazon S3 Express One Zone is a high-performance, single-zone Amazon S3 storage class that is purpose-built to deliver consistent, single-digit millisecond data access for your most latency-sensitive applications. S3 Express One Zone is the lowest latency cloud-object storage class available today, with data access speeds up to 10x faster and with request costs 50 percent lower than S3 Standard
	- **Lifecycle configuration:** Can move between classes manually or using S3 Lifecycle configurations 
	- **Bucket Policies:** Grant public access to the bucket • Force objects to be encrypted at upload • Grant access to another account (Cross Account)
	- **Block Public Access**:The Amazon S3 Block Public Access feature provides settings for access points, buckets, and accounts to help you manage **public access** to Amazon S3 resources. **Override bucket polices**.
	- **IAM Access Analyzer for S3**: • Ensures that only intended people have access to your S3 buckets
	- **Replication:** 
		- **Cross-region Replication**: compliance, lower latency access, replication across account
		- **Single-region-replication**: log aggregation, live replication between production and test accounts
2. **AWS Snow Family**: Highly-secure, portable devices to collect and process data at the edge, and migrate data into and out of AWS
	1. **Snowcone:** Snowcone is the smallest member of the AWS Snow Family of edge computing and data transfer devices. Snowcone is portable, rugged, and secure. You can use Snowcone to collect, process, and move data to AWS, either oﬄine by shipping the device, or online with AWS DataSync.
	2. **Snowball: ** Snowball is a data migration and edge computing device that comes in two device options: Compute Optimized and Storage Optimized.
3. **AWS OpsHub:** AWS OpsHub is a graphical user interface you can use to manage your AWS Snowball devices, enabling you to rapidly deploy edge computing workloads and simplify data migration to the cloud.
4. **AWS Storage Gateway**: is a hybrid storage service that allows your on-premises applications to seamlessly use AWS cloud storage (Bridge between on-premise data and cloud data in s3). Use example: backups to the cloud, using on-premises file shares backed by cloud storage, and providing low-latency access to data in AWS for on-premises applications.
	1. File gateway
	2. Volume gateway
	3. Tape Gateway
5. **RDS (Relational Database Service)**: It’s a managed DB service for DB use SQL as a query language. It allows you to create databases in the cloud that are managed by AWS. (puoi decidere di usa un db relazionale anche senza rds sulle ec2 ma te lo devi gestire tu)
	**Scelte architetturali:**
	1. **Read Replicas:** **disaster recovery** solution if the primary DB instance fails. **Scaling** beyond the compute or I/O capacity of a single DB instance for read-heavy database workloads. Serving read traffic while the source DB instance is unavailable.
	3. **Multi-AZ:** Failover/Disaster recovery in case of AZ outage (**high availability**), can only have 1 other AZ as failover
	4. **Multi-Region (Read Replicas):** **Disaster recovery** in case of region issue, local performance for global reads
7. **Aurora**:  ***Serverless***,• Automated database instantiation and auto -scaling based on actual usage • PostgreSQL and MySQL are both supported as Aurora Serverless DB. **Aurora costs more than RDS (20% more)**
8. **ElastiCache**: Amazon ElastiCache is a web service that makes it easy to set up, manage, and scale a distributed in-memory data store or cache environment in the cloud
9. **DynamoDB**: Amazon DynamoDB is a ***serverless***, NoSQL, fully managed database with single-digit millisecond performance at any scale. **• Fully Managed Highly available with replication across 3 AZ**
10. **DynamoDB Accelerator - DAX**: Fully Managed in-memory cache for DynamoDB • 10x performance improvement – single- digit millisecond latency to microseconds latency – when accessing your DynamoDB tables, Active-Active replication (read/write to any AWS Region)
11. **DynamoDB – Global Tables**: Make a DynamoDB table accessible with low latency in **multiple-regions**, Active-Active replication.
12. **Redshift**: **Serverless,** An Amazon Redshift is an enterprise-class relational database based on postgresql, Is thinked for data warehouse and for OLAP(analytics and data warehousing). Storage is done by column and not by row. **Pay as you go based**
13. **EMR(Elastic MapReduce):** Amazon EMR is a managed cluster **platform that simplifies running big data frameworks**, such as [Apache Hadoop](https://aws.amazon.com/elasticmapreduce/details/hadoop),  on AWS to process and analyze vast amounts of data.
14. **Athena**: Serverless query service to analyze data stored in Amazon S3 that uses standard SQL language to query the files and their content
15. **QuickSight**: ***Serverless*** machine learning-powered business intelligence service to create interactive dashboards
16. **DocumentDB**: Same of aurora but for MongoDB (which is a NoSQL database).
17. **Neptune**: Amazon Neptune is a fast, reliable, fully managed graph database service that makes it easy to build and run applications that work with highly connected datasets.
18. **Timestream**: Amazon Timestream offers fully managed, purpose-built time-series (dates or similar data) database engines for workloads from low-latency queries to large-scale data ingestion. **Serverless**
19. **QLDB(Quantum Ledger Database):** Used to review history of all the changes made to your **application data** over time, serverless. **Difference with Amazon Managed Blockchain**: no decentralization component
20. **Amazon Managed Blockchain**: Amazon Managed Blockchain is a managed service to: Join public blockchain networks or create your own scalable private network
21. **Glue**: Managed extract, transform, and load (ETL) service. Useful to prepare and transform data for analytic (una sorta di adapter). **Serverless**
22. **DMS – Database Migration Service**: AWS Database Migration Service (AWS DMS) is a managed migration and replication service that helps move your database and analytics workloads to AWS quickly, securely, and with minimal downtime and zero data loss.
## Other Compute Section

1. **Elastic Container Service(ECS)**: Amazon Elastic Container Service (Amazon ECS) is a fully managed container orchestration service that helps you easily deploy, manage, and scale containerized applications. Compare to fargate you need to build the infrastructure on your own (EC2 instances).
2. **Fargate**: AWS Fargate is a technology that you can use with Amazon ECS to run [containers](https://aws.amazon.com/what-are-containers) without having to manage servers or clusters of Amazon EC2 instances. With AWS Fargate, you no longer have to provision, configure, or scale clusters of virtual machines to run containers. 
3. **ECR (Elastic Container Registry):** Private Docker Registry on AWS, similar to docker hub 
4. **Lambda**: **Serverless and Global**, Virtual functions, you can use AWS Lambda to run code without provisioning or managing servers. Lambda runs your code on a high-availability compute infrastructure and performs all of the administration of the compute resources, including server and operating system maintenance, capacity provisioning and automatic scaling, and logging. pay per request or pay per compute time
5. **API Gateway**: Amazon API Gateway is an AWS service for creating, publishing, maintaining, monitoring, and securing REST, HTTP, and WebSocket APIs at any scale. API developers can create APIs that access AWS or other web services, as well as data stored in the [AWS Cloud](https://aws.amazon.com/what-is-cloud-computing/).
6. **AWS Batch**: AWS Batch helps you to run batch computing workloads on the AWS Cloud. Batch computing is a common way for developers, scientists, and engineers to access large amounts of compute resources. Rispetto alle Lambda: **No time limit** • • Batch jobs are defined as Docker images and run on ECS • **Rely on EBS and EC2 / instance store for disk space**.
8. **Lightsail**: Amazon Lightsail is the easiest way to get started with AWS for anyone who needs to build websites or web applications. It includes everything you need to launch your project quickly. Great for people with little cloud experience.
## Deploy an managing Infrastructure

1. **CloudFormation**: CloudFormation is a declarative way of outlining your AWS Infrastructure, for any resources (most of them are supported) is a service that helps you model and set up your AWS resources. **Infrastructure as code**
	1. **CloudFormation + Application Composer**: We can see all the resources and the relations between the component
2. **AWS Cloud Development Kit (CDK)**: Define your cloud infrastructure using a familiar language: JavaScript/TypeScript, Python, Java, and .NET The code is “compiled” into a CloudFormation template (JSON/YAML).
3. **Elastic Beanstalk (PaaS)**: With Elastic Beanstalk you can quickly deploy and manage applications in the AWS Cloud without having to learn about the infrastructure that runs those applications. 
4. **AWS CodeDeploy:** CodeDeploy is a deployment service that **automates** application deployments **to Amazon EC2 instances**, works with On-Premises Servers • **Hybrid service**
5. **AWS CodeCommit**: is a version control service hosted by Amazon Web Services that you can use to privately store and manage assets. Non più supportato
6. **AWS CodeBuild:** AWS CodeBuild is a **fully managed** build service in the cloud. CodeBuild compiles your source code, runs unit tests, and produces artifacts that are ready to deploy. **Pay-as-you-go**
7. **AWS CodePipeline**: Orchestrate the different steps to have the code automatically pushed to production, Code => Build => Test => Provision => Deploy, Basis for CICD (Continuous Integration & Continuous Delivery)
8. **AWS CodeArtifact**: AWS CodeArtifact is a secure, highly scalable, managed artifact repository service that helps organizations to store and share software packages for application development. Dependencies management
9. **AWS CodeStar**: (non più supportato). Unified UI to easily manage software development activities in one place
10. **AWS Cloud9**: AWS Cloud9 is an integrated development environment, or _IDE_.
11. **AWS Systems Manager (SSM)**: Helps you manage **your EC2 and On-Premises systems** at scale. **Designed for hybrid solutions environment**. • Patching automation for enhanced compliance • Run commands across an entire fleet of servers. Thanks to the SSM agent, we can run commands, patch & configure our server. We need to install the SSM agent onto the systems we control
## Global Infrastructure Section

1. **Amazon Route 53**: Amazon Route 53 is a highly available and scalable Domain Name System (DNS) web service. You can use Route 53 to perform three main functions in any combination: domain registration, DNS routing, and health checking.
	- **Simple routing policy** – Use for a single resource that performs a given function for your domain, for example, a web server that serves content for the example.com website. You can use simple routing to create records in a private hosted zone.
	- **Failover routing policy** – Use when you want to configure active-passive failover. You can use failover routing to create records in a private hosted zone.
	- **Latency routing policy** – Use when you have resources in multiple AWS Regions and you want to route traffic to the Region that provides the best latency. You can use latency routing to create records in a private hosted zone.
	- **Weighted routing policy** – Use to route traffic to multiple resources in proportions that you specify. You can use weighted routing to create records in a private hosted zone.
	
1. **Amazon CloudFront**: Amazon CloudFront is a web service that speeds up distribution of your static and dynamic web content. When a user requests content that you're serving with CloudFront, **the request is routed to the edge location that provides the lowest latency** (time delay), so that content is delivered with the best possible performance. **DDoS protection (because worldwide), integration with Shield, AWS Web Application Firewall**. Great for static content
2. **S3 Transfer Acceleration:** Increase transfer speed by transferring file to an AWS **edge location** which will forward the data to the S3 bucket in the target region. Metti che hai un sito statico hostato in s3.
3. **AWS Global Accelerator**:• Improve global application **availability** and performance using the AWS global network • Leverage the AWS internal network to optimize the route to your application - 2 Static Anycast IP are created for your application and traffic is sent through **Edge Locations**
4. **AWS Outposts**: Hardware rented by aws to companies that are using an **hybrid cloud** solution in order to benefit from the advantage of using aws ecosystem.
5. **AWS WaveLength**: WaveLength Zones are infrastructure deployments embedded within the telecommunications providers’ datacenters at the edge of the 5G networks • **Brings AWS services to the edge of the 5G networks.** Ultra-low latency applications through 5G networks
6. **AWS Local Zones**: Places AWS compute, storage, database, and other selected AWS services closer to end users to run latency-sensitive applications. • Extend your VPC to more locations **"Extension of an AWS Region"**


Cloudfront usa gli edge location per cachare il contenuto, top per HTTP

Global accelerator usa gli edge location come pro, migliore per contesti che non utilizzano HTTP per esempio il gaming voice over services e altre cose di questo tipo. Usa un IP statico così non devo cambiare le configurazioni del DNS
### Global architecture
Different types:
- Single Region, Single AZ
- Single Region, Multi AZ
- Multi Region, **Active-Passive** (Si intende che si hanno più EC2 in più Regioni, però alcune sono accedute solo in lettura(passive))
- Multi Region, **Active-Active** (Si intende che si hanno più EC2 in più Regioni, entrambe sono accedute sia in lettura che in scrittura)
## Cloud Integration

1. **Amazon SQS**: Fully managed, Amazon Simple Queue Service (Amazon SQS) offers a secure, durable, and available hosted queue that lets you integrate and decouple distributed software systems and components. (Vedi esempio slide per capire meglio).
2. **Amazon Kinesis**: Managed service to collect, process, and analyze real-time streaming data at any scale
3. **Amazon SNS**: Amazon Simple Notification Service (Amazon SNS) is a managed service that provides message delivery from publishers to subscribers (also known as _producers_ and _consumers_). In this way is possible to send one message to many receivers.
4. **Amazon MQ**: Amazon MQ is a managed message broker service that makes it easy to migrate to a message broker in the cloud. Traditional applications running from on-premises may use open protocols. • When migrating to the cloud, instead of re-engineering the application to use SQS and SNS, we can use MQ.
## Cloud Monitoring

1. **Amazon CloudWatch**: **High Availability**, Amazon CloudWatch monitors your Amazon Web Services (AWS) resources and the applications you run on AWS in real time. **Amazon CloudWatch is basically a metrics repository. An AWS service puts metrics into the repository, and you retrieve statistics based on those metrics. If you put your own custom metrics into the repository, you can retrieve statistics on these metrics as well.**.
	1. **Amazon CloudWatch Alarms:(like budget but less powerful)** Alarms are used to trigger notifications for any metric • **Auto Scaling**: increase or decrease EC2 instances “desired” count • **EC2 Actions**: stop, terminate, reboot or recover an EC2 instance • **SNS notifications**: send a notification into an SNS topic 
		1. You can create a billing alarm • **Billing data metric is stored in CloudWatch us-east-1** Billing data are for overall worldwide AWS costs • It’s for actual cost, not for projected costs
	2. **Amazon CloudWatch Logs:** • Enables **real-time monitoring** of logs • Adjustable CloudWatch Logs retention 
		 CloudWatch Logs can collect log from:
		   **EC2 machines or on-premises servers** • Elastic **Beanstalk**: collection of logs from application • **ECS:** collection from containers • **AWS Lambda**: collection from function logs • **CloudTrail** based on filter • **Route53**: Log DNS queries
1. **Amazon EventBridge**: Schedule aws operations based on time or trigger operations in response to event
2. **AWS CloudTrail**: **Can be applied to All Regions**, AWS CloudTrail is an AWS service that helps you enable **operational and risk auditing, governance, and compliance** of your AWS account. Enabled by default, **Get an history of events / API calls made within your AWS Account**.
	1. **Event history** – The **Event history** provides a viewable, searchable, downloadable, and immutable record of the past 90 days of management events
	2. **CloudTrail Lake** – [AWS CloudTrail Lake](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-lake.html) is a managed data lake for capturing, storing, accessing, and analyzing user and API activity
	3. **Trails** – _Trails_ capture a record of AWS activities, delivering and storing these events in an Amazon S3 bucket
3. **AWS X-Ray**:  **Visual analysis** of our applications. When you're application structured become very complex it can be hard to track log files, run test. Help you debug application with complex architecture
4. **Amazon CodeGuru**: An ML-powered service for automated code reviews and application performance reccomandation.
5. **AWS Health Dashboard Service ** : Show general status of your services health • Shows historical information for each day • Has an RSS(easy way for you to keep up with news and information) feed you can subscribe to
6.  **AWS Account Health Dashboard**: Account Health Dashboard gives you a personalized view into the performance and availability for the services you're actually using.

## Security
1.  **VPC Virtual Private Cloud:** private network to deploy your resources (regional resource). **Not global**
2. **Internet Gateway**: helps our VPC instances connect with the internet • Public Subnets have a route to the internet gateway.
3. **NAT Gateways e NAT Instances:** allow your instances in your Private Subnets to access the internet while remaining private
4. **Network access control lists (ACL)**: • A firewall which controls traffic **from and to subnet** • Can have **ALLOW and DENY** rules • Are attached at the Subnet level • Rules only include IP addresses. **FREE**
5. **Security Groups:** A firewall that controls traffic to and from an **EC2 Instance** • Can have only **ALLOW** rules • Rules include IP addresses, ports, in/out traffic • You can assign a security group only to resources created in the same VPC as the security group • It’s good to maintain one separate security group for SSH access
![[Pasted image 20240902145817.png]]
1. **VPC Flow Logs:** Capture information about IP traffic going into your interfaces • Helps to monitor & troubleshoot connectivity issues
2. **VPC Peering:** • Connect two VPC, privately using AWS’ network • Make them behave as if they were in the same network • VPC Peering connection **is not transitive** (2 way)
3. **Transit Gateway:** For having transitive peering between thousands of VPC and on -premises, hub-and-spoke (star) connection. (differente da private link che collega solo servizi)
4. **AWS PrivateLink (VPC Endpoint Services):** AWS PrivateLink establishes private connectivity between virtual private clouds (VPC) and supported AWS services, services hosted by other AWS accounts, and supported AWS Marketplace services. Need **network load balancer** for other VPC and **Elastic Network Interface** for yours
5. **VPC Endpoints:** • Endpoints allow you to connect to AWS Services using a private network instead of the public www network • This gives you enhanced security and lower latency to access AWS services. per fare questo usa: 
	1. **VPC Endpoint Interface:** all services execpt s3 & dynamo
	2. **VPC Endpoint Gateway:** S3 & DynamoDB
7. **Site to Site VPN:** • Connect an on-premises VPN to AWS **going over internet** The connection is automatically **encrypted** • Goes over **the public internet** • It uses a **Virtual Private Gateway** on the VPC side and **Customer Gateway** on the Client side
8. **Direct Connect:** Establish a physical connection between on-premises and AWS **going over private connection**. The connection is private, secure and fast.
9. **AWS Client VPN:** Connect from **your computer** using OpenVPN to your VPC in AWS and on-premises system.
10. **AWS Shield:** • Provides protection from attacks of layer **3** and **4**
	- **AWS Shield Standard:** • Free service that is activated for every AWS customer • Provides protection from attacks such as SYN/UDP Floods, Reflection attacks and other layer 3/layer 4 attacks 
	- **AWS Shield Advanced:** • Optional DDoS mitigation service ($3,000 per month per organization) 
		- **Protect against more sophisticated attack on:**
			- Amazon EC2
			- Elastic Load Balancing (ELB), 
			- Amazon CloudFront
			- AWS Global Accelerator
			- Route 53
		- 24/7 access to AWS DDoS response team (DRP) • Protect against higher fees during usage spikes due to DDoS
11. **AWS WAF – Web Application Firewall:** Protects your web applications from common web exploits **Layer 7**. • Deploy on Application Load Balancer, API Gateway, CloudFront • Protects from common attack - **SQL injection and Cross-Site Scripting (XSS)** • Size constraints, **geo-match** (block countries) • **Rate-based** rules (to count occurrences of events) – for DDoS protection
12. **AWS Network Firewall:** AWS Network Firewall is a stateful, managed, network firewall and intrusion detection and prevention service for your VPC. Protect your entire Amazon VPC • **From Layer 3 to Layer 7 protection**
13. **AWS Firewall Manager:** AWS Firewall Manager simplifies your administration and maintenance tasks across multiple accounts and resources for a variety of protections, including AWS WAF, AWS Shield Advanced, Amazon VPC security groups and network ACLs, AWS Network Firewall, and Amazon Route 53 Resolver DNS Firewall
14. **AWS KMS:** Anytime you hear “encryption” for an AWS service, it’s most likely KMS • KMS = AWS manages the encryption keys for us
	- **Customer Managed Key:**
		- Create, manage and used by the customer, can enable or disable
		- Possibility of rotation policy (new key generated every year, old key preserved)
		- Possibility to bring-your-own-key 
	- **AWS Managed Key:**
		- Created, managed and used on the customer’s behalf by AWS
		- Used by AWS services (aws/s3, aws/ebs, aws/redshift)
	- **AWS Owned Key:**
		- Collection of CMKs that an AWS service owns and manages to use in multiple accounts
		- AWS can use those to protect resources in your account (but you can’t view the keys)
	- **CloudHSM Keys (custom keystore):** 
		- Keys generated from your own CloudHSM hardware device
		- Cryptographic operations are performed within the CloudHSM cluster

15. **CloudHSM:**  A hardware security module (HSM) is a computing device that processes cryptographic operations and provides secure storage for cryptographic keys
16. **AWS Certificate Manager (ACM):** • Let’s you easily provision, manage, and deploy SSL/TLS Certificates • Used to provide in-flight encryption for
17. **AWS Secrets Manager:** • Newer service, meant for storing secrets • Capability to force rotation of secrets every X days • Automate generation of secrets on rotation (uses Lambda) • Integration with Amazon RDS (MySQL, PostgreSQL, Aurora)
18. **AWS Artifact (not really a service):** Portal that provides customers with on-demand access to AWS compliance documentation and AWS agreements
19. **Amazon GuardDuty:** Intelligent **Threat discovery** to protect your AWS Account. Uses Machine Learning algorithms, **anomaly detection**, 3rd party data. **Uses EventBridge and lambda funtion to make action**. Protect from Cryptocurrency
20. **Amazon Inspector:** Amazon Inspector is an **automated** security assessment service that helps improve the security and compliance of applications deployed **on your Amazon EC2 instances**. Amazon Inspector automatically assesses applications for exposure.
21. **AWS Config:** AWS Config provides a detailed view of the configuration of AWS resources in your AWS account. This includes how the resources are related to one another and how they were configured in the past so that you can see how the configurations and relationships change over time.
22. **AWS Macie:** Amazon Macie is a fully managed data security and data privacy service that uses machine learning and pattern matching to discover and protect your sensitive data in AWS.
23. **AWS Security Hub:** Central security tool to manage security across several AWS accounts and automate security checks • Integrated dashboards showing current security and compliance status to quickly take actions
24. **Amazon Detective:** Amazon Detective analyzes, investigates, and quickly identifies the root cause of security issues or suspicious activities (using ML and graphs). More powerful than **GuardDuty, Macie, and Security Hub**
## Account Management

1. **AWS Organizations**: Global service that allows to manage multiple AWS accounts. API is available to automate AWS account creation.
	1. • Cost Benefits: 
		1. [Consolidated Billing](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/consolidatedbilling-other.html)
		2. Pricing benefits from aggregated usage (volume discount for EC2, S3…)
		3. Pooling of Reserved EC2 instances for optimal savings
2. **AWS Control Tower**: **AWS Control Tower is a service that helps customers set up and govern a secure, compliant, and multi-account AWS environment based on best practices established by AWS. It automates the process of setting up a well-architected multi-account AWS.** Give you a dashboard that let you manage your accounts• It automatically sets up AWS Organizations to organize accounts and implement SCPs (Service Control Policies)
3. **AWS Resource Access Manager (AWS RAM):** Share AWS resources that you own with other AWS accounts. **Avoid resource duplication**. Supported resources includes: Aurora, VPC Subnets, Transit Gateway, Route 53, EC2 Dedicated Hosts, License Manager Configurations
4. **AWS Service Catalog**: Users that are new to AWS have too many options, administrator can create a catalog of services available for him.
5. **Savings Plan**: Commit a certain $ amount per hour for 1 or 3 years • Easiest way to setup long-term commitments on AW. You can save up to 72 percent on your AWS compute workloads.
6. **AWS Compute Optimizer** : Reduce costs and improve performance by **recommending** optimal AWS resources for your workloads. Uses Machine Learning to analyze your resources’ configurations and their utilization CloudWatch metrics. Lower your costs by up to 25%
7. **AWS Budgets**: Allows to set alarms based on cost threshold that you decide. **4 types of budgets: Usage, Cost, Reservation, Savings Plans**
8. **Billing and Costing Tools**: overview of your AWS cloud financial management data and to help you make faster and more informed decisions, **only one month of data**.
9. **Cost Explorer**: AWS Cost Explorer is a tool that enables you to view and analyze your costs and usage. You can explore your usage and costs using the main graph, the Cost Explorer cost and usage reports, or the Cost Explorer RI reports. **You can view data for up to the last 13 months (less powerful than cost Cost and Usage Reports)**, forecast how much you're likely to spend for the next 12 months, and get recommendations for what Reserved Instances to purchase.
10. **Cost and Usage Reports**: The AWS Cost and Usage Reports (AWS CUR) contains ***the most comprehensive*** set of cost and usage data available. Analyze your data at a high level: total costs and usage **across all accounts**.
11. **AWS Cost Anomaly Detection**: Continuously monitor your cost and usage using **ML** to detect unusual spends. **It learns your unique, historic spend patterns** to detect one-time cost spike and/or continuous cost increases (you don’t need to define thresholds)
12. **AWS Service Quotas**: Notify you when you’re close to a service quota value threshold. Create CloudWatch Alarms on the Service Quotas console
13. **Trusted Advisor**: No need to install anything – high level AWS account assessmen, AWS Trusted Advisor helps you in the following section: **Cost optimization • Performance • Security • Fault tolerance • Service limits • Operational Excellence**
14. **AWS Support Plans**: AWS Support provides a mix of tools and technology, people, and programs designed to proactively help you optimize performance, lower costs, innovate faster, and focus on solving some of the toughest challenges that hold you back in your cloud journey. There are multiple support plans to choose
	- **AWS Basic Support Plan:** Customer Service & Communities - 24x7 access to customer service,• AWS Trusted Advisor - Access to the 7 core Trusted Advisor checks • AWS Personal Health Dashboard
	- **AWS Developer Support Plan:** All Basic Support Plan + • Business hours email access to Cloud Support Associates • Case severity / response times: (• General guidance: < 24 business hours • System impaired: < 12 business hours)
	- **AWS Business Support Plan:** • Intended to be used if you have production workloads • Trusted Advisor – Full set of checks + API access • 24x7 phone, email, and chat access to Cloud Support Engineers
	- **AWS Enterprise On-Ramp Support Plan:** All of Business Support Plan +, Access to a pool of Technical Account Managers (TAM), **Concierge Support Team** (for billing and account best practices).
	- **AWS Enterprise Support Plan:** Infrastructure Event Management, Well-Architected & Operations Reviews

### Encryption
**Encryption Opt-in:** 
- EBS volumes: encrypt volumes 
- Redshift database: encryption of data 
- RDS database: encryption of data 
- EFS drives: encryption of data 

**Encryption Automatically enabled:** 
- CloudTrail Logs 
- S3 Glacier 
- S3 buckets: Server-side encryption of objects 
- Storage Gateway
	- File Gateway 
	-  Volume Gateway 
	-  Tape Gateway
## Advanced identity Section

1. **AWS STS(SecurityToken Service)**: **Global service**, enables you to create temporary, limited- privileges credentials to access your AWS resources. 
	1. you can only access it programmatically.
	2. Short-term credentials: you configure expiration period. Guarantee temporary access to other company ecc...
3. **Amazon Cognito (simplified)**: Identity for your Web and Mobile applications (Logins) users, often attach to a user databases.  
4. **AWS Directory Services**: Service used to create Microsoft Active Directory  in aws in order to make it easier to connect with on-premises servers
5. **AWS IAM Identity Center**: Single sign-on for multi account in aws 
## Machine Learning

1. **Amazon Rekognition**:Find objects, people, text, scenes in images and videos using ML • Facial analysis and facial search to do user verification, people counting
2. **Amazon Transcribe**: **Automatically convert speech to text** • Uses a deep learning process called automatic speech recognition (ASR) to convert speech to text quickly and accurately • Automatically remove Personally Identifiable Information (PII) using Redaction • Supports Automatic Language Identification for multi-lingual audio
3. **Amazon Polly**: Turn text into lifelike speech using deep learning • Allowing you to create applications that talk
4. **Amazon Translate**: Natural and accurate language translation
5. **Amazon Comprehend**: Fully managed and serverless service for Natural Language Processing. Uses machine learning to find and analyse insights and relationships in text. use cases: analyze customer interactions (emails) to find what leads to a positive or negative experience
6. **Amazon SageMaker**: Fully managed service for developers / data scientists to build ML models
7. **Amazon Forecast**: Fully managed service that uses ML to deliver highly accurate forecasts • predict the future sales of a raincoat
8. **Amazon Kendra**: Fully managed **document search** service powered by Machine Learning
9. **Amazon Personalize**: Fully managed ML-service to build apps with real-time personalized recommendations
10. **Amazon Textract**: Automatically extracts text, handwriting, and data from any scanned documents using AI and ML
11. **Amazon Connect:** Amazon Connect is an AI-powered application that provides one seamless experience for your contact center customers and users. It's comprised of a full suite of features across communication channels.
## Other services

1. **Amazon WorkSpaces**: Gives you virtual machine, in order to substitute work infrastracture. 
2. **Amazon AppStream 2.0**: Enable you to open application using the browser without connecting to a virtual machine and download them.
3. **AWS IoT Core**: Connect to multiple devices.
4. **Amazon Elastic Transcoder**: Elastic Transcoder is used to convert media files stored in S3 into media files in the formats required by consumer playback devices
5. **AWS AppSync**: Store and sync data across mobile and web apps in real-time. Makes use of GraphQL (mobile technology from Facebook)
6. **AWS Amplify**: A set of tools and services that helps you develop and deploy scalable full stack web and mobile applications
7. **AWS Application Composer**: **Visually design and build serverless applications quickly on AWS.** Deploy AWS infrastructure code without needing to be an expert in AWS 
8. **AWS Device Farm**: Give you access to virtual mobile devices and environment to testing mobile and web app.
9. **AWS Backup**: Cross-region, Fully-managed service to centrally manage and automate backups across AWS services
10. **AWS Elastic Disaster Recovery (DRS)**: Let you to recover quickly servers through continuous replication. Most critical things. 
	![[Pasted image 20240909175158.png]]
11. **AWS DataSync**: Move large amount of data from on-premises to AWS, client needs **AWS DataSync Agent**
12. **AWS Application Discovery Service**: AWS Application Discovery Service helps you **plan your migration** to the AWS cloud by collecting usage and configuration data about your on-premises servers and databases. Application Discovery Service is integrated with AWS Migration Hub and AWS Database Migration Service Fleet Advisor.
13. **AWS Application Migration Service (MGN)**:AWS Application Migration Service (MGN) is a **highly automated lift-and-shift** (rehost) solution that simplifies, expedites, and reduces the cost of migrating applications to AWS. It allows companies to lift-and-shift a large number of physical, virtual, or cloud servers without compatibility issues, performance disruption, or long cutover windows
14. **AWS Migration Evaluator**: identifies the most cost-effective options, provides visibility into multiple migration strategies, and captures insights
15. **AWS Migration Hub**: Central location to collect servers and applications inventory data for the assessment, planning, and tracking of migrations to AWS
16. **AWS Fault Injection Simulator (FIS)**: A fully managed service for running fault injection experiments on AWS workloads. Based on Chaos Engineering – stressing an application by creating disruptive events (e.g., sudden increase in CPU or memory), observing how the system responds, and implementing improvements
17. **AWS Step Functions**: Build serverless visual workflow to orchestrate your Lambda functions
18. **AWS Ground Station**: Fully managed service that lets you control satellite communications, process data, and scale your satellite operations. Provides a global network of satellite ground stations near AWS regions
19. **Amazon Pinpoint**: Scalable 2-way (outbound/inbound) marketing communications service. Supports email, SMS, push, voice, and in-app messaging. Ability to segment and personalize messages with the right content to customers. Use cases: run campaigns by sending marketing, bulk, transactional SMS messages
20. **AWS Managed Services (AMS)**: Provides infrastructure and application support on AWS. AMS offers a team of AWS experts who manage and operate your infrastructure for security, reliability, and availability

#### AWS Professional Services & Partner Network (APN)
• The AWS Professional Services organization is a global team of experts 
• They work alongside your team and a chosen member of the APN 
• APN = AWS Partner Network 
• **APN Technology Partners:** providing hardware, connectivity, and software 
• **APN Consulting Partners:** professional services firm to help build on AWS 
• **APN Training Partners:** find who can help you learn AWS 
• **AWS Competency Program:** AWS Competencies are granted to APN Partners who have demonstrated technical proficiency and proven customer success in specialized solution areas. 
• **AWS Navigate Program:** help Partners become better Partners

# Lista Tools and Concepts

- **Edge Location**: AWS Edge Locations are **strategically positioned points in the AWS network optimized for low-latency content delivery**, ensuring that data reaches users swiftly
- **Access Key ID**: username
- **Secret Access Key**: password
- **User Data EC2:** where to insert script for bootstrapping ec2 instances
- **Elastic IP**: Let you connect ipv4 IP to ec2 instances
- **IAM Credentials Report (account-level):**  a report that lists all your account's users and the status of their various credentials
- **IAM Access Advisor (user-level):** Access advisor shows the service permissions granted to a user and when those services were last accessed.
- **Multi-factor-autentichation**: Virtual MFA device, Universal 2nd Factor (U2F) Security Key, Hardware MFA devices (MFA devices options in AWS, Hardware Key Fob MFA Device for AWS GovCloud (US)).
- **AMI(Amazon Machine Image):** customization of an EC2 instance, **built for a specific region** (and can be copied across regions)
- **Cost Allocation Tags**: Use cost allocation tags to track your AWS costs on a detailed level • AWS generated tags • Automatically applied to the resource you create • Starts with Prefix aws: (e.g. aws: createdBy) • User-defined tags • Defined by the user • Starts with Prefix user
- **Tags**: are used for labelling resources, act like metadata.
- **Resources groups:** You can use _resource groups_ to organize your AWS resources. AWS Resource Groups is the service that lets you manage and automate tasks on large numbers of resources at one time.
# AWS Architecting & Ecosystem Section

1. **AWS Well-Architected Tool:** Free tool to review your architectures against the 6 pillars Well-Architected Framework and adopt architectural best practice
2. **AWS Customer Carbon Footprint Tool**: Track, measure, review, and forecast the Carbon emissions generated from your AWS usage
3. **AWS Right Sizing**: 
4. **AWS Training:**
5. **AWS Professional Services & Partner Network (APN):**
6. **AWS IQ:** Quickly find professional help for your AWS projects, 3rd party experts for on-demand project work
7. **AWS rePost:** AWS-managed Q&A service offering crowd -sourced, expert -reviewed answers to your technical questions about AWS that replaces the original AWS Forums. **Free tier**.
8. **AWS Knowlesge Center**: Contains the most frequent & common questions and requests
9. **AWS Managed Services:** AMS offers a team of AWS experts who manage and operate your infrastructure for security, reliability, and availability. Fully managed service, so AWS handles common activities such as change requests, monitoring, patch management, security, and backup service

# Well Architected Framework 6 Pillars
1. Operational Excellence  -> processi e procedure
	- **Anticipate failure, Learn from all operational failures**
2.  Security -> sicurezza
	- **Enable traceability, Shared Responsibility Model**
3.  Reliability -> recovery, scale
	- **Stop guessing capacity, Scale horizontally**
4.  Performance Efficiency -> computing resources efficiently
	- **Democratize advanced technologies, Go global in minutes, Mechanical sympathy**
5.  Cost Optimization -> migliorare e ottimizzare le spese
	- **Measure overall efficiency
6.  Sustainability -> massimizzare l'utilizzo e ridurre l'impatto
	- **Understand your impact, Use managed services, Maximize utilization**

# Shared Responsability Model

![[Pasted image 20240828170225.png]]
Security **of** the cloud: AWS responsibility - Security of the Cloud 
• Protecting infrastructure (hardware, software, facilities, and networking) that runs all the AWS services 
• Managed services like S3, DynamoDB, RDS, etc.

Security **in** the cloud: For EC2 instance, customer is responsible for management of the guest OS (including security patches and updates), firewall & network configuration, IAM 
• Encrypting application data

**Shared controls**: • Patch Management, Configuration Management, Awareness & Training
### Shared Responsibility Model for IAM
![[Pasted image 20240828170719.png]]
### Shared Responsibility Model for EC2
![[Pasted image 20240828170802.png]]
### Shared Responsibility for EC2 Storage
![[Pasted image 20240828170828.png]]
### Shared Responsibility Model for S3
![[Pasted image 20240828171049.png]]

![[Pasted image 20240904093711.png]]
# Advantages of Cloud

- **Trade fixed expense(capital expenses) for variable expense(operational expenses)** – Instead of having to invest heavily in data centers and servers before you know how you’re going to use them, you can pay only when you consume computing resources, and pay only for how much you consume.
    
- **Benefit from massive economies of scale** – By using cloud computing, you can achieve a lower variable cost than you can get on your own. Because usage from hundreds of thousands of customers is aggregated in the cloud, providers such as AWS can achieve higher economies of scale, which translates into lower pay as-you-go prices.
    
- **Stop guessing capacity** – Eliminate guessing on your infrastructure capacity needs. When you make a capacity decision prior to deploying an application, you often end up either sitting on expensive idle resources or dealing with limited capacity. With cloud computing, these problems go away. You can access as much or as little capacity as you need, and scale up and down as required with only a few minutes’ notice.
    
- **Increase speed and agility** – In a cloud computing environment, new IT resources are only a click away, which means that you reduce the time to make those resources available to your developers from weeks to just minutes. This results in a dramatic increase in agility for the organization, since the cost and time it takes to experiment and develop is significantly lower.
    
- **Stop spending money running and maintaining data centers** – Focus on projects that differentiate your business, not the infrastructure. Cloud computing lets you focus on your own customers, rather than on the heavy lifting of racking, stacking, and powering servers.
    
- **Go global in minutes** – Easily deploy your application in multiple regions around the world with just a few clicks. This means you can provide lower latency and a better experience for your customers at minimal cost.

![[Pasted image 20240828173548.png]]


# IAM Guidelines & Best Practices

**To access AWS, you have three options:** 
• AWS Management Console (protected by password + MFA) • AWS Command Line Interface (CLI): protected by access keys • AWS Software Developer Kit (SDK) - for code: protected by access keys

• Don’t use the root account except for AWS account setup
• One physical user = One AWS user
• Assign users to groups and assign permissions to groups
• Create a strong password policy
• Use and enforce the use of Multi Factor Authentication (MFA)
• Create and use Roles for giving permissions to AWS services
• Use Access Keys for Programmatic Access (CLI / SDK)
• Audit permissions of your account using IAM Credentials Report & IAM 
Access Advisor
• Never share IAM users & Access Keys

# Pricing Models in AWS

Pricing Fundamentals.
- **Compute**:
	 - Pay for compute time
- **Storage**:
	 - Pay for data stored in the Cloud
- **Data transfer OUT of the Cloud**:
	 - Data transfer IN is free

## Pricing model

• **Pay as you go**: pay for what you use, remain agile, responsive, meet scale 
demands
• **Save when you reserve**: Reservations are available for: 
	- **EC2 Reserved Instances**
	- **DynamoDB Reserved Capacity**
	- **ElastiCache Reserved Nodes**
	- **RDS Reserved Instance**
	- **Redshift Reserved Nodes**
• **Pay less by using more**: volume-based discounts
• **Pay less as AWS grows**

# AWS CAF
**AWS Cloud Adoption Framework (AWS CAF):** Helps you build and then execute a comprehensive plan for your digital transformation through innovative use of AWS

## Perspective
1. **Business:** The Business perspective helps ensure that your cloud investments accelerate your digital transformation ambitions and business outcomes. Common stakeholders include chief executive officer (CEO), chief financial officer (CFO), chief operations officer (COO), chief information officer (CIO), and chief technology officer (CTO).
2. **People:** The People perspective serves as a bridge between technology and business, accelerating the cloud journey to help organizations more rapidly evolve to a culture of continuous growth, learning, and where change becomes business-as-normal, with focus on culture, organizational structure, leadership, and workforce. Common stakeholders include CIO, COO, CTO, cloud director, and cross-functional and enterprise-wide leaders.
3. **Governance:** The Governance perspective helps you orchestrate your cloud initiatives while **maximizing organizational benefits** and minimizing transformation-related risks. Common stakeholders include chief transformation officer, **CIO, CTO, CFO, chief data officer (CDO)**, and chief risk officer (CRO).
4. **Platform:** The Platform perspective helps you build an enterprise-grade, scalable, hybrid cloud platform, modernize existing workloads, and implement new cloud-native solutions. Common stakeholders include CTO, technology leaders, architects, and engineers.
5. **Security:** The Security perspective helps you achieve the confidentiality, integrity, and availability of your data and cloud workloads. Common stakeholders include chief information security officer (CISO), chief compliance officer (CCO), internal audit leaders, and security architects and engineers.
6. **Operations:** The Operations perspective helps ensure that your cloud services are delivered at a level that meets the needs of your business. Common stakeholders include infrastructure and operations leaders, site reliability engineers, and information technology service managers.
## Domains
- **Technology:** using the cloud to migrate and modernize legacy infrastructure, applications, data and analytics platforms
- **Process:** digitizing, automating, and optimizing your business operations 
	- leveraging new data and analytics platforms to create actionable insights 
	- using machine learning (ML) to improve your customer service experience
- **Organization:** Reimagining your operating model 
	- Organizing your teams around products and value streams 
	- Leveraging agile methods to rapidly iterate and evolve
- **Product:** reimagining your business model by creating new value propositions (products & services) and revenue models
## Phases
- **Envision** focuses on demonstrating how cloud will help accelerate your business outcomes. It does so by identifying and prioritizing transformation opportunities across each of the four transformation domains in line with your strategic business objectives. Associating your transformation initiatives with key stakeholders (senior individuals capable of influencing and driving change) and measurable business outcomes will help you demonstrate value as you progress through your transformation journey.
    
- **Align** focuses on identifyifng capability gaps across the six AWS CAF perspectives, identifying cross-organizational dependencies, and surfacing stakeholder concerns and challenges. Doing so will help you create strategies for improving your cloud readiness, ensure stakeholder alignment, and facilitate relevant organizational change management activities.
    
- **Launch** focuses on delivering pilot initiatives in production and on demonstrating incremental business value. Pilots should be highly impactful and if/when successful they will help influence future direction. Learning from pilots will help you adjust your approach before scaling to full production.
    
- **Scale** focuses on expanding production pilots and business value to desired scale and ensuring that the business benefits associated with your cloud investments are realized and sustained.

# Cloud migration strategies

Ordinate in ordine di difficoltà di implementazione

- **Retire**: • Turn off things you don’t need (maybe as a result of Re-architecting), Helps with reducing the surface areas for attacks
- **Retain**: This is the migration strategy for applications that you want to keep in your source environment or applications that you are not ready to migrate. **Security, data compliance, performance, unresolved dependencies**
- **Relocate** "lift" and "shift"**: Move apps from on-premises to its Cloud version, Example: transfer servers from VMware Software-defined Data Center (SSDC) to VMware Cloud on AWS
- **Rehost "lift" and "shift"**: Simple migrations by re-hosting on AWS (applications, databases, data…). Migrate using **AWS Application Migration Service**. Minimum changes.
- **Repurchase "_drop and shop_"**: replace your application with a different version or product. The new application should provide more business value than the existing, on-premises application
-  **Replatform “lift and reshape”**: Moving your application to the cloud and make adjustment.  Example: migrate your database to RDS • Example: migrate your application to Elastic Beanstalk, 
- **Refactor or re-architect**: Using this strategy, you move an application to the cloud and modify its architecture by taking full advantage of cloud-native features to improve agility, performance, and scalability.







```
git config user.name "Claudio Giaretta"
git config user.email claudio.giaretta@vaisoftware.it
```