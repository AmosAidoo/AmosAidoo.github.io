---
title: A Look at HTTP
excerpt: HTTP is an application layer protocol. On a high level, it specifies how a program, known as a client requests for documents such HTML documents, images, videos, e.t.c. from another program, known as a server.
tags: http networking
---
### Introduction

Bro 1: "Yo bro, charley wosop"<br/>
Bro 2: "Charley adey o, your side?"<br/>
Bro 1: "Everything dey cool"<br/>
Bro 2: "Vhim. Today what be the vibe?"<br/>
Bro 1: "Bro adey want learn coding o. What make I do?"<br/>
...

What we just witnessed above is a conversation between two bros (in Ghanaian Pidgin English). This conversation began with the first bro checking up on the second bro and the second bro responded on a good note. He in turn did the same thing which bro 1 also responded to on a good note. From there, bro 1 made an enquiry (and the conversation continues...). We can call this a protocol because at least, this happens almost everytime two guys meet to have a conversation. Now let's move our attention from the two bros to computer networking. In computer networking also, there are protocols that two entities use when communicating with each other. 

### What is a protocol?
A protocol can be defined as a set of rules defining how data is transmitted between entities within a network. These rules include the order in which data should be send, the format of that data and the actions that should be taken upon the transmission or recepient of the data. There are a number of protocols in networking. Examples are SMTP, UDP, ICMP, HTTP, e.t.c. There are already so many stories that have been told about HTTP but it doesn't hurt for me to retell the story.

### What then is HTTP?

HTTP stands for Hypertext Transfer Protocol. It is an application layer protocol. On a high level, it specifies how a program, known as a client requests for documents such HTML documents, images, videos, e.t.c. from another program, known as a server. In operating system terms, a process is a program in execution. So more accurately, a client process requests for documents from a server process. The internet uses a set of protocols known as the TCP/IP stack. These set of protocols have been grouped into layers(a stack) and the topmost layer, the application layer, is were HTTP resides.

<p align="center">
	<img src="/assets/images/rect.png" alt="TCP/IP Stack"/>
	<p align="center">The TCP/IP Stack</p>
</p>

### TCP/IP

Network applications, together with application layer protocols reside in the application layer. For these applications to exchange information(known as messages) with each other over a network, they use the protocols available to them. The other layers in the TCP/IP stack then assist in getting messages from one system to another. Typically, a process in the application layer sends a message to the Transport Layer through a software interface called a **socket**. The Transport Layer then adds some details to this message and passes the result to the Network Layer. The Network Layer also does the same thing. It adds some more details to the information it received from the Transport Layer and passes the results to the Link Layer. The Link Layer also adds its own and passes the results to the Physical Layer which does the actual transmission of bits through a transmission medium. Here the details each layer adds to the information it receives are known as **headers**. There is a beautiful analogy in [Computer Networking: A Top-Down Approach](https://www.amazon.com/Computer-Networking-Top-Down-Approach-7th/dp/0133594149) to help with thinking about a process and a socket: *"A process is analogous to a house and its socket is analogous to its door.
When a process wants to send a message to another process on another host, it shoves
the message out its door (socket). This sending process assumes that there is a trans-
portation infrastructure on the other side of its door that will transport the message to
the door of the destination process. Once the message arrives at the destination host,
the message passes through the receiving process’s door (socket), and the receiving
process then acts on the message."*

### The Transport Layer
Now let's consider briefly the Transport Layer. This layer contains two protocols, Transmission Control Protocol(TCP) and User Datagram Protocol(UDP). TCP is a connection-oriented protocol since it establishes a connection between the communicating processes whilst UDP is connectionless. HTTP relies on the services of TCP in transmitting messages from one process to another. Therefore, for two communicating process to exchange messages using the HTTP protocol, a TCP connection would first have to be established (this is what is know as the [Three-way Handshake](https://en.wikipedia.org/wiki/Handshaking#TCP_three-way_handshake), a discussion for another day) before the exchange takes place.


### HTTP Response Messages
There are two forms of HTTP messages that are transmitted when a client and a server communicate, which are:
1. HTTP Request Message
2. HTTP Response Message

#### HTTP Request Message
The image below shows the structure of an HTTP request message: 
<p>
	<img src="/assets/images/request_msg.png" alt="TCP/IP Stack"/>
	<!-- <p>HTTP Request Message Structure</p> -->
</p>

The first line is know as the **Request-Line**. It contains the method, url and version of the protocol. These fields are separated by a space.
- The method defines the operation that a client wants to perform. Examples are GET, POST, e.t.c.
- The url specifies a path to a resource a client is interested in.
- The version specifies the specific version of HTTP.


The second line is made up of headers. Headers allow the client to pass additional information about the request, and the client itself, to the server. A header is made up of a header field and a value like so, `Header-Field: Value`.

The third line contains the body(when necessary). The body is some data that the client would like to send to the server. The snippet below is an extremely simple example of an HTTP request. The method is GET. The GET method is used to retrieve some resource from a server. The url is `/api/path-to-resource` and the protocol version is HTTP/1.1. The headers in this request are `Host` and `Content-Type`. `Host` identifies the client making the request and `Content-Type` specifies the type of resource the client expects from the server. In this case, the client is expecting a plain text. There are many more headers which can be found [here](https://www.iana.org/assignments/message-headers/message-headers.xhtml).
```
GET /api/path-to-resource HTTP/1.1
Host: 127.0.0.1
Content-Type: text/plain
```

#### HTTP Response Message
The image below shows the structure of an HTTP response message: 
<p>
	<img src="/assets/images/response_msg.png" alt="TCP/IP Stack"/>
	<!-- <p>HTTP Request Message Structure</p> -->
</p>

The first line is known as the **Status-Line**. It contains the version, status code and phrase.
- The version is the HTTP version.
- The status code is a 3-digit number that describes what happened when the server tried to serve the request.
- The phrase gives a short description of the status code.


The second line is made up of headers just like in the request message. These headers allows the server to pass additional information about the response which cannot be placed in the Status-Line. They give information about the server and about further access to the resource identified by the Request-URI. [Section 6.2 in [RFC 2616](https://tools.ietf.org/html/rfc2616)]

The third line contains the body(when necessary). The body contains data that the server might want to send in response to a request the client makes. The snppet below is an extremely simple example of a response message. The protocol version is HTTP/1.1. The status code is 200 which means the request was served successfully. The phrase "OK" verifies that everything is OK. The `Content-Length` header shows the length of the body in bytes and as the `Content-Type` says correctly, the response body is made up of plain text.

Status codes are broken into the following five blocks:
1. 1xx Informational
2. 2xx Success
3. 3xx Redirection
4. 4xx Client Error
5. 5xx Server Error

The “xx” refers to different numbers between 00 and 99.

So the next time you get a 404 Not Found, you should know that the error is from your end :).


```
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 20

A beautiful response
```

So that's it. This is a brief explanation of how HTTP works. A very detailed explanation of the inner workings of the protocol can be found in [RFC 2616](https://tools.ietf.org/html/rfc2616). I hope you enjoyed it and learnt something new. You can connect with me and let me know of what you think. See you in another post :).

### References
1. [https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
2. [RFC 2616](https://tools.ietf.org/html/rfc2616)
3. [Computer Networking: A Top-Down Approach](https://www.amazon.com/Computer-Networking-Top-Down-Approach-7th/dp/0133594149)
4. [https://cloudflare.com/en-gb/learning/ddos/glossary/hypertext-transfer-protocol-http/](https://cloudflare.com/en-gb/learning/ddos/glossary/hypertext-transfer-protocol-http/)