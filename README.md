# Graylog Web Interface
## Includes

* Input (GELF udp 5414)
* Extractor (WinDNS_Debug_Log)
* GROK Patterns
* Dashboard (WinDNS Summary)

## Requirements

* Windows DNS server configured for "Log packets for debugging" & "Packet direction: Incoming"
* A GELF supported log exporter/collector such as nxlog or Graylog Collector monitoring the log file path