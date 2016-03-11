# Windows DNS Content Pack

This version requires Graylog 1.3 minimum, check branches for previous versions.

(Tested with nxLog/Windows 2008 R2/Graylog 1.3)

Newer versions of nxLog with Gelf 1.1 support require an additional parameter for the gelf module "ShortMessageLength -1"

## Includes

* Input (WinLogs-gelf - GELF/UDP/5414)
* Extractors (WinDNS_Debug_Log, WinDNS_Name)
* GROK Patterns
* Dashboard (WinDNS Summary)

## Requirements
* Graylog 1.3  (due to REPLACE extractor)
* Windows DNS server configured for "Log packets for debugging" & "Packet direction: Incoming"
* A GELF capable log exporter/collector such as nxlog or Graylog Collector monitoring the log file path
* Create an ES template to force the ThreadID field type to "String", otherwise ES may dynamically map the field type as INT which would cause indexing errors later on when an alphanumeric ThreadID comes around.

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
            "index":"not_analyzed",
            "type":"String"
          }
        }
      }
    }
}'
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
