---
layout: post
title: "Firsts steps in SAML 2.0"
description:
tags: [saml, saml2, authentication, sso]
categories: [authentication, saml2]
---

# Firsts steps in SAML 2.0

Most of you have heard about SSO and maybe SAML. But it's always an obscure part of authentication system in people's minds. Here is some basic explanation about SAML, I will not enter in details about security and all the data sent from a platform to another. The aim is not to be too much technical to understand the connection process.
The first thing to know is what a SSO is, and what is the purpose.

### 1 - What is a SSO?

SSO is the name of an authentication process. An SSO (Single Sign-On) allows a person to connect to multiple applications without entering his password each time. One access control is enough. The opposite process is SLO (Single LogOut).

![SSO example]({{ site.url }}/images/posts/20161111_saml2/ssoBascis.png)

Benefits are:
* One password to remember.
* Less prompts, so less time spent typing his password.
* Help desk costs lowered.

Problems are:
* Security must be enforced.
* If the authentication server is down, all these applications are not accessible anymore.
* Need a good identity data governance.

Here is an example of a Facebook SSO in the Spotify login page.

![spotify sign in with facebook]({{ site.url }}/images/posts/20161111_saml2/signupFbSpotify.png)

The behaviour here is, if you have an account on Facebook and you are already logged in, once you click on the "Sign up with Facebook" button, Facebook will give some of your data to Spotify to create an account (if you don't have one yet) or will retrieve your existing account. If you are not already logged into Facebook, the website will ask you for your Facebook credentials before accessing the music platform.

### 2 - How SAML works

#### a- Overview 
 
SAML stands for **Security Assertion Markup Language** and the most common version is SAML 2.0, the latest version. This protocol is based on XML exchange and enables web-based authentication and authorization scenarios. The exchange is made between an Identity Provider (**IDP**) and a Service Provider (**SP**). The role of the IDP is to confirm if the user can loggin or not. The SP sends a request to the IDP to know if the user can log in and the IDP grants access if the login is validated. In the example above, Facebook stands for the IDP and Spotify the SP. The internet browser is the bridge between the IDP and the SP because they both communicate with HTTP requests made in GET, POST or Artifact.

#### b- Technical details 
 
__Setup__

A SAML connection has to be configured : the IDP must store the **connection urls and the security key** of the SP, and vice-versa. These informations are referred to as the IDP and SP **metadata**, they ensure a secure transaction between the IDP and a SP. The **security key** is a public key that will be used to encrypt the message. To simplify and understand what encryption is, in the movie "the imitation game" Alan Turing and Christopher Morcom were chatting in class with encrypted messages written on papers that no one could read without knowing the encryption code, to avoid being caught by the teacher. Once the public keys are shared, all communications will be encrypted with the keys.

Once both sides are configured the authentication is possible.

__The connection sequence__

The scenario is: the user wants to connect to his eLearning platform through his company intranet. Both are SP. The IDP is just a server that contains all the company accounts.

__What the user sees__

 1 -The user access his intranet, but he is not connected, he clicks on the "Connect to intranet button".

![authentication step1]({{ site.url }}/images/posts/20161111_saml2/step1.png)

2 - He is redirected to the company authentication server (IDP), where he is not connected. He fills the credentials and clicks on login. He is now connected to this server and get redirected to the intranet site.

![authentication step2]({{ site.url }}/images/posts/20161111_saml2/step2.png)

3 - Now the user is on his company intranet with his account. He can access all the content of the intranet and he wants to do his eLearning content. He clicks on the button to access the eLearning platform.

![authentication step3]({{ site.url }}/images/posts/20161111_saml2/step3.png)

4 - The user is redirect to the eLearning platform with the same account has the one on the intranet.

![authentication step4]({{ site.url }}/images/posts/20161111_saml2/step4.png)

__What really happens__

 - _The SSO request_: When the user clicks on the login button of the intranet, the intranet will send an **AuthnRequest** to the company authentication server (IDP). An AuthnRequest is a request sent from the SP to the IDP to ask if the user can connect (image 1).
- _The SSO authorization_: The IDP will analyse the AuthnRequest (does he know the intranet, are the certificates correct, etc). If everything is correct, the IDP will check if the user is already logged in (with cookies for example). In our scenario it's the first connection of the user to a company platform so he is not connected for now. Therefor IDP displays a login/password page (image 2). Once the credentials are validated, the employee is connected on the IDP and this one will send a **Response** to the SP. The Response is a message from the IDP to the SP that contains the user data (ex lastname, firstname, employee id, etc) and a status (access granted or not). Once the access is granted he will be redirected to the intranet homepage (image 3).
- _The SSO in action_: The user clicks on a link in his intranet that redirects him on the company eLearning platform. During the page loading, the eLearning platform will send to the IDP an AuthnRequest. The IDP then replies with a Response that grants access to the eLearning platform. The employee is now connected on the eLearning platform with his account (image 3). He will never see the eLearning platform login page because he is already connected on the IDP.

Messages between SP and IDP, such as the AuthnRequest and the Response, are called **Assertions**.

### 3 - Why use SAML over other protocols?

SAML 2.0 is mostly used in B2B because of the Active Directory Federation Services (**ADFS**) that is a component of Windows Server. This allow to plug the company 's Active Directory (**AD**) with all the applications you want. Lots of companies use the AD to manage employees account. If an employee is not present in the AD, he will not be able to connect any company application. The IT team have to manage only one database for all employees. Futhermore SAML 2.0 and ADFS are easy to install and configure, SP and IDP communicate through HTTP requests, no need to open ports in firewalls.

In this example with Facebook and Spotify, the protocol used is OAuth (protocol used by Facebook, Gmail, Yammer to communicate with other applications), not SAML. OAuth is a most recent protocol that's why this protocol is not very common in companies and mostly used in B2C businesses.

### 4 - TL;DR

SAML is just the way authentication and authorization are transmitted between an IDP and an SP. The SP asks for an authorization (AuthnRequest), the IDP checks if the request is valid, checks if the user exists and is authorized to access the SP application and sends its response (Response). Those messages are called Assertions. If the access is granted, the user will be connected on the SP with his account, otherwise the connection will be denied. The company manages only one database with authorized accounts. SAML communication transits with HTTP requests that allows to deploy SAML without modifying the company firewall.