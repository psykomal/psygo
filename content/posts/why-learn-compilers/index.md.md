+++
author = "psykomal"
title = "Why am I learning Compilers"
date = "2023-01-03"
description = ""
tags = [
    "compilers", "interpreters"
]
+++

{{< figure
		  src="crafting.png"
		  caption="([source](https://craftinginterpreters.com/a-map-of-the-territory.html))"
>}}

>   *In fact, **Compiler Construction** is, in my own humble and probably embarrassingly wrong opinion, the second most important CS class you can take in an undergraduate computer science program.*
>
>   -- Steve Yegge

## Background


Recently, I attended [the lambda retreat](https://anandology.com/lambda-retreat/), a week-long dive into [SICP - the wizard book](https://mitp-content-server.mit.edu/books/content/sectbyfn/books_pres_0/6515/sicp.zip/full-text/book/book.html) with some [passionate programmers](https://twitter.com/anandology/status/1603804285493743616?s=20) where we built a **scheme interpreter**(the metacircular evaluator), a **scheme compiler to wasm**, discussed programming languages, and a bunch of other [cool stuff](https://github.com/psykomal/lambda-retreat). This inspired me to further build a [**toy scheme interpreter in python**](https://github.com/psykomal/pylisp), and a compiler for the Jack programming language as part of the [**nand2tetris**](https://www.nand2tetris.org/) course.

Yet, it felt like I merely scratched the surface. And as Steve says in his article [Rich Programmer Food](http://steve-yegge.blogspot.com/2007/06/rich-programmer-food.html) - 

> If you don't know how compilers work, then you don't know how computers work. If you're not 100% sure whether you know how compilers work, then you don't know how they work.


This hit me right in the spot and made me wanna learn more. 

## But why?

But why learn compilers, if I am never gonna use them right? This question bothered me and kept me searching for answers. This post is the **compilation** (haha!) of my research so far. 


1. **It's fun and deeply satisfying in itself**. For the sheer joy of it. Demystify our tools. and Flow

	Let's get the main thing out of the way. It's deeply human to get to the heart of anything we love. As a programmer or yet a craftsman, we care about the tools we use and a compiler is a tool we use all the time, often without realizing it. As Phil Eaton says on the [**Spirit of Building**](https://letters.eatonphil.com/2023-01-01-letter-to-a-frontend-developer-asking-about-database-development.html) -

	> *The last thing I want to mention is that (in my experience, anyway) it helps to develop a spirit of building things you don't understand. Write your own React clone, write your own HTTP server, write your own lisp interpreter, write your own compiler, write your own database.*
	> 
	> *I think I started doing this because I just didn't like black boxes. I didn't like when people spoke with hesitation on subjects when answers *could* be discovered. I wanted to know how stuff worked. Or know when further knowledge was truly undiscoverable or impractical.*

	Here's [Julia Evans](https://jvns.ca/about/) for you -
	> *I have one main opinion about programming, which is that deeply understanding the underlying systems you use (the browser, the kernel, the operating system, the network layers, your database, HTTP, whatever you’re running on top of) is essential if you want to do technically innovative work and be able to solve hard problems.*

	According to Amjad Masad, [writing a compiler hits all the buttons to get you into a state of flow](https://amasad.me/compilers)

2. **To learn how a computer works and computer science. `Compilers` is a microcosm of computer science**. 

	Compiler Construction brings in many areas of computer science and software engineering - underlying machine architecture, operating systems, theory of computation (formal languages, automata), programming language design, program design, optimizations, error handling and testing. It also involves complex data structures, data types, and data processing. Even AI (greedy algos and heuristic search)

3. **Open new artistic avenues. Live Programming. Interactivity**

	Amjad Masad demonstrates a program that compiles to [music](http://soundofjs.com/). He [mentions](https://amasad.me/compilers) another project Qalb, an Arabic programming language that can print programs as Arabic calligraphy tiles. Yet another project in his post is [DOMQL](https://amasad.github.io/DOMQL/), a SQL-like language that claims to be a better frontend programming experience than JavaScript. All this makes me wonder what will my code look like as a painting or what it tastes like. [Live coding](https://github.com/toplap/awesome-livecoding) is another interesting thing.
	
	Fun aside, compilers can be used for various productive things. repl.it uses it for detecting the packages from the code and auto-installing them. Compilers can also be made much more interactive, they can detect our intent and suggest edits (AI application?). Jack Rusher in his **[Stop Writing Dead Programs](https://youtu.be/8Ab3ArE8W3s?list=PLcGKfGEEONaDO2dvGEdodnqG5cSnZ96W1)** talk, mentions a lot of possible improvements and insights. Particularly, humans have a large and powerful visual cortex that can be leveraged for programming, and compilers and languages can be designed accordingly, in a more interactive and programmer-friendly way.

4. **Programs that deal with code of any form**

	> Compilers take a stream of symbols, figure out their structure according to some domain-specific predefined rules, and transform them into another symbol stream.
	> 	-- Steve Yegge

	That stream of symbols can be anything from images to sounds. Those might be a rare use case but any form of program that deals with code - Auto format tools, Syntax highlighting, doc extractor, linters, even rendering code in a text editor. Steve mentions [a set of interesting examples in his post](http://steve-yegge.blogspot.com/2007/06/rich-programmer-food.html).

5. **Applicable to many areas**

	Compilers exist in many forms even though they might not be called compilers, the underlying principles and designs are reused in many places. Some of them are :
	- [Code Generation applications](https://yangdanny97.github.io/blog/2022/09/03/scratching-the-pl-itch) - Swagger, JSON schema, ORMs, GenType 
	- [API design](https://yangdanny97.github.io/blog/2022/09/03/scratching-the-pl-itch) 
	- [Domain Specific Languages](https://tomassetti.me/domain-specific-languages/) - GraphQL, DOT, graphviz sed, gawk, ANTLR, make, latex, md. Even HTML, CSS, SQL, all are DSLs
	- [Web Browser Engine](https://web.dev/howbrowserswork/) - Browsers compile HTML and CSS to a rendered web page. See the similarity between the below flow and that of a compiler
			{{< figure
				  src="browser_render.png"
				  caption="Rendering engine basic flow ([source](https://web.dev/howbrowserswork/))"
				>}}
	- [Web Performance](https://yangdanny97.github.io/blog/2022/09/03/scratching-the-pl-itch) - compilers optimizations can also be applied in optimizing browser web performance


6. **High-leverage tools**

	This deserves a separate section just for its weight. Every programmer has to deal with a compiler to build anything. Any small improvement in a widely used compiler means a [huge](https://www.statista.com/statistics/627312/worldwide-developer-population/) impact on the world. 

7. **pdf2json**

	This is a project I worked on a few years back for a client. Anyone who worked on this knows it is a hard problem with tons of challenges given the variance in pdf formats. I am not gonna get into that rant now, but my point is I realize now what I was doing is so much similar to a compiler and could have done a much better job and cleaner code if I was equipped with this knowledge before.

8. **Security** 

	I have a limited understanding of this field but this user has mentioned various ways they used compiler knowledge in reverse engineering products for security vulnerabilities and vulnerability analysis.

9. **Understanding programming languages better**

	Writing a compiler equips one with a deeper understanding of many concepts like static vs dynamic types, GC, JIT, interpreters. All this will make you a better programmer and also help you choose the best tool for the job when the need arises.


## Resources

** (*I will actively keep updating this section as I learn and discover more resources*)

Now that I have enough convincing reasons to go down the compiler rabbit hole and learn by building one. Here are some resources I plan to use -

- [Crafting Interpreters](https://craftinginterpreters.com/)
- [So You Want to Be a (Compiler) Wizard](https://belkadan.com/blog/2016/05/So-You-Want-To-Be-A-Compiler-Wizard/)
- [Nice collection of resources by Phil Eaton](https://lists.eatonphil.com/compilers-and-interpreters.html)
- [Lessons from Writing a Compiler](https://borretti.me/article/lessons-writing-compiler)

If you are just beginning, I would recommend using some of these to get you going and excited -

- [(How to Write a (Lisp) Interpreter (in Python))](http://www.norvig.com/lispy.html)
- [Make a lisp](https://github.com/kanaka/mal)
- [Teeny Tiny compiler (3 parts)](https://austinhenley.com/blog/teenytinycompiler1.html)
- [Lisp interpreter in Rust](https://vishpat.github.io/lisp-rs/overview.html)


### References

- [Rich Programmer Food by Steve Yegge](http://steve-yegge.blogspot.com/2007/06/rich-programmer-food.html)
- [Why Learn Compilers by Amjad Masad](https://amasad.me/compilers)
- This [reddit post](https://www.reddit.com/r/Compilers/comments/101b6kb/why_should_i_learn_compilers/)where I asked this question and got many awesome answers
- [Scratching the PL Itch by Danny Yang](https://yangdanny97.github.io/blog/2022/09/03/scratching-the-pl-itch)
- [Stackoverflow answer](https://stackoverflow.com/a/733190)
- ["Stop Writing Dead Programs" by Jack Rusher (Strange Loop 2022)](https://youtu.be/8Ab3ArE8W3s?list=PLcGKfGEEONaDO2dvGEdodnqG5cSnZ96W1)
- [nikiv digital garden](https://wiki.nikiv.dev/compilers/)


#### Random Musings

- Will keep editing along my journey

