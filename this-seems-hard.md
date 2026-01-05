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

<!--more-->
So, the two types of sockets are "Stream Sockets" and "Datagram Sockets", which may hereafter be 
referred to as "`SOCK_STREAM`" and "SOCK_DGRAM`" respectively. Datagram sockets are sometimes
called "connectionless sockets". 

<!--more-->
Stream sockets are reliable two-way connected communication streams. If you output two items into
the socket in the order "1,2" they will arrive in the order "1,2" at the opposite end. They will
also be error-free! [useful for my protocol!]

<!--more-->
Both telnet and ssh applications use stream sockets, and so do web browsers via HTTP which uses 
stream sockets to get pages! "Indeed, if you telnet to a website on port 80, and type 
`GET/HTTP/1.0` and hit RETURN twice, it'll dump the HTML back at you!"

<!--more-->
So, how do stream sockets conduct high level quality data transmission? Transmission Control Protocol!
TCP makes certain that data arrives sequentially and error-free. Commonly referenced in TCP/IP,
where IP stands for Internet Protocol and deals primarily with Internet routing and is not generally
responsible for data integrity.

<!--more-->
So what about Datagram sockets? why are they connectionless, why unreliable? If you send it,
it may arrive, it may arrive our of order. If it arrives, the data within the packet will be error-free.

Datagram sockets also use IP for routing but they use UDP rather than TCP.

<!--more-->
Why are they connectionless? Because you dont have to maintain an open connection as you do with
stream sockets. You just build the packet, give it an IP header with destination info, and send it.
They are generally used when a TCP stack is unavailable or when a few dropped packets here and There
don't mean anything. examples are `tftp` (trivial file transf protocol), multiplayer games, streaming
audio, video conferencing, etc. But tftp and similar programs ship binarys, so data loss cant
occur? That's why there are protocols written on top of UDP (like for tftp) that have require an 
"ACK" packet to the sender in order to complete the procedure. The ACK procedure is important when
implementing reliable `SOCK_DGRAM` applications.

<!--more-->
For unreliable applications like games, audio, or video, you just ignore the dropped packets,
or perhaps try to compensate for them in a clever way. "Quake players will know the manifestation
of this effect by the technical term: *accursed lag*. The word "accursed", in this case, represents
an extremelly profane utterance.)

<!--more-->
The core reason for using an unreliable underlying protocol is speed. If your sending 40 positional
updates per second of the players in the world, maybe it doesn't matter so much if one or two get 
dropped, and UDP is a good choice.


##### 2.2 Low level Nonsense and Network Theory
This is practically useless, but good to know for protocol layering.

<!--more-->
When a packet is born, it is wrapped (or encapsulated) in a header by the first protocol (say TFTP),
then the whole thing (TFTP header included) is encapsulated again by the next protocol (say UDP),
then again, by the next (IP), and again by the final protocol on the hardware (physical, say ethernet).

<!--more-->
When the receiving computer gets the packet, the hardware strips the ethernet header, the kernel
strips the IP and UDP headers, the TFTP program strips the TFTP header, and it finally has the data.

<!--more-->
Now we talk about the *Layered Network Model* (aka "ISO/OSI"). This model beats many others through its 
ability to write sockets programs that are exactly the same without caring *how* the data is physically
transmitted (serial, thin Ethernet, AUI, whatever) because programs on lower levels deal with it for
you. Th actual network hardware and topology is completely transparent to the socket programmer.

<!--more-->
The model:
 - Application 
 - Presentation
 - Session
 - Transport
 - Network 
 - Data Link 
 - Physical 

 "Now, this model is so general you could probably use it as an automobile repair guide if you really
wanted to. A layerd model more consistent with Unix might be:"
 - Application Layer (telnet, ftp, etc.)
 - Host-to-Host Transport Layer (TCP, UDP)
 - Internet Layer (IP and routing)
 - Network Access Layer (Ethernet, wi-fi, or whatever)

All you have to do for stream sockets is `send()` the data out. All that must be done for datagram sockets
is encapsulation of the packet (by method of your choosing) and `sendto()` it out. The kernel builds
Transport Layer and Internet Layer on for you and the hardware does the Network Access Layer.


### Chapter 3: IP Addresses. `struct`s, and Data Munging

##### I'm skipping the IPv4 stuff lol.
basically we ran out of addresses using four octets like `192.0.2.111` because we thought there 
would not come a time where billions of computers existed. There did.

##### 3.3 `struct`s (skipping a bit of stuff including byte order which will prob be important)
Finally. Programming.

The data types used by the sockets interface, some of them are tuff to understand.

First is easy, the socket descriptor, type: `int`

It gets weird from there. `struct addrinfo` is used to prep the socket address structures for 
subsequent use. It's also used in hostname lookups and service name lookups. It'll be make more
sense in actual usage later, but just remember its one of the first things you call when making
a connection.

``c
struct addrinfo {
    int             ai_flags;       // AI_PASSIVE, AI_CANONNAME, etc.
    int             ai_family;      // AF_INET, AF_INET6, AF_UNSPEC
    int             ai_socktype;    // SOCK_STREAM, SOCK_DGRAM
    int             ai_protocol;    // use 0 for "any"
    size_t          *ai_addr;       // size of ai_addr in bytes
    struct sockaddr *ai_addr;       // struct sockaddr_in or _in6
    char            *ai_canonname;  // full canonical hostname  

struct addrinfo *ai_next;       // linked list, next node
};
``
I'll use this in a bit then call `getaddrinfo()`. It'll return a pointer to anew linked list of
these structures filled out with all the "goodies" i'll need. One can force it to use IPv4 or v6 
in the ai_family field, or leave it as `AF_UNSPEC` to use whatever. This is cool because the 
code can be IP version-agnostic.

