---
title: Dockerize a Facebook bot (Botkit) and deploy it to Heroku
date: 2018-04-28 14:02:46
tags:
- Bot
- BotKit
- Facebook
- NLP
- Docker
categories:
- Chatbot
cover_index: /assets/Dockerize-a-Facebook-bot-Botkit-and-deploy-it-to-Heroku-index.png
---
In this article, we will dockerize our Facebook bot built with Botkit Framework and deploy to Heroku using Heroku Container Registry.

#### First, build the bot project from Botkit

* Install Botkit on your computer :

{% codeblock %}
npm install -g botkit
{% endcodeblock %}

* Use Botkit CLI to build a facebook bot project :

{% codeblock %}
new --platform facebook
{% endcodeblock %}

![](https://cdn-images-1.medium.com/max/2000/1*6aDmJjPNp2_xSZ4DB27HfQ.png)

The bot project I named : **dockerized-bot** is created. You can get into it :

{% codeblock %}
cd dockerized-bot
{% endcodeblock %}

The bot project is a Node.js application based on an express.js server listening on port : 3000.

This information allows us to chose our Docker Image.

#### How docker works ?

It allows you to bundle your app with all its dependencies into a container. “A container image is a lightweight, stand-alone, executable package of a piece of software that includes everything needed to run it: code, runtime, system tools, system libraries, settings.” ~Docker.

Indeed, after building our container, it will be our packaged application.

To do so, our app should include a Dockerfile which will describe how will the container be build. Our image will be based on Docker Image for Nodejs (carbon)

{% codeblock %}
touch Dockerfile
{% endcodeblock %}

Open the file using a Text Editor then copy these line inside:

{% codeblock %}
    FROM node:carbon
    # Create app directory
    WORKDIR /usr/src/app
    # Install app dependencies
    # A wildcard is used to ensure both package.json AND package-lock.json are copied
    # where available (npm@5+)
    COPY package*.json .
    RUN npm install
    # If you are building your code for production
    # RUN npm install --only=production
    # Bundle app source
    COPY . .
    EXPOSE 3000
    CMD [ "npm", "start" ]
{% endcodeblock %}

You must be asking yourself what these lines reffer to. If not. Well, you should have !

These are instructions. Each calls an action :

* **FROM** is used to specify the Base Image from which yours will be built. the Base Image is stored in a DockerHub.
* **WORKDIR** sets the working directory for other instructions.
* **COPY** is used to copy files and directories from a source to a destinaiton.
* **RUN** runs a shell command
* **EXPOSE** tells Docker that the app is listening on a specific port at the runtime.
* **CMD** specifies the command to execute when running the container.

To prevet copying node_modules and log files to your image, a dockerignore file can be created.

{% codeblock %}
touch .dockerignore
{% endcodeblock %}

It contains :

{% codeblock %}
node_modules
npm-debug.log
{% endcodeblock %}

Build the image :

{% codeblock %}
docker build -t <your username>/dockerized-bot
{% endcodeblock %}

![](https://cdn-images-1.medium.com/max/1600/1*0gjNl871sKqyQpximntmMQ.png)

The image is succefuly built. We can see it docker images :

{% codeblock %}
docker images
{% endcodeblock %}

![](https://cdn-images-1.medium.com/max/2000/1*B94txU4-nf0C0orlPHeHgQ.png)

Now we should run the image to package our app and generate our container. This will happen in detach mode using `-d`flag. It means that the container will run in background. Also, we’ll use `-p` to match a public port to the private port of the container. Here we will use the same port 3000 in both cases.

{% codeblock %}
docker run -p 3000:3000 -d <your_username>/dockerized-bot
{% endcodeblock %}

_Note_: that you can use `--name` flag to name your container. In my case, Docker generated a name for my container.

![](https://cdn-images-1.medium.com/max/2000/1*QB81SmI4Yv4tj6ivvrMc9g.png)

As you can see, a container is running (`docker ps`lists the containers).

The app is running too. You can check it in your navigator :

![](https://cdn-images-1.medium.com/max/1600/1*xNXc7u8hsZUmsxbKgS-hFQ.png)

At this point, your bot project is dockerized. But since Facebook webhooks require a public adress. We will deploy our container to Heroku.

First you should login to heroku using Heroku CLI. (To install the CLI: `curl https://cli-assets.heroku.com/install-standalone.sh | sh`)

{% codeblock %}
heroku login
{% endcodeblock %}

Then install heroku-container-plugin which is a plugin that allows you to deploy Docker-based apps to heroku.

{% codeblock %}
heroku plugins:install heroku-container-registry
{% endcodeblock %}

You should login to the registry too :

{% codeblock %}
heroku container:login
{% endcodeblock %}

Create a Heroku app :

{% codeblock %}
heroku create dockerized-bot
{% endcodeblock %}

Set configuration vars to Heroku :

{% codeblock %}
heroku config:set page_token=$PAGE_TOKEN
heroku config:set verify_token=$VERIFY_TOKEN
{% endcodeblock %}

Your bot will need them to match your code to your Facebook App :

![](https://cdn-images-1.medium.com/max/1600/1*0HOxcWxhNaomoKMlSTd7PQ.png)

You can check that you app is well created on Heroku Dashboard :

![](https://cdn-images-1.medium.com/max/1600/1*A14kzNNtiIJmmW-gf-eANg.png)

Remember that the commands are executed inside of your working directory :

![](https://cdn-images-1.medium.com/max/2000/1*tijaPeRcFDe4mIDowzprDA.png)

All we have to do now is to push our container to the heroku app :

{% codeblock %}
heroku container:push web --app dockerized-bot
{% endcodeblock %}

Notice that we didn’t need a Procfile for our web app. This option is [Container Registry](https://dashboard.heroku.com/apps/dockerized-bot/deploy/heroku-container) and it can be an alternative for the Github/Dropbox/Heroku Git options.

![](https://cdn-images-1.medium.com/max/1600/1*viFGQlOAVw5YWQgNjrTtQQ.png)

On Heroku :

![](https://cdn-images-1.medium.com/max/1600/1*F27pyZoHykC9atf9yExXOA.png)

Once the app is pushed to Heroku, you can open the app :

{% codeblock %}
heroku open --app dockerized-bot
{% endcodeblock %}

![](https://cdn-images-1.medium.com/max/1600/1*0HFCtK8NvqfdRb6rIGbmZw.png)

The URL of your app on Heroku will be your base Webhook URL in Facebook.

You can push your image to Docker Hub which the Native Registry of Docker Cloud. Once you push your image to Docker Hub, it is available on Docker Cloud.

{% codeblock %}
docker login
docker tag <IMAGE_ID> <USERNAME>/dockerized-bot:firsttry
{% endcodeblock %}

`dockerized-bot` here is the name of your registry on Docker Hub.

This process can be automated using a CI/CD pipeline. <br> I’ll be writing another article for that purpose.

Thanks for reading ❤
