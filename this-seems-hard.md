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
#### What's a Socket
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
