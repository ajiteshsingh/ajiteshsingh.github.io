---
layout: post
title: "Proxy Vs Reverse Proxy"
date: 2020-02-25 12:00:00 -0700
categories: [networking, web]
tags: [proxy, reverse-proxy, nginx, networking, vpn, load-balancing]
---

We will start off by first understanding proxy and reverse proxy servers and later we will also discuss how we can store the location of a user from request headers via nginx reverse proxy server without browser location permission.

**What is a (forward) proxy server?**

Whenever someone is talking about a proxy server, the person is referring to a forward **proxy**. A “proxy” simply means someone acting on behalf of someone else. So a proxy server is a server acting on behalf of another server and it hides the identity (i.e. IP address) of the client from the server and fulfills the request on behalf of him. So in a nutshell Proxy servers act as a middleman between the client and the server and also hides client IP from the server.

![]({{ "/assets/proxy-vs-reverse-proxy/image2.jpg" | relative_url }})

For eg. let’s say there are three computers connected to the internet

**A** - Client Computer

**B** - The proxy server

**C** - The server you actually want to retrieve data from

Normally one would connect directly from **A → B,** but in some scenarios it is better to connect **B → C** on behalf of **A**, which chains the request as **A → B → C**.

So when C receives the request, the request header has the Ip address of **B (proxy server)** and not **A (Client)**. Hence the IP address of **A** is hidden from **C** which protects the identity of **A**.

Note: There are some types of forward proxies that might still make the source A IP address retrievable.

**What is the purpose of having a proxy server?**

- When **A** is not able to access **C** directly. There can be two possibilities when **A** is not able to directly access **C**.
  - System Administrator of **A** has blocked access to **C**. For eg. **A** works in a company where access to facebook.com is not allowed
  - The administrator of **C** has blocked **A**. For eg. The administrator of **C** has noticed some hacking attempts from **A**.

There are many forward proxy softwares, for eg. glype, cgi-proxy, PHP-proxy etc.

So if you have used VPNs, this is how VPNs work in a simplified way which helps you access the websites which might be restricted (GeoFenced) in your country or your region.

**What is a reverse proxy?**

While in forward proxy the server acts on behalf of the client in an interaction between a Client and a server, in reverse proxy the reverse proxy server acts on behalf of the server in an interaction between a client and a server. Reverse proxy is slightly tricky as it might look just like a proxy server at first glance.

Let’s understand the difference with an example. There are three computers connected to the internet

![]({{ "/assets/proxy-vs-reverse-proxy/image3.jpg" | relative_url }})

**A** - Client Computer

**B** - Reverse Proxy Server

**C** - the server you actually want to hit

Ideally if you are Client **A**, you would want to directly hit server **C**. However in some scenarios, it is better for the administrator of server **C** to restrict direct access to itself and force any request to go through **B** first.

Just take a step back here and think how this is different from a proxy server. I hope we are on the same page until now. So even though the chain of request execution (**A → B → C**) is same in both forward proxy and reverse proxy server, In proxy server the client A was aware of the fact that it is hitting a proxy server which would act on behalf of him to get the data. But in reverse proxy the client has no idea that there is a reverse proxy server configured. For the client he thinks that he is hitting directly to the destination server, whereas the destination server is kind of forcing the request to go through a reverse proxy server first before reaching it. So in a nutshell, Client **A** sees he is communicating with **B**. The server **C** is invisible to client **A**. Even though Client **A** thinks he is communicating with **B**, all in reality **B** is forwarding all its requests to **C**.

**What is the use of a reverse proxy?**

- As a Load balancer. For eg. Instagram has millions of active users at a time, one single server cannot handle all the traffic. So Instagram must have configured a reverse proxy server, which will redirect the request to the server which is least busy.
- For security reasons. For eg. if there is some sensitive data which is being served and the administrator does not want the request to come directly to it. The administrator can configure some security mechanisms on the reverse proxy servers which blocks any suspicious requests.

There are many reverse proxy softwares, for eg. nginx, Apache mod_proxy, HA-proxy etc.

That’s all we need to know about forward and reverse proxy servers.

**Let’s see how we can store the location of any user requesting your backend service without asking for the location permission from the browser?**

It is really simple actually. We will take an example of nginx server to illustrate this.

With default nginx configuration, if you try to read the IP address of the client requesting, you would end up getting the IP of the reverse proxy server. But there is a way you can forward the client IP address from proxy server to your actual server.

In Your nginx conf file just add X-Forwarded-For in your headers before redirecting the request via nginx.

So your /etc/nginx/conf.d/example.com.conf file would look like this.

![]({{ "/assets/proxy-vs-reverse-proxy/image1.png" | relative_url }})

On your real server which actually processes the request you can read X-Forwarded-For from headers which will be a string of IPs separated by commas. The first IP address is the client’s IP address.

So what do you do with this IP? We wanted the location of the user and not the IP. Yeah you must have already guessed it. It’s not that difficult to get location from IP address. There are many services out there. [https://ip-api.com/](https://ip-api.com/) is a good service. So that’s how you get the location of a user even if you have configured reverse proxy. Thanks for reading :P
