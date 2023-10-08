+++
author = "psykomal"
title = "Recurse Center Week 3 - Diversions, Impossible Stuff"
date = "2023-10-08"
description = ""
tags = [
	"RC", "retreat", "RC append-only log"
]
+++

{{< figure
		  src="rockets.png"
		  caption="powered by DreamStudio"
>}}


## Things I've been working on



- Database pingcap assignments are going great. This week was mostly on Data Definition Language. More specifically TinySQL's DDL (which is a subset of TiDB's DDL). The asynchronous schema change in TinySQL follows the algorithm for schema changes in Google F1. The algorithm is described in the paper [Online, Asynchronous Schema Change in F1](http://static.googleusercontent.com/media/research.google.com/zh-CN//pubs/archive/41376.pdf). Reading the paper is a prerequisite (although focussing on the algo and less on the proof should do) to understand what was going on in the code and add code for the missing parts for `updateVersionAndTableInfo`, `Add Column`, and `Drop Column`. I will soon add a blog post about TinySQL, although for now focussing on understanding all the moving parts.

- Focussed sometime to improve Rust knowledge. Read Rust for Rustaceans Ch 2. However realized R4R is for intermediate developers, and went through multiple books and resources to understand what was going on. 

- So when it was the **Impossible Stuff Day**, my heart naturally went to a blend of Rust + Databases. Luckily pingcap has a course called [Practical Networked Applications in Rust](https://github.com/psykomal/talent-plan/blob/master/courses/rust/README.md#practical-networked-applications-in-rust) focussing on building a single networked, multithreaded, and asynchronous key-value store in Rust. I was however putting this off for the future due to my lack of experience with Rust. However, this is what Impossible Stuff Day is for. I quickly realized it can be simple (not easy) to code in Rust. I was able to get `clap` working and use `serde` to serialize data and store it to disk using a `BufWriter`. Maybe because I had already done this kind of thing in Golang before, the DB part (based on Bitcask) was clear in my head. All I had to do was focus on Rust. Although I could make it through 1.5/6 assignments, it gave me a flavor of the language. I am consciously holding this as a side project for now and will actively focus on TinySQL and TinyKV.

- The pairing session with [Piya Gehi](https://pjg1.site)  on [sadservers.com](https://sadservers.com) was fun. Looking forward to more of grokking the servers and terminal

- Coffee chats were great as usual. One thing I didn't mention till now was how awesome the Presentations are at RC. From cool products, to ML, IOT, biology, finance, and computer systems, the topics, interests, and depth vary widely and are inspiring. I have got so much FOMO already about the things I should be doing or learning. Just check these blogs by recursers to know what I mean - [Flappy Dird: Flappy Bird implemented in MacOS Finder](https://blaggregator.herokuapp.com/post/uHx79O/view "https://blaggregator.herokuapp.com/post/uHx79O/view"), [An Interactive Intro to CRDTs](https://blaggregator.herokuapp.com/post/RpluCV/view "https://blaggregator.herokuapp.com/post/RpluCV/view").


## Reflections/Thoughts


- This week has gone much better than last week and I do feel accomplished. I think it is a combination of getting things done, meeting interesting people, and moving closer to my goals.

- I am being drawn to the world of low-level ML slowly. I am still focussing my time on Databases but this is a topic that is slowly capturing my interest as well.

- For next week, the focus is on TinySQL and completing and understanding as much as possible. A little pinch of Rust, Interpreters, and maybe ML :) 

