---
title: Git Exp.
author: lofyer
layout: post
comments: true
duoshuo_thread_id:
  - 1234836220387786881
categories:
  - Program
---
Supposing that you had a git server, we can use exist git as your own working bare.  
  
**[On your own git server] Create a repo.**  
Clone an exist git.

<pre>$ git clone git@exist-server:exist.git
</pre>

Make its master branch writable.

<pre>$ echo -e "[receive]ntdenyCurrentBranch = ignore" >> exist.git/.git/config
</pre>

**[On your client] Create a branch & make a change.**  
Clone exist.git as your own git-src, in which you can see the old commits and branch.

<pre>$ git clone git@your-server:exist.git
$ cd exist
</pre>

We create a new branch based on its master.

<pre>$ git checkout -b mybranch
$ echo "Add README" > README
$ git add README
$ git commit -m "Add READEME in ROOT"
</pre>

Since this is our first commit, we should push our branch to origin. Next time you should just type &#8216;git push&#8217;

<pre>$ git push origin mybranch
</pre>

**[On your own git server] An update in master.**  
Let&#8217;s get some new commits.

<pre>$ git pull
</pre>

Here we get something like this&#8230;  
[<img src="http://69.164.197.168/wp-content/uploads/2013/04/git21.png" alt="git2" width="397" height="333" class="alignnone size-full wp-image-2000" />][1]  
  
Now we get commit0 and commitA on mybranch, with commit0 and commit1 on branch master.  
What we want is something like this.  
[<img src="http://69.164.197.168/wp-content/uploads/2013/04/git3.png" alt="git3" width="318" height="307" class="alignnone size-full wp-image-2010" />][2]  
  
**[On your client] Merge commit1 to mybranch**  
Way 1.Just merge them from your own git server

<pre>$ git pull origin master</pre>

Way 2.Pull commit1 to local master, then rebase ***or*** merge it to mybranch  
Merge:

<pre>$ git merge master</pre>

Rebase:

<pre>$ git rebase master</pre>

Final step:

<pre>$ git push</pre>

*  
There&#8217;s a little difference between merge and rebase in history.  
Reference:  
<a href="http://gitbook.liuhui998.com/4_2.html" title="GitBook - Rebase" target="_blank">http://gitbook.liuhui998.com/4_2.html</a>*

 [1]: http://69.164.197.168/wp-content/uploads/2013/04/git21.png
 [2]: http://69.164.197.168/wp-content/uploads/2013/04/git3.png