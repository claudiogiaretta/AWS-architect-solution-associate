esami di prova: https://www.exampro.co/saa-c03

AWS_CLI_AUTO_PROMPT: aiuta a fare comandi usando il cli in aws

Informazioni utili:
- Non concentrarsi molto su kubernetis 
- La parte global infrastructure molto simile a quella del cloud practitioner




# Storage and Migration

## S3
**S3**: (**not global and SERVERLESS**) Amazon Simple Storage Service (Amazon S3) is an object storage service offering industry-leading scalability, data availability, security, and performance. Buckets are defined at the **region level**. Object are encrypted at server-side level.

- Folder in S3 are object, no metadata e no permissions, don't contain anything
- ETAG represent an hash of the object, is not part of metdata use MD5, are used to check if it was a change in the object
- Checksum is used to check the integrity of a file, determine if this somehting wrong with the file
- Prefix: are part of the name of the object, are used to group and filter togheter objects
- Metadata; provides information about data but not the contents. 
- There are 2 types,
	- System defined. 
	- User defined: 
		- Access and Security
		- Media File
		- Custom Application
		- Project Specific
		- Document versioning
		- Content related
		- Compliance and Legal
- WORM (Write once and Read Many):  
- S3 object lock: to prevent that object are deleted, you can only anable it by AWS CLI
- S3 Bucket URI: Is a way to reference address of s3 bucket and object
- AWS S3 CLI: 
	- aws s3: high level
	- aws s3api: low level
	- aws s3control: Managing s3 access points, s3 outpost ecc..
	- aws s3outposts: Specific for outpost management
- S3 Request Styles: 
	- VIrutal hosted
	- Path style
- S3 Glacier Flexible Retrival (you should save big file in zip insted of larger number of small files):
	- Expedited Tier
	- Standard Tier
	- Buck Tier
- s3 Glacier deep archive:
	- standard tier
	- Bulk tier: for petabytes of data
- s3 intelligent-tier: Ha accesso a tutti i servizi tranne, Archive e Deep archive che sono opzionali
- Security:
	- Bucket Policies: Define permissions for an entire S3 bucket using JSON-based access policy language.
	- Access Control Lists (ACLs): Provide a legacy method to manage access permissions on individual objects and buckets.
	- AWS PrivateLink for Amazon S3: Enables private network access to S3, bypassing the public internet for enhanced security.
	- Cross-Origin Resource Sharing (CORS): Allows restricted resources on a web page from another domain to be requested.
	- Amazon S3 Block Public Access: Offers settings to easily block public access to all your S3 resources.
	- IAM Access Analyzer for S3: Analyzes resource policies to help identify and mitigate potential access risks.
	- Internetwork Traffic Privacy: Ensures data privacy by encrypting data moving between AWS services and the Internet.
	- Object Ownership: Manages data ownership between AWS accounts when objects are uploaded to S3 buckets.
	- Access Points: Simplifies managing data access at scale for shared datasets in S3.
	- Access Grants: Providing access to S3 data via a directory services e.g. Active Directory
	- Versioning: Preserves, retrieves, and restores every version of every object stored in an S3 bucket.
	- MFA Delete: Adds an additional layer of security by requiring MFA for the deletion of S3 objects.
	- Object Tags: Provides a way to categorize storage by assigning key-value pairs to S3 objects.
	- In-Transit Encryption: Protects data by encrypting it as it travels to and from S3 over the internet.
	- Server-Side Encryption: Automatically encrypts data when writing it to S3 and decrypts it when downloading.
	- Client-Side Encryption: Encrypts data client-side before uploading to S3 and decrypts it after downloading.
	- Compliance Validation for Amazon S3: Ensures S3 services meet compliance requirements like HIPAA, GDPR, etc.
	- Infrastructure Security: Protects the underlying infrastructure of the S3 service, ensuring data integrity and availability.
- ACL It ha been used to allow other aws account to upload objects to bucket
- Bucket Polices: resource based policy that grant access to s3 object from other sevices or accounts
- IAM vs Bucket Policies
	![[Pasted image 20240930140617.png]]
- Amazon Access Grants: map identities in a direcory service
- IAM Access Analyzer: will alert you when s3 buckets are exposed to the internet or other AWS account
- Internetwork traffic privacy: is a concept not a service, i about keeping data private as it travel across different network
	Servizi legati:
	- AWS PrivateLink: Allows you to connect an Elastic Network Interface (ENI) directly to other AWS Services 
	- VPC Gateway Endpoint: Same as private link but specific to s3 and less fine grain permission
- CORS: is an HTTP-header based mechanism that allows a server to indicate any other origins than its own from  which a browser should permit
	- Request Headers
		- Origin
		- Access-Control-Request-Method
		- Access-Control-Request-Headers
	- Response Headers
		- Access-Control-Allow-Origin
		- Access-Control-Allow-Credentials
		- Access-Control-Expose-Headers
		- Access-Control-Max-Age
		- Access-Control-Allow-Methods
		- Access-Control-Allow-Headers
	Amazon allow you to set corse using JSON or XML files
- Encryption in transit: Data is secured when moving between locations (uses TLS and SSL)
- Server side Encryption, is always active for s3 objects
	- SSE-S3: Amazon S3 manage the keys, encrypts using AES-GCM (256-bit) algoritm
	- SSE-KMS: is when  you use a KMS key managed by AWS
		![[Pasted image 20240930144537.png]]
	
	- SSE-C: 
		![[Pasted image 20240930144906.png]]
	- DDSE-KMS: Dual layer, both client and server side encryption
	 ![[Pasted image 20240930145407.png]]
- S3 Bucket key, let's you generate a KEY every time a request to s3 is made
- Client-Side Encryption: is when you encrypt you rfile before uploading
- Data Consistency: 
	![[Pasted image 20240930150935.png]]
- Object Replication: 
- Versioning: recover easily from accident deletion or overwrite, once enabled it it can't be disabled
- Transfer acceleration: service that are used to transfer s3 files over long distances
- presigned URL: temporary access to upload or download object frmo s3
	- example:
		![[Pasted image 20240930151923.png]]
- Access Points:  Simplify bucket policy, are named network endpoint attached to the bucket that you can use to perform S3 object
- Multi region Endpoint: global endpoint to route request to the buckets with the lowest latency.
- Object Lambda Access Point, allows you to transform the output request of s3 object
- Mountpoint for amazon s3: allows you to mount s3 buckets to linux local file system
	![[Pasted image 20240930154353.png]]
- Archived object: Rarely access object that cannot be access in real-time in exchange of reduced cost.
- Requester Pay: it is a bucket option that allow the bucket owner to offset specific s3 cost to the requester
	- Request Pay Header:
- AWS Marketplace S3: alternatives to AWS services that works with s3
- Aws Batch operations: 
		![[Pasted image 20240930155842.png]]
- Amazon S3 Inventory: give reports of all s3 buckets changes history. 
- Event notification: allows your buckets to notify other AWS services about s3 event data.
	![[Pasted image 20240930160204.png]]
- Storage Class Analysis allows you to analyze storag access patterns of objects within a bucket to reccomend objects to move between standard to standard_IA. Observe infrequent access pattern
- Amazon S3 Storage Lens is a storage analysis tool for s3 buckets acrross your entire AWS organization. Display data in interactive dashboard
- S3 Static Website Hosting , allow you to host and serve a static (no server side interactivity) website from an s3 bucket,  no HTTPS unless you use AWS cloudfront alongside.
- Amazon s3 Multipart Upload: support multipart upload so you can upload a single object in a set of parts. It breaks your file in multiple part and upload each part sepratly.
- Byte range fetching: Amazon s3 allow to fetch a range of bytes of data from s3 objecs using the range header during s3 getObject API request
- Interoperability: the capability of the cloud services to exchange and utilize information seamlessly with each other.


## EBS

![[Pasted image 20241001174016.png]]
- EBS: (vedi appunti AWS)
	![[Pasted image 20241001174105.png]]
	N.B io2 doesn't exist anymore use io2 Block Express
- Volume type usages:
	![[Pasted image 20241001174432.png]]
	![[Pasted image 20241001174656.png]]

## EFS
(vedi appunti AWS per la definizione)
![[Pasted image 20241001175602.png]]
- EFS Client (amazon-efs-utils package): is an open-source collection of Amazon EFS tools.
	- enables the ability to use Amazon CloudWatch to monitor an EFS file system's mount status
	- You need to install the Amazon EFS client on an Amazon EC2 instance prior to mounting an EFS file system
	- ![[Pasted image 20241001180345.png]]

## Fsx
Allows you to deploy scale feature-rich, high performance file systems in the cloud. Fsx support a variety of file system protocol.

- types:
	- Amazon FSX for NetApp ONTAP
	- Amazon Fsx for OpenZFS
	- Amazon FSX for Windows File Server (WFS)
	- Amazon FSX for Lustre

- Amazon FSX for Windows File Server (WFS)
- File Cache: high speed cache for datasets stored anywhere, accelerate bursting workloads

## AWS BACKUP
Allows you to centrally manage backup accross AWS Services

- Component:
	- Backup Plan: A backup policy defines the backup schedule, backup window, backup lifecycle.
	- Backup Vault: backup are stored in a backup vault
	- AWS backup Audit Manager is built in reporting and auditing for AWS backup

## Snow Family
AWS Snow Family are storage and compute devices used to phisically move data in or out the cloud when moving data over the internet or private connection is to slow, difficult or costly.

![[Pasted image 20241002093152.png]]
- Snowcone: Portable secure devicce for edge computing
	- types:
		- Snowcone: 8 TB HDD
		- Snowcone SSD: 8 TB SSD
	- Two ways of sending data:
		- Phisically shipping the device which runs on the device's compute
		- AWS DataSync which runs on the device's compute
	- Has an E link shipping label for easy shipping
- SnowballEdge: Similar to Snowcone but with more local processing, edge computing worloads, and device configuration options.
	- Config Options:
		- Storage Optimized (for data transfer): 
			- 100 TB (80 TB usable)
		- Storage Optimized
			- 210 TB usable
		- Storage Optimized with EC2-Compatible compute
			- 80 TB usable storage , 40vCPUs and 80 GB of memory
		- Compute Optimized
			- Up to 104 vCPUs, 416 GB of memory,and 28 TB of dedicated NVMe SSd
		- Compute Optimized with GPU
			- Has addition of GPUs equivalent to the one available in the Instance Type P3
- AWS Snowmobile: 45-foot ship container truck. Used to handle 100 PB of data
- Comparison
	![[Pasted image 20241002094449.png]]

## AWS transfer family
Offers fully managed support for the tansfer files over SFTP, AS2,FTPS and FTP directly into and out of Amazon S3.
![[Pasted image 20241002094829.png]]
![[Pasted image 20241002094906.png]]

## AWS Migration Hub
Is a single place to discover your existing servers, plan migrations, and track the status of each application migration.

AWS Migration hub can monitor migration using:
- Application Migration Service (AMS)
- Database Migration Service (DMS)

Tools to migrate:
- AWS Discovery Agent: Agent installed on your VM
- Migration Evaluator Collector : You submit a request to ask helps from AWS team
Other tools:
- AWS Migration Hub Refactor: Bridges networking accross AWS Accounts so that legacy and new services can communicate while they maintain the independence of separate accounts
- AWS Migration Hub Journey: guided templates
## AWS Data Sync
Is a data transfer service that simplifies data migration to, from, and between cloud storage services
## Data Migration Service (DMS)
AWS Database Migration Service (AWS DMS) is a managed migration and replication service that helps move your database and analytics workloads to AWS quickly, securely, and with minimal downtime and zero data loss.
![[Pasted image 20241002101336.png]]
**AWS Schema Conversion** is udes in many cases to automatically convert a source database schema to a target database schema
For data warehouse you can use the desktop app **AWS schema conversion tool** 

- Migration Methods:
	![[Pasted image 20241002101956.png]]


## Auto Scaling (different from ASG)
AWS auto scaling is a service that can discover scaling resources whithin your aws account, and quickly add scaling plans to your scaling resources

![[Pasted image 20241002102340.png]]

## Storage Gateway
- **AWS Storage Gateway**: is a hybrid storage service that allows your on-premises applications to seamlessly use AWS cloud storage (Bridge between on-premise data and cloud data in s3). Use example: backups to the cloud, using on-premises file shares backed by cloud storage, and providing low-latency access to data in AWS for on-premises applications.
![[Pasted image 20241004101100.png]]
- **Amazon S3 File Gateway**: allows your files to be stored as objects inside your s3 buckets. Access your files through a Network File System (NFS) or SMB mount point.
- **Amazon FsX File Gateway**: allows your files to be stored in Amazon FSx Windows FIle Storage (WFS). Allow your Windoes developers to easily store data in the cloud using the tools they already know
- **Volume Gateway**: presents your applications with disk volumes using the Internet Small Computer Systems Interface(iSCI) block protocol.
	- Compress storage to save space
	- Can be asynchronously backed up as point-in-time snapshots of the volume.
	- types
		- **Store Volumes:** store primary data locally and asynchronously that data to AWS:
			- Provide your on-premises applications with low-latency access to their entire datasets while still providing durable off-site backups.
			- Create storage volumes and mount them as iSCSI devices from your on-premises servers.
			- Any data written to stored volumes are stored on your on-premises storage hardware.
			- Amazon Elastic Block Store (EBS) snapshots are backed up to AWS S3.
			- Stored Volumes can be between 1GB - 16TB in size
		- **Cache volumes:** 
			- Minimizes the need to scale your on-premises storage infrastructure while still providing your applications with low-latency data access.
			- Create storage volumes up to 32TB in size and attach them as iSCSI devices from your on-premises servers.
			- Your gateway stores data that you write to these volumes in S3 and retains recently read data in your on-premises storage gateway cache, and upload buffer storage.
			- Cached volumes can be between 1GB - 32GB in size
- **Tape Gateway**: **durable**, cost effective solution to archive your data in thw AWS CLoud. Use VTL interface, that helps you leverage existing tape-based backups.Store data on virtual tape that you create on you tape gateway. Tape storage has proven redability for 30 years
	- VTL media changer is a a robot that moves tapes around:
	![[Pasted image 20241004104332.png]]



## Instance Store
- **Hardware storage directly attached to EC2 instance** (cannot be detached and attached to another instance)
- **Highest IOPS** of any available storage (millions of IOPS)
- **Ephemeral storage** (loses data when the instance is stopped, **hibernated** or terminated)
- Good for buffer / cache / scratch data / temporary content
- AMI created from an instance does not have its instance store volume preserved

> You can specify the instance store volumes only when you launch an instance. You can’t attach instance store volumes to an instance after you’ve launched it.

### RAID

- **RAID 0**
    - Improve performance of a storage volume by distributing reads & writes in a stripe across attached volumes
    - If you add a storage volume, you get the straight addition of throughput and IOPS
    - For high performance applications
- **RAID 1**
    - Improve data availability by mirroring data in multiple volumes
    - For critical applications
## Amazon AppFlow
Is managed integration service for data transfer between data sources. Easily exchange data with over 80+ cloud services. By specifing a source destination. (Easy way to connect services)

![[Pasted image 20241002103209.png]]

# Computing
## EC2
- Cloud-Init: is the industry standard multi-distribution method for cross-platform cloud instance initialization. It is supported accross all major public cloud providers, provisioning systems for private cloud infrastructure, and bare metal installations.
- EC2 User data: script launch on EC2 boot
- EC2 Meta Data: You can access to Ec2 Metadata from MDS(Meta data service). There are 2 typers Version 1 adn Verison 2.
	- You can enforce the use of tokens 
	- you can turn off endpoints all together
- EC2 Naming convention:
	![[Pasted image 20241001121648.png]]
- Instance Family: Are different combination of CPU,Memory,Storage and Networking capacity
	![[Pasted image 20241001122001.png]]
	N.B there are not all but only the most important
- EC2 Processors: 
	 ![[Pasted image 20241001122628.png]]
- EC2 Instance Profile: is a reference to an IAM role that will be passed and assumed by the EC2 instance when it starts up. allows you to avoid passing long live AWS credentials.
- EC2 Lifecycle: 
	![[Pasted image 20241001123523.png]]
- EC2 Instance Console Screenshot: will take a screenshot of the current state of the instance
- Hostname; unique name in your network to identify a machine via DNS.
	- two types:
		- IP Name: legacy name based on private access
		- Resource Name: EC2 instance ID is included in the hostname
- EC2 Default user name: The default user name for an operating system managed by AWS will vary based on distribution:
	- When using SSM you will need to change yoour user to default
	- ```sudo su -ec2-user```
-  EC2 Bustable Instances: allow workloads to handle bursts of higher CPU utilization fro very short duration
- EC2 system log: Accessed by the EC2 Management Console, allows you to observe the system log for the EC2 Instance. Troubleshoot on boot to see if anything is wrong
- EC2 Placement Group: the logical placement of your instances to optimize communication, performance, or durability. Free
	- Cluster
	- Partition
	- Availability Zone
- EC2 Connect:
	- SSh Client
	- EC2 instance connect: short lived ssh keys controlled by IAM policies
	- Session Manager: Connect to linux windoes via reverse connection
	- Fleet Manager Remote Desktop: Connect to a Windows Machine using RDP
	- EC2 Serial console: establishes a serial connection giving you direc access
- EC2 Amazon Linux: AWS's managed Linux distribution is based off CentOS and Fedora which in turn is based off Red Hat Linux. Reccomended to use Amazon Linux AL2023 e non AL2
	- Amazon Linux extra: is a feature of AL2 that provides a way for users to install additional software packages.
## AMI
- Provides the information required to launch an instance. You can turn your EC2 instances into AMI's so you can create copies of your servers.
	- Help you keep incremental changes to your OS, application code, and system packages.
	- AMI Id is different per region. You need to be carefull when you buy one
	- AMI has two types of boot model:
		- Unified Extensible Firmware Interface (UEFI): Modern
		- Legacy BIOS: Legacy da evitare
	- Elastic Network Adapter supports network speeds of up to 100 GBps for supported instance types.
	- AMI Settings:
		- Public:
		- Explicit: Specific to certain AWS accounts
		- Implicit: The owner can launch the AMI
## ASG
Check "AWS appunti" first
- Schema:
	![[Pasted image 20241001144825.png]]
- Automatic scaling can occur via:
	- Capacity settings
	- Health Check Replacements
	- Scaling Policy: (vedi appunti AWS)
- Capacity Settings: The size of an Auto Scaling group based on Min Size, Max Size, Desired Capacity
- Health Check Replacement: when an ASG replaces an instances if is considered unhealty. Two types of health checks ASG can perform
	- EC2 Health Check: If the EC2 instance fails either of its EC2 Status Check
	- ELB Health Check: ASG will perform a health check based on the ELB health check. ELB pings an HTTP endpoint at a specific path, port and status code
- ELB integration: An ELB can be attached to you Auto Scaling Group 
	- Classic Load Balancers are associeted directly to the ASG
	- ALB, NLB or GWLB are associeted indirectly via their Target Group
	 **N.B attached = can use Load Balancer Health check**
- Dynamic Scaling Policies: how much ASG should change capacity. Policies are triggered based on Cloud Watch Alarms:
	- Simple, Step and Target Scaling
	- Based on adjustment types: how capacity should change
		- ChangeInCapacity
		- ExactCapacity
		- PercentChangeInCapacity
- Simple Scaling: (vedi appunti AWS)
- Step Scaling: (vedi appunti AWS)
- Target Scaling: (vedi appunti AWS)
- Predictive Scaling Policies: (vedi appunti AWS)
## ELB
It is a suite of Load balancer from AWS

- Rules of traffic
	- Struttura:
		- Listeners: incoming traffic is evaluetad against listeners. check matches with the Port (ex port 443 or port 80)
		- Rules (only ALB): Listeners will then invoke rules to decide what to do with the traffic. Generally, the next step is to forward traffic to a Target Group
		- Target Groups(not CLB): Are logical grouping of possible targets such as specific EC2 instances, IP addresses
		- **only for CLB traffic** is sent to the Listeners. When the port matches, it forwards the traffic to any EC2 instances that are registered to che Classic Load Balancer. CLB does not allow you to apply rules to listeners.
- Application Load Balancer:
	![[Pasted image 20241001154721.png]]
- Network Load Balancer:
	![[Pasted image 20241001154905.png]]
	-
- Classic Load Balancer:
	![[Pasted image 20241001155036.png]]
## AWS Lambda
No charge when your code is not running.

Example of use:
![[Pasted image 20241002122149.png]]
2 ways to be called:
- Sync invocation
- Async invocation

Settigns an limits:
- ![[Pasted image 20241002122429.png]]
- Function Version: You can use the version to manage the deployment of your AWS Lambda Function
- 2 ways to reference aLambda
	- Qualified ARN (specified version)
	- Unqualified ARN (non specified version take always the last)
- Aliasis: provide name to reference Lambda function
- Lambda Layer: Pull in additional code and content in the form of layer. Layer is a zip archive that **contains libraries, a custom runtime, or other dependencies**. You can use libraries in your function without needing to include them in your deployment package. Maximum 5, maximum package: 250 MB
- Instruction Sets: 
	![[Pasted image 20241002123047.png]]
- Runtimes: preconfigure environment to run specific programming languages. It doesn' require you to configure or set OS.
	- they are published continuesly, so you had to keep it updated and not retain older version
	- Code is delivered as a zip archive
- OS Only Runtime:
	![[Pasted image 20241002123430.png]]
- Deployment Packages: package which contains the actual code.
	- 2 types:
		- ZIP archive:
			- ZIP archive:larger than 50 MB have to be uploaded to an S3 Bucket first
			- Rely on Lambda runtimes
		- Container Image: you need to create a docker file
		- build and push image to ECR
- 
## Step functions
![[Pasted image 20241002124321.png]]

- 2 types State Machines:
	- Standard: general purpose -> reccomended for Long Workload
	- Express: for sreaming data -> reccomended Short Workload, (streaming, data ingection, even driven ecc...)
- Some use cases:
	- Manage a Batch job , if job fails send message with SNS
	- Manage a Fargate Container
	- Transfer Data Records
	- **Per altri use cases andare a cercare step function use cases nella guida ufficiale di amazon**
- Step Functions states: Allows us to pass nput and output without any work. Useful when constructing state machines
	- Structure of state
		- Parameters
		- Result
		- ResulthPath
	- Task State
		- ...
		- Activites: Enables you to have a task in your state machine where the work is performed by a worker that can be hosted on anywhere eg: EC2, ECS, mobile phones
	- Wait State: delays state machine from coninuing for a specificng time
	- Succed State: stop an execution succesfully
	- Fail State: stops eh executiion of a state machine and set failure
	- Parallel States: can be used to create parallel branches of execution in your state machine The state machine does not move forward until both states complete
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
## AWS Batch
 AWS Batch helps you to run batch computing workloads on the AWS Cloud. Batch computing is a common way for developers, scientists, and engineers to access large amounts of compute resources. Rispetto alle Lambda: **No time limit** • • Batch jobs are defined as Docker images and run on ECS • **Rely on EBS and EC2 / instance store for disk space**.


# Network & Security
## AWS VPC
- AWS VPC: Logically isolated network, Region specific. 5 VPC per region. 200 subnet per VPC
- AWS has default VPC per region 
- Shared VPC :  you can share a VPC trough the sam eaccount using AWS Resources Acess Manager
- NACL: (vedi l'altro foglio di appunti). Rispetto a security group puoi bloccare anche un singolo ip.
- Security groups: (vedi l'altro foglio di appunti). Limiti: 10000 security groups per region, 60 in/outbound rules per secGroup, 16 secGroups per Elastic Network Interface
- Stateless vs Stateful: Stateles Firewalls like AWS NACL are not aware of the state of the request. block all the request both way.
- Route tables: where network traffic are directed.
	- different types:
		- main route tables: created with VPC cannot be deleted
		- custom route table: toute table that you can create
- Elastic Network Interfaces (ENIs): are virtual network interfaces that provide networking capabilities to Amazon EC2 instances. An ENI functions as a virtual network card, enabling instances to connect to networks and communicate with other resources within the AWS environment.
-  Gateway: network service that stands between two different network, es: (load balancer, reverse proxy, firewall ecc...).
	![[Pasted image 20240930180536.png]]
- Internet Gateway:
	- Work both with IPV4 and IPV6, and perform network translation for instances that have been assigned public IPV4 addresses.
	![[Pasted image 20241001091723.png]]
- Egress-only (EO-IGW): specifically for IPV6 when you want to allow outbound traffic to the internet but prevent inbound from the internet:
	![[Pasted image 20241001092144.png]]
- Elastic IP: Addresses in AWS that always stay the same (different from that of the EC2 instances)
	- IPV6 are already unique so they don't need it
	- are charge 1$ for each allocated
	- You can associate, allocate, disassociate, deallocate, reassociate, recover, customize (with your IPV6 ip)
- AWS ipv6 support: provide a solution for the eventual exhaustion of all IPV4 address
- Migrating from IPV4 to IPV6
	![[Pasted image 20241001093351.png]]
- AWS Direct Connect: 
	- ![[Pasted image 20241001093756.png]]
	- 2 types
		- Lower Bandwith 50Mbps -500Mbps
		- Higher Bandwith 1Gbps, 10Gbps, 100Gbps
	- Your network is co-located with an existing AWS Direct Connect location
	- You work with a AWS Partner Network member
	- Support IPV4 and IPv6 
	- Pricing
		- Capacity
		- Port hours:
			- Dedicated: Billed per hour
			- Hosted Billed subject to the AWS direct connect Delivery partner
		- DTO (Data Transfer Out)
			- Charged based on outbound traffic sent through Direct Connect to destinations outside of AWS
			- Data transfer in the same region is free
- VPC Endpoints: (vedi appunti AWS):
	- 3 types: Interface Endpoint, Gateway Endpoint, Gateway Load Balancer Endpoint
	- Do not require public IPV4 and doesn't leave the AWS network
	- Eliminates the need for Internet Gateway, NAT Device, VPN connection, AWS Direct Connect
	![[Pasted image 20241001095236.png]]
- PrivateLink: (vedi appunti AWS):
	- ![[Pasted image 20241001095444.png]]
	- Need an Interface endpoint and Service endpoint in order to work
- Interface endpoint: it is a Elastic Network Interfaces (ENI) with a private IP addess. they serve as an entry point for traffic going to a supported service.
- Gateway Load Balancer: Powered by PrivateLink allows you to distribute traffic  to a fleet ofnetwork virtual appliances
	- ![[Pasted image 20241001100306.png]]
- Gateway Endpoint
	- ![[Pasted image 20241001100941.png]]
	- Use only by s3 and DynamoDB
- VPC Endpoints Comparison
	 ![[Pasted image 20241001101531.png]]
- VPC flow logs: (vedi appunti AWS)
- AWS VPN: 
	- AWS site-to-site: (vedi appunti AWS)
	- AWS Client VPN: (vedi appunti AWS)
	- IPSec: Internet Protocol Security (IPSec) is a secure network protocol suite that authenticates and encryptes the packets of data between two computers over Internet Protocol Network.
- AWS Site-to-site VPN:
	- ![[Pasted image 20241001102606.png]]
	- ![[Pasted image 20241001102856.png]]
	- ![[Pasted image 20241001103123.png]]
- Virtual Private Gateway: VPN endpoint on the Amazon side of your Site-to-site VPN connection that can be attached to a single VPC
	- You need to assign an Amazon ASN or custom ASN (a unique identifier that is globally allocated to each autonomous system)
- Customer Gateway: 
	- ![[Pasted image 20241001103628.png]]
- Transit Gateway: (vedi appunti AWS)
	- ![[Pasted image 20241001103854.png]]
- AWS Client VPN: (vedi appunti AWS) (ecample of use: you travel around the world and need connection to services)
	- ![[Pasted image 20241001104054.png]]
	- ![[Pasted image 20241001104307.png]]
- NAT (Network Address Translation):
	- ![[Pasted image 20241001104604.png]]
- NAT Gateway: (Vedi appunti AWS)
	- ![[Pasted image 20241001104657.png]]
	- 1 Nat gateway per subnet
	- Pricing
		- Per hour 0.045
		- Per GB 0.045
	- 2 types: Public and Private
- NAT Instance (legacy not used anymore): is an AWS managed IAM to launch a NAT onto an individual EC2 instances. NAT Instances required the customer to handle scaling
- Jumpbox/Bastion: security hardenend virtual machines that provide secure access to private subnets
	- ![[Pasted image 20241001105435.png]]
	- Nat should not be used as Bastion
- VPC Lattice: is a fully managed application networking service that you use to connect, secure, and monitor the services for your application -> Turn your AWS resources into services for a micro services architecture
	- ![[Pasted image 20241001110138.png]]
- Transit Gateway: (vedi appunti AWS) 
	- ![[Pasted image 20241001110353.png]]
	- ![[Pasted image 20241001110427.png]]
- Traffic Mirroring: sends a copy network traffic from a source ENI to target ENI, or UDP-enabled NLB or GWLB
- Route 53 Resolver DNS Firewall: Firewall that protect against DNS exfiltration of your data
	![[Pasted image 20241001111708.png]] ![[Pasted image 20241001111926.png]]
- AWS Network firewall: is a stateful, managed, network firewall and IDS/IPS for VPCs (use open source Suricata )
- VPC Peering: (vedi appunti aws): behave like they are on the same network
	- use star configuration
	- no-transitive
- Network Address Usage(NAU): is a metric applied in your VPC to help you plan for and monitor the size of your VPC, in a way that you don't run out of space for your VPC.
## Route 53
- Hosted Zones: Container for records sets, scoped to route traffic for a specific domain or subdomains
	- Public: How you want to route traffic through an Internet
	- Private: How you want to route traffic through an Amazon VPC
- Records set: collection of records which determine where to send traffic
	- Always changed using batch via the API
	- Record Alias: sppecific DNS record of AWS which extends DNS functionality. It rwill route traffic to specific AWS resources.
	- ![[Pasted image 20241001160633.png]]
- Traffic Flow: Visual tool, lets you create soshisticated routing configurations for your resources using routing types (50$ per policy -> very expensive)
- Routing Policies:
	![[Pasted image 20241001161413.png]]
	- Simple Routing Policies: Most basic Routing policies
		- you have 1 record and provide multiple IP addresses
		- When multiple values are specified for a record, Route53 will return values back to the user in a random order
		- ![[Pasted image 20241001163913.png]]
	- Weighted Routing Polcies: Let you split up traffic based on different weight assigned
		- This allows you to send a certain percentage of overall traffic to one server and have other traffic to be directed to a completeley different server
	- Latency Based Routing: allows you to direct traffic based on the lowest network latency possible for your end-user based on region
	- Failover Routing Policy: allow you to create active passive/setup in situations where you want a primary site in one location and a secondary data recovery. Automatically monitors health checks from your primary site
	- Geolocation Routing Policies: allow you to direct traffic based on the geographic location of where **the request originated from**
	- Geoproximity routing policy: allow you to direct traffic based on the geographic location of **your users and your AWS resources**
- Multi- Value Answe Policies let you configure Route 53 to return multiple values such as IP addresses for your web servers, in response to DNS queries.
-  Health Checks (not free): 
	![[Pasted image 20241001165610.png]]
- Route 53 Resolver: is a DNS server that allows you to resolve DNS queries between your on-premise network and your VPC
	- ![[Pasted image 20241001165819.png]]
- DNSSEC: Are a suite of extensions specifications by the Internet Engineering Task Force (IETF) for securing data exchanged in the Domain Name System (DNS) in Internet Protocol (IP) networks.
	- ![[Pasted image 20241001170421.png]]
- Zonal Shift: is a capability in Route 53 Application Recovery Controller (Route 53 ARC). Shift a load balancer resource away from an impaired Availability Zone with a single action. Only supported by ALB an NLB
	- ![[Pasted image 20241001170649.png]]
- Route 53 Profiles: lets you apply and manage DNS related Route 53 configurations accross many VPCs and in differente AWS accounts

## AWS Global Accelerator
can find the optimal path from the end user to your web servers. Is  deployed within Edge Locations, so you send user traffic to an Edge location insted of directly to your web application.
	 ![[Pasted image 20241001171236.png]]

## Cloud Front
![[Pasted image 20241001172029.png]]
- Cloud Front is a CDN (negli appunti AWS c'è un altra definizione)
	- CDN(Content Delivery Network): A CDN is a distributed network of servers that delivrs web pages and content to users based on their geographical location, the origin of the wbpage and a content delivery server 

- Lambda Edge: are lambda functions that override the behaviour of request and responses 
	![[Pasted image 20241001172905.png]]
- CloudFront functions:
	![[Pasted image 20241001173411.png]]
- CloudFront functions vs Lambda Edge:
	![[Pasted image 20241001173651.png]]
	![[Pasted image 20241001173709.png]]
- CloudFront Origin: is the source where CloudFront will send the requests





## AWS Shield
 - **AWS Shield:** • Provides protection from attacks of layer **3,4 and 7**
	- **AWS Shield Standard:** • Free service that is activated for every AWS customer • Provides protection from attacks such as SYN/UDP Floods, Reflection attacks and other layer 3/layer 4 attacks 
	- **AWS Shield Advanced:** • Optional DDoS mitigation service ($3,000 per month per organization) 
		- **Protect against more sophisticated attack on:**
			- Amazon EC2
			- Elastic Load Balancing (ELB), 
			- Amazon CloudFront
			- AWS Global Accelerator
			- Route 53
		- 24/7 access to AWS DDoS response team (DRP) • Protect against higher fees during usage spikes due to DDoS

## WAF
Protects your web applications from common web exploits **Layer 7**.

![[Pasted image 20241004094345.png]]
## Cloud HSM
**HSM:**
Hardware Security Module it is a piece of hardware designed to store encryption keys. HSM hold keys in memory and never write them to disk.
It follows FIPS (Federal Information Processing Standards): US and CANAdian governmant standard for cryptographic modules.

2 types:
- Multi-tenant (FIPS 140-2 Level 2 Compliant): 
- Single-Tenant (FIPS 140-2 Level 3 Compliant):

**Cloud HSM:** 
Is single tenant HSM as a service that automates hardware provisioning. Enables you to generate and use your encryption keys on a FIPS 140-2 Level 3 validated hardware.

It is easy to migrate aws key between cloudHSM.

## Amazon GuardDuty
Intrusion Detection System and Intrusion Protection System.(IDS)

Intelligent **Threat discovery** to protect your AWS Account. Uses Machine Learning algorithms, **anomaly detection**, 3rd party data. **Uses EventBridge and lambda funtion to make action**. Check for (CloudTrail logs, VPC flow logs, DNS logs)

## AWS Firewall Manager
 AWS Firewall Manager allows  you to centrally configure and manage firewall rules accross accounts and applications. 
 ![[Pasted image 20241003170926.png]]
## AWS Inspector
Amazon Inspector is an **automated** security assessment service that helps improve the security and compliance of applications deployed **on your Amazon EC2 instances**. Amazon Inspector automatically assesses applications for exposure.
## Amazon Macie
![[Pasted image 20241003171751.png]]
## AWS Security Hub
Central security tool to manage security across several AWS accounts and automate security checks. Allow you to generate a security score to determine your security posture. Allows you to enable standards(collection of security control)ù

## KMS
![[Pasted image 20241003154434.png]]
- Multi tenant: means there are multiple customers that are using the same piece of hardware. In this way is it possible to reduce the cost of the service. Each part is isolated per user.
	- **CloudHSM:** is single tenant
- type of key
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
- CMK(Customer Master Key): are the primary resources in AWS KMS. Is a logical representation of a master key . Support both Symmethric(one key) and Assymethric CMKs(2 keys).
## Audit Manager
Continually audit your AWS usage to simplify risk and compliance assessment. 
AWS Audit Manager contains: Framework Library, Control Library.
You can create assesments to review the evidence collected and generate an assesment report.

## AWS Certificate Manager
 Let’s you easily provision, manage, and deploy SSL/TLS Certificates. 2 types of certificate **Public, Private**. ACM can be attached to the following AWS resources: **Elastic Load Balancer,CloudFront, API Gateway, Elastic Beanstalk.**
 
 - **Terminating SSL at the Load Balancer.** All traffic in-transit beryond the ALB is unencrypted. You can add as many as EC2 instances that you want
 ![[Pasted image 20241003163140.png]]
 - **Terminating SSL End-to-End**: Traffic is encrypted in-transit all the way to the application
 ![[Pasted image 20241003163241.png]]
## AWS Cognito
Is a Customer identity and access management system. It provides authentication, authorization and user management for your web and mobile apps. It also provides authentication to AWS Services 
![[Pasted image 20241003163859.png]]
## Amazon Detective
Amazon Detective analyzes, investigates, and quickly identifies the root cause of security issues or suspicious activities (using ML and graphs). More powerful than **GuardDuty, Macie, and Security Hub.**
![[Pasted image 20241003164356.png]]

## Directory Service
A directory service  that maps the names of network resources to their network addresses.
A directory service is shared information infrastructure for locating, managing, administering and organizing resources.

Well known services:
- Domain Name Service (DNS), Microsoft Active Directory, Apache Directory Server, Oracle Internet Directory, Open LDAP, CLoud Identity, JumpCloud
AWS Directory service: provides multiple ways to use Microsoft Active Directory. 
- Simple AD: Not available for all the regions.  powered by Sambda.
- AD Connector: a proxy service to connect your existing on-premise AD Directory
- AWS Managed Microsoft AD: A full feature managed version of MS active directory
- Amazon Cognito
## Secret Manager
Store and rotate automatically database credentials. Enforce encryption at rest by using KMS. It as a cost. For credential access monitor you can use Cloud Trail.

You can setup automatic rotation, is not enabled by default. 
(Under the hood it uses lambda)
# Other
## Elastic Map Reduce (EMR)

- Used to create **Big Data clusters** to **analyze and process** vast amounts of data
- Uses **Hadoop**, an open-source framework, to distribute your data and processing across a **cluster of 100s of EC2 instances**.
- Supports open-source tools such as **Apache Spark**, **HBase**, **Presto**, **Flink**, etc.
- EMR takes care of all the provisioning and configuration
- **Auto-scaling**
- Integrated with **Spot Instances**
- Can be used to process large amounts of log files
## Disaster Recovery

- Any event that has a negative impact on a company’s business continuity or finances is a disaster
- **Recovery Point Objective (RPO)**: how often you backup your data (determines how much data are you willing to lose in case of a disaster)
- **Recovery Time Objective (RTO)**: how long it takes to recover from the disaster (down time)
- Diagram
    - ![attachments/Pasted image 20220513102636.jpg](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/attachments/Pasted%20image%2020220513102636.jpg)

### Strategies
#### Backup & Restore

- High RPO (hours)
- Need to spin up instances and restore volumes from snapshots in case of disaster => High RTO
- Cheapest & easiest to manage

#### Pilot Light

- **Critical parts of the app are always running** in the cloud (eg. continuous replication of data to another region)
- Low RPO (minutes)
- Critical systems are already up => Low RTO
- Ideal when RPO should be in minutes and the solution should be inexpensive
- DB is critical so it is replicated continuously but EC2 instance is spin up only when a disaster strikes

#### Warm Standby

- A complete backup system is up and running at the minimum capacity. This system is quickly scaled to production capacity in case of a disaster.
- Very low RPO & RTO (minutes)
- Expensive

#### Multi-Site or Hot Site Approach

- A backup system is running at full production capacity and the request can be routed to either the main or the backup system.
- Multi-data center approach
- Lowest RPO & RTO (minutes or seconds)
- Very Expensive

## CloudFormation

- **Infrastructure as Code (IaC)** allows us to write our infrastructure as a config file which can be easily replicated & versioned using Git
- **Declarative** way of outlining your AWS Infrastructure (no need to specify ordering and orchestration)
- Resources within a stack is tagged with an identifier (easy to track cost of each stack)
- Ability to destroy and re-create infrastructure on the fly
- Deleting a stack deletes every single artifact that was created by CloudFormation (clean way of deleting resources)

### CloudFormation Templates

- YAML file that defines a CloudFormation Stack
- **Templates have to be uploaded in S3** and then referenced in CloudFormation
- To update a template, upload a new version of the template
- Template components:
    - **Resources**: AWS resources declared in the template (**mandatory**)
    - Parameters: Dynamic inputs for your template
    - Mappings: Static variables for your template
    - Outputs: References to what has been created (will be returned upon stack creation)
    - Conditionals: List of conditions to perform resource creation
    - Metadata
- Templates helpers:
    - References
    - Functions

> - You can associate the `CreationPolicy` attribute with a resource to prevent its status from reaching create complete until CloudFormation receives a specified number of `cfn-signal` or the timeout period is exceeded.
> - Use CloudFormation with **securely configured templates** to ensure that applications are deployed in secure configurations

### Stack Sets

- Create, update, or delete stacks across **multiple accounts and regions** with a single operation
- When you update a stack set, all associated stack instances are updated throughout all accounts and regions.

### Updating Stacks

- CloudFormation provides two methods for updating stacks:
    - **Direct update**
        - CloudFormation immediately deploys the submitted changes. You cannot preview the changes.
    - **Change Sets**
        - You can preview the changes CloudFormation will make to your stack, and then decide whether to apply those changes.


## AWS API
- Smithy: is AWS open source interface definition language for web services. It is used to define service
- STS: Temporary limited credential. global service that go through a singlel endpoint
- Signing API: when you send API request, you sign the requests so that AWS can identify who sent them
- Service Enpoints: Is the URL of the entry point for an AWS web services
	types:
	- **Global Endpoints**
	- **Regional Endpoints**
	- **FIPS Endpoints**
	- **Dualstack Endpoints**
	example:
		![[Pasted image 20240930165811.png]]
- AWS CLI input flag: we can populate parameters if the --cli-input flag is available for a subcommand
- Configuration Files: 
	- ![[Pasted image 20240930170425.png]]
- Named Profiles: AWS config files support the ability to have multiple profiles. Profiles allow you to switch different configurations quickly for different environment.
- AWS Configure Commands:
- AWS CLI environment variable: watch the video to check all the diffeerent options
- AWS Autocompletion Options: AWS Completer (not used old version), AWS Shell Shell (Not recommended by the people), AWS CLI Auto prompt.
- AWS CLI Autoprompt 




## GraphSQL and AppSync

- **GraphQL**: is an open-source agnostic query adaptor that allows you to query data from different data sources. GraphQL is used to build APIs where clients will send a query for nested data. GraphQL mitigates the issue of versioned or rapidly changing APIs compared to REST API because you can request the data you want
- **AppSync**: Store and sync data across mobile and web apps in real-time. Makes use of GraphQL.
	![[Pasted image 20241002104051.png]]

## OpenSearch Service
Is a managed full-text search service that makes it easy to deploy operate and scale OpenSearch and ElasticSearch, a popular open-source search and analytics engine.

## Elastic Transcoder
Elastic Transcoder is video-transcoding server, used to convert media files stored in S3 into media files in the formats required by consumer playback devices. (Very expensive)

![[Pasted image 20241002105413.png]]
## SNS
- **Publish Subscribe pattern:** 
	![[Pasted image 20241002105940.png]]
- **SNS**: Amazon Simple Notification Service (Amazon SNS) is a managed service that provides message delivery from publishers to subscribers (also known as _producers_ and _consumers_). In this way is possible to send one message to many receivers. Used to decouple microvservices.
	- Source: Publisher
	- Destinations: They are the subscribers:
		- Application to application (A2A)
		- Application to person (A2P)
- **Topics:** allows you to group multiple subscriptions together.
	- 2 types: **Standard** and **FIFO**
	- ![[Pasted image 20241004152801.png]]
- **Messages:** Is it possible to programmatically publish messages to SNS Topic. Maximum limit of message dimension: 256KB. Message bigger than 256KB can be publish using AMAzon SNS Ectended Client library (Max: 2GB). 
- **Subscription:** A subscription can only subscribe to one protocol and one topic
	![[Pasted image 20241002111335.png]]

- **Filter policy:** allows you to filter a subset a messages only to be delivery.
- **Message Data Protection**: Message data protection safeguards the data that's published to your Amazon SNS Topic. by using protection polices to audit, mask,redact, or block the sensitive information that moves between applications or AWS services.
- Raw Message Delivery 
- Delivery Policy: SNS Delivery Policy retries the delivery of messages when server-side errors occur. Each delivery protocol has its own delivery policy.
	- ![[Pasted image 20241004155004.png]]
- **SNS Dead Letter Queue (DLQ)** will send fail message attempts to an SQS queue. Topic and Queue type need to match.
	- ![[Pasted image 20241004155317.png]]
## SQS
![[Pasted image 20241002111647.png]]
![[Pasted image 20241004155752.png]]
Maximum limit of message dimension: 256KB. Message bigger than 256KB can be publish using AMAzon SQS Extended Client library (Max: 2GB). 

- 2 types of queue: 
	- **Standard:** allows you to send nearly unlimited number of transactions per second
	- **FIFO:** guarantees the order of messages when being consumed. 300 transaction, no duplicates, order by id, 10 message read a time.
- **ABAC**: Is an authorization process that defines permissions based on tags that are attached to users aqnd AWS resources. Use tag and aliases.
- **Acccess Policy:** Allows you to grant other principals permission to the SQS Queue.
- **Message Metadata:** allows you to attach metadata to messages
- **Visibility Timeout:** is a period of time message will be invisible after they are read/consumed by an application to avoid being read and processed by other applications
- **Delay Queues:** let you postpone the delivery of new messages to consumers for a number of second when your app needs more time. Message remain invisible to consumers for the duration of the delay period.
- **Message Timers:** let you specify an initial invisibility period for an individual message when sending to the queue
- **Temporary Queues:** Let you specify an initial invisibility period for an individual message when sending to the queue. Don't support timers on individual message
- **Short vs Long Polling:**
	- Polling is the method by which we retrive messages from the queues
	- ![[Pasted image 20241004163711.png]]
## Amazon MQ 
Is a managed message broker service for the opensource Apache ActiveMQ and RabbitMQ
![[Pasted image 20241002114243.png]]
- AMQP: 
	
- **MQTT**:
	![[Pasted image 20241002115035.png]]
- **STOMP:**
	![[Pasted image 20241002115116.png]]



## Amazon Kinesis
AWS fully managed solution for collecting processing, and analyzing streaming real-time data in the cloud.

- 4 Different types
	- Kineses Data Streams: is a real time data streaming service. Most flexible option 
		- 2 capacity mode
			![[Pasted image 20241002145345.png]]
		- Producer and consumer
			![[Pasted image 20241002150103.png]]
	- Amazon Data FireHose: Serverless, simple version of data streams. allows for simple transformation and delivery of data.
		- Usage
			- You choose one consumer from a predefined list
			- Data immediatly disappears once it's consumed
		- Sources (Producers): In firehose allows easily to configure a source
		- Destination(Consumer):
			![[Pasted image 20241002152246.png]]
		- Data Transformation: before data is sent to a destination it can be tranformed with AWS Lambda
		- Dynamic Partitioning: enables you to continuosly streaming data by using keys within data and then deliver the data grouped by these keys into Amazon S3 Prefixes. Makes easier to run anlalytics. Once enabled it cnnot be disabled.
		- Convert record format: Data firehose can convert JSON data into different file formats before being delivered to S3
	- Manage Service for Apache Fink: Is a fully managed AWS service, to stream live video from devices to tha AWS cloud, or build applications for real time video processing or batch oriented analytics.
	- Kinesis video streams: allows you to run queries agains that is flowing through your real time stream so you can create and analysis on emerging data.
		![[Pasted image 20241002153311.png]]
- Shards:
	![[Pasted image 20241002150247.png]]
- Partition Keys and Sequence Number:
	![[Pasted image 20241002150355.png]]
- Retention period: how long data will remain in the stream. It can be increase
- Data Streams CLI: 
	- Using the AWS CLI the PutRecord alow us to send data to the stream. Note that data has to be based64 encoded
	- Using the AWS CLI the getRecords we can retrive data
- EFO(Enhance Fan Out): allows upto 20 consumers to retrive records from a stream throughput of up to 2 MB of data per second per shard
- KPL managed library by AWS to let you publish data to a Kineses data stream. Is a Java library
- KCL: Kinese Client Library is a java library that makes it easy for developers to easily consume data for kineses. via the MultiDaemon other programming languages can be used



## AWS Data Exchange
Is a catalogue of third-party datasets. You can download for free subscribe or purchase datasets.

You can download dataset and upload your own dataset. Data grants is the tool that allow you to control access to your datasets.

## AWS Glue
AWS Glue is serverless data integration that makes it easy for analytics users to discover, prepare, move and integrate data from multiple sources.

![[Pasted image 20241003092304.png]]
- 3 Engines for AWS Glue Jobs
	- Python Shell Engine
	- Ray Job
	- Spark
AWS Glue ETL jobs are charged based on the number of data processing units DPUs
- AWS GLue allocates 10DPUs to each hspark job
- 2 DPU to each Spark Streaming job
- AWS Glue allocates 6 M-DPUs to each Ray job
A combination of Work type and Number of worker will determine DPUs

- AWS Glue Studio allows to visually build ETL pipelines
	- Pipeline:
		- Composed by nodes
			- Sources
			- Transform
			- Targets
		- You can use version control
- AWS Glue  Data Catalog: is a fully manage Apache Hive Metastore-compatible catalog service that makesit easy for customer to store, anntoate and share metadata about their data
	- Components:
		- Glue Database: is a container for multiple AWS Glue tables
		- Table
			- Glue Table: metadata definition that represent your data, including schema.
			- Apache Iceberg table
		- Glue Crawler:Is a tool that is used to analyze a targeted data source to determine its schema and generate AWS Glue Data Tables
			- Data Sources: S3, JDBC, DynamoDb, MongoDB, Delta lake, Apache Iceberg, Hudi
			- Can be run on demand or on schedule

## Amazon MSK
Amazon Managed Streaming for Apache Kafka is a fully managed service that enables you to build and run application that use Apache Kafka to process streaming data. 
Amzon utilizes Zookeper servers.

Amazon MSK:
- Provisioned: you managed the broker instances 
- Serverless: you pay for what you use, and you don't have to manage instances

**Bootstrap brokers** refers to list of brokers endpoints that an Apache Kafka client use as a starting point to connect to the cluster.

Zookeper connection string URL is used with Kafka to specify the host and port of the ZooKeeper ensemble that Kafka should connect to for managing cluster metadata coordinator.

**Amazon MSK Connect:** is a feature of Amazon MSK that makes it easy for developers to stream data to and from their apache kafka clusters.

- Apache kafka: Open source streaming platform to create high performance data pipelines, streaming analytics, data integration, and mission critical applications. (Developed by Linkedin). based on Consumer/Producer architecture.




# Develop
## ElasticBeanStalk 
With Elastic Beanstalk you can quickly deploy and manage applications in the AWS Cloud without having to learn about the infrastructure that runs those applications. 

**Is not reccomended for production level application**

- Supported Languages
	![[Pasted image 20241002143955.png]]
- Web vs Worker environment
	![[Pasted image 20241002144228.png]]
	- Web environment types:
		- ![[Pasted image 20241002144443.png]]
## DeviceFarm
Give you access to virtual mobile devices and environment to testing mobile and web app.
## AWS Amplify
A set of tools and services that helps you develop and deploy scalable full stack web and mobile applications

Components:
![[Pasted image 20241002102613.png]]

Support a set of famous framework (Angular, React, Flutter ecc...)
## Amazon API Gateway

**Open API**: is a specification, defines a standard, language agnostic interface to RESTFUL api. Represented as either JSON or YAML.

**API Gateway:** is a program that sits between a single-entry point and multiple backends.
**Amazon API Gateway:** is a soultion for creating secure APIs in your cloud environment at any scale.
Create APIs that act as a front door for applications to access data, business logic or functionality from a back-end service.
![[Pasted image 20241003095122.png]]
- 3 types:
	- Rest API (API Gateway V1)
		- Higher cost
		- Both private and public options
	- HTTP API(API Gateway V2)
		- Simple feature
		- Only public APIs
	- Web Socket API: persistent connections for real time use cases such as chat application or dashboard

Diffeerence between REST and HTTP API Gateway:
![[Pasted image 20241003100632.png]]
![[Pasted image 20241003100745.png]]
**REST API Components:**
![[Pasted image 20241003101019.png]]

**HTTP API Components:**
![[Pasted image 20241003101213.png]]

## ECR
Private Docker Registry on AWS, similar to docker hub. Makes it easier for developers to store, manage, and deploy Docker container images.
Lets you to store Docker and Open Container Initiative (OCI) Images.

You can 
- controll acces via Register Policy:
	- ReplicateImage
	- BatchImportUpstreamImage
- Controll access via Repo Policy:
	- DescribeImages
	- DescribeReposiory

You must  remotely lognt using docker in you environmetn

Image tag mutability feature prevent images tags from being overwritten
ECR Lifecyle policy: can be used to expire old images based on specific criteria

## ECS
 Amazon Elastic Container Service (Amazon ECS) is a fully managed container orchestration service that helps you easily deploy, manage, and scale containerized applications. Compare to fargate you need to build the infrastructure on your own (EC2 instances).
 - **Cluster**
 - **Task definition**
 - **Task**
 - **Service**
 - **Conatiner Agent**
 - **ECS Controller / Scheduler**

**ECS Fargate**
AWS Fargate is a technology that you can use with Amazon ECS to run [containers](https://aws.amazon.com/what-are-containers) without having to manage servers or clusters of Amazon EC2 instances. With AWS Fargate, you no longer have to provision, configure, or scale clusters of virtual machines to run containers. 
![[Pasted image 20241003143010.png]]

You can apply a security group to a task
You can apply an IAM role to the Task

- **Execution role**: used to prepare or manage the contianer
- **Task role:** is the role that is used by the running compute of the container (when the container is running)
- **ECS capacity providers** manage the scaling of the infracture for tasks in your clusters.

- **ECS Task Lifecycle:**
	![[Pasted image 20241003143946.png]]
- **Task Definition:**
	- Family, Execution role, Task role, Network Mode, CPU and Memory, Requires compatibilities, Container definition
	- Container Definition
		- Name, Dockerhub, Essential, Health Chekcs, port mapping, log configuration, environment, secrets
- Port Mappings
	![[Pasted image 20241003144650.png]]
- ECS Exec: allows you to direct interact with containers without needing to first interact with the host conatiner operating system, open inbound ports, or manage SSH Keys. cannot be used with AWS console.
- Log configuration: You can set log driver that tells where the container should log. There are other third-party log driver. AWS log is blocked by default you need to configure it to enable it.
- ECS Service Connect: makes it easy to setup a service mesh for service-to-service communication. Evolution of AppMesh.
	- Service Connect will deploy a sidecar proxy container. You can use the service discovery name to easily talk to other service
- ECS Optimized AMIs are preconfigured with the requirements and reccomnadations to run your containerA
	![[Pasted image 20241003150216.png]]
- ECS ANywhere: allows you to register external VMS residing from you ron-premises network to your cluster
	![[Pasted image 20241003151049.png]]
## EKS
Amazon Elastic Kubernetis Service is a managed service that eliminates the need to install, operate, and maintain your own kubernetes control panel
![[Pasted image 20241003151405.png]]
 EKS can use for its compute nodes: ...
 EKS Connector: if you want to connect your own kubernetis cluster
 EKS CTL: is CLI tool for waily setting up kubernetis clusters on AWS.
 - EKS Distro: is a Kubernetis distribution based on and used by EKS to create reliable and secure kubernetis clusters.
	 ![[Pasted image 20241003152350.png]]
- **EKS anywhere**: is a deployment option for Amazon EKS to easily create and operate kubernetis clusters on premises with your own Vms or bare metal hosts.
- Traces and Spans: 
	- Trace is a data/execution path through the system, and can be thought of as a directed acyclic graph (DAG) of spans
	- Span: represents a logical unit of work in Jaeger that has an operation name, the start time of the operation, and the duration. Spans may be nested and ordered to model causal relationships
- AWS Distro for Open Telemetry is a secure, production ready, AWS supported distribution of the Open Telemetry project 
- Prometheus: Open soruce monitoring and alerting toolkit originally built at SounCLoud. Is a timeseries database (Collect and stores itrs metrics as time series data). 
- Amazon Managed Service for Prometheus (AMP) is a Prometheus compatible monitoring service for container infraswtructure and application metrics
	![[Pasted image 20241003153857.png]]
- Graphana: is an open source analytics web application. is used with timeseries database like Prometheus
- Amazon Managed Service for Graphana: is fully managed and secure data visualization service that you can use to instanly query, correlate, and visualize operational metrics , logs, and traces from multiple sources.


## Lake Formation 
**Data Lakes**: A data lake is a centralized data repositoru for unstructured and semi structured data. contain vast amount of data. Data lakes generally use object(blob) or file as its storage type.
![[Pasted image 20241003093952.png]]
**AWS Lake Formation:** is a data lake to centrally govern secure and globally share data for analytics and machine learning.
![[Pasted image 20241003094159.png]]

## Amazon Dev Tools
- Amazon Q is an AI chatboat using multiple LLm models via Bedrock Ask Amazon Q a question similar to chatGPT or other generative AI. 

![[Pasted image 20241003174240.png]]
- **Amazon CodeWhisper**: is a realt-time AI coding companion. Genaretes suggested code while you're writing code.
![[Pasted image 20241003174535.png]]
# Database
## RDS
It’s a managed DB service for DB use SQL as a query language. It allows you to create databases in the cloud that are managed by AWS. (puoi decidere di usa un db relazionale anche senza rds sulle ec2 ma te lo devi gestire tu). 
![[Pasted image 20241003101453.png]]

**Encryption:**
- Encryption in transit: enabled by default
- Encryption at rest: can be enabled 
- Encrypt also automated backups, snapshot and read replica
- Can be enabled only during creation

**RDS Backup:**
- Types
	- Automated backup: choose retention period between 0 and 35 days
		- enabled by default
		- stored in s3
		- no ccharge for additional data in automated backups
	- Manual Snapshot
		- taken manually
		- additional storage charge 
- Restoring a backup: creates a new RDS instance and restores data in that instance 
		- Restoring a manual snapshot
		- Restoring Point in time (PITR): similar but provide restore time
- Take long time to restore database: is not immidiate.

**DB Subnet Group**: collection of subnet that you create in a VPC and that you then designate for your DB instances.
- Each DB subnet group should have subnets in at least two Availability Zones in a given AWS Region.
- RDS will choose a subnet from your subnet group to deploy your RDS Instance
- Subnets in a DB subnet group are either public or private
- For a DB instance to be publicly accessible, all of the subnets in its DB subnet group must be public
![[Pasted image 20241003103052.png]]
**Scelte architetturali:**
- **Read Replicas:** 
	- **disaster recovery** solution if the primary DB instance fails  
	- Improve **read contention** which means improve performance latency. 
		- Read contention: when multiple proccesses or instances competing for access to the same index or data block at the same time.
	- You must have automatic backups enabled. 
	- type of replicaion: asyncronous replication
	- Maximum of 5 replicas of a database
	- Replica techniques:
		![[Pasted image 20241003104330.png]]
	
- **Multi AZ vs Read Replicas**
	![[Pasted image 20241003104808.png]]
- **DB instances:** is an isolated database environment running in the clous
	- DB instance can contain one or multiple user created database
	- Each database has user defined database identifier which forms part of the DNS hostname
	- DB instance Class: just like ec2 instance class determine available compute memory or a db instance
	- DB instance use Elastic Block Storage (EBS) volumes for database and log storage
	- Maximum storage of 64TB
- **RDS Performance Insights:** helps you easily identify bottlenecks and performance issues. Is default by default and provides  1 week of performing data. For additional cost you can change the retention period to 2 years.
- **RDS Custom:** automates database administration tasks and operatio. Allows customers to directlu manage aspects of RDS aspects of RDS instead of AWS, for company that needs third party applications for their dataabse.
	![[Pasted image 20241003105648.png]]
- **RDS Proxy:** create a connection pooler so that short lived AWS Lambda functions connecting to RDS does not quick exahaust all connection. Reuse existing connection to do this.
	![[Pasted image 20241003110329.png]]
- **Optimized Reads and Writes:** allow database operations to maximize perfrmance efficency and throughtput. Use NVMe based SSD block storage instead
	- Query use temporary tables
- **IAM Authentication:** allows you to authenticate with an IAM authentication token to an RDS instance's database insteas of using a password.
	 ![[Pasted image 20241003110737.png]]
	- You have to create a policy and attach to user
	- You have to create db user 
	- You need to generate auth token to be used for password authentication
- **Kerberos Authentication:** Is a network auth protocol which is also directly integreted into Microsoft Active Directoruy.
- **RDS Secrets Manager Integration:** can manage an RDS instance's master user password, allowing it to be rotated. default rotate pass every 7 days.
- **Master User Account:** is the initial databse account that's created when you provision a new Db instance. Has full privilages. Pass and username set at creation time.
- **Database Activity Streams:** allows you to control administrator access to data streams to secure both external and internal security threats. It can be turned on from CLI.
	- Work with kinesis pushing data from rds to kinesis in real time.
- **Parameter Groups:** act as a container for engine configuration values that are applied to one or more DB instances. Let you change database parameters specify how the database is configured.
- **Public Accessibility:** is an option that changes if the DNS Endpoint reslve to the private IP address from traffic from outside the VPC
- **Establishing Public Connections:** few options:
	- Use a DB management/ DB IDE 
	- AWS CloudShell and use database client or database driver
	- Programmatcially connect with database driver from your language of choice
	- Connection url string: A single string containing all the parameters to connect to a database drivers and database command line clients.
- **Establishing Private Connections:** 
	- ![[Pasted image 20241003113624.png]]
	- How
		- Using Cloud9
		- Through Bation or Jumpbox
		- Launch EC2 instances and connect via SSH or session manager
		- Use AWS Client VPN to connect 
		- For On-premise using AWS Direct Connect
		- AWS CLoudShell can't be used because is not in the same VPC
- **RDS Security Groups:** In order to establish connection for both public and private you need to open the port
- **RDS Blue Green Deployments:** copies a production database environment in a separate, synchronized staging environment
- **RDS Extended Support:** allows you to run your database on a major engine version past the RDS end of standard support date for an additional cost.

## Aurora
***Serverless***,• Automated database instantiation and auto -scaling based on actual usage • PostgreSQL and MySQL are both supported as Aurora Serverless DB. **Aurora costs more than RDS (20% more).

- **Aurora Scaling:** 
	![[Pasted image 20241003114728.png]]
- **Aurora Serverless Provisioned:** default compute configuration for Aurora. primary db that perform read and writes and up to 15 Replica. primary DB instance is not created by default
- **Reader vs Writer Instances:** 
	![[Pasted image 20241003115330.png]]
- **Aurora Serverless V2:** fully manages the autoscaling configuration for Amazon Aurora
	- Capacity is adjusted automatically based on application demand
	- charge only for resources used
	- use case: Highly variable workloads
	- Does not scale to zeero, must mainaine at least 0.5 ACU
	![[Pasted image 20241003115938.png]]
- **Aurora Serverless V2 vs Provisioned:**
	![[Pasted image 20241003120332.png]]
- **Aurora Global Database:** is a database spanning multiple regions for global low latency  and high availabilty. Primary cluster is in a separate region
- **RDS Data API:** allows you to use HTTP to securely query an Aurora database. Unlimited max request per seconds. Unlimited max request per seconds. Must be enabled 
## DocumentDB
 - **MongoDB:** use BSON(Binary JSON). has a shell(mongosh).
 - **DocumentDB**: is a NOSQL document database that is "MongoDB compatible" MongoDB is very popular NoSQL among developers. 
	![[Pasted image 20241003122328.png]]
## DynamoDB
Amazon DynamoDB is a ***serverless***, NoSQL (key/value), fully managed database with single-digit millisecond performance at any scale. **• Fully Managed Highly available with replication across 3 AZ.

![[Pasted image 20241003122625.png]]
- **Read Consistency:** When data needs to be updated in all of its copy and keep data consistent.
	- there are different types of read consistency:
		![[Pasted image 20241003123002.png]]
- **Partitions:** is an allocation of storage for a able, and automatically replicated accross multiple AZs. DyanmoDB creates partitions for you as your data grow. starts off with single partition
	- creates a partition evry 10 GB or when you exceed ...
- **Primary Keys:** cannot be changed.
- **Simple Primary Key:** a primary key with ony a Partition key to choose which partition
- **Composite Primary Key:** a primary key with both a Partition and a sort key to choose partition
- **Query:** 
	![[Pasted image 20241003124503.png]]
- **Scan:** scan throush all items return one or more items. (much less efficient then query - not said by amazon)


## Amazon Keyspace
![[Pasted image 20241003125003.png]]


## Neptune
Amazon Neptune is a fast, reliable, fully managed graph database service that makes it easy to build and run applications that work with highly connected datasets.

![[Pasted image 20241003130018.png]]
- Netpune Database: 2 types
	- Provisioned: you choose an instance type
	- Serverless: set a min and max Nepune Capacity units
- Neptun Anlyses:  provide capabilites to run large-scale graph analytics algorithms efficiently
- Gremlin: is the graphic traversal language for Apache TinkerPop
	- designed to the WORA (Write once, run anywhere)
	- real time database query (OLTP)
	- batch analytics query (OLAP)
- OpenCypher: opensource implementation of cypher 
	- more developer friendly to write query insetad of Gremlin
- SparQL is an RDF(Resource Description Framework) query language. Allows users to write queries against what can loosely be called key-value data.



## ElastiCache
Amazon ElastiCache is a web service that makes it easy to set up, manage, and scale a distributed in-memory data store or cache environment in the cloud

 Can only accessible by resources in the same VPC
	- this ensure low-latency 
It can be cross region if enabled
- Deployment Options:
	- Serverless or Standard
		![[Pasted image 20241002154102.png]]
- Chaching types:
	- Memcache: is generally preferred for caching HTML fragments. Memcached is a simple key/value store. The trade off to being simple is taht it is very fast 
	- Redis: Can perform many different kind of operations on your data. It is very good for leaderboards, and keep track of unread notification data. It is very fast, but arguably not as fast as memcached
- Redis: Open source in-memory database store. Redis acts as caching layer, or a very fast database. is key/value store
	- Strings: most basic value, max length 512MB
		- Command:
	- Sets: unorder collection of strings. String are unique
		- Command.
	- Sorted Sets: are a collection of strings that are sorted based on an associated score
		- Command:
	- Lists: Order collection of strings, you can have duplicate
		- Command:
	- Hashes: represent mapping between string fields and string values
		- Command:
- MemCached: is an open-source distributed memory object caching system. It's a caching layer for web-applications. Key/value store
	- Command
		- Set
		- get
		- delete
		- incr
		- decr
		- add
		- replace
		- flush_all
		- appened
		- prepend
		- stats
## MemoryDB
Is a Redis-compatible in-memory database for ultra-fast performance
Suitable to be primary database. Slower writes but better performance in general.
## Amazon Redshift
 **Serverless,** An Amazon Redshift is an enterprise-class relational database based on postgresql, Is thinked for data warehouse and for OLAP(analytics and data warehousing). Storage is done by **column** and not by row. **Pay as you go based**

Datawerehouse: Built to store large quantities of historical data and enable fast, complex queries across all the data.

**OLAP** applications look at multiple records at the same time. You save memory because you fetch  just the columns of data you need
 instead of whole rows.

- Different type of configurations
	![[Pasted image 20241002163927.png]]
different type of nodes:
	![[Pasted image 20241002164003.png]]
- Compression
	- Redshift uses multiple compression techniques.
	- Similar data is stored sequentially on disk
	- Does not require indexes or materialized views, compared to traditional systems
	- When loading data to an empty table, data is sampled and the most appropriate compression scheme is selected automatically
- Processign
	- Uses massively Parallel Processing(MPP)
	- Automatically distributes data and query loads accross all nodes
	- Lets you easily add new nodes to your data warehouse while still maintaining fast query performance.
- Backup
	- enabled by default 1 day retention,up to 35
	- always attempt to keep 3 copies of your data
- Billing
	- total number hours
	- 1 unit per node
	- not charged for leader node hours only compute nodes incur changes
- Security
	- Dat in transit: SSL
	- Data at rest: AES 256
	- Encryption applied using KMS, CloudHSM
- Availability
	- Redshift is Single AZ. In order to reach high availability you need to run multiple redshift cluster in different AZ. Anapshot can be restored o a different AZ 

**CHEETSHET**
![[Pasted image 20241002165058.png]]

## Athena
**Serverless** query service to analyze data stored in Amazon S3 that uses standard SQL language to query the files and their content

![[Pasted image 20241002165508.png]]
- Athena SQL
	- Component
		- workgroup: saved queries which you grant permissions to other user to access
		- Data source: a group database 
		- Database: a group of tables
		- Table: data organized as group
		- dataset: raw data of table
	- Table
		- You can create using SQL and AWS Glue Wizard
	- SerDe: is a serialization and deserialization libraries for parsing data form different format
		- Can override the DDL configuration that you specify in Athena when you create your table
## Quantum Ledger Database
Used to review history of all the changes made to your **application data** over time, serverless. **Difference with Amazon Managed Blockchain**: no decentralization component

![[Pasted image 20241002105101.png]]

# Monitoring
## Cloud Trail
Is a service that enables governance compliance operational auditing of your AWS account

Very useful to monitor API calls and made Actions made on an AWS account.
Easily identify which users on an AWS account made the call.

Is Active by default and will collect logs for 90 days via Event History. If you need more than 90 days you need to create a Trail.
**To analyze a Trail you'd have to use Amazon Athena**
## CloudWatch
- **Amazon CloudWatch**: **High Availability**, Amazon CloudWatch monitors your Amazon Web Services (AWS) resources and the applications you run on AWS in real time. **Amazon CloudWatch is basically a metrics and logs repository. An AWS service puts metrics into the repository, and you retrieve statistics based on those metrics
	- **Amazon CloudWatch Alarms:(like budget but less powerful)** Alarms are used to trigger notifications for any metric • **Auto Scaling**: increase or decrease EC2 instances “desired” count • **EC2 Actions**: stop, terminate, reboot or recover an EC2 instance • **SNS notifications**: send a notification into an SNS topic 
		- You can create a billing alarm • **Billing data metric is stored in CloudWatch us-east-1** Billing data are for overall worldwide AWS costs • It’s for actual cost, not for projected costs
	- **Amazon CloudWatch Logs:** Is used to store, manage and access your log files. Is a centralized log management server
		 - CloudWatch Logs can collect log from:
			 - **EC2 machines or on-premises servers** • Elastic **Beanstalk**: collection of logs from application • **ECS:** collection from containers • **AWS Lambda**: collection from function logs • **CloudTrail** based on filter • **Route53**: Log DNS queries
		- **CloudWatch log groups**: a collection of log streams ex: /exampro/prod/app
		- **CloudWatch streams**: represent a sequence of events from an application or instance being monitored. Is created automatically but you can create your own.
		- **CloudWatch log events**: Represent a single event in a log files:
			![[Pasted image 20241004114212.png]]
		- **CloudWatch Insights:** enables you to interactively search and analyze your CloudWatch log data
			- CloudWatch discover fields, when reads a logs 
		- **CloudWatch Metrics:** time order set of data points, its a variable that monitored over time. You can provide you customize Metrics using CLI/SDK
		- **CloudWatch Agent**: Some metrics need to install CloudWatch agent to been tracked
			- ![[Pasted image 20241004115502.png]]
## Amazon EventBridge
**Event Bus**: An event bus receives events from a source and routates events to a target based on rules. 
**EventBridge** is a serverless event bus service that is used for application integration by streaming real-time data to your applications.

## Health Dashboard
- **Service Health Dashboard:** Show General status of the service in various region.
- **Personal Health Dashboard:** AWS Personal Health Dashboard provides alerts and guidance for AWS events that might affect your environment.

## Config
- Regional service
- Can be aggregated across regions and accounts
- Record configurations changes over time
- **Evaluate compliance of resources using config rules**
- **Does not prevent non-compliant actions** from happening (no deny)
- Evaluate config rules
    - for each config change (ex. configuration of EBS volume is changed)
    - at regular time intervals (ex. every 2 hours)
- Can make custom config rules (must be defined in Lambda functions) such as:
    - Check if each EBS disk is of type gp2
    - Check if each EC2 instance is t2.micro
- **Can be used along with CloudTrail** to get a timeline of changes in configuration and compliance overtime.
- **Integrates with EventBridge or SNS** to trigger notifications when AWS resources are non-compliant

### Remediation

- Automate remediation of non-compliant resources using **SSM Automation Documents**
    - AWS-Managed Automation Documents
    - **Custom Automation Documents**
        - to invoke a Lambda function for automation
- You can set Remediation Retries if the resource is still non-compliant after auto remediation
- Ex. if IAM access key expires (non-compliant), trigger an auto-remediation action to revoke unused IAM user credentials.


## X-Ray

- Provides an **end-to-end view of requests as they travel through your application**, and shows a map of your application’s underlying components.
- AWS X-Ray helps developers analyze and debug production, distributed applications, such as those built using a **micro-services architecture**.
- Helps understand how your application and its underlying services are performing to identify and troubleshoot the root cause of performance issues and errors.
- Can collect data across AWS Accounts. The **X-Ray agent** can **assume a role** to publish data into an account different from the one in which it is running. This enables you to publish data from various components of your application into a **central account**.
# Management
## Service Catalog
- ![[Pasted image 20241002115439.png]]
- Anatomy:
	![[Pasted image 20241002115631.png]]
- **Users**
	![[Pasted image 20241002115733.png]]



## AWS Resource Access Manager (AWS RAM)
  Share AWS resources that you own with other AWS accounts. **Avoid resource duplication**. Supported resources includes: Aurora, VPC Subnets, Transit Gateway, Route 53, EC2 Dedicated Hosts, License Manager Configurations

## AWS IAM Identity Center: 
Single sign-on for multi account in aws 
## OpsWorks

- **Chef & Puppet** are two open-source softwares that help you perform **server configuration** automatically, or repetitive actions
- AWS OpsWorks is nothing but **AWS Managed Chef & Puppet**
- They work great with EC2 & On Premise VM
- It’s an **alternative to AWS SSM**
- Exam tip: **Chef & Puppet ⇒ AWS OpsWorks**
## AWS Organizations

- **Global service**
- Manage multiple AWS accounts under an organization
    - 1 master account
    - member accounts
- An AWS account can only be part of one organization
- **Consolidated Billing** across all accounts (lower cost)
- Pricing benefits from aggregated usage of AWS resources
- API to automate AWS account creation (on demand account creation)
- Establish Cross Account Roles for Admin purposes where the master account can assume an admin role in any of the children accounts

> Organization API can only create member accounts. They cannot configure anything within those accounts (use [CloudFormation](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/CloudFormation) for that).

### Organizational Units (OU)[¶](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/aws%20organizations/#organizational-units-ou "Permanent link")

- Folders for grouping AWS accounts of an organization
- Can be nested
    - ![attachments/Pasted image 20220511200502.jpg](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/attachments/Pasted%20image%2020220511200502.jpg)

### Service Control Policies (SCP)

- **Whitelist or blacklist IAM actions at the OU or Account level**
- **Does not apply to the Master Account**
- Applies to all the Users and Roles of the member accounts, including the root user. So, if something is restricted for that account, even the root user of that account won’t be able to do it.
- Must have an explicit allow (**does not allow anything by default**)
- **Does not apply to service-linked roles**
- **Explicit Deny** has the highest precedence

### Migrating Accounts between Organizations

- To migrate member accounts from one organization to another
    1. Remove the member account from the old organization
    2. Send an invite to the member account from the new organization
    3. Accept the invite from the member account
- To migrate the master account
    1. Remove the member accounts from the organizations using procedure above
    2. Delete the old organization
    3. Repeat the process above to invite the old master account to the new org
## (IAM) Identity Access Manager
- Anatomy of IAM policy
	- ![[Pasted image 20241001113136.png]]
- Principle of Least Privilege:
	- Just Enough Access: Permitting only the exact actions  for the identity to perform task
	- Just In Time: Permitting the smallest length of duration a identity can use permissions
- Password policy: you can set expires date for user password in rder to rotate them
- Access Key: you can have a maximum of 2 access key per account
- MFA: (vedi appunti aws)
- Temporary Security Credentials: Generated dynamically, every time you use a roles a you a temporary security cred are created automatically
- IAM Identity federetion: the means of linking a person to eletronic identity and attribute. When you connect using facebook or google you're using identity fed
- STS: (vedi appunti AWS)
- Cross Account Role: You can grant users from different AWS account access to resources in your account through a Cross-Account Role. This allows you to not to have to create them a user account within your systems
	- ![[Pasted image 20241001115206.png]]
- IAM AssumeRoleWithWebIdentity: Returns a set of temporary security credentials for users who have been authenticated in a mobile or web application with web identity provider
	- ![[Pasted image 20241001115642.png]]
	- You always authenticate with the web identity first
## AWS Artifact
Portal that provides customers with on-demand access to AWS compliance documentation and AWS agreements.

## Amazon Workspaces

- Persistent **desktop virtualization service** that enables users to access data, applications, and resources from any supported device
- Provision either **Windows** or **Linux** desktops
- Managed & Secure **Cloud Desktop**
- Great to eliminate management of on-premise **VDI (Virtual Desktop Infrastructure)**
- Pay per usage (no provisioning)
- Integrated with **Microsoft Active Directory

# Pricing
## CostExplorer

- Visualize and manage your costs and service usage over time
- Create **custom reports** that analyze cost and usage data
- **Recommendations** to choose an optimal savings plan
- View usage for the **last 12 months & forecast up to 12 months**
## EC2 Pricing Models
 Type of purchasing option:
- **On-Demand Instances** : Has **high cost but no upfront payment** is a **pay-as-you-go** model. Thi is the default option when create EC2. Recommended for **short-term and un-interrupted** workloads commitment
- **Reserved Instance(1 & 3 years)**: up to 72% save, long workloads/ long workloads with flexible instances. 
	- 2 types: **Standard:** 72% save, **Convertible:** 54% save, you can change RI based on RI Attributes. If greater or equal in value.
		![[Pasted image 20241004110849.png]]
	- Type of payments: **All upfront, Partial upfront, No upfront**
	- **RI Attributes**: can affect cost (Instance type, Region, Tenancy, Platform)
	- **Limits**: Per month -> 20 Regional RI per region, 20, Zonal RI per zone
	- **RI Marketplace**: allows you to sell your unused Standard RI to recup your RI spend for RI you do not intend or cannot use.
- **Savings Plans (1 & 3 years)** :commitment to an amount of usage, long workload, up to 72%
- **Spot Instances** : short workloads, cheap, can lose instances (less reliable), up to 90%
- **Dedicated Hosts** and Dedicated Instance:
	 ![[Pasted image 20241004112027.png]]

**Capacity Reservations** : Is a service of EC2 that allows you to request a reserve of EC2 instance type for a specific Region and AZ



# Machine Learning 
## CodeGuru
![[Pasted image 20241002171037.png]]
## Comprehend
Fully managed and **serverless** service for Natural Language Processing. Uses machine learning to find and analyse insights and relationships in text. use cases: analyze customer interactions (emails) to find what leads to a positive or negative experience.

Analyze Text and extract: Entities, Key phrases, language,PII (personal information), Sentiment (feeling of the sentence), Targeted sentiment (specific word ), syntex, custom models

## Rekognition:
Amazon Rekognition is image and video reognition service. Analyze images and videos to detect and labels objects, people, clebrities.

Find objects, people, text, scenes in images and videos using ML • Facial analysis and facial search to do user verification, people counting

![[Pasted image 20241002175456.png]]
## Transcribe:
**Automatically convert speech to text** • Uses a deep learning process called automatic speech recognition (ASR) to convert speech to text quickly and accurately • Automatically remove Personally Identifiable Information (PII) using Redaction • Supports Automatic Language Identification for multi-lingual audio

## Polly:
Is a Text-to-speech service • Allowing you to create applications that talk
Engine Types (from less expensive to most expensive):
- Standard - 
- Long Form -
- Neural -
Lexicon - for specialized pronuciation
Speech Marks - metadata that describes speech

Use an xml based language: **Speech Synthesis Markup Language (SSML)**

## Translate:
Natural and accurate language translation

## SageMaker:
Fully managed service for developers / data scientists to build ML models

## Forecast:
Fully managed service that uses ML to deliver highly accurate forecasts • example: predict the future sales of a raincoat

Amazon Forecast Workflow: 
![[Pasted image 20241002172257.png]]

## Fraud Detection
Fully managed fraud detection service. Identify potentially illegal online activities such as online payment fraud and the creation of fake accounts

Predefined Model:
- Online Fraud Detection
- Transactional Fraud Detection 
- Account Takeover insight
You can also **create you custom model detection**

- Components:
	![[Pasted image 20241002172935.png]]
- In order to create your model you need to define event, we need defines **Lables, Entities and Variables**
## Kendra:
Fully managed **document search** service powered by Machine Learning. Use keyword based search with semantic contextual understanding capabilites.

Components:
![[Pasted image 20241002173433.png]]
Two version: Developer and Enterprise
![[Pasted image 20241002173605.png]]
You need to create Index

## Lex
is a conversion interface service. With Lex you can build conversational **voice and text chatbots**. 

Components:
![[Pasted image 20241002174306.png]]
## Personalize:
Fully managed ML-service to build apps with real-time personalized recommendations 

![[Pasted image 20241002174520.png]]

Data:
	![[Pasted image 20241002174606.png]]


## Textract:
Automatically extracts text, handwriting, and data from any scanned documents using AI and ML.

![[Pasted image 20241002175758.png]]

## Connect:
Amazon Connect is an AI-powered application that provides one seamless experience for your contact center customers and users. It's comprised of a full suite of features across communication channels.


