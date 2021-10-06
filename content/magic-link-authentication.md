---
title: "How To Implement MojoAuth Magic Link Authentication Using JWT in Node?"
date: 2021-10-06
coverImage: "magic-link-authentication.png"
author: "Vishal Sharma"
tags: ["Magic Link"]
description: "An implement of password-less authentication is known as "Magic Links".
---

## What is Magic Link Authentication?

Magic links provide a way for user's to authenticate directly without using a password something like we use OTP(One Time Password). The whole process of authentication with a magic link revolves around the email given by the user. User get's an email with link in it by clicking in the link user get directly signed into the application and services without asking for a password. 

As there are many organizations in the world who have been moved beyond password-based authentication to magic link authentication,they use magic login as an emerging popular consumer authentication method,depending on a company risk. Whether we use Slack magic link or a tumblr magic link a free and a easy way to get easily access our own applictions and services, using magic link login which frees us from remembering a long list of password.

<hr>

## How do Magic Link Works?

Magic link login revolves around three steps:-

1. The user enters their email address on their sign-in screen.<br/>
2. If it is a registered email address,then the user will receive an email with an magic link.<br/>
3. The user opens their email and clicks the magic link to complete the sign in process without asking for a password.

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
like `https://magiclinkapp.com/login?token=...` From this point we need to handle final request. To keep things simple and easy let’s us see how it can be done by using JWT strategy.

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

And now finally we will be able to process the request of login user to the app if the token is valid.

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
 
