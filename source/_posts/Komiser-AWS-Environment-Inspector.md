---
title: 'Komiser: AWS Environment Inspector'
date: 2018-03-26 12:58:24
tags:
- DevOps
- AWS
- Infrastructure
- Monitoring
categories:
- DevOps
cover_index: /assets/Komiser-AWS-Environment-Inspector-Index.png
---
![](https://cdn-images-1.medium.com/max/1000/1*KrqHQzlltWXp3dVrI3j_Nw.png)

In order to build **HA** & **Resilient** applications in **AWS**, you need to assume that everything will fail. Therefore, you always design and deploy your application in multiple **AZ** & **regions**. So you end up with many unused AWS resources (Snapshots, ELB, EC2, Elastic IP, etc) that could cost you a fortune.

One pillar of [AWS Well-Architected Framework](https://d0.awsstatic.com/whitepapers/architecture/AWS_Well-Architected_Framework.pdf) is **Cost optimization**. That’s why you need to have a global overview of your **AWS Infrastructure**. Fortunately, AWS offers many fully-managed services like **CloudWatch**, **CloudTrail**, **Trusted Advisor** & **AWS Config** to help you achieve that. But, they require a deep understanding of AWS Platform and they are not straighforward.

![](https://cdn-images-1.medium.com/max/1000/0*OI-LFN2UFBJSp3SR.)

That’s why I came up with [Komiser](https://github.com/mlabouardy/komiser) a tool that simplifies the process by querying the **AWS API** to fetch information about almost all critical services of AWS like **EC2**, **RDS**, **ELB**, **S3**, **Lambda** … in real-time in a single **Dashboard**.

Note: To prevent excedding AWS API rate limit for requests, the response is cached in **in-memory cache** by default for **30 minutes**.

**Komiser** supported **AWS Services**:

![](https://cdn-images-1.medium.com/max/1000/0*N_JD580JK3AFTpf_.)

**Compute:**

* Running/Stopped/Terminated EC2 instances
* Current EC2 instances per region
* EC2 instances per family type
* Lambda Functions per runtime environment
* Disassociated Elastic IP addresses
* Total number of Key Pairs
* Total number of Auto Scaling Groups

**Network & Content Delivery:**

* Total number of VPCs
* Total number of Network Access Control Lists
* Total number of Security Groups
* Total number of Route Tables
* Total number of Internet Gateways
* Total number of Nat Gateways
* Elastic Load Balancers per family type (ELB, ALB, NLB)

**Management Tools:**

* CloudWatch Alarms State
* Billing Report (Up to 6 months)

**Database:**

* DynamoDB Tables
* DynamoDB Provisionned Throughput
* RDS DB instances

**Messaging:**

* SQS Queues
* SNS Topics

**Storage:**

* S3 Buckets
* EBS Volumes
* EBS Snapshots

**Security Identity & Compliance:**

* IAM Roles
* IAM Policies
* IAM Groups
* IAM Users

**1 — Configuring Credentials**

**Komiser** needs your AWS credentials to authenticate with AWS services. The **CLI** supports multiple methods of supporting these credentials. By default the CLI will source credentials automatically from its **default credential chain**. The common items in the credentials chain are the following:

**Environment Credentials:**

* *AWS_ACCESS_KEY_ID*
* *AWS_SECRET_ACCESS_KEY*
* *AWS_DEFAULT_REGION*

**Shared Credentials file** (**~/.aws/credentials**)

**EC2 Instance Role Credentials**

To get started, create a new [IAM user](https://console.aws.amazon.com/iam/home?region=us-east-1#), and assign to it this following **IAM policy**:

{% gist 9f9d50c4711868949144ae809ed7ccbb policy.json %}

Next, generate a new **AWS Access Key** & **Secret Key**, then update *~/.aws/credentials* file as below:

{% gist 6645507bf441fe47404607a8412ee411 credentials %}

**2 — Installation**

**2.1 — CLI**

Find the [appropriate package](https://github.com/mlabouardy/butler) for your system and download it. For linux:

{% codeblock %}
wget https://s3.us-east-1.amazonaws.com/komiser/1.0.0/linux/komiser
chmod +x komiser
{% endcodeblock %}

Note: The **Komiser CLI** is updated frequently with support for new AWS services. To see if you have the latest version, see the [project Github repository](https://github.com/mlabouardy/komiser).

After you install the **Komiser CLI**, you may need to add the path to the executable file to your **PATH** variable.

**2.2 — Docker Image**

Use the official **Komiser Docker Image**:

{% codeblock %}
docker run -d -p 3000:3000 -e AWS_ACCESS_KEY_ID=”” -e AWS_SECRET_ACCESS_KEY=”” -e AWS_DEFAULT_REGION=”” — name komiser mlabouardy/komiser
{% endcodeblock %}

**3 — Overview**

Once installed, start the **Komiser server**:

{% codeblock %}
komiser start — port 3000 — duration 30
{% endcodeblock %}

If you point your favorite browser to [http://localhost:3000](http://localhost:3000/), you should see **Komiser Dashboard**:

![](https://cdn-images-1.medium.com/max/1000/0*vWIHZGHyeemfh6Uk.)

Hope it helps ! The CLI is still in its early stages, so you are welcome to contribute to the project on [Github](https://github.com/mlabouardy/komiser).
