# Windows DNS Content Pack

This version requires Graylog 3.1 minimum, check tags for previous versions.

(Tested with Filebeats/Windows 2016 R2/Graylog 3.1)

**Note this was built using filebeats as the log exporter.  It is possible to use your own input with nxlog or alternatives but will require manually importing the extractors_standalone.json to the input.**

Newer versions of nxLog with Gelf 1.1 support require an additional parameter for the gelf module "ShortMessageLength -1"

## Includes

* Input (TCP_WindDNS_1555 - Beats/TCP/1555) w/ Extractors (WinDNS_Debug_Log, WinDNS_Name)
* GROK Patterns (prefixed with WINDNS to avoid override)
* Dashboards (DNS requests (24h), DNS requests (7d))

## Requirements
* Graylog 3.1 
* Windows DNS server configured for "Log packets for debugging" & "Packet direction: Incoming"
* A log exporter/collector such as nxlog or filebeats monitoring the log file path specified in dns debug (e.g. c:\temp\dns_log.txt)
* Create a dynamic ES template to force the ThreadID field type to "keyword", otherwise ES may dynamically map the field type as INT which would cause indexing errors later on when an alphanumeric ThreadID comes around.

For example in ES 5+:
```
curl -XPUT localhost:9200/_template/graylog -d '
{
  "template":"graylog*",
  "settings":{
    "index.refresh_interval":"30s"
    },
    "mappings":{
      "message":{
        "properties":{
          "ThreadID":{
            "index":"true",
            "type":"keyword"
          }
        }
      }
    }
}'
```

## Filebeats/Sidecar Windows Configuration Example using variables ${user.dnslog_path} and ${user.graylog_server}
```
# Needed for Graylog
fields_under_root: true
fields.collector_node_id: ${sidecar.nodeName}
fields.gl2_source_collector: ${sidecar.nodeId}

filebeat.inputs:
- input_type: log
  paths:
    - "${user.dnslog_path}"
  encoding: utf-8
  type: log
output.logstash:
   hosts: ["${user.graylog_server}:1555"]
path:
  data: "C:/Program Files/Graylog/sidecar/cache/winlogbeat/data"
  logs: "C:/Program Files/Graylog/sidecar/logs"
```

## NXLog Configuration Example
```
define ROOT C:\Program Files (x86)\nxlog

Moduledir %ROOT%\modules
CacheDir %ROOT%\data
Pidfile %ROOT%\data\nxlog.pid
SpoolDir %ROOT%\data
LogFile %ROOT%\data\nxlog.log

<Extension gelf>
    Module xm_gelf
    ShortMessageLength -1
</Extension>

<Input dns>
    Module  im_file
    File  "C:\dns.txt"
    SavePos TRUE
    InputType LineBased
</Input>

<Output out> 
    Module      om_udp
    Host        graylog.server.com
    Port        5414
    OutputType  GELF
</Output>

<Route 2>
    Path        dns => out
</Route>
```

## Screenshots

![Dashboard](http://i0.wp.com/www.ohjeah.net/wp-content/uploads/2015/09/windows_dns_logs.png)
