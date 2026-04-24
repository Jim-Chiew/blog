---
title: Wazuh - Ingest application(kasm) logs. With custom rules and decoders
description:
draft: false
tags:
  - wazuh
  - SIEM
  - EDR
created: 06/29/2025 10:29
updated: 23/23/2026 21:23
---
#wazuh #SIEM #EDR
I have Kasm running and I want Wazuh to monitor its logs. I have Wazuh agent already installed on the host where kasm is installed.

Kasm - Kasm Workspaces provides browser-based access to on-demand containerized desktops and applications.

# Wazuh-agent Configuration for monitoring log files
first is finding the logs for kasm. Official documents shows that the logs is located at `/opt/kasm/current/log/. 

Next edit the `/var/ossec/etc/ossec.conf` configuration file to include the application log file.
Ensure it is within `<ossec_config>` with the rest of the `<localfile>`
```xml
  <!-- Custom stuff -->
  <localfile>
    <location>/opt/kasm/current/log/api_server_json.log</location>
    <log_format>json</log_format>
  </localfile>
```

## last section of the <ossec_config> in conf file should look something like this:
```xml
<ossec_config>
  <localfile>
    <log_format>journald</log_format>
    <location>journald</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/ossec/logs/active-responses.log</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/dpkg.log</location>
  </localfile>

  <!-- Custom stuff -->
  <localfile>
    <location>/opt/kasm/current/log/api_server_json.log</location>
    <log_format>json</log_format>
  </localfile>
</ossec_config>
```

## Restart wazuh-agent
```cmd
sudo systemctl restart wazuh-agent
```
## Resource:
[Kasm](https://www.kasmweb.com/docs/latest/index.html)
[Configure wazuh-agent to monitor logs](https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/monitoring-log-files.html)

# Custom Decoders
The logs is in JSON that contains different KEY_NAME from what I wanted it to be extracted as. So to solve this I'll overwrite the default json decoder and create a modified one.

Note: I have since found a better way of changing the logs before it is sent which allows the use of prematch in the decoder instead of overwriting the original. View [[Wazuh - Ingesting application log (Jellyfin)]]. 
## Modify default decoders
```bash
sudo cp /var/ossec/ruleset/decoders/0006-json_decoders.xml /var/ossec/etc/decoders/
sudo cp /var/ossec/ruleset/decoders/0006-json_decoders.xml /var/ossec/ruleset/decoders/0006-json_decoders.xml.bak
sudo rm /var/ossec/ruleset/decoders/0006-json_decoders.xml
sudo chown wazuh:wazuh /var/ossec/etc/decoders/0006-json_decoders.xml
```

Next, configure the server to exclude the `0006-json_decoders.xml` in  `/var/ossec/etc/ossec.conf` under `ruleset`
```xml
<!-- Custom Stuff-->
    <decoder_exclude>ruleset/decoders/0006-json_decoders.xml</decoder_exclude>
```

your `ruleset` section should look something like this:
```xml
  <ruleset>
    <!-- Default ruleset -->
    <decoder_dir>ruleset/decoders</decoder_dir>
    <rule_dir>ruleset/rules</rule_dir>
    <rule_exclude>0215-policy_rules.xml</rule_exclude>
    <list>etc/lists/audit-keys</list>
    <list>etc/lists/amazon/aws-eventnames</list>
    <list>etc/lists/security-eventchannel</list>

    <!-- User-defined ruleset -->
    <decoder_dir>etc/decoders</decoder_dir>
    <rule_dir>etc/rules</rule_dir>

    <!-- Custom Stuff-->
    <decoder_exclude>ruleset/decoders/0006-json_decoders.xml</decoder_exclude>
  </ruleset>
```

Restart Wazuh manager
```bash
systemctl restart wazuh-manager
```


Modified the `/var/ossec/etc/decoders/0006-json_decoders.xml` to ass `json-kasm`:
```xml
<decoder name="json-msgraph">
  <prematch>"integration":"ms-graph"</prematch>
  <plugin_decoder>JSON_Decoder</plugin_decoder>
  <json_null_field>discard</json_null_field>
</decoder>

<decoder name="json">
  <prematch>^{\s*"</prematch>
  <plugin_decoder>JSON_Decoder</plugin_decoder>
</decoder>
```

# Sibling Decoders
Next we need to specify some decoder rules to get the information we want. 

Wazuh web interface: `Server management -> Decoders -> Add new decoders file`
custom_kasm.xml:
```xml
<decoder name="json-kasm">
<prematch>^{"asctime":"</prematch>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex>"name":"(\S+)"</regex>
  <order>type</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex offset="after_regex">"filename":"(\S+)"</regex>
  <order>program_name</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex offset="after_regex">"funcName":"(\S+)"</regex>
  <order>action</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex offset="after_regex">"message":"(\.*)"</regex>
  <order>data</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex>"message":"Authentication attempt invalid user: \((\S+)\)"</regex>
  <order>user</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex>"message":"Authentication attempt invalid password for user: \((\S+)\)"</regex>
  <order>user</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex>"message":"Successful authentication attempt for user: \((\S+)\)"</regex>
  <order>user</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex>Successfully authenticated request \((\S+)\)</regex>
  <order>action</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex offset="after_regex">"kasm_user_name":"(\S+)"</regex>
  <order>user</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex offset="after_regex">"kasm_user_id":"(\S+)"</regex>
  <order>user_id</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex offset="after_regex">"metric_name":"(\S*)"</regex>
  <order>status</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex offset="after_regex">"kasm_image_name":"(\S+)"</regex>
  <order>kasm_image_name</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex offset="after_regex">"kasm_image_friendly_name":"(\S+)"</regex>
  <order>kasm_image_friendly_name</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex offset="after_regex">"kasm_id":"(\S+)"</regex>
  <order>kasm_container_id</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex offset="after_regex">"path_info":"(\.+)"</regex>
  <order>request_path</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex offset="after_regex">"request_ip":"(\d+.\d+.\d+.\d+), (\d+.\d+.\d+.\d+)"</regex>
  <order>srcip, dstip</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex offset="after_regex">"request_ip":"(\d+.\d+.\d+.\d+)"</regex>
  <order>srcip</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex offset="after_regex">"user_agent":"(\.*)"</regex>
  <order>user_agent</order>
</decoder>

<decoder name="kasm_sub_decoder">
  <parent>json-kasm</parent>
  <regex offset="after_regex">"timestamp":"(\S+)"</regex>
  <order>timestamp</order>
</decoder>
```

Sample JSON log
```
{"asctime":"1111-11-11 11:11:11,111","name":"admin_api_server","processName":"MainProcess","filename":"provider_manager.py","funcName":"get_agent_request","levelname":"DEBUG","lineno":1858,"module":"provider_manager","threadName":"CP Server Thread-8","message":"Requesting Hello for Server(aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa) via URL: (https://proxy:999/agent/api/v1/hello/)","taskName": null, "kasm_user_name":"user","kasm_user_id":"aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa","kasm_id":"aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa","path_info":"/get_kasm_status","request_ip":"10.11.11.11","user_agent":"Mozilla/99.0 (Windows NT 99.0; Win99; x99; rv:999.0) Gecko/11111111 Firefox/999.0","timestamp":"1111-11-11T11:11:11.111111+11:11"}
```
## Resource:
[Custom Decoders](https://documentation.wazuh.com/current/user-manual/ruleset/decoders/custom.html#modify-default-decoders)
[Decoder Syntex](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/decoders.html#regex)
[Sibling Decoders](https://documentation.wazuh.com/current/user-manual/ruleset/decoders/sibling-decoders.html)

# Custom rules
Now to set some rules that will trigger alerts:

```xml
<var name="IGNORE_CLIENT_ACTION">ACTION1|ACTION2|ACTION3|ACTION4</var>

<!-- Custom kasm Rules 102000-102599 -->
<group name="custom_rules, json, kasm,">
 <rule id="102001" level="0">
    <decoded_as>json-kasm</decoded_as>
    <field name="program_name" negate="yes">_cplogging.py</field>
    <field name="type" negate="yes">cherrypy.access.127634205967664</field>
    <field name="request_path" negate="yes">^/__healthcheck$</field>
    <description>kasm grouper rule</description>
  </rule>
  
  <rule id="102101" level="3">
    <if_sid>102001</if_sid>
    <field name="type">admin_api_server</field>
    <description>admin action taken</description>
  </rule>
  
  <rule id="102201" level="3">
    <if_sid>102001</if_sid>
    <field name="type">client_api_server</field>
    <action type="osregex" negate="yes">$IGNORE_CLIENT_ACTION</action>
    <description>client action taken</description>
  </rule>
</group>
```
NOTE: 
- `kasm grouper rule` I negated a few signatures that constantly performs health checks or spamming logging on my system. so excluded it at the very top to minimise the amount of logs being processed.
- There are some action I deemed as not "important" and excluded them to minimise the flood of logs.

## Resource:
[Rules Syntax](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.htm)
[Rules classification/Levels](https://documentation.wazuh.com/current/user-manual/ruleset/rules/rules-classification.html)
[Regex Syntax](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html)

# Thoughts
Creating custom decoders and rules for application is not easy. Some challenges I faced was:
- The web interface does not show you what is wrong with your syntax only that is wrong. 
- The logs in the application server does not match 1-for-1. I think the agent removes unnecessary white spaces.
But overall it was a good learning experience implementing an SIEM.