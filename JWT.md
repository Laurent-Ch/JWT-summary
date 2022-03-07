# Using JWTs to identify users
*NB* This little document is a summary of these 2 videos provided within the lesson on Rails and JSON (next, week 9, day 42): 

-   [What is JWT authorization really about](https://www.youtube.com/watch?v=soGRyl9ztjI)
-   [What is the structure of a JWT](https://www.youtube.com/watch?v=_XbXkVdoG_0)

## 1. A reminder on the HTTP protocol
HTTP is a *stateless protocol*, and two of its characteristics that matter to our topic are the following: 

1. each request (i.e. interaction) **requires all the necessary information**.
2. **no state is maintained** during multiple requests.

This isn't problem with static pages, but with dynamic ones, such as with user-based content for example, you need to identify yourself continuously. 

## 2. Two main ways used to identify users 
#### 2.1 Session tokens 
Here, a server creates a **ticket** to keep track of a user (session ID). It's the most popular approach, and it's mostly done via **cookies**.  It has an issue though: it assumes a *simple 'one-server'* web application. Most modern apps run on *multiple servers* managed by a *load balancer* which takes requests and dispatch them. In that case, one server might keep track of the user while others don't. To deal with that issue: 

a. you can set up a **shared session cache** that all servers consult, *BUT* it reintroduces the issue of a single weak link within the architecture. 

b. you can set up a **sticky session pattern** in which the load balancer remembers both **who** the user is and **which** server it's been connected to, so that it can always redirect them to that server all the time, *BUT* it's not scalable and with micro-services, several servers collaborate with each other, which makes this solution useless. 

#### 2.2. Jason Web Tokens
Change of paradigm: rather than giving all the necessary information to the server, why not let the users carry it everywhere? 

In the previous part, users were given a cookie, that is to say a token ID that **links to** all the details/info server-side. Now, they'll get a JWT, a **value token** that **contains** all the information. 

The issue with this new solution is *trust*. To deal with it, a system based on signatures is used. A JWT is based on 3 main parts separated by dots:

1. a header
2. a payload
3. a signature

They're all encrypted, but it's important to understand that base64 encoding is used for practicality and not for protection: anyone can decrypt it. Therefore, **no sensitive/confidential information** should ever be put in a JWT.

[jwt.io](https://jwt.io/) is a useful website that let you play with JWTs and see how they work. 

The *payload* contains the relevant information. `iat` means issued at. 

The *header* and *signature* both deal with authenticity. The header says **how** it was signed (type as well as algorithm used) and the signature is created by combining the two public parts —the header and the payload— and applying a **secret key (stored server-side)** on them when the JWT is created. This same key will later be used by the server to check the validity of the JWT.

Just like the session tokens seen in part 2.1, JWTs have flaws: 

- As we've already seen above, the value/payload is out in the open: JWTs are not made to deal with private data.
- Since the signature is linked to the content, it cannot be tempered with, which is great. However, it's not linked to any specific user, which means that it can be stolen/used by anyone. Other protocols, such as https, OAuth etc. have to be put in place to make it more secure.
- Disabling it (end of session, stolen etc.) isn't as simple as with cookies. There are two main options:  
	- you can put an **expiration date** on it when you create it in the first place. 
	- you can **blacklist** it at any given moment. In that case, a JWT is a *single-use* token: create it, use it, throw it away when you're done (for example when you log off), and repeat as much as needed.

A JWT is **NOT** an authentication mechanism, it comes into play when the authentication is completed. It can be saved within the localstorage or in a cookie; it doesn't matter. It's passed on every request via the http header. 
