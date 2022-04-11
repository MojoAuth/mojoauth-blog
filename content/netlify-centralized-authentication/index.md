---
title: "Netlify Centralized Authentication using MojoAuth "
date: 2022-04-10T14:41:23+05:30
coverImage: "netlify-centralized-authentication.jpg"
author: "Mehul Sharma"
tags: ["MojoAuth"]
description: "Learn how to setup a Netlify Centralized Authentication using MojoAuth with a working demo."
---
 

In this blog you will learn about netlify centralized authentication using mojoauth with a working Demonstration. 


# Getting Credentials

Lets get started with creating an account on MojoAuth.

## Register at MojoAuth

Here you just need to follow three simple steps:

- Login to [MojoAuth](https://mojoauth.com/dashboard/signin) Dashboard:

![Login](../assets/images/netlify-centralized-authentication/login.png)

- Create your Fist Project by adding your **Website URL** and **Project Name** as displyed in the below screen:

![Project](../assets/images/netlify-centralized-authentication/project.png)

**For the website URL, Enter the url of the protected/profile page of your website.** 

- Get your API Credentials, The API key and API Secret are used to interact with MojoAuth's APIs.

![Dashboard](../assets/images/netlify-centralized-authentication/dashboard.png)

# Your Website

Create your website frontend using html and js or any of the frontend frameworks like ReactJS and NextJS. In your frontend, create a login page and your profile/protected page.
## Login page

Now, On the Login page of your website, Add a login Button. Link it to the Netlify Authentication link with the redirect Url as a query parameter. The redirect URL would be the profile/protected page of your website. 

**Check our [Demo](https://netlify-centric-authentication-demo.netlify.app) for more details on how to setup the login button.**



## Profile page

On the profile page, Add the mojoauth JavaScript SDK in the head of your html and follow the mentioned steps-

```js
<script
 src="https://cdn.mojoauth.com/js/mojoauth.min.js"
 type="text/javascript"
></script>
```
OR 

Install mojoauth npm package using **`npm install mojoauth-web-sdk`** and import it into your frontend framework. 

```js
import MojoAuth from 'mojoauth-web-sdk'
```


Create MojoAuth instance with your api key. 


```js
const mojoauth = new MojoAuth("Your MojoAuth API Key")
```

Add the following function to log in using the state ID received from the netlify Authentication. 

```js
mojoauth.signInWithStateID().then( response => { console.log(response)})
```
**The token will be received in the response.**

# Netlify Authentication 

Create your own netlify authentication page using mojoauth for your Login using plain html and js or any of the frontend frameworks like ReactJS and NextJS.

On this netlify Authentication page, add MojoAuth javascript SDK in the head and follow the mentioned steps:
```js
<script
 src="https://cdn.mojoauth.com/js/mojoauth.min.js"
 type="text/javascript"
></script>
```
OR

Install mojoauth npm package using **`npm install mojoauth-web-sdk`** and import it into your frontend framework. 

```js
import MojoAuth from 'mojoauth-web-sdk'
```

 Get the redirect URL which you set on your login button from the search parameters. 

```js
const params = new URLSearchParams( window.location.search );

```
 Create MojoAuth instance with your api key and pass the source as an object.

```js
const mojoauth = new MojoAuth("Your MojoAuth API Key", {
 language: "language_code",
 redirect_url: params.get(‘redirect_url’),
 source: [{ type: "email", feature: "magiclink" }],
})
```

Add the following div on your web page where you want the MojoAuth passwordless login form to be rendered

```js
<div id="mojoauth-passwordless-form"></div>
```

**MojoAuth passwordless login form will be rendered in the above div on your web page.**

Add the MojoAuth passwordless login with the following method. 


```js
mojoauth.signIn().then(response => window.close())
```

**We will close this window once the response is received and you will be redirected to your site once you click on the magiclink received on your mail.**

# Demo

1. Clone mojoauth-demo repository from [github](https://github.com/MojoAuth/mojoauth-demo). 

2. Navigate to netlify authentication demo directory. You will find netlify-auth and website-template folders.

3. In the netlify Auth directory, add your APIKEY in config.js file. Alternatively, add it as an environment variable in netlify.

4. In the website template directory, add APIKEY, REDIRECT_URL and MOJOAUTH_NETLIFY_URL which would be your netlify url for mojoauth authentication in the config.js file. Alternatively add them as environment variables in netlify. 

5. Set up your website and host them on netlify. Your netlify centralized authentication is now live.

**Visit [netlify-centric-authentication-demo.netlify.app](https://netlify-centric-authentication-demo.netlify.app) for a working demo.**

- In the Demo repository, create react app has been used for both netlify auth and website template. You can use the frontend of your choice. Alternatively you can also use plain html and js to set up your frontend. Learn more about html and JS setup on [MojoAuth Docs](https://mojoauth.com/docs/guides/html-and-js). 


