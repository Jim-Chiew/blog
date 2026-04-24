---
title: Jellyfin - Enabling trickplay
description:
draft: true
tags:
  - jellyfin
created: 08/23/2025 15:23
updated: 23/08/2026 21:08
---
Jellyfin is a self-hosted media server. Think Netflix but running it on your own device playing you own content. To install or read more, here is the official [documentation](https://jellyfin.org/docs/)  

Trickplay allows for live video scrubbing with preview window when looking through the video timeline.

It took me a while to get this working as the official documentation had a few gaps missing (at the time of this writing).  
Figured it out by noticing some keywords while messing around with the settings.

# How?
Trickplay needs to be enabled per library and it is disabled by default(time of writing). 

Jellyfin web interface admin dashboard: `Libraries -> Libraries -> <Your library of choice> -> <Kebab menu> -> Manage library -> Trickplay`
![[Jellyfin - Enabling trickplay img2.png]]

Enable them and your done
![[Jellyfin - Enabling trickplay img1.png]]

# Trickplay hardware decoding:
I setup jellyin with access to my GPU thus i want trickplay to be extracted withing it.

Jellyfin web interface admin dashboard: `Playback -> Trickplay`
Check `enable hardware decoding`.