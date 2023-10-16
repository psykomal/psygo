+++
author = "psykomal"
title = "Recurse Center Week 4 - One Ship, Multiple Worlds"
date = "2023-10-08"
description = ""
tags = [
	"RC", "retreat", "RC append-only log"
]
+++

{{< figure
		  src="ship.png"
		  caption="powered by DreamStudio"
>}}



## This week

- 💭 First things first, these weekly checkpoint reviews help me see what's working and not, realign with the larger goal, and also realign the goal accordingly (haha :)

- 💽 The pingcap assignments are going smoothly. Worked on Predicate Pushdown, Count min Sketch (learned about how it is used in query estimation, other approaches being Histogram and TopN), Join Reordering and Access Path Selection using Skyline Pruning. One realization is every pingcap project if done right will take 1-2 weeks (reading relevant research papers, understanding code, etc), which I would love to give but not currently. So my strategy to short-circuit the learning process is to try to understand the high-level picture and not spend too much time diving deep into any single topic. After all, my goal is to develop a good understanding of all the moving parts in a Distributed Database System at a high level, develop the framework, and then go deep into whatever is interesting.  With that framework, I can even go to real-world projects and contribute.

- 🦀 This was the week I started becoming confident in Rust and really started enjoying it. The continuous fighting with the compiler and getting it right is a sort of high (might not be optimal but it is what it is). Beginner Rustaceans correct me if I am wrong.

- 🦀 Continued my Bitcask-based [Key-Value Store in Rust](https://github.com/psykomal/kvrs). Learnt File IO and clap (argument parsing crate for Rust) better. Incorporated and refactored the code. This was basically what gave me confidence that I can after all code in Rust, it's not that big a deal (sure it will take some time to grok it, but it is worth it). An interesting observation is how the compiler forces you to learn systems concepts :) The next to-do here is completing the compaction part (learning about size and level-tiered compaction) and later introducing networking, logging, and benchmarking. I think this will be my other main project for RC. 

- 🦀 Rust Book meetup crew is amazing. Although some things go over my head, I think just attending the meetup is accelerating my learning. 

- 🧶 Crafting Interpreters is another thing that is going on slowly. I am doing the first part of the book in Golang. 

- 🧑‍💻 In a parallel universe (as a part of Empowered Programmer Cohort), I am also learning about concurrency at a low level. Reading about CAS (Compare and Swap), Non Blocking Data Structures, Primitives. Tried implementing a [simple broadcast op](https://github.com/psykomal/broadcast-rs) in Rust. Picked up [The Art of Multiprocessor Programming](https://www.oreilly.com/library/view/the-art-of/9780123705914/), and its associated [lectures](https://www.youtube.com/playlist?list=PLbsY-4I8oat9o7p4re3308L4uk0YJe8ez), and the CMU Parallel Computer Architecture and Programming course is amazing (doing it in ***parallel*** 😆)


## Reflections

- I think it's going great. Too many parallel projects, and not enough time. That's life ig.

- Last week was a little in the heads-down mode. Will try to balance it this week with being more social. 

- I also am not producing stuff currently (more focused on consumption and learning). That's ok I think. Will get to write some blog posts after I am in a comfortable place with all the concepts I am dealing with now. 
