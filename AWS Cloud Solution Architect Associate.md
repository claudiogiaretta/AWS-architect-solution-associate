esami di prova: https://www.exampro.co/saa-c03

AWS_CLI_AUTO_PROMPT: aiuta a fare comandi usando il cli in aws

Informazioni utili:
- Non concentrarsi molto su kubernetis 
- La parte global infrastructure molto simile a quella del cloud practitioner

# Storage and Migration

## Simple Storage Service (S3)
### Intro

- Global Service
- Object store (key-value pairs)
- Buckets must have a globally unique name
- **Buckets are defined at the regional level**
- The key is composed of bucket + _prefix_ + **object name** s3://my_bucket _/my_folder/another_folder/_ **my_file.txt**
- There’s **no concept of directories** within buckets (just keys with very long names that contain slashes)
- **Max Object Size: 5TB**
- Durability: 99.999999999% (total 11 9's)
- **SYNC** command can be used to **copy data between buckets**, possibly in **different regions**

> - S3 delivers **strong read-after-write consistency** (if an object is overwritten and immediately read, S3 always returns the latest version of the object)
> - S3 is strongly consistent for all GET, PUT and LIST operations

### Bucket Versioning

- Enabled at the bucket level
- Protects against unintended deletes
- Ability to restore to a previous version
- Any file that is not versioned prior to enabling versioning will have version “null”
- Suspending versioning does not delete the previous versions, just disables it for the future
- To restore a deleted object, delete it's "delete marker"

> - Versioning can only be suspended once it has been enabled.
> - Once you version-enable a bucket, it can never return to an unversioned state.

### Encryption

- Can be enabled at the bucket level or at the object level
- **Server Side Encryption (SSE) - implemented by default**
    - **SSE-S3 (Default)**
        - Keys managed by S3
        - AES-256 encryption
        - HTTP or HTTPS can be used
        - Must set header: `"x-amz-server-side-encryption": "AES256"`
    - **SSE-KMS**
        - Keys managed by KMS
        - HTTP or HTTPS can be used
        - KMS provides control over who has access to what keys as well as audit trails
        - Must set header: `"x-amz-server-side-encryption": "aws:kms"`
    - **SSE-C**
        - Keys managed by the client
        - Client sends the key in HTTPS headers for encryption/decryption (S3 discards the key after the operation)
        - **HTTPS must be used** as key (secret) is being transferred
- **Client Side Encryption (CSE)**
    - Keys managed by the client
    - Client encrypts the object before sending it to S3 and decrypts it after retrieving it from S3

### Access Management
- **User based security**
    - IAM policies define which API calls should be allowed for a specific user
    - Preferred over bucket policy for **fine-grained access control**
- **Resource based security (Bucket Policy)**
    - Grant **public access** to the bucket
    - Force objects to be encrypted at upload
    - Cross-account access
    - Object Access Control List (ACL) - applies to the objects while uploading
    - Bucket Access Control List (ACL) - access policy that applies to the bucket

> An IAM principal can access an S3 object if the IAM permission allows it or the bucket policy allows it and there is no explicit deny.
> 
> By default, an S3 object is owned by the account that uploaded it even if the bucket is owned by another account. To get full access to the object, the object owner must explicitly grant the bucket owner access. As a bucket owner, you can create a bucket policy to require external users to grant `bucket-owner-full-control` when uploading objects so the bucket owner can have full access to the objects.

### S3 Static Websites

- Host static websites (may contain **client-side scripts**) and have them accessible on the public internet over **HTTP only** (for HTTPS, use [CloudFront](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/CloudFront) with S3 bucket as the origin)
- The website URL will be either of the following:
    - `<bucket-name>.s3-website-<region>.amazonaws.com`
    - `<bucket-name>.s3-website.<region>.amazonaws.com`
- If you get a `403 (Forbidden)` error, make sure the bucket policy allows public reads
- **For cross-origin access to the S3 bucket, we need to enable [CORS](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/Concepts#cross-origin-resource-sharing-cors) on the bucket**
    - ![attachments/Pasted image 20220507175558.jpg](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/attachments/Pasted%20image%2020220507175558.jpg)
- To host an S3 static website on a custom domain using Route 53, the bucket name should be the same as your domain or subdomain Ex. for subdomain `portal.tutorialsdojo.com`, the name of the bucket must be `portal.tutorialsdojo.com`

### MFA Delete
- MFA required to
    - permanently delete an object version
    - suspend versioning on the bucket
- **Bucket Versioning must be enabled**
- Can only be enabled or disabled by the root user

### Server Access Logging
- Most detailed way of logging access to S3 buckets (better than [CloudTrail](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/CloudTrail))
- Does not support **Data Events** & **Log File Validation** (use [CloudTrail](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/CloudTrail) for that)
- Store S3 access logs into another bucket
- Logging bucket should not be the same as monitored bucket (logging loop)

### Replication

- **Asynchronous replication**
- Objects are replicated with the **same version ID**
- Supports **cross-region, same region, bidirectional, batch replication**
- **Versioning must be enabled for source and destination buckets**
- For DELETE operations:
    - Replicate delete markers from source to target (optional)
    - Permanent deletes are not replicated
- There is **no chaining of replication**. So, if bucket 1 has replication into bucket 2, which has replication into bucket 3. Then objects created in bucket 1 are not replicated to bucket 3.

### Pre-signed URL

- Pre-signed URLs for S3 have temporary access token as query string parameters which allow anyone with the URL to temporarily access the resource before the URL expires (default 1h)
- Pre-signed URLs inherit the permission of the user who generated it
- Uses:
    - Allow only logged-in users to download a premium video
    - Allow users to upload files to a precise location in the bucket

### Storage Classes

- Data can be transitioned between storage classes manually or automatically using **lifecycle rules**
- Data can be put directly into any storage class
- **Standard**
    - **99.99% availability**
    - Most expensive
    - Instant retrieval
    - No cost on retrieval (only storage cost)
    - For frequently accessed data
- **Express One-Zone**: 50% discount but reduced availability
- **Infrequent Access**
    - For data that is infrequently accessed, but requires rapid access when needed
    - Lower storage cost than Standard but **cost on retrieval**
    - Can **move data into Infrequent Access from Standard only after 30 days**
    - Two types:
        - **Standard IA**
            - **99.9% Availability**
        - **One-Zone IA**
            - **99.5% Availability**
            - Data is lost if AZ fails
            - Storage for infrequently accessed data that can be easily recreated
- **Glacier**
    - For data archival
    - Cost for storage and retrieval
    - **Can move data into Glacier from Standard anytime**
    - Objects cannot be directly accessed, they first need to be restored which could take some time (depending on the tier) to fetch the object.
    - **Default encryption for data at rest and in-transit**
    - Three types:
        - **Glacier Instant Retrieval**
            - **99.9% availability**
            - Millisecond retrieval
            - Minimum storage duration of **90 days**
            - Great for data accessed once a quarter
        - **Glacier Flexible Retrieval**
            - **99.99% availability**
            - 3 retrieval flexibility (decreasing order of cost):
                - Expedited (1 to 5 minutes)
                    - Can **provision retrieval capacity** for reliability
                    - Without provisioned capacity expedited retrievals might be rejected in situations of high demand
                - Standard (3 to 5 hours)
                - Bulk (5 to 12 hours)
            - Minimum storage duration of **90 days**
        - **Glacier Deep Archive**
            - **99.99% availability**
            - 2 flexible retrieval:
                - Standard (12 hours)
                - Bulk (48 hours)
            - Minimum storage duration of **180 days**
            - Lowest cost
- **Intelligent Tiering**
    - **99.9% availability**
    - Moves objects automatically between Access Tiers based on usage
    - Small monthly monitoring and auto-tiering fee
    - **No retrieval charges**
- #### Moving between Storage Classes
    
    - In the diagram below, transition can only happen in the downward direction
    - ![attachments/Pasted image 20220514201314.jpg](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/attachments/Pasted%20image%2020220514201314.jpg)

### Lifecycle Rules

- **Used to automate transition or expiration actions on S3 objects**
- **Transition Action** (transitioned to another storage class)
- **Expiration Action** (delete objects after some time)
    - delete a version of an object
    - delete incomplete multi-part uploads
- Lifecycle Rules can be created for a prefix (ex `s3://mybucket/mp3/*`) or objects tags (ex Department: Finance)

> - When you apply a retention period to an object version explicitly, you specify a `Retain Until Date` for the object version
> - When you use bucket default settings, you don't specify a `Retain Until Date`. Instead, you specify a duration, for which every object version placed in the bucket should be protected.
> - Different versions of a single object can have different retention modes and periods

### Performance

- 3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD requests per second per prefix
- Recommended to **spread data across prefixes** for maximum performance
- SSE-KMS may create bottleneck in S3 performance
- Performance Optimizations
    - **Multi-part Upload**
        - parallelizes upload
        - **recommended for files > 100MB**
        - **must use for files > 5GB**
    - **Byte-range fetches**
        - Parallelize download requests by fetching specific byte ranges in each request
        - Better resilience in case of failures since we only need to refetch the failed byte range and not the whole file
    - **S3 Transfer Acceleration**
        - Speed up **upload and download** for **large objects (>1GB)** for **global users**
        - Data is ingested at the nearest edge location and is transferred over AWS private network (uses [CloudFront](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/CloudFront) internally)

### Data Transfer Costs
- **Uploads** to S3 are **free**, **Downloads** from S3 are **paid**
- Using **S3 Transfer Acceleration**, you pay only for transfers that are accelerated
- Since buckets are defined within a region, **data transfer within a region is free**

### S3 Notification Events
- Optional
- Generates events for operations performed on the bucket or objects
- Object name filtering using prefix and suffix matching
- **For the same combination of prefix and event type, we can only have one event rule.** Example: we can send S3 notification for object created at `/files` to only one destination (single rule).
- Targets
    - SNS topics
    - **SQS Standard** queues (not FIFO queues)
    - Lambda functions

### Requester Pays Buckets

- Requester pays the cost of the request and the data downloaded from the bucket. The bucket owner only pays for the storage.
- Used to share large datasets with other AWS accounts
- The requester must be authenticated in AWS (cannot be anonymous)

### Object Lock (feature)
- Can only be turned on during creation
- **WORM (Write Once Read Many)** model
- Block an object version modification or deletion for a specified amount of time
- Modes:
    - **Governance mode**
        - Only users with special permissions can overwrite or delete the object version or alter its lock settings
    - **Compliance mode**
        - A protected object version cannot be overwritten or deleted by any user, including the root user
        - The object's retention mode can’t be changed, and the retention period can’t be shortened

#### Glacier Vault Lock

- WORM (Write Once Read Many) model for Glacier
- For compliance and data retention

## EBS
Non global service, allows you to create "EBS volume" which are **network drive** you can attach to your instances while they run. **They can only be mounted to one instance at a time. EBS volume is not multi Availability Zone**.
	1. **EBS Snapshot Archive:** Move a Snapshot to an ”archive tier” that is 75% cheaper
	2. **Recycle Bin for EBS Snapshots**: Specify retention (from 1 day to 1 year)
### Volume Types

#### General Purpose SSD Volumes (gp2)
- **Use Case**:
    - Most general workloads
    - Transactional workloads
    - Virtual desktops
    - Medium-sized, single-instance databases
    - Low-latency interactive applications
    - Boot volumes
    - Development and test environments
- **Durability**: 99.8% - 99.9%
- **Volume Size**: 1 GB – 16 TB
- **Max IOPS**: 16,000 (16 KB I/O)
- **Max Throughput**: 250 MB/s
- **Price**: 0,10 USD per GB per month

**General Purpose SSD Volumes (gp3)**: 
like gp2 but:
- **Price**: 0,08 USD per GB month
- **Throughput:** 1000 MB
#### Provisioned IOPS SSD Volumes (io2 Block Express)
- **Use Case**:
    - Sub-millisecond latency
    - Sustained IOPS performance
- **Durability**: 99.999%
- **Volume Size**: 4 GB – 64 TB
- **Max IOPS**: 256,000 (16 KB I/O)
- **Max Throughput**: 4,000 MB/s
#### Throughput Optimized HDD (st1)
- **Use Case**:
    - Big data
    - Data warehouses
    - Log processing
- **Durability**: 99.8% - 99.9%
- **Volume Size**: 125 GB - 16 TB
- **Max IOPS**: 500 (1 MiB I/O)
- **Max Throughput**: 500 MB/s
#### Cold HDD (sc1)
- **Use Case**:
    - Throughput-oriented storage for data that is infrequently accessed
    - Where lowest storage cost is important
- **Durability**: 99.8% - 99.9%
- **Volume Size**: 125 GiB - 16 TiB
- **Max IOPS**: 250 (1 MiB I/O)
- **Max Throughput**: 250 MiB/s
#### Magnetic (standard)
- **Use Case**:
    - Workloads where data is infrequently accessed
- **Durability**: N/A
- **Volume Size**: 1 GiB - 1 TiB
- **Max IOPS**: 40-200
- **Max Throughput**: 40-90 MiB/s
---

### Snapshots
- It is a point-in-time copies of your EBS volume
- **Data Lifecycle Manager (DLM)** can be used to automate the creation, retention, and deletion of snapshots of EBS volumes
- Snapshots are incremental
- Only the most recent snapshot is required to restore the volume

### RAID
- **RAID 0**
    - Improve performance of a storage volume by distributing reads & writes in a stripe across attached volumes
    - If you add a storage volume, you get the straight addition of throughput and IOPS
    - For high performance applications
- **RAID 1**
    - Improve data availability by mirroring data in multiple volumes
    - For critical applications
## EFS
Managed NFS (network file system) that can be mounted on 100s of EC2.
- EFS works with **Linux** EC2 instances in multi-AZ 
- **Highly available, scalable, expensive** (3x gp2)
- AWS managed Network File System (NFS)
- Can be mounted to multiple EC2 instances **across AZs**
- **Pay per use** (no capacity provisioning)
- **Auto scaling** (up to PBs)
- Compatible with **Linux** based AMIs (**POSIX** file system)
- Uses **security group** to control access to EFS
- Lifecycle management feature to move files to **EFS-IA** after N days
- Creates multiple **mount targets**

#### EFS Client
- Is an open-source collection of Amazon EFS tools.
	- enables the ability to use Amazon CloudWatch to monitor an EFS file system's mount status
	- You need to install the **Amazon EFS client** on an Amazon EC2 instance prior to mounting an EFS file system

## Fsx
Allows you to deploy scale feature-rich, high performance file systems in the cloud. Fsx support a variety of file system protocol.

- Types:
	- **Amazon FSX for NetApp ONTAP**: Property enterprise storage platform known for handling petabytes of data
	- **Amazon Fsx for OpenZFS**: Open-source storage platform originally developer by Sun Microsystem
	- **Amazon FSX for Windows File Server (WFS)**: File storage on a Windows server supporting native window features for Windows developer
	- **Amazon FSX for Lustre**: Open-source file system for parallel computing (several processors work simultaneously)

- File Cache: high speed cache for datasets stored anywhere, accelerate bursting workloads
### FSx for Windows

- Shared File System for Windows (like EFS but for Windows)
- Supports **SMB** protocol, Windows **NTFS**, Microsoft **Active Directory** integration, ACLs, user quotas
- Built on **SSD**, scale up to 10s of GB/s, millions of IOPS, 100s PB of data
- Supports **Multi-AZ** (high availability)
- Data is backed-up daily to S3
- **Does not integrate with S3** (cannot store cold data)

## AWS BACKUP
Allows you to centrally manage backup accross AWS Services

- Component:
	- **Backup Plan**: A backup policy defines the backup schedule, backup window, backup lifecycle.
	- **Backup Vault**: backup are stored in a backup vault following WORM model
	- **AWS backup Audit Manager** is built in reporting and auditing for AWS backup

## Snow Family
AWS Snow Family are storage and compute devices used to phisically move data in or out the cloud **when moving data over the internet or private connection is too slow, difficult or costly.**
- **Takes around 2 weeks to transfer the data**
- **Snowball cannot import to Glacier directly** (transfer to S3, configure a lifecycle policy to transition the data into Glacier)
- Pay per data transfer job

![[Pasted image 20241002093152.png]]
- **Snowcone**: Portable secure device for edge computing
	- types:
		- **Snowcone**: 8 TB HDD
		- **Snowcone SSD**: 8 TB SSD
	- Two ways of sending data:
		- **Phisically shipping** the device which runs on the device's compute
		- **AWS DataSync** which runs on the device's compute
- **SnowballEdge**: Similar to Snowcone but with more local processing, edge computing workloads, and device configuration options.
	- Config Options:
		- **Storage Optimized**
			- 210 TB usable
		- **Storage Optimized (for data transfer):** 
			- 100 TB (80 TB usable)
		- **Storage Optimized with EC2-Compatible compute**
			- 80 TB usable storage , 40vCPUs and 80 GB of memory
		- **Compute Optimized**
			- Up to 104 vCPUs, 416 GB of memory,and 28 TB of dedicated NVMe SSd
		- **Compute Optimized with GPU**
			- Useful for video processing
- **AWS Snowmobile**: 45-foot ship container truck. Used to handle 100 PB of data. (not used anymore)
- Comparison
	![[Pasted image 20241002094449.png]]

## AWS transfer family
Offers fully managed support for the transfer files over SFTP, AS2,FTPS and FTP directly into and out of Amazon S3.

- Supported Protocols
    - **FTP** (File Transfer Protocol) - unencrypted network protocol
    - **FTPS** (File Transfer Protocol over SSL) - FTP with SSL/TLS encryption
    - **SFTP** (Secure File Transfer Protocol) -  uses SSH to provide secure but unencryped 
    - **AS2** (Applicability Statement): Enable secure and reliable messaging

**Transfer Family Managed File Transfer Workflows:**
Fully managed serverless File Transfer Workflow service to set up, run, automate, and monitor processing of files uploaded using AWS Transfer Family.

Workflow allow you to: copy, tag, delete, decrypt file
## AWS Migration Hub
Is a single place to discover your existing servers, plan migrations, and track the status of each application migration.

AWS Migration hub can monitor migration using:
- **Application Migration Service (AMS)**
- **Database Migration Service (DMS)**

Tools to migrate:
- **AWS Discovery Agent:** Agent installed on your VM
- **Migration Evaluator Collector** : You submit a request to ask helps from AWS team
Other tools:
- AWS Migration Hub Refactor: Bridges networking accross AWS Accounts so that legacy and new services can communicate while they maintain the independence of separate accounts
- AWS Migration Hub Journey: guided templates
## AWS Data Sync
 Move **large amounts of data** from your **on-premises NAS or file system** via **NFS** or **SMB** protocol to AWS over the **public internet using TLS**.
 - Need to install **AWS DataSync Agent** on premises

For Use cases check: https://docs.aws.amazon.com/datasync/latest/userguide/what-is-datasync.html
## Data Migration Service (DMS)
**AWS Database Migration Service (AWS DMS)** is a managed migration and replication service that helps move your database and analytics workloads to AWS quickly, securely, and with minimal downtime and zero data loss.
![[Pasted image 20241002101336.png]]
**AWS Schema Conversion** is used in many cases to automatically convert a source database schema to a target database schema
For data warehouse you can use the desktop app **AWS schema conversion tool** 

### Migration Methods
- **Homogenous data migration**
	- Migrate data with native database tools. **Eg. pg_dump, pg_restore**
	- You create a migration project in DMS and it will perform the migration using a serverless compute
		- Uses a **pay as you go** model
- **Instance Replication**
	- Provision an instance with chosen instance type to perform the replications between databases
- **Serverless Replication (DMS Serverless)**
	- a **serverless** offering where you pay as you go, **with some limitations**:
	- Does not have public Ips
	- Must use VPC endpoints to access specific AWS services. Eg. S3, Kinesis, DynamoDB, OpenSearch
	- Limited selection of possible sources and targets
	- Doesn't support views with selection and transformation rules.

## Auto Scaling (different from ASG)
**AWS auto scaling** is a service that can discover scaling resources whithin your aws account, and quickly add scaling plans to your scaling resources.

It can manage and make reccomandations for the following scaling resources: ASG, ECS, EC2,Aurora, DynamoDB, Spot Fleet

- **Predictive Scaling**: analyze historical load, generate a forecast and scale based on that forecast
- **Dynamic Scaling**: when a metric changes so does the capacity.

## Storage Gateway
**AWS Storage Gateway**: is a hybrid storage service that allows your on-premises applications to seamlessly use AWS cloud storage (Bridge between on-premise data and cloud data in s3). Use example: backups to the cloud, using on-premises file shares backed by cloud storage, and providing low-latency access to data in AWS for on-premises applications.

### Types of Storage

#### S3 File Gateway
 Amazon S3 File Gateway supports a file interface into s3 and combines a service and a virtual software appliance. By using this combination, you can store and retrieve objects in Amazon S3 using industry-standard file protocols such as **Network File System (NFS)** and **Server Message Block (SMB)**.
- **Data is cached at the file gateway** for low latency access
-  Integrated with **Active Directory (AD)** for user authentication
![[Pasted image 20241007125534.png]]
#### Volume Gateway
Allows you to mount s3 as local drive using the **iSCSI** protocol
- 2 types: 
		- **Store Volumes:** store primary data locally and asynchronously to AWS:
			- Provide your on-premises applications with **low-latency access** to their entire datasets while still providing durable off-site backups.
			- **Create storage volumes** and mount them as iSCSI devices from your on-premises servers.
			- Any data written to stored volumes are stored on your on-premises storage hardware.
			- Amazon Elastic Block Store (EBS) snapshots are backed up to AWS S3.
			- **Stored Volumes can be between 1GB - 16TB in size**
		- **Cache volumes:** 
			- **Minimizes the need to scale your on-premises storage infrastructure** while still providing your applications with low-latency data access.
			- **Create storage volumes up to 32TB in size and attach them as iSCSI** devices from your on-premises servers.
			- Your gateway stores data that you write to these volumes in S3 and retains recently read data in your on-premises storage gateway cache, and upload buffer storage.
			- Cached volumes can be between 1GB - 32GB in size

Think of it as a remote drive

![[Pasted image 20241007125555.png]]
#### Tape Gateway
Stores files onto **Virtual Library Tapes** for backing up you files on very cost effective long term

**Durable**, cost effective solution to archive your data in thw AWS CLoud. Use **VTL** interface, that helps you leverage existing tape-based backups. Store data on virtual tape that you create on you tape gateway. Tape storage has proven redability **for 30 years**
#### Amazon FsX File Gateway
Allows your files to be stored in Amazon FSx Windows FIle Storage (WFS). Allow your Windoes developers to easily store data in the cloud using the tools they already know

## Instance Store
- Used as **cache**
- **Hardware storage directly attached to EC2 instance** (cannot be detached and attached to another instance)
- **Highest IOPS** of any available storage (millions of IOPS)
- **Ephemeral storage** (loses data when the instance is stopped, **hibernated** or terminated)
- **Good for buffer / cache / scratch data / temporary content**
- AMI created from an instance does not have its instance store volume preserved

> You can specify the instance store volumes only when you launch an instance. You can’t attach instance store volumes to an instance after you’ve launched it.

### RAID

- **RAID 0**
    - **Improve performance** of a storage volume by distributing reads & writes in a stripe across attached volumes
    - If you add a storage volume, you get the straight addition of throughput and IOPS
    - For high performance applications
- **RAID 1**
    - **Improve data availability by mirroring data** in multiple volumes
    - For critical applications
## Amazon AppFlow
Is managed integration service for data transfer between data sources. Easily exchange data with over 80+ cloud services. By specifing a source destination. **(Easy way to connect services)**

![[Pasted image 20241007142942.png]]
- **Run on Demand** - User manually run the flow as needed
- **Run on event** - AppFlow runs the flow in response to an event from an SaaS application
- **Run on Schedule** - AppFlow runs te flow on a recurring schedule

**Feature:**
- Create dataflows between application whithin minutes
- Aggregate data from multiple sources
- Data can be encrypted at rest and in transit
- Use partition and aggregation settings to optimize query performance 
- Develop custom connectors via the Amazon AppFlow Custom Connector SDKs
- You can create private flow via AWS Private Link
- You can catalog data transfered to s3 via AWS Glue Data Catalog


# Computing
## EC2
Amazon Elastic Compute Cloud (Amazon EC2) provides on-demand, scalable computing capacity in the Amazon Web Services (AWS) Cloud. Using Amazon EC2 reduces hardware costs so you can develop and deploy applications faster.

- **EC2 User data**: Used to automate **dynamic** boot tasks (that cannot be done using AMIs). **Runs with root privileges**
-  **EC2 Placement Group:** the logical placement of your instances to optimize communication, performance, or durability. Free
	- Cluster
	- Partition
	- Availability Zone
- **EC2 Meta Data**: You can access to Ec2 Metadata from **MDS(Meta data service)** a special endpoint. There are 2 versions of MDS v1 and v2. Should be used v2.
- **EC2 Naming convention:**
	![[Pasted image 20241001121648.png]]
- **Instance Family:** Are different combination of CPU,Memory,Storage and Networking capacity
- **EC2 Instance Profile**: is a reference to an IAM role that will be passed and assumed by the EC2 instance when it starts up. Allows you to avoid passing long live AWS credentials.
- **EC2 Lifecycle:** 
	![[Pasted image 20241009172903.png]]
- **EC2 Instance Console Screenshot**: will take a screenshot of the current state of the instance
- **Hostname**: unique name in your network to identify a machine via DNS.
	- two types:
		- IP Name: legacy name based on private access
		- Resource Name: EC2 instance ID is included in the hostname
- **EC2 Default user name:** The default user name for an operating system managed by AWS will vary based on distribution:
	- When using SSM you will need to change your user to default
	- ```sudo su -ec2-user```
-  **EC2 Bustable Instances:** allow workloads to handle bursts of higher CPU utilization fro very short duration
- **EC2 system log:** Accessed by the EC2 Management Console, allows you to observe the system log for the EC2 Instance. Troubleshoot on boot to see if anything is wrong
- **EC2 Connect:**
	- SSh Client
	- EC2 instance connect: short lived ssh keys controlled by IAM policies
	- Session Manager: Connect to linux windoes via reverse connection
	- Fleet Manager Remote Desktop: Connect to a Windows Machine using RDP
	- EC2 Serial console: establishes a serial connection giving you direc access
- **EC2 Amazon Linux:** AWS's managed Linux distribution is based off CentOS and Fedora which in turn is based off Red Hat Linux. Reccomended to use Amazon Linux AL2023 e non AL2
	- Amazon Linux extra: is a feature of AL2 that provides a way for users to install additional software packages.
### Type of instances

![[Pasted image 20241001122001.png]]
N.B these are not all but only the most important

### Intsance Classes
 Type of purchasing option:
- **On-Demand Instances**: Has **high cost but no upfront payment** is a **pay-as-you-go** model. This is the default option when create EC2. Recommended for **short-term and un-interrupted** workloads commitment
- **Reserved Instance(1 & 3 years)**: up to 72% save, long workloads/ long workloads with flexible instances. 
	- Types:
		- **Standard**: These provide the most significant discount, but can only be modified. Standard Reserved Instances can't be exchanged.
		- **Convertible**: These provide a lower discount than Standard Reserved Instances, but can be exchanged for another Convertible Reserved Instance with different instance attributes. Convertible Reserved Instances can also be modified.
	- Type of payments: **All upfront, Partial upfront, No upfront**
	- **RI Attributes**: can affect cost (Instance type, Region, Tenancy, Platform)
	- **Limits**: Per month -> 20 Regional RI per region, 20, Zonal RI per zone
	- **RI Marketplace**: allows you to sell your unused Standard RI to recup your RI spend for RI you do not intend or cannot use.
- **Savings Plans (1 & 3 years)** :commitment to an amount of usage, long workload, up to 72%
- **Spot Instances** : short workloads, cheap, can lose instances (less reliable), up to 90%
- **Dedicated Hosts** and Dedicated Instance:
	 ![[Pasted image 20241004112027.png]]

**Capacity Reservations** : Is a service of EC2 that allows you to request a reserve of EC2 instance type for a specific Region and AZ

### AMI
- Provides the information required to launch an instance. You can turn your EC2 instances into AMI's so **you can create copies of your servers**.
	- Help you keep incremental changes to your OS, application code, and system packages.
	- AMI Id is different per region. You need to be carefull when you buy one
	- AMI has two types of boot model:
		- Unified Extensible Firmware Interface (UEFI): Modern
		- BIOS: Legacy (to avoid)
	- **Elastic Network Adapter** supports network speeds of up to 100 GBps for supported instance types.
	- AMI Settings:
		- Public:
		- Explicit: Specific to certain AWS accounts
		- Implicit: The owner can launch the AMI
## ASG(Auto Scaling Group)
Creation of instances based on amount of workload. **High Availability
![[Pasted image 20241007144844.png]]
- Automatic scaling can occur via different method:
- **Capacity Settings:** The size of an Auto Scaling group based on **Min Size, Max Size, Desired Capacity**
- **Health Check Replacement:** ASG replaces an instances if is considered unhealty. Two types of health checks ASG can perform
	- **EC2 Health Check**: If the EC2 instance fails either of its EC2 Status Check
	- **ELB Health Check**: ASG will perform a health check based on the ELB health check. ELB pings an HTTP endpoint at a specific path, port and status code
- **Dynamic Scaling Policies**: how much ASG should change capacity. Policies are triggered based on **Cloud Watch Alarms**:
	- **Adjustment types**: how capacity should change
		- ChangeInCapacity
		- ExactCapacity
		- PercentChangeInCapacity
	- **Scaling Policies:**
		- **Simple Scaling**: When a **CloudWatch alarm** is triggered(example CPU > 70%), then add 2 units or/and (example CPU < 30%), then remove 1
		- **Step Scaling:** Also scale when CloudWatch alarm are trigger but more granular control over how the ASG. You can define multiple **steps** in the scaling policy, where each step specifies different scaling actions based on specific ranges of the metric.
		- **Target Tracking Scaling:** **Example:** I want the average ASG CPU to stay at around 40%
		- **Predictive Scaling**: Triggers scaling by analyzing historical load data to detect daily or weekly patterns in traffic flows

***Extra:*** 
ELB integration: An ELB can be attached to you Auto Scaling Group 
	- Classic Load Balancers are associeted directly to the ASG
	- ALB, NLB or GWLB are associeted indirectly via their Target Group
	 **N.B attached = can use Load Balancer Health check**
## ELB
It is a suite of Load balancer from AWS. A **Load balancer** is a tool to distribute traffic through different servers.

- Rules of traffic
	- Structure:
		- **Listeners**: incoming traffic is evaluetad against listeners. check matches with the Port (ex port 443 or port 80)
		- **Rules (only ALB):** Listeners will then invoke rules to decide what to do with the traffic. Generally, the next step is to forward traffic to a Target Group
		- **Target Groups(not CLB):** Are logical grouping of possible targets such as specific EC2 instances, IP addresses
		- **only for Classic Load Balancer traffic** is sent to the Listeners. When the port matches, it forwards the traffic to any EC2 instances that are registered to che Classic Load Balancer. CLB does not allow you to apply rules to listeners.
### Application Load Balancer:
- ALB is designed to balance HTTP and HTTPS traffic.
- It operate at **Layer 7 (OSI Model)**
- It allows you to add routing rules to your listeners based on the HTTP Protocol
- Supports WebSockets and HTTP for real time, bi-directional communications
- **Security Groups can be attached to ALBs** to filters requests
- **ACM** can be attached to listeners to server custom domains over **SSL/TLS** for HTTPS
- **Use Cases:**
	- Microservices and Containerized Applications
	- E-commerce and Retail Websites
	- Corporate Websites and Web Applications
	- SaaS Applications
### Network Load Balancer:
- NLB is designed to balance TCP/UDP.
- It operate at **Layer 4** (OSI Model)
- It can handle millions of requests per second while still maintaining extremely low latency.
- **Global Accelerator** can be placed in front of ALB to improve **global** availability
- Preserves the client source IP
- When a static IP address is needed for a load balancer
- **Use cases:**
	- High-Performance Computing and Big Data Applications
	- Real-Time and Multiplayer Gaming Platforms
	- Financial Trading Platforms
	- IoT and Smart Device Ecosystems
	- Telecommunications Networks

### Classic Load Balancer:
- CLB is AWS's first load balancer (legacy)
- Can balance HTTP, HTTPS or TCP traffic (not at the same time)

**Note**: Is not reccomended anymore
## AWS Lambda
**Serverless** service that let you run your code on a high-availability compute infrastructure and performs all of the administration of the compute resources.

**Pay per execution time and number. No charge when your code is not running.**

Example of use:
![[Pasted image 20241002122149.png]]
2 ways to be called:
- Sync invocation
- Async invocation

### Limit:
#### Time settings:
- Default: 3 seconds
- Min: 1 second
- Max 15 minutes
#### Storage settings:
- Default: 512 MB
- Min: 512 MB
- Max 10,240 MB (~10 GB)
#### Memory settings: (Increments of 1 MB)
- Default: 128 MB
- Min: 128 B
- Max 10,240 MB (~10 GB)

- **Function Version**: You can use the version to manage the deployment of your AWS Lambda Function
- 2 ways to reference a Lambda
	- Qualified ARN (specified version)
	- Unqualified ARN (non specified version take always the last)
- **Aliasis**: provide name to reference Lambda function
- **Lambda Layer**: Pull in additional code and content in the form of layer. Layer is a zip archive that **contains libraries, a custom runtime, or other dependencies**. You can use libraries in your function without needing to include them in your deployment package. 
- **Runtimes**: preconfigure environment to run specific programming languages. It doesn' require you to configure or set OS.
	- they are published continuesly, so you had to keep it updated and not retain older version
	- Code is delivered as a zip archive

N.B to improve performance of lambda function use SnapStart.
## Step functions
With AWS Step Functions, you can create workflows, also called [State machines](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-statemachines.html), to build distributed applications, automate processes, orchestrate microservices, and create data and machine learning pipelines.
**State machines (think of it as flow chart)**

- 2 types State Machines:
	- **Standard**: general purpose -> reccomended for Long Workload
	- **Express**: for sreaming data -> reccomended Short Workload, (streaming, data ingection, even driven ecc...)
- Some use cases:
	- Manage a Batch job , if job fails send message with SNS
	- Manage a Fargate Container
	- Transfer Data Records
	- **Other use cases can be found on the official web page**
- **Step Functions states**: Allows us to pass input and output without any work. Useful when constructing state machines
	- Pass State: passes its input and output, without performing work (dummy/mock). Pass states are useful when constructing and debugging state machines.
	- Task State: represent a single unit of work performed by state machine
		- Enables you to have a task in your state machine where the work is performed by a worker that can be hosted on anywhere eg: EC2, ECS, mobile phones
	- Wait State: delays state machine from coninuing for a specificng time
	- Succed State: stop an execution succesfully
	- Fail State: stops eh executiion of a state machine and set failure
	- Parallel States: can be used to create parallel branches of execution in your state machine The state machine does not move forward until both states complete

## AWS Compute Optimizer
AWS Compute Optimizer is a service that analyzes your AWS resources' configuration and utilization metrics to provide you with rightsizing recommendations. 
It reports whether your resources are optimal, and generates optimization recommendations to reduce the cost and improve the performance of your workloads.
## AWS Batch
 AWS Batch helps you to run batch computing workloads on the AWS Cloud. Batch computing is a common way for developers, scientists, and engineers to access large amounts of compute resources. 
 - compared to Lambdas they have **no time limit**
 - Batch jobs are defined as Docker images and run on ECS 
 - **Rely on EBS and EC2 / instance store for disk space**.


# Network & Security
**Elastic Network Interfaces (ENIs):** Are **virtual network interfaces** that provide networking capabilities to Amazon EC2 instances. An **ENI** functions as a virtual network card, enabling instances to connect to networks and communicate with other resources within the AWS environment.
## AWS VPC
With Amazon Virtual Private Cloud (Amazon VPC), you can launch AWS resources in a logically isolated virtual network. They are Region Specific

- Max 5 VPC per region. Max 200 subnet per VPC
- AWS has **default VPC per region** 
- **NACL**: **Network access control lists** is a firewall which controls traffic **from and to subnet**. It is free: 
	- Can have **ALLOW and DENY** rules 
	- Are **attached at the Subnet level** 
	- Rules only include IP addresses. 
- **Security groups:** Rules that controls traffic to and from an **EC2 Instance** 
	- Can have only **ALLOW** rules. 
	- Limits: 10000 security groups per region, 60 in/outbound rules per security group, 16 security group per Elastic Network Interface.
- **NACL vs Security group:**

| Security Group                                                                                                                                               | Network ACL                                                                                                                                           |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Operates at the instance level                                                                                                                               | Operates at the subnet level                                                                                                                          |
| Supports allow rules only                                                                                                                                    | Supports allow rules and deny rules                                                                                                                   |
| Is **stateful**: Return traffic is automatically allowed, regardless of any rules                                                                            | Is **stateless**: Return traffic must be explicitly allowed by rules                                                                                  |
| We evaluate all rules before deciding whether to allow traffic                                                                                               | We process rules in number order when deciding whether to allow traffic                                                                               |
| Applies to an instance only if someone specifies the security group when launching the instance, or associates the security group with the instance later on | Automatically applies to all instances in the subnets it's associated with (therefore, you don't have to rely on users to specify the security group) |

- **Route tables:** where network traffic are directed.
	- different types:
		- Main route tables: created with VPC cannot be deleted
		- Custom route table: toute table that you can create
- You can share a VPC trough the same account using **AWS Resources Acess Manager**
### Internet Gateway 
Used to **connect public resources to the internet** (use NAT gateway for private resources since they need network address translation)
- Work both with IPV4 and IPV6, and perform network translation for instances that have been assigned public IPV4 addresses.
#### Egress-only (EO-IGW)
Specifically for IPV6 when you want to allow outbound traffic to the internet but prevent inbound from the internet:
### Elastic IP
Addresses in AWS that always stay the same **(different from that of the EC2 instances)**
	- IPV6 are already unique so they don't need it
	- **are charge 1$ for each allocated**
	- You can associate, allocate, disassociate, deallocate, reassociate, recover, customize (with your IPV6 ip)
- **AWS ipv6 support**: provide a solution for the eventual exhaustion of all IPV4 address
- Is it possible to migrate from ipv4 and ipv6 following some rules. (It isn't always possible).

### VPC Peering
Connect two VPC, privately using AWS’ network 
- VPC peering can be between IPv4 or IPv6
- Make them behave as if they were in the same network 
- VPC Peering connection **is not transitive** (2 way)
- Must have **non-overlapping CIDR**
- Data transfer cross AZ or cross Region incurs charges
### VPC Endpoints: (vedi appunti AWS)
Endpoints allow you to connect to AWS Services **using a private network** instead of the public network 
- This gives you enhanced security and lower latency to access AWS services.
- Do not require public IPV4 and doesn't leave the AWS network
- Eliminates the need for Internet Gateway, NAT Device, VPN connection, AWS Direct
- **Bound to a region**
![[Pasted image 20241007170821.png]]
#### Type of VPC Endpoint
##### Interface Endpoint
 It is a Elastic Network Interfaces (ENI) with a private IP addess. they serve as an entry point for traffic going to a supported service. It is used by **AWS PrivateLink** **(Does not include S3 and DynamoDB)**
- **Use Case**: Private connection to AWS services, partner services, and other VPCs without public IPs.
- **Service Integration**: AWS PrivateLink
- **Supported Services**: Many AWS Services
- **Pricing**: 
  - Per hour when provisioned
  - Data processed
- **Routing Mechanism**: DNS interception and routing
- **Traffic Direction**: Bidirectional

##### Gateway Endpoint
Like interface endpoint used by S3 and DynamoDB 
- **Use Case**: Private connections to S3 and DynamoDB from VPC
- **Supported Services**: S3 and DynamoDB 
- **Pricing**: Free
- **Routing Mechanism**: Route table entries for specific destinations
- **Traffic Direction**: Unidirectional

##### GWLB Endpoint
Powered by PrivateLink allows you to distribute traffic  to a fleet ofnetwork virtual appliances
- **Use Case**: Route traffic to third-party virtual appliances like firewalls in another VPC.
- **Supported Services**: Third-party virtual applications
- **Pricing**: 
  - Endpoint hours
  - Data processed
- **Routing Mechanism**: Integrates with GWLB
- **Traffic Direction**: Usually Unidirectional
### AWS Direct Connect
	![[Pasted image 20241014175942.png]]
- 2 types:
	- **Lower Bandwith** 50Mbps -500Mbps
	- **Higher Bandwith** 1Gbps, 10Gbps, 100Gbps
- Your network is co-located with an existing AWS Direct Connect location
- You work with a AWS Partner Network member
- Support IPV4 and IPv6 
-  **Data in transit is not-encrypted** but the connection is private (secure)
- Time to setup > 1 month
#### Connection Types
- **Dedicated Connection**
    - **1 Gbps and 10 Gbps** (fixed capacity)
    - Physical ethernet port dedicated to a customer
- **Hosted Connection**
    - **50Mbps, 500 Mbps, up to 10 Gbps**
    - **On-demand capacity scaling** (more flexible than dedicated connection)
#### Pricing
- **Capacity**
- **Port hours:**
	- Dedicated: Billed per hour
	- Hosted Billed subject to the AWS direct connect Delivery partner
- **DTO (Data Transfer Out)**
	- Charged based on outbound traffic sent through Direct Connect to destinations outside of AWS
	- Data transfer in the same region is free
### PrivateLink
AWS PrivateLink establishes private connectivity between virtual private clouds (VPC) and supported AWS services, services hosted by other AWS accounts, and supported AWS Marketplace services. Need **network load balancer** for other VPC and **Elastic Network Interface** for yours

![[Pasted image 20241001095444.png]]
**N.B:** Need an Interface endpoint and Service endpoint in order to work
### AWS Site-to-site VPN:
![[Pasted image 20241001102606.png]]

**Components:**
- **VPN Connection** - secure connection between VPC and on-premises equipment
- **VPN tunnel** - encrypted connection for your data
- **Customer gateway (CGW)**-provides information to AWS about your customer gateway device
- **Customer gateway device** - A physical device or software application on your side of the Site-to-Site VPN connection.
- **Target gateway** - A generic term for the VPN endpoint on the Amazon side of the Site-to-Site VPN connection.
- **Transit gateway or Virtual private Gateway**
	- **Virtual private gateway (VGW)** -VPN endpoint on the Amazon side of your Site-to-Site VPN connection that can be attached to a single VPC
	- **Transit gateway** - A transit hub that can be used to interconnect multiple VPCS and on-premises networks.

**Features:**
- CloudWatch metrics
- Reusable IP addresses for your customer gateways
- Additional encryption options
	- AES 256-bit encryption
	- SHA-2 hashing,
	- Additional Diffie-Hellman groups
	- Configurable tunnel options

**Limitations:**
- IPv6 traffic is not supported for VPN connections on a virtual private gateway.
- Recommend that you use non-overlapping CIDR blocks for your networks

**Pricing:**
- Each VPN connection hour
- Data transfer out from Amazon EC2 to the internet.
### AWS Client VPN: 
Connect from **your computer** using OpenVPN to your VPC in AWS and on-premises system.
**Use case**: You travel around the world and need connection to services.
![[Pasted image 20241007172433.png]]
- Certificate based authentication (Mutual authentication)
- Active Directory authentication (AWS Directory Service)
- Federation authentication (Single-Sign-On SAML)
- Uses a single tunnel
- Use Security Groups for granular control
- Use AD Groups for granular control
- Self-service portal to download AWS VPN Desktop Client

### NAT Gateway: 
**What is Network Address Translation (NAT)?**
A method of mapping an IP address space into another by modifying network address information in the IP header of packets while they are in transit across a traffic-routing device

**NAT Gateway(Network Address Translation)**: Allow your instances in your Private Subnets to access the internet while remaining private

![[Pasted image 20241001104657.png]]
- 1 Nat gateway per subnet
- Pricing
	- Per hour 0.045
	- Per GB 0.045
- **2 types**: Public and Private
- **NAT Instance (not very used anymore):** is an AWS managed IAM to launch a NAT onto an individual EC2 instances. NAT Instances required the customer to handle scaling

### Transit Gateway: 
[AWS Transit Gateway](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html) provides a hub and spoke design for connecting VPCs and on-premises networks as a fully managed service without requiring you to provision third-party virtual appliances. No VPN overlay is required, and AWS manages high availability and scalability.

- Transit Gateway leverages AWS Resource Manager (RAM)
- **Operates at the regional level.**
- You can attach up to 5000 VPCs to each gateway
	- An ENI will be provisioned in target VPCs to facilitate VPC to Transit Gateway communication
- Each attachment can handle up to 50 Gbits/second of trafficù
- Support IP multicast
- You can peer connection with other Transit Gateways

	![[Pasted image 20241010120643.png]]
### Network Address Usage(NAU)
Is a metric applied in your VPC to help you plan for and monitor the size of your VPC, in a way that you don't run out of space for your VPC.
## Route 53
Amazon Route 53 is a highly available and scalable Domain Name System (DNS) web service. You can use Route 53 to perform three main functions in any combination: **domain registration, DNS routing, and health checking.**

- **Hosted Zones**: Container for records sets, scoped to route traffic for a specific domain or subdomains
	- **Public**: How you want to route traffic through an Internet
	- **Private**: How you want to route traffic through an Amazon VPC
- **Records set**: collection of records which determine where to send traffic
	- Always changed using batch via the API
	- **Record Alias:** specific DNS record of AWS which extends DNS functionality. It rwill route traffic to specific AWS resources.
	- **Alias Target** can point to:
		- CloudFront
		- Elastic Beanstalk environment
		- ELB load balancer
		- S3 website endpoint
		- Resource record set
		- VPC endpoint
		- API Gateway Endpoint custom regional API
- **Traffic Flow**: Visual tool, lets you create soshisticated routing configurations for your resources using routing types (50$ per policy -> very expensive)
### Routing Policies:
- **Simple Routing Policies**: Most basic Routing policies
	- You have 1 record and provide multiple IP addresses
	- When multiple values are specified for a record, Route53 will return values back to the user in a random order
	- No health check
	- ***Example***: if you had a record for www.exampro.co with 3 different IP addess values, users would be directly randomly to 1 of them
- **Weighted Routing Policies**: Let you split up traffic based on different weight assigned
	- This allows you to send a certain percentage of overall traffic to one server and have other traffic to be directed to a completeley different server
	- Health check
- **Latency Based Routing Policies:** allows you to direct traffic based on the lowest network latency possible for your end-user based on region
	- Can be used for **Active-Active failover** strategy (two resources active at the same time)
- **Failover Routing Policy Policies:** allow you to create **active passive/setup** in situations where you want a primary site in one location and a secondary data recovery. **Active-passive** means that one is actively used and the other in stanby until the first fail.
	- Automatically monitors health checks from your primary site
- **Geolocation Routing Policies**: allow you to direct traffic based on the geographic location of where **the request originated from**
- **Geoproximity routing polices**: allow you to direct traffic based on the geographic location of **your users and your AWS resources**
- **Multi-Value Answer Policies:** let you configure Route 53 to return multiple values such as IP addresses for your web servers, in response to DNS queries.

### Health Checks (not free): 
- HTTP Health Checks are only for public resources
- Checks health every **30s** by default. Can be reduced to every **10s**
- A health check can **initiate a failover** if the status is returned unhealthy
- A CloudWatch Alarm can be created to alert you of status unhealthy
- A health check can monitor other health checks to create a chain of reactions.
- Can create up to **50 health checks** for AWS endpoints within or linked to the same AWS account.
### Other Feature and tools
- **Route 53 Resolver:** is a DNS server that allows you to resolve DNS queries between your on-premise network and your VPC
	![[Pasted image 20241008092916.png]]
- **DNSSEC**: Are a suite of extensions specifications by the Internet Engineering Task Force (IETF) for securing data exchanged in the Domain Name System (DNS) in Internet Protocol (IP) networks.
- **Zonal Shift**: is a capability in **Route 53 Application Recovery Controller** (Route 53 ARC). Shift a load balancer resource away from an impaired Availability Zone with a single action. Only supported by ALB an NLB
- **Route 53 Profiles**: lets you apply and manage DNS related Route 53 configurations accross many VPCs and in differente AWS accounts

## AWS Global Accelerator

Improve global application **availability** and performance using the AWS global network (Can find the optimal path from the end user to your web servers) 
- Leverage the AWS internal network to optimize the route to your application 
-  **2 Static Anycast IP** are created for your application and traffic is sent through **Edge Locations
- 2 types Routing: 
	- **Standard**: automatically route to the nearest healthy endpoint
	- **Custom**: Route specifies EC2 Instances
- Components: 
	- **Listeners**: Listens for traffic on a specific port and sends traffic to a endpoint group
	- **Endpoint Groups**: collection of endpoint
	- **Endpoints**: represent a resource to send traffic to
#### Unicast vs Anycast IP
- **Unicast IP**: one server holds one IP address
- **Anycast IP**: all servers hold the same IP address and the client is routed to the nearest host with that IP
## Cloud Front
![[Pasted image 20241009101827.png]]
- Cloud Front is a **CDN**
	- **CDN(Content Delivery Network):** A distributed network of servers that delivers web pages and content to users based on their geographical location, the origin of the webpage and a content delivery server.


- **Global service**
- **Edge Locations are present outside the VPC** so the origin's SG must be configured to allow inbound requests from the list of public IPs of all the edge locations.
- Supports HTTP/RTMP protocol (**does not support UDP protocol**)
- **Caches content at edge locations**, reducing load at the origin
- **Geo Restriction feature**
- Improves performance for both cacheable content (such as images and videos) and dynamic content (such as API acceleration and dynamic site delivery)
- To block a specific IP at the CloudFront level, deploy a [WAF](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/Web%20Application%20Firewall%20(WAF)) on CloudFront
- Supports **Server Name Indication (SNI)** to allow SSL traffic to multiple domains
### Origin

- **[S3](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/Simple%20Storage%20Service%20(S3)) Bucket**
    - For distributing static files
    - **Origin Access Identity (OAl) or Origin Access Control (OAC)** allows the S3 bucket to only be accessed by CloudFront
    - Can be used as ingress to upload files to S3
- **Custom Origin** (for HTTP) - need to be publicly accessible on HTTP by public IPs of edge locations
    - EC2 Instance
    - ELB
    - S3 Website (may contain client-side script)
    - On-premise backend

> To restrict access to ELB directly when it is being used as the origin in a CloudFront distribution, create a VPC Security Group for the ELB and use AWS Lambda to automatically update the CloudFront internal service IP addresses when they change.

## Pricing

- Price Class All: all regions (best performance)
- Price Class 200: most regions (excludes the most expensive regions)
- Price Class 100: only the least expensive regions
### Origin Groups

- Consists of a **primary** and a **secondary** origin (can be in **different regions**)
- Automatic failover to secondary
    ![attachments/Pasted image 20220508161659.jpg](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/attachments/Pasted%20image%2020220508161659.jpg)
- Provides **region-level** High Availability
- Use when getting 504 (gateway timeout) Error
## Field-level Encryption

- Sensitive information sent by the user is **encrypted at the edge** close to user which can only be decrypted by the web server (intermediate services can't see the encrypted fields)
- **Asymmetric Encryption** (public & private key)
- Max 10 encrypted field
### CloudFront Functions
Lightweight edge functions for high-scale, latency-sensitive CDN customizations. CloudFront Functions are cheaper, faster, but more limited than Lambda edge functions.

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
		- 24/7 access to AWS DDoS response team (DRP) 
		- Protect against higher fees during usage spikes due to DDoS
## WAF
Protects your web applications from common web exploits **Layer 7**.
**ALLOW and DENY** rules based on the contents of HTTP request. 
Can only be deployed on
- Application Load Balancer
- API Gateway
- CloudFront
WAF contains **Web ACL (Access Control List)** containing rules to **filter requests** based on:
- **IP addresses**
- HTTP headers
- HTTP body
- URI strings
- Size constraints (ex. max 5kb)
- **Geo-match** (block countries)
- **Rate-based rules** (to count occurrences of events per IP) for **DDoS protection**
## Cloud HSM
**HSM:**
Hardware Security Module it is a piece of hardware designed to store encryption keys. HSM hold keys in memory and never write them to disk.
It follows FIPS (Federal Information Processing Standards): US and Canadian governament standard for cryptographic modules.

**2 types:**
- **Multi-tenant** (FIPS 140-2 Level 2 Compliant): 
- **Single-Tenant** (FIPS 140-2 Level 3 Compliant):

**Cloud HSM:** 
Is **single tenant** HSM as a service that automates hardware provisioning. Enables you to generate and use your encryption keys on a FIPS 140-2 Level 3 validated hardware.

It is easy to migrate aws key between cloudHSM.

## Amazon GuardDuty
Intrusion Detection System and Intrusion Protection System.(IDS)

Intelligent **Threat discovery** to protect your AWS Account. Uses Machine Learning algorithms, **anomaly detection**, 3rd party data. **Uses EventBridge and lambda funtion to make action**. Check for **(CloudTrail logs, VPC flow logs, DNS logs)**
## AWS Firewall Manager
 AWS Firewall Manager allows  you to centrally configure and manage firewall rules accross accounts and applications. 
 - Must be a member of AWS organizzation
 - Must be AWS Firewall Manager administrator
 - Must have AWS Config enabled for your accounts and region
## AWS Inspector
Amazon Inspector is an **automated** security assessment service that helps improve the security and compliance of applications deployed **on your Amazon EC2 instances**. Amazon Inspector automatically assesses applications for exposure.
## Amazon Macie
Amazon Macie is a fully managed data security and data privacy service that uses machine learning and pattern matching to discover and protect your sensitive data in AWS. Work alongside CloudTrail
## AWS Security Hub
Central security tool to manage security across several AWS accounts and **automate** security checks. Allow you to generate a security score to determine your security posture. Allows you to enable standards(collection of security control)ù

## KMS
KMS makes it easy for you to create, control and rotate encryption keys in AWS.
- **Regional service (keys are bound to a region)**
- Is **multi-tenant**: There are multiple customers that are using the same piece of hardware. In this way is it possible to reduce the cost of the service. Each part is isolated per user.
- It is used for the encryption of almost every service in aws.
-  **Encrypt up to 4KB of data per call (if data > 4 KB, use envelope encryption)**
#### Customer Master Key (CMK)
**Symmetric keys**
- **Type of key**
	- **Customer Managed Key:**
		- Create, manage and used by the customer, can enable or disable
		- Possibility of rotation policy (new key generated every year, old key preserved)
		- Possibility to bring-your-own-key
	- **AWS Managed Key:**
		- Created, managed and used on the customer’s behalf by AWS
		- Used by AWS services **(aws/s3, aws/ebs, aws/redshift)**
	- **AWS Owned Key:**
		- Collection of CMKs that an AWS service owns and manages to use in multiple accounts
		- AWS can use those to protect resources in your account (but you can’t view the keys)
	- **CloudHSM Keys (custom keystore):** 
		- Keys generated from your own CloudHSM hardware device
		- Cryptographic operations are performed within the CloudHSM cluster
#### Key Policies
Cannot access KMS keys without a key policy
- **Default Key Policy**
    - Created if you don’t provide a specific Key Policy
    - Complete access to the key for the root user ⇒ any user or role can access the key (most permissible)
- **Custom Key Policy**
    - Define users, roles that can access the KMS key
    - Define who can administer the key
    - Useful for cross-account access of your KMS key
## AWS Certificate Manager
 Let’s you easily provision, manage, and deploy SSL/TLS Certificates. 
 - Is it possible to perform 3 actions:
	 - **Create Public** certificate
	 - **Create Private** certificate
	 - **Import third party certificate** certificate
- Is it possible to import third part certificate from external sources
 - ACM can be attached to the following AWS resources: **Elastic Load Balancer,CloudFront, API Gateway, Elastic Beanstalk.**
![[Pasted image 20241014153547.png]]
 
 - **Terminating SSL at the Load Balancer.** All traffic in-transit beyond the ALB is unencrypted. You can add as many as EC2 instances that you want
 - **Terminating SSL End-to-End**: Traffic is encrypted in-transit all the way to the application

#### Certificate Renewal
There are 2 ways to renew your certificates:
- **Automatically** using DNS validation
- **Manually** using emaill validation
## AWS Cognito
Is a Customer identity and access management system. It provides authentication, authorization and user management for your web and mobile apps. It also provides authentication to AWS Services.

**Components:**
- **Cognito User Pools**: User directory with authentication to grant access to your app
- **Cognito Identity Pools**: Provide temporary credential for users
- **Cognito Sync**: Sync user data accross all devices
## Amazon Detective
Amazon Detective analyzes, investigates, and quickly identifies the root cause of security issues or suspicious activities (using ML and graphs). More powerful than **GuardDuty, Macie, and Security Hub.**
## Directory Service
A directory service  that maps the names of network resources to their network addresses.
A directory service is shared information infrastructure for locating, managing, administering and organizing resources.

**Well known services:**
- Domain Name Service (DNS), Microsoft Active Directory, Apache Directory Server, Oracle Internet Directory, Open LDAP, CLoud Identity, JumpCloud
**AWS Directory service**: provides multiple ways to use Microsoft Active Directory. 
- Simple AD: Not available for all the regions.  powered by Sambda.
- AD Connector: a proxy service to connect your existing on-premise AD Directory
- AWS Managed Microsoft AD: A full feature managed version of MS active directory
- Amazon Cognito
## Secret Manager
Store and rotate automatically database credentials. Enforce encryption at rest by using KMS. It as a cost. For credential access monitor you can use Cloud Trail.

You can setup automatic rotation, is not enabled by default. 
(Under the hood it uses lambda)
# Utils
## QuickSight
***Serverless*** machine learning-powered business intelligence service to create interactive dashboards
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
- **Service Enpoints**: Is the URL of the entry point for an AWS web services
	types:
	- **Global Endpoints**
	- **Private Endpoints** 
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
Elastic Transcoder is video-transcoding server, used to convert media files stored in S3 into media files in the formats required by consumer playback devices. **(Very expensive)**

- AWS Elastic Transcoder cannot be used via AWS CFN
	- If you need to automate you need to use the AWS SDK or AWS CLI
- Create a pipeline
	- Create a job
		- Choose a preset (determines what to cover the file to)
		- Choose source and destination bucket
## SNS
### Concept:
**Publish Subscribe pattern:** Common message pattern, the sender send their message to an event bus that categorized message in groups and than the receiver subscribe o these groups. Whenever new messages appear within their subscription the messages are immediatly delivered to them
![[Pasted image 20241008111643.png]]
### SNS
 Amazon Simple Notification Service (Amazon SNS) is a managed service that provides message delivery from publishers to subscribers (also known as _producers_ and _consumers_). In this way is possible to send one message to many receivers. Used to decouple microvservices.
	- Source: Publisher
	- Destination: Subscriber
#### Topics:
Allows you to group multiple subscriptions together.
- 2 types: **Standard** and **FIFO**

|                      | **Standard**                                                                                                                    | **First-in-First-Out**                                                                                                 |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **Throughput**       | High throughput                                                                                                                 | Lower throughput compared to Standard                                                                                  |
| **Delivery**         | At least once (possibility of duplicates)                                                                                       | Exactly once (no duplicates)                                                                                           |
| **Ordering**         | No ordering guarantees                                                                                                          | Messages are delivered in the order they are sent within a message group                                               |
| **Use-Case**         | Where the volume of messages is high and exact ordering/delivery isn't critical: <ul><li>alerts</li><li>notifications</li></ul> | Where the order and exact delivery are crucial: <ul><li>banking transactions</li><li>ordered data processing</li></ul> |
| **Message Grouping** | N/A                                                                                                                             | Supports message grouping, allowing multiple ordered streams within the same topic                                     |

- **Messages:** Is it possible to programmatically publish messages to SNS Topic. **Maximum limit** of message dimension: 256KB. **Message bigger than 256KB can be publish using Amazon SNS Extended Client library (Max: 2GB).** 
- **Subscription:** A subscription can only subscribe to one protocol and one topic: HTTP/S, Email, Amazon SQS, AWS Lambda, SMS, Platform application endpoint
- **Filter policy:** allows you to filter a subset a messages only to be delivery.
- **Message Data Protection**: Message data protection safeguards the data that's published to your Amazon SNS Topic. 
- **Delivery Policy**: defined how SNS **retries** the delivery of messages when **server-side errors** occur. Each delivery protocol has its own delivery policy.
- **SNS Dead Letter Queue (DLQ)** will send fail message attempts to an SQS queue. Topic and Queue type need to match.
![[Pasted image 20241008100115.png]]
## SQS
![[Pasted image 20241014155049.png]]
### Concept

**Messaging System**: Used to provide asynchronous communication and decouple processes via messages/events from a sender and receiver (producer and consumer)
**Queueing System**: messaging system that generally will delete messages once they are consumed. Simple communication. Not Real-time. Have to pull. Not reactive.
### SQS
**Simple Queueing Service (SQS)**: Fully managed queuing service that enables you to decouple and scale microservices, distributed systems, and serverless applications

**Use Case**: You need to queue up transaction emails to be sent e.g. Signup, Reset Password.
SQS Queue types:

- 2 types of queue: 
	- **Standard:** allows you to send nearly unlimited number of transactions per second, but message are out of order
	- **FIFO:** guarantees the order of messages when being consumed. 300 transaction, no duplicates, order by id, 10 message read a time. doesn't accept duplicates.

- **Message Size**: The maximum size of a message is 256 KB. Message bigger than 256KB can be publish using Amazon SQS Extended Client library (Max: 2GB).
- **Message retention**: by default is 4 days. can be adjusted from a minimum of 60 seconds to a max of 14 Days
- **Queue Encryption**: SQS queue can be encrypted using Amazon managed Server-Side-Encryption (SSE) or using KMS
- Message are grouped by ID
- Is possible to attach an ASG to the consumer instances
- **ABAC**: Is an authorization process that defines permissions based on tags that are attached to users aqnd AWS resources. Use tag and aliases.
- **Access Policy:** Allows you to grant other principals permission to the SQS Queue.
- **Message Metadata:** allows you to attach metadata to messages
- **Message Visibility Timeout:** is a period of time message will be invisible after they are read/consumed by an application to avoid being read and processed by other applications. Default is 30 second
- **Delay Queues:** let you postpone the delivery of new messages to consumers for a number of second when your app needs more time. Message remain invisible to consumers for the duration of the delay period.
- **Message Timers:** let you specify an initial invisibility period for an individual message when sending to the queue
- **Temporary Queues:** Let you specify an initial invisibility period for an individual message when sending to the queue. Don't support timers on individual message
- **Short vs Long Polling:** Polling is the method by which we retrive messages from the queues

|                          | **Short Polling (default)**                                                 | **Long Polling**                                                    |
|--------------------------|-----------------------------------------------------------------------------|---------------------------------------------------------------------|
| **Description**           | Returns messages immediately, even if the message queue being polled is empty. | Waits until message arrives in queue or till long poll timeout expires. |
| **Use-Case**              | When you need a message **right away**                                      | When you need to **save money** by reducing how often you poll       |

## Amazon MQ 
Is a managed message broker service for the opensource Apache ActiveMQ (Apache message broker) and RabbitMQ :
- Supporta vari tipi di protocolli: **AMQP,MQTT,STOMP**
## Amazon Kinesis
AWS fully managed solution, not serverless, for collecting processing, and analyzing streaming real-time data in the cloud.
### Types
#### Kineses Data Streams:
is a real time data streaming service. **Most flexible option**.
**Has 2 capacity mode:**

|                | **On Demand**                                                                                                                                                                                                                                  | **Provisioned**                                                                                                                                                                      |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Use Case**   | Unpredictable workloads                                                                                                                                                                                                                        | Predictable workloads                                                                                                                                                                |
| **Management** | Automatically scales                                                                                                                                                                                                                           | Customer manages shards                                                                                                                                                              |
| **Billing**    | Volume of data ingested and retrieved                                                                                                                                                                                                          | Number of shards, data transfer                                                                                                                                                      |
| **Capacity**   | <ul>Writes: <li>200 MiB per second</li><br><li>200,000 records per second</li></ul><br><ul>Reads: <li>400 MiB per second per consumer</li><br><li>2 consumers by default</li><br><li>Enhanced Fan-Out (EFO) to add 20 more consumers</li></ul> | <ul>Write: <li>1 MB per second per shard</li><br><li>1,000 records per second per shard</li></ul><br><ul>Read: <li>2 MiB per second per shard</li><br><li>Max shards is 200</li><ul> |
- **Data Retention:** 1 day (default) to 365 days.
- **Partition Keys**
	- used to group data by shard within a stream
	- partition keys are Unicode strings, with a maximum length limit of 256 characters for each key
	- MD5 hash function is used to map partition keys to 128-bit integer values and to map associated data records to shards using the hash key ranges of the shards
	- When an application puts data into a stream, it must specify a partition key.
- **Sequence Number**
	- Each data record has a sequence number that is unique per partition-key within its shard
	- Sequence numbers for the same partition key generally increase over time
		- The longer the time period between write requests, the larger the sequence numbers become
- **Data Streams CLI:** 
	- Using the AWS CLI the PutRecord alow us to send data to the stream. Note that data has to be based64 encoded
	- Using the AWS CLI the getRecords we can retrive data
- **EFO(Enhance Fan Out)**: allows up to 20 consumers to retrive records from a stream throughput of up to 2 MB of data per second per shard
- **Producers** use SDK, Kinesis Producer Library (KPL) or **Kinesis Agent** to publish records
	- **Kinesis Producer Library(KPL)** is a managed library by AWS to let you publish data to a Kineses data stream. Is a Java library
- **Consumers** use SDK or KCL: 
	- **Kineses Client Library** is a java library that makes it easy for developers to easily consume data for kineses. via the Multi-Daemon other programming languages can be used
#### Amazon Data FireHose
**Serverless**, simple version of data streams. allows for simple transformation and delivery of data.
- Usage
	- You choose one consumer from a predefined list
	- Data immediatly disappears once it's consumed
- **Sources (Producers)**: In firehose allows easily to configure a source
- **Destination (Consumer)**: Data Firehose can send to:
	- (**S3, Redshift**, OpenSearch)
	- Splunk, MongoDB, DataDog, NewRelic, etc.
	- HTTP endpoint.
- **Data Transformation:** before data is sent to a destination it can be tranformed with AWS Lambda
- **Dynamic Partitioning**: enables you to continuosly streaming data by using keys within data and then deliver the data grouped by these keys into Amazon S3 Prefixes. Makes easier to run anlalytics. Once enabled it cnnot be disabled.
- **Convert record format**: Data firehose can convert JSON data into different file formats before being delivered to S3
#### Kineses Data Analytics
**Serverless**, AWS service, to stream live video from devices to tha AWS cloud, or build applications for real time video processing or batch oriented analytics.
#### Kinesis video streams
allows you to run queries agains that is flowing through your real time stream so you can create and analysis on emerging data.



## AWS Data Exchange
Is a catalogue of third-party datasets. You can download for free subscribe or purchase datasets.

You can download dataset and upload your own dataset. Data grants is the tool that allow you to control access to your datasets.

## AWS Glue
AWS Glue is serverless data integration that makes it easy for **analytics** users to discover, prepare, move and integrate data from multiple sources. Visually create, run, monitor, extract and load ETL pipelines to load data into your data lakes. A pipeline is composed of more jobs

- Search and catalog using, Athena, EMR and Redshift
- **AWS Glue Studio**, feature that allows you to visually build ETL jobs and pipelines
- **AWS Glue ETL jobs are charged based on the number of data processing units (DPUs)**
- **AWS Glue Data Catalog:** is a fully manage Apache Hive Metastore-compatible catalog service that makes it easy for customer to store, anntoate and share metadata about their data


## AWS Instance Scheduler
The Instance Scheduler on AWS solution automates the starting and stopping of various AWS services including [Amazon Elastic Compute Cloud](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html) (Amazon EC2) and [Amazon Relational Database Service](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) (Amazon RDS) instances.

You are responsible for the cost of the AWS services used while running Instance Scheduler on AWS. As of the latest revision, the cost for running this solution a small deployment in two accounts and two Regions is approximately **$13.15 per month.**
# Develop
## ElasticBeanStalk 
With Elastic Beanstalk you can quickly deploy and manage applications in the AWS Cloud without having to learn about the infrastructure that runs those applications. It provides CloudFormation templates for you.

**Is not reccomended for production level application**
### Environment
Elasti Beanstalk provide 2 types of environment:
#### Web environment
Used for running web application, cames in 2 types:
- **Single Instance:**
	- Is designed to scale
	- Uses an ESG and ELB
- **Load Balance**
	- Still uses an ASG but Desired capacity is set to 1 to ensure server is always running.
	- No ELB to save cost
	- Public IP addess has to be used to route traffic
#### Worker environment
Used for running background job
- Create ASG 
- Create SQS Queue
- Installs the SQS Daemon on the EC2 instances
- Create CloudWatch Alarm to dynamically scale instances based on health.

## DeviceFarm
Device Farm is an app testing service that you can use to test and interact with your Android, iOS, and web apps on real, physical phones and tablets that are hosted by Amazon Web Services (AWS).
Let you simulate the launch of your app i different devices and browser.

Used for:
- Automated app testing
- Remote access interaction
## AWS Amplify
A set of tools and services that helps you develop and deploy scalable full stack web and mobile applications.

Support a set of famous framework (Angular, React, Flutter ecc...)
## Amazon API Gateway

**Amazon API Gateway** is an AWS service for creating, publishing, maintaining, monitoring, and securing REST, HTTP, and WebSocket APIs at any scale. API developers can create APIs that access AWS or other web services, as well as data stored in the **AWS Cloud**.
![[Pasted image 20241008142105.png]]
- **3 types:**
	- Rest API (API Gateway V1)
		- Higher cost
		- Both private and public options
	- HTTP API(API Gateway V2)
		- Simple feature
		- Only public APIs
	- Web Socket API: persistent connections for real time use cases such as chat application or dashboard

-  Rate Limiting (throttle requests) - returns **429 Too Many Requests**
-  Cache API responses
-  **Can be integrated with any HTTP endpoint in the backend or any AWS API**


### IAM Policy

- Create an IAM policy and attach to User or Role to allow it to call an API
- Good to provide **access within your own AWS account**
## Endpoint Types

- **Edge-Optimized** (default)
    - For global clients
    - Requests are routed through the [CloudFront](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/CloudFront) edge locations (improves latency)
    - The API Gateway lives in only one region but it is accessible efficiently through edge locations
- **Regional**
    - For clients within the same region
    - Could manually combine with your own CloudFront distribution for global deployment (this way you will have more control over the caching strategies and the distribution)
- **Private**
    - Can only be accessed within your VPC using an **Interface VPC endpoint** (ENI)
    - Use resource policy to define access
## ECR
Private Docker Registry on AWS, similar to **docker hub**. Makes it easier for developers to store, manage, and deploy Docker container images. Lets you to store Docker and Open Container Initiative (OCI) Images.
- You can controll repo acces  **Repo Policy**
- You can control private register access via **Register Policy**
- You must  remotely login using docker
- **Pay per use**
- **Encryption at rest** for repo images
- **ECR Lifecyle policy**: can be used to expire old images based on specific criteria
## ECS
 Amazon Elastic Container Service (Amazon ECS) is a fully managed container orchestration service that helps you easily deploy, manage, and scale containerized applications. Compare to fargate you need to build the infrastructure on your own (EC2 instances).

- **ECS Task Lifecycle:**
	![[Pasted image 20241003143946.png]]

#### With Load Balancer
- For every container, the container port is mapped to a random free port on the host (instance). So the application running inside that container will be reached by the ALB on that random port.
- **Dynamic Host Port Mapping** - Once the ALB is registered to a service in the ECS cluster, it will automatically find the right port on the EC2 Instances. This only works with ALB, not CLB.
- You **must allow on the EC2 instance’s security group any port from the ALB security group** because it may attach on any port
### ECS Fargate
AWS Fargate is a technology that you can use with Amazon ECS to run [containers](https://aws.amazon.com/what-are-containers) without having to manage servers or clusters of Amazon EC2 instances. With AWS Fargate, you no longer have to provision, configure, or scale clusters of virtual machines to run containers. 

- You can create an **empty** ECS cluster (no EC2's provisioned) and then launch Tasks as Fargate
- You **no longer have to provision, configure, and scale clusters** of EC2 instances to run containers
- You are charged by seconds but for **at least one minute**.
- When using ELB to point to Fargate you have to use IP addresses since Fargate tasks do not have a hostname

#### With Load Balancer
- **Each task has a unique IP but the same container port**
- The ALB connects to each task directly on its IP and container port since these containers are not run on a defined host (instance).
- You **must allow on the ENI’s security group the task port from the ALB security group**

### Scaling ECS Tasks using EventBridge
- You can use EventBridge (CloudWatch Events) to run Amazon ECS tasks when certain AWS events occur.
- Ex: set up a CloudWatch Events rule that runs an Amazon ECS task whenever a file is uploaded to an S3 bucket. You can also declare a reduced number of ECS tasks whenever a file is deleted from the S3 bucket.
## EKS
Amazon **Elastic Kubernetis Service** is a managed service that  simplifies the process of building, securing, operating, and maintaining Kubernetes clusters on AWS
![[Pasted image 20241003151405.png]]
 EKS can use different type of compute nodes: 
 - EC2 instances 
 - Fargate Instances
 - External instances

 **EKS Connector**: if you want to connect your own kubernetis cluster
 **EKS CTL**: is CLI tool for easily setting up kubernetis clusters on AWS.
 **EKS Distro**: is a Kubernetis distribution based on and used by EKS to create reliable and secure kubernetis clusters.
 
**Use Cases:**
- Hybrid Deployments - consistency between AWS and on-prem
- Development and Testing - identical prod and dev environment
- AWS Services Extension - AWS integration to on-premises setups

- **EKS anywhere(EKS-A)**: is a deployment option for Amazon EKS to easily create and operate **kubernetis(k8s)** clusters on premises with your own Vms or bare metal hosts. EKS anywhere uses **EKS Distro** as the **kubernetis** cluster distribution.
## Amazon managed service for Prometheus (AMP) 	
 **Prometheus**: Open source monitoring and alerting toolkit originally built at SoundCLoud. **Is a timeseries database** (Collect and stores its metrics as time series data). 

**Amazon managed service for Prometheus**: Is a Prometheus compatible monitoring service for container infrastructure and application metrics
## Amazon managed service for Graphana
**Graphana**: is an open source analytics web application. It is used with timeseries database like Prometheus.

**Amazon managed service for Graphana**: Fully managed and secure data visualization service that you can use to instanly query, correlate, and visualize operational metrics , logs, and traces from multiple sources.
## Lake Formation 
**Data Lakes**: A data lake is a centralized data **repository** for unstructured and semi structured data. Contain vast amount of data. Data lakes generally use object(blob) or file as its storage type. Often used for analytics

**AWS Lake Formation:** It is a data lake to centrally govern secure and globally share data for analytics and machine learning.


![[Pasted image 20241003093952.png]]

## AWS CodeDeploy
 CodeDeploy is a deployment service that automates application deployments to Amazon EC2 instances, works with On-Premises Servers • Hybrid service
![[Pasted image 20241014110813.png]]

## AWS CodeBuild
 AWS CodeBuild is a fully managed build service in the cloud. CodeBuild compiles your source code, runs unit tests, and produces artifacts that are ready to deploy. **Pay-as-you-go**
 ![[Pasted image 20241014111131.png]]
## AWS CodePipeline
 Orchestrate the different steps to have the code automatically pushed to production, Code => Build => Test => Provision => Deploy, Basis for CICD (Continuous Integration & Continuous Delivery)
 ![[Pasted image 20241014111207.png]]
## AWS CodeArtifact
It is a secure, highly scalable, managed artifact repository service that helps organizations to store and share software packages for application development. **Dependencies management**
 ![[Pasted image 20241014111235.png]]
## AWS Cloud9
 AWS Cloud9 is an integrated development environment, or _IDE_.
## AWS CodeStar (Non più supportato)
Unified UI to easily manage software development activities in one place
## AWS CodeCommit (Non più supportato)
 Is a version control service hosted by Amazon Web Services that you can use to privately store and manage assets. 
# Database
## RDS
Amazon Relational Database Service (Amazon RDS) is a web service that makes it easier to set up, operate, and scale a relational database in the AWS Cloud. It provides cost-efficient, resizable capacity for an industry-standard relational database and manages common database administration tasks.
![[Pasted image 20241009092943.png]]

**DB instances:** is an isolated database environment running in the cloud
- DB instance **can contain one or multiple user** created database
- Each database has user defined database identifier which forms part of the DNS hostname
- **DB instance Class:** just like ec2 instance class determine available compute memory or a db instance
- DB instance **use Elastic Block Storage (EBS) volumes** for database and **log storage**
- **Maximum storage of 64TB**
- We don't have access to the underlying instance

### RDS Storage Auto Scaling 
Automatically scales storage capacity in response to growing database workloads, with zero downtime. (only storage capacity)
### Encryption
- Encryption in transit: enabled by default
- Encryption at rest: can be enabled 
- Encrypt alsoare automatically enabled on Automated backups, snapshot and read replicas
- **Can be enabled only during creation**

### RDS Backup
- Types
	- **Automated backup:** choose retention period between 0 and 35 days
		- **enabled by default**
		- stored in s3
		- Log backed up every 5minutes
		- no charge for additional data in automated backups
	- **Manual Snapshot**
		- taken manually
		- additional storage charge 
- **Restoring a backup:** creates a new RDS instance and restores data in that instance 
		- Restoring a manual snapshot
		- Restoring Point in time (PITR): similar but provide restore time
- Take long time to restore database: is not imidiate.

#### Scelte architetturali
#### Read Replicas: 
- **Disaster recovery solution** if the primary DB instance fails  
- Improve **read contention** which means improve performance latency. 
	- **Read contention:** when multiple proccesses or instances competing for access to the same index or data block at the same time.
- You must have **automatic backups enabled**. 
- **Type of replication:** asyncronous replication
- Maximum of **5 replicas** of a database
- Replica techniques:
	- **Standard**
	- **Multi-az**
	- **Cross-region**
#### Multi AZ vs Read Replicas

| Multi-AZ Deployments                                      | Read Replicas                                                       |
| --------------------------------------------------------- | ------------------------------------------------------------------- |
| Synchronous replication – highly durable                  | Asynchronous replication – highly scalable                          |
| Only database engine on primary instance is active        | All read replicas are accessible and can be used for read scaling   |
| Automated backups are taken from standby                  | No backups configured by default                                    |
| Always span two Availability Zones within a single Region | Can be within an Availability Zone, Cross-AZ, or Cross-Region       |
| Database engine version upgrades happen on primary        | Database engine version upgrade is independent from source instance |
| Automatic failover to standby when a problem is detected  | Can be manually promoted to a standalone database instance          |
### Subnet Group

**DB Subnet Group**: collection of subnet that you create in a VPC and that you then designate for your DB instances.
- Each DB should have subnets in at least two Availability Zones in a given AWS Region.
- RDS will choose a subnet from your subnet group to deploy your RDS Instance
- Subnets in a DB subnet group are either public or private
- For a DB instance to be publicly accessible, all of the subnets in its DB subnet group must be public
![[Pasted image 20241003103052.png]]

### Other feature
- **RDS Performance Insights:** helps you easily identify bottlenecks and performance issues. Is default by default and provides  1 week of performing data. For additional cost you can change the retention period to 2 years.
- **RDS Proxy:** create a connection pooler so that short lived AWS Lambda functions connecting to RDS does not quick exahaust all connection. Reuse existing connection to do this.
- **IAM Authentication:** allows you to authenticate with an IAM authentication token to an RDS instance's database insteas of using a password.
	- Each token has a lifetime of 15 minutes
	- You have to create a policy and attach to user
	- You have to create db user 
	- You need to generate auth token to be used for password authentication
- **Master User Account:** is the initial databse account that's created when you provision a new Db instance. Has full privileges. Pass and username set at creation time.
- **Public Accessibility:** is an option that changes if the DNS Endpoint resolve to the private IP address from traffic from outside the VPC

### Extra
- **RDS Custom:** automates database administration tasks and operation. Allows customers to directly manage aspects of RDS instead of AWS, for company that needs third party applications for their databse.
- **Optimized Reads and Writes:** allow database operations to maximize perfrmance efficency and throughtput. Use NVMe based SSD block storage instead
- **RDS Security Groups:** In order to establish connection for both public and private you need to open the port
- **RDS Blue Green Deployments:** copies a production database environment in a separate, synchronized staging environment
- **RDS Extended Support:** allows you to run your database on a major engine version past the RDS end of standard support date for an additional cost.

## Aurora
**Serverless**, Automated database instantiation and auto-scaling based on actual usage.
- **Aurora costs more than RDS (20% more).
- Supports only MySQL & PostgreSQL
- **Asynchronous Replication** (milliseconds)
- Up to 15 read replicas

- **Aurora Global Database:** It is a database spanning multiple regions for global low latency  and high availabilty. Primary cluster is in a separate region
- **RDS Data API:** allows you to use HTTP to securely query an Aurora database. Unlimited max request per seconds. Unlimited max request per seconds. Must be enabled 
### Durability and Fault Tolerance
- **Aurora Backup and Failover are handled automatically** 
- Snapshots of data can be shared with other AWS accounts
- Storage is self-healing, in that data blocks and disks are continuously scanned for errors and repaired automatically.
### Availability
- **Aurora deploys in a minimum of 3 availability zones each contain 2 copies of your data** at all times.
- Maintain 6 copies
- Lose up to 2 copies of your data without affecting write availability.
- Lose up to 3 copies of your data without affecting read availability.
### Storage
- A cluster **starts with 10GB** of storage and scale in 10GB increments **up to 64TB or 128 TB** depending on DB engine version. Storage is autoscaling.
- Computing resources can scale up to 32 vCPUs and 244GB of memory.
### Security
- Encryption at rest using KMS (same as RDS)
- Encryption in flight using SSL (same as RDS)
- You can’t SSH into Aurora instances (same as RDS)
- Network Security is managed using Security Groups (same as RDS)
### Type
#### Aurora Serverless Provisioned
Default compute configuration for Aurora. primary db that perform read and writes and up to 15 Replica. primary DB instance is not created by default
#### Aurora Serverless V2
fully manages the autoscaling configuration for Amazon Aurora
	- Capacity is adjusted automatically based on application demand
	- charge only for resources used
	- use case: **Highly variable workloads**
	- Does not scale to zero, must mantain at least 0.5 ACU(unit mesurement of Aurora to determine cost vs capacity)
#### Aurora Serverless V2 vs Provisioned
|                        | Aurora Serverless V2                                       | Aurora Provisioned                                       |
| ---------------------- | ---------------------------------------------------------- | -------------------------------------------------------- |
| **Scaling**            | Fine-grained, almost instant scaling.                      | Manual scaling; requires planning and downtime.          |
| **Capacity Range**     | 0.5-128 ACUs, **more flexible**.                           | **Fixed**, based on instance size chosen.                |
| **Scaling Speed**      | Seconds                                                    | N/A (manual intervention required).                      |
| **Read/Write Scaling** | Independently                                              | Depends on instance type and read replica configuration. |
| **Compatibility**      | Broader version support.                                   | Wide version support, depending on instance type.        |
| **Use Cases**          | Highly variable workloads needing immediate scaling.       | Stable workloads with predictable performance needs.     |
| **Billing**            | ACUs per second, more granular, + storage.                 | Instance hours + storage.                                |
| **Start/Stop**         | Responsive start/stop, cost-saving for intermittent loads. | Manual start/stop.                                       |
| **Maintenance**        | Minimal downtime, more seamless.                           | Scheduled maintenance windows.                           |

## DynamoDB
Amazon DynamoDB is a ***serverless***, NoSQL (key/value), fully managed database with single-digit millisecond performance at any scale.  
- **Highly available** with replication **across 3 AZ**.
- **Multi-Region**
- **Single digit millisecond** response time at any scale
- **Maximum size of an item: 400 KB**
- Max 100 table
- DyanmoDB creates partitions for you as your data grow. Creates a partition every 10 GB or when you exceed.
### Capacity
- **Provisioned Mode** (default)
    - Provision read & write capacity
    - **Pay for the provisioned capacity**
    - Auto-scaling option (eg. set RCU and WCU to 80% and the capacities will be scaled automatically based on the workload)
- **On-demand Mode**
    - Capacity auto-scaling based on the workload
    - **Pay for what you use (more expensive)**
    - Great for unpredictable workloads
### Read Consistency 
When data needs to be updated in all of its copy and keep data consistent
there are different types of read consistency:

| Eventual Consistent Reads (DEFAULT)                                                                | Strongly Consistent Reads                                                                                            |
| -------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| When copies are being updated, it is possible for you to read and be returned an inconsistent copy | When copies are being updated, and you attempt to read, it will not return a result until all copies are consistent. |
| Reads are fast, but there is **no guarantee of consistent**                                        | You have a **guarantee of consistency**, but the trade-off is **higher latency (slower reads)**.                     |
| All copies of data eventually **become generally consistent within a second**                      | All copies of data **will be consistent within a second**                                                            |
### Primary Keys 
- **Simple Primary Key:** a primary key with ony a **partition key** to choose which partition
- **Composite Primary Key:** a primary key with both a **partition key** and a **sort key** to choose partition
- **Query:** 
	![[Pasted image 20241003124503.png]]
- **Scan:** scan through all items return one or more items. (much less efficient then query - not said by amazon)

## DocumentDB
NOSQL, document database that is based on MongoDB: 
- Cluster types:
	- **Instance based Cluster**: manage your instances directly choosing instance type
	- **Elastic Cluster**: clusters automatically scale, you choose vCPU and number of instances per shard
- Compatible with MongoDB 4.0 and 5.0
- DocumentDB does not support all functionality for MongoDB.
- **DocumentDB storage volume grows in increments of 10 GB, up to a maximum of 128 TB.**
- Create up-to **15 replicas**
- Automatically recover from instance failover
- Backup is turned on by default (can't be turned off) with a **max retention period of 35 days**
- Supports **point-in-time recovery**.
- Clusters are deployed into a customer's VPC.
- Performance Insights feature to determine bottlenecks for reads and writes.
- In-transit and at-rest encryption. You must connect using TLS connect.
## Amazon Keyspace
Amazon Keyspaces is a fully managed Apache Cassandra database. Cassandra is an open-source NoSQL key/value database similar to DynamoDB. It is a  columnar store database but has some additional functionality.

- **Cluster** - a collection of nodes 
- **Nodes** - holds 2 - 4 TB of data
	- All nodes read and write
	- Nodes represent smallest unit of a database
	- Data is replicated on multiple nodes
- **Ring** - Nodes are arranged in a ring where all nodes connect to each other
- **Keyspace** - a namespace that specifies data replication on nodes
- **Table** - tabular data of columns and rows with a primary key
## Neptune
Amazon Neptune is a fast, reliable, fully managed graph database service that makes it easy to build and run applications that work with highly connected datasets.
- Highly available across **3 AZ** with up to **15 read replicas**
- **Netpune Database**: **2 types**
	- **Provisioned**: you choose an instance type
	- **Serverless**: set a min and max Neptune Capacity units
- **Neptune Analyses**:  provide capabilites to run large-scale graph analytics algorithms efficiently
## ElastiCache
Amazon ElastiCache is a web service that makes it easy to set up, manage, and scale a distributed in-memory data store or cache environment in the cloud

 - **Can only accessible by resources in the same VPC** (to ensure low latency)
- It can be **cross region** if enabled
- **Using ElastiCache requires heavy application code changes**
- Deployment Options:

|            | Serverless                                                     | Standard                                                                       |
| ---------- | -------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Use case   | Unpredictable workloads                                        | Predictable Workloads                                                          |
| Management | Automatically scales                                           | Customer managers cluster and nodes                                            |
| Billing    | <ul><li>Data Stored</li><li>ElasticCache Processing Units</li> | <ul><li>Based on number of nodes</li><li>Based of node type(size of node)</li> |
### Types
#### Redis
Open source in-memory database store. Redis acts as caching layer, or a very fast database. is key/value store.
Can perform many different kind of operations on your data. It is very good for leaderboards, and keep track of unread notification data. It is very fast, but arguably not as fast as memcached

#### MemCached
Is an open-source distributed memory object caching system. It's a caching layer for web-applications. Key/value store.

#### Redis vs Memcache

| Redis                                                          | Memcached                                            |
| -------------------------------------------------------------- | ---------------------------------------------------- |
| In-memory data store                                           | **Distributed** memory object cache                  |
| Read Replicas (for scaling reads & HA)                         | No replication                                       |
| Backup & restore                                               | No backup & restore                                  |
| **Single-threaded**                                            | **Multi-threaded**                                   |
| **HIPAA compliant**                                            | **Not HIPAA compliant**                              |
| Data is stored in an in-memory DB which is replicated          | Data is partitioned across multiple nodes (sharding) |
| **Redis Sorted Sets** are used in realtime Gaming Leaderboards |                                                      |
| Good for auto-completion                                       |                                                      |
| Multi-AZ support with automatic failover (disaster recovery)   |                                                      |

## MemoryDB
Is a Redis-compatible in-memory database for ultra-fast performance
Suitable to be primary database. **Slower writes** but better performance in general.
## Amazon Redshift
 **Serverless,** Is an enterprise-class relational database based on postgresql. Is thinked for data warehouse and for **OLAP(analytics and data warehousing)**. 
 
 - Storage is done by **column** and not by row. 
 - **Pay as you go based**
 -  **Based on postgresql**
 -  **No multi-AZ support** (all the nodes will be in the same AZ)

**Datawerehouse**: Built to store large quantities of historical data and enable fast, complex queries across all the data.

**OLAP** applications look at multiple records at the same time. You save memory because you fetch  just the columns of data you need
 instead of whole rows.

- **Different type of configurations:**
	- **Single Node**: Nodes comes in sizes of 160GB. You can launch a single node to get started with Redshift.
	- **Multi Node**: You can launch a cluster of nodes with Multi-node mode
	- **Leader Node**: manages client connections and receiving queries
	- **Compute Node**: stores data and performs queries up to 128 computes nodes
- **Different type of nodes:**
	- **Dense Compute(dc)**: best for high performance, but they have less storage
	- **Dense Storage(dc)**: cluster in which you have a lot of data
- **Compression**
	- **Redshift** uses multiple compression techniques.
	- Similar data is stored sequentially on disk
	- Does not require indexes or materialized views, compared to traditional systems
	- When loading data to an empty table, data is sampled and the most appropriate compression scheme is selected automatically
- **Processing**
	- Uses massively Parallel Processing(MPP)
	- Automatically distributes data and query loads accross all nodes
	- Lets you easily add new nodes to your data warehouse while still maintaining fast query performance.
- **Backup**
	- enabled by default, 1 day retention, up to 35
	- always attempt to keep 3 copies of your data
- **Billing**
	- Total number hours
	- not charged for leader node hours only compute nodes incur changes
- **Security**
	- Dat in transit: **SSL**
	- Data at rest: **AES 256**
	- **Encryption applied using KMS, CloudHSM**
- **Availability**
	- Redshift is Single AZ. In order to reach high availability you need to run multiple redshift cluster in different AZ. Anapshot can be restored o a different AZ 

![[Pasted image 20241002165058.png]]

## Athena
**Serverless** is a query service to analyze data stored in Amazon S3. It uses standard SQL language to query the files and their content.
Athena can do 2 things:
- Lets you run SQL queries on S3 Buckets
- Interactively, run data analytics, using Apache Spark

- Athena SQL
	- Component
		- Workgroup: saved queries which you grant permissions to other user to access
		- Data source: a group database 
		- Database: a group of tables
		- Table: data organized as group
		- dataset: raw data of table
	- Table
		- You can create using SQL and AWS Glue Wizard
	- **SerDe**: is a serialization and deserialization libraries for parsing data form different format
		- Can override the DDL configuration that you specify in Athena when you create your table
## Quantum Ledger Database
Used to review history of all the changes made to your **application data** over time, serverless. **Difference with Amazon Managed Blockchain**: no decentralization component

**Features:**
- **Immutable Logs**: Data cannot be altered or deleted after entry, ensuring permanent records.
- **Cryptographic Verification**: Utilizes SHA-256 hashing for secure, verifiable transaction histories.
- **Fully Managed**: Automated management of infrastructure allows users to focus on application development without worrying about underlying hardware.
- **Serverless**: Scales automatically with application demands, enhancing efficiency.
- **SQL-like Queries**: Supports PartiQL for flexible, SQL-compatible data querying.
- **Central Governance**: Managed under a central trusted authority, ideal for applications needing a reliable transaction log.
- **High Throughput and Scalability**: Designed for rapid and frequent updates with minimal latency.
- **AWS Integration**: Works seamlessly with other AWS services for improved functionality and simpler management.
- **ACID Transactions**: Ensures database integrity with transactions that are atomic, consistent, isolated, and durable.
- **Journal Storage**: Records changes sequentially in a document-oriented format for structured data handling.

# Monitoring
## Cloud Trail
Is a service that enables governance compliance operational auditing of your AWS account

- **Global Service** 
- Active by default
- Collect logs for 90 days via Event History. If you need more than 90 days you need to create a Trail.
- Very useful to monitor API calls and made Actions made on an AWS account. Easily identify which users on an AWS account made call.

**To analyze a Trail you'd have to use Amazon Athena**
## CloudWatch
Amazon CloudWatch monitors your Amazon Web Services (AWS) resources and the applications you run on AWS in real time. **Amazon CloudWatch is basically a metrics and logs repository. An AWS service puts metrics into the repository, and you retrieve statistics based on those metrics

- **High Availability**
- **Serverless**
![[Pasted image 20241014155339.png]]
### Amazon CloudWatch Alarms
(like budget but less powerful) Alarms are used to trigger notifications for any metric.

- You can create a billing alarm 
-  **Billing data metric is stored in CloudWatch us-east-1**
- **Alarm States:**
    - OK
    - INSUFFICIENT_DATA
    - ALARM
- **Period:**
    - Length of time in seconds to evaluate the metric before triggering the alarm
    - High resolution custom metrics: **10 sec**, 30 sec or multiples of 60 sec
- **Target/Action**:
	- **Auto Scaling Group**, increase or decrease EC2 instances “desired” count  
	- **EC2 Actions**: stop, terminate, reboot or recover an EC2 instance 
	-  **SNS notifications**: send a notification into an SNS topic 
### Amazon CloudWatch Logs
- It is used to store, manage and access your log files. Is a centralized log management server
 - CloudWatch Logs can collect log from:
	 - **EC2 machines or on-premises servers** 
	 - **Elastic Beanstalk**: collection of logs from application 
	 - **ECS:** collection from containers 
	 - **AWS Lambda**: collection from function logs 
	 - **CloudTrail** based on filter 
	 - **Route53**: Log DNS queries
- **CloudWatch log groups**: a collection of log streams ex: /exampro/prod/app
- **CloudWatch streams**: represent a sequence of events from an application or instance being monitored. Is created automatically but you can create your own.
- **CloudWatch log events**: Represent a single event in a log files:
	![[Pasted image 20241004114212.png]]
- **CloudWatch Insights:** enables you to interactively search and analyze your CloudWatch log data
	- CloudWatch discover fields, when reads a logs 
- **CloudWatch Metrics:** time order set of data points, its a variable that monitored over time. You can provide you customize Metrics using CLI/SDK
- **CloudWatch Agent**: Some metrics need to install CloudWatch agent to been tracked.
	- Host Level Metrics
		- These are what you get without installing the Agent
			- CPU usage
			- Network Usage
			- Disk Usage
			- Status Check
				- Underlying Hypervisor status
				- Underlying EC2 instance status
	- **Agent Level Metrics**
		- These are what you get when installing the Agent
			- **Memory utilization**
			- Disk Swap utilization
			- **Disk Space utilization**
			- Page file utilization
			- Log collection
## Amazon EventBridge
**Event Bus**: An event bus receives events from a source and routates events to a target based on rules. 
**EventBridge** is a serverless event bus service that is used for application integration by streaming real-time data to your applications.
![[Pasted image 20241014143558.png]]
## Health Dashboard
- **Service Health Dashboard:** Show General status of the service in various region.
- **Personal Health Dashboard:** AWS Personal Health Dashboard provides alerts and guidance for AWS events that might affect your environment.

## Config
AWS Config provides a detailed view of the configuration of AWS resources in your AWS account. This includes how the resources are related to one another and how they were configured in the past so that you can see how the configurations and relationships change over time.

- Regional service
- Can be aggregated across regions and accounts
- Record configurations changes over time
- **Evaluate compliance of resources using config rules**
- **Does not prevent non-compliant actions** from happening (no deny)
- Evaluate config rules
    - For each config change (ex. configuration of EBS volume is changed)
    - At regular time intervals (ex. every 2 hours)
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
## (IAM) Identity Access Manager
- Anatomy of IAM policy
	- **Version policy language** version. 2012-10-17 is the latest version.
	- **Statement container** for the policy element you are allowed to have multiples
	- **Sid (optional)** a way of labeling your statements.
	- **Effect Set** whether the policy will Allow or Deny
	- **Action list** of actions that the policy allows or denies
	- **Principal** account, user, role, or federated user to which you would like to allow or deny access
	- **Resource** the resource to which the action(s) applies
	- **Condition (optional)** circumstances under which the policy grants permission
- **Principle of Least Privilege:**
	- Just Enough Access: Permitting only the exact actions  for the identity to perform task
	- Just In Time: Permitting the smallest length of duration a identity can use permissions
- **Password policy**: you can set expires date for user password in rder to rotate them
- **Access Key**: you can have a maximum of 2 access key per account
- **MFA**: (vedi appunti aws)
- **Temporary Security Credentials**: Generated dynamically, every time you use a roles a you a temporary security cred are created automatically
- **IAM Identity federetion**: the means of linking a person to eletronic identity and attribute. When you connect using facebook or google you're using identity fed
- **STS**: (vedi appunti AWS)
- **Cross Account Role:** You can grant users from different AWS account access to resources in your account through a Cross-Account Role. This allows you to not to have to create them a user account within your systems
	- ![[Pasted image 20241001115206.png]]
- **IAM AssumeRoleWithWebIdentity**: Returns a set of temporary security credentials for users who have been authenticated in a mobile or web application with web identity provider
	- You always authenticate with the web identity first
## Service Catalog

AWS Service Catalog enables organizations to create and manage catalogs of products that are approved for use on AWS to achive consistent governance and meet compliance requirements.
Alternative to granting direct access to AWS resources via the AWS console
## AWS Resource Access Manager (AWS RAM)
  Share AWS resources that you own with other AWS accounts. **Avoid resource duplication**. Supported resources includes: Aurora, VPC Subnets, Transit Gateway, Route 53, EC2 Dedicated Hosts, License Manager Configurations

## AWS IAM Identity Center: 
**Single sign-on** for multi account in aws.

**AWS Identity Center (formerly AWS Single Sign-On)** is a service that provides centralized management of user and group access to AWS resources. It enables authentication and authorization, integrating with identity providers like Active Directory or other SAML-compliant IdPs.
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
## Cost allocation tag

- As a best practice, reactivate your cost allocation tags when moving organizations. When an account moves to another organization as a member, previously activated cost allocation tags for that account lose their "active" status and need to be activated again by the new management account.
    
- As a best practice, do not include sensitive information in tags.
    
- Only a management account in an organization and single accounts that aren't members of an organization have access to the **cost allocation tags** manager in the Billing and Cost Management console.
## CostExplorer

- Visualize and manage your costs and service usage over time
- Create **custom reports** that analyze cost and usage data
- **Recommendations** to choose an optimal savings plan
- View usage for the **last 12 months & forecast up to 12 months**

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

## AI Dev Tools
#### Amazon Q
Is an AI chatboat using multiple LLm models via Bedrock Ask Amazon Q a question similar to chatGPT or other generative AI. 
#### Amazon Q Business
connect it to company data, information, and systems, made simple with more than 40 built-in connectors.
#### Amazon Q Developer
 coding, testing and upgrading to troubleshooting and optimizing your AWS resources.
 Integrated into the AWS Managed Console, VSCode via the AWS Toolkit, Cloud9, AWS Lambda Code Editor, Slack and other places
#### Amazon Q for Amazon QuickSight
Ask questions about your BI data within QuickSight
Generative BI capabilities to quickly build compelling visuals, summarize insights,
answer data questions, and build data stories using natural language.
#### Amazon Q for Amazon Connect
real-time conversation with the customer along with relevant company content 
#### Amazon Q for AWS Supply Chain
get intelligent answers about what is happening in their supply chain
#### Amazon CodeWhisper
Is a realt-time AI coding companion. Genaretes suggested code while you're writing code.
![[Pasted image 20241003174535.png]]

## Connect:
Amazon Connect is an AI-powered application that provides one seamless experience for your contact center customers and users. It's comprised of a full suite of features across communication channels.




