---
title: "Send Phone OTP In NODEJS"
date: 2022-03-10T14:41:23+05:30
coverImage: "send_phone_otp_in_nodejs.jpg"
author: "Ashish Sharma"
tags: ["Article"]
description: "In this article, we will explore setup twillo account and send phone otp in NodeJs."
---

If you are comfortable with programming in Node.js, you can send text messages with Twilio! Just follow these simple steps, and in the next minutes, you will have sent yourself an SMS using Node.js.

## Step 1: Creating a Twilio Account

Go to Twilio’s new account page, sign up for an account, and provide your details.
* Verify your email address
* Verify a phone number. 
* Getting a Phone Number for Twilio
 1.  Buy a Number page in [Console](https://www.twilio.com/console/phone-numbers/search).
 2. Enter the criteria for the number you need, and click Search.
 3. Search results in console will be displayed with the phone number, location, capabilities, type,  and price listed. 
 4. Click Buy to purchase a phone number for your project .

![Twillio](../assets/images/send-phone-otp-in-nodejs/twilio_credentials.png)

You get some free credits to start using Twilio to send text messages. 

## Step 2: Configuration of Twilio Account

You can find credentials you need TWILIO_ACCOUNT_SID , TWILIO_AUTH_TOKEN, and TWILIO_PHONE_NUMBER from the Twilio console, which helps us send simple HTTP POST (a Twillio API) for sending an SMS. 

We have this super simple library that provides Twilio implementation. Install the Twilio-node library from the terminal using npm:

```js
npm install Twilio
```
## Step 3: Implementation of Twilio 

Create a file named index.js and open it in your favorite text editor. At the top of the file, require the Twilio-node helper library. 

```js
const twilio = require('twilio');
```
As we already defined, the credentials of Twilio, which you can find in the console of Twilio.

try the code

```js
const accountSid = 'ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'; // Your Account SID from www.twilio.com/console
const authToken = 'your_auth_token'; // Your Auth Token from www.twilio.com/console

const twilio = require('twilio');
const client = new twilio(accountSid, authToken);

client.messages
  .create({
    body: 'Hello from Node',
    to: '+12345678901', // Text this number
    from: '+12345678901', // From a valid Twilio number
  })
  .then((message) => console.log(message.sid));

```

Wow, you have completed the SMS provider setup!! and for the implementation authentication, many steps are their many steps are there 

## Trial Limitations
Twilio Free Trial projects have some restrictions on their projects. For a list of the limitations, you may check out Twilio [Free Trial Limitations](https://support.twilio.com/hc/en-us/articles/360036052753-Twilio-Free-Trial-Limitations). At this time, when I am writing this blog, twillio gives 16$ in credits to a free trial account.

To remove these restrictions, you can upgrade to a paid project account which costs according to current plans.


## Conclusion : 
You can choose between using these steps and many more handlings or a simple 2 min setup for Passwordless authentication via phone OTP. Please make your choice and give us a chance because MojouAuth provides you with a secure and straightforward passwordless solution. 


