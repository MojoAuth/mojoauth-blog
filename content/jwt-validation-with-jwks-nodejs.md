---
title: "JWT validation with JWKs in Node.js"
date: 2021-09-20T14:41:23+05:30
coverImage: "jwk_validation_node.jpg"
author: "Mehul Sharma"
tags: ["Tutorial"]
description: "Learn how to create, sign and validate your JWT tokens using RS256 with JWKS endpoint in Node.JS"
---

## What is a JWT?

[JSON Web Token (JWT)](https://mojoauth.com/blog/what-is-jwt/) is an open standard that defines how to transmit information between two parties in a compact and self-sustained way. This information is highly trusted and verified as it is signed digitally.

JWTs use a public/private key pair or a secret for signing. The advantage of JWT is in its feature; it just needs a simple cryptographic operation. The client needs to access the public/private key pair used for signing the JWT, and he can validate it using that key pair.

## What are JWKs?

​​A JSON Web Key (JWK) is a JSON data structure that represents a cryptographic key. JWKs are a set of keys shared between different services and are used to verify the JWT token from the authorization server.

JWKS exposes the public keys to all the clients who need to validate signatures that the signing parties use.

### To validate a JWT using JWKS in node js:

1. Create/have a JWKS endpoint.
2. Create/have a token endpoint and sign the token.
3. Retrieve the JWKS from the JWKs endpoint.
4. Extract the JWT from the request's authorization header.
5. Decode the JWT and grab the unique kid (Key ID) property of the token from the header.
6. Find the signature verification key in JWKS with a matching kid property.
7. Verify the token with the filtered JWKs.

### Creating a JWKS endpoint

Before actually validating the JWT token, you need to have an endpoint that returns the JSON Web Key set you need to verify your token. We will use the node package 'node-jose' to create the key set or key store.

```js
const fs = require("fs");

const jose = require("node-jose");

const keyStore = jose.JWK.createKeyStore();

keyStore.generate("RSA", 2048, { alg: "RS256", use: "sig" }).then((result) => {
  fs.writeFileSync(
    "Keys.json",
    JSON.stringify(keyStore.toJSON(true), null, "  ")
  );
});
```

> jose.JWK.createKeyStore creates an empty Keystore, and then we generate a JWK using RS256 algorithm, which is used for signing the token.
> The 'true' argument returns the public and the private section of the asymmetric key, which will be used later to sign the tokens.

Now lets return the public key to JSON using expressJs:

```js
router.get("/jwks", async (req, res) => {
  const ks = fs.readFileSync("keys.json");

  const keyStore = await jose.JWK.asKeyStore(ks.toString());

  res.send(keyStore.toJSON());
});
```

This will return the JSON web key set that will further be used to verify the token.

### Creating token endpoint and signing

To create a token endpoint, first retrieve the key with which you need to sign. Then, following JWT standards(compact: true and field type: 'jwt'), sign the token with the retrieved key using the jose.JWS.createSign() function.

```js
router.get("/tokens", async (req, res) => {
  const JWKeys = fs.readFileSync("keys.json");

  const keyStore = await jose.JWK.asKeyStore(JWKeys.toString());

  const [key] = keyStore.all({ use: "sig" });

  const opt = { compact: true, jwk: key, fields: { typ: "jwt" } };

  const payload = JSON.stringify({
    exp: Math.floor((Date.now() + ms("1d")) / 1000),
    iat: Math.floor(Date.now() / 1000),
    sub: "test",
  });

  const token = await jose.JWS.createSign(opt, key).update(payload).final();

  res.send({ token });
});
```

This will return a signed token with an 'expiry date' and 'issued at date' complying with the JWT standards.

### Validating the token

To validate the token, first, you need to get the JSON web key set from the JWKs endpoint. The token will be received as JSON in the validation endpoint in the body. Now, we need to decode the token to get the kid, which will be used to find the key we need to verify the signature. After that, we call the JWKs endpoint, retrieve the key set, and find the keys matching the kid of the token. The public key is then obtained by converting the retrieved key by using node package 'jwk-to-pem'. Once we have the token and the matched JWKs, the only thing left is verifying the token.

```js
router.post("/verify", async (req, res) => {
  let resourcePath = "token/jwks";

  let token = req.body;

  let decodedToken = jwt.decode(token, { complete: true });

  let kid = decodedToken.headers.kid;

  return new Promise(function (resolve, reject) {
    var jwksPromise = config.request("GET", resourcePath);

    jwksPromise
      .then(function (jwksResponse) {
        const jwktopem = require("jwk-to-pem");

        const jwt = require("jsonwebtoken");

        const [firstKey] = jwksResponse.keys(kid);
        const publicKey = jwktopem(firstKey);
        try {
          const decoded = jwt.verify(token, publicKey);
          resolve(decoded);
        } catch (e) {
          reject(e);
        }
      })
      .catch(function (error) {
        reject(error);
      });
  });
});
```

Validating JWT with JWKS is the most secure authentication method as the signing occurs as a cryptographic operation. If you don't want to do any of the above code, you can use MojoAuth verifyToken function to verify your token.

Add project dependency and MojoAuth SDK using npm by running the following command in the command line:

"`npm install express body-parser mojoauth-sdk` "

Upon installation, you will find MojoAuth Node.js SDK under the node module.

Get your Apikey from the MojoAuth Dashboard and configure the MojoAuth instance using the API key

```js
var ma = require('mojoauth-sdk')(“<<Apikey>>”);
```

Call the MojoAuth verifyToken() function and pass the MojoAuth JWT Token to verify the token.

```js
var jwtToken = "<Your JWT Token>";

ma.mojoAPI
  .verifyToken(jwtToken)
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```
