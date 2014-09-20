---
layout: post
title: "Writing a web server from scratch - 1"
date: 2014-04-07 21:15
comments: true
categories: 
---

I have been reading more about system calls in the Unix kernel recently and am thinking of writing a small application to apply what have learnt and gain some experience with systems programming. So, I'll be building a minimalist high performance web server in C over the next few days. 

Why reinvent the wheel?, you may ask but this is just to expand my personal knowledge and may not be used by anyone else. So the first step would be get a simple static server running that would be just serving all the files present in a directory provided as an option to it. Architecturally, this wouldn't be radically different from a web server with dynamic content and this is the part I want to focus on for now. So to handle multiple requests concurrently there a number of possible designs like using a pre-forking model, multithreading or using non blocking I/O operations. I will be exploring each one of these methods. There will also be some necessary plumbing tasks like parsing and creating HTTP headers. Creating custom handlers for routes would be an interesting problem. Perhaps, I could extend the project by making a micro web framework in C.

I will be blogging about my experiences building this in subsequent posts. 