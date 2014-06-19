---
title: arduino+cpuload+serial+led
author: lofyer
layout: post
comments: true
simplecatch-sidebarlayout:
  - 
duoshuo_thread_id:
  - 1234836220387786888
categories:
  - Arduino 2560
---
用libgtop2的库，已完成。  
注意：  
数据以ascii类型传输，调用led时转化为int  
cpu利用率算式:

<pre>(float)(100*(cpu_time2.(sys+user)-cpu_time2.(sys+user))/(cpu_time2.total-cpu_time1.total))
</pre>