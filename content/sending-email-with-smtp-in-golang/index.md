---
title: "Sending Email With SMTP In Golang"
date: 2022-02-07T10:10:10+05:30
coverImage: "sending_email_with_smtp_in_golang.png"
author: "Ashish Sharma"
tags: ["Tutorial"]
description: "Learn how to send emails with smtp using golang"
---

## Introduction

Hello Developers, This blog teaches you how to send an email in golang quickly.
Also, the primary focus of this tutorial will be sending emails in golang. We will also look at relevant resources to go deeper for specific terminology.
There are already multiple libraries available on GitHub, which will help you send an email.
I discovered two packages with good open-source contributions, So I shortlisted the below.

- **[net/smtp](https://golang.org/pkg/net/smtp/)** : This is the golang core package.
- **[go-gomail/gomail](https://github.com/go-gomail/gomail)** : This is a community-driven project.

## net/smtp

It is a golang core package. Go offers you the net/stmp package out of the box.
All you need are two functions:

**PlainAuth:** it provides a simple authentication mechanism using username and password.

**SendMail:** This function sends an email that accepts five arguments.

- addr: its type is a string that contains an address and port number of the server (e.g., "smtp.gmail.com:587"),
- auth: auth that we got from the PlainAuth function.
  from: its type is a string containing the sender's mail address.
- to: a slice of a string containing the receiver's mail address.
- msg: it is a slice of the byte that contains the mail's body.

```go
package main

import (
	"log"
	"net/smtp"
)

func main() {

	// Setup host
	host := "smtp.mailazy.com"
	addr := "587"

	// Setup headers
	to := []string{"to@example.com"}
	msg := []byte("To: to@example.com\r\n" +
		"Subject: hello Gophers!\r\n" +
		"\r\n" +
		"This is the email body.\r\n")
	from := "sender@mailazy.com"

	// Set up authentication information.
	username := "smtp_username"
	password := "smtp_password"

	auth := smtp.PlainAuth("", username, password, host)

	err := smtp.SendMail( host+":"+addr, auth, from, to, msg)
	if err != nil {
		log.Fatal(err)
	}
}

```

## go-gomail/gomail

It is an efficient package to send an email-driven to the community.
It provides the LOGIN Authentication mechanism. Whenever it requires it automatically uses CRAM-MD5 when it's available. You don't need to specify explicitly like net/smtp.

```go
package main

import (
	"crypto/tls"
	"fmt"
	"gopkg.in/gomail.v2"
)

func main() {
	m := gomail.NewMessage()
	m.SetHeader("From", "sender@example.com")
	m.SetHeader("To", "bob@example.com")
	m.SetHeader("Subject", "Hello!")
	m.AddAlternative("text/html", "Hello <b>Bob</b>!")
	m.AddAlternative("text/plain", "Hello Bob!")

	// Send the email to Bob
	dialer := gomail.NewPlainDialer("smtp.mailazy.com", 587, "smtp_username", "smtp_password")
	if err := dialer.DialAndSend(m); err != nil {
		panic(err)
	}

}
```

## Conclusion

Sending an email is very easy using the Go language.
Whether you are going with the Core package or using a third-party library like go-gomail/gomail, you will need a few lines to send an email to someone.

Mojoauth provides the magic link and EmailOTP authentication system where you do not need to set up send email functionality for authentication. You can setup authentication in just 2 minutes to your application
