---
title: Chatbot with Angular 5 & DialogFlow
date: 2018-03-26 18:59:05
tags:
- Bot
- DialogFlow
- Angular
- NLP
categories:
- Chatbot
cover_index: /assets/Chatbot-with-Angular-5-DialogFlow-Index.png
---
![](https://cdn-images-1.medium.com/max/1000/1*D8fHIsIuZ_QTPXcKJ-2o9A.png)

I have seen many posts on how to build a chatbot for a wide variety of collaboration platforms such as **Slack**, **Facebook Messenger**, **HipChat** … So I decided to build a chatbot from scratch to production using **Angular** latest release **v5.0.0,** **DialogFlow**, and** AWS**.

Here is how our [chatbot](http://smartbot-mlabouardy.s3-website-us-east-1.amazonaws.com/) will look like at the end of this post:

![](https://cdn-images-1.medium.com/max/1000/1*blUU1TvCMn87urlElADdTA.gif)

Note: This project is open source and can be found my [Github](https://github.com/mlabouardy/dialogflow-angular5).

To get started, create a brand new Angular project using the [Angular CLI](https://github.com/angular/angular-cli):

{% codeblock %}
ng new smartbot --style=scss
{% endcodeblock %}

**1 — Chatbot Architecture**

We will split out chat app in different components and each component will be able to communicate with others using attribute directives:

![](https://cdn-images-1.medium.com/max/1000/0*kvG6eG9yVEW4l6mM.)

**2 — Message Entity**

Create an empty class by issuing the following command:

{% codeblock %}
ng generate class models/message
{% endcodeblock %}

The message entity has 3 fields:

{% gist 8e5bef33ea0ab9a062dc2f29a3eb5ec3 message.ts %}

**3 — Message List Component**

Generate a new component:

{% codeblock %}
ng generate component components/message-list
{% endcodeblock %}

Now we can display the messages by iterating over them:

{% gist 8d5f88864edf884b744d77c5e67ba7b7 message-list.component.html %}

The code of this component should look like this:

{% gist 21a48baa643b3bcc821af6130855aa67 message-list.component.ts %}

Note the usage of *@app/models* instead of the relative path, its called **alias**. To be able to use aliases we have to add the *paths* properties to our *tsconfig.json* file like this:

{% gist 7c8e9dc6e9a896a3f0b2808713468cdb tsconfig.json %}

Note: I also added *@env* alias to be able to access environment variables from anywhere in our application.

**4 — Message Item Component**

Let’s build a component that will simply display a message in our message list:

{% codeblock %}
ng generate component components/message-item
{% endcodeblock %}

In *message-item.component.html*, add the following content:

{% gist aedb3c3a17ca3986ee7fdec64bbb484c message-item.component.html %}

The code of the component should look like this:

{% gist b30dbebeca4def80c338f689c1dd0427 message-item.component.ts %}

**5 — Message Form Component**

Let’s build the form that will be responsible for sending the messages:

{% codeblock %}
ng generate component components/message-item
{% endcodeblock %}

In the *message-form.component.html*, add the following content:

{% gist df4a1e4d80e5e21f74a0dd8c1db6ef7a message-form.component.html %}

And it’s corresponding typescript code in *message-form.component.ts:*

{% gist bd0a3854199f62e848735968b1e312f5 message-form.component.ts %}

The **sendMessage()** method will be called each time a user click on send button.

That’s it! Try it by yourself and you will see that it’s working.

{% codeblock %}
ng serve
{% endcodeblock %}

At this moment, you wont get any response, that’s where **NLP** comes to play.

**6 — NLP Backend**

I choose to go with [DialogFlow](https://dialogflow.com/) Sign up to **DialogFlow** and create a new agent:

Then, enable the **Small Talk** feature to have a simple chitchat:

![](https://cdn-images-1.medium.com/max/1000/0*cmi4XRBKu8AkKosS.)

Note: You can easily change the responses to the questions if you don’t like them. To go further you can create your own **Intents **& **Entities** as described in my [previous tutorial](http://www.blog.labouardy.com/bot-in-messenger-with-dialogflow-golang/).

Copy the **DialogFlow Client Access Token**. It will be used for making queries.

![](https://cdn-images-1.medium.com/max/1000/0*mus40T-LGCoWCKE9.)

Past the token into your *environments/environment.ts* file:

{% gist 601c2b4d880047f35885426ed441657c environment.ts %}

**7 — DialogFlow Service**

Generate a **DialogFlow Service** which will make calls the **DialogFlow API** to retreive the corresponding response:

{% codeblock %}
ng generate service services/dialogflow
{% endcodeblock %}

It uses the **DialogFlow API** to process natural language in the form of text. Each API request, include the **Authorization** field in the **HTTP header**.

{% gist bfccb836018a60b292e4db9bf03ebaae dialogflow.service.ts %}

Update the *sendMessage()* method in **MessageFormComponent** as follows:

{% gist 17acec777feb40408d419e2521dfc47a message-form.component.ts %}

Finally, in *app.component.html, *copy and past the following code to include the *message-list* and the *message-form* directives:

{% gist 1d7224ed37cc9e446ae9fee5bb59c759 app.component.html %}

**8 — Deployment to AWS**

Generate production grade artifacts:

{% codeblock %}
ng build --env=prod
{% endcodeblock %}

The build artifacts will be stored in the *dist/* directory

Next, create an **S3 bucket** with **AWS CLI**:

{% codeblock %}
aws s3 mb s3://smartbot-mlabouardy
{% endcodeblock %}

Upload the build artifacts to the bucket:

{% codeblock %}
aws s3 cp dist/ s3://smartbot-mlabouardy --recursive --grants read=uri=http://acs.amazonaws.com/groups/global/AllUsers
{% endcodeblock %}

Finally, turns website hosting on for your bucket:

{% codeblock %}
aws s3 website s3://smartbot-mlabouardy --index-document index.html
{% endcodeblock %}

If you point your browser to the [S3 Bucket URL](http://smartbot-mlabouardy.s3-website-us-east-1.amazonaws.com/), you should see the chatbox:

![](https://cdn-images-1.medium.com/max/1000/0*bUWQOyXc2cHLoHfR.)