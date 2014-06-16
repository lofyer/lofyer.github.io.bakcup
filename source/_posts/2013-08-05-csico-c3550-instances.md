---
title: Csico C3550 Instances
author: lofyer
layout: post
permalink: /csico-c3550-instances/
categories:
  - Networking
---
For details, see  
<a href="http://www.cisco.com/en/US/docs/switches/lan/catalyst3550/software/release/12.1_19_ea1/configuration/guide/SwCfgIX.html" title="3550software" target="_blank">http://www.cisco.com/en/US/docs/switches/lan/catalyst3550/software/release/12.1_19_ea1/configuration/guide/SwCfgIX.html</a>

## Assign IP on port 1

<pre>> enable
# configure terminal
(config)# interface f0/1
(config-if)# no switchport
(config-if)# ip address 192.168.1.254 255.255.255.0
(config-if)# no shutdown
(config-if)# end 
</pre>