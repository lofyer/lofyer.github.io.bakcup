---
title: Compile Android Spice(aSpice)
author: lofyer
layout: post
permalink: /ndk-compile-android-spiceaspice/
categories:
  - Virtualization
---
Ref:  
<a href="http://comments.gmane.org/gmane.comp.emulators.spice.devel/13944" title="http://comments.gmane.org/gmane.comp.emulators.spice.devel/13944" target="_blank">http://comments.gmane.org/gmane.comp.emulators.spice.devel/13944</a>

OS: ubuntu 12.04 x86_64

## 1. You need download some packages at first.

cerbero  
celt-0.5.1.tar.bz2  
libjpegturbo  
openssl-1.0.1e

## 2. What can cerbero do?

Just like Buildroot, cerbero can build an SDK from scratch for you.

### install

<pre>$ git clone http://anongit.freedesktop.org/git/gstreamer-sdk/cerbero.git
$ sudo python setup.py install
$ cp data/* /usr/local/share/cerbero/data/ #In case of autobuild error
cerbero bootstrap
</pre>

In my case, cerbero used ndk-android-r8e.

### essential packages

celt-0.5.1.tar.bz2  
openssl-1.0.1e  
(optional)libjpegturbo  
To build celt, we need source a file

<pre>export NDK=/home/demo/cerbero/android-ndk-r8e
export SYSROOT=${NDK}/platforms/android-9/arch-arm
export CC="${NDK}/toolchains/arm-linux-androideabi-4.7/prebuilt/linux-x86/bin/arm-linux-androideabi-gcc --sysroot=${SYSROOT}"
export CFLAGS='-mfloat-abi=softfp'
</pre>

Save it as **android.buildenv**  
To build celt

<pre>$ source android.buildenv
$ ./configure --prefix=/home/demo/cerbero/dist/android_arm --enable-shared --enable-static --prefix=arm-linuxandroid-eabi
$ make; sudo install
</pre>

To build openssl

<pre>$ source android.buildenv
$ ./config --prefix=/home/demo/cerbero/dist/android_arm --shared os/compiler:arm-linux
$ make; make install
</pre>

## 3. Build bVNC

We need modify bVNC/jni/src/Android.mk to make ndk-build success.  
Add **libffi.a** to CROSSDIR  
Add **&#8211;static** to CFLAGS

Alter **bVNC/jni/Appication.mk** and **bVNC/AndroidManifest.xml** from **android-8** to **android-9**

BTW, we use FreeRDP with commit dbbb341.