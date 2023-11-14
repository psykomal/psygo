+++
author = "psykomal"
title = "RC Retro - The Hogwarts of Codecraft and Nerdery"
date = "2023-11-14"
description = ""
tags = [
	"RC", "retreat", "RC append-only log"
]
+++

{{< figure
		  src="blackhole.png"
		  caption="powered by DreamStudio"
>}}


You must be wondering why there has been no new post in the last few weeks. It's because the spaceship got sucked into a black hole. A hole filled with iron oxide 🦀.


## What was I upto the last few weeks 🚀🚀


- 🦀 Continued my Bitcask-based [Key-Value Store in Rust](https://github.com/psykomal/kvrs/tree/little-raft) work. What started as an impossible day-stuff project became my main project. Added simple size-tiered compaction. Added networking (created a custom protocol) and multithreading using a custom threadpool. Had to understand Arc<> better for this. Also used Rayon crate to benchmark. bench-rs is an amazing crate. Also benchmarked against sled engine. 

- 🦀 Spent a lot of time digging into raft consensus protocol and made the KV store distributed. Found that [openraft](https://github.com/datafuselabs/openraft) had the best documentation but used [little-raft](https://github.com/andreev-io/little-raft) instead because of readability and I wanted to understand Raft from the code level. The paper does an amazing job presenting Raft. Jon Gjengset's [Guide to Raft](https://thesquareplanet.com/blog/students-guide-to-raft/) further helped me debug some livelocks I was facing. I need to find time to write a blog post on all this. 

- 🦀 Ended up creating a KV store server using Axum. Had to learn async-rust as a result which was a rabbit hole ride in itself. Best introductory resource (suggested in the RC Rust Book Reading Club) was the [Tokio tutorial](https://tokio.rs/tokio/tutorial). Next good resources would be the [Programming Rust](https://g.co/kgs/ubMsvp) book and [Rust for Rustaceans](https://g.co/kgs/58jzKt). 

- ∥∥ Another topic I focussed on was Concurrency and Parallelism. Finally understanding topics like Compare-and-Swap, Sequential Consistency, Linearizability, and Lock-free Data Structures. [The Art of Multiprocessor Programming](https://www.oreilly.com/library/view/the-art-of/9780123705914/) is a good book on this topic, although I just read the first 2 chapters. Another few good books I discovered were C++ Concurrency in Action and Rust Atomics and Locks. The [CMU Parallel Computing](http://15418.courses.cs.cmu.edu/spring2016/home) course is a great one as well.

- ⚙️ Distributed Systems was the last topic I focussed on completing Martin Kleppmann's youtube lectures and [Distributed Systems for Fun and Profit](http://book.mixu.net/distsys/index.html)

- I also joined the [Empowered Coder](https://www.empoweredcoder.com) cohort during this time which was amazing and was a really great addition to all this. Would highly recommend it to anyone interested in going deep into distributed systems.


## Things I wanted to do but couldn't

- I wanted to do more low-level ML but didn't find the time with so much to do already. All I did was the first Karpathy Zero to Hero back-propagation project in Rust. And some EfficientML videos

- I also did not find time to connect with some Synthetic Bio enthusiasts and the AI alignment club. I also wanted to connect with some folks whose projects I found mind-blowing but couldn't find the time. Luckily RC happens to be a lifelong thing and I promise to do these once some of my priority projects are done.


## Reflections 🔎



{{< figure
		  src="hogwarts1.png"
		  caption="powered by DreamStudio"
>}}


- I think I really found my tribe in RC. I do mean it when I say RC is the Hogwarts of Codecraft and Nerdery. So many of the links I shared above are due to the courtesy of RC and its wonderful people. 

- I finally understood what it means to work at the **edge of my abilities**. Initially, my plan was to learn Database Internals (in Golang) and just learn basic Rust. The first push for me to start building my Database in Rust was the Impossible Day. Then I started really enjoying it and just worked on it dropping the Golang thing. The **vocational muscles** concept also comes here because that was what I really enjoyed. Now finally I have a Distributed Key-Value Store in Rust. 

- RC also made me comfortable meeting and pairing with new people as a shy introvert. I found myself really enjoying the coffee chats, pair programming sessions, and meetups. Rust Book Club was the one I enjoyed the most. I always went out with a load of information to when I came in. 

- I got exposed to so many new concepts and resources due to RC. AI Alignment, GPU-accelerated compilers, ML in compilers, intricacies of Rust, Moldable Development, the exciting world of low-level ML, Kernel-dev modern applications, awesome ways to present stuff, power of Frontend in explaining stuff, Shaders, Embedded Rust, emulators, runtime, mushroom hunting, WebGPU, and so many interesting things. Did you know that Alan Turing's final work was on [Theoretical Biology](https://www.dna.caltech.edu/courses/cs191/paperscs191/turing.pdf)?

- I am super grateful for having this opportunity and finally Never Graduating from RC 💫💫 . Highly recommend RC to anyone who is looking for a community of passionate friendly and talented hackers to hack on absolutely anything they find interesting. Apply here - https://www.recurse.com/apply 



{{< figure
		  src="hogwarts2.png"
		  caption="powered by DreamStudio"
>}}