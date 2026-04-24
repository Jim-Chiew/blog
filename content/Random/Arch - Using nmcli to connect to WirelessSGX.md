---
title: Arch - Using nmcli to connect to WirelessSGX
description:
draft: false
tags:
  - arch
  - networking
  - wireless
created: 10/18/2025 17:18
updated: 23/48/2026 20:48
---
# Initial setup of wirelessSGx:
Visit: https://eservice.imda.gov.sg/wirelessSGx/

Device without local sim -> Android/ChromeOS & Other
Operating Systems -> Create profile or Reset profile.

Continue with your setup. at the end you will be given a user_ID and password.

# Setup:
Enter the following in your arch or whatever distro that is using nmcli is its network manager:
```
nmcli con add type wifi con-name "wiresgx" ifname wlan0 ssid "Wireless@SGx"
nmcli con modify "mywifi" wifi-sec.key-mgmt wpa-eap
nmcli con modify "mywifi" 802-1x.eap peap
nmcli con modify "mywifi" 802-1x.phase2-auth mschapv2
nmcli con modify "mywifi" 802-1x.identity "UserID"
nmcli con modify "mywifi" 802-1x.password "Password"
```
Replace UserID and Password accordingly.

# Some cool stuff about 802-1x:
802-1x in short is a authentication standard help to securely authenticate a user for network access. There are a few EAP methods available with the most popular being `PEAP` and `EAP-TLS`. 

`EAP-TLS`: Requires a certificate on the user side. it is considered the most secure because it authenticates both the user and server (mutual authentication).  
`PEAP`: Password based that first gets the server cert to establish a secure TLS connection to the REDUIS or authentication server.  Within the established secure tunnel it then authentication the user using protocols like 'mschapv2'.

mschapv2 is interesting to me because of the fact that your password never actually leave your device. by using a Challenge-Response Mechanism by using your credential in the server to encrypt a random challenge. The client uses the password to decrypt the challenge to confirm his identity.  

# Resources:
https://www.securew2.com/solutions/802-1x 
https://www.securew2.com/blog/everything-you-need-to-know-about-peap-security
https://www.fortinet.com/resources/cyberglossary/802-1x-authentication
https://jumpcloud.com/it-index/what-is-eap-mschapv2
https://kcore.org/2022/02/05/lxc-subuid-subgid/