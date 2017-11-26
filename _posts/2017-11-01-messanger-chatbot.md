---
author: Ilyes
layout: post
title:  "ChatBot In A Microservices Architecture"
date:   2017-11-01
comments: true
category:
- AI
- Software-Architecture
---

A ChatBot designed to help new <a href="http://www.uvsq.fr/master-1-informatique-351265.kjsp" target="_blank">UVSQ's CS</a> students to explore and to know more about their course, designed in two versions : API.ai & ChatFuel and based in Microservices Architecture



## Demo
[<img src="/assets/img/blog/2017-11-01-Messanger-Chatbot/bloggif_59353fe79e3c6.gif" style="height: 30%;width: 30%;"/>](/assets/img/blog/2017-11-01-Messanger-Chatbot/bloggif_59353fe79e3c6.gif)
[<img src="/assets/img/blog/2017-11-01-Messanger-Chatbot/ezgif.com-video-to-gif2.gif" style="height: 30%;width: 30%;"/>](/assets/img/blog/2017-11-01-Messanger-Chatbot/ezgif.com-video-to-gif2.gif)

☕ Source code in [Github!](https://github.com/ilyeSudo/ChatBot_nodeJS_TER_Projet) ☕

## prerequisite
You need to download and install : [npm & nodejs](https://nodejs.org/en/download/)  -  [ngrok](https://ngrok.com/download)

## Development Setup
### 1. Developing a Facebook Messenger App
Before configuring your Messenger app on Facebook Developer, let’s create a bare minimum webhook with Node.js to get started.
#### Setting Up a Temporary Webhook Endpoint with ngrok
I choose ngrok to serves a localhost to a public URL because it is simple and easy to use. This URL will be used as a Messenger webhook endpoint during the development, so you don’t need to deploy to a server until the app is completed.
After downloading and installing it, run it with a port number, let’s use 5000 (it should be the same port as our main app is running)

``` $ ngrok http 5000 ```

When you start ngrok, it will display a public URL of your tunnel in the terminal. We will need the URL later when setting up the Facebook app. (In the screenshot, the URL is https://47ba4dd4.ngrok.io)

[<img src="/assets/img/blog/2017-11-01-Messanger-Chatbot/ngrok.jpg" style="height: 100%;width: 100%;"/>](/assets/img/blog/2017-11-01-Messanger-Chatbot/ngrok.jpg "ngrok")

#### Setting Up a Facebook App

Step1: You need a Facebook Page to set up your chat bot. Create one from facebook.com/pages/create. Choose a category, and select a sub category from the dropdown and fill out the required filed. Then click Get Started.

[<img src="/assets/img/blog/2017-11-01-Messanger-Chatbot/create-a-page.jpg" style="height: 100%;width: 100%;"/>](/assets/img/blog/2017-11-01-Messanger-Chatbot/create-a-page.jpg "Create a page")

Then create an app at developers.facebook.com/quickstarts.
Give it a name and click the button, then fill out the required info.

[<img src="/assets/img/blog/2017-11-01-Messanger-Chatbot/create-an-app.jpg" style="height: 100%;width: 100%;"/>](/assets/img/blog/2017-11-01-Messanger-Chatbot/create-an-app.jpg "Create an app")

Once your app is created, follow the steps to configure or skip it to your Dashboard.

[<img src="/assets/img/blog/2017-11-01-Messanger-Chatbot/FB-dashboard.jpg" style="height: 100%;width: 100%;"/>](/assets/img/blog/2017-11-01-Messanger-Chatbot/FB-dashboard.jpg "FB Dashboard")

Click Add Product from the left menu, then choose Messenger. Click Get Started.

[<img src="/assets/img/blog/2017-11-01-Messanger-Chatbot/create-an-app-add-product.jpg" style="height: 100%;width: 100%;"/>](/assets/img/blog/2017-11-01-Messanger-Chatbot/create-an-app-add-product.jpg "Create an App and add Product")

At the Token Generation, (1) choose the page you just created from the dropdown menu, and it will generate a token (2) that you will need to include in the code later (IMPORTANT: save the token in somewhere, you'll need it later).

Then, at the Webhooks, (3) click the Setup Webhooks button:

[<img src="/assets/img/blog/2017-11-01-Messanger-Chatbot/create-an-app-messenger.jpg" style="height: 100%;width: 100%;"/>](/assets/img/blog/2017-11-01-Messanger-Chatbot/create-an-app-messenger.jpg "Create an App Messenger")

In the dialog, fill out the (1) Callback URL with your ngrok URL, (2) the random string for validation, then (3) check messages.

[<img src="/assets/img/blog/2017-11-01-Messanger-Chatbot/create-an-app-messenger-webhook.jpg" style="height: 100%;width: 100%;"/>](/assets/img/blog/2017-11-01-Messanger-Chatbot/create-an-app-messenger-webhook.jpg "Create an App Messanger Webhook")

Click the Verify and Save. If you get a red icon with x at the Callback URL, it means the URL is not validated- so re-run ngrok and reset the webhook url. Otherwise, you are ready to go on.

## Running the App
The project has been built in two versions : API.ai version & ChatFuel version.

### 1st version : API.ai
API.ai allows developers to integrate your app with the AI system with speech-to-text and natural language processing.
Let’s get started by [sigining up](https://api.ai/).

[<img src="/assets/img/blog/2017-11-01-Messanger-Chatbot/apiai-create-agent.jpg" style="height: 100%;width: 100%;"/>](/assets/img/blog/2017-11-01-Messanger-Chatbot/apiai-create-agent.jpg "Create APIai Agent")

After doing so, go to the tab "Export & Import" and hit "IMPORT FROM ZIP" button and [download the zip configuration](https://drive.google.com/open?id=0B7hzSdAUtDKCRjRtRHFkWVUwcHM).

[<img src="/assets/img/blog/2017-11-01-Messanger-Chatbot/apiai_import.jpg" style="height: 100%;width: 100%;"/>](/assets/img/blog/2017-11-01-Messanger-Chatbot/apiai_import.jpg "import configuration in API.ai")

Now, You need your API.ai API key and API secret to use the service with your bot. From the menu, click the “Config” icon to get your API key "Client access token" and copy it.

[<img src="/assets/img/blog/2017-11-01-Messanger-Chatbot/apiai-apikey.jpg" style="height: 100%;width: 100%;"/>](/assets/img/blog/2017-11-01-Messanger-Chatbot/apiai-apikey.jpg "APIai-key")

Go to **Main_App (API.AI)/workspace/server/service.js** and set the **Client access token** key in the 5th line.

```const apiaiApp = apiai('Client access token');```

Now, let's set the token key that we saved from the facebook App earlier.
In the same file **Main_App (API.AI)/workspace/server/service.js** set the access_token at the **function send_reply(message){..}**
```
....{
   access_token: ""
    },.....
```
At this point, all what's left, is to run the microservices one-by-one by typing...

```node service.js```

### 2nd version : ChatFuel
Chatfuel is a bot builder for FB. It has a visual development environment, so you can click and add blocks of functionality like “Text Card” to send a text response and other blocks to send images etc..
It's much easier to integrate with facebook bots since it integrats FB apps automatically.
To take a look on the project's structure in ChatFuel, first, [log in Facebook](https://www.facebook.com/) with the following credentials.
```
Email : chatbot.chatfuel@gmail.com
Password : chatfuelchatfuel
```
Second, open [ChatFuel's app](https://chatfuel.com/) and hit "GET STARTED FOR FREE" or "LOG IN" to connect automatically with the running facebook account.

[<img src="/assets/img/blog/2017-11-01-Messanger-Chatbot/chatfuel_home_page.jpg" style="height: 100%;width: 100%;"/>](/assets/img/blog/2017-11-01-Messanger-Chatbot/chatfuel_home_page.jpg)


Once you're in, reset the webhook to the one you have generated by **ngrok** (https://47ba4dd4.ngrok.io in our case).

Edit all the folowing blocs : **all_profs** - **profs_plus_ues** - **all_ues** - **look_for_ue_details** - **my_group**

NB: wait a couple seconds after resetting to have the changes automatically saved. (until the ✔️ in the right corner appears)

[<img src="/assets/img/blog/2017-11-01-Messanger-Chatbot/InkedInkedInkedblocs_webhook_chatfuel_LI.jpg" style="height: 100%;width: 100%;"/>](/assets/img/blog/2017-11-01-Messanger-Chatbot/InkedInkedInkedblocs_webhook_chatfuel_LI.jpg)


At the end, let's run the microservices...

```node service.js```

### IMPORTANT:
- You need to have a facebook page associated with each version, so you got to have 2 in sum.
- After you clone this repo, go to each directry's root and run `npm install` to install all its dependencies.
- Run all the microservices's servers listed above for both versions, except **Main_App (API.AI)** and **Main_App (ChatFuel)**, run one of them according to the version you're testing.

## Contributing
- [Fork the repo](https://github.com/ilyeSudo/ChatBot_nodeJS_TER_Projet).
- Create your feature branch: git checkout -b my-new-feature.
- Commit your changes: git commit -am 'Add some feature'.
- Push to the branch: git push origin my-new-feature.
- Submit a pull request 😎
