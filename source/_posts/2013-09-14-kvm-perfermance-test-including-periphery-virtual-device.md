---
title: kvm perfermance test, including periphery virtual device
author: lofyer
layout: post
comments: true
categories:
  - Memo
---
I mean ***test***, not ***profiling***.

## To virtual server and virtual desktop, we should inspect them from different standards.

For an server:  
response time, throughput, concurrent, utilization  
For and desktop:  
response time, video/audio latency, utilization

## Tools we can use to test

### CPU:

SPEC(open, but not free)  
Unixbench  
Super PI  
Compile linux kernel  
pcmark(not open, not free, but cracked can be found..)  
ffmpeg convert

### graphic:

3dmark

### fs,hd:

hdparm  
iozone  
blogbench  
dbench

### net:

iperf

### ram

ramspeed

## How to start

Initial host env: i5-3470, 8G, gentoo-linux-3.11+, qemu-1.6.0  
Kernel para: 

TBD