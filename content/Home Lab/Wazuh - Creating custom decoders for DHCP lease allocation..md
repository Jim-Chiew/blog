---
title: Wazuh - Creating custom decoders for DHCP lease allocation.
description:
draft: false
tags:
  - wazuh
  - SIEM
  - EDR
created: 04/12/2025 18:12
updated: 23/22/2026 21:22
---
A decoder to keep track of which devices is assigned to which IP.

# Creating the decoder:
Wazuh web interface: `Server management -> Decoders -> Add new decoders file`
![[Wazuh - Creating custom decoders image1.png]]
```xml
<decoder name="kdhcp4">
  <program_name>kea-dhcp4</program_name>
  <prematch>DHCP4_LEASE_ALLOC</prematch>
</decoder>

<decoder name="kdhcp4-field">
  <parent>kdhcp4</parent>
  <regex>hwtype\p(\d+) (\S+:\S+:\S+:\S+:\S+:\S+)\p</regex>
  <order>hwtype,mac_address</order>
</decoder>

<decoder name="kdhcp4-field">
  <parent>kdhcp4</parent>
  <regex offset="after_regex">lease (\S*) has been allocated for (\d+)</regex>
  <order>srcip, lease_time_sec</order>
</decoder>
```

# Explanation
Split into sibling decoders with one main and 2 child.

## Main decoder:
```xml
<decoder name="kdhcp4">
  <program_name>kea-dhcp4</program_name>
  <prematch>DHCP4_LEASE_ALLOC</prematch>
</decoder>
```

program_name - matches the program extracted during pre-decoding. in this case we want logs related to  kea-dhcp4. To get the program_name, look at [testing](#Testing) potion of this article.
prematch - ensure that the logs also contains the keyword DHCP4_LEASE_ALLOC.

## Sub/child - decoders (field extractor):
```xml
<decoder name="kdhcp4-field">
  <parent>kdhcp4</parent>
  <regex>hwtype\p(\d+) (\S+:\S+:\S+:\S+:\S+:\S+)\p</regex>
  <order>hwtype,mac_address</order>
</decoder>

<decoder name="kdhcp4-field">
  <parent>kdhcp4</parent>
  <regex offset="after_regex">lease (\S*) has been allocated for (\d+)</regex>
  <order>srcip, lease_time_sec</order>
</decoder>
```
parent - Defines the parent decoder. will trigger child if parent is triggered.
regex -  the rule and what to extract.
order - the extracted data names in its order.

# Testing
Geting log data. Initially I took it from the pfsense web portal but after testing. it did not match what Wazuh was actually receiving. only noticed this after a while of testing. resolved by using `nc` to get the raw data. below is a sample. 

sample data `Apr  1 11:11:11 pfsense kea-dhcp4[00001]: INFO  [kea-dhcp4.leases.0x111111111111] DHCP4_LEASE_ALLOC [hwtype=1 aa:aa:aa:aa:aa:aa], cid=[01:aa:aa:aa:aa:aa:aa], tid=0xaaaaaaa: lease 192.168.1.1 has been allocated for 5555 seconds`

## To test the decoder:
In the same editing decoder page, there is a `Decoders Test` right next to `save`. save your decoder first before testing.
![[Wazuh - Creating custom decoders image2.png]]
The pre-decoding should automatically get timestamp, hostname and program_name.
ensure that your custom/dynamic fields shows up as well.

# Note: Overwrite pre-decoder values.
you can overwrite things like timestamp, hostname and program_name. example:
![[Wazuh - Creating custom decoders image3.png]]
Notice `Phase 1: Completed pre-decoding` did not extract anything.

Fake log `Apr 1 11:11:11 pfsense kea-dhcp4[00001]: INFO [kea-dhcp4.leases.0x111111111111] DHCP4_LEASE_ALLOC [hwtype=1 aa:aa:aa:aa:aa:aa], cid=[01:aa:aa:aa:aa:aa:aa], tid=0xaaaaaaa: lease 192.168.1.1 has been allocated for 5555 seconds`

decoders:
```xml
<decoder name="kdhcp4_fake_pre_decoder">
  <prematch>\S+ \S+ \S+ \S+ kea-dhcp4\S+ \S+ \S+ DHCP4_LEASE_ALLOC</prematch>
</decoder>

<decoder name="kdhcp4_fake_pre_decoder-fields">
  <parent>kdhcp4_fake_pre_decoder</parent>
  <regex>(\S+ \S+ \S+) (\S+) (kea-dhcp4)</regex>
  <order>timestamp,hostname,program_name</order>
</decoder>
```

Note: fake log removed a few spaces here and there. This was what I got when i copied the log from Pfsense web portal. Thus no reading.

# Note: Custom decoder regex only starts from message.
```
Apr 14 19:28:21 gorilla sshd[31274]: Connection closed by 192.168.1.33

**Phase 1: Completed pre-decoding.
        full event: 'Apr 14 19:28:21 gorilla sshd[31274]: Connection closed by 192.168.1.33'
        timestamp: 'Apr 14 19:28:21'
        hostname: 'gorilla'
        program_name: 'sshd'
```
Regex only starts from: `Connection closed by 192.168.1.33.`

# Resources:
[Decoder Syntax](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/decoders.html)
[Regular Expression Syntax](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html)
[About Sibling Decoders](https://documentation.wazuh.com/current/user-manual/ruleset/decoders/sibling-decoders.html)
