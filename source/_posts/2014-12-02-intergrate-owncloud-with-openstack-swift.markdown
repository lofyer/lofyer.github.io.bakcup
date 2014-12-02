---
layout: post
title: "Intergrate ownCloud with OpenStack Swift"
date: 2014-12-02 23:53:44 +0800
comments: true
categories: 
  - owncloud
---

Recently I've got someone to help him using swift as backend in owncloud, and I wrote this down just in case.

- Create a container, get a publicurl like

```
http://192.168.2.160:8080/v1/AUTH_7d11dd5a3f3544149e8b6a9799a2aa48/oc
```

- Create an endpoint with a region in service swift

```
keystone endpoint-create --service swift --region swift_region \
  --publicurl "http://192.168.2.160:8080/v1/AUTH_7d11dd5a3f3544149e8b6a9799a2aa48/oc"
```

- Install owncloud thirdparty app: ExternalStorage, fill like this

    directory_name: the display name of this containter

    user: project user name

    bucket: container name

    region: swift_region, like above

    key: password

    tenant: project name

    passowrd: password

    service name: swift

    url: with keystone authentication, http://192.168.2.160:5000/v2.0

    timeout: whatever you like, or blank
