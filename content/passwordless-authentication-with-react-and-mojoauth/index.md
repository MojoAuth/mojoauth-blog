---
title: "Passwordless Authentication with React and MojoAuth"
date: 2022-06-07T14:41:23+05:30
coverImage: "passwordless-authentication-with-react-and-mojoauth.png"
author: "Mehul Sharma"
tags: ["Tutorial"]
description: "Learn how to create your own passwordless authentication system with React and MojoAuth. "
---

Learn how to set up a Passwordless Authentication in React JS using MojoAuth. 

We will be building a React application with passwordless authentication using **MojoAuth Web SDK.** We will learn the benefits of using passwordless over traditional login setup and how it can provide better security than the latter.

## What is Passwordless Authentication

Passwordless Authentication, as the name suggests, is a way of authenticating your users without the need for passwords. Instead, an identifier is used to authenticate such as an email or phone number. Then a magic link or OTP is sent to the identifier, which is used to authenticate the user. Each request generates a new link or OTP, and invalidates any previous ones to provide a factor of security. 

Passwordless authentication also works as 2-factor authentication since it requires you to have your email or phone number with you.

Passwordless authentication is a better choice for your application as it eliminates the need for passwords. As the user creates an account on multiple sites, the complexity of passwords keeps getting lower. The users prefer convenience over security, so they often use weak or the same passwords for their multiple accounts, thus increasing the vulnerability of your application. 

## Who is using passwordless

Many companies have already boarded the passwordless ship, including slack, medium, and WhatsApp. Passwordless authentication can be found in both mobile and web apps but is essentially suited for mobile applications since they are already on their device which will receive the link or OTP, which will create a very intuitive workflow. 

Also, biometric authentication, another aspect of passwordless authentication, works very well with mobile apps since it relies on the face ID and fingerprint scanner and each device nowadays has at least one of the two above.

## What we will build

In this tutorial, we will get a basic idea of passwordless authentication and learn to include it in a React application. We will create an intuitive workflow application managing our routings for a better User Experience. 

To check out the working demo of the app built in this tutorial, visit [MojoAuth React Demo.](https://mojoauth-react-demo.netlify.app)

## Starting with React and MojoAuth

#### Prerequisites: 

- Basic knowledge of React.
- MojoAuth Account.
- MojoAuth web SDK.

For the demonstration, we will be using the MojoAuth library as the passwordless authentication service. Go ahead and create an account on MojoAuth and check the dashboard. After that, create a react project using the create-react-app and name it mojoauth-demo. 

```js
npx create-react-app mojoauth-demo
```

Now, install the mojoauth web SDK for creating the MojoAuth instance-

```js
npm install mojoauth-web-sdk
```
For the UI, we will be using the semantic UI react components, 

```js
npm install semantic-ui-react semantic-ui-css
```
After successful installation, import the semantic UI CSS to the index.js file.

```js
import 'semantic-ui-css/semantic.min.css'
```

For the routing, we will need the react-router-dom. Let's quickly install that.

```js
npm install react-router-dom
```

To get the stateID from the query, we will be using the npm package query-string.

```js
npm install query-string
```

In the MojoAuth Dashboard, get your Apikey and whitelist your project URL in the settings section. Since the local react server runs on port 3000,

```js
http://localhost:3000
```

In the src folder, create a file called config.js and add the following:-

```js
function getConfig() {
   return {
       api_key:
           process.env.REACT_APP_MOJOAUTH_APIKEY ||
           'YOUR_API_KEY',
       redirect_url:
               process.env.REACT_APP_REDIRECT_URL || 'http://localhost:3000/dashboard',
   };
}
 
export default getConfig();

 ```

Add your API key obtained from the mojoauth dashboard in place of YOUR_API_KEY in the config file. 

## Mojoauth passwordless authentication 

### Login.js

Now, let's create our Login interface. In the src folder, create another folder called Login. Inside Login, create a file called index.js. This file would contain our login code. 

Import react, your mojoauth web SDK and the config file in the index.js file.

```js
import React from "react"
import MojoAuth from "mojoauth-web-sdk"
import config from "./config"
```

In the use effect function, initialize the MojoAuth instance and call the signIn function,

```js
let mojoconfig={
               language:"en",
               source:[{type:'phone', feature:'otp'}, {type:'email', feature:'magiclink'}],
               redirect_url: config.redirect_url
           }
          
       const mojoauth = new MojoAuth(config.api_key,mojoconfig);
           mojoauth.signIn().then(payload=>{
               localStorage.setItem('React-AccessToken', payload.oauth.access_token)
               localStorage.setItem('React-Identifier', payload.user.identifier)
           })

```
We will save our access token and identifier in the local storage for further use. 
The current Login.js would look something like this:-

```js
import React from "react"
import MojoAuth from 'mojoauth-web-sdk'
import config from '../config'
const Login = () => {
      React.useEffect(()=>{
      
       let mojoconfig={
               language:"en",
               source:[{type:'phone', feature:'otp'}, {type:'email', feature:'magiclink'}],
               redirect_url: config.redirect_url
           }
          
       const mojoauth = new MojoAuth(config.api_key,mojoconfig);
                  localStorage.getItem('React-AccessToken')?navigate('/dashboard'):
           mojoauth.signIn().then(payload=>{
              
               localStorage.setItem('React-AccessToken', payload.oauth.access_token)
               localStorage.setItem('React-Identifier', payload.user.identifier)
             
           })
            
          
   },[])
  
   return(<>
   
   </>)
};
 
export default Login;
 
 

```
We will work on the return function in the next section.

### Dashboard.js

In the dashboard component, we want to log in when the user is redirected with the state ID to the redirect URL mentioned in the login component. 

Firstly, we import the required components,

```js
import React, { useEffect } from "react";
import MojoAuth from "mojoauth-web-sdk";
import * as QueryString from 'query-string';
import config from '../config'


```

 Now, in the Dashboard component, we get the state_id from the URL.

```js
const params = search
     ? QueryString.parse(search.search)
     :{}

```
In the useEffect function, we login using the stateID, by calling the signInWithStateID function,

```js
if (params.state_id) {
       const mojoauth = new MojoAuth(config.api_key);
       mojoauth.signInWithStateID(params.state_id).then((payload) => {
          
           localStorage.setItem('React-AccessToken', payload.oauth.access_token)
           localStorage.setItem('React-Identifier', payload.user.identifier)
 
       });
   }

```
The current Dashboard.js would look something like this:-

```js
 
import React, { useEffect } from "react";
import MojoAuth from "mojoauth-web-sdk";
import config from '../config'
 
import * as QueryString from 'query-string';

const Dashboard = (props) => {

const search = useLocation();
const params = search
     ? QueryString.parse(search.search)
     :{}
 useEffect(() => {
   if (params.state_id) {
       const mojoauth = new MojoAuth(config.api_key);
       mojoauth.signInWithStateID(params.state_id).then((payload) => {
          
           localStorage.setItem('React-AccessToken', payload.oauth.access_token)
           localStorage.setItem('React-Identifier', payload.user.identifier)
       });
   }
 }, []);
 
 return (
   <React.Fragment>
 
   </React.Fragment>
 );
};
 
export default Dashboard;

```

We will work on the return function in the next section.

## Creating our Interface

### Login.js

Now, we want to show the login form in a popup. For that, we will use the Modal from Semantic UI react. Create a state that would hold the opening and closing state of the popup.

```js
const [open, setOpen] = React.useState(false)
```
In the return function of the Login component, use the Modal to show the popup on the click on the login button.

```js
return(<>
   <div className='container'>
       <h1 className='main-title'>React Demo </h1>
       <div className='login-container'>
           <h1 className='title'>Welcome</h1>
           <h4 className='subtitle'>Login using MojoAuth</h4>
           <Modal
     onClose={() => setOpen(false)}
     onOpen={() => setOpen(true)}
     open={open}
     dimmer={true}
     basic={true}
     size='tiny'
     trigger={<Button className='login' primary>Login</Button>}
   >
        <div id='mojoauth-passwordless-form'></div>
   </Modal>
   <p className='footer'>By MojoAuth</p>
       </div>
       </div>
  
</>)

```
 For our MojoAuth instance to run only when the popup opens, we set a condition in the use effect function. 

```js
if(open){
        
           mojoauth.signIn().then(payload=>{
               setOpen(false);
               localStorage.setItem('React-AccessToken', payload.oauth.access_token)
               localStorage.setItem('React-Identifier', payload.user.identifier)
             
           })
       }
```

The current Login.js would look like this:-

```js
import React from "react"
import MojoAuth from 'mojoauth-web-sdk'
import { Button, Modal } from "semantic-ui-react";
import config from '../config'
const Login = () => {
   const [open, setOpen] = React.useState(false)
   React.useEffect(()=>{
      
       let mojoconfig={
               language:"en",
               source:[{type:'phone', feature:'otp'}, {type:'email', feature:'magiclink'}],
               redirect_url: config.redirect_url
           }
          
       const mojoauth = new MojoAuth(config.api_key,mojoconfig);
       if(open){
                      mojoauth.signIn().then(payload=>{
               setOpen(false);
               localStorage.setItem('React-AccessToken', payload.oauth.access_token)
               localStorage.setItem('React-Identifier', payload.user.identifier)
             
           })
       }
      
          
   },[ open])
  
   return(<>
   <div className='container'>
       <h1 className='main-title'>MojoAuth React Demo </h1>
       <div className='login-container'>
           <h1 className='title'>Welcome</h1>
           <h4 className='subtitle'>Login using MojoAuth</h4>
           <Modal
     onClose={() => setOpen(false)}
     onOpen={() => setOpen(true)}
     open={open}
     dimmer={true}
     basic={true}
     size='tiny'
     trigger={<Button className='login' primary>Login</Button>}
   >
        <div id='mojoauth-passwordless-form'></div>
   </Modal>
   <p className='footer'>By MojoAuth</p>
       </div>
       </div>
</>)
};
 
export default Login;
 
 

```

### Dashboard.js

In dashboard.js, we would show the Username(identifier) if the user is logged in. 
Also, we will show a logout button which would log the user out and remove the access token from the local storage.
In the return function, 

```js
return(
<React.Fragment>
    
     <div className='container'>
       <h1 className='main-title'>MojoAuth React Demo Dashboard </h1>
       <h3 className="main-subtitle">You have been Logged in Successfully!!</h3>
       <div className='login-container'>
           <h1 className='title'>Welcome </h1>
           <h4 className='subtitle'>{localStorage.getItem('React-Identifier')}</h4>
           <Button className='login' primary onClick={()=>{
       localStorage.removeItem('React-AccessToken');
     }}>Logout</Button>
   <p className='footer'>By MojoAuth</p>
       </div>
       </div>
 
    
   </React.Fragment>
)

```
The current Dashboard.js would look like this:-

```js
 
import React, { useEffect } from "react";
import MojoAuth from "mojoauth-web-sdk";
import config from '../config'
import { useLocation } from "react-router-dom";
import { Button } from "semantic-ui-react";
import * as QueryString from 'query-string';

 
const Dashboard = (props) => {
 
const search = useLocation();
const params = search
     ? QueryString.parse(search.search)
     :{}
 useEffect(() => {
   if (params.state_id) {
       const mojoauth = new MojoAuth(config.api_key);
       mojoauth.signInWithStateID(params.state_id).then((payload) => {
          
           localStorage.setItem('React-AccessToken', payload.oauth.access_token)
           localStorage.setItem('React-Identifier', payload.user.identifier)
 
       });
   }else if(!localStorage.getItem('React-AccessToken')){
    
   }
 }, []);
 
 return (
   <React.Fragment>
    
     <div className='container'>
       <h1 className='main-title'>MojoAuth React Demo Dashboard </h1>
       <h3 className="main-subtitle">You have been Logged in Successfully!!</h3>
       <div className='login-container'>
           <h1 className='title'>Welcome </h1>
           <h4 className='subtitle'>{localStorage.getItem('React-Identifier')}</h4>
           <Button className='login' primary onClick={()=>{
       localStorage.removeItem('React-AccessToken');
     }}>Logout</Button>
   <p className='footer'>By MojoAuth</p>
       </div>
       </div>
 
    
   </React.Fragment>
 );
};
 
export default Dashboard;

```

Add some css to give a good look to your demo:-

### App.css

```css
.container{
  text-align: center
}
.main-title{
 padding-top:20px;
}
.login-container{
 background-color: white;
 color: black;
 box-shadow: 0 0 25px #5e00cf;
 border-radius: 20px;
 max-width:320px;
 margin:10% auto;
 padding: 30px 40px;
 align-items: center;
 height: 300px
}
 
.title{
 text-align: center;
}
.subtitle{
 text-align: center;
 margin-bottom: 20px;
 color: rgba(76, 76, 76, 0.75)
}
button{
 width: 100%;
 margin-top: 15px !important;
}
.footer{
 font-size: 10px;
 color: gray;
 text-align: center;
 margin-top: 35px;
 
}
code {
 font-family: source-code-pro, Menlo, Monaco, Consolas, 'Courier New',
   monospace;
}
 
.login{
 background-color:#5e00cf !important;
}

```

## Managing our Routings

Once we are done with our login and dashboard code, next comes the routing part. In App.js, we would use the Routes from react-router-dom for routing. 

### App.js

```js
import './App.css';
 
import { Route, Routes} from 'react-router-dom';
import Dashboard from './Dashboard';
import Login from './Login';
 
 
 
const App=()=> {
  return (
  <Routes>
    <Route path='/' exact element={<Login/>}/>
      
    <Route path='/dashboard' exact element={<Dashboard/>}/>
  </Routes>
 );
}
 
export default App;
 
```

### Login.js

In Login.js, we would navigate to the dashboard once the user is logged in. For that, we will use the useNavigate function from react-router-dom.

We also want the user to navigate to the dashboard if the user is already logged in and tries to login again.


```js
localStorage.getItem('React-AccessToken')?navigate('/dashboard'):mojoauth.signin()

```

The final Login.js would look like this:-

```js
import React from "react"
import MojoAuth from 'mojoauth-web-sdk'
import { Button, Modal } from "semantic-ui-react";
import { useNavigate} from "react-router-dom";
import config from '../config'
const Login = () => {
   const [open, setOpen] = React.useState(false)
   const navigate=useNavigate()
   React.useEffect(()=>{
      
       let mojoconfig={
               language:"en",
               source:[{type:'phone', feature:'otp'}, {type:'email', feature:'magiclink'}],
               redirect_url: config.redirect_url
           }
          
       const mojoauth = new MojoAuth(config.api_key,mojoconfig);
       if(open){
           localStorage.getItem('React-AccessToken')?navigate('/dashboard'):
           mojoauth.signIn().then(payload=>{
               setOpen(false);
               localStorage.setItem('React-AccessToken', payload.oauth.access_token)
               localStorage.setItem('React-Identifier', payload.user.identifier)
              navigate('/dashboard')
           })
       }
      
          
   },[ open])
  
   return(<>
   <div className='container'>
       <h1 className='main-title'>MojoAuth React Demo </h1>
             <div className='login-container'>
           <h1 className='title'>Welcome</h1>
           <h4 className='subtitle'>Login using MojoAuth</h4>
           <Modal
     onClose={() => setOpen(false)}
     onOpen={() => setOpen(true)}
     open={open}
     dimmer={true}
     basic={true}
     size='tiny'
     trigger={<Button className='login' primary>Login</Button>}
   >
        <div id='mojoauth-passwordless-form'></div>
   </Modal>
   <p className='footer'>By MojoAuth</p>
       </div>
       </div>
</>)
};
 
export default Login;
 
```
### Dashboard.js

In the dashboard.js, after the user is logged out, or there's no access token in the local storage, we would want the user to navigate to the home page. 

The final dashboard.js would look like this:-

```js
import React, { useEffect } from "react";
import MojoAuth from "mojoauth-web-sdk";
import config from '../config'
import { useNavigate, useLocation } from "react-router-dom";
import { Button } from "semantic-ui-react";
import * as QueryString from 'query-string';
 
const Dashboard = (props) => {
const navigate = useNavigate();
const search = useLocation();
const params = search
     ? QueryString.parse(search.search)
     :{}
 useEffect(() => {
   if (params.state_id) {
       const mojoauth = new MojoAuth(config.api_key);
       mojoauth.signInWithStateID(params.state_id).then((payload) => {
          
           localStorage.setItem('React-AccessToken', payload.oauth.access_token)
           localStorage.setItem('React-Identifier', payload.user.identifier)
navigate('/dashboard')
       });
   }else if(!localStorage.getItem('React-AccessToken')){
     navigate('/')
   }
 }, []);
 
 return (
   <React.Fragment>
    
     <div className='container'>
       <h1 className='main-title'>MojoAuth React Demo Dashboard </h1>
       <h3 className="main-subtitle">You have been Logged in Successfully!!</h3>
       <div className='login-container'>
           <h1 className='title'>Welcome </h1>
           <h4 className='subtitle'>{localStorage.getItem('React-Identifier')}</h4>
           <Button className='login' primary onClick={()=>{
       localStorage.removeItem('React-AccessToken');
       navigate('/')
     }}>Logout</Button>
   <p className='footer'>By MojoAuth</p>
       </div>
       </div>
 
   </React.Fragment>
 );
};
 
export default Dashboard;

```


## Testing our App

Let's test our app now. With the **`npm start`**, the webpack would compile the application and run it on **localhost:3000.**

![Log-in](../assets/images/passwordless-authentication-with-react-and-mojoauth/log-in.png)

Once you click on the Login button, The mojoauth login popup will appear.

![MojoAuth-login](../assets/images/passwordless-authentication-with-react-and-mojoauth/mojoauth-login.png)

> Note:- For using the Login with phone number, refer to the **[how to add SMS authentication](https://mojoauth.com/docs/howto/add-sms-authentication/)** document. 

![Phone-Login](../assets/images/passwordless-authentication-with-react-and-mojoauth/phone-login.png)

Upon entering the user's email, magiclink is sent to the respective mail.

![Email-sent](../assets/images/passwordless-authentication-with-react-and-mojoauth/mail-sent.png)

The email is received right away and upon clicking on the link, the user is navigated to the redirect URL mentioned in the mojoauth instance.

![Email](../assets/images/passwordless-authentication-with-react-and-mojoauth/email.png)

> Note:- For redirection to work, don't forget to whitelist your URL. For more details, refer to **[how to whitelist your URL.](https://mojoauth.com/docs/configurations/redirection/)**

After successful login, you are navigated back to the dashboard. 

![LoggedIn](../assets/images/passwordless-authentication-with-react-and-mojoauth/logged-in.png)

To see the app working live, visit the [demo.](https://mojoauth-react-demo.netlify.app)

## A little more - Social Login

Passwordless authentication with MojoAuth is very easy, reliable, and secure. With just a couple of lines, we were able to build a fully functioning passwordless authentication system using the MojoAuth web SDK. 

Although passwordless is a great option, MojoAuth goes a little further and allows your users to log in with your desired social network with ease.
 
We can use the same lines of code to add social authentication like google, Facebook, and apple. The only thing you have to do is visit the MojoAuth dashboard and configure the social integrations. **[How to add social Authentication](https://mojoauth.com/docs/howto/social-login/)** document covers the topic in much more detail.

The UI changes would be instantly visible.

![Social-Login](../assets/images/passwordless-authentication-with-react-and-mojoauth/social-login.png)


## Wrapping up

Passwordless authentication is the new and improved alternative to the traditional username and password since it is easy to implement and increases the overall security while decreasing the cost for the company. 

It could be tricky to build a passwordless application from scratch, but MojoAuth reduces t most of the complexity by providing the out-of-the-box passwordless solution, while providing the flexibility to choose our favorite mode of authentication including magic link, OTP, and social login.


