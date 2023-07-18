+++
author = "psykomal"
title = "Load Balancing inside the Load Balancer, nginx"
date = "2023-07-11"
description = ""
tags = [
    "12-days", "web-server", "nginx"
]
+++


## Intro

***Note**: This article is limited to the internals of nginx. Other servers might have different solutions to the problems stated in this article and I plan to revise with new info as I learn and explore along the way.*


Well, load balancing might be a poorly used term to use here and this is not about high-level system design of load balancing a load balancer/web server. I am talking about something much sneakier that goes on deeper down the stack, under the hood in nginx. I was always curious about what happens at a socket level in web servers. How are so many requests handled? How are the requests buffered? When do requests start getting dropped? 

To answer all these questions, we first need to understand a little bit about the internals of the web server. I will be using nginx as a reference. Everything from here is in relation to nginx and other technologies (Envoy, Pingora (Cloudflare's internal tool which replaced nginx)) might have different solutions to the problems stated above.


## NGINX

Nginx is a popular open-source web server. At its core, nginx uses an asynchronous event-driven non-blocking architecture to handle requests. It uses I/O multiplexing and event notifications so that one worker process can juggle between multiple connections rather than the previous 1 thread/process per connection (initially used by Apache). This allows it to scale to handle many concurrent connections with a low memory footprint.

A little history: nginx was created to tackle the C10K problem of achieving 10,000 simultaneous connections for a web server. It is suitable for nonlinear scalability which is a root requirement of a scalable web server.


{{< figure
		  src="nginx_imgs/architecture.png"
		  caption="([source](https://aosabook.org/en/v2/nginx.html))"
>}}


#### Components of nginx:

- Master: The master process which spawns multiple workers, orchestrates them, configuration updates, socket management, and more
- Worker: Performs the serving work, handles network connections, reads and writes content to disk, and communicates with upstream servers.
- Cache Loader: runs at startup to load the disk‑based cache into memory, and then exits. Scheduled conservatively
- Cache Manager: runs periodically and prunes entries from the disk caches to keep them within the configured sizes.


Here's a brief lifecycle of a request:

{{< figure
		  src="nginx_imgs/infographic-Inside-NGINX_request-flow.png"
		  caption="([source](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/))"
>}}


The core of nginx is responsible for maintaining a tight run-loop and executing appropriate sections of modules' code at each stage of request processing. Modules in nginx take care of most of the application layer work. They read from and write to the network and storage, transform content, do outbound filtering, apply server-side include actions, and pass the requests to the upstream servers when proxying is activated. nginx modules come in slightly different incarnations, namely **core modules, event modules, phase handlers, protocols, variable handlers, filters, upstreams, and load balancers**.



To read more about the internals of nginx:
- [The Architecture of Open Source Applications – NGINX](http://www.aosabook.org/en/nginx.html)
- [Inside NGINX: How We Designed for Performance & Scale](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/)


### How connections are handled


{{< figure
		  src="nginx_imgs/infographic-Inside-NGINX_nonblocking.png"
		  caption="([source](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/))"
>}}

1. The worker waits for events on the listen and connection sockets.
2. Events occur on the sockets and the worker handles them:
    - An event on the listen socket means that a client has started a new chess game. The worker creates a new connection socket.
    - An event on a connection socket means that the client has made a new move. The worker responds promptly.


### Diving Deeper


Let's dive a bit deeper into the code and understand the flow of some critical pieces here.

The *ngx_spawn_process* launches worker processes. 


```C
// src/os/unix/ngx_process.c

ngx_pid_t
ngx_spawn_process(ngx_cycle_t *cycle, ngx_spawn_proc_pt proc, void *data,
    char *name, ngx_int_t respawn)
{
    ....

    pid = fork();

    switch (pid) {

    case -1:
        ngx_log_error(NGX_LOG_ALERT, cycle->log, ngx_errno,
                      "fork() failed while spawning \"%s\"", name);
        ngx_close_channel(ngx_processes[s].channel, cycle->log);
        return NGX_INVALID_PID;

    case 0:
        ngx_parent = ngx_pid;
        ngx_pid = ngx_getpid();
        proc(cycle, data);
        break;
```

The proc function passed above is *ngx_worker_process_cycle* which is the event loop. The core logic of the listening loop is:

```C 
// src/os/unix/ngx_process_cycle.c 

for ( ;; ) {
    ngx_log_debug0(NGX_LOG_DEBUG_EVENT, cycle->log, 0, "worker cycle");

    ngx_process_events_and_timers(cycle);

    if (ngx_terminate || ngx_quit) {
        ngx_log_error(NGX_LOG_NOTICE, cycle->log, 0, "exiting");
        exit(0);
    }

    if (idle) {
        ngx_log_debug0(NGX_LOG_DEBUG_EVENT, cycle->log, 0, "idle worker");
        ngx_process_idle_timers(cycle);
    }
}
```

NGINX provides the following connection methods which is set in the `ngx_event_core_init_conf`  method in `ngx_event.c` :
- select
- poll
- kqueue
- epoll
- /dev/poll
- eventport


These are used for I/O multiplexing and event notification which is at the core of nginx. For more about I/O multiplexing read https://notes.shichao.io/unp/ch6/#io-multiplexing-model

Few more observations:
- The master process creates the sockets and listens
- This is a bit unclear to me from the code as of now, but seems like multiple processes are calling `accept()` when a new connection is available through the chosen multiplexing method. Given that these methods are thread-safe, it makes sense. This is also confirmed by Cloudflare's blog https://blog.cloudflare.com/the-sad-state-of-linux-socket-balancing/ . **It's still a bit unclear and I plan to dig deeper in the code maybe in a separate article. Check the below section for some traditional ways this is done.**

{{< figure
		  src="nginx_imgs/nginx-multi-worker.png"
		  caption="([source](https://blog.cloudflare.com/the-sad-state-of-linux-socket-balancing/))"
>}}

- According to this [presentation](https://www.slideshare.net/joshzhu/nginx-internals), there is an accept_mutex that the worker processes take a lock on before calling the accept. Then pick the accept events for themselves. Below are a few useful images related to the flow. However remember, code is the source of truth.


{{< figure
		  src="nginx_imgs/events-timers.png"
		  caption="([source](https://www.slideshare.net/joshzhu/nginx-internals))"
>}}
{{< figure
		  src="nginx_imgs/accept-mutex.png"
		  caption="([source](https://www.slideshare.net/joshzhu/nginx-internals))"
>}}


But hang on, there's more...

## Socket Sharding

{{< figure
		  src="nginx_imgs/so_reuseport.png"
		  caption="man 7 socket"
>}}

The socket() interface offers what's called SO_REUSEPORT functionality which enables multiple sockets to be bound to the same address, effectively listening from the same PORT. Note that all processes binding should have the same UID (for security to prevent port hijacking). This allows **"load distribution"**  by using different listener sockets for each thread. 

This [article](https://lwn.net/Articles/542629/) from lwn.net goes deeper into discussing SO_REUSEPORT. As mentioned, the 2 traditional approaches in load balancing are :
1. Have a separate thread call accept() and distribute across other workers. Here the listening thread becomes the bottleneck
2. Have all threads wait on accept(). Under high load, incoming connections may be distributed unevenly across all threads. This also is a concern in the thundering-herd problem

By contrast, SO_REUSEPORT allows for even distribution of load across all the listening threads/processes. The kernel takes care of this distribution. 

{{< figure
		  src="nginx_imgs/nginx-before-after.png"
		  caption="https://www.nginx.com/blog/socket-sharding-nginx-release-1-9-1/"
>}}


Check this [article](https://blog.cloudflare.com/the-sad-state-of-linux-socket-balancing/) for a deeper discussion and numbers of how the load balancing plays, and also some of the issues with this approach. Unfortunately in high-load situations, the latency distribution might degrade even for SO_REUSEPORT. The best approach seems to be to use epoll with FIFO behavior and EPOLLEXCLUSIVE flag noted in [this](https://idea.popcount.org/2017-02-20-epoll-is-fundamentally-broken-12/) article. Cloudflare seems to address these issues in its NGINX replacement, [Pingora](https://blog.cloudflare.com/how-we-built-pingora-the-proxy-that-connects-cloudflare-to-the-internet/). 


### Closing Notes

- We have seen nginx basic internals and how the load is distributed among its workers. What is Socket Sharding and how it works
- nginx thread pools are something interesting to look at next - https://www.nginx.com/blog/thread-pools-boost-performance-9x/
  - Thread pools are essentially separate threads to which the worker processes can offload tasks so that they don't block the event loop. aio is used here.
- Scaling accept is another interesting area. How does the kernel maintain incoming connections in a queue and what is its limit
- EPOLLEXCLUSIVE is something to have a look at.
- All that said, nginx is not the most performant proxy anymore and there are new kids on the block, the latest being Pingora by Cloudflare. [Here's](https://dropbox.tech/infrastructure/how-we-migrated-dropbox-from-nginx-to-envoy) a comparison between NGINX and Envoy and why Dropbox made the shift.