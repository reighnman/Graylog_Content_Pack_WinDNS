# Windows DNS Content Pack
## Includes

* Input (GELF udp 5414)
* Extractor (WinDNS_Debug_Log)
* GROK Patterns
* Dashboard (WinDNS Summary)

## Requirements

* Windows DNS server configured for "Log packets for debugging" & "Packet direction: Incoming"
* A GELF supported log exporter/collector such as nxlog or Graylog Collector monitoring the log file path

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
</Extension>

<Route 1>
    Path        in => out
</Route>

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