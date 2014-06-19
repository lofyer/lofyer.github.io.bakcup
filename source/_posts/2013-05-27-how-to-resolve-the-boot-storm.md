---
title: 'How to solve the &#8216;boot storm&#8217; problem: BCACHE'
author: lofyer
layout: post
comments: true
permalink: /how-to-resolve-the-boot-storm/
duoshuo_thread_id:
  - 1234836220387786876
categories:
  - Memo
---
**Some solutions**  
Way 1. Add more cpus.(ABANDONED)  
Way 2. Add a SSD as &#8220;boot cache&#8221;  
Way 3. Sort the boot process

**POC:**  
**Way 2:**  
Sometimes we need cache the boot section of the OS into a SSD, since no SSD on hand, let&#8217;s try to use a block device made in /dev/shm

**Way 3:**  
Considering that the parrellization of the &#8220;boot action&#8221;, we have to predict the action in the near future

Experiment:  
Using BCACHE now&#8230;or flashcache