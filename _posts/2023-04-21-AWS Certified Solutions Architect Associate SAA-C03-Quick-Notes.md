---
title: AWS Certified Solutions Architect Associate SAA-C03 Quick Notes 
date: 2023-04-21 14:02:00 +0800  
categories: [Technology, AWS Learning Journey]  
tags: [cloud]  
---
Some quick notes for preparing the exam of AWS Certified Solutions Architect Associate SAA-C03.

## AWS EC2
### [Amazon EBS volume types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html)
- Solid state drive (SSD) volumes
SSD-backed volumes are optimized for transactional workloads involving frequent read/write operations with small I/O size, where the dominant performance attribute is IOPS. SSD-backed volume types include General Purpose SSD and Provisioned IOPS SSD.

|  | General Purpose SSD (gp3) volumes | General Purpose SSD (gp2) volumes | Provisioned IOPS SSD (io2 Block Express) volumes | Provisioned IOPS SSD (io2) volumes | Provisioned IOPS SSD (io1) volumes |
| -------- | ------- | ------- | ------- | ------- | ------- |
| Max IOPS per volume | 500 IOPS per GiB × 32 GiB = <u>16,000 IOPS</u> | 3 IOPS per GiB X 5,334 GiB = <u>16,000 IOPS</u> | 1,000 IOPS per GiB × 256 GiB = <u>256,000 IOPS</u> | 50 IOPS per GiB × 1,280 GiB = <u>64,000 IOPS</u> | 500 IOPS per GiB × 128 GiB = <u>64,000 IOPS</u> |
| Max throughput per volume | 4,000 IOPS × 0.25 MiB/s per IOPS = 1,000 MiB/s | 250 MiB/s | 4,000 MiB/s | 1,000 MiB/s | 1,000 MiB/s |
| Amazon EBS Multi-attach | <u>Not supported</u> | <u>Not supported</u> | Supported | Supported | Supported |
| Boot volume | Supported | Supported | Supported | Supported | Supported |

- Hard disk drive (HDD) volumes
HDD-backed volumes are optimized for large streaming workloads where the dominant performance attribute is throughput. HDD volume types include Throughput Optimized HDD and Cold HDD.

|  | Throughput Optimized HDD(st1) volumes | Cold HDD(sc1) volumes |
| -------- | :------- | :------- |
| Use cases | Big data<br/>Data warehouses<br/>Log processing| Throughput-oriented storage for data that is infrequently accessed<br/>Scenarios where the lowest storage cost is important |
| Max IOPS per volume | 500 | 250 |
| Max throughput per volume | 500 MiB/s | 250 MiB/s |
| Amazon EBS Multi-attach | Not supported | Not supported |
| Boot volume | Not supported | Not supported |

- Previous generation volumes
Magnetic (standard) volumes are <u>previous generation volumes</u> that are backed by magentic drives. They are suited for workloads with small datasets where data is accessed infrequently and performance is not of primary importance. These volumes deliver approximately 100 IOPS on average, with burst capability of up to hundreds of IOPS, and they can range in size from 1 GiB to 1 TiB.

### [Placement groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html)
1. Cluster  
<u>packs instances close together inside an Availability Zone</u>. This strategy enables workloads to achieve the low-latency network performance necessary for tightly-coupled node-to-node communication that is typical of high-performance computing (HPC) applications.
1. Partition  
spreads your instances across logical partitions such that groups of instances in one partition do not share the underlying hardware with groups of instances in different partitions. This strategy is typically used by large distributed and replicated workloads, <u>such as Hadoop, Cassandra, and Kafka</u>.
1. Spread  
strictly places a small group(<u>a maximum of seven running instances per Availability Zone</u>) of instances across distinct underlying hardware to reduce correlated failures.

### Amazon EBS Multi-Attach
Amazon EBS Multi-Attach enables you to attach a single Provisioned IOPS SSD (io1 or io2) volume to multiple instances that are in <u>the same Availability Zone</u>. You can attach multiple Multi-Attach enabled volumes to an instance or set of instances. Each instance to which the volume is attached has full read and write permission to the shared volume. Multi-Attach makes it easier for you to achieve higher application availability in clustered Linux applications that manage concurrent write operations

### Elastic Fabric Adapter
Elastic Fabric Adapter (EFA) is a network interface for Amazon EC2 instances that enables customers to run applications requiring high levels of inter-node communications at scale on AWS. Its custom-built operating system (OS) bypass hardware interface enhances the performance of inter-instance communications, which is critical to scaling these applications. With EFA, High Performance Computing (HPC) applications using the Message Passing Interface (MPI) and Machine Learning (ML) applications using NVIDIA Collective Communications Library (NCCL) can scale to thousands of CPUs or GPUs.

### EFS
Amazon Elastic File System (Amazon EFS) provides a simple, serverless, set-and-forget elastic file system for use with Amazon Web Services Cloud services and on-premises resources. <u>Using Amazon EFS with Microsoft Windows–based Amazon EC2 instances is not supported.</u>
#### EFS storage classes
Amazon EFS offers a range of storage classes that are designed for different use cases. These include EFS Standard, EFS Standard–Infrequent Access (Standard-IA), EFS One Zone, and EFS One Zone–Infrequent Access (EFS One Zone-IA). 

#### [Amazon EBS Multi-attach vs Amazon EFS](https://repost.aws/questions/QUK2RANw1QTKCwpDUwCCI72A/efs-vs-ebs-mult-attach)
EFS is a fully-managed file system service. It's fully elastic, growing and shrinking as you add and remove files, so you don't need to worry about manually managing capacity, either to accommodate growth, or to optimize utilization - you pay only for the storage that you use. EBS multi-attach volume's don't support re-sizing capacity once they are created.  
EFS is also designed for high availability and high durability. To achieve these levels of availability and durability, EFS automatically replicates data within and across 3 Availability Zones, with no single points of failure. EBS multi-attach volumes can be used for clients within a single Availability Zone. If there is a volume failure at the EBS infrastructure layer, all clients will be impacted.  
EFS also has a number of 'bells and whistles' in terms of features and integrations with other services - things like automatic cost optimization with EFS Infrequent Access and Lifecycle Management, IAM authn/authz and Access Points, integration with AWS Backup, and integrations with services like ECS and SageMaker.

### RAID configuration on Linux
A RAID array uses multiple EBS volumes to improve performance or redundancy. Creating a RAID 0 array allows you to achieve a higher level of performance for a file system than you can provision on a single Amazon EBS volume. Use RAID 0 when I/O performance is of the utmost importance. With RAID 0, I/O is distributed across the volumes in a stripe. <u>When fault tolerance is more important than I/O performance a RAID 1 array should be used which creates a mirror of your data for extra redundancy.</u>

## Containers on AWS
### [Amazon ECS vs EKS](https://aws.amazon.com/blogs/containers/amazon-ecs-vs-amazon-eks-making-sense-of-aws-container-services/)
Amazon ECS delivers an AWS-opinionated solution for running containers at scale. It reduces the time it takes customers to build, deploy, or migrate their containerized applications successfully. 
Teams choose Kubernetes for its vibrant ecosystem and community, consistent open source APIs, and broad flexibility. They rely on Amazon EKS to handle the undifferentiated heavy lifting of building and operating Kubernetes at scale.
<u>Amazon EKS provides the flexibility of Kubernetes with the security and resiliency of being an AWS managed service that is optimized for customers building highly available services</u>. Amazon EKS provides a secure, reliable, scalable, and resilient Kubernetes environment for customers such as Intel, Snap, Intuit, GoDaddy, and Fidelity, and helps Amazon.com deliver an incredible customer experience. Customers adopting Kubernetes that want the resiliency of AWS should start with Amazon EKS.

## Elastic Beanstalk vs Cloudformation
- Elastic Beanstalk
With AWS Elastic Beanstalk, you can quickly deploy and manage applications in the AWS Cloud <u>without worrying about the infrastructure</u> that runs those applications. AWS Elastic Beanstalk reduces management complexity without restricting choice or control. You simply upload your application, and AWS Elastic Beanstalk automatically handles the details of capacity provisioning, load balancing, scaling, and application health monitoring.
- Cloudformation
AWS CloudFormation enables you to <u>create and provision AWS infrastructure deployments predictably and repeatedly</u>. It helps you leverage AWS products such as Amazon EC2, Amazon Elastic Block Store, Amazon SNS, Elastic Load Balancing, and Auto Scaling to build highly reliable, highly scalable, cost-effective applications in the cloud without worrying about creating and configuring the underlying AWS infrastructure. AWS CloudFormation enables you to use a template file to create and delete a collection of resources together as a single unit (a stack).

### AWS SAM
AWS Serverless Application Model (AWS SAM) is an extension of AWS CloudFormation that is used to package, test, and deploy serverless applications.

## Route53
### Routing Policy
- Simple routing policy
Use for a single resource that performs a given function for your domain, for example, a web server that serves content for the example.com website. You can use simple routing to create records in a private hosted zone.
- Failover routing policy 
Use when you want to configure active-passive failover. You can use failover routing to create records in a private hosted zone.
- Geolocation routing policy 
Use when you want to route traffic based on the location of your users. You can use geolocation routing to create records in a private hosted zone.
- Geoproximity routing policy 
Use when you want to route traffic based on the location of your resources and, optionally, shift traffic from resources in one location to resources in another.
- Latency routing policy 
Use when you have resources in multiple AWS Regions and you want to route traffic to the region that provides the best latency. You can use latency routing to create records in a private hosted zone.
- IP-based routing policy 
Use when you want to route traffic based on the location of your users, and have the IP addresses that the traffic originates from.
- Multivalue answer routing policy 
Use when you want Route 53 to respond to DNS queries with up to eight healthy records selected at random. You can use multivalue answer routing to create records in a private hosted zone.
- Weighted routing policy 
Use to route traffic to multiple resources in proportions that you specify. You can use weighted routing to create records in a private hosted zone.

### AWS Global Accelerator vs Route53
AWS Global Accelerator is a networking service that helps you improve the availability and performance of the applications that you offer to your global users. AWS Global Accelerator automatically checks the health of your applications and routes user traffic only to healthy application endpoints. <u>Route53 Need to set health check not like AWS Global Accelerator has built in natively.</u>

## Elastic Load Balancing
Below is the OSI model

| Layer | Name |
|-------| -------- |
| 7     | Application Layer |
| 6     | Presentation Layer |
| 5     | Session Layer |
| 4     | Transport Layer |
| 3     | Network Layer |
| 2     | Data Link Layer |
| 1     | Physical Layer |

### Classic Load Balancers
Operates at both <u>layer 4 and 7</u>. The only load balancer works with applications in the EC2-Classic network.
### Application Load Balancers
Operates at <u>Layer 7</u> and supports HTTP/HTTPS, HTTP 1.1/HTTP 2, gRPC, WebSocket.
### Network Load Balancers
Operates at <u>Layer 4</u> and is TCP/UDP connection based.
### Gateway Load Balancers
Operates at Layer 4 and is TCP/UDP connection based. It supports <u>Zonal Isolation</u>, Sticky sessions, Long Lived TCP connections.

## CloudFront
### CloudFront vs Global Accelerator
Global Accelerator works like CloudFront with Latency-based routing. While <u>CloudFront can achieve some of the same objectives as Global Accelerator, it is limited to HTTP traffic</u>, CloudFront can cache HTTP objects at edge locations and it uses a DNS name with changing IP addresses rather than static ones. Gloabal Accelerator can be used for all TCP and UDP traffic, and Global Accelerator does not support Edge Caching. 

### [CloudFront Functions](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cloudfront-functions.html)
With CloudFront Functions in Amazon CloudFront, you can write lightweight functions in JavaScript for high-scale, latency-sensitive CDN customizations.

## AWS AppSync 
AWS AppSync is a serverless GraphQL and Pub/Sub API service that simplifies building modern web and mobile applications. API Gateway is a more low-level service than AppSync. It exposes HTTP requests and WebSocket connections. Before AppSync was released, you had to build a GraphQL API with API Gateway and Lambda. A Lambda function had to use a GraphQL server library, like Apollo, to do all the work.

## [AWS Control Tower](https://docs.aws.amazon.com/controltower/latest/userguide/what-is-control-tower.html)
AWS Control Tower offers a straightforward way to set up and govern an AWS multi-account environment, following prescriptive best practices. 

## RDS
### Read replicas, Multi-AZ deployments, and multi-region deployments
- Multi-AZ deployments
Main purpose is high availability
- Multi-Region deployments
Main purpose is disaster recovery and local performance
- Read replicas
Main purpose is scalability

### [Working with storage for Amazon RDS DB instances](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIOPS.StorageTypes.html)
- Increasing DB instance storage capacity
If you need space for additional data, you can scale up the storage of an existing DB instance. To do so, you can use the Amazon RDS Management Console, the Amazon RDS API, or the AWS Command Line Interface (AWS CLI). 
<u>Scaling storage usually doesn't cause any outage or performance degradation of the DB instance. After you modify the storage size for a DB instance, the status of the DB instance is storage-optimization.</u>
- Managing capacity automatically with Amazon RDS storage autoscaling
If your workload is <u>unpredictable</u>, you can enable storage autoscaling for an Amazon RDS DB instance. To do so, you can use the Amazon RDS console, the Amazon RDS API, or the AWS CLI.

## AWS Data Exchange 
AWS Data Exchange is on a mission to increase speed to value for third-party data sets in the cloud.

## GuardDuty
Amazon GuardDuty is a threat detection service that continuously monitors for malicious activity and unauthorized behavior to <u>protect your AWS accounts</u>, Amazon Elastic Compute Cloud (EC2) workloads, container applications, Amazon Aurora databases, and data stored in Amazon Simple Storage Service (S3). 

### GuardDuty vs [Inspector](https://aws.amazon.com/inspector/)
Amazon Inspector is a vulnerability management service that continually discovers workloads, such as Amazon EC2 instances, containers, and Lambda functions, and scans them for software vulnerabilities and unintended network exposure while Amazon GuardDuty helps with analyzing your entire AWS environment for potential threats.

## AWS Batch
You can use multi-node parallel jobs to run single jobs that span multiple Amazon EC2 instances.

## S3 
### Object Lambda 
Add your own code to S3 <u>GET, HEAD, and LIST</u> requests to modify and process data as it is returned to an application

### S3 Object Lock
With S3 Object Lock, you can store objects using a write-once-read-many (WORM) model. Object Lock can help prevent objects from being deleted or overwritten for a fixed amount of time or indefinitely. You can use Object Lock to help meet regulatory requirements that require WORM storage, or to simply add another layer of protection against object changes and deletion.
1. Retention period   
Specifies a fixed period of time during which an object remains locked. <u>During this period</u>, your object is WORM-protected and can't be overwritten or deleted. For more information, see Retention periods
- <u>Governance mode</u>
users can't overwrite or delete an object version or alter its lock settings unless they have special permissions.
- <u>Compliance mode</u>
a protected object version can't be overwritten or deleted by any user, including the root user in your AWS account.
1. Legal hold  
Provides the same protection as a retention period, but it <u>has no expiration date</u>. Instead, a legal hold remains in place until you explicitly remove it. Legal holds are independent from retention periods.

### [Best practices design patterns: optimizing Amazon S3 performance](https://docs.aws.amazon.com/AmazonS3/latest/userguide/optimizing-performance.html)
1. Adding randomness to S3 key  
Your application can achieve at least <u>3,500 PUT/COPY/POST/DELETE or 5,500 GET/HEAD</u> requests per second per partitioned prefix.
1. Amazon S3 Transfer Acceleration  
Amazon S3 Transfer Acceleration can speed up content transfers to and from Amazon S3 by as much as <u>50-500% for long-distance transfer of larger objects</u>. 

## Machine Learning
### Amazon Transcribe 
Amazon Transcribe is an automatic speech recognition (ASR) service that makes it easy for developers to add speech-to-text capability to their applications. 

### Amazon Rekognition
With Amazon Rekognition, you can identify objects, people, text, scenes, and activities in images and videos, as well as detect any inappropriate content.

### Amazon Comprehend
Amazon Comprehend is a natural-language processing (NLP) service that uses machine learning to uncover valuable insights and connections in text.

### Amazon Kendra 
Amazon Kendra is an intelligent enterprise search service that helps you search across different content repositories with built-in connectors.

### Amazon Polly 
Amazon Polly uses deep learning technologies to synthesize natural-sounding human speech, so you can convert articles to speech. With dozens of lifelike voices across a broad set of languages, use Amazon Polly to build speech-activated applications.

### Amazon Translate 
Amazon Translate is a neural machine translation service that delivers fast, high-quality, affordable, and customizable language translation.

### Amazon Lex
Amazon Lex is a fully managed artificial intelligence (AI) service with advanced natural language models to design, build, test, and deploy conversational interfaces in applications.

## Data & Analytics
### Amazon Redshift 
Amazon Redshift uses SQL to analyze structured and semi-structured data across data warehouses, operational databases, and data lakes, using AWS-designed hardware and machine learning to deliver the best price performance at any scale.
#### Querying external data using Amazon Redshift Spectrum 
Using Amazon Redshift Spectrum, you can efficiently query and retrieve <u>structured and semistructured data</u> from files in Amazon S3 without having to load the data into Amazon Redshift tables. 
#### Querying data with federated queries
By using federated queries in Amazon Redshift, you can query and analyze data across operational databases, data warehouses, and data lakes.

### Amazon Athena 
Amazon Athena is an interactive query service that makes it easy to analyze data in Amazon S3 using standard SQL. Athena is serverless, so there is no infrastructure to manage, and you pay only for the queries that you run.
#### Using Athena Data Connector for External Hive Metastore
You can use the Amazon Athena data connector for external Hive metastore to query data sets in Amazon S3 that use an Apache Hive metastore. 
#### Using Amazon Athena Federated Query
If you have data in sources other than Amazon S3, you can use Athena Federated Query to query the data in place or build pipelines that extract data from multiple data sources and store them in Amazon S3. With Athena Federated Query, you can run SQL queries across data stored in relational, non-relational, object, and custom data sources.

### [Amazon Redshift Spectrum vs Amazon Athena](https://www.integrate.io/blog/amazon-redshift-spectrum-vs-athena/)
- Redshift Spectrum runs in tandem with Amazon Redshift, while Athena is a standalone query engine for querying data stored in Amazon S3.
- With Redshift Spectrum, you have control over resource provisioning, while in the case of Athena, AWS allocates resources automatically.
- The performance of Redshift Spectrum depends on your Redshift cluster resources and optimization of S3 storage, while the performance of Athena only depends on S3 optimization.
- Redshift Spectrum can be more consistent performance-wise while querying in Athena can be slow during peak hours since it runs on pooled resources.
- Redshift Spectrum is more suitable for running large, complex queries, while Athena is more suited for simplifying interactive queries.
- Redshift Spectrum needs cluster management, while Athena allows for a truly serverless architecture.

## Amazon Keyspaces
Amazon Keyspaces (for Apache Cassandra which is an open source NoSQL distributed database trusted by thousands of companies for scalability and high availability without compromising performance.) is a scalable, highly available, and managed Apache Cassandra–compatible database service.

## AWS Compute Optimizer vs AWS Trusted Advisor vs AWS Cost Explorer
- AWS Compute Optimizer
AWS Compute Optimizer helps avoid overprovisioning and underprovisioning four types of AWS resources—Amazon <u>Elastic Compute Cloud (EC2) instance types, Amazon Elastic Block Store (EBS) volumes, Amazon Elastic Container Service (ECS) services on AWS Fargate, and AWS Lambda functions—based on your utilization data</u>.
- AWS Trusted Advisor
AWS Trusted Advisor provides recommendations that help you follow AWS best practices. Trusted Advisor evaluates your account by using checks.
1. Cost optimization  
1. Performance  
1. Security  
1. Fault tolerance  
1. Service quotas  
- AWS Cost Explorer  
AWS Cost Explorer is a tool that enables you to view and analyze your costs and usage. 

## Monitor & Audit
### CloudTrail 
AWS CloudTrail is an AWS service that helps you enable operational and risk auditing, governance, and compliance of your AWS account. Actions taken by a user, role, or an AWS service are recorded as events in CloudTrail.

#### CloudTrail vs S3 access log
<u>Server access logging</u> provides detailed records for the requests that are made to a bucket but does <u>not have an option for choosing data events or log file validation</u>.

#### [Creating a trail for an organization](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/creating-trail-organization.html)
You can choose to edit an existing trail in the management account or delegated administrator account, and apply it to an organization, making it an organization trail. Organization trails log events for the management account and all member accounts in the organization. When you create an organization trail, a trail with the name that you give it is created in every AWS account that belongs to your organization.

## VPC
### AWS PrivateLink
AWS PrivateLink provides private connectivity between virtual private clouds (VPCs), supported AWS services, and your on-premises networks without exposing your traffic to the public internet. Interface VPC endpoints, powered by PrivateLink, connect you to services hosted by AWS Partners and supported solutions available in AWS Marketplace.

### Gateway endpoints
Gateway VPC endpoints provide reliable connectivity to <u>Amazon S3 and DynamoDB</u> without requiring an internet gateway or a NAT device for your VPC. Gateway endpoints do not use AWS PrivateLink, unlike other types of VPC endpoints.
PS: <u>Amazon S3 supports both gateway endpoints and interface endpoints</u>

### AWS Transit Gateway 
AWS Transit Gateway connects your Amazon Virtual Private Clouds (VPCs) and on-premises networks through a central hub.

### VPC peering
A VPC peering connection is a networking connection between two VPCs that enables you to route traffic between them using private IPv4 addresses or IPv6 addresses.

## [IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
### Permissions boundaries for IAM entities
AWS supports permissions boundaries for <u>IAM entities (users or roles)</u>. A permissions boundary is an advanced feature for using a managed policy to set the <u>maximum permissions that an identity-based policy can grant to an IAM entity</u>. An entity's permissions boundary allows it to perform only the actions that are allowed by both its identity-based policies and its permissions boundaries.

### Organizations SCPs
SCPs are applied to <u>an entire AWS account</u>. They limit permissions for every request made by a principal within the account.

## AWS Direct Connect
<u>You cannot change the port speed after you create the connection request. To change the port speed, you must create and configure a new connection.</u>
### Dedicated Connections
To create an AWS Direct Connect dedicated connection, you need the following information:
- AWS Direct Connect location
<u>Work with a partner</u> in the AWS Direct Connect Partner Program to help you establish network circuits between an AWS Direct Connect location and your data center, office, or colocation environment. They can also help provide colocation space within the same facility as the location.
- Port speed
The possible values are <u>1 Gbps, 10 Gbps, and 100 Gbps</u>.
### Hosted Connections
To create an AWS Direct Connect hosted connection, you need the following information:
- AWS Direct Connect location
<u>Work with an AWS Direct Connect Partner</u> in the AWS Direct Connect Partner Program to help you establish network circuits between an AWS Direct Connect location and your data center, office, or colocation environment.
- Port speed
For hosted connections, the possible values are <u>50 Mbps, 100 Mbps, 200 Mbps, 300 Mbps, 400 Mbps, 500 Mbps, 1 Gbps, 2 Gbps, 5 Gbps, and 10 Gbps</u>. Note that only those AWS Direct Connect partners who have met specific requirements may create a 1 Gbps, 2 Gbps, 5 Gbps or 10 Gbps hosted connection.

## AWS Storage Gateway
### File Gateway
#### Amazon S3 File Gateway
Store and access objects in Amazon S3 from <u>NFS or SMB file data with local caching.</u>
#### Amazon FSx File Gateway
Amazon FSx File Gateway optimizes on-premises access to fully managed, highly reliable file shares in Amazon FSx for Windows File Server. 
### Volume Gateway
Volume Gateway presents cloud-backed <u>iSCSI</u> block storage volumes to your on-premises applications. Volume Gateway stores and manages on-premises data in Amazon S3 on your behalf and operates in either cache mode or stored mode.
- Cached volumes 
You store your data in Amazon Simple Storage Service (Amazon S3) and <u>retain a copy of frequently accessed data subsets locally.</u>
- Stored volumes 
If you need low-latency access to your entire dataset, first configure your on-premises gateway to <u>store all your data locally.</u>

### Tape Gateway
A Tape Gateway provides cloud-backed virtual tape storage with its virtual tape library (<u>VTL</u>) interface.
