+++
author = "psykomal"
title = "Recurse Center Week 2 - Progress, Propulsion Issues, Reflections"
date = "2023-10-01"
description = ""
tags = [
	"RC", "retreat", "RC append-only log"
]
+++

{{< figure
		  src="repair.png"
		  caption="powered by DreamStudio"
>}}


## Things I've been working on

- The TinySQL database project started with a bang. Got the serialization part and the parser tests working by editing the goyacc file. Got a better understanding of how TiDB query parsing works (from SQL to AST). Btw super useful resource - https://pingcap.github.io/tidb-dev-guide/index.html. The CMU video about Query Execution was very helpful (attached my notes). Plan to write some blog posts after gaining a better understanding of the moving parts

{{< figure
		  src="Query Execution.png"
		  caption="made using MindNode and CMU DB lectures"
>}}


- Realized I needed to understand how parsers work better and got started with Crafting Interpreters. Blitzed through the first few chapters and coded using the "Writing an Interpreter in Go" book - https://github.com/psykomal/monkey-lang . Learned about Pratt and Recursive Descent parsing. Also went into the rabbit hole of Lex, Yacc, and Goyacc. Pretty good talk on the topic - https://www.youtube.com/watch?v=NG0s3-s3whY&pp=ygUGZ295YWNj

- As always, coffee chats with interesting people and pair programmed on TCP in Python. 

- Rust for Rustaceans and Crafting Interpreters book meetups were really fun with interesting conversations and I look forward to attending them. There are always questions after reading through a book, and book reading groups like this really help with these things.

- The RC presentations are something truly awesome and jaw dropping every time. It's awe inspiring to see the things that can happen when you work on edge of your abilities on things you love, merging technical skills with creativity.


## Reflections / What could have gone better

- I feel I have spent much time going into rabbit holes, reading theory, and consuming content rather than programming and producing. Which is fine but somehow I do not feel accomplished. 

- There is also slight overwhelm with having goals of learning a ton and so little time. I need to understand better why I feel I do not have time. As a result, I ended up blitzing through some material and although I do understand some parts of it, there is also some lack of clarity. This makes me think if that time could have been spent more efficiently.

- Anyway, moving forward I think the plan of action is to focus more on the project assignments and learn just enough to complete them. More about taking action and programming first, than theory and rabbit holes. The hole is as deep as you want it to be, but that is the realization as well. It is up to me how deep I want to take it and the level of depth I am happy with. And also the realization that I understand things better when I do them hands-on and hence it's better to focus on doing stuff more than reading stuff. Because at the end of the day, that's what I want to be good at - doing and building stuff. This also has something to do with ["Quantity leads to Quality" / "Pounds of Clay"](https://austinkleon.com/2020/12/10/quantity-leads-to-quality-the-origin-of-a-parable/).


I'll leave this week with a quote about perfectionism by Julia Cameron:

> Perfectionism is the enemy, not the friend of creativity. When we try to get something “right”— meaning perfect— we create a debilitating loop, as we focus only on fixing what we see as wrong and are blind to what is right. The perfectionist re-draws the chin line until there is a hole in the paper. The perfectionist rewrites a sentence until it makes no sense. The perfectionist edits a musical passage over and over, losing sight of the whole. For the perfectionist, nothing is ever quite good enough. Obsessed with the idea that something must be perfect, we lose sight of the joy of creation.
>
> Perfectionism is the ego’s wicked demand. It denies us the pleasure of process. Instead, we are told by the ego that we must have instantaneous success— and our perfectionism believes it, lock, stock and barrel. Perfectionism tells us that to push ahead, we must be perfect. And yet, it is often perfectionism that stalls us and keeps us from moving ahead at all. Perfectionism is the opposite of humility, which allows us to move slowly and steadily forward, making and learning from our mistakes. Perfectionism says do it “right”— or not at all.
>
> *- Julia Cameron from "It's Never Too Late to Begin Again"*