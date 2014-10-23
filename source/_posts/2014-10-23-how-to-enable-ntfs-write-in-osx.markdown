---
layout: post
title: "How to enable NTFS write in OSX"
date: 2014-10-23 10:54:53 +0800
comments: true
categories: 
  - Apple
---

Since NTFS driver does exist in OS X originaly, we don't need to use third-party software to do that.

For example, you have got a NTFS u-disk, and its label is `MYDISK` .

* Open `Terminal`, run command like this:
```
sudo su
echo "LABEL=MYDISK none ntfs rw,auto,nobrowse" > /etc/fstab
```

Then umount your disk by clicking the umount icon in Finder, and re-plug MYDISK. You shall see `MYDISK` in /Volumes and can mkdir in it.


* The only problem is now, when you plug your device, you’re not gonna get a icon on Desktop or Finder. Mac stores its mounted devices in hidden folder /Volumes, so while we have Terminal up and running, make symbolic link of it to the Desktop:
```
ln -s /Volumes ~/Desktop/Volumes
```
