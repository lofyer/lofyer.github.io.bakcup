---
title: Some data structure about clisp
author: lofyer
layout: post
comments: true
duoshuo_thread_id:
  - 1234836220387786880
categories:
  - Data Processing
---
After reading ANSI Common Lisp, I knew that there are vec, list(construct cell), structure, hash_table and some other views I may not know. I&#8217;ll try Pratical Lisp in next period.  
Using clisp, enjoy this pattern of view of btree

<pre>[110]> (sdraw (cons '(1) '(2)))
[*|*]--------->[*|*]--->NIL
 |              |
 v              v
[*|*]--->NIL    2
 |
 v
 1
</pre>

<pre>[112]> (sdraw (cons '1 '2))
[*|*]--->2
 |
 v
 1

</pre>

<pre>[115]> (sdraw '('1 '2))
[*|*]------------------>[*|*]--->NIL
 |                       |
 v                       v
[*|*]--->[*|*]--->NIL   [*|*]--->[*|*]--->NIL
 |        |              |        |
 v        v              v        v
QUOTE     1             QUOTE     2
</pre>

<pre>[116]> (sdraw '('1 ''2))
[*|*]------------------>[*|*]--->NIL
 |                       |
 v                       v
[*|*]--->[*|*]--->NIL   [*|*]--->[*|*]--->NIL
 |        |              |        |
 v        v              v        v
QUOTE     1             QUOTE    [*|*]--->[*|*]--->NIL
                                  |        |
                                  v        v
                                 QUOTE     2
</pre>