---
layout: post
title: What I know about Oauth 2.0
date: 2017-04-04 11:00
tag:
- oauth
category: blog
---

## What is Oauth ?

* An authorization protocol
* For third party application to do things on behalf of the user

## Oauth Roles

There are four roles, namely

* Resource Owner
    the user basically, who wants to do a particular action on a third party application
* Resource Server
    server which stores the credential of the user.
* Client
    third party application,
* Authorization Server
    capable of issuing access token to the client after successful authentication of the resource owner

Consider that a service `generatememes.com` exists which creates memes. A user `devguy` wants to use this service and posts the memes generated to his Twitter account.

In this case, following will be the identifications

* Resource Owner - devguy (**D**)
* Resource Server - Twitter (**T**)
* Client - generatememes.com (**G**)
* Authorization Server - Twitter (**T**)

## Flow

Continuing with the above example, G expresses its intent with T to register as a Client. In response to this intent, T provides G a CLIENT_ID and CLIENT_SECRET. This can be considered as a way to identify uniquely and authenticate G by T. Note that this activity is one time only.

Below are the steps when a user comes and generates meme which is to be posted on Twitter.

* Now user namely D visits G and generates a meme.

* D wants his generated meme to be posted on T. So he asks G if its possible to do so.

* G responds positively and shows D a button which says `Login with Twitter`. D clicks on that button and then is presented with a official Twitter page, where in D enters his credential (username and password).

* These credentials are then sent to Twitter along with the CLIENT_ID of G, so that T can identify "Ok user D has come and is in need of using G."

* T then receives username and password of D and authenticates D. After the authentication is successful, it responds back to the client G with the authorization code. Till this T has verified that D is genuine and this has been acknowledged to G by sharing a authorization code with him.

* Now its time to verify that G is genuine or not. So to do this now a server to server call will be done by G where in G will be sharing the authorization grant received along with its CLIENT_ID and CLIENT_SECRET.

* T now receives a new request where in the client G verifies itself. On successfully verifying the client G by T, it creates a new access token which can be used by client G on behalf of user D. This access token is then shared with G.

* Now for subsequent requests, G can use this access token whenever it needs to talk to T, on behalf of user D to do certain actions.
