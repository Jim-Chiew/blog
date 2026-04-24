---
title: Wazuh - Ingesting application log (Jellyfin)
description:
draft: false
tags:
  - wazuh
  - SIEM
  - EDR
created: 10/16/2025 11:16
updated: 23/23/2026 21:23
---
#wazuh #SIEM #EDR
Below is an example of `<localfile>` in the `/var/ossec/etc/ossec.conf` on an endpoint with jellyfin running. The `<localfiles>` have functionalities like:
- Getting log files with daily dates.
- Customizing output format.
- Restrict and Ignore. Filtering out logs
There are more functionalities but those 3 above are my most used ones.

Jellyfin is a self hosted media service platform. Think Netflix but running on your device with the media you own.

# Configuration:
Jellyfin as an example:
```xml
 <localfile>
    <log_format>syslog</log_format>
    <location>/jellyfin/Config/log/log_%Y%m%d.log</location>
    <ignore type="osregex">Emby.Server.Implementations.MediaEncoder.EncodingManager</ignore>
    <ignore type="osregex">Emby.Server.Implementations.ScheduledTasks.TaskManager</ignore>
    <ignore type="osregex">Emby.Server.Implementations.Session.SessionManager</ignore>
    <out_format>jellyfinlog $(log)</out_format>
  </localfile>
```

# Location with datetime:
location defines where the log file is stored. 
You can use `strftime` format strings to get logs that have datetime.
Here is a link to [strftime cheat sheet](https://strftime.org/)

example: `<location>/jellyfin/Config/log/log_%Y%m%d.log</location>`
UPDATE: Log file location for Jellyfin installtion without docker is at: `/var/log/jellyfin/` unless specified with `$JELLYFIN_LOG_DIR` env variable.

## NOTE: also accepts wildcard and environment variables:
Examples
Wildcards: `<location>/nginx/logs/proxy-host-*_access.log</location>`
environment variables (windows endpoint only): `<location>%WINDIR%\Logs\StorGroupPolicy.log</location>` 

See document [here](https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/monitoring-log-files.html)

# Ignore logs that matches a condition:
As the name suggest, it can be used as a way to filter out logs. 
example:
```xml
<ignore type="osregex">Emby.Server.Implementations.MediaEncoder.EncodingManager</ignore>
<ignore type="osregex">Emby.Server.Implementations.ScheduledTasks.TaskManager</ignore>
<ignore type="osregex">Emby.Server.Implementations.Session.SessionManager</ignore>
```
Note: 
- The default regex type is `PCRE2`. For more acceptable regex type checkout [here](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/localfile.html#ignore). checkout regex syntax [here](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html)
- if you want to only include logs matching a condition checkout [restrict](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/localfile.html#restrict).

# Custom output format to be sent to wazuh server:
You can customise how to output is to help with `prematch` when decoding. This is useful when the `pre-decoder` is not decoding it well. or if you just want to use your own decoder.

example:
```xml
<out_format>jellyfinlog $(log)</out_format>
```

field substitution is done with the syntax `$(parameter_here)`  
Some acceptable parameter include the original log labelled as `log`:

|   |   |
|---|---|
|`log`|Message from the log.|
|`json_escaped_log`|Message from the log, escaping JSON reserver characters.|
|`base64_log`|Message from the log, encoded in base64.|
|`output`|Output from a command. Alias of `log`.|
|`location`|Path to the source log file.|
|`command`|Command line or alias defined for the command. Alias of `location`.|
|`timestamp`|Current timestamp (when the log is sent), in RFC3164 format.|
|`timestamp <FORMAT>`|Custom timestamp, in `strftime` string format.|
|`hostname`|System's host name.|
|`host_ip`|Host's primary IP address.|

# Resources
[`<Localfile>` Options](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/localfile.html)
[Configuration for monitoring log files](https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/monitoring-log-files.html)
https://jellyfin.org/docs/general/administration/configuration/