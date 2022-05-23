---
title: "Google Chatbot Setup With Golang"
date: 2022-05-23T14:41:23+05:30
coverImage: "google-chatbot-setup-with-golang.jpg"
author: "Mehul Sharma"
tags: ["Tutorial"]
description: "Learn how to setup a google chatbot asynchronously in golang with cards UI that posts messages to your G Suite Room."
---

# Google chatbot setup with Golang

This tutorial will cover how to build a bot that responds to pings (i.e., @) and sends messages to a chat room. The bot will run on a golang server and receive pings via an HTTP endpoint on a high level. 

We will be using payload in the HTTP request for responses to pings(synchronous), while Google Hangout Chat API will be used for bot-initiated messages(asynchronous)

## Google Chatbots 

When you think about a bot, you need to decide how you want to interact with it.

- Synchronously — Question and Answer (easy)

- Asynchronously — Bots pushing messages on their own (harder)

## Synchronous Bots

This is a simple task where you ping a bot, and the bot replies back. We won’t even need the complicated functions of the Google Chat API.  This means you can make a bot in a few lines of code that respond to HTTP POST requests.

```go
package main
import (
"encoding/json"
"Fmt"
"Log"
"net/http"
chat "google.golang.org/api/chat/v1"
)
func main() {
f := func(w http.ResponseWriter, r *http.Request) {
   if r.Method != http.MethodPost {
     w.WriteHeader(http.StatusMethodNotAllowed)
     Return
   }
   var event chat.DeprecatedEvent
   if err := json.NewDecoder(r.Body).Decode(&event); err != nil {
     w.WriteHeader(http.StatusBadRequest)
     w.Write([]byte(err.Error()))
     Return
   }
   switch event.Type {
   case "ADDED_TO_SPACE":
     if event.Space.Type != "ROOM" {
       Break
     }
     fmt.Fprint(w, `{"text":"thanks for adding me."}`)
   case "MESSAGE":
     fmt.Fprintf(w, `{"text":"you said %s"}`, event.Message.Text)
   }
 }
 log.Fatal(http.ListenAndServe(":8080", http.HandlerFunc(f)))
}
```

You can deploy your app with your preferred method and access it as an API. 

Testing the functionality of the above written code- 

```c
curl 'http:localhost:8080/' \
-H 'Content-Type: application/json' \
-d '{"type":"MESSAGE","message":{"text": "Hello!"}}'
```

## Asynchronous Bots

Asynchronous interaction is where the actual functionality of the Google Chat API comes in. You now have to figure out [service accounts](https://developers.google.com/chat/api/guides/auth/service-accounts). 

```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"

	"golang.org/x/oauth2"
	"golang.org/x/oauth2/google"
	chat "google.golang.org/api/chat/v1"
)

func getGoogleChatOauthClient(serviceAccountKeyPath string) *http.Client {
	ctx := context.Background()
	data, err := ioutil.ReadFile(serviceAccountKeyPath)
	if err != nil {
		log.Fatal(err)
	}
	creds, err := google.CredentialsFromJSON(
		ctx,
		data,
		"https://www.googleapis.com/auth/chat.bot",
	)
	if err != nil {
		log.Fatal(err)
	}

	return oauth2.NewClient(ctx, creds.TokenSource)
}

func GoogleNotification(data map[string]string) error {

	// Setup client to write messages to chat.google.com
	client := getGoogleChatOauthClient("YOURKEYSPATH")
	service, err := chat.New(client)
	if err != nil {
		return err
	}

	msgService := chat.NewSpacesMessagesService(service)

	msg := ChatCard("Title", "Subtitle", data)
	_, err = msgService.Create("spaces/YOUR_ROOM_ID", msg).Do()
	if err != nil {
		return err
	}

	return nil
}

func ChatCard(title string, Subtitle string, data map[string]string) *chat.Message {

	var widgets []*chat.WidgetMarkup
	for key, value := range data {
		widgets = append(widgets, &chat.WidgetMarkup{KeyValue: &chat.KeyValue{TopLabel: key, Content: value}})
	}

	cards := `{
       "cards": [
           {
               "header": {
                   "title": "%s",
                   "subtitle": "%s"
               },
               "sections": [
                   {
                       "widgets": []
                   }
               ]
           }
       ]
   }`
	outputString := fmt.Sprintf(cards, title, Subtitle)
	var message chat.Message
	json.Unmarshal([]byte(outputString), &message)

	message.Cards[0].Sections[0].Widgets = widgets
	return &message
}

func main(){
   f := func(w http.ResponseWriter, r *http.Request) {
       if r.Method != http.MethodPost {
            w.WriteHeader(http.StatusMethodNotAllowed)
            Return
        }

        dummyData := map[string]string{"data": "dummy data"}
        GoogleNotification(title, subtitle, dummyData)

   }
   
   log.Fatal(http.ListenAndServe(":8080", http.HandlerFunc(f)))
}
```

This way, you can call the custom function `GoogleNotification()` anywhere in your code with the described parameters title, subtitle and data which you want to display in the room. Else, you can make your bot ping to your G Suite room on an api call with your preferred data.

## Configuring Service Accounts

Login to the [developer console](https://console.developers.google.com). Create a new project, and enable Hangout Chat API. Under configuration, set:

- status: live
- bot name (this is how you will add and ping the bot)
- avatar
- description
- functionality: rooms
- connection settings - bot URL:
- permission: everyone in your domain

Restart your local server, and that’s it! Make sure you have your bot added to the chat room. 

## Use Cases

There could be multiple implications where you can use synchoronous or asynchoronous bots( or both ) to get results. 
It is upto your imagination to use the google chatbot which ever way you like. Some of the use cases are listed below-

- Setup a question answer application on google hangouts built in golang (Synchoronous)

- Get the data from your application in your G Suite room using Google Chat API (Asynchoronous)

- Setup a google bot (Synchoronous) to listen to pings sent by another bot in the room (Asynchoronous)


