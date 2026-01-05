# TCP Project 1: Spec LDP/1.0 -- Length-Delimited Protocol

### Purpose
I feel that having a solid understanding of network programming it necessary for anyone who wants
to be a solid engineer. I will be insecure until I write a few, and it will be particularly useful
for me as a cloud infrastructure engineer. 

Secondarily, I believe that understanding MCP (Model Context Protocol) holistically, and 
(very importantly) being able to implement MCP's will be the difference between Matthew the 
infra junior at USAA and Matthew the cloud guru mid level eng at google. I genuinely believe 
that building these protocols from scratch will (one day) be what seperates me from the pack,
and what makes me the engineer I want to be. This is the ground work to contributing to cutting
edge FOSS and that is the difference maker between people with the title "swe" and hardcore
engineers. Also it'll make me really fucking rich.

### Barebones Description (what I know before reading documentation)
TCP stands for Transmission Control Protocol (looked it up lol). My base knowledge of protocols
is that they are programs that define how programs communicate. I assume that "Transmission Control" 
dictates that this flavor tells two programs some rules about communicating with eachother. I think
thats what all of them do to an extent, but UDP is like unary datagram protocol (didnt look it up)
and that refers to packet transmission I believe, and http -- hyper textual transfer protocol -- is
probably for ascii or something (I'll figure it out later when I force myself to write them).

### Real Notes (per [Beej's](https://beej.us/guide/bgnet/)
#### Chapter 2: What's a Socket
When people say "everything in Unix is a file" they mean that when Unix programs do I/O of any 
sort, they do it by reading or writing to a file descriptor. A File Descriptor is simply an 
integer associated with an open file.

That file can be anything though:
- network connection
- FIFO
- pipe
- terminal
- real on-the-disk file
- or just about anything else

<!--more-->
So when you want to communicated with another program over the Internet, you'd better believe
that it's through a file descriptor.

<!--more-->
If you want to get a file descriptor, you have to make a call to the `socket()` system routine. 
it returns the socket descriptor, and you communicate through it using the specialized `send()`
and `recv()` (man send, man recv) socket calls.

<!--more-->
One with more C knowledge than I would ask "why can't I just use normal `read()` and `weite()` 
calls to communicate throught the socket?" and you can, but `send()` and `recv()` offer much
greater control ovre data transmission.

<!--more-->
There are all kinds of addresses:
- DARPA Internet addresses (internet sockets)
- CCITT X.25 addresses (X.25 sockets that you can safely ignore)
- and probably many others depending on Unix flavor

<!--more-->
Beej's only deals with Internet Sockets!

----
##### 2.1 Two Types of Internet Sockets
There are more than two types of Internet Sockets, but this will focus on only two (though it mentions
"Raw Sockets" which are apparently very powerful and worth a lookup)

So, the two types of sockets are "Stream Sockets" and "Datagram Sockets", which may hereafter be 
referred to as "`SOCK_STREAM`" and "SOCK_DGRAM`" respectively. Datagram sockets are sometimes
called "connectionless sockets". 

Stream sockets are reliable two-way connected communication streams. If you output two items into
the socket in the order "1,2" they will arrive in the order "1,2" at the opposite end. They will
also be error-free! [useful for my protocol!]

Both telnet and ssh applications use stream sockets, and so do web browsers via HTTP which uses 
stream sockets to get pages! "Indeed, if you telnet to a website on port 80, and type 
`GET/HTTP/1.0` and hit RETURN twice, it'll dump the HTML back at you!"

So, how do stream sockets conduct high level quality data transmission? Transmission Control Protocol!
TCP makes certain that data arrives sequentially and error-free. Commonly referenced in TCP/IP,
where IP stands for Internet Protocol and deals primarily with Internet routing and is not generally
responsible for data integrity.

So what about Datagram sockets? why are they connectionless, why unreliable? If you send it,
it may arrive, it may arrive our of order. If it arrives, the data within the packet will be error-free.

Datagram sockets also use IP for routing but they use UDP rather than TCP.

Why are they connectionless? Because you dont have to maintain an open connection as you do with
stream sockets. You just build the packet, give it an IP header with destination info, and send it.
They are generally used when a TCP stack is unavailable or when a few dropped packets here and There
don't mean anything. examples are `tftp` (trivial file transf protocol), multiplayer games, streaming
audio, video conferencing, etc. (stop page 7: 190% zoom on lower screen)
