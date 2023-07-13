+++
author = "psykomal"
title = "It's queues all the way down"
date = "2023-07-13"
description = ""
tags = [
    "blog", "12-days", "socket", "networking", "queue"
]
+++



This one is going to be short. Recently I wrote about NGINX socket multiplexing. And there was one question I was curious about which was left unanswered - "How are new connections buffered". What happens at the kernel level? I did some digging. So let's start...


## TCP Sockets


{{< figure
		  src="queue-imgs/figure_4.1.png"
		  caption="Unix Network Programming Fig 4.1"
>}}

- A socket is a file-like abstraction/interface provided by the OS to perform network I/O operations
- Timeline:
	- The server **creates** a socket and **binds** it to a (IP address, PORT)
	- It then calls **listen** on the socket
	- The client then calls **connect** to connect the server
	- A TCP handshake is performed 
	- The server then calls **accept**, which creates a new connection (a different PORT in the server) to handle this client
	- The client and server can freely communicate using the sockets until one of them closes the connection or some network issue occurs

```python
# A simple TCP server in python that reads headers and data
import socket
import json

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

s.bind(('0.0.0.0', 7777))

s.listen()
print("Listening for connections...")

while True:
    conn, addr = s.accept()
    print(f"New connection at {addr}")
    
    req = conn.recv(4096)
    
    headers, body = req.split(b'\r\n\r\n')

    data = {}
    for header in headers.split(b'\r\n')[1:]:
        k, v = header.split(b': ')
        data[k.decode('utf-8')] = v.decode('utf-8')
    
    conn.send(b'HTTP/1.1 200 OK\r\n\r\n')
    conn.send(json.dumps(data, indent=4).encode('utf-8'))
    
    conn.close()
```


### What happens when there are more clients coming at once


We need to take a closer look at **listen** and **accept** calls to understand what's going on here. 

## listen(2)

{{< figure
		  src="queue-imgs/man-listen.png"
		  caption="man listen"
>}}

Well there you go, the kernel does maintain a queue and its limit can be set with the backlog parameter.

- A socket when initially created is a default state to **connect()**. Assumed to be an active socket
- Calling listen changes it from a `CLOSED` state (unconnected socket) to `LISTEN` state (passive socket)


### Connection Queues

The kernel maintains 2 queues:
1. An **incomplete connection queue**, which contains an entry for each SYN that has arrived from a client for which the server is awaiting completion of the TCP three-way handshake. These sockets are in the `SYN_RCVD` state 
2. A **completed connection queue**, which contains an entry for each client with whom the TCP three-way handshake has been completed. These sockets are in the `ESTABLISHED` state 

{{< figure
		  src="queue-imgs/figure_4.7.png"
		  caption="Unix Network Programming Fig 4.7"
>}}


- The kernel takes care of the connection creation. The server process is not involved


{{< figure
		  src="queue-imgs/figure_4.8.png"
		  caption="Unix Network Programming Fig 4.8"
>}}

Steps:
- The diagram is self-explanatory for the normal flow
- In case the entry times out, it is removed from the incomplete queue
- When the process calls accept, the first entry from the completed queue is returned to the process. 
	- If the queue is empty, it receives EAGAIN and is put to sleep if it is blocking accept

### *backlog* argument

- The arg is the sum of both the queues historically. But varies implementation wise and is different in different OS.
- Its multiplied by 1.5 internally as a fudge factor in Berkeley-derived implementations
- From linux 2.2, backlog arg is the queue length for completely established sockets waiting to be accepted, instead of the number of incomplete connection requests. If the backlog argument is greater than `/proc/sys/net/core/somaxconn`, it is silently truncated to that value. The maximum length of the queue for incomplete sockets can be set using `/proc/sys/net/ipv4/tcp_max_syn_backlog`.
- **Fixed number of connections**. Historically the reason for queuing a fixed number of connections is to handle the case of the server process being busy between successive calls to accept. This implies that of the two queues, the completed queue should normally have more entries than the incomplete queue. Again, busy Web servers have shown that this is false. The reason for specifying a large backlog is that the incomplete connection queue can grow as client SYNs arrive, waiting for the completion of the three-way handshake.
- **No RST sent if queues are full**. If the queues are full when a client SYN arrives, TCP ignores the arriving SYN; it does not send an RST. This is because the condition is considered temporary, and the client TCP will retransmit its SYN, hopefully finding room on the queue in the near future. If the server TCP immediately responded with an RST, the client's `connect` would return an error, forcing the application to handle this condition instead of letting TCP's normal retransmission take over. Also, the client could not differentiate between an RST in response to a SYN meaning "there is no server at this port" versus "there is a server at this port but its queues are full."
- **Data queued in the socket's receive buffer**. Data that arrives after the three-way handshake completes, but before the server calls `accept`, should be queued by the server TCP, up to the size of the connected socket's receive buffer. 


### SYN Flooding

A famous attack is the SYN flood where the idea is to fill the incomplete queue with bogus SYNs so that legitimate SYNs are not queued providing a denial of service to the legitimate clients


## Closing Notes


- A lot of the information in this blog is from the amazing **UNIX Network Programming book** and [Shichao's Notes](https://notes.shichao.io/unp/ch4/)
- For even more in-depth info regarding this, check [TCP Socket Listen: A Tale of Two Queues](http://arthurchiao.art/blog/tcp-listen-a-tale-of-two-queues/#6-a-tale-of-two-queues)
- Another interesting thing here is how **accept** scales which leads to the infamous thundering herd problem. This is a topic for another blog (next one maybe!)
- Ending on a philosophical note

> Life is queues all the way down. A store is a queue with one side closed. Things flow from one end to another and stored in a place for a while. Money flows, Information flows, Matter flows, Time flows, and so do we...