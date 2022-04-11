---
title: 'My Blog on Linux: A Burdensome Tour'
date: 2022-04-10 23:03:05
tags: Essay
---

I always write some essays on my hexo-style blog using different Windows PCs. Today when I changed my PC to Linux OS, many problems incurs, especially on how to implement version control on diversified softwares.

<!--more-->

## node, npm, nvm

Node.js is commonly used in areas of Web and GUI without doubt. My blog is also deployed and decorated in hexo, a node module. In past few days, When I use hexo deploy blog on my windows PC, it is easy and convenient. Just download node.js and Node Package Manager(npm), and use some intstructions, all the things would be done. 

But things are different on my Linux PC. 

First, I want to use apt-get to install node.js. But the version of my Linux is ubuntu18.04 and the node.js package downloaded from apt is 3.5.2, which is much lower than many package requirements. Besides, after installing npm I want to update my node.js, but it told me npm version was not suitable for node version. So I have to use the low version node.js and npm to install hexo and there are many warnings during the process of installation. 

Second, when I use the node and npm from apt, the installation of hexo will incur a EACCESS error. That is, it can not handle the global installation. 'npm install xx -g' is actually a joke! I need to find alternative methods.

Third, finally I read the hexo document, it told me that if I want to install node and npm on Linux, you'd better install nvm first. what the hell is that??? I just want to use hexo. But I have no choice to find out how to install it. Fortunately, it is a POSIX-compliant bash script. Cloning and running this script compliation, dazzled versions of node and npm were installed. At last, I can change it with fluency and hexo is constructed on my Linux PC. 

## hexo and themes

When I finish my installation tour of nvm, node, npm and hexo, I think the remaining will be done quickly. **However**, I was in a cheerful mood too early. 

After cloning my blog source code to local and implement 'npm install' instruction, hexo can not convert markdown files to html format. Themes also need to be installed. So I clone the latest version of my hexo theme icarus repository and received a unexpected result: version not suitable. Fortunatelly, I find the icarus github page and solve this question by installing it using npm.

## Enlightment

This process took me about whole afternoon time. I was taught a lesson and will consider below tips when I install something on my Linux PC next time.

1. Apt-get is a useful tool while downloading and installing some mature Unix tools such as curl, wget, sed, grep and etc. With regard to the rapidly changed softwares, Apt-get is usually not a good scheme. There are so many versions of some new widely used tools and their packages, such as node, python, go, docker and so on. Apt-get is not a professional expert in the version control domains. Using specific version control tools but not apt maybe an efficient method for these softwares.

2. Official websites and documents is really useful. If you want to use some tools to do some interesting things, it is not bad to read the official instructions in advance. Don't do it for granted, but plan the best way.

3. Linux is not as good as in imagination because many problems incur when you would like things to run. If you don't care the details in one software, Windows will be better than Linux.

