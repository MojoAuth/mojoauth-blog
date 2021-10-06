---
title: "How To Implement MojoAuth Magic Link Authentication Using JWT in Node?"
date: 2021-10-06T11:31:23+05:30
coverImage: "How_To_implement_Magic_Link_Authentication.jpg"
author: "Vishal Sharma"
tags: ["Webauth"]
description: "When we implement passwordless authentication with is also known as "magic links", the user is sent an email with 
a link in it. This link allow's user to login directly by clicking on it.The user then clicks the button or link in the email and 
is automatically signed in to your application without asking for a passowrd."
---

## What is Magic Link Authentication?
Magic links provide a way for user's to authenticate directly without using a password. The whole process of authentication with a magic link involves around the email given by the user. User get's an email with link in it by clicking on it the user get directly signed into the application without asking for a password. 

<hr>

## How To Implement Magic Link Authentication Using JWT(JSON Web Token) In Node?

The passwordless system was presented by slack as a magic link which is really good alternative of basic email/password authentication system as user does not have to remember a password or even use a password manger.

In the very first step we need to generate JWT for example:-

```javascript
const jwt = require('jsonwebtoken')

const loginSecretKey = 'hhQqDlk/Hp+8d...' 

const jwtOptions = {
  issuer: 'magiclinkapp.com',
  audience: 'magiclinkapp.com',
  algorithm: 'HS256',
  expiresIn: '5m',
}

const generateLoginJWT = (user: User): Promise<string> => {
  return new Promise((resolve, reject) => {
    return jwt.sign(
      { sub: user.uuid },
      loginSecretKey,
      jwtOptions,
      (err, token) => {
        if (err) {
          reject(err)
        } else {
          resolve(token)
        }
      },
    )
  })
}
```
Now in the second step we use it for handling login request for example:-

```javascript
router.post('/login', (req, res) => {
  const email = sanitizeEmail(req.body.email)
  userRepository.fetchByEmail(email).then(user => {
    if (user) {
      generateLoginJWT(user).then(loginToken => {
        sendAuthenticationEmail(user, loginToken)
      })
    }
    res.redirect('/login-check-email')
  })
})
```

This should result in user receiving our email and clicking button that will redirect him to URL something 
like `https://magiclinkapp.com/login?token=...` From that point we need to handle the final request. To keep things simple let’s see how it can be done using JWT strategy for Passport. It will take care of checking token’s validity. Firstly, we need to configure the strategy.

```javascript

const JwtStrategy = require('passport-jwt').Strategy
const ExtractJwt = require('passport-jwt').ExtractJwt  

const jwtOptions = {
  secretOrKey: 'hhQqDlk/Hp+8d...', //the same one we used for token generation
  algorithms: 'HS256', //the same one we used for token generation
  jwtFromRequest: ExtractJwt.fromUrlQueryParameter('token'), //how we want to extract token from the request
}

passport.use(
  'jwt',
  new JwtStrategy(jwtOptions, (token, done) => {
    const uuid = token.sub
    userRepository.fetch(uuid).then(user => {
      if (user) {
        done(null, user)
      } else {
        done(null, false)
      }
    })
  })
)

```

And finally we can process the request and login user to the app if the token is valid.

```javascript

 router.get(
    '/login',
    (req, res, next) => {
      const { incorrectToken, token } = req.query

      if (token) {
        next()
      } else {
        res.render('login', {
          incorrectToken: incorrectToken === 'true',
        })
      }
    },
    passport.authenticate('jwt', {
      successReturnToOrRedirect: '/app',
      failureRedirect: '/login?incorrectToken=true',
    })
  )

```
  *I hope this blog gave you valid use case for using JWT in authentication scenario and the implementation details*.
 
