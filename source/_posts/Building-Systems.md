---
title: Building Systems
date: 2022-06-26 08:22:10
tags: Essay
---

Information technology is composed of various systems, just like mechanical technology and electronic technology. Of all these systems, building system is an important part for the final systems.

<!-- more -->

## Why we need building systems?

Different from the physical systems, programmers realize one information system via the code which can be analyzed by the compliers.

Considering that the code line number of modern software is usually beyond 10000+, the software is always organized in different files wirttern by different person. 

Usually, compiling one source code file is easy while compiling hundreds of files is difficult. And that's why there are so many building systems.

## Some popular building systems

I always meet different building system while I look through the open source information systems.

The programming language is easy for me to understand whereas the way to organize their code files is sometimes difficult to understand.

Python-based projects are easy to understand. Actually there is not a explicit building system in these projects since compilers will automatically identify the "import" key word and execute the program in a runtime way.

Javascript-based projects are harder compared to Python-based since they need to be compiled overall firstly. C-based projects are the most difficult to understand. Their 'make' tools are full of inexplicable options, such as '-I', '-f' and etc.
