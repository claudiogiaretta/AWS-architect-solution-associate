
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

> [!NOTE]
>  - S3 delivers **strong read-after-write consistency** (if an object is overwritten and immediately read, S3 always returns the latest version of the object)
>  - S3 is strongly consistent for all GET, PUT and LIST operations

### Bucket Versioning

- **Enabled at the bucket level**
- Protects against unintended deletes
- Ability to restore to a previous version
- Any file that is not versioned prior to enabling versioning will have version “null”
- Suspending versioning does not delete the previous versions, just disables it for the future
- To restore a deleted object, delete it's "delete marker"

> [!NOTE]
>  - Versioning can only be suspended once it has been enabled.
>  - Once you version-enable a bucket, it can never return to an unversioned state.

### Encryption

- Can be enabled at the **bucket level** or at the object level
- **Server Side Encryption (SSE)
    - **SSE-S3**
        - Keys managed by S3
        - AES-256 encryption
        - HTTP or HTTPS can be used
        - Must set header: `"x-amz-server-side-encryption": "AES256"`
    - **SSE-KMS**
        - Keys managed by KMS
        - HTTP or HTTPS can be used
        - KMS provides control over who has access to what keys as well as audit trails
        - Must set header: `"x-amz-server-side-encryption": "aws:kms"`
    - **SSE-C (Customer provided key)**
        - Keys managed by the client
        - Client sends the key in HTTPS headers for encryption/decryption (S3 discards the key after the operation)
        - **HTTPS must be used** as key (secret) is being transferred
        - must set header:
	        - `x-amz-server-side-encryption-customer-algorithm`
	        - `x-amz-server-side-encryption-customer-key`
	        - `x-amz-server-side-encryption-customer-key-MD5`
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
#### When to Use One Over the Other

| Scenario                                  | Recommended Policy | Reason                                                                                               |
| ----------------------------------------- | ------------------ | ---------------------------------------------------------------------------------------------------- |
| Centralized management of user access     | User-based policy  | Allows for managing user permissions across multiple services in one place.                          |
| Cross-account access to a specific bucket | Bucket policy      | Bucket policies allow access from external AWS accounts without modifying user permissions.          |
| Public access to an S3 bucket             | Bucket policy      | Simplifies granting public access to the bucket.                                                     |
| Access control based on IAM roles/groups  | User-based policy  | Provides more flexibility for permissions based on groups or roles.                                  |
| Specific access for multiple buckets      | User-based policy  | Allows you to manage permissions for users needing access to several buckets.                        |
| Limited access to a single bucket for all | Bucket policy      | Easy to manage permissions for all users accessing a single bucket, especially for shared resources. |

> [!NOTE]
>  By default, an S3 object is owned by the AWS account that uploaded it, even if the bucket belongs to another account. The object owner retains full control over the object unless they explicitly grant permissions to the bucket owner. To ensure the bucket owner has full access to objects uploaded by external users, the bucket owner can create a bucket policy requiring users to include the `bucket-owner-full-control` ACL when uploading objects. This ACL gives the bucket owner full access to those objects.

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
feature in Amazon S3 specifically designed to prevent accidental or unauthorized deletion of data
- MFA required to
    - permanently delete an object version
    - suspend versioning on the bucket
- **Bucket Versioning must be enabled**
- Can only be enabled or disabled by the root user

### Server Access Logging
- Most detailed way of logging access to S3 buckets (**better than CloudTrail**)
- Does not support **Data Events** & **Log File Validation** (**use CloudTrail for that**)
- Store S3 access logs into another bucket
- Logging bucket should not be the same as monitored bucket (logging loop)
### Replication
- **Asynchronous replication**(data are not replicated istantaneously)
- Objects are replicated with the **same version ID**
- Supports **cross-region, same region, bidirectional, batch replication**
- **Versioning must be enabled for source and destination buckets**
- For DELETE operations:
    - Replicate delete markers from source to target (optional)
    - Permanent deletes are not replicated
- There is **no chaining of replication**. So, if bucket 1 has replication into bucket 2, which has replication into bucket 3. Then objects created in bucket 1 are not replicated to bucket 3.

### Pre-signed URL

- Pre-signed URLs for S3 have temporary access token as query string parameters which allow anyone with the URL to **temporarily access the resource before the URL expires** (default 1h)
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
    - No cost on retrieval **(only storage cost)**
    - For **frequently accessed data**
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
    - **Cost for storage and retrieval**
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
#### Moving between Storage Classes
- In the diagram below, transition can only happen in the downward direction
![attachments/Pasted image 20220514201314.jpg](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/attachments/Pasted%20image%2020220514201314.jpg)

### Lifecycle Rules

- **Used to automate transition or expiration actions on S3 objects**
- **Transition Action** (transitioned to another storage class)
- **Expiration Action** (delete objects after some time)
    - delete a version of an object
    - delete incomplete multi-part uploads
- Lifecycle Rules can be created for a prefix (ex `s3://mybucket/mp3/*`) or objects tags (ex Department: Finance)

> [!NOTE]
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
        - **Better resilience in case of failures** since we only need to refetch the failed byte range and not the whole file
    - **S3 Transfer Acceleration**
        - Speed up **upload and download** for **large objects (>1GB)** for **global users**
        - Data is ingested at the nearest edge location and is transferred over AWS private network (uses CloudFront internally)

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
#### Object retention
Object Lock provides two ways to manage object retention:
- **Legal hold** – A legal hold provides the same protection as a retention period, but it has no expiration date.

#### Glacier Vault Lock

- WORM (Write Once Read Many) model for Glacier
- For compliance and data retention

## EBS
Non global service, allows you to create "EBS volume" which are **network drive** you can attach to your instances **while they run.** 
- **They can only be mounted to one instance at a time. 
- EBS volume is not multi Availability Zone**.
- Volume **Network Drive** (provides low latency access to data)
- Must provision capacity in advance (size in GB & throughput in IOPS)
- An EBS volume is off-instance storage that **can persist independently from the life of an instance.**
- **By default**, upon instance termination, **the root EBS volume is deleted** and any other attached EBS volume is not deleted. Can be over-ridden using `DeleteOnTermination` attribute;
### Volume Types
#### SSD
Optimized for **Transaction-intensive Applications** with high frequency of **small & random IO operations**
##### General Purpose SSD Volumes (gp3)
- Good for system boot volumes, virtual desktops
- **Use Case**: Most general workloads, Boot volumes, Low-latency interactive applications ecc..
- **Durability**: 99.8% - 99.9%
- **Volume Size**: 1 GB – 16 TB
- **Max IOPS**: 16,000 (16 KB I/O)
- **Price**: 0,08 USD per GB month
- **Throughput:** 1000 MB

##### General Purpose SSD Volumes (gp2)
It is an older version of gp3. 

Same as general purpose SSD volume but:
- **Throughput**: 250 MB/s
- **Price**: 0,10 USD per GB per month
##### Provisioned IOPS SSD Volumes (io2 Block Express)
- **Supports EBS Multi-attach** (not supported by other types)
- Maintain high IOPS while keeping I/O latency down by maintaining a **low queue length**
- **Use Case**:
    - Sub-millisecond latency
    - Sustained IOPS performance
- **Durability**: 99.999%
- **Volume Size**: 4 GB – 64 TB
- **Max IOPS**: 256,000 (16 KB I/O)
- **Max Throughput**: 4,000 MB/s
#### HDD
- Optimized for **Throughput-intensive Applications** that require **large & sequential IO operations
- Maintain high throughput to HDD-backed volumes by maintaining a **high queue length****
##### Throughput Optimized HDD (st1)
- **Cannot be used as boot volume** for an EC2 instance
- **Use Case**:
    - Big data
    - Data warehouses
    - Log processing
- **Durability**: 99.8% - 99.9%
- **Volume Size**: 125 GB - 16 TB
- **Max IOPS**: 500 (1 MiB I/O)
- **Max Throughput**: 500 MB/s
##### Cold HDD (sc1)
- **Use Case**:
    - Throughput-oriented storage for data that is infrequently accessed
    - Where lowest storage cost is important
- **Durability**: 99.8% - 99.9%
- **Volume Size**: 125 GiB - 16 TiB
- **Max IOPS**: 250 (1 MB I/O)
- **Max Throughput**: 250 MB/s
##### Magnetic (standard)
- **Use Case**:
    - Workloads where data is infrequently accessed
- **Durability**: N/A
- **Volume Size**: 1 GiB - 1 TiB
- **Max IOPS**: 40-200
- **Max Throughput**: 40-90 MiB/s
- **More expensive than Cold HDD**
---
### Snapshots
- It is a point-in-time copies of your EBS volume
- **Data Lifecycle Manager (DLM)** can be used to automate the creation, retention, and deletion of snapshots of EBS volumes
- Snapshots are incremental
- Only the most recent snapshot is required to restore the volume
### Encryption
- **Encryption is enabled by default, by the creation of encrypted volume. It can be turn off.**
- All snapshots are encrypted
**In order to enable encryption in EBS volume**
- Create an EBS snapshot of the volume
- Copy the EBS snapshot and encrypt the new copy
- Create a new EBS volume from the encrypted snapshot (the volume will be automatically encrypted)

### RAID
- **RAID 0**
    - **Improve performance** of a storage volume by distributing reads & writes in a stripe across attached volumes
    - If you add a storage volume, you get the straight addition of throughput and IOPS
    - For high performance applications
- **RAID 1**
    - **Improve data availability** by mirroring data in multiple volumes
    - For critical applications
### Amazon Data Lifecycle Manager (DLM)
Is an AWS service. It automates the process of managing the lifecycle of **Amazon Elastic Block Store (EBS)** snapshots and **Amazon Machine Images (AMIs)**.DLM allows you to define policies for automatic creation, retention, and deletion of snapshots and AMIs,
## EFS
Managed NFS (network file system) that can be mounted on 100 EC2.
- **Serverless**
- EFS works with **Linux** EC2 instances in **multi-AZ** 
- **Highly available, scalable, expensive** (3x gp2)
- Can be mounted to multiple EC2 instances **across AZs**
- **Pay per use** (no capacity provisioning)
- Uses **security group** to control access to EFS
- Uses **POSIX Permissions** to control access from hosts by IAM User or Group
- **Auto scaling** (up to PBs)
- Lifecycle management feature to move files to **EFS-IA** after N days
- EFS is not encrypted by default
### Storage Classes 
- **Standard** - for frequently accessed files
- **Infrequent access (EFS-IA)** - cost to retrieve files, lower price to store

### EFS Performance Mode
EFS offers two performance modes: **General Purpose** and **Max I/O**. Each mode balances performance and scalability differently based on workload needs.
- **General Purpose** (default)
    - latency-sensitive use cases (web server, CMS, etc.)
- **Max I/O**
    - higher latency & throughput (big data, media processing)
### EFS Throughput Modes
EFS also offers two throughput options that allow you to fine-tune the performance and cost balance for your workload:
- **Bursting** (default)
    - Throughput: 50MB/s per TB
    - Burst of up to 100MB/s.
- **Provisioned**
    - Fixed throughput (provisioned)
### Mount target
  Is essentially a network endpoint in your **VPC** that provides access to your **EFS file system**. EC2 instances in the same VPC (or in a peered VPC) can use this mount target to connect to the file system via the **NFS (Network File System)** protocol.
  - **One per Availability Zone**
## Fsx
Allows you to deploy scale feature-rich, high performance file systems in the cloud that can be accessed from your **on-premise infrastructure**. Fsx support a variety of file system protocol.

- Types:
	- **Amazon FSX for NetApp ONTAP**: Property enterprise storage platform known for **handling petabytes** of data
	- **Amazon Fsx for OpenZFS**: provides the features of the **OpenZFS file system**
	- **Amazon FSX for Windows File Server (WFS)**: File storage on a Windows server.
	- **Amazon FSX for Lustre**: Open-source file system for parallel computing (High-performance computing (HPC).

- File Cache: high speed cache for datasets stored anywhere, accelerate bursting workloads
### FSx for Windows

- Shared File System for Windows **(like EFS but for Windows)**
- Supports **SMB** protocol, Windows **NTFS**, Microsoft **Active Directory** integration, ACLs, user quotas
- Built on **SSD**, scale up to 10s of GB/s, millions of IOPS, 100s PB of data
- Supports **Multi-AZ** (high availability)
- Data is backed-up daily to S3
- **Does not integrate with S3** (cannot store cold data)

## AWS BACKUP
Centrally manage and automate backups of AWS services **across regions and accounts**

- Component:
	- **Backup Plan**: A **backup policy** defines the backup schedule, backup window, backup lifecycle.
	- **Backup Vault**: backup are stored in a backup vault following WORM model. **Even the root user cannot delete backups**
	- **AWS backup Audit Manager** is built in reporting and auditing for AWS backup

**USE CASE**: Create an AWS Backup plan to take daily snapshots with a retention period of 90 days.
## Snow Family
AWS Snow Family are storage and compute devices used to phisically move data in or out the cloud **when moving data over the internet or private connection is too slow, difficult or costly.**
- Used when it takes a long time to transfer data over the network
- **Takes around 2 weeks to transfer the data**
- **Snowball cannot import to Glacier directly** (transfer to S3, configure a lifecycle policy to transition the data into Glacier)
- **Pay per data transfer job**

![](images/02763ff5027e9a93ae396a8d9fbf0f11.png)

- **Snowcone**: Portable secure device for edge computing
    - types:
        - **Snowcone**: 8 TB HDD
        - **Snowcone SSD**: 8 TB SSD
    - Two ways of sending data:
        - **Phisically shipping** the device which runs on the device's compute
        - **AWS DataSync** which runs on the device's compute
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

|                        | **AWS Snowcone** | **AWS Snowball Edge Storage Optimized 80 TB** | **AWS Snowball Edge Storage Optimized 210 TB** | **AWS Snowball Edge Compute Optimized** |
| ---------------------- | ---------------- | --------------------------------------------- | ---------------------------------------------- | --------------------------------------- |
| **Usable HDD Storage** | 8 TB             | 80 TB HDD                                     | 210 TB HDD                                     | N/A                                     |
| **Usable SSD Storage** | 14 TB            | 1 TB SSD                                      | 1 TB SSD                                       | 7.68 TB NVMe                            |
| **Usable vCPUs**       | 2 vCPUs          | 40 vCPUs                                      | 52 vCPUs                                       | 52 vCPUs                                |
| **Usable RAM**         | 4 GB             | 80 GB                                         | 208 GB                                         | 208 GB                                  |
| **Storage Clustering** | No               | No                                            | No                                             | Yes, 5-10 nodes                         |
| **256-bit Encryption** | Yes              | Yes                                           | Yes                                            | Yes                                     |
| **HIPAA Compliant**    | No               | Yes, eligible                                 | Yes, eligible                                  | Yes, eligible                           |
- **AWS Snowmobile (not used anymore)**: 45-foot ship container truck. Used to handle 100 PB of data. (not used anymore)
### Data Migration
- **Provides block storage and Amazon S3-compatible object storage**
- Usage process
    1. Request Snowball devices from the AWS console for delivery
    2. **Install the snowball client / AWS OpsHub on your servers**
    3. Connect the snowball to your servers and copy files using the client
    4. Ship back the device when you’re done (goes to the right AWS facility)
    5. Data will be loaded into an S3 bucket
    6. **Snowball is completely wiped**
- Devices for data migration
    - Snowcone
    - Snowball Edge - Storage Optimized
    - Snowmobile
## AWS transfer family
Offers fully managed support for the transfer files over **SFTP, AS2,FTPS and FTP** directly into and out of **Amazon S3 and EFS**.

- Supported Protocols
    - **FTP** (File Transfer Protocol) - unencrypted network protocol
    - **FTPS** (File Transfer Protocol over SSL) - FTP with **SSL/TLS** encryption
    - **SFTP** (Secure File Transfer Protocol) -  uses **SSH** to provide secure but unencryped 
    - **AS2** (Applicability Statement): Enable secure and reliable messaging
## AWS Migration Hub
Is a single place to discover your existing servers, plan migrations, and track the status of each  migration application.

AWS Migration hub can monitor migration using:
- **Application Migration Service (AMS)**
- **Database Migration Service (DMS)**

Tools to migrate:
- **AWS Discovery Agent:** Agent installed on your VM
- **Migration Evaluator Collector** : You submit a request to ask helps from AWS team
## AWS Data Sync
**AWS DataSync** allows you to move **large amounts of data** from your **on-premises NAS or file system** to AWS via **NFS** or **SMB** protocol. Data transfer can occur over the **public internet using TLS encryption** for security, or it can be directed through **AWS Direct Connect** for private, higher-speed connections.
 - Need to install **AWS DataSync Agent** on premises

**Use Case**: 
- Automates moving data between on-premises storage and AWS, as well as between AWS storage services (e.g., **Amazon S3, EFS, FSx**).
- Can also be used to transfer between two EFS in different regions

Other use cases: https://docs.aws.amazon.com/datasync/latest/userguide/what-is-datasync.html
## Database Migration Service (DMS)
**AWS Database Migration Service (AWS DMS)** is a managed migration and replication service that helps move your database and analytics workloads to AWS quickly, securely, and with minimal downtime and zero data loss.
- **Continuous Data Replication** using **CDC (change data capture)**
- Requires **EC2 instance running the DMS software** to perform the replication tasks. If the amount of data is large, use a large instance. If multi-AZ is enabled, need an instance in each AZ.
- **The source database remains available during migration**
![](images/8989ee8fedf49a6a3563591df32f26f0.png)

### Migration Methods
- **Homogenous data migration**
	- When the source and target DB engines are the same (eg. Oracle to Oracle)
	- Migrate data with native database tools. **Eg. pg_dump, pg_restore**
	- You create a migration project in DMS and it will perform the migration using a serverless compute
		- Uses a **pay as you go** model
### Heterogeneous Migration
- When the source and target DB engines are different (eg. Microsoft SQL Server to Aurora)
- Two step process:
    1. Use the **Schema Conversion Tool (SCT)** to convert the source schema and code to match that of the target database
    2. Use the **Database Migration Service (DMS)** to migrate data from the source database to the target database

> [!NOTE]
> **AWS Schema Conversion tool** is used alongside to **DMS** in many cases to automatically convert a source database schema to a target database schema
> For data warehouse you can use the desktop app **AWS schema conversion tool** 

## Storage Gateway
**AWS Storage Gateway**: is a hybrid storage service that allows your on-premises applications to seamlessly use Amazon S3 (Bridge between on-premise data and cloud data in s3). 
- Use example: backups to the cloud
- providing low-latency access to data in AWS for on-premises applications. 

**Local Caching**: AWS Storage Gateway is unique in its support for local caching
### Types of Storage

#### S3 File Gateway
 Amazon S3 File Gateway supports a file interface into s3 and combines a service and a virtual software appliance. you can store and retrieve objects in Amazon S3 using file protocols such as **Network File System (NFS)** and **Server Message Block (SMB)** (same protocol of DataSync).
 
- **Data is cached at the file gateway** for low latency access
- Integrated with **Active Directory (AD)** for user authentication
![](images/7f23bb887af05de3e64dca5caf1bab72.png)
#### Volume Gateway
Allows you to mount s3 as local drive using the **iSCSI** protocol
- Use cases: **Hybrid Cloud, Backup and disaster recovery Migration of application data**
- 2 types: 
- **Stored Volumes:**
    - Provide your on-premises applications with **low-latency access** to their entire datasets.
    - Enable durable off-site backups.
    - **Stored Volumes can be up to 16 TB in size.**
- **Cached Volumes:**
    - Retain a copy of frequently accessed subsets of data locally.
    - Offer substantial cost savings.
    - Provide **low-latency access** to frequently accessed data.
    - **Minimize the need to scale your on-premises storage infrastructure.**
    - **Cached Volumes can be up to 32 TB in size.**

Think of it as a remote drive

![](images/a49cee233ca60e4adf40f97ad4d95e7a.png)
#### Tape Gateway
Stores files onto **Virtual Library Tapes** for backing up you files on very cost effective long term. 
**Durable**, cost effective solution to archive your data in the AWS Cloud. 

- Use **VTL** interface, that helps you leverage existing tape-based backups.
- Tape storage has proven redability **for 30 years**. 
- Long retrival Time

#### Amazon FsX File Gateway
Allows your files to be stored in Amazon FSx Windows FIle Storage (WFS). Allow you

r Windows developers to easily store data in the cloud using the tools they already know

## Instance Store
- Used as **cache**
- **Hardware storage directly attached to EC2 instance** (cannot be detached and attached to another instance)
- **Highest IOPS** of any available storage (millions of IOPS)
- **Ephemeral storage** (loses data when the instance is stopped, **hibernated** or terminated)
- You can't attach instance store volumes to an instance after you've launched it
- **Good for buffer / cache / scratch data / temporary content**
- AMI created from an instance does not have its instance store volume preserved

**Use case:** Running complex simulations in fields like genomics, financial modeling, or fluid dynamics where performance is critical.

> [!NOTE]
> - You can specify the instance store volumes only when you launch an instance. You can’t attach instance store volumes to an instance after you’ve launched it.
> - Can use RAID, the same way EBS does

## Amazon AppFlow
Is managed integration service for data transfer between data sources. Easily exchange data with over 80+ cloud services. By specifing a source destination. **(Easy way to connect services)**

![](images/dc66c94a2bfecb47ffe1edc526f8472a.png)
- **Run on Demand** - User manually run the flow as needed
- **Run on event** - AppFlow runs the flow in response to an event from an SaaS application
- **Run on Schedule** - AppFlow runs the flow on a recurring schedule

**Feature:**
- Create dataflows between application whithin minutes
- **Aggregate data from multiple sources**
- Data can be **encrypted at rest** and in transit
- You can create private flow via AWS Private Link

# Database
## RDS
Amazon Relational Database Service (Amazon RDS) is a web service that makes it easier to set up, operate, and scale a relational database in the AWS Cloud. It provides cost-efficient, resizable capacity for an industry-standard **relational database** and manages common database administration tasks.
![](images/c9cc3d56b9937914dee191b5e9393e9d.png)

**DB instances:** is an isolated database environment running in the cloud
- DB instance **can contain one or multiple user** created database
- Each database has user defined database identifier which forms part of the DNS hostname
- **DB instance Class:** just like ec2 instance class determine available compute memory or a db instance
- DB instance **use Elastic Block Storage (EBS) volumes** to store log and data
- **Maximum storage of 64TB**
- We don't have access to the underlying instance
- **RDS Proxy:** is a fully managed, highly available database proxy for **Amazon RDS** that makes applications **more scalable, more resilient to database failures, and more secure**.
### RDS Storage Auto Scaling 
Automatically scales **storage capacity** in response to growing database workloads, with zero downtime. (only storage capacity)
### Encryption
- **Data-at-Rest Encryption**: RDS uses **AWS Key Management Service (KMS)** to encrypt data at the storage level. Read replicas and snapshot are automatic enabled.
- **Data-in-Transit Encryption**: Data moving between the RDS instance and applications can be secured using **SSL/TLS** to prevent unauthorized interception.
- **Key Management**: AWS KMS handles encryption keys, allowing users to either use default keys managed by AWS or specify their own Customer Master Key (CMK).
- **Compliance and Security**: RDS encryption assists with meeting compliance requirements such as HIPAA, PCI DSS, and FedRAMP, ensuring a secure environment for sensitive data.
- **Limitations**: **Encryption must be enabled at creation**, as enabling it on an existing instance requires creating a new encrypted instance and migrating data.
- **To encrypt an un-encrypted RDS database:**
    - Create a snapshot of the un-encrypted database
    - Copy the snapshot and enable encryption for the snapshot
    - Restore the database from the encrypted snapshot
    - Migrate applications to the new database, and delete the old database
### RDS Backup
- Types
	- **Automated backup:** choose retention period between 0 and 35 days
		- **enabled by default**
		- **store in snapshot of the primary DB in s3**
		- Log backed up every 5 minutes
		- no charge for additional data in automated backups
	- **Manual Snapshot**
		- Taken manually
		- Additional storage charge 

- **Restoring a backup:** creates a new RDS instance and restores data in that instance 
	- Restoring a manual snapshot
	- Restoring Point in time (PITR): similar but provide restore time
- Take long time to restore database: is not imidiate.
#### Scelte architetturali
#### Read Replicas
- **Asynchronous replication**
- Improve **read contention** which means improve performance latency. 
	- **Read contention:** when multiple processes or instances competing for access to the same index or data block at the same time.
- You must have **automatic backups enabled**.
-  **Applications must update the connection string to leverage read replicas**
- Maximum of **5 replicas** of a database
-  Network fee for replication
    - Same region: free
    - Cross region: paid

#### Multi AZ deploment
- **Synchronous replication**
- Only database engine on primary instance is active
- Automatic failover to standby when a problem is detected
- **Connection string does not require to be updated** (both the databases can be accessed by one DNS name, which allows for automatic DNS failover to standby database)
- **Cannot be used for scaling as the standby database cannot take read/write operation**
##### Multi-Az Cluster vs Multi-Az instance

| Feature          | Multi-AZ Cluster                                           | Multi-AZ Instance                                                                                 |
| ---------------- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| **Architecture** | Multiple nodes, distributed across AZs                     | One primary, one standby                                                                          |
| **Availability** | Higher fault tolerance, high availability, faster failover | High availability, slightly slower failover                                                       |
| **Performance**  | Better for read-heavy workloads                            |                                                                                                   |
| **Scalability**  | Horizontal scaling (via read replicas)                     | Limited scaling (one active instance)                                                             |
| **Use Case**     | High-read workloads, high availability                     | Good for applications that need **high availability** but do not require **extra read capacity**. |
| **Cost**         | Higher                                                     | Lower                                                                                             |

### Access Management
- Username and Password can be used to login into the database
- EC2 instances & [Lambda](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/Lambda) functions should access the DB using **IAM DB Authentication** (**AWSAuthenticationPlugin** with **IAM**) - token based access
	- Only works with **MySQL** and **PostgreSQL**
	- Each token has a lifetime of **15 minutes**
	- Network traffic is encrypted in-flight using SSL
	- **Central access management using IAM** (instead of doing it for each DB individually)

### Subnet Group

**DB Subnet Group**: collection of subnet that you create in a VPC and that you then designate for your DB instances.
- Each DB should have subnets in at least two Availability Zones in a given AWS Region.
- RDS will choose a subnet from your subnet group to deploy your RDS Instance
- Subnets in a DB subnet group are either public or private
- For a DB instance to be publicly accessible, all of the subnets in its DB subnet group must be public
![](images/0b709ad82211781811157a693afca5f0.png)

### Amazon RDS monitoring
- **CloudWatch Metrics for RDS**
    - Gathers metrics from the **hypervisor** of the DB instance
        - CPU Utilization
        - Database Connections
        - Freeable Memory
- **Enhanced Monitoring**
	- Gathers metrics from an agent running on the RDS instance
	    - OS processes
	    - RDS child processes
	- Used to monitor different **processes or threads on a DB instance** (ex. percentage of the CPU bandwidth and total memory consumed by each database process in your RDS instance)
## AWS Aurora
**Serverless**, Automated database instantiation and auto-scaling based on actual usage.
- **Aurora costs more than RDS (20% more)**.
- Supports only **MySQL & PostgreSQL**
- Supports Multi AZ
- **Asynchronous Replication** (milliseconds)
- Up to 15 read replicas
- **Backtrack**: restore data at any point of time without taking backups
- **RDS Data API:** allows you to use HTTP to securely query an Aurora database. Unlimited max request per seconds. **Must be enabled** 
### Aurora Global Database
 It is a database spanning multiple regions for global low latency  and high availabilty. Primary cluster is in a separate region.
- Useful to manage fast failover recovery. Entire database is replicated across regions to recover from region failure.
- Designed for **globally distributed applications** with **low latency local reads** in each region
- 1 **Primary Region (read / write)**
- **Up to 5 secondary (read-only) regions (replication lag < 1 second)**
- Up to 16 Read Replicas per secondary region
- Helps for **decreasing latency for clients in other geographical locations**
- **RTO of less than 1 minute** (to promote another region as primary). 

### Durability and Fault Tolerance
- **Aurora Backup and Failover are handled automatically** 
- Snapshots of data can be shared with other AWS accounts
- Storage is self-healing, in that data blocks and disks are continuously scanned for errors and repaired automatically.
- **Automated failover**
	- In case **no replica** is available, Aurora will attempt to **create a new DB Instance** in the **same AZ** as the original instance. This replacement of the original instance is done on a **best-effort basis** and may not succeed.
	- Aurora flips the **CNAME** record for your DB Instance to point at the healthy replica
### Availability
- **Aurora deploys in a minimum of 3 availability zones each contain 2 copies of your data** at all times.
- Maintain 6 copies
- Lose up to 2 copies of your data without affecting write availability.
- Lose up to 3 copies of your data without affecting read availability.
  
### Storage
- A cluster **starts with 10GB** of storage and scale in 10GB increments **up to 64TB or 128 TB** depending on DB engine version. 
- Computing resources can scale up to 32 vCPUs and 244GB of memory.
### Security
- Encryption at rest using KMS (same as RDS)
- Encryption in flight using SSL (same as RDS)
- You can’t SSH into Aurora instances (same as RDS)
- Network Security is managed using Security Groups (same as RDS)

### Endpoint
Allow clients to connect to the database cluster, specifying different purposes for each endpoint type. By default aurora has one reader and one writer enpoint:
- **Writer Endpoint (cluster)**: Provides a single connection point to the Aurora cluster. **Directs requests to the primary** (writer) instance for read/write operations (like DDL). Ideal for applications that require a primary endpoint for mixed operations. But not suitable for handling queries
- **Reader Endpoint**: Distributes read-only traffic across all available Aurora replicas. Useful for scaling read-heavy applications without impacting the primary instance's performance.
- **Custom(Cluster) Endpoint**: Allows you to define and group specific instances (e.g., high-performance replicas) and assign an endpoint to them. Useful for advanced load balancing or specific application requirements.
### Types
#### Aurora Provisioned
You choose the DB instance class for the writer and reader instances based on your expected workload. Aurora provisioned has several options
#### Aurora Serverless V2
Fully manages the autoscaling configuration for Amazon Aurora
- Capacity is adjusted automatically based on application demand
- charge only for resources used
- use case: **Highly variable workloads**
- Does not scale to zero, must maintain at least 0.5 ACU(unit measurement of Aurora to determine cost vs capacity)
#### Aurora Serverless V2(On-demand) vs Provisioned
|                        | Aurora Serverless V2                                       | Aurora Provisioned                                                          |
| ---------------------- | ---------------------------------------------------------- | --------------------------------------------------------------------------- |
| **Scaling**            | Fine-grained, almost instant scaling.                      | Manual scaling; requires planning and downtime.                             |
| **Capacity Range**     | 0.5-128 ACUs, **more flexible**.                           | **Fixed**, based on instance size chosen. Pay also when databse is not used |
| **Scaling Speed**      | Seconds                                                    | N/A (manual intervention required).                                         |
| **Read/Write Scaling** | Independently                                              | Depends on instance type and read replica configuration.                    |
| **Compatibility**      | Broader version support.                                   | Wide version support, depending on instance type.                           |
| **Use Cases**          | Highly variable workloads needing immediate scaling.       | Stable workloads with predictable performance needs.                        |
| **Billing**            | ACUs per second, more granular, + storage.                 | Instance hours + storage.                                                   |
| **Start/Stop**         | Responsive start/stop, cost-saving for intermittent loads. | Manual start/stop.                                                          |
| **Maintenance**        | Minimal downtime, more seamless.******                     | Scheduled maintenance windows.                                              |

> [!NOTE]
> In order to move from one instance to another you need to use **AWS Database Migration**

## DynamoDB
Amazon DynamoDB is a ***serverless***, NoSQL (key/value), fully managed database with single-digit millisecond performance at any scale.  
- **Highly available** with replication **across 3 AZ**.
- **Multi-Region**
- **Single digit millisecond** response time at any scale
- **Maximum size of an item: 400 KB**
- Max 100 table
- DyanmoDB creates partitions for you as your data grow. Creates a partition every 10 GB or when you exceed.
- Supports **Transactions** (either write to multiple tables or write to none)
### Table classes
DynamoDB offers two table classes designed to help you optimize for cost.
-  **DynamoDB Standard table**: is the default, and is recommended for the vast majority of workloads.
- **DynamoDB Standard-Infrequent Access**: table class is optimized for tables where storage is the dominant cost. **Not recommended for high throughtput**
	- **Use cases:** tables that store infrequently accessed data, such as application logs, old social media posts, e-commerce order history, and past gaming achievements
### Capacity
- **Provisioned Mode** (default)
    - Provision read & write capacity
    - **Pay for the provisioned capacity**
	- Auto-scaling option 
		- eg. set RCU and WCU(measures the number of reads and writes) to 80% and the capacities will be scaled automatically based on the workload
- **On-demand Mode**
    - Capacity auto-scaling based on the workload
    - **Pay for what you use (more expensive)**
    - Great for **unpredictable workloads**
### Read Consistency 
When data needs to be updated in all of its copy and keep data consistent
there are different types of read consistency:

| Eventual Consistent Reads (DEFAULT)                                                                | Strongly Consistent Reads                                                                                            |
| -------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| When copies are being updated, it is possible for you to read and be returned an inconsistent copy | When copies are being updated, and you attempt to read, it will not return a result until all copies are consistent. |
| Reads are fast, but there is **no guarantee of consistent**                                        | You have a **guarantee of consistency**, but the trade-off is **higher latency (slower reads)**.                     |
| All copies of data eventually **become generally consistent within a second**                      | All copies of data **will be consistent within a second**                                                            |
- Eventual Consistent Reads: **Asynchronous**
- Strongly Consistent Reads: **Synchronous**
### Primary Keys
**Types:**
- **Simple Primary Key**: Consists of only a **partition key**.
- **Composite Primary Key**: Consists of both a **partition key** and a **sort key**. 

**Partition key**: is a unique attribute used to identify and distribute items across partitions within a table. This key helps DynamoDB manage data efficiently and optimize performance.
- **Single Attribute**: The partition key is a single attribute, such as `UserID` or `OrderID`, that uniquely identifies each item.
- **Importance of High Cardinality**: Choosing a partition key with a wide range of unique values (high cardinality) ensures an even data distribution, improving table scalability and read/write performance.
### Dynamo DB accelerator
- Caches the queries and scans for DynamoDB items
- Solves read congestion (`ProvisionedThroughputExceededException`)
- **Microseconds latency for cached data**
- Doesn’t require application code changes
- 5 minutes TTL(Time to Live) for cache (default)
### DynamoDB Streams
- Ordered stream of notifications of item-level modifications (create/update/delete) in a table
- **Data are kept for 24 hours**
- Destination can be
    - Kinesis Data Streams
    - AWS Lambda
    - Kinesis Client Library applications

## DocumentDB
NOSQL, document database that is **based on MongoDB**: 
- Cluster types:
	- **Instance based Cluster**: manage your instances directly choosing instance type
	- **Elastic Cluster**: clusters automatically scale, you choose vCPU and number of instances per shard
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
Amazon Keyspaces is a fully managed **Apache Cassandra database.** Cassandra is an open-source NoSQL key/value database similar to DynamoDB. 
- **It is a columnar store database.**
- **Serverless**
- Use cases: 
	- Best suited for workloads that need **Cassandra compatibility**
## Neptune
Amazon Neptune is a fast, reliable, fully managed graph database service that makes it easy to build and run applications that work with **highly connected datasets.**
- Highly available across **3 AZ** with up to **15 read replicas**
- **Point-in-time recovery** due to continuous backup to S3
- **Netpune Database**: **2 types**
	- **Provisioned**: you choose an instance type
	- **Serverless**: set a min and max Neptune Capacity units

> [!NOTE]
>  **Neptune Analyses**:  provide capabilites to run large-scale graph analytics algorithms efficiently

## ElastiCache
Amazon ElastiCache is a web service that makes it easy to set up, manage, and scale a distributed **in-memory data store** or cache environment in the cloud

- AWS managed caching service
- **In-memory key-value store** with **sub-millisecond latency**
-  **Using ElastiCache requires heavy application code changes**
 - **Can only be accessible by resources in the same VPC** (to ensure low latency)
- It can be **cross region** if enabled
- Deployment Options:

|            | Serverless                                                     | Standard                                                                       |
| ---------- | -------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Use case   | Unpredictable workloads                                        | Predictable Workloads                                                          |
| Management | Automatically scales                                           | Customer managers cluster and nodes                                            |
| Billing    | <ul><li>Data Stored</li><li>ElasticCache Processing Units</li> | <ul><li>Based on number of nodes</li><li>Based of node type(size of node)</li> |
- Usage:
    - **DB Cache** (lazy loading): cache read operations on a database (reduced latency)
    - **Session Store**: store user's session data like cart info (allows the application to remain stateless)
    - **Global Data Store**: store intermediate computation results
### Caching engines
#### Redis
Open source in-memory database store. Redis acts as caching layer, or a very fast database. is key/value store.
Can perform many different kind of operations on your data. It is very good for leaderboards, and keep track of unread notification data. It is very fast, but arguably not as fast as memcached
![](images/2660c2c94b3f1aa1bb814dfe8332b743.png)
#### MemCached
Is an open-source distributed memory object caching system. It's a caching layer for web-applications. Key/value store.
![](images/acc3dcb94e3c8f29ca4fd926a1715c27.png)
#### Redis vs Memcached

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
##### Memcache caching strategies
- **Lazy Loading**: The application checks the cache for data first; if a cache miss occurs, it retrieves the data from the database, stores it in the cache, and returns it, populating the cache on demand.
- **Write-Through Caching**: Data is written to the cache and the database simultaneously. This ensures that the cache always contains the latest version of the data, reducing the risk of stale data.
- **Adding TTL (Time-to-Live)**: Each cached item is assigned a TTL value, which determines how long it remains in the cache before being automatically invalidated. This **helps prevent stale data and ensures that fresh data is fetched from the database regularly**.
## MemoryDB
Amazon MemoryDB is a durable, in-memory database service that delivers ultra-fast performance. It is purpose-built for modern applications with microservices architectures.

- Redis-compatible
- **Slower writes** but better performance in general.
- **very expensive**
## Amazon Redshift
 **Serverless,** Is an enterprise-class **relational** database based on postgresql. 
 
 - Used for data warehouse and for **OLAP(analytics and data warehousing)**.
 - Storage is done by **column** and not by row. 
 - **Pay as you go based**
 - **Based on postgresql**
 -  **No multi-AZ support** (all the nodes will be in the same AZ)
 - Faster querying than [Athena](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/Athena) due to indexes
 - Need to provision instances as a part of the Redshift cluster (pay for the instances provisioned)
- Uses massively Parallel Processing(MPP)
- Auto-healing feature


- **Security**
	- Dat in transit: **SSL**
	- Data at rest: **AES 256**
	- **Encryption applied using KMS, CloudHSM**
- **Availability**
	- Redshift is Single AZ. In order to reach high availability you need to run multiple redshift cluster in different AZ. Anapshot can be restored o a different AZ 

### Snapshots

- Stored internally in **S3**
- **Incremental** (only changes are saved)
- Can be restored into a new Redshift cluster
- Automated
    - based on a schedule or storage size (every 5 GB)
    - set retention
- Manual
    - retains until you delete them
- Feature to **automatically copy snapshots into another region**
### Loading data into Redshift
- **S3**
    - Use **COPY command** to load data from an S3 bucket into Redshift
    - **Without Enhanced VPC Routing**
        - data goes through the public internet
    - **Enhanced VPC Routing**
        - data goes through the VPC without traversing the public internet
- **Kinesis Data Firehose**
    - **Sends data to S3** and issues a **COPY** command to load it into Redshift
- **EC2 Instance**
    - Using **JDBC driver**
    - Used when an application needs to write data to Redshift
    - Optimal to write data in batches
## Athena
**Serverless**, is a query service to analyze data stored in Amazon S3. It uses standard SQL language to query the files and their content.
Athena can do 2 things:
- Lets you run SQL queries on S3 Buckets
- Interactively, run data analytics, using Apache Spark

- Athena SQL
	- Component
		- **Workgroup**: saved queries which you grant permissions to other user to access
		- **Data source**: a group database 
		- **Database**: a group of tables
		- **Dataset**: raw data of table
		- **Table**: data organized as group
	- **SerDe**: is a serialization and deserialization libraries for parsing data form different format
		- Can override the DDL configuration that you specify in Athena when you create your table
## Quantum Ledger Database
Used to review history of all the changes made to your **application data** over time, serverless. **Difference with Amazon Managed Blockchain**: no decentralization component

- **Fully Managed**
- **Serverless**
- **Immutable Logs**
- Utilizes SHA-256 hashing for secure, verifiable transaction histories.
- Pay only for what you use
- It keeps a full record of all changes to your data that can't be modified or overwritten.

# Computing
## EC2
Amazon Elastic Compute Cloud (Amazon EC2) provides on-demand, scalable computing capacity in the Amazon Web Services (AWS) Cloud. Using Amazon EC2 reduces hardware costs so you can develop and deploy applications faster.
- **Infrastructure as a Service (IaaS)**
- **EC2 User data**: Used to automate **dynamic** boot tasks (that cannot be done using AMIs). **Runs with root privileges**
- **EC2 Naming convention:**
	![](images/b9bbade1373c953fa310518a14a4d093.png)
- **EC2 Lifecycle:** 
	![](images/d7269a7df552cb09b3bc6c85ffee1cd1.png)
- **Hostname**: Resource name to identify resources
- **EC2 Default user name:** The default user name for an operating system managed by AWS will vary based on distribution:
	- When using SSM you will need to change your user to default: ```sudo su -ec2-user```
- **EC2 type of connection:**
	- SSh Client
	- **EC2 instance connect**: short lived ssh keys controlled by IAM policies
	- **Session Manager**: Connect to linux windows via reverse connection
	- **Fleet Manager Remote Desktop**: Connect to a Windows Machine using RDP
	- EC2 Serial console: establishes a serial connection giving you direc access
### Type of instances

### General Purpose
Balance of compute, memory and networking resources
- **Types**: **A1 T2 T3 T3a T4g M4 M5 M5a M5n M6zn M6g M6i Mac**
- **Use-cases:** web servers and code repositories
### Compute Optimized
Ideal for compute-bound applications that benefit from high-performance processors
- **Types**: **C5 C4 Cba C5n C6g C6gn**
- **Use-cases:** scientific modeling, dedicated gaming servers, and ad server engines
### Memory Optimized
Fast performance for workloads that process large data sets in memory
- **Types**: **R4 R5 R5a R5b R5n X1 X1e High Memory z1d**
- **Use-cases:** in-memory caches, in-memory databases, real-time big data analytics
### Accelerated Optimized
Hardware accelerators, or co-processors
- **Types**: **P2 P3 P4 G3 G4ad G4dn F1 Inf1 VT1**
- **Use-cases:** machine learning, computational finance, seismic analysis, speech recognition
### Storage Optimized
High, sequential read and write access to very large data sets on local storage
- **Types**: **I3 I3en D2 D3 D3en H1**
- **Use-cases:** NoSQL, in-memory or transactional databases, data warehousing

***N.B these are not all but only the most important***
### Purchasing Options
- **On-Demand Instances**:
	- This is the default option when create EC2
	- **High cost but no upfront payment** 
	- **pay-as-you-go** model.
	- Recommended for **short-term and un-interrupted** workloads commitment
- **Reserved Instance(1 & 3 years)**: up to 72% save, long workloads/ long workloads with flexible instances. 
	- Types:
		- **Standard**: These provide the most significant discount. Standard Reserved Instances can't be exchanged.
		- **Convertible**: These provide a lower discount than Standard Reserved Instances, but can be exchanged for another Convertible Reserved Instance with different instance attributes.
	- Type of payments: **All upfront, Partial upfront, No upfront**
	- **Limits**: Per month -> 20 Regional RI per region, 20, Zonal RI per zone
	- **RI Marketplace**: allows you to sell your unused **Standard RI (not Convertible)** to recup your RI spend for RI you do not intend or cannot use.
- **Savings Plans (1 & 3 years)** :commitment to an amount of usage, long workload, up to 72%. For more information see [Saving Plan](#saving%20plan) section.
- **Spot Instances** : short workloads, cheap, can lose instances (less reliable), up to 90%
- **Dedicated Hosts** and **Dedicated Instance**:

| **Dedicated Host**                                                                   | **Dedicated Instance**                                                            |
| ------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------- |
| In dedicated host, there is Physical Server-level isolation from other AWS Accounts. | In dedicated instance, there is Instance-level isolation from other AWS Accounts. |
| Pricing is per-host billing.                                                         | Pricing is per-instance billing.                                                  |
| In dedicated host, there is visibility of Sockets, Cores and Host ID.                | In dedicated instance, there is not visibility of Sockets, Cores and Host ID.     |
| Dedicated Host allows you control over placement of instances in the physical host.  | In dedicated instance, you don't have control over placement of instances.        |
| Automatic instance recovery is not supported.                                        | Automatic instance recovery is supported.                                         |
| It supports Bring Your Own License (BYOL).                                           | It does not support Bring Your Own License (BYOL).                                |

**Capacity Reservations** : Is a service of EC2 that allows you to request a reserve of EC2 instance type for a specific Region and AZ
### Instance States
- **Stop**
    - EBS root volume is preserved
- **Terminate**
    - EBS root volume gets destroyed
- **Hibernate**
    - Hibernation saves the contents from the instance memory (RAM) to the EBS root volume
    - Is not possible to enable or disable hibernation for an instance after it has been launched.
    - EBS root volume is preserved
    - The instance boots much faster as the OS is not stopped and restarted
    - When you start your instance:
        - EBS root volume is restored to its previous state
        - RAM contents are reloaded
        - Processes that were previously running on the instance are resumed
        - Previously attached data volumes are reattached and the instance retains its instance ID
    - Should be used for applications that take a long time to start
    - **Not supported for Spot Instances**
    - Max hibernation duration = **60 days**
- **Standby**
    - Instance remains attached to the [ASG](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/Auto%20Scaling%20Group%20(ASG)) but is temporarily put out of service (the ASG doesn't replace this instance)
    - Used to install updates or troubleshoot a running instance
### AMI
 AMIs are the image of the instance after installing all the necessary OS, software and configuring everything.
- It boots much faster because the whole thing is pre-packaged and doesn’t have to be installed separately for each instance.
- Good for static configurations
- **Bound to a region** (can be copied across regions)

> [!NOTE]
>  When the new AMI is copied from region A into region B, it automatically creates a snapshot in region B because AMIs are based on the underlying snapshots.

### EC2 Placement Group
The logical placement of your instances to optimize communication, performance, or durability. **Free**.

**3 types:**
- **Cluster Placement Group: (optimize for network)**
- **Spread Placement Group: (maximize availability)**
- **Partition Placement Group: (balance of performance and availability)**

## ASG(Auto Scaling Group)
Creation of instances based on amount of workload. **High Availability
![](images/29022a8fb0d126136d4092678693c9f5.png)

- **Health Check Replacement:** ASG replaces an instances if is considered unhealty. Two types of health checks ASG can perform
	- **EC2 Health Check (default)**
	- **ELB Health Check**
-  To prevent ASG from replacing unhealthy instances, suspend the **ReplaceUnhealthy** process type
- After a scaling activity happens, the ASG **goes into cooldown period (default 300 seconds)** during which it does not launch or terminate additional instances.
- ASG compensates by **rebalancing** the AZs by **launching new instances before terminating the old ones**,
### Lifecycle Hooks
- Used to perform extra steps before creating or terminating an instance. **Example:**
    - Install some extra software or do some checks (during pending state) before declaring the instance as "in service"
    - Before the instance is terminated (terminating state), extract the log files
- **Without lifecycle hooks, pending and terminating states are avoided**
### Scaling Policies

- **Dynamic Scaling**
	- **Simple Scaling**
		- Scale to certain size on a CloudWatch alarm
		- **example:** **CloudWatch alarm** is triggered for CPU > 70%, then add 2 units or/and (example CPU < 30%), then remove 1
	- **Step Scaling**
		- Also scale when CloudWatch alarm are trigger but more granular control over how the ASG scale. 
		- You can define multiple **steps** in the scaling policy, where each step specifies different scaling actions based on specific ranges of the metric.
	- **Target Tracking Scaling** 
		- ASG maintains a CloudWatch metric and scale accordingly (automatically creates CW alarms)
		-  **Example:** I want the average ASG CPU to stay at around 40%
- **Predictive Scaling** 
	- Triggers scaling by analyzing historical load data to detect daily or weekly patterns in traffic flows
- **Scheduled Scaling** 
	- Scaling actions are based on a predefined schedule, such as increasing capacity during specific times (e.g., peak hours).

> [!NOTE]
> ELB integration: An ELB can be attached to you Auto Scaling Group 
> - Classic Load Balancers are associeted directly to the ASG
> - ALB, NLB or GWLB are associeted indirectly via their Target Group
>  **N.B attached = can use Load Balancer Health check**

### Launch type
- **Launch Configuration** (legacy)
    - Cannot be updated (must be re-created)
    - **Does not support Spot Instances**
- **Launch Template** (newer)
    - Versioned
    - Can be updated
    - **Supports both On-Demand and Spot Instances**
    - Recommended by AWS
### Termination Policy

- Select the AZ with the most number of instances
- Delete the instance with the oldest launch configuration
- Delete the instance which is closest to the next billing hour
## ELB
It is a suite of Load balancer from AWS. A **Load balancer** is a tool to distribute traffic through different servers.
- Rules of traffic
	- Structure:
		- **Listeners**: incoming traffic is evaluated against listeners. Check matches with the Port (ex port 443 or port 80)
		- **Target Groups:** Are logical grouping of possible targets such as specific EC2 instances, IP addresses
### Application Load Balancer
- ALB is designed to balance HTTP and HTTPS traffic.
- It operate at **Layer 7 (OSI Model)**: 
- It allows you to add routing rules to your listeners based on the HTTP Protocol
- Supports **WebSockets and HTTP request** for real time, bi-directional communications
- You **can't assign an Elastic IP** address to an Application Load Balancer
- **Security Groups can be attached to ALBs** to filters requests
- **ACM** can be attached to listeners to server custom domains over **SSL/TLS** for HTTPS
- **Listeners Rules:** rules to decide what to do with the traffic. Generally, the next step is to forward traffic to a Target Group
- Supported protocol:
	- **HTTP**
	- **HTTPS**
	- **gRPC**
	- **HTTP/2:**
### Network Load Balancer
- NLB is designed to balance TCP/UDP.
- It operate at **Layer 4** (OSI Model)
- It can handle millions of requests per second while still maintaining extremely low latency.
- **Lower latency** ~ 100 ms (vs 400 ms for ALB)
- **Global Accelerator** can be placed in front of ALB to improve **global** availability
- Preserves the client source IP
- **1 static public IP per AZ** (vs a static hostname for CLB & ALB)
- **Elastic IP can be assigned to NLB** (helpful for whitelisting specific IP)
- **Use cases:**
	- High-Performance Computing and Big Data Applications
	- Real-Time and Multiplayer Gaming Platforms
	- Financial Trading Platforms
	- IoT and Smart Device Ecosystems
	- Telecommunications Networks
#### Gateway Load Balancer (GWLB)
- Operates at **layer 3** (Network layer) - IP Protocol
- Used to route requests to a fleet of 3rd party virtual appliances like Firewalls, Intrusion Detection and Prevention Systems (IDPS), etc.
- Performs two functions:
    - **Transparent Network Gateway** (single entry/exit for all traffic)
    - Load Balancer (distributes traffic to virtual appliances)
- Uses GENEVE protocol
- Target groups for GWLB could be
    - EC2 instances
    - IP addresses
### Classic Load Balancer (legacy)
- CLB is AWS's first load balancer 
- Can balance HTTP, HTTPS or TCP traffic (not at the same time)

**Note**: Is not reccomended anymore
## AWS Lambda
**Serverless** service that let you run your code on a high-availability compute infrastructure and performs all of the administration of the compute resources.
- Serverless
- **Not good for running containerized applications**
- **2 ways to be called:** Sync invocation or Async invocation
- **Pay per execution time and number.** No charge when your code is not running.

Usage example:
![](images/23880bca90bb901c2d87c8fb8b078af5.png)
### Limit:
#### Time settings:
- Default: 3 seconds
- **Min: 1 second**
- **Max 15 minutes**
#### Storage settings:
- Default: 512 MB
- Min: 512 MB
- Max 10,240 MB (~10 GB)
#### Memory settings: (Increments of 1 MB)
- Default: 128 MB
- Min: 128 B
- Max 10,240 MB (~10 GB)

#### Lambda SnapStart
Tool for Java that can improve startup performance for latency-sensitive applications by up to 10x at no extra cost, typically with **no changes to your function code.**

### Lambda function URL 
A function URL is a dedicated HTTP(S) endpoint for your Lambda function. You can create and configure a function URL through the Lambda console or the Lambda API
## Step functions
With AWS Step Functions, you can create workflows, also called [State machines](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-statemachines.html), to build distributed applications, automate processes, orchestrate microservices, and create data and machine learning pipelines.
![](images/c7438544a217f84003a4917cb4e8dd7e.png)

- 2 types State Machines:
	- **Standard**: general purpose -> reccomended for Long Workload
	- **Express**: for streaming data -> reccomended Short Workload, (streaming, data injection, even driven ecc...)

| Standard workflows                                                                                                                                                          | Express workflows                                                          |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 2,000 per second execution rate                                                                                                                                             | 100,000 per second execution rate                                          |
| 4,000 per second state transition rate                                                                                                                                      | Nearly unlimited state transition rate                                     |
| Priced by state transition                                                                                                                                                  | Priced by number and duration of executions                                |
| Show execution history and visual debugging                                                                                                                                 | Show execution history and visual debugging based on **log level**         |
| See execution history in Step Functions                                                                                                                                     | Send execution history to [CloudWatch](https://aws.amazon.com/cloudwatch/) |
| Support integrations with all services.<br><br>Support optimized integrations with some services.                                                                           | Support integrations with all services.                                    |
| Support _Request Response_ pattern for all services<br><br>Support _Run a Job_ and/or _Wait for Callback_ patterns in specific services (see following section for details) | Support _Request Response_ pattern for all services                        |
- Some use cases:
	- Manage a Batch job , if job fails send message with SNS
	- Manage a Fargate Container
	- Transfer Data Records
	- **Other use cases can be found on the official web page**

- **Step Functions states**: Allows us to pass input and output without any work. Useful when constructing state machines
- **Task workflow state**: Add a single unit of work to be performed by your state machine.
- **Choice workflow state**: Add a choice between branches of execution to your workflow.
- **Parallel workflow state:** Add parallel branches of execution to your workflow.
- **Map workflow state**: Dynamically iterate steps for each element of an input array. Unlike a `Parallel` flow state, a `Map` state will execute the same steps for multiple entries of an array in the state input.
- **Pass workflow state:** Pass state input through to the output. Optionally, filter, transform, and add fixed data into the output
- **Wait workflow state:** Pause your workflow for a certain amount of time or until a specified time or date.
- **Succeed workflow state:** Stops your workflow with a success.
- Fail workflow state: Stops your workflow with a failure.

## AWS Compute Optimizer
AWS Compute Optimizer is a service that analyzes your AWS resources' configuration and utilization metrics to provide you with rightsizing recommendations. 
It reports whether your resources are optimal, and generates optimization recommendations to reduce the cost and improve the performance of your workloads.
## AWS Batch
 AWS Batch helps you to run batch computing workloads on the AWS Cloud. Batch computing is a common way for developers, scientists, and engineers to access large amounts of compute resources. 
 - compared to Lambdas they have **no time limit**
 - Batch jobs are defined as Docker images and run on ECS 
 - **Rely on EBS and EC2 / instance store for disk space**.


# Network & Security
**Concept:**
**Elastic Network Interfaces (ENIs):** Are **virtual network interfaces** that provide networking capabilities to Amazon EC2 instances. An **ENI** functions as a virtual network card
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
	- Limits: 
		- 10000 security groups per region
		- 60 in/outbound rules per security group
		- 16 security group per Elastic Network Interface.
- **NACL vs Security group:**


| Security Group                                                                                                                                               | Network ACL                                                                                                                                           |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Operates at the instance level                                                                                                                               | Operates at the subnet level                                                                                                                          |
| Supports allow rules only                                                                                                                                    | Supports allow rules and deny rules                                                                                                                   |
| Is **stateful**: Return traffic is automatically allowed, regardless of any rules                                                                            | Is **stateless**: Return traffic must be explicitly allowed by rules                                                                                  |
| We evaluate all rules before deciding whether to allow traffic                                                                                               | We process rules in number order when deciding whether to allow traffic                                                                               |
| Applies to an instance only if someone specifies the security group when launching the instance, or associates the security group with the instance later on | Automatically applies to all instances in the subnets it's associated with (therefore, you don't have to rely on users to specify the security group) |

Rules are evaluated starting with the lowest numbered rule. As soon as a rule matches traffic, it's applied immediately regardless of any higher-numbered rule that may contradict it.
**Example** This allow all incoming traffic even if other rules denied it:
![](images/c58a59e29ea5d40621780f17156be53d.png)
- **Route tables:** where network traffic are directed.
	- different types:
		- **Main route tables:** created with VPC cannot be deleted
		- **Custom route table:** toute table that you can create
- You can share a VPC trough the same account using **AWS Resources Acess Manager**
- **Default IP addressing system in VPC is IPv4.** You cannot have ipv6 VPC only and you cannot disable ipv4.

> [!NOTE]
> **There is no concept of Public and Private subnets.** Public subnets are subnets that have: - “Auto-assign public IPv4 address” set to “Yes” - The subnet route table has an attached Internet Gateway
> 
> This allows the resources within the subnet to make requests that go to the public internet. **A subnet is private by default.**


### Elastic IP
Addresses in AWS that always stay the same **(different from that of the EC2 instances)**
	- **IPV6 are already unique so they don't need it**
	- **are charge 1$ for each allocated**
	- You can associate, allocate, disassociate, deallocate, reassociate, recover, customize (with your IPV6 ip)
- **AWS ipv6 support**: provide a solution for the eventual exhaustion of all IPV4 address
- Is it possible to migrate from ipv4 and ipv6 following some rules. (It isn't always possible).
### Subnets

- Sub-ranges of IP addresses within the VPC
- **Each subnet is bound to an AZ**
- Subnets in a VPC cannot have overlapping CIDRs
- **Default VPC only has public subnets** (1 public subnet per AZ, no private subnet)
- **AWS reserves 5 IP addresses (first 4 & last 1) in each subnet**. These 5 IP addresses are not available for use. Example: if CIDR block 10.0.0.0/24, then reserved IP addresses are 10.0.0.0, 10.0.0.1, 10.0.0.2, 10.0.0.3 & 10.0.0.255
- Every subnet that you create is automatically associated with the main route table for the VPC.

> [!NOTE]
>  To make the EC2 instances running in private subnets accessible on the internet, place them behind an internet-facing (running in public subnets) Elastic Load Balancer.
> 
>  **There is no concept of Public and Private subnets.** Public subnets are subnets that have: - “Auto-assign public IPv4 address” set to “Yes” - The subnet route table has an attached Internet Gateway
>  
>  This allows the resources within the subnet to make requests that go to the public internet. **A subnet is private by default.**
> 
> Since the resources in a private subnet don't have public IPs, they need a NAT gateway for address translation to be able to make requests that go to the public internet. NAT gateway also prevents these private resources from being accessed from the internet.

### VPC Peering
Connect two VPC, privately using AWS’ network 
- VPC peering can be between IPv4 or IPv6
- Make them behave as if they were in the same network 
- VPC Peering connection **is not transitive** (2 way)
- Must have **non-overlapping CIDR**
- Data transfer cross AZ or cross Region incurs charges
![](images/7a40900d4d99d364c8e2a6500d1d2bcd.png)
### VPC Endpoints: 
Endpoints is the URL that allow you to connect to AWS Services **using a private network** instead of the public network. 
- This gives you enhanced security and lower latency to access AWS services.
- Do not require public IPV4 and doesn't leave the AWS network
- Eliminates the need for Internet Gateway, NAT Device, VPN connection, AWS Direct
- **Route table is updated automatically**
- **Bound to a region** (do not support inter-region communication)
![](images/9617aeb6db95baabc989deff185dbee6.png)
#### Type of VPC Endpoint
- **Endpoint policy:**  controls access to the service to which you are connecting. 
	- **Example**: allow access to trusted buckets. (You can do it also with bucket policy but in that case you should enable each one of them)

##### Interface Endpoint
 It is a Elastic Network Interfaces (ENI) with a private IP address. They serve as an entry point for traffic going to a supported service. It is used by **AWS PrivateLink** **(Does not include S3 and DynamoDB)**
- **Use Case**: Private connection to AWS services, partner services, and other VPCs without public IPs.
- **Service Integration**: AWS PrivateLink
- **Supported Services**: Many AWS Services
- **Pricing**: 
  - Per hour when provisioned
  - Data processed
- **Traffic Direction**: Bidirectional

##### Gateway Endpoint
Like interface endpoint used by S3 and DynamoDB 
- **Use Case**: Private connections to S3 and DynamoDB from VPC
- **Supported Services**: S3 and DynamoDB 
- **Pricing**: Free
- **Routing Mechanism**: Route table entries for specific destinations
- **Traffic Direction**: Unidirectional
##### GWLB Endpoint
The **Gateway Load Balancer Endpoint** in AWS is a type of **VPC endpoint** that enables you to privately and securely route traffic to and from a **Gateway Load Balancer** (GWLB) without traversing the public internet.
- **Use Case**: Route traffic to third-party virtual appliances like firewalls in another VPC.
- **Supported Services**: Third-party virtual applications
- **Pricing**: 
  - Endpoint hours
  - Data processed
- **Routing Mechanism**: Integrates with GWLB
- **Traffic Direction**: Usually Unidirectional
### AWS Direct Connect
**AWS Direct Connect** is a cloud service that provides a **dedicated, private network connection** between your on-premises infrastructure. **Physical connection**. 
![](images/d38cf2de07bc29de30001536f099dddc.png)
- 2 types:
	- **Lower Bandwith** 50Mbps -500Mbps
	- **Higher Bandwith** 1Gbps, 10Gbps, 100Gbps
- **Data in transit is not-encrypted** but the connection is private (secure)
- More stable and secure than Site-to-Site VPN
- Access public & private resources on the same connection using **Public & Private Virtual Interface (VIF)** respectively
- Your network is co-located with an existing AWS Direct Connect location
-  AWS Partner Network can help setting up the connection
- Support IPV4 and IPv6 
- **Time to setup > 1 month**
#### Connection Types
- **Dedicated Connection**
    - **1 Gbps or 10 Gbps** (fixed capacity)
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
AWS PrivateLink establishes private connectivity between virtual private clouds (VPC) and supported AWS services, services hosted by other AWS accounts, and supported AWS Marketplace services. External VPC need to use network load balancer and your VPC need an **Elastic Network Interface** 
- Used to expose **services** in one VPC to multiple other VPCs, possibly in **different accounts**
Require **NLB (common) or GWLB in the service VPC** and **ENI in the consumer VPC**

![](images/7dd6ebce06ed6ae475bab12da0a8e066.png)
**N.B:** Need an **Interface endpoint** and **Service endpoint** in order to work
### AWS Site-to-site VPN:
Connection between on-premises VPN and VPC.
- **IPSec Encrypted** connection through the public internet
- **IPv6 traffic is not supported** for VPN connections on a virtual private gateway.
- Recommend that you use non-overlapping CIDR blocks for your networks
- If you need to ping EC2 instances from on-premises, make sure you add the **ICMP protocol** on the inbound rules of your security groups
![](images/e5967b387a310ba7519aef1ae65d516f.png)

**Components:**
- **VPN Connection** - secure connection between VPC and on-premises equipment
- **VPN tunnel** - encrypted connection for your data
- **IPSec Encrypted** connection through the public internet
- **Customer gateway (CGW)** - A physical device or software application on the **on premise side** of the Site-to-Site VPN connection. 
	- In order to establish the connection you need to set up an Internet-routable IP address (static) of the customer gateway's external interface.
- **Target gateway**
	- **Virtual private gateway (VGW)**: VPN endpoint on the **VPC side**. Can be attached only to one VPC
	- **Transit gateway**: VPN endpoint on the **VPC side**. Can be attached to multiple VPC
### AWS Client VPN
Connect from **your computer** using OpenVPN to your VPC in AWS and on-premises system.
**Use case**: You travel around the world and need connection to services.
![](images/3e076d28af82b836ea709f273de7d8fa.png)
### Internet Gateway 
Used to **connect public resources to the internet** (use NAT gateway for private resources since they need network address translation)
- Work both with IPV4 and IPV6, and perform network translation for instances **that have been assigned public IPV4 addresses.**
![](images/24424185b20ecd96c8c0bdf9f960039b.png)

#### Egress-Only Internet Gateways 
- VPC component that allows **outbound** communication **over IPv6** from instances in your VPC to the Internet, **and prevents the Internet from initiating an IPv6 connection with your instances.**
- An egress-only Internet gateway is stateful.
- **You cannot associate a security group with an egress-only Internet gateway.**
- You can use a network ACL to control the traffic to and from the subnet for which the egress-only Internet gateway routes traffic.
### NAT Gateway 
**What is Network Address Translation (NAT)?**
A method of mapping an IP address space into another by modifying network address information in the IP header of packets while they are in transit across a traffic-routing device

**NAT Gateway**: Allow your instances in your Private Subnets to access the internet while remaining private

![](images/aceb08550edeb84adc14ed3e1dd394e6.png)
- 1 Nat gateway per subnet
- AWS managed NAT with **bandwidth autoscaling** (up to 45Gbps)
- **Uses an Elastic IP** and Internet Gateway behind the scenes
- **Bound to an AZ**
- **Cannot be used by EC2 instances in the same subnet** (only from other subnets)
- **2 types**: Public and Private
- **Cannot be used as a Bastion Host**
- **NAT Instance (not very used anymore):** is an AWS managed IAM to launch a NAT onto an individual EC2 instances. NAT Instances required the customer to handle scaling

### Transit Gateway: 
[AWS Transit Gateway](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html) provides a hub and spoke design for connecting VPCs and on-premises networks as a fully managed service without requiring you to provision third-party virtual appliances. No VPN overlay is required, and AWS manages high availability and scalability.

- Transit Gateway leverages AWS Resource Manager (RAM)
- **Operates at the regional level.**
- You can attach up to 5000 VPCs to each gateway
	- An ENI will be provisioned in target VPCs to facilitate VPC to Transit Gateway communication
- Each attachment can handle up to 50 Gbits/second of traffic
- Support IP multicast
- You can peer connection with other Transit Gateways

	![](images/84cc536b3f5e2a53486fff13eec131f4.png)

## Bastion Hosts
In AWS, a **bastion host** is typically an EC2 instance deployed in a **public subnet** within a **Virtual Private Cloud (VPC)**. This host serves as a secure entry point, allowing administrators to access resources located in **private subnets** that are otherwise isolated from the internet.

- A EC2 instance running in the public subnet (accessible from public internet), to allow users to SSH into the instances in the private subnet.
    - ![attachments/Pasted image 20220512100455.jpg](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/attachments/Pasted%20image%2020220512100455.jpg)
- Security groups of the private instances should only allow traffic from the bastion host.
- Bastion host should only allow port 22 traffic from the IP address you need (**small instances are enough**)

#### High Availability
- 2 options
    - Run 2 Bastion Hosts across 2 AZ
    - Run 1 Bastion Host across 2 AZ with ASG 1:1:1
- Routing to the bastion host
    - If 1 bastion host, use an elastic IP with EC2 user-data script to access it
    - If 2 bastion hosts, use a public-facing NLB (layer 4) deployed in multiple AZ. Bastion hosts can live in the private subnet (more secure)
- Can’t use ALB as it works in layer 7 (HTTP protocol) and SSH works with TCP
- Diagram
    - ![attachments/Pasted image 20220513222559.jpg](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/attachments/Pasted%20image%2020220513222559.jpg)
### Network Address Usage(NAU)
Is a metric applied in your VPC to help you plan for and monitor the size of your VPC, in a way that you don't run out of space for your VPC.
## Route 53
Amazon Route 53 is a highly available and scalable Domain Name System (DNS) web service. You can use Route 53 to perform three main functions in any combination: **domain registration, DNS routing, and health checking.**

- **Hosted Zones**: Container for records sets, define how to route traffic for a specific domain or subdomains
	- **Public**: How you want to route traffic through the Internet. 
	- **Private**: How you want to route traffic through an Amazon VPC. 
- **Records set**: collection of records which determine where to send traffic
	- Always changed using batch via the API
	- **Record Alias:** AWS Functionality. Will route traffic to specific AWS resources. 
		- **A** - maps a hostname to IPv4
		- **AAAA** - maps a hostname to IPv6
		- **CNAME** - maps a hostname to another hostname
		    - The target is a domain name which must have an A or AAAA record
		    - **Cannot point to root domains (Zone Apex)** Ex: you can’t create a CNAME record for `example.com`, but you can create for `something.example.com`
		- **NS** (Name Servers) - controls how traffic is routed for a domain
		- **Alias Target** can point to:
			- CloudFront
			- Elastic Beanstalk environment
			- ELB load balancer
			- S3 website endpoint
			- Resource record set
			- VPC endpoint
			- API Gateway Endpoint custom regional API
- **Traffic Flow**: Visual tool, lets you create sophisticated routing configurations for your resources using routing types (50$ per policy -> very expensive)
### Routing Policies
- **Simple Routing Policies**: Most basic Routing policies
	- You have 1 record and provide multiple IP addresses
	- When multiple values are specified for a record, Route53 will return values back to the user in a random order
	- **No health check**
	- ***Example***: if you had a record for www.exampro.co with 3 different IP address values, users would be directly randomly to 1 of them
- **Weighted Routing Policies**: Let you split up traffic based on different weight assigned
	- This allows you to send a certain percentage of overall traffic to one server and have other traffic to be directed to a completley different server
	- Health check
- **Latency Based Routing Policies:** allows you to direct traffic based on the lowest network latency possible for your end-user based on region
	- Can be used for **Active-Active failover** strategy (two resources active at the same time)
- **Failover Routing Policy Policies:** allow you to create **active passive/setup** in situations where you want a primary site in one location and a secondary data recovery. 
	- Automatically monitors health checks from your primary site
- **Geolocation Routing Policies**: allow you to direct traffic based on the geographic location of where **the request originated from**
- **Geoproximity routing polices**: allow you to direct traffic based on the geographic location of **your users and your AWS resources**.  It routes traffic to the closest resource that is available
- **Multi-Value Answer Policies:** let you configure Route 53 to return multiple values such as IP addresses for your web servers, in response to DNS queries.

### Health Checks (not free): 
- HTTP Health Checks are o**nly for public resources**
- Checks health every **30s** by default. Can be reduced to every **10s**
- A health check can **initiate a failover** if the status is returned unhealthy
- A CloudWatch Alarm can be created to alert you of status unhealthy
- Can create up to **50 health checks** for AWS endpoints within or linked to the same AWS account.
- 3 types:
	- Monitor an endpoint (application or other AWS resource)
	- Monitor other health checks (**Calculated Health Checks**)
	- Monitor CloudWatch Alarms (to perform health check on private resources)
### Other Feature and tools
- **Route 53 Resolver:** is a DNS server that allows you to resolve DNS queries between your on-premise network and your VPC
- **DNSSEC**: Are a suite of extensions specifications by the Internet Engineering Task Force (IETF) for securing data exchanged in the Domain Name System (DNS) in Internet Protocol (IP) networks.
- **Zonal Shift**: is a capability in **Route 53 Application Recovery Controller** (Route 53 ARC). Shift a load balancer resource away from an impaired Availability Zone with a single action. Only supported by ALB an NLB
- **Route 53 Profiles**: lets you apply and manage DNS related Route 53 configurations accross many VPCs and in differente AWS accounts

## AWS Global Accelerator

Improve global application **availability** and performance using the AWS global network (Can find the optimal path from the end user to your web servers) 
- Leverage the AWS internal network to optimize the route to your application 
-  **Does not cache anything at the edge location**
-  **2 Static Anycast IP** are created for your application and traffic is sent through **Edge Locations
- 2 types Routing: 
	- **Standard**: automatically route to the nearest healthy endpoint
	- **Custom**: Route specifies EC2 Instances
- Components: 
	- **Listeners**: Listens for traffic on a specific port and sends traffic to a endpoint group
	- **Endpoint Groups**: collection of endpoint in a region
	- **Endpoints**:  can be Network Load Balancers, Application Load Balancers, EC2 instances, or Elastic IP addresses.
### Disaster recovery
- Global Accelerator performs **health checks** for the application
- **Failover in less than 1 minute** for unhealthy endpoints
### Security
- Only 2 static IP need to be whitelisted by the clients
- Can be integrated with **AWS Shield for DDoS protection**
#### Unicast vs Anycast IP
- **Unicast IP**: one server holds one IP address
- **Anycast IP**: all servers hold the same IP address and the client is routed to the nearest host with that IP
## Cloud Front
![](images/20aa7d42bc383b9a63c8e7b473092fcf.png)
- Cloud Front is a **CDN**
	- **CDN(Content Delivery Network):** A distributed network of servers that delivers web pages and content to users **based on their geographical location**, the origin of the webpage and a content delivery server.
- **Global service**
- **Edge Locations are present outside the VPC** so the origin's **Security group must be configured to allow inbound requests** from the list of public IPs of all the edge locations.
- Supports HTTP/RTMP protocol (**does not support UDP protocol**)
- **Caches content at edge locations**, reducing load at the origin
- **Geo Restriction feature**
- Improves performance for both cacheable content (such as images and videos) and dynamic content (such as API acceleration and dynamic site delivery)****
- To block a specific IP at the CloudFront level, deploy a **WAF**
- Supports **Server Name Indication (SNI)** to allow SSL traffic to multiple domains
### Origin

- **S3 Bucket**
    - For distributing static files
    - **Origin Access Identity (OAl) or Origin Access Control (OAC)** allows the S3 bucket to only be accessed by CloudFront
    - Can be used as ingress to upload files to S3
- **Custom Origin** (for HTTP) - need to be publicly accessible on HTTP by public IPs of edge locations
    - EC2 Instance
    - ELB
    - S3 Website (may contain client-side script)
    - On-premise backend

> [!NOTE]
>  To restrict access to ELB directly when it is being used as the origin in a CloudFront distribution, **create a VPC Security Group for the ELB and use AWS Lambda to automatically update the CloudFront internal service IP addresses when they change.**

### Signed URL / Cookies
- Used to **make a CloudFront distribution private** (distribute to a subset of users)
- **Signed URL** ⇒ access to **individual files**
- **Signed Cookies** ⇒ access to **multiple files**
- Whenever we create a signed URL / cookie, we attach a policy specifying:
    - URL / Cookie Expiration (TTL)
    - **IP ranges** allowed to access the data
    - Trusted signers (which AWS accounts can create signed URLs)

#### Use case comparison

Use **signed URLs** for the following cases:
- You want to use an **RTMP(_Real Time Messaging Protocol_)** distribution. Signed cookies aren't supported for RTMP distributions.
- You want to restrict access to individual files, for example, an installation download for your application.
- Your users are using a client (for example, a custom HTTP client) that doesn't support cookies.
Use **signed cookies** for the following cases:
- You want to provide access to multiple restricted files, for example, all of the files for a video in HLS format or all of the files in the subscribers' area of a website.
### Pricing

- Price Class All: all regions (best performance)
- Price Class 200: most regions (excludes the most expensive regions)
- Price Class 100: only the least expensive regions
### Origin Groups(failover purpose)

An origin group includes two origins **(a primary origin and a second origin to failover to)** and a failover criteria that you specify. **You create an origin group to support origin failover in CloudFront.**
- Provides **region-level** High Availability
- **Use when getting 504 (gateway timeout) Error**
### Field-level Encryption
Field-level encryption allows you to enable your users to securely upload sensitive information to your web servers. The sensitive information provided by your users is encrypted at the edge, close to the user
### Lambda Edge
Lambda@Edge is a feature of Amazon CloudFront that lets you run code closer to users of your application, **which improves performance and reduces latency.**
You can perform various tasks such as:
- **Modifying HTTP headers**
- generating dynamic responses
- implementing security measures, and customizing content based on user preferences, device type, location, or other criteria.
![](images/ad11c50d518ac6bb8e7b9846fd9c89e7.png)

### CloudFront Functions types
- **CloudFront Functions:** Designed for lightweight, short-duration tasks, primarily focused on manipulating and inspecting HTTP requests and responses. Common use cases include URL rewrites, header modifications, simple redirects, and bot filtering.
- **Lambda@Edge:** Intended for more complex logic, supporting a broader set of use cases like content transformation, A/B testing, user authentication, and data personalization. Lambda@Edge allows for more advanced compute capabilities than CloudFront Functions.
## AWS Shield
AWS Shield Standard and AWS Shield Advanced provide protections against Distributed Denial of Service (DDoS) attacks for AWS resources at the network and transport layers **(layer 3 and 4)**
#### AWS Shield Standard: 
- Free service that is activated for every AWS customer 
- Provides protection from attacks such as **SYN/UDP Floods, Reflection attacks and other layer 3/layer 4 attacks** 
#### AWS Shield Advanced
- Optional DDoS mitigation service ($3,000 per month per organization) 
- **Protect against more sophisticated attack on:**
	- Amazon EC2
	- Elastic Load Balancing (ELB), 
	- Amazon CloudFront
	- AWS Global Accelerator
	- Route 53
- **24/7 access to AWS DDoS response team (DRP)** 
- **Protect against higher fees during usage spikes due to DDoS**
## WAF
Protects your web applications from common web exploits **Layer 7**. ex. **SQL Injection** and **Cross-Site Scripting (XSS)**
- **ALLOW and DENY** rules based on the contents of HTTP request. 
- Can only be deployed on:
	- **Application Load Balancer**
	- **API Gateway**
	- **CloudFront**
- WAF contains **Web ACL (Access Control List)** containing rules to **filter requests** based on:
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
- It follows **FIPS (Federal Information Processing Standards)**: US and Canadian governament standard for cryptographic modules.
- **Supports both symmetric and asymmetric encryption**
-  Good option to use with **SSE-C** encryption
- CloudHSM clusters are spread across **Multi AZ (high availability)**
- **2 types:**
	- **Multi-tenant** (FIPS 140-2 Level 2 Compliant): 
	- **Single-Tenant** (FIPS 140-2 Level 3 Compliant):

> [!NOTE]
> It is easy to migrate aws key between cloudHSM.

## Amazon GuardDuty
Intrusion Detection System and Intrusion Protection System.(IDS)

- Intelligent **Threat discovery** to protect your AWS Account. 
- Uses Machine Learning algorithms, **anomaly detection**, 3rd party data.
- **Uses EventBridge and lambda funtion to make action**.
- Monitors:
    - **CloudTrail Logs** (unusual API calls, unauthorized deployments)
    - **VPC Flow Logs** (unusual internal traffic, unusual IP address)
    - **DNS Logs** (compromised EC2 instances sending encoded data within DNS queries)
    - **EKS Audit Logs** (suspicious activities and potential EKS cluster compromises)
## AWS Firewall Manager
 AWS Firewall Manager allows  you to centrally configure and manage firewall rules accross accounts and applications. 
 - **Must be a member of AWS organizzation**
 - **Must be AWS Firewall Manager administrator**
 - **Must have AWS Config enabled for your accounts and region**
Does not support NACL as of now
## AWS Inspector
Amazon Inspector is an **automated** security assessment service that helps improve the security and compliance of applications deployed **on your Amazon EC2 instances**. Amazon Inspector automatically assesses applications for exposure.
## Amazon Macie
Amazon Macie is a fully managed data security and data privacy service that uses machine learning and pattern matching to discover and protect your sensitive data in AWS. **Work alongside CloudTrail**
## AWS Security Hub
Central security tool to manage security across several AWS accounts and **automate** security checks. Allow you to generate a security score to determine your security posture. Allows you to enable standards(collection of security control)

## KMS
KMS makes it easy for you to create, control and rotate encryption keys in AWS.
- **Regional service (keys are bound to a region)**
- Is **multi-tenant**: There are multiple customers that are using the same piece of hardware. In this way is it possible to reduce the cost of the service. **Each part is isolated between users.**
-  **Encrypt up to 4KB of data per call (if data > 4 KB, use envelope encryption)**
- Need to set **IAM Policy & Key Policy** to allow a user or role to access a KMS key
- Audit key usage with CloudTrail
#### Customer Master Key (CMK)

|                          | **Customer managed keys**                                                                                                                                         | **AWS managed keys**                                                                                                                                              | **AWS owned keys**                                                                  |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| **Key policy**           | Exclusively controlled by the customer                                                                                                                            | Controlled by service; viewable by customer                                                                                                                       | Exclusively controlled and only viewable by the AWS service that encrypts your data |
| **Logging**              | CloudTrail customer trail or event data store                                                                                                                     | CloudTrail customer trail or event data store                                                                                                                     | ***Not viewable by the customer***                                                  |
| **Lifecycle management** | Customer manages rotation, deletion and Regional location                                                                                                         | AWS KMS manages rotation (annual), deletion, and Regional location                                                                                                | AWS service manages rotation, deletion, and Regional location                       |
| **Pricing**              | Monthly fee for existence of keys (pro-rated<br><br>hourly). Also charged for key usage                                                                           | No monthly fee; but the caller is charged for API usage on these keys                                                                                             | No charges to customer                                                              |
| **Deletition**           | Deletion has a waiting period (**pending deletion state**) between **7 - 30 days** (default 30 days). The key can be recovered during the pending deletion state. | Deletion has a waiting period (**pending deletion state**) between **7 - 30 days** (default 30 days). The key can be recovered during the pending deletion state. |                                                                                     |
- **CloudHSM Keys (custom keystore):** 
	- Keys generated from your own CloudHSM hardware device
	- Cryptographic operations are performed within the CloudHSM cluster


#### Key Policies
**Cannot access KMS keys without a key policy**
- **Default Key Policy**
    - Created if you don’t provide a specific Key Policy
    - Complete access to the key for the root user ⇒ any user or role can access the key (most permissible)
- **Custom Key Policy**
    - Define users, roles that can access the KMS key
    - Define who can administer the key
    - Useful for cross-account access of your KMS key

> [!NOTE]
> It is used for the encryption of almost every service in aws.

## AWS Certificate Manager (ACM)
 Let’s you easily provision, manage, and deploy SSL/TLS Certificates. 
 - Is it possible to perform 3 actions:
	 - **Create Public** certificate
	 - **Create Private** certificate
	 - **Import third party certificate** certificate
 - ACM can be attached to the following AWS resources: 
	 - **Elastic Load Balancer**
	 - **CloudFront, API Gateway**
	 - **Elastic Beanstalk.**
![](images/e5dfd27669182f68293f8819ceeebbab.png)
 
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
- **Cognito Identity Pools**: Provide temporary credential for users. To connect with **third party app** like Google.
- **Cognito Sync**: Sync user data accross all devices
## Amazon Detective
Amazon Detective analyzes, investigates, and quickly identifies the root cause of security issues or suspicious activities (using ML and graphs). 
## Directory Services
 Used to extend the AD network by involving services like EC2 to be a part of the AD to share login credentials.
 
**Well known services:**
- Domain Name Service (DNS), Microsoft Active Directory, Apache Directory Server, Oracle Internet Directory, Open LDAP, CLoud Identity, JumpCloud

**AWS Directory service**s: provides multiple ways to use Microsoft Active Directory. 
- **AWS Managed Microsoft AD**: A full feature managed version of MS active directory
- **Simple AD**:  AD-compatible managed directory on AWS (**cannot be joined with on-premise AD**). Not available for all the regions.
- **AD Connector**: a proxy service to connect your existing on-premise AD Directory
## Secret Manager
Store and rotate automatically database credentials. Enforce encryption at rest by using KMS. It as a cost. For credential access monitor you can use Cloud Trail.

You can setup automatic rotation, is not enabled by default. 
(Under the hood it uses lambda)
# Utils
## QuickSight
***Serverless*** Machine learning-powered business intelligence service to create interactive dashboards
## Elastic Map Reduce (EMR)

Amazon EMR is the industry-leading cloud big data solution for Big Data (petabyte-scale) processing , interactive analytics, and machine learning using open-source frameworks such as Apache Spark, Apache Hive, and Presto

- Uses **Hadoop**, an open-source framework, to distribute your data and processing across a **cluster of 100s of EC2 instances**.
- Supports open-source tools such as **Apache Spark**, **HBase**, **Presto**, **Flink**, etc.
- EMR takes care of all the provisioning and configuration
- **Auto-scaling**
- Integrated with **Spot Instances**
- Can be used to process large amounts of log files
## Disaster Recovery
Disaster recovery refers to the strategies and processes implemented to protect and restore a company’s IT infrastructure, data, and operations after a disruptive event.

- **Recovery Point Objective (RPO)**: how often you backup your data (determines how much data are you willing to lose in case of a disaster)
- **Recovery Time Objective (RTO)**: how long it takes to recover from the disaster (down time)

![](images/b8348de23a4b2fd5e3ed01a5d23da7d0.png)
### Strategies
![](images/673d1fd1f954a496c1c654b67b9624c4.png)
#### Backup & Restore
- **High RPO (hours)**
- Need to spin up instances and restore volumes from snapshots in case of disaster => **High RTO**
- Cheapest & easiest to manage

#### Pilot Light
- **Critical parts of the app are always running** in the cloud (eg. continuous replication of data to another region)
- **Low RPO** **( several minutes)**
- Critical systems are already up => **Low RTO**
- **Ideal when RPO should be in minutes and the solution should be inexpensive**
- DB is critical so it is replicated continuously but EC2 instance is spin up only when a disaster strikes

#### Warm Standby

- A complete backup system is up and running at the minimum capacity. This system is quickly scaled to production capacity in case of a disaster.
- Very low RPO & RTO (minutes)
- Expensive

#### Multi-site active/active

- A backup system is running at full production capacity and the request can be routed to either the main or the backup system.
- Multi-data center approach
- Lowest RPO & RTO (minutes or seconds)
- Very Expensive

## CloudFormation
- **Infrastructure as Code (IaC)** allows us to write our infrastructure as a config file which can be easily replicated & versioned using Git
- **Declarative** way of outlining your AWS Infrastructure (no need to specify ordering and orchestration)
- **Resources within a stack is tagged** with an identifier (easy to track cost of each stack)
- **Ability to destroy and re-create infrastructure on the fly**
- Deleting a stack deletes every single artifact that was created by CloudFormation (clean way of deleting resources)

### CloudFormation Templates
An AWS CloudFormation template is a YAML file that defines the AWS resources you want to create, update, or delete as part of a stack.

- **Templates have to be uploaded in S3** and then referenced in CloudFormation
- To update a template, upload a new version of the template replacing the older
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


> [!NOTE]
> - You can associate the **CreationPolicy** attribute with a resource to prevent its status from reaching **CREATE_COMPLETE** until CloudFormation receives the specified number of **cfn-signal** notifications or the timeout period is exceeded. 
> - Use CloudFormation with securely configured templates to ensure that applications are deployed in secure configurations. 

### Stack Sets

- Create, update, or delete stacks across **multiple accounts and regions** with a single operation
- When you update a stack set, all associated stack instances are updated throughout all accounts and regions.

### Updating Stacks

- CloudFormation provides two methods for updating stacks:
    - **Direct update**
        - CloudFormation immediately deploys the submitted changes. You cannot preview the changes.
    - **Change Sets**
        - You can preview the changes CloudFormation will make to your stack, and then decide whether to apply those changes.

## Security Token Service (STS)
**AWS Security Token Service (STS)** is a service that allows you to **generate temporary, limited-privilege credentials** for AWS resources. 
- Ideal for scenarios where you want to **limit access duration and permissions**
## AppSync
AWS AppSync creates **serverless** GraphQL and Pub/Sub APIs that simplify application development through a single endpoint to securely query, update, or publish data.

 **GraphQL**: is an open-source agnostic query adaptor that allows you to query data from different data sources. GraphQL is used to build APIs where clients will send a query for nested data. 
 
Can create 2 types of API
**API Types:**
- **GraphQL APIs:** single API from multiple data sources
- **Merged APIs:** a collection of GraphQL APIs that act as one API.
	- Useful when you have multiple teams that manage their own API
## Elastic Transcoder
AWS Elastic Transcoder is a fully managed service that simplifies the conversion of media files into formats suitable for playback on various devices.

- **Automatic Scaling**: Efficiently handles varying transcoding workloads.
- **Easy Integration**: Works seamlessly with AWS services like Amazon S3 and CloudFront.
- **Cost-Effective**: Utilizes a pay-as-you-go pricing model with no upfront costs.
## SNS
 Amazon Simple Notification Service (Amazon SNS) is a managed service that provides message delivery from publishers to subscribers (also known as _producers_ and _consumers_). In this way is possible to send one message to many receivers. 
 - **Used to decouple microvservices.**
- **In-Transit** Encryption enabled by **default** done by using Https API
- **At-rest encryption using KMS keys (optional)**

**N.B Respond to event and immediately send notification**
#### Topics:
Allows you to group multiple subscriptions together.
- 2 types: **Standard** and **FIFO**

|                      | **Standard**                                                                                                                    | **First-in-First-Out**                                                                                                 |     |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | --- |
| **Throughput**       | High throughput                                                                                                                 | Lower throughput compared to Standard                                                                                  |     |
| **Delivery**         | At least once (possibility of duplicates)                                                                                       | Exactly once (no duplicates)                                                                                           |     |
| **Ordering**         | No ordering guarantees                                                                                                          | Messages are delivered in the order they are sent within a message group                                               |     |
| **Use-Case**         | Where the volume of messages is high and exact ordering/delivery isn't critical: <ul><li>alerts</li><li>notifications</li></ul> | Where the order and exact delivery are crucial: <ul><li>banking transactions</li><li>ordered data processing</li></ul> |     |
| **Message Grouping** | N/A                                                                                                                             | Supports message grouping, allowing multiple ordered streams within the same topic                                     |     |

- **Messages:** Is it possible to programmatically publish messages to SNS Topic. 
	- **Maximum limit** of message dimension: 256KB. 
	- **Message bigger than 256KB can be publish using Amazon SNS Extended Client library (Max: 2GB).** 
- **Subscription:** A subscription can only subscribe to one protocol and one topic: HTTP/S, Email, Amazon SQS, AWS Lambda, SMS, Platform application endpoint
- **Filter policy:** allows you to filter a subset a messages only to be delivery. 
	- JSON policy used to filter messages. 
	- Each subscriber will have its own filter policy
- **Message Data Protection**: Message data protection safeguards the data that's published to your Amazon SNS Topic. 
- **Delivery Policy**: defined how SNS **retries** the delivery of messages when **server-side errors** occur. Each delivery protocol has its own delivery policy.
## SQS
![](images/c901d901a23d887a5475bf74418aa727.png)
**Message:** small data package (not necessarly intended sms or email).
### SQS
Amazon Simple Queue Service (SQS) is a fully managed AWS service that enables reliable, asynchronous message passing between application components to help decouple and scale systems.

**Use Case**: 
- You need to queue up transaction emails to be sent e.g. Signup, Reset Password.
SQS Queue.
- You have a website that offers a free and a premium account that guarantees faster processing.
	- **Solution:** Create an SQS queue for free members and another one for premium members. Configure your EC2 instances to consume messages from the premium queue first and if it is empty, poll from the free members' SQS queue.


- The consumer polls the queue for messages. 
- Once a consumer processes a message, **it is important to deletes it, in order to avoid duplicates**
-  **Max message size: 256KB**. Bigger message needs **Amazon SQS Extended Client library** (Max: 2GB).
- **Default message retention: 4 days (max: 14 days)** after that message are automatically deleted.
- **Consumers could be EC2 instances or Lambda functions**
- Message are grouped by ID
- Is possible to attach an ASG to the consumer instances
#### Types of Queue
- **Standard:** allows you to send nearly unlimited number of transactions per second, but message are out of order. It has low latency. 
	- Unlimited throughput (publish any number of message per second into the queue)
	- Low latency (<10 ms on publish and receive)
	- Can have duplicate messages (at least once delivery)
	- Can have out of order messages (best effort ordering)
- **FIFO:** guarantees the order of messages when being consumed. 
	- **Throughput:** 300 transaction per second. 3000 with batching
	- no duplicates
	- order by id
	- 10 message read a time.
#### Encryption
- In-flight encryption using HTTPS API
- At-rest server-side encryption:
    - **SSE-SQS**: keys managed by SQS
    - **SSE-KMS**: keys managed by KMS
- Client-side encryption

#### SQS Dead letter Queue
A **Dead Letter Queue (DLQ)** in Amazon **SQS (Simple Queue Service)** is a specialized type of queue used to store messages that have failed to be processed successfully after a certain number of attempts. It is essentially a mechanism for handling and isolating problematic messages that cannot be processed by the consumer for some reason.

- After the `MaximumReceives` threshold is exceeded, the message goes into the DLQ
- **Redrive to Source** - once the bug in the consumer has been resolved, messages in the DLQ can be sent back to the queue (original queue or a custom queue) for processing
- Prevents resource wastage
- Recommended to set a high retention period for DLQ (14 days)

![](images/6eb92718f517c23396d5e0082a72a2ad.png)
#### Configurations

- **Access Policy(resource based policy):** Allows you to grant other principals permission to the SQS Queue.
- **Message Visibility Timeout:** is a period of time message will be invisible after they are read/consumed by an application to avoid being read and processed by other applications. **Default is 30 second**
- **Delay Queues:** let you postpone the delivery of new messages to consumers for a number of second when your app needs more time. Message remain invisible to consumers for the duration of the delay period.
- **Short vs Long Polling:** Polling is the method by which we retrive messages from the queues

|                 | **Short Polling (default)**                                                    | **Long Polling**                                                        |
| --------------- | ------------------------------------------------------------------------------ | ----------------------------------------------------------------------- |
| **Description** | Returns messages immediately, even if the message queue being polled is empty. | Waits until message arrives in queue or till long poll timeout expires. |
| **Use-Case**    | When you need a message **right away**                                         | When you need to **save money** by reducing how often you poll          |
- **Message Timers:** let you specify an initial invisibility period for an individual message when sending to the queue
- **Temporary Queues:** Let you specify an initial invisibility period for an individual message when sending to the queue. Don't support timers on individual message
- **ABAC**: Is an authorization process that defines permissions based on tags that are attached to users aqnd AWS resources. Use tag and aliases.
- **Message Metadata:** allows you to attach metadata to messages
### SQS vs SNS
https://www.youtube.com/watch?v=mXk0MNjlO7A&ab_channel=BeABetterDev
https://www.youtube.com/watch?v=RoKAEzdcr7k&ab_channel=BeABetterDev
## Amazon MQ
Amazon MQ is a managed message broker service for [Apache ActiveMQ](https://activemq.apache.org/) and [RabbitMQ](https://www.rabbitmq.com/). A **_message broker_** enables software applications and components to communicate using various programming languages, operating systems, and formal messaging protocols through either topic or queue event destinations.

- Supports different types of protocol: **AMQP,MQTT,STOMP**
## Amazon Kinesis
AWS fully managed solution, not serverless, for collecting processing, and analyzing streaming **real-time data** in the cloud.
### Types
#### Kineses Data Streams
Amazon Kinesis Data Streams is a serverless streaming data service that makes it easy to capture, process, and store data streams at any scale.
![](images/19ac3e42267dfac1d1d32d9ad4a9dd66.png)

> [!NOTE]
> **Shard**: unit of capacity within a Kinesis data stream that defines how much data the stream can ingest and process
> 

- **Data Retention:** 1 day (default) to 365 days.
- **EFO(Enhance Fan Out)**: allows up to 20 consumers to retrive records from a stream throughput of up to 2 MB of data per second per shard
- **Partition Keys**
	- used to group data by shard within a stream
	- partition keys are Unicode strings, with a maximum length limit of 256 characters for each key
	- When an application puts data into a stream, it must specify a partition key.
- **Sequence Number**: Each data record has a sequence number that is unique per partition-key within its shard
- **Producers** use SDK, Kinesis Producer Library (KPL) or **Kinesis Agent** to publish records
	- **Kinesis Producer Library(KPL)** is a managed library by AWS to let you publish data to a **Kineses data stream**. Is a Java library
- **Consumers** use SDK or KCL: 
	- **Kineses Client Library** is a java library that makes it easy for developers to easily consume data for kineses. via the Multi-Daemon other programming languages can be used
### Capacity mode

|                | **On Demand**                                                                                                                                                                                                                                  | **Provisioned**                                                                                                                                                                      |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Use Case**   | Unpredictable workloads                                                                                                                                                                                                                        | Predictable workloads                                                                                                                                                                |
| **Management** | Automatically scales                                                                                                                                                                                                                           | Customer manages shards                                                                                                                                                              |
| **Billing**    | Volume of data ingested and retrieved                                                                                                                                                                                                          | Number of shards, data transfer                                                                                                                                                      |
| **Capacity**   | <ul>Writes: <li>200 MiB per second</li><br><li>200,000 records per second</li></ul><br><ul>Reads: <li>400 MiB per second per consumer</li><br><li>2 consumers by default</li><br><li>Enhanced Fan-Out (EFO) to add 20 more consumers</li></ul> | <ul>Write: <li>1 MB per second per shard</li><br><li>1,000 records per second per shard</li></ul><br><ul>Read: <li>2 MiB per second per shard</li><br><li>Max shards is 200</li><ul> |

#### Kinesis Data FireHose
Amazon Kinesis Data Firehose is an _extract, transform, and load (ETL) service_ that reliably captures, transforms, and **_delivers streaming data to data lakes, data stores, and analytics services._**
- **Destination (Consumer)**: 
	- (**S3, Redshift**, OpenSearch)
	- Splunk, MongoDB, DataDog, NewRelic, etc.
	- HTTP endpoint.
- **Serverless**
- **Can ingest data in real time directly from source**
- **Auto-scaling**
- Pay for data going through Firehose (no provisioning)
- **Data Transformation:** before data is sent to a destination it can be transformed with AWS Lambda
- **Convert record format**: Data firehose can convert **JSON** data into different file formats before being delivered to S3
#### Kinesis Data Analytics (KDA)
**Serverless.** Amazon Kinesis cost-effectively processes and analyzes streaming data at any scale as a fully managed service. Simple version of Kineses data stream.

- Perform **real-time analytics on Kinesis streams using SQL**
- **Cannot ingest data directly from source** (ingests data from KDS or KDF)
- **Auto-scaling**
- **Serverless**
#### Kinesis video streams
Allows you to run queries against that is flowing through your real time stream so you can create and analysis on emerging data.

- **Use Case**: Used in applications such as video surveillance, live video streaming, video analytics, and IoT devices.
![](images/22f5b2ec1358f9bef71045cf85f9ed1c.png)

#### Comparison
- **Kinesis Data Streams (KDS)**: Provides maximum flexibility and control for handling large volumes of real-time streaming data. However, it requires custom applications for data processing. Ideal for scenarios where you need to develop your own real-time analytics or data processing applications.

- **Kinesis Data Firehose**: Fully managed and simplifies delivering streaming data to predefined destinations (like S3, Redshift, Elasticsearch). It is great for backup and storage integration without the need to manage scaling or custom processing.
    
- **Kinesis Data Analytics (KDA)**: Designed for analyzing streaming data in real-time using SQL or Apache Flink. It extracts real-time insights from the data stream without building complex custom applications, ideal for live dashboards, anomaly detection, and streaming ETL.
    
- **Kinesis Video Streams**: Specializes in handling real-time video data (such as security feeds or IoT video streams). It is used mainly for video surveillance, smart home applications, and machine learning on video (e.g., object detection, facial recognition).
## AWS Glue
**ETL** = Extract, transform, and load

AWS Glue is **serverless** data integration that makes it easy for **analytics** users to discover, prepare, move and integrate data from multiple sources. Visually create, run, monitor, extract and load ETL pipelines to load data into your data lakes. A pipeline is composed of more jobs

- **AWS Glue ETL jobs are charged based on the number of data processing units (DPUs)**
- Search and catalog using, Athena, EMR and Redshift
- **AWS Glue Studio**, feature that allows you to visually build ETL jobs and pipelines
- **AWS Glue Data Catalog:** is a fully manage Apache Hive Metastore-compatible catalog service that makes it easy for customer to store, annotate and share metadata about their data

## AWS App Mesh
Is a service mesh that helps manage and monitor the communication **between microservices** in an application. It provides a way to control how different parts of your microservices architecture communicate with each other

## OpenSearch Service
Is a managed full-text search service that makes it easy to deploy operate and scale OpenSearch and ElasticSearch, a popular open-source search and analytics engine.
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
![](images/a4de21583aeb06067df9d479b5622e0d.png)
- Serverless
- Invoke Lambda functions using REST APIs (API gateway will proxy the request to lambda)
- **3 types:**
	- **Rest API (API Gateway V1)**
		- Higher cost
		- Both private and public options
	- **HTTP API(API Gateway V2)**
		- Simple feature
		- Only public APIs
	- **Web Socket API:** persistent connections for real time use cases such as chat application or dashboard
- **Throttling limits:** help control the rate at which requests are processed, **preventing overuse and ensuring resources are not overwhelmed.**
	- returns **429 Too Many Requests** when limit are exceeded
-  Cache API responses
-  **Can be integrated with any HTTP endpoint in the backend or any AWS API**
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
### Access Management

- Create an IAM policy and attach to User or Role to allow it to call an API
- Good to provide **access within your own AWS account**
#### Lambda Authorizer
- Uses a Lambda function to validate the token being passed in the header and return an **lAM** policy to determine if the user should be allowed to access the resource.
- Option to **cache result of authentication**
- For **OAuth / SAML / 3rd party type of authentication**
- Good to provide access outside your AWS account if you have an **existing IDP**
#### Cognito User Pools

- **Seamless integration with CUP** (no custom lambda implementation required)
- **Only supports authentication** (authorization must be implemented in the backend)

### Canary release
Implementing a canary release deployment strategy for the API Gateway is a great way to ensure your APIs remain stable and reliable. This strategy involves releasing a new version of your API to a small subset of users, allowing you to test the latest version in a controlled environment.
## ECR
Private Docker Registry on AWS, similar to **docker hub**. Makes it easier for developers to store, manage, and deploy Docker container images. Lets you to store Docker and Open Container Initiative (OCI) Images.
- You can controll repo access with  **Repo Policy**
- You can control private register access via **Register Policy**
- You must  remotely login using docker
- Storage backed by S3
- **Pay per use**
- **Encryption at rest** for repo images
- **ECR Lifecyle policy**: can be used to expire old images based on specific criteria
## ECS
 Amazon Elastic Container Service (Amazon ECS) is a fully managed container orchestration service that helps you easily deploy, manage, and scale containerized applications. Compare to fargate you need to build the infrastructure on your own (EC2 instances).
 - **Not Serverless**
 - ECS takes care of launching & stopping containers (ECS tasks)
 - **EC2 instances have ECS agent running on them as a docker container**
- **ECS Task Lifecycle:**
	![](images/3c308cc24812f3d390fc5092b40183a0.png)

#### With Load Balancer
- For every container, the container port is mapped to a random free port on the host (instance). So the application running inside that container will be reached by the ALB on that random port.
- **Dynamic Host Port Mapping** - Once the ALB is registered to a service in the ECS cluster, it will automatically find the right port on the EC2 Instances. This only works with ALB, not CLB.
- You **must allow on the EC2 instance’s security group any port from the ALB security group** because it may attach on any port
### ECS Fargate
AWS Fargate is a technology that you can use with Amazon ECS to run **containers** without having to manage servers or clusters of Amazon EC2 instances. With AWS Fargate, you no longer have to provision, configure, or scale clusters of virtual machines to run containers. 
- **Serverless**
- You can create an **empty** ECS cluster (no EC2's provisioned) and then launch Tasks as Fargate
- You **no longer have to provision, configure, and scale clusters** of EC2 instances
- ECS launches the required containers based on the CPU / RAM needed (**we won’t know where these containers are running**)
- You are charged by seconds but for **at least one minute**.
- When using ELB to point to Fargate you have to use IP addresses since Fargate tasks do not have a hostname
### IAM Roles for ECS Tasks

- **EC2 Instance Profile** (IAM role for the EC2 instance)
    - Used by the ECS agent to:
        - Make API calls to ECS service
        - Send container logs to Cloud Watch
        - Pull Docker image from ECR
- **Task Execution Role**
    - **Allows ECS tasks to access AWS resources**
    - Each task can have a separate role
    - **Use different roles for the different ECS Services**
    - Task Role is defined in the task definition
    - Use `taskRoleArn` parameter to assign IAM policies to ECS Task Execution Role
    - Ex. Reference sensitive data in **Secrets Manager** or **SSM Parameter Store**
### ECS Services
- An ECS Service is a **collection of long-running ECS tasks** (eg. web application) that perform the same function
- We can use ALB to send requests to these tasks
- **Service CPU Usage** or the **SQS queue length** for a service are used for scaling

#### With Load Balancer
- **EC2 Launch Type**
	- For every container, the container port is mapped to a random free port on the hots (instance). So the application running inside that container will be reached by the ALB on that random port.
	- **Dynamic Host Port Mapping** - Once the ALB is registered to a service in the ECS cluster, it will automatically find the right port on the EC2 Instances. This only works with ALB, not CLB.
	- You **must allow on the EC2 instance’s security group any port from the ALB security group** because it may attach on any port
- **Fargate Launch Type**
	- **Each task has a unique IP but the same container port**
	- The ALB connects to each task directly on its IP and container port since these containers are not run on a defined host (instance).
	- You **must allow on the ENI’s security group the task port from the ALB security group**
## EKS
Amazon **Elastic Kubernetis Service** is a managed service that  simplifies the process of building, securing, operating, and maintaining Kubernetes clusters on AWS
**Use Cases:** If your company is already using Kubernetes on-premises or in another cloud, and wants to migrate to AWS using Kubernetes
![](images/98a15c5166afba7e6e8ece9937fb03da.png)

 - **EKS Connector**: if you want to connect your own kubernetis cluster
 - **EKS CTL**: is CLI tool for easily setting up kubernetis clusters on AWS.
 - **EKS Distro**: is a Kubernetis distribution based on and used by EKS to create reliable and secure kubernetis clusters.
 - **Kubernetes Metrics Server** è un aggregatore di dati sull'utilizzo delle risorse nel cluster e non viene distribuito per impostazione predefinita nei cluster Amazon.
- **EKS anywhere(EKS-A)**: is a deployment option for Amazon EKS to easily create and operate **kubernetis(k8s)** clusters on premises with your own Vms or bare metal hosts. EKS anywhere uses **EKS Distro** as the **kubernetis** cluster distribution.
### Autoscaling in EKS
Autoscaling is a function that automatically scales your resources out and in to meet changing demands.
- **Karpenter**: is a flexible, high-performance Kubernetes cluster autoscaler that helps improve application availability and cluster efficiency.
- **Cluster Autoscaler**: automatically adjusts the number of nodes in your cluster when pods fail or are rescheduled onto other nodes.

## Amazon managed service for Prometheus (AMP) 	
 **Prometheus**: Open source monitoring and alerting toolkit originally built at SoundCLoud. **Is a timeseries database** (Collect and stores its metrics as time series data). 

**Amazon managed service for Prometheus**: Is a Prometheus compatible monitoring service for container infrastructure and application metrics
## Amazon managed service for Graphana
**Graphana**: is an open source analytics web application. It is used with timeseries database like Prometheus.

**Amazon managed service for Graphana**: Fully managed and secure data visualization service that you can use to instanly query, correlate, and visualize operational metrics , logs, and traces from multiple sources.

## Lake Formation 
**Data Lakes**: A data lake is a centralized data **repository** for unstructured and semi structured data. Contain vast amount of data. Data lakes generally use object(blob) or file as its storage type. Often used for analytics

**AWS Lake Formation:** It is a data lake to centrally govern secure and globally share data for analytics and machine learning.


![](images/1d80118c46681e076b0f11a174c44daf.png)
## Blue-Green Deployment (Not a service)
- Blue-green deployment is a technique to test features in the new environment without impacting the currently running version of your application
    - **Blue** - current version
    - **Green** - new version
- When you’re satisfied that the green version is working properly, you can gradually reroute the traffic from the old blue environment to the new green environment.
- Can mitigate common risks associated with deploying software, such as **downtime** and **rollback** capability.
## Canary Release (Not a service)
A **canary release** is a software deployment strategy where a new version is gradually rolled out to a small subset of users before being released to the entire user base. This approach allows teams to monitor performance, stability, and user impact in real-world conditions
## AWS CodeDeploy
 CodeDeploy is a deployment service that automates application deployments to Amazon EC2 instances, works with On-Premises Servers • Hybrid service
![](images/8d844b8835aa0c50995e2934ad0a19e3.png)

## AWS CodeBuild
 AWS CodeBuild is a fully managed build service in the cloud. CodeBuild compiles your source code, runs unit tests, and produces artifacts that are ready to deploy. **Pay-as-you-go**
 ![](images/30f2a25d7deca983bbfb18650637f668.png)
## AWS CodePipeline
 Orchestrate the different steps to have the code automatically pushed to production, Code => Build => Test => Provision => Deploy, Basis for CICD (Continuous Integration & Continuous Delivery)
 ![](images/09c0b920b12b6f8def70c791ce0eee8b.png)
## AWS CodeArtifact
It is a secure, highly scalable, managed artifact repository service that helps organizations to store and share software packages for application development. **Dependencies management**
 ![](images/61210af1bd54a678e606cc36840ad7a7.png)
## AWS Cloud9
 AWS Cloud9 is an integrated development environment, or _IDE_.
## AWS CodeStar (Non più supportato)
Unified UI to easily manage software development activities in one place
## AWS CodeCommit (Non più supportato)
 Is a version control service hosted by Amazon Web Services that you can use to privately store and manage assets. 

# Monitoring
## Cloud Trail
Is a service that enables governance compliance operational auditing of your AWS account

- **Global Service** 
- Active by default
- Collect logs for 90 days via Event History. If you need more than 90 days you need to create a Trail.
- Very useful to monitor API calls and made Actions made on an AWS account. Easily identify which users on an AWS account made call.

**CloudTrail provides three ways to record events:**
- **Event history** – The **Event history** provides a viewable, searchable, downloadable, and immutable record of the past 90 days of management events in an AWS Region.
- **CloudTrail Lake** – **AWS CloudTrail Lake** is a managed data lake for capturing, storing, accessing, and analyzing user and API activity on AWS for audit and security purposes.
- **Trails** – _Trails_ capture a record of AWS activities, delivering and storing these events in an Amazon S3 bucket, with optional delivery to **CloudWatch Logs** and **Amazon EventBridge**.
## CloudWatch
Amazon CloudWatch monitors your Amazon Web Services (AWS) resources and the applications you run on AWS in real time. **Amazon CloudWatch is basically a metrics and logs repository. An AWS service puts metrics into the repository, and you retrieve statistics based on those metrics**

- **High Availability**
- **Serverless**
![](images/ae98c9d1d552345b8ccabeb15f6ce4da.png)
### Amazon CloudWatch Alarms
(like budget but less powerful) Alarms are used to trigger notifications for any metric.

2 types: 
- Metric Alarm: A _metric alarm_ watches a single CloudWatch metric or the result of a math expression based on CloudWatch metrics.
- Composite Alarm: A _composite alarm_ includes a rule expression that takes into account the alarm states of other alarms that you have created.

- **Alarm States:**
    - OK
    - INSUFFICIENT_DATA
    - ALARM
- **Period:**
    - Length of time in seconds to evaluate the metric before triggering the alarm
    - High resolution custom metrics: **10 sec**, 30 sec or multiples of 60 sec
- **Action**:
	- **Auto Scaling Group**, increase or decrease EC2 instances “desired” count  
	- **EC2 Actions**: stop, terminate, reboot or recover an EC2 instance 
	- **SNS notifications**: send a notification into an SNS topic 
	- **Invoke a Lambda function
	-  You can create a billing alarm
		- **Billing data metric is stored in CloudWatch us-east-1**
### Logs
You can use Amazon CloudWatch Logs to monitor, store, and access your log files from Amazon Elastic Compute Cloud (Amazon EC2) instances, AWS CloudTrail, Route 53, and other sources.

Actions:
- **Two log classes for flexibility**: CloudWatch Logs offers two log classes so that you can have a cost-effective option for logs that you access infrequently.
- **Query your log data**: You can use CloudWatch Logs Insights to interactively search and analyze your log data.
- **Detect and debug using Live Tail**: You can use Live Tail to quickly troubleshoot incidents by viewing a streaming list of new log events as they are ingested.
- **Monitor logs from Amazon EC2 instances**: You can use CloudWatch Logs to monitor applications and systems using log data.
- **Monitor AWS CloudTrail logged events**: You can create alarms in CloudWatch and receive notifications of particular API activity as captured by CloudTrail.
- **Audit and mask sensitive data**: If you have sensitive data in your logs, you can help safeguard it with data protection policies.
- **Log retention**: By default, logs are kept indefinitely and never expire.
- **Archive log data**: You can use CloudWatch Logs to store your log data in highly durable storage.
- **Log Route 53 DNS queries**: You can use CloudWatch Logs to log information about the DNS queries that Route 53 receives.

You can use Cloudwatch logs alongside with : **CloudTrail, IAM, Kinesis Data Stream, Lambda**
## Amazon EventBridge


**EventBridge** is a serverless event bus service that is used for application integration by streaming real-time data to your applications. EventBridge was formerly called Amazon **CloudWatch Events.**
![](images/6bacaa5a91d9fef965abd074419ab1d5.png)
#### Event Bus
An event bus receives events from a source and rotates events to a target based on rules. 
![](images/f09ac2fea58ad9c25687d4eed3c60c52.png)
- Event buses types:
    - **Default event bus**: events from AWS services are sent to this
    - **Partner event bus**: receive events from external SaaS applications
    - **Custom Event bus**: for your own applications
#### Rules
 A rule specifies which events to send to which [targets](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-targets.html) for processing. A single rule can send an event to multiple targets, which then run in parallel.
## Health Dashboard
- **Service Health Dashboard:** Show General status of the service in various region.
- **Personal Health Dashboard:** AWS Personal Health Dashboard provides alerts and guidance for AWS events that might affect your environment.

## AWS Config
AWS Config provides a detailed view of the configuration of AWS resources in your AWS account. This includes how the resources are related to one another and how they were configured in the past so that you can see how the configurations and relationships change over time.

- Regional service
- Can be aggregated across regions and accounts
- Record configurations changes over time
- Evaluate compliance of resources using **config rules**. By creating an AWS Config rule, you can enforce your ideal configuration in your AWS account
- **Does not prevent non-compliant actions** from happening (no deny)
- Can make custom config rules (must be defined in Lambda functions) such as:
    - Check if each EBS disk is of type gp2
    - Check if each EC2 instance is t2.micro
- **Can be used along with CloudTrail** to get a timeline of changes in configuration and compliance overtime.
- **Integrates with EventBridge or SNS** to trigger notifications when AWS resources are non-compliant

**Use case:**
- Needs: The company’s solutions architect must implement a solution that checks for untagged AWS resources.
- Solution: **You can use an AWS Config rule to detect non-compliant tags.**

- Action
	- **Resource Administration**: You can use AWS Config rules to evaluate the configuration settings of your AWS resources
	- **Auditing and Compliance**: You might be working with data that requires frequent audits to ensure compliance with internal policies and best practices
	- **Managing and Troubleshooting Configuration Changes**: you can view how the resource you intend to modify is related to other resources and assess the impact of your change.
	- **Security Analysis**: you can use AWS Config to view the IAM policy that was assigned to a user, group, or role at any time in which AWS Config was recording
## X-Ray
AWS X-Ray receives data from services as _segments_. X-Ray then groups segments that have a common request into _traces_. X-Ray processes the traces to generate a _service graph_ that provides a visual representation of your application.

- Provides an **end-to-end view of requests as they travel through your application**, and shows a map of your application’s underlying components.
- AWS X-Ray helps developers analyze and debug production, distributed applications, such as those built using a **micro-services architecture**.
- Helps understand how your application and its underlying services are performing to identify and troubleshoot the root cause of performance issues and errors.
- Can collect data across AWS Accounts. The **X-Ray agent** can **assume a role** to publish data into an account different from the one in which it is running. This enables you to publish data from various components of your application into a **central account**.
# Management
## (IAM) Identity Access Manager
AWS Identity and Access Management (IAM) is a web service that helps you securely control access to AWS resources.

- **Root User** has full access to the account
- **IAM User** has limited permission to the account
- **Principle of Least Privilege:**
	- Just Enough Access: Permitting only the exact actions  for the identity to perform task
	- Just In Time: Permitting the smallest length of duration a identity can use permissions
- **Password policy**: you can set expires date for user password in rder to rotate them
- **Access Key**: you can have a maximum of 2 access key per account

#### Groups
Collections of users and have policies attached to them
- **Only IAM Policy can be assigned to group (no-role)**
- Groups cannot be nested
- User can belong to multiple groups
- User doesn't have to belong to a group

#### MFA
AWS Multi-Factor Authentication (MFA) enhances account security by requiring users to provide an additional verification factor beyond just their username and password.

- Different types:
	- Virtual MFA device
	- Universal 2nd Factor (U2F) Security Key
	- Hardware MFA devices (MFA devices options in AWS, Hardware Key Fob MFA Device for AWS GovCloud (US)).

### Feature
- **Temporary Security Credentials**: Generated dynamically, every time you use a roles a you a temporary security cred are created automatically
- **IAM Identity federetion**: the means of linking a person to eletronic identity and attribute. When you connect using facebook or google you're using identity federation.
- **Cross Account Role:** You can grant users from different AWS account access to resources in your account through a Cross-Account Role. This allows you to not to have to create them a user account within your systems

### Policies
- Policies are JSON documents that outline permissions for users, groups or roles
- Two types
    - **User based policies**
        - IAM policies define which API calls should be allowed for a specific user
    - **Resource based policies**
        - Control access to an AWS resource
        - Grant the specified principal permission to perform actions on the resource and define under what conditions this applies

There are more policies descripted here below (not all are related to IAM):

| Policy Type                         | Description                                                                                   | Example Use Case                                                     |
| ----------------------------------- | --------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| **Identity-based Policies**         | Policies attached to IAM users, groups, or roles, either as managed or inline policies.       | Grant full S3 access to a group of developers.                       |
| **Resource-based Policies**         | Policies attached to AWS resources that define who can access that resource and what actions. | Grant cross-account access to an S3 bucket.                          |
| **Permissions Boundaries**          | Limit the maximum permissions a user or role can have.                                        | Limit the privileges of IAM roles created by administrators.         |
| **Service Control Policies (SCPs)** | Manage permissions across AWS accounts in AWS Organizations.                                  | Restrict access to specific services across accounts in an org.      |
| **Access Control Lists (ACLs)**     | Control access to specific resources at a lower level of granularity (e.g., object-level).    | Control access to individual S3 objects or VPC traffic.              |
| **Session Policies**                | Temporary policies applied during an AWS STS session.                                         | Grant temporary access to perform a specific action in a session.    |
| **ABAC Policies**                   | Access control based on attributes (tags) of users and resources.                             | Allow access to resources with specific tags like `project:alpha`.   |
| **Backup Policies**                 | Manage backup configurations across AWS accounts in AWS Organizations.                        | Enforce backup schedules for critical data across multiple accounts. |
#### Trust Policies
- Defines which principal entities (accounts, users, roles, federated users) can assume the role
- An IAM role is both an identity and a resource that supports resource-based policies.
- You must attach both a trust policy and an identity-based policy to an IAM role.
- The **IAM service supports only one type of resource-based policy** called a **role trust policy**, which is **attached to an IAM role**.
#### Anatomy of Policy
![](images/95f171c750a994b0c177e4ec38ad6af3.png)
- **Version policy language** version. 2012-10-17 is the latest version.
- **Statement container** for the policy element you are allowed to have multiples
- **Sid (optional)** a way of labeling your statements.
- **Effect Set** whether the policy will Allow or Deny
- **Action list** of actions that the policy allows or denies
- **Principal** account, user, role, or federated user to which you would like to allow or deny access
- **Resource** the resource to which the action(s) applies
- **Condition (optional)** circumstances under which the policy grants permission
### Guidelines
- Use root account only for account setup
- 1 physical user = 1 IAM user
- Enforce MFA for both root and IAM users
- Never share lAM credentials & Access Keys
### Reporting Tools
- **Credentials Report**
    - lists all the users and the status of their credentials (MFA, password rotation, etc.)
    - **account level** - used to audit security for all the users
- **Access Advisor**
    - shows the service permissions granted to a user and when those services were last accessed
    - **user-level**
    - used to revise policies for a specific user
### Permission Boundaries
- Set the maximum permissions an IAM entity can get
- **Can be applied to users and roles (not groups)**
- Used to ensure some users can’t escalate their privileges (make themselves admin)

#### Assume Role vs Resource-based Policy
- When you assume an IAM Role, you give up your original permissions and take the permissions assigned to the role
- When using a resource based policy, the principal doesn’t have to give up their permissions
## Amazon Service Catalog

AWS Service Catalog enables organizations to create and manage catalogs of products that are approved for use on AWS to achive consistent governance and meet compliance requirements.
Alternative to granting direct access to AWS resources via the AWS console.
## AWS Resource Access Manager (AWS RAM)
AWS Resource Access Manager (AWS RAM) helps you securely share your resources across AWS accounts, within your organization or organizational units (OUs), and with AWS Identity and Access Management (IAM) roles and users for supported resource types.

**Use cases:** A company has multiple AWS accounts for different departments, such as Development, QA, and Production. Each department runs its workloads in separate accounts but needs access to shared network resources.
## AWS IAM Identity Center 
**AWS Identity Center (formerly AWS Single Sign-On)** is a service that provides centralized management of user and group access to AWS resources. 

**Single sign-on** for multi account in aws.
## OpsWorks
AWS OpsWorks is a configuration management service that helps you configure and operate applications in a cloud enterprise by using Puppet or Chef

- **Chef & Puppet** are two open-source softwares that help you perform **server configuration** automatically, or repetitive actions
- It’s an **alternative to AWS SSM**

> [!NOTE]
> Exam tip: **Chef & Puppet ⇒ AWS OpsWorks**

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

> [!NOTE]
>  Organization API can only create member accounts. They cannot configure anything within those accounts (use [CloudFormation](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/CloudFormation) for that).

### Organizational Units (OU)
- Folders for grouping AWS accounts of an **organization**
- Can be **nested**
### Service Control Policies (SCP)
- **Whitelist or blacklist IAM actions at the OU or Account level**
- **Does not apply to the Master Account**
- Applies to all the Users and Roles of the member accounts, including the root user. So, if something is restricted for that account, even the root user of that account won’t be able to do it.
- Must have an explicit allow (**does not allow anything by default - stateless** )
- **Does not apply to service-linked roles (roles that are automatically created and managed by AWS services)**
- **Explicit Deny has the highest precedence**
- SCPs are just guardrails for setting up the maximum allowable permissions an IAM identity can have. It's not capable of checking for non-compliant tags.

### Migrating Accounts between Organizations
- To migrate member accounts from one organization to another
    1. Remove the member account from the old organization
    2. Send an invite to the member account from the new organization
    3. Accept the invite from the member account
- To migrate the master account
    1. Remove the member accounts from the organizations using procedure above
    2. Delete the old organization
    3. Repeat the process above to invite the old master account to the new org

## AWS Control tower
**AWS Control Tower** provides a single location to easily set up your new well-architected multi-account environment and govern your AWS workloads with rules for security, operations, and internal compliance.
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
## Saving Plan
Savings Plans are a flexible pricing model that offer low prices on Amazon EC2, AWS Lambda, and AWS Fargate usage, in exchange for a commitment to a consistent amount of usage (measured in $/hour) for a 1 or 3 year term.
#### Compute Savings Plans
- Reduce your costs up to **66%.** 
- These plans automatically apply to EC2, Fargate or Lambda usage.
- **Complete flexibility (you can change type, family, AZ ecc...)**
- **Example**
	- With Compute Savings Plans, you can change from C4 to M5 instances, shift a workload from EU (Ireland) to EU (London)
	- Move a workload from EC2 to Fargate or Lambda at any time and automatically continue to pay the Savings Plans price.
#### EC2 Instance Savings Plans
- Reduce your costs up to **72%**
- Give you the flexibility to change your usage between instances **but only within a family in that region.**
- **Example**
	- You can move from c5.xlarge running Windows to c5.2xlarge running Linux and automatically benefit from the Savings Plan prices.
## Cost allocation tag

- As a best practice, reactivate your cost allocation tags when moving organizations. When an account moves to another organization as a member, previously activated cost allocation tags for that account lose their "active" status and need to be activated again by the new management account.
- As a best practice, do not include sensitive information in tags.
- Only a **management account** in an organization and single accounts that aren't members of an organization have access to the **cost allocation tags** manager in the Billing and Cost Management console.
## CostExplorer

- Visualize and manage your costs and service usage over time
- Create **custom reports** that analyze cost and usage data
- **Recommendations** to choose an optimal savings plan
- View usage for the **last 12 months & forecast up to 12 months**

# Machine Learning 
## CodeGuru
CodeGuru is machine learning code analysis service. CodeGuru performs code-reviews and will suggest changes to improve the quality of code.
### Type of service:
- **Amazon CodeGuru Security:** Amazon CodeGuru Security is a static application security tool that uses machine learning to detect security policy violations and vulnerabilities
- **CodeGuru Profiler:** Amazon CodeGuru Profiler collects runtime performance data from your live applications, and provides recommendations that can help you fine-tune your application performance
- **CodeGuru Reviewer**: Amazon CodeGuru Reviewer is a service that uses program analysis and machine learning to detect potential defects that are difficult for developers to find and offers suggestions for improving your Java and Python code.
## Comprehend
Fully managed and **serverless** service for Natural Language Processing. Uses machine learning to **find and analyse insights and relationships in text**. 
- Use cases: analyze customer interactions (emails) to find what leads to a positive or negative experience.

## Rekognition:
Amazon Rekognition is image and video reognition service. Analyze images and videos to detect and labels objects, people, clebrities.

Find objects, people, text, scenes in images and videos using ML
Facial analysis and facial search to do user verification, people counting

## Transcribe:
**Automatically convert speech to text** 
- Uses a deep learning process called automatic speech recognition (ASR) to convert speech to text quickly and accurately 
- **Automatically remove Personally Identifiable Information (PII)** using Redaction 
- Supports **Automatic Language Identification** for multi-lingual audio
## Polly:
Is a Text-to-speech service 
- Allowing you to create applications that talk
- Engine Types (from less expensive to most expensive):
	- Standard -
	- Long Form 
	- Neural 

> [!NOTE]
> **Lexicon:** for specialized pronuciation
> **Speech Marks:** metadata that describes speech

Use an xml based language: **Speech Synthesis Markup Language (SSML)**
## Translate
Natural and accurate language translation

## SageMaker
Amazon SageMaker is a **fully managed** machine learning (ML) service. With SageMaker, data scientists and developers can quickly and confidently **build, train, and deploy ML models**

## Forecast:
Amazon Forecast is a fully managed service that uses statistical and machine learning algorithms to deliver highly accurate time-series forecasts. 
- Example: predict the future sales of a raincoat


### Amazon Forecast Workflow: 
- **Importing Datasets** – _Datasets_ are collections of your input data. Dataset groups are collections of datasets that contain complimentary information. Forecast algorithms use your dataset groups to train custom forecasting models, called predictors.
- **Training Predictors** – _Predictors_ are custom models trained on your data. You can train a predictor by choosing a prebuilt algorithm,or by choosing the AutoML option to have Amazon Forecast pick the best algorithm for you.
- **Generating Forecasts** – You can generate forecasts for your time-series data, query them using the QueryForecast API, or visualize them in the console.

## Kendra
Fully managed **document search** service powered by Machine Learning. Use keyword based search with semantic contextual understanding capabilites.

#### Components
- An **index** that holds your documents and makes them searchable.
- A **data source** that stores your documents and Amazon Kendra connects to. You can automatically synchronize a data source with an Amazon Kendra index so that your index stays updated with your source repository.
- A **document addition API** that adds documents directly to an index.

**Two version:**

| Edition            | Developer Edition                            | Enterprise Edition                          |
| ------------------ | -------------------------------------------- | ------------------------------------------- |
| Indexes            | 5 indexes with up to 5 data sources each     | 5 indexes with up to 50 data sources each   |
| Documents          | 10,000 documents or 3 GB of extracted text   | 10,000 documents or 3 GB of extracted text  |
| Queries            | 4,000 queries per day or 0.05 queries/second | 8,000 queries per day or 0.1 queries/second |
| Availability Zones | Runs in 1 AZ                                 | Runs in 3 AZs                               |

> [!NOTE]
> Developer Edition has a free tier with 750 hours for the first 30 days.
## Lex
Is a conversion interface service. With Lex you can build conversational **voice and text chatbots**. 
- Same conversational engine that powers Amazon Alexa
## Personalize:
Amazon Personalize is a fully managed machine learning service that uses your data to generate item recommendations for your users. It can also generate user segments based on the users' affinity for certain items or item metadata.

**Use cases:** 
- Personalizing a video streaming app
- Adding product recommendations to an ecommerce app
- Personalizing search results

**Workflow:** 
- Match your use case to Amazon Personalize resources
- Prepare your training data
- Create schema JSON files for your data
- Create a dataset group
- Create schemas and datasets
- Import training data into datasets
- Train and deploy a model
- Get recommendations
- Record real-time events
## Textract:
Amazon Textract helps you add document text detection and analysis to your applications using ML.
Some action that Textract can perform:
- Detect typed and handwritten text
- Extract text, forms, and tables from documents with structured data
- Process invoices and receipts with the AnalyzeExpense API.


## Amazon Q
Amazon Q Business is a generative artificial intelligence (generative AI)-powered assistant that you can tailor to your business needs. Amazon Q Developer is an AWS generative AI assistant that helps you understand, build, extend, and operate applications and workloads on AWS.
### Amazon Q Business
Amazon Q Business is a fully managed, **generative-AI powered assistant** that you can configure to answer questions, provide summaries, generate content, and complete tasks based on your enterprise data
### Amazon Q Developer
Amazon Q Developer is a generative artificial intelligence (AI) powered conversational assistant that can help you understand, build, extend, and operate AWS applications
### Amazon Q for Amazon QuickSight
Ask questions about your BI data within QuickSight
Generative BI capabilities to quickly build compelling visuals, summarize insights,
answer data questions, and build data stories using natural language.
#### Amazon Q for Amazon Connect
real-time conversation with the customer along with relevant company content 
#### Amazon Q for AWS Supply Chain
get intelligent answers about what is happening in their supply chain
## Amazon Connect:
Amazon Connect is an AI-powered application that provides one seamless experience for your contact center customers and users. It's comprised of a full suite of features across communication channels.

# Concepts
- **Throughput**: is a measure of how many units of information a system can process in a given amount of time.
- **Active-passive** means that one is actively used and the other in stanby until the first fail.
- **Publish Subscribe pattern:** Common message pattern, the sender send their message to an event bus that categorized message in groups and than the receiver subscribes to these groups. Whenever new messages appear within their subscription the messages are immediatly delivered to them. 
![](images/dca3c1328ca6b5dac37aca42e9c1a7eb.png)
- **Messaging System**: Used to provide asynchronous communication and decouple processes via messages/events from a sender and receiver (producer and consumer)
- **Queueing System**: messaging system that generally will delete messages once they are consumed. Simple communication. Not Real-time. Have to pull. Not reactive.
- **Datawerehouse**: Built to store large quantities of historical data and enable fast, complex queries across all the data.
- **OLAP** applications look at multiple records at the same time. You save memory because you fetch  just the columns of data you need instead of whole rows.