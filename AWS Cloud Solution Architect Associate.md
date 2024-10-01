esami di prova: https://www.exampro.co/saa-c03

AWS_CLI_AUTO_PROMPT: aiuta a fare comandi usando il cli in aws

Informazioni utili:
- Non concentrarsi molto su kubernetis 
- La parte global infrastructure molto simile a quella del cloud practitioner

## S3
- Folder in s3 are object, no metadata e no permissions, don't contain anything
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
# Network
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
- Amazon Fsx