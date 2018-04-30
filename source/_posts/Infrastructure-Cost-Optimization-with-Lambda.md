---
title: Infrastructure Cost Optimization with Lambda
date: 2018-04-30 21:47:12
tags:
- Serverless
- AWS
- Lambda
- Goland
- CloudWatch
categories:
- Serverless
cover_index: /assets/Infrastructure-Cost-Optimization-with-Lambda-Index.png
---
Having multiple environments is important to build a continuous integration/deployment pipeline and be able to reproduce bugs in production with ease but this comes at price. In order to reduce cost of AWS infrastructure, instances which are running 24/7 unnecessarily (**sandbox **& **staging** environments) must be shut down outside of regular business hours.

The figure below describes an automated process to schedule, stop and start instances to help cutting costs. The solution is a perfect example of using **Serverless computing**.

![](https://cdn-images-1.medium.com/max/1600/1*wq1p8S7Zc6sOewlJFdEnwg.png)

_Note_: full code is available on my[GitHub](https://github.com/mlabouardy/cost-optimization).

2 Lambda functions will be created, they will scan all environments looking for a specific tag. The tag we use is named *‘***Environment***’*. Instances without an **Environment** tag will not be affected:

{% gist a99d04d4a29782a4dd9de6b90e4531dc main.go %}

The **StartEnvironment** function will query the **StartInstances** method with the list of instance ids returned by the previous function:

{% gist f231ff9dba90080394cf6c20d24ec59e start_env.go %}

Similarly, the **StopEnvironment** function will query the **StopInstances** method:

{% gist 50b2dc5d6ad9ce485325979f5b1f2150 stop_env.go %}

Finally, both functions will post a message to **Slack** channel for real-time notification:

{% gist de1eb318e22740d5f6b5feb3b1648ef3 slack.go %}

Now our functions are defined, let’s build the deployment packages (*zip *files) using the following Bash script:

{% gist 438760000d3f231c0e2636242b7b92e7 build.sh %}

The functions require an **IAM** role to be able to interact with **EC2**. The StartEnvironment function has to be able to *describe* and *start* EC2 instances:

{% gist 25b5c65c8e5948ee34281b0a22c1ae2a policy.json %}

The StopEnvironment function has to be able to *describe* and *stop* EC2 instances:

{% gist 2f1621f53d44f8e9b8e8b27c851d064b policy.json %}

Finally, create an IAM role for each function and attach the above policies:

{% gist 735fdd820c4356d39f13a2e2bcc645b3 iam.sh %}

The script will output the **ARN** for each IAM role:

![](https://cdn-images-1.medium.com/max/1600/1*vKoGQwKbRT2v0OC7k_gUUQ.png)

Before jumping to deployment part, we need to create a [Slack WebHook](https://my.slack.com/services/new/incoming-webhook/) to be able to post messages to Slack channel:

![](https://cdn-images-1.medium.com/max/1600/1*4o5gkMzVwFqrCIjBJqwy7Q.png)

Next, use the following script to deploy your functions to **AWS Lambda **(make sure to replace the IAM roles, Slack WebHook token & the target environment):

{% gist 71b6356ec04d07261c8cfb0526605aa5 deploy.sh %}

Once deployed, if you sign in to [AWS Management Console](http://console.aws.amazon.com/console/home), navigate to Lambda Console, you should see both functions has been deployed successfully:

**StartEnvironment:**

![](https://cdn-images-1.medium.com/max/1600/1*ejkQfrUjVJKc8C45DRWM3A.png)

**StopEnvironment:**

![](https://cdn-images-1.medium.com/max/1600/1*59NMitvf57QVdjVKJ79sPw.png)

To further automate the process of invoking the Lambda function at the right time. AWS **CloudWatch Scheduled Events** will be used.

Create a new CloudWatch rule with the below *cron* expression (It will be invoked everyday at 9 AM):

![](https://cdn-images-1.medium.com/max/1600/1*x04ekKEVFJnJPoe6OoRZ0A.png)

And another rule to stop the environment at 6 PM:

![](https://cdn-images-1.medium.com/max/1600/1*CFEVsycfSBhh71bIgYgNKQ.png)

_Note_: All times are **GMT** time.

**Testing:**

**a — Stop Environment**

![](https://cdn-images-1.medium.com/max/1600/1*28Ih4crWNlqO6-wtaS_dDA.png)

Result:

![](https://cdn-images-1.medium.com/max/1600/1*md_uLj6HAn9-GL0yqVnLAQ.png)

**b — Start Environment**

![](https://cdn-images-1.medium.com/max/1600/1*UQt5Xb9QkEAhgYKlm-N7dw.png)

Result:

![](https://cdn-images-1.medium.com/max/1600/1*MB8l8mrlMTZpU-cwqS8d0A.png)

The solution is easy to deploy and can help reduce operational costs.

*Full code can be found on my *[GitHub](https://github.com/mlabouardy/cost-optimization)*. Make sure to drop your comments, feedback, or suggestions below — or connect with me directly on Twitter *[@mlabouardy](https://twitter.com/mlabouardy)*.*