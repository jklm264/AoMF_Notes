# Linux Network Intrusion Detection System

Purpose: Captures and analyzes packets coming and going from a network.

- Can place at environment edge AND between LANs
- Are strictly signature based

## Snort
Purpose: Either IDS or IPS. Easier to set up and deploy than bro.

1. Sniffer - reads network packet and displays to console
2. Packet Logger - will log network traffic
3. NIDS - rule set and analyze traffice
  - Get rules from snort.org and emergingthreats.net

Syntax: `action proto src_ip src_port direction dst_ip [options]`
Ex: `alert tcp $EXTERNAL_NET any -> $HOME_NET 22` 

### Snort's Steps
1 Packet Decoder - decode pkt from iface
2 Prepocessor - reassemble
3 Detection Engine - apply rules
4 Logging and Alerting - action from rules
5 Output modules

## Bro
- Multithreaded
- More aware of services/protocols
- Deesigned for high speed networks

### Bro's Steps
1. Event Engine - capture pkts and puts them together
2. Policy script interpreter - convert to "bro language" then apply rules OR discards events.


