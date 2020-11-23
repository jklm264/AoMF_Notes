# Linux Packet Sniffers

Purposes:

- only works on unencrypted traffic, though metadata can help
- troubleshoot iptable issue
- Etc.



## tcpdump
- industry standard
- `tcpdump -i <interface> `
  - *-n* disables DNS lookup
  - *-w* save output to file
  - *-x* prints entire packet (not just header)
  - *host*, *src*, *dst*

