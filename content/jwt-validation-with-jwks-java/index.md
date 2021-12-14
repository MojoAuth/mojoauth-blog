---
title: "JWT validation with JWKs in Java"
date: 2021-12-13T14:41:23+05:30
coverImage: "jwk_validation_java.jpg"
author: "Ashish Sharma"
tags: ["Tutorial"]
description: "Learn how to create, sign and validate your JWT tokens using RS256 with JWKS endpoint in Java."
---

One of the benefits of [JSON Web Token (JWT)](https://mojoauth.com/blog/what-is-jwt/) is that you can validate a token using an easy cryptographic operation. JWT tokens are encoded and signed JSON.

The main advantage of allowing JWKS endpoint design is its ability to handle key rotation by external identity providers. Configuring this endpoint would enable you to programmatically find JSON web keys and allow the third-party identity providers to publish new keys without having the overhead of notifying every client application. This allows smooth key rollover and integration.

We use JWKS to expose the public keys used by the signing party to all the clients required to validate signatures. The example of a JWKS is something that looks like this:

```json
{
  "keys": [
    {
      "use": "sig",
      "kty": "RSA",
      "kid": "public:c424b67b-fe28-45d7-b015-f79da50b5b21",
      "alg": "RS256",
      "n": "sttddbg-_yjXzcFpbMJB1fIFam9lQBeXWbTqzJwbuFbspHMsRowa8FaPw44l2C9Q42J3AdQD8CcNj2z7byCTSC5gaDAY30xvZoi5WDWkSjHblMPBUT2cDtw9bIZ6FocRp46KaKzeoVDv3a0EBg5cdAdrefawfZoruPZCLmyLqXZmBM8RbpYLChb-UFO25i7e4AoRJ2hNFYg0qM-hRZNwLliDfkafjnOgSu7_w0WDInNzbUuy26rb_yDNGEIylXHlt0BKcMoeO3sJEwS5EDAkXkvz_7zQ6lgDQ4OLihC4QDwkp7dV2iQxvd7D-XEaSIahiqdHlqR8cUYOJANDVRIufAzzkyK8Shu_MXhVUW7hH3hNjlEh198bCWANHcsZWF2_V78Rl-UzCjsAFWtttf6FYpR9Kt-8ILM3aAYTAk3OwsvzSeqTtWLHp96QE8Bcm1AmZfPWzsd3PpLuSM_wfx4oxDWhdaKQ-HK1hCYLNv2Vity2uNC_tbGxOD9syRujWKS6wFf2b3jFEudV0NUXQ_1Beu8Ir0jHzuA_0D22wgiaSJ9svfpJ7XyoD6fxyHSyhpMsXIDLmnwOPKmD67MFQ7Bv_9H91KZmr34oeh6PVWEwb4wUAkDaCebo6h0gdMoDfZTq9Gn5S-Aq0-_-fIfyN9qrrQ0E1Q_QDhvqXx8eQ1r9smM",
      "e": "AQAB"
    }
  ]
}
```

In the above example some important fields are

• e: is the exponent for a standard pem

• n: is the moduluos for a standard pem

• alg: the signing algorithm.

• kid: a unique id for every key in the set.

## Verify JWT using JWKS

1. Create a public key using modulus and exponent from our jwks JSON.

```java
  private static PublicKey getPublicKey(String modulus, String exponent)  {
    try {
    byte exponentB[] = Base64.getUrlDecoder().decode(exponent);
          byte modulusB[] = Base64.getUrlDecoder().decode(modulus);
          BigInteger bigExponent = new BigInteger(1,exponentB);
          BigInteger bigModulus = new BigInteger(1,modulusB);



    PublicKey publicKey;

      publicKey = KeyFactory.getInstance("RSA").generatePublic(new RSAPublicKeySpec(bigModulus, bigExponent));

    return publicKey;

    } catch (InvalidKeySpecException | NoSuchAlgorithmException  e) {

      error = e.toString();
      return error
    }
    return null;

  }
```

2. Signed data and signature from JWT

```java
        String jwt = "<Enter your JWT token>"
        String signedData = jwt.substring(0, jwt.lastIndexOf("."));
        String signatureB64u = jwt.substring(jwt.lastIndexOf(".")+1,jwt.length());
        byte signature[] = Base64.getUrlDecoder().decode(signatureB64u);

```

3. Verify Signature

```java
        Signature sig = Signature.getInstance("SHA256withRSA");
        sig.initVerify(publicKey);
        sig.update(signedData.getBytes());
        boolean isVerify = sig.verify(signature);
```

Here if it returns true, this means our token is verified. This is all we need to validate our jwt token with jwks. Using JWKS for public key sharing provides an extra layer of convenience to this verification process.

MojoAuth is also using this extra security layer in their passwordless authentication process which is really easy to use. and also provide a java sdk for token verificaton using jwks.
