---
layout: post
title: "octopress cheatsheet"
date: 2014-07-15 08:12:16 +0800
comments: true
categories: 
  - octopress
---

Here's some useful tips of octopress.

1. rake deploy non-fast-forward error
---

Please keep in mind this is not considered best practice, but it may work for you.

The solution is to force a push on the master branch.

Edit the Rakefile and look for this line:

```
system "git push origin #{deploy_branch}"
```

Alter the line by adding a plus (+) before the #{deploy_branch} tag:

```
system "git push origin +#{deploy_branch}"
```

Run the command

```
rake deploy
```

It should succeed.

Undo the edit you made to the Rakefile!

2. update octopress
---

```
git pull octopress master
bundle install
rake update_source
rake update_style
```
