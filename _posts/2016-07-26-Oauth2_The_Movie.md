---
layout: post
title: "Oauth2, the movie"
description:
tags: [authentication, oauth2, oauth]
categories: [authentication, oauth2]
---

Welcome! You're here because you're curious about Oauth2 and wonder what the heck it is?
You are in the right place, young dummie. Go ahead ask your question!

*** What's Oauth2 in three words??***

An authorization protocol.

***And with a few more words?***

Basically, it's a protocole that defines a way for an application to get an **authorization** to request an API in the name of a user. The authorization is materialized by a **token** which can be acquired in four different ways, depending on the server which proposes it.
There you go, you know everything, thank you for your attention!
Ok just kiddin'. Let's get started!

## Development: Oauth2 genesis

A word about Oauth2 history:
It was written to improve Oauth1 complexity, in a good way, though some would say at the expense of security... The workflow is similar, but there are less parameters and configuration to do.

***I've never cared much for History anyway,  I'm more curious 'bout how it works and how to implement it!***

Neither did I so let's get started, for real this time.

## Cast

Let's talk about the actors and their roles. Even the extras.

***What?! All at once?!!!!!***

Yeah, all at once, cause it's the best way to understand it.

Let's say you have a website which needs some information about the Google account of its users. It can be for authentication, but it might also want to access info like the Youtube channels the user is following. (here is the list of all those Google APis that are available: https://developers.google.com/apis-explorer/#p/).
Google allows you to do just that by following the Oauth2 protocole.

That's for the synopsis and the actors. Now about the official roles as defined in the protocol:

- The information you need are called the **resources**.
- Your website, the one which needs information on the user by Google, is called the **client**.
- Your user is called the **ressource owner**. Logic, it's his information!
- Google, or any other Oauth2 server, is the **resources server**. That's were are stored the information.

***Got it! Now how does it work?***

Not so fast young Padawan. As I said in introduction, the authorization is matherialized by a token. Before requesting the ressource server, you will need to get one. That's when the guest star makes its unexpected apparition!

***Who's that? o_O***

- Well it's an entity which will be called the authorization server, and which is divided in two sub entities: the **authorization endpoint**, and the **token endpoint**. 

Don't worry for now they're wearing a mask, but you'll see their faces and understand what they are exactly in the following.


## Plot (Authorization Grant Code Version)

This is one of the four ways to get the token I talked about earlier, it might be the most complex, but above all it's the most used, and the most complete and secure!

### <i class="icon-pencil"></i> Pre-production: the Client Registration
First and before everything, the client has to be registered to the authorization server. 
It's done backstage, before the user enters on the scene.
During this registration, the client will be given a **client Id** and a **client Secret** while giving the authorization server a **redirect Url**.

***What are they? Oh god, this movie is gettin' complicated. -_-***

Come on we're almost there, they are merely part of the movie set! 
The client Id and the client secret are confidential string identifiers for the client to use.
The redirect Url, you'll see quite soon.

### <i class="icon-pencil"></i> Diagram Workflow

Quickly, when the user will browse the Client and this one will need resources from the resource server, it will redirect the user to the Authorization endPoint on which the user will authenticate and grant rights to his resources to the Client. He will be redirected by the Authorization to the Client with a code which the Client will use to request (using curl for example) a token to the Token Endpoint. With this token, the Client can request the Resource Server Api. The End!

And I did it without catching my breath! But let's take a look to the diagram, u'll undestand everything:

<div class="diagram">
Ressource Owner (RO/User)->Client:browse
Note right of Client:Client needs resources
Ressource Owner (RO/User)->Authorization EndPoint:is redirected by Client (1)
Ressource Owner (RO/User)->Authorization EndPoint:authenticates and agrees the Client to access his ressources(2)
Authorization EndPoint->Client:redirects RO to Client with a code as parameter(3)
Client-->Token EndPoint:uses the code to request token(4)
Token EndPoint-->Client:returns a token(5)
 Client-->Resource Endpoint:request resources as many times as it needs, as long as the token is valid(6)
Resource Endpoint-->Client:returns resources(7)
</div>

> **(1) Client redirects the Resource Owner with the following GET Parameters:**

> - **response_type (r)** = code (for the authorization code grant it's always that)
> - **client_id (r)** = the one given to Client by the Authorization server at registration
> - **redirect_url (r)** = the redirection url entered by Client on the registration to Authorization server
> - **scope (o)** = a new part of the movie set! It is the set of rights the Client wants to be granted by the Ressource Owner. It's optional, not all Oauth2 server require some (Facebook for example doesn't)
> - **state (o)** = it's like a CSRF token that the client will generate. The authorization Endpoint won't make anything of it, it will just give it back untouched in (3)

(r) = required
(o) = optional

> **(2) Ressources Owner identifies himself on the Authorization Server and grants the rights to the Client:**

> If the RO is already loggued in on the Authorization Platform (for example, he has already opened his Gmail Inbox), he will only be asked to grant the rights
> When the RO has granted the rights once, this grant is stored and he won't be asked again.
> As a result of those two postulats, ***if the user is already loggued in and has already granted the rights (at a previous browsing on the Client for example), this step is ignored.***


> **(3) Authorization EndPoint redirects Resource Owner to Client the following GET parameters:**

> - **code (r)** = a code that the Client will use to request a token
> - **state (o)** = the state potentially previously given by the Client to the Authorization Server during (1). The client may use it to make sure the redirection is from the AS.

> **(4)(5) Client uses the code to request the token to the Token EndPoint**

Here it's not a redirection. The Client server uses a cUrl (or equivalent). The GET parameters are the following:

> - **code (r)** = the code handed to the Client by the Authorization Server
> - **grant_type (r)** = authorization_code (for the authorization code grant it's always that)
> - **client_id (r)** = the one given to Client by the Authorization server at registration
> - **client_secret (r)** = the one given to Client by the Authorization server at registration
If the parameters are recognized by the Token EndPoint, it will sent back the token, most of the time as a json (then again it's not normalized by the protocol, for exemple Facebook gives a query string, and neither is the format of the resulting array, levels and indexes).


This last step is not part of the protocole per say. The protocole stops when the token is issued. Actually not really, but we'll get into that later.

***Oh my god, another twist, I want my mummy! T_T***

Don't be a baby! Keep going!

> **(6)(7) Client uses the token to request resources to the Resource Server**

Here it's not a redirection either. The Client server uses a cUrl (or equivalent) to request the resources to the Resource Server.
Beware: the use of the token is not part of the protocole, so the Ressource Server might accept it as an Authorization header or GEt parameter with the name of its fantasy...


Ok you're all set. The End!

*** Wait no! You said there was a twist, that this didn't cover it all! ***

I know, this movie is endless...

## Rewind with the refresh token


There's something you haven't asked.

***What's your favorite actor or movie?***

No -_-.

The lifetime of tokens! Indeed, tokens are not immortal, they expires. 

***Oh... So when it happens, the client has to do it all over again?***

It depends, sometimes yes. But as I said previously, once the user has granted the client the rights to access his ressources and if he still loggued in the Oauth2 server, the redirection is completly transparent for him he will just see a fast reload of the page (depending on you client code efficiency for whatever it does of course). Then the request to the token server to server (Client to Token Endpoint) and BOOM, the client gets a new token.

***Sometimes? But other times?***

Sometimes the Token Endpoint will issue with the access token what's called a refresh token. This token  will survive the former and can be used to get a new access token directly with the Token Endpoint. You don't have to go throught the process with the Authorization Endpoint to get a code, the refresh token is the code in this exchange.

<div class="diagram">
Authorization EndPoint->Client:code to request token
Client-->Token EndPoint:uses the code and grant_type=code
Token EndPoint-->Client:returns the access token and refresh token
Note right of Client: access token expires
Client-->Token EndPoint:uses the refresh token and grant_type=refresh_token
Token EndPoint-->Client:returns the access token and refresh token
</div>

## Alternative ends: the other Oauth2 workflows

The basis of Oauth2 is the token to request an API. We've just seen the most common worflow, the Authorization Code Grant. Three others types exist:

**The Implicit Grant workflow:**
Basically it looks like this:
**response_type** = token
<div class="diagram">
Ressource Owner (RO/User)->Client:browse
Note right of Client:Client needs resources
Ressource Owner (RO/User)->Authorization EndPoint:is redirected by Client with response_type=token
Ressource Owner (RO/User)->Authorization EndPoint:authenticates and agrees the Client to access his ressources
Authorization EndPoint->Client:redirects RO to Client with the token as parameter(3)
 Client-->Resource Endpoint:request resources as many times as it needs, as long as the token is valid
Resource Endpoint-->Client:returns resources
</div>

Here the client is a Javascript one, meaning that the request to Resource Endpoint are done on the user side, with an Access-Control-Allow-Origin header. The Client gets the access token in one call, but it transists throught client side which makes it the least secure.

**The RO Credentials Grant workflow:**
Here it can be confusing. Basically, the Client will ask the Resource Owner for his credential on the Oauth2 server and send them directly to the Authorization Server which will return the token. 
Obviously, there has to be complete trust between Client and Authorization Server, they have to be part of the same "entity" since the former will have the credential of the RO on the latter. For example we can imagine that the login page domain of Google is the only one to be able to do this with the Google Authorization server.
**response_type** = password
<div class="diagram">
Ressource Owner (RO/User)->Client:authenticate with login and password
Client->Authorization Server:requests Access token with login and password
Authorization Server->Client:returns access token, refresh token(o)
 Client-->Resource Endpoint:request resources as many times as it needs, as long as the token is valid
Resource Endpoint-->Client:returns resources
</div>

**The Client Credentials Grant workflow:**
When the client is the ressource Owner... Its credentials are the client_id and client_secret and it will send them directly to get directly an access token.
**response_type** = client_credentials
<div class="diagram">
Client->Authorization Server:requests Access token with its client_id and client_secret
Authorization Server->Client:returns access token, refresh token(o)
 Client-->Resource Endpoint:request resources as many times as it needs, as long as the token is valid
Resource Endpoint-->Client:returns resources
</div>

## Trailer: example of code in PHP

Here is your page **oauth2.php** which is accessed via the url **OAUTH2_CLIENT_URL**.
It will be the page which will first redirect the user, then be the redirect url which will require the token and store it.
**OAUTH2_AUTHORIZATION_URL** is the Authorization Endpoint Url.
**TOKEN_ENDPOINT_URL** is the Token Endpoint Url.
Let's imagine you have a class Oauth2TokenManager which has the following method:
 - getAccessToken(): check that it has a token stored in session, return it if so, false otherwise.
 - requestAccessToken(\$code,\$tokenEndPoint): makes the curl to the TokenEndPoint to get the token with the code  and returns the returned access Token.
 - storeAccessToken($accessToken): stores the access Token in Session

**oauth2.php**
{% highlight php %}
 <?php

$tokenManager = new Oauth2TokenManager();
$token = $tokenManager->getAccessToken();
/* if no token, either we haven't requested the code to the Authorization Endpoint Yet, or we have just been redirected from it (it is a redirection from AE and there is an error, drop it or try again by redirect to self with no $_GET parameters)*/
if (!$token &&!isset($_GET['error']))
{
    /* if not error in $_GET, if it's not a redirection from AE , there is no code in $_GET  so let's redirect now */
    if (!isset($_GET['code']))
    {
        $authorizationEndPointRedirect=**OAUTH2_AUTHORIZATION_URL**;
        $parameters = 'response_type=code
        &client_id='.**CLIENT_ID**.
        '&redirect_uri='.**OAUTH2_CLIENT_URL**;
        // redirect to AE with the right parameters which will redirect back to here 
        header("location:".$authorizationEndPointRedirect."?".$parameters);
        exit;
    }/* otherwise it's a redirection from the AE with the code!*/
    else
    {
    // makes the curl to Token Endpoint with the code given as $_GET
    $access_token = $tokenManager->requestAccessToken($_GET['code'],**TOKEN_ENDPOINT_URL**);
    // store the token
     $tokenManager->storeAccessToken($access_token);
    }
    // from now there should be a token 
    $token = $tokenManager->getAccessToken();
}
else if (isset($_GET['error']))
{
//Treat by redirecting again without $_GET or do whatever you want
}

// At this point $token exists and is ready to request the API, enjoy!!!
{% endhighlight %}

## A word about security
Quick word about security: there are some security issues with Oauth2 which you are advised to look into before implementing your Oauth2 Client / Server. Google is your friend to look into this as I am too tired (and not actually adequate) to talk about it.

----------
Here you go, hope you have a better understanding of Oauth2 after that.

## The End
