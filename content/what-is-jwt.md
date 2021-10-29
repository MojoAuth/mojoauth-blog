---
title: "What is JWT? How does it work?"
date: 2021-10-28T13:33:58+05:30
coverImage: "what-is-jwt.jpg"
author: "Mehul Sharma"
tags: ["Article"]
description: "JWT or JSON Web Tokens are the new industry standards for securing APIs to and from the server. But what exactly is JWT? How does it work? Let us understand it more in detail.
"
---

## Introduction to JWT

JWT, or JSON Web Token, is an open standard(RFC 7519) set of claims to share security information or authorization/authentication requests between a client and a server. Each JWT contains encoded JSON objects. JWTs are signed using a cryptographic algorithm by the token's issuer to ensure that No one could alter the claims after the token is issued and later can be used by the receiving party to verify the token.

### What is JSON

JSON stands for JavaScript Object Notation used for data transfer. It is a lightweight format for storing and transporting data, often used when server sends the data to a web page.

### What are tokens

Now that we understand JSON, which is the data text format, the token is the string of data that usually represents the sender's identity. The servers use it to validate the sender's identity.

## Structure of JWT

The JWT is divided into three parts- Header, Payload, and Signature. Each piece is separated from the other using a dot(.)

```css
Header.Payload.Signature
```

### Header

The header is the part that identifies which algorithm is being used to generate the signature. It usually consists of two parts, the type of the token, which is JWT, and the hashing algorithm, like HMAC-SHA256.

```js
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload

The payload is the part that contains a set of claims. Claims are used to transmit information between two parties. There are no mandatory claims, but JWT specification defines a set of standard claims, such as iat and sub, which are short for issues at and subject. Custom Claims are also included in the payload if required.

```js
{
  "sub":"1234567890",
  "loggedInAs": "admin",
  "iat": 1422779638
}
```

### Signature

The signature is the part that securely validates the token. The encoded header(Base64url) is concatenated with the encoded payload(Base64url) and then signed by hashed with the algorithm already mentioned in the header with the secret-key only known to the issuer. Then this whole string is again encoded with Base64url to obtain the final signature.

Since the secret key is only available to the issuer, only it can issue new tokens with a valid signature. Thus, the signature ensures that the token hasn't been altered.

## Advantages of JWT

A JWT has many advantages over simple web tokens and Security Assertion Markup Language tokens.

- **Compact structure:** JWT uses JSON, which is less verbose than XML, making it smaller than the SAML token when it is encoded. Hence, the server could easily pass it to the HTTP and HTML environments.
- **Increased Security:** Since JWTs use a public/private key pair for signing and a hashed algorithm to protect the token's contents, it has the factor of increased security.
- **Convenience:** JSON parsers are standard in most programming languages because the parser can map them directly to objects. Contrarily, XML doesn't have a natural document-to-object mapping, thus making JWTs easier to use.
- **Compatibility:** JWT, widely used over the internet, is processed easier on users' devices.

## Use Cases of JWT

JWT or JSON Web Tokens are mainly used for authentication, authorization, and information exchange.

- **Authentication:** In the case of authentication, a JWT is returned when the user successfully logs in using their credentials. User can save it locally either in the local storage, session storage, or cookies.

    ```js
    {
    "access_token": "eyJhb...",
    "token_type": "Bearer",
    "expires_in": 3600
    }
    ```

- **Authorization:** Once the user successfully logs in, there may be a need to access data from the server. In such cases, the user can use JWT to retrieve the data. JWT should be sent by the user, typically in the Authorization header using the Bearer schema.

    ```js
    Authorization: Bearer eyJhbGci...<snip>...yu5CSpyHI
    ```

- **Information Exchange:** JWTs are widely used to exchange a set of information between parties. Since they are signed, you can be sure of the sender that he is genuine. Also, the signature part of the token allows you to make sure that the token has not been tampered with.

## Security aspects of JWT

Since the JWTs are digitally signed, they are widely trusted, and the information is verified. A secret(with the HMAC algorithm) or a public/private key pair is used to sign the tokens. The use of this public/private key pair ensures that the party who signed the token holds the private key, thus guaranteeing the tokens' integrity.

### Tampering with the signing algorithm

There are many algorithms used to sign the JWT tokens, one of which is none algorithm. This algorithm can be used as a gateway for attackers to tamper with the security of application.

**None algorithm** is supported by JWT in the cases where the integrity of the token is already verified by other means. With this algorithm, a JWT token can be issued by the server without any signature.

The attacker can use this vulnerablity to their advantage by setting the algorithm to 'none,' giving a null signature, and fooling the server to accept it as a valid token. However, many libraries have already been patched for this vulnerability and added security checks to reject the none algorithm.

### Leaking of the information

Before a received JWT is used, it should be appropriately validated using the signature. Although a successfully validated signature means that the token is not being tampered with, it doesn't guarantee that no one has seen the information contained in the token stored in plain text.

Therefore, sending sensitive information through JWT should be avoided. AdditionalThe user should take additional steps to ensure the JWTs are not intercepted, such as sending the tokens over HTTPS, following the best practice available using only secured and up-to-date libraries.

## Conclusion

Now that we know what JWT is and how they work, we can create more secure APIs for authentication, authorization, and information exchange. Such APIs using the best practices for JWT would be a lot more trusting and reliable.
One such platform that uses JWTs for authentication is MojoAuth.
