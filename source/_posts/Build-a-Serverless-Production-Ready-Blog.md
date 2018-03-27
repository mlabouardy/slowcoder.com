---
title: Build a Serverless Production-Ready Blog
date: 2018-03-27 11:46:40
tags:
- Serverless
- AWS
- Cloud
- Web
categories:
- Serverless
cover_index: /assets/Build-a-Serverless-Production-Ready-Blog-Index.png
---
![](https://cdn-images-1.medium.com/max/720/1*LnVkpxJgIzB_58DAHrg8-w.png)

Are you tired of maintaining your **CMS** (WordPress, Drupal, etc) ? Paying expensive hosting fees ? Fixing security issues everyday ?

![](https://cdn-images-1.medium.com/max/720/0*epQp_10DEgGvI33A.)

I discovered not long ago a new blogging framework called [Hexo](https://hexo.io/)** **which let you publish **Markdown** documents in the form of blog post. So as always I got my hands dirty and wrote this post to show you how to build a production-ready blog with **Hexo** and use the **AWS S3** to make your blog **Serverless** and pay only per usage. Along the way, I will show you how to automate the deployment of new posts by setting up a **CI/CD** pipeline.

To get started **Hexo** requires **Node.JS** & **Git** to be installed. Once all requirements are installed, issue the following command to install **Hexo CLI**:

{% codeblock %}
npm install -g hexo-cli
{% endcodeblock %}

Next, create a new empty project:

{% codeblock %}
hexo init slowcoder.com
{% endcodeblock %}

Modify blog global settings in *_config.yml* file:

{% gist a6d7a0a48b159cc9c553200aae3cf0b0 _config.yml %}

Start a local server with “**hexo server**“. By default, this is at [http://localhost:4000](http://localhost:4000/). You’ll see Hexo’s pre-defined “**Hello World**” test post:

![](https://cdn-images-1.medium.com/max/720/0*M97KDwf3ulIxdZqy.)

If you want to change the default theme, you just need to go [here](https://hexo.io/themes/) and find a new one you prefer.

I opt for [Magnetic Theme](https://github.com/klugjo/hexo-theme-magnetic) as it includes many features:

* Disqus and Facebook comments
* Google Analytics
* Cover image for posts and pages
* Tags Support
* Responsive Images
* Image Gallery
* Social Accounts configuration
* Pagination

Clone the theme **GitHub** repository as below:

{% codeblock %}
git clone https://github.com/klugjo/hexo-theme-magnetic themes/magnetic
{% endcodeblock %}

Then update your blog’s main* _config.yml* to set the theme to *magnetic*. Once done, restart the server:

![](https://cdn-images-1.medium.com/max/720/0*seGeOToeSEOaEGHh.)

Now you are almost done with your blog setup. It is time to write your first article. To generate a new article file, use the following command:

{% codeblock %}
hexo new POST_TITLE
{% endcodeblock %}

Now, sign in to **AWS Management Console**, navigate to **S3 Dashboard** and create an **S3 Bucket** or use the **AWS CLI** to create a new one:

{% codeblock %}
aws s3 mb s3://slowcoder.com
{% endcodeblock %}

Add the following **policy** to the **S3 bucket** to make all objects public by default:

{% gist f760ce832525670210af6998107d479d policy.json %}

Next, enable **static website hosting** on the S3 bucket:

{% codeblock %}
aws s3 website s3://slowcoder.com — index-document index.html
{% endcodeblock %}

In order to automate the process of deployment of the blog to production each time a new article is been published. We will setup a **CI/CD** pipeline using **CircleCI**.

Sign in to [CircleCI](https://circleci.com/) using your **GitHub** account, then add the *circle.yml* file to your project:

{% gist 6ed9875ed82fedf4a9492e0f7475023f circle.yml %}

Note: Make sure to set the **AWS Access Key ID** and **Secret Access Key** in your Project’s Settings page on **CircleCI** (*s3:PutObject* permission).

Now every time you push changes to your **GitHub** repo, **CircleCI** will automatically deploy the changes to **S3**. Here’s a passing build:

![](https://cdn-images-1.medium.com/max/720/0*dkTJjHSTQAGENEl0.)

Finally, to make our blog user-friendly, we will setup a custom domain name in **Route53** as below:

![](https://cdn-images-1.medium.com/max/720/0*EbePGgVR7oD6TP_A.)

Note: You can go further and setup a **CloudFront Distribution** in front of the **S3** bucket to optimize delivery of blog assets.

You can test your brand new blog now by typing the following adress [http://slowcoder.com](http://slowcoder.com/) :

![](https://cdn-images-1.medium.com/max/720/0*ebPs7tKhxtUcqCkt.)