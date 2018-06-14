---
title: Build a new Skill with AWS Lambda to automatically build a CircleCI bot project 
date: 2018-06-14 10:30:00
tags:
- AWS
- Alexa
- Lambda
- Serverless
categories:
- Serverless
cover_index: /assets/alexa-lambda.jpg
---

Amazon Alexa : Build a new Skill with AWS Lambda to automatically build a CircleCI bot project
Alexa is the Amazon intelligent voice-control agent. It is a cloud based vocal service. To take advantage of Alexa, Amazon offers voice products which are headless devices with no face or UI. For instance : Echo, Echo Plus, Echo Dot, Tap.
![](https://cdn-images-1.medium.com/max/800/1*qd4j69sbk5wHci8K7fJz1Q.png)

Amazon Echo Dot, Echo 1st and 2nd generations (left to right)
Keep in mind that Alexa is the technology that allows us to speak to the device and have it respond the rightest way to our request. Alexa has Skills which are commands that can be executed. That said, every time you ask Alexa a request, there’s a essentially a Skill triggered.

## How does Alexa work ?
Echo devices are triggered when you start your request with a magic word : “Alexa”. They stream your vocal message to the Cloud where it is analysed. Anything that comes after “Alexa” is processed with Natural Language Processing System build by Amazon. According to the intent detected, an Alexa Skill it invoked and a task is executed. Finally a female voice answers from the speaker. The key word “Alexa” is by default. It can be changed to another word like “Amazon”, “Computer”.

This article was written in order to highlight the build of a new Alexa Skill using AWS Lambda Service. More specifically, we will ask Alexa to deploy a Botkit Chatbot project (Node.js application) automatically with a CI pipeline to Heroku using AWS Lambda Function in Node.js.

![](https://cdn-images-1.medium.com/max/800/1*HOOpwB3mkqYWkLoxjO_xiQ.png)

On the left of the diagram, we have an Echo Dot. It could be any Echo device. When a user speaks to Echo, the command is passed to the Interaction Model running in the Alexa Service. The Interaction Model is used to detect what the user wants to say : The intent. The Interaction Model is connected to a Lambda Function via Endpoints. Depending on the Intent detected, Lambda Function running on AWS executes a task (</code>) and the result is returned to the Interaction Model. Finally, Echo Dot receives the response to Speak.

## Requirements :
- Amazon.com acount.
- Amazon Web Services account.
- A curious brain and about 20 minutes of time.

## Setting up Accounts
### Amazon Developer Account
I’ll assume that you already have an Amazon shopping account. Access to Amazon Developer Page and sign in using your Amazon shopping account :

![](https://cdn-images-1.medium.com/max/800/1*1_iAk1luDp3gFzUOsLN9xA.png)

After signing in, fill out the profile information and accept the app’s distribution agreement. You’ll be directly taken to your Dashboard :

![](https://cdn-images-1.medium.com/max/800/1*XNLcPDNo7Pd_TstzibGLdA.png)

### Amazon Web Services Account
Again, I’ll assume that you already have an AWS account. Sign in and head over to AWS Console :

![](https://cdn-images-1.medium.com/max/800/1*868mKt50SLvwK-uVC__4Xg.png)

This is your AWS Dashboard where you can choose services you want to work with.

Now that we have set up he accounts. We can start with creating the Interaction Model behind our Skill.

## Creating the Interaction Model
An Interaction Model is a system of rules and flows that handle the inputs and outputs while interacting with Alexa Voice Service. This is where we define the intents that our Skill may support including the utterances a user might say to call an intent.

### Create a Skill
In your Amazon Developer Dashboard, access to Alexa Skills Kit.

![](https://cdn-images-1.medium.com/max/800/1*774_4p7G5Kag3ZMM69ibTQ.png)

This is the new console for Alexa Skills Kit. To my view, it is more ergonomic and user-friendly than the previous console. To know more about what had changed, head over to About The Developer Console.

To create a new Skill, you’ll need a Skill Name and a Language. The language should be the same as your Alexa’s language. In my case, SkillName = “Deployment” and Language : “English(US)”.

Depending on what your Skill will do, you will decide on how this Skill will integrate with Alexa. Alexa Skills Kit supports many types of Skills. They call them : Models. Every Skill Model has an Interaction Model. You can define a new Custom Interaction Model or use a Pre-built Models in which requests and utterances are already defined.

At the moment, there are 4 Skill Models:
- Custom Interaction Model : More control over the Skill (Intents, Utterances…)
- Smart Home Skills : A pre-built Model to build Skills for smart homes.
- Flash Briefing Skills : A pre-built Model to provide a customer with flash briefing.
- Video Skills : A pre-built Model to provide TV shows and movies for customers.

We’ll go for a Custom Interaction Model because it’s the best way to understand Models and define the structures we need :

![](https://cdn-images-1.medium.com/max/800/1*7zX74o1bupauX0gJzgABaw.png)

Choose Custom Model as the model of the skill
To build the Interaction Model, 4 steps are required :
- Find an Invocation Name : Invocation Name is the identifier used to trigger the Skill. For this purpose, the user should say : “Alexa, ask/open invocation_name.
- Intents, Samples and Slots : User’s spoken input will be mapped to intents you define in your Skill. They reflect the meaning of what he wants to say. Samples are a set of sentences mapped to an intent. They are used by The Natural Language Processing System to help Alexa learn you intents. Slots are types of items that are not covered by Amazon (entities). You also can define a Dialog Model (optional) which is a flow describing the steps for a multi-turn conversation.
- Build Model.
- Endpoints : The Endpoint will receive POST requests when a user interacts with your Alexa Skill. You can link your Skill with Lambda (Recommended) or you can host your endpoint using an HTTPS web service that you manage.

### Invocation
Let’s call our Alexa Skill using the invocation : “automated deployment” :

![](https://cdn-images-1.medium.com/max/800/1*DA_Xcu_Tv4CmH_RKMxotrg.png)

#### Set up invocation name of your Skill
Notice that even with a Custom model, we already have some built-in Intents such as : Cancel, help or Stop :

![](https://cdn-images-1.medium.com/max/800/1*pzkidrkoxDPrzbNzbHqEzQ.png)

### Built-in intents of Custom Model
Add the new Intent (Custom Intent)and set up some utterances. I called the intent “deployProject” (No slots are needed for the moment):

![](https://cdn-images-1.medium.com/max/800/1*V3vDfDMF12qoKS6Jvgh-OQ.png)

#### Create a new intent and set up some utterances
Save the model then Build it. It might take few moments. Once built, you should see a Success push notification at the bottom right of your screen.

Last step is the Endpoint Configuration. We will come back to this configuration later because for now, we have no lambda Endpoint.

## Creating The Lambda Function to handle Skill’s intent
Now that we’ve built the Interaction Model, we’ll head over to AWS to create an Intent Handler using Lambda Service.

### Lambda Function
AWS Lambda is a compute service. It takes your code, runs it in AWS without requiring any servers’ provisioning or management. It executes your code only when needed and scales automatically.

Note : Our Lambda Function will be triggered via the Alexa Skill Kit. Keep in mind that this component is only available in some regions. So, make sure the region you’re working on is this list: US East (N. Virginia), US West (Oregon), EU (Ireland), Asia Pacific (Tokyo).

![](https://cdn-images-1.medium.com/max/800/1*bFH-grUvDcyU6rE3ORaNNA.png)

#### Create a Lambda Function

Search for Lambda in AWS Services
Select Lambda and create a new Lambda Function from scratch. I choose to name my Lambda function: “deploymentFunction”, in Node.js and set the role to lambda_basic_execution to manage execution permissions :

![](https://cdn-images-1.medium.com/max/800/1*FcuwmXfxHB7ao3ZuMiionw.png)

As mentioned in the “Author from Scratch” info, the created Lambda function is a simple “Hello world” example.

We can test the current function in order to make sure that everything works. To do so, create a new test configuration. All you need is to set up a test as follows :

![](https://cdn-images-1.medium.com/max/800/1*v-h_eDBN3cHKDqifZ8VZNg.png)

#### Configure Test
Save the test configuration and hit “Test”. A success message will be displayed :

![](https://cdn-images-1.medium.com/max/800/1*5P7-iEyEnZqhXyMU01UC8A.png)

The code of this simple function is :

```
exports.handler = (event, context, callback) => {
    // TODO implement
    callback(null, 'Hello from Lambda');
};
```

Note that the function should expose a handler object.

To link this Lambda function to our Alexa Skill, we should add the Alexa Skills Kit Trigger as an Input of this function. To do so, Select Alexa Skills Kit from the Triggers list on the left. You will be asked to set up the Alexa Skills Kit Trigger with an Endpoint. It’s the one generated by the Interaction Model we have created on the first section

![](https://cdn-images-1.medium.com/max/800/1*fgn04ZsjU8jXcb9ixX2hzQ.png)

#### Endpoints in Alexa Skills kit
The endpoint needed is the one on the Red frame. At the same time, copy your Lambda function URL and add i as input in Default Region on the Green frame.

![](https://cdn-images-1.medium.com/max/1000/1*tYJ6BX3S6-4bgpM-et11KA.png)

Now that we set the endpoints, the connection between our Alexa Skill and Lambda Function should be fine.
One last thing to take care of is updating Lambda function to return an appropriate JSON for Alexa Skill.
The new Lambda function is :

```
exports.handler = (event, context, callback) => {
    var responseJson =  {
    version: "1.0",
    response: {
      outputSpeech:
       {
         type: "PlainText",
         text: "Hello from Lambda",
       },
      shouldEndSession: true
      },
      sessionAttributes: {}
    };
    callback(null,responseJson);
};
```

Don’t forget to save your Lambda.

Go back to Alexa Skills Kit, and re-build the Model. Once done, Test your Skill on the Test Tab :

![](https://cdn-images-1.medium.com/max/800/1*z-xvT0YKCYpf3PGutFvsgg.png)

#### Test Your Skill : Alexa open deployment
Indeed, we received the expected response while invoking our Skill using the sentence : “Alexa open deployment” where deployment is the Invocation name of the Skill.

Now that we know how to build a simple Hello world Lambda function, let’s upgrade it to handle Intents.

```
exports.handler = (event, context, callback) => {
    try {
        if (event.request.type === 'LaunchRequest') {
            callback(null, buildResponse('Hello from Lambda'));
        } else if (event.request.type === 'IntentRequest') {
            const intentName = event.request.intent.name;

            if (intentName === 'deployProject') {
                callback(null, buildResponse("I've heard a deployment intent"));
            } else {
                callback(null, buildResponse("Sorry, i don't understand"));
            }
        } else if (event.request.type === 'SessionEndedRequest') {
            callback(null, buildResponse('Session Ended'));
        }
    } catch (e) {
        context.fail(`Exception: ${e}`);
    }
};

function buildResponse(response) {
    return {
        version: '1.0',
        response: {
            outputSpeech: {
                type: 'PlainText',
                text: response,
            },
            shouldEndSession: true,
        },
        sessionAttributes: {},
    };
}
```

In this Lambda Function, we first verify the type of the request :
- LaunchRequest is created with an invoke message similar to : “Aexa, open deployment”. In this case, I set up the response to “Hello from Lambda”.
- IntentRequest is created using a message like “Alexa ask deployment to deploy project”. The response will be “I’ve heard a deployment Intent”.

Save The function and go back to test the Skill’s responses.

![](https://cdn-images-1.medium.com/max/800/1*RfVuS_oeMoyBSYecjA0YBQ.png)

#### Test Intent Request
At this point, we’ve built a Skill with Alexa Skills Kit. We also created a Lambda Function and we glued the whole thing using Endpoints.

The Last step will be to implement the automated deployment using a CI tool : CircleCI. I won’t go into details about how to build a CI/CD Pipeline using CircleCI. Everything you need to know is on this Post.

We will use CircleCI’s API endpoint for building a branch in a GitHub project. This means that we will need a Node.js module to make external requests. But, Lambda Functions only know few modules. Therefore, we will need a node_module/ directory that contains the needed modules. We will use ‘request’ module to make the HTTP call.

Beforehand, we wrote Lambda Functions directly on the Online Editor. Another way to build Lambda Function is to upload a .zip file in which you put your function and dependencies. (By default, Lambda waits for an index.js file. But it can be any file providing that you change the default settings of your Lambda function from index.handler to file_name.handler)

```
npm install request
zip -r index.zip index.js node_modules
```

Our new Lambda Function looks like :

```
var request = require('request');

exports.handler = (event, context, callback) => {
    try {
        if (event.request.type === 'LaunchRequest') {
            callback(null, buildResponse('Hello from Lambda'));
        } else if (event.request.type === 'IntentRequest') {
            const intentName = event.request.intent.name;

            if (intentName === 'deployProject') {
                buildCircleCiProject(function (err, result) {
                    if(!err) callback(null, buildResponse("The build was launched. make sure you take a look. See you."));
                    else callback(null, buildResponse("Please check your build logs. Something went wrong."));
                });
            } else {
                callback(null, buildResponse("Sorry, i don't understand"));
            }
        } else if (event.request.type === 'SessionEndedRequest') {
            callback(null, buildResponse('Session Ended'));
        }
    } catch (e) {
        context.fail(`Exception: ${e}`);
    }
};

function buildResponse(response) {
    return {
        version: '1.0',
        response: {
            outputSpeech: {
                type: 'PlainText',
                text: response,
            },
            shouldEndSession: true,
        },
        sessionAttributes: {},
    };
}

function buildCircleCiProject(callback) {
    var options = {
        url: `https://circleci.com/api/v1.1/project/github/Raniazy/bot-backend/tree/dockerized-bot?circle-token=${process.env.CIRCLE_CI_TOKEN}`,
        headers: {
            'User-Agent': 'alexa-skill'
        }
    };
    request.post(options, function(error, response, body){
        if(error){
            callback("ERROR");

        } else {
            callback(null,"SUCCESS");
        }
    });
}
```

All I had to do is add a new function that calls a POST request to CircleCI in order to build a project.
Remember the Test button on Lambda Function. To perform a test, we should prepare all the inputs needed by our function.
There’s a pre-configured Test sample automatically created for each of Skill’s intent:
```
{
  "session": {
    "new": true,
    "sessionId": "amzn1.echo-api.session.[unique-value-here]",
    "attributes": {},
    "user": {
      "userId": "amzn1.ask.account.[unique-value-here]"
    },
    "application": {
      "applicationId": "amzn1.ask.skill.[unique-value-here]"
    }
  },
  "version": "1.0",
  "request": {
    "locale": "en-US",
    "timestamp": "2016-10-27T18:21:44Z",
    "type": "IntentRequest",
    "requestId": "amzn1.echo-api.request.[unique-value-here]",
    "intent": {
      "name": "deployProject"
    }
  },
  "context": {
    "AudioPlayer": {
      "playerActivity": "IDLE"
    },
    "System": {
      "device": {
        "supportedInterfaces": {
          "AudioPlayer": {}
        }
      },
      "application": {
        "applicationId": "amzn1.ask.skill.[unique-value-here]"
      },
      "user": {
        "userId": "amzn1.ask.account.[unique-value-here]"
      }
    }
  }
}
```

The Test should reply to all requirements of your Lambda Function. In the example, the test defines the request as an IntentRequest (request.type = ‘intentRequest’) and the intent.name as ‘deployProject’. It’s a simulation of the request sent by Alexa to Lambda Function.

![](https://cdn-images-1.medium.com/max/1000/1*zl4VRyLx9ALJiSpqJ0B15g.png)

#### Test result
To test the whole thing on Alexa :
![](https://cdn-images-1.medium.com/max/800/1*EpCcEZAO4iaKNwU1wILRwQ.png)

Heading over to CircleCI platform, I found a running build :

![](https://cdn-images-1.medium.com/max/1000/1*pPw4A9B2qg4G5BOqvvTRrQ.png)

Success ! When I asked Alexa to deploy the project, the Alexa Skill Kit trigger sent an event to Lambda Function that was waiting for a signal having the exact ‘deployProject’ intent to call CircleCI.

Now the project is well dockerized and automatically deployed to Heroku.
Code is available on this [GitHub](https://github.com/Raniazy/alexa-skills) repository.

In this post, we’ve seen the full path from Setting up Amazon accounts to automated project deployment. We have created an Alexa Skill using Alexa Skills Kit and Linked it to a Lambda Function created with AWS Lambda service. To go further, you can update you Interaction Model with Slots : Project Name. This way, you can specify to Alexa which project to deploy.

