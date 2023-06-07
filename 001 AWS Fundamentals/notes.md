# AWS Fundamentals 

## PUBLIC INTERNET Zone (ie. gmail, e-commerce) 

- "AWS PUBLIC" Zone (ie. s3. Network connected to Public Internet.)
- AWS PRIVATE Zone (ie. EC2. Behind an API gateway. Needs to pass through AWS Public)
- Region - all services are available
- Edge Location - selected services, used to mitigate high latency. There are more edge locations than regions.
- Globally Resilient (ex. IAM, Route53. One centralized data for all)
Region Resilient (ex. EC2. Each region has different copies)
AZ Resilient (prone to hardware failure. Other AZ inside the same region will not have a copy)

<br>

## VPC Introduction

- VPC = networks inside AWS. It is within 1 acct and 1 region.
- VPC is Regionally resilient.
- VPC CIDR (172.31.0.0/16) - range of IPS the VPC can use. 
- VPC can be subdivided into Subnets. Set during creation and can never be changed. Subnets are within CIDR

### 2 types of VPC

 - Default
    - max 1 per region (can be removed or recreated)
    - initially created by AWS
    - a lot of preconfiguration already done by AWS, means its inflexible
    - always configured with 1 CIDR only and it's always 172.31.0.0/16
    - 1 /20 Subnet -> 1 AZ
    - by default already has Internet gateway(IGW), Security Group(SG), NACL, Subnets, assigned public IPv4 addresses
- Custom
    - can have multiple per region
    - 100% private by default
    - everything needs to be configured
    - used more in real life
    - can have multiple CIDR

</br>

## EC2 Introduction

- EC2 is IAAS. Provides Virtual Machines
- EC2 is in private AWS zone.
- EC2 is configured to launch in single VPC subnet, so it is AZ resilient.
- can connect to public zone, but the VPC will need to support public access
- Has On-demand billing per second.
-  local storage, on-host storage, EBS (Elastic Block Store) = network storage available to instance
- port 3389 RDP (windows instances)
- port 22 SSH (linux instances)
- Needs key pair to connect (private and public key)
    - private - used by user to match with public key
    - public - AWS keep in EC2 instance

Access EC2 local ssh:<br>

    cd <A4L path location>
    ssh -i "A4L.pem" ec2-user@ec2-35-172-200-244.compute-1.amazonaws.com

### An instance can have **states**:

- Running
    - Bills charges for hardware components
- Stopped
    - You're not billed charges (except storage)
- Terminated 
    - One way change, non reversible.
    - You're not billed charges even for storage

<br>

### Amazon Machine Image (AMI) = image of an EC2 instance

- AMI -> EC2 -> AMI

- AMI Contains Permissions
    - Public = everyone allowed
    - owner = implicitly allowed because they own it
    - explicit - specific accounts allowed, granted by owner
- AMI Contais Volumes
    - Root Volume = kinda like boot drives like C:/ in Windows
    - It has at least 1 volume (root volume) but there can be more (data volumes)
- AMI has Block Device Mapping
    - Connects volumes and how they are presented to OS (which is root? data?)
    - mapping volume -> device ID which OS understands

<br>

## S3 Introduction

- Global Storage Platform
- Regional Resilient
- Public Service (in AWS public zone)
- Allows unlimited data and multi-user
- Objects - files
    - key = file name
    - value = content
- Buckets - containers of objects
    - key can be used to map to bucket
    - created in specific region (primary home region).
    - can be placed in other region, but unless configured, the default is it will stay in 1 region
    - blast radius = region
    - bucket name needs to be GLOBALLY unique
        - 3 - 63 characters, all lower case, no underscore
        - starts with lower case or number
        - can't be IP formatted (1.1.1.1)
    - can store unlimited number and size of data
    - has a flat structure. No folders within folders.
    - Bucket number: 100 soft limit for an AWS account, 1000 hard limit (via support request)
    - a bucket can have UNLIMITED number of objects

### S3 is an object store, not file or block.
 - it has no folder (not a file)
 - can't be mounted (not a block)
 - can be accessed by multiple users (not like a block that only limits one user at a time)

 ### Amazon Resource name (ARN) 
 - used as identifiers

        example: =============
        arn:aws:s3:::koalacampaign20231234567

<br>

## CloudFormation Introduction

- can be a YAML or json
- When you give the template to cloudformation, CFN will create a stack.
- Stack has all the logical resources that the template tells it to contain.
- per 1 logical resource in your stack -> 1 physical resource is created

<br>

### Different Parts of CloudFormation Templates

- Resources : the only mandatory part of the CloudFormation template.
- Description : free text field.
    - needs to immediately follow AWSTemplateFormatVersion if used.
- Metadata : controls grouping, order, labels etc. (how it's presented in AWS console)
- Parameters : where you can add fields which prompt the user for more information.
- Mappings : it allows you to create lookup tables
- Conditions : Allows decision making in the template
- Output : shows output once the template is finished. 

<br>

## CloudWatch Introduction

- collects and manages operational data

- 3 main jobs
    - Metrics (CPU, disk usage, visitors/sec etc.)
        - Gathers data natively
        - non-native data (outside AWS) needs CloudWatch agent
        - monitoring things inside products which arent exposed to AWS needs CloudWatch agent as well (ie. which processes are running inside EC2 instance)
    - CloudWatch Logs
    - CloudWatch Events
        - Generates an event in response to something 

### Namespace 
- container for monitoring data
- has rule set for names
- all AWS data goes to AWS namespace: AWS/[service] 

### Metric 
- time ordered set of datapoints

### Datapoint
- consist of timestamp and value

### Dimension
- instance ID + instance type
- used to filter datapoints for a particular instance only

### Alarms
- Connected to a metric.
- based on configuration, it will take an action based on that metric
- Has the following states:
    - OK - do nothing
    - ALARM - do something (SNS or action)
    - INSUFFICIENT_DATA - the alarm is ongoing gathering data before it determines OK or ALARM state

Inside EC2, you can do stress test using this command:

        sudo yum install stress -y

        // -c 1 = t2.micro has 1 virtual cpu
        // -t 3600 = run the stress for 3600 seconds
        stress -c 1 -t 3600 
        
## Shared Responsibility Model

- AWS is responsible for the security "of" the cloud.
- Customer is responsible for security "in" the cloud.

![Alt text](pic/SharedResponsibilityModel-2.png)

<br>

## High-Availability vs Fault-Tolerance vs Disaster Recovery

- Abbreviated as HA, FT and DR.

### High Availability (HA)
- Minimise any outages
- ensure agreed level of operation performance.
- maximizes a systems online time.
- Doesn't care about user experience. There might be some hassle for the user but it's tolerable as long as there's availability.


### Fault Tolerance (FT)
- Operate through faults
- much more complex and expensive vs HA
- enables system to continue operating properly in the event of the failure of some components
- Not having FT means potentially putting life at risk. That's the key difference with HA. For example, you would rather have a plane with Fault Tolerance rather than High Availability.

### Disaster Recovery (DR)
- Used when HA and FT did not work
- a set of policies/tools/procedures to enable recovery following a natural or human-induced disasters.