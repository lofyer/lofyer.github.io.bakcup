---
title: 'Something about curl &#8212; Connecting IPA Server using json as an example'
author: lofyer
layout: post
comments: true
permalink: /all-about-curl/
duoshuo_thread_id:
  - 1234836220387786883
categories:
  - Linux Admin
---
With Arduino as a server.

What we want is to keep a cookie and build a HEADER  
<a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html" title="HTTP/1.1" target="_blank"></p> <p>
  http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html</a>
</p>

<p>
  [P1]
</p>

<pre>
curl -e
curl -H referer:whereicamefrom.com
curl -d @file.txt
curl -d "somecmd"
curl -cookie
curl -D
</pre>

<p>
  [P2]<br /> <strong>Communicating with ipa server</strong><br /> <a href="https://git.fedorahosted.org/cgit/freeipa.git/tree/API.txt" title="freeipa api" target="_blank">https://git.fedorahosted.org/cgit/freeipa.git/tree/API.txt</a>
</p>

<p>
  Get ca.crt
</p>

<pre>curl -k https://$YOURHOST/ipa/config/ca.crt >> /tmp/ipa.ca.cert</pre>

<p>
  Get sessionid
</p>

<pre>
sessid=$(curl -v 
-H referer:https://ipa.test.net/ipa/ui/index.html 
-H "Content-Type:application/x-www-form-urlencoded" 
-H "Accept:*/*" 
--negotiate -u : 
--cacert ./ca.crt 
-d "user=admin" -d "password=12345678" 
-D cookie.txt 
-X POST 
-k      

https://ipa.test.net/ipa/session/login_password

2>&#038;1 | grep -o "ipa_session=[a-zA-Z0-9]*")
</pre>

<p>
  Post a json file with cmd in it
</p>

<pre>curl -v 
-H referer:https://ipa.test.net/ipa/ui/index.html 
-H "Content-Type:application/json" 
-H "Accept:applicaton/json" 
-negotiate -u : 
--cacert ./ca.crt 
--cookie $sessid 
-d @ipa.json 
-X POST 
-k      

https://ipa.test.net/ipa/session/json

</pre>

<p>
  Here&#8217;s a json file
</p>

<pre>
{
"method":"user_find",
"params":[
        [""],
        {"uid":"admin"}
        ],
"id":0
}
</pre>

<pre>
{
"method":"user_add",
"params":[
        [],
        {
         "uid":"test1",
         "cn":"cn",
         "givenname":"test1",
         "sn":"test1"
        }
        ],
"id":0
}
</pre>