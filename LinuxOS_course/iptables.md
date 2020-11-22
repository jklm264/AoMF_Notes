# Linux IP Tables

> "The standard for firewalls on Linux operating systems".

Purpose: CLI firewall. Can also act as router, config external interfaces, NAT (to hide IPs)

- Can do stateless and stateful packet filtering

- More at https://netfilter.org

  

Three tables:

1. Filter - filter, reject, drop packets
2. NAT - proxying: DNAT/SNAT (pre/post routing)
3. Mangle - manipulate content (not just header) of packets



Three chains:

1. Input - when packet dst is FW
2. Output - when packet src is FW
3. Forward - when dst is NOT FW



## Synatx

- iptables [-t table_name=Filter] command chain_name rule_specfiication[options]
- -L lists rules -----> Use **-nvL** instead
- -N/-X create/delete chain
- -F flush-removes all rules
- -A/I[i]/R/D add/insert/replace/delete rule to chain
- -P set default policy for chain; if no rules defined
- -s/-d src/dst IP addr or subnet
- -i/-o specify input/output per network interface
- -j specifies the action [ACCEPT,REJECT,DROP,LOG]
- -p protocol
- -sports/-dports



## Table Policies

Policies are used to set the default action for a chain via *DROP* or *ACCEPT*

Suggested:

- Since most boxes aren't routers/FW that forward traffic, enter: `iptables -P FORWARD DROP`



## Rules

1. Allow external users access to the webserver hosted on *this* machine where eth0 is the iface to the internet: `iptables -A forward -i eth0 -p tcp --dport 80 -j ACCEPT`

- -A FORWARD: only effects packets that aree routed
- --dport 80 matching traffic on port 80
- -j ACCEPT jumps to the target chain of *ACCEPT*



2. **Now**, add a specific webserver (via eth1 iface at 10.0.13.10) to the destination.

`iptables -A FORWARD -i eth0 -o eth1 -p tcp --dport 80 -d 10.0.13.10 -j ACCEPT`



3. **Now**, only external subnet is allowed to access this server (adding src subnet)

`iptables -A FORWARD -i eth0 -o eth1 -p tcp --dport 80 -d 10.0.13.10 -s 10.0.14.0/24 -j ACCEPT`



## Creating Chains

- Without chains, we have to match all rules before we deny the packet (tree vs linear search)

**IF** you want to add (from above examples) rulees for a specific group of machines you would have to make three seperate rules. **OR** use chains!

#### Example

```console
iptables -A FORWARD -p tcp --dport 80 -d
10.0.12.2 -s 10.0.0.1 -j ACCEPT
iptables -A FORWARD -p tcp --dport 80 -d
10.0.12.2 -s 10.0.0.2 -j ACCEPT
iptables -A FORWARD -p tcp --dport 80 -d
10.0.12.2 -s 10.0.0.3 -j ACCEPT
iptables -A FORWARD -p tcp --dport 80 -d
10.0.12.2 -s 10.0.0.4 -j ACCEPT
```

**VS**

```console
iptables -N web_allow
iptables -A web_allow -s 10.0.0.1 -j ACCEPT
iptables -A web_allow -s 10.0.0.2 -j ACCEPT
iptables -A web_allow -s 10.0.0.3 -j ACCEPT
iptables -A web_allow -s 10.0.0.4 -j ACCEPT
iptables –A web_allow –j DROP
iptables -A FORWARD -p tcp --dport 80 -d 10.0.12.2 -j web_allow
```



## Rule Logging

Stored in `/var/log/messages` or `/var/log/syslog`

- can add --log-level [0-7] 
- --log-prefix "<string>" will prepend string on error logged in messages/syslog



## NATting in iptables

Purpose: Iptables can modify packet src/dst NAT

Special/added chains:

- PREROUTING - preform NAT *before* rules kick in
  - You don't want to advertise ssh is in use (port 22)
  - You can't have multiple ssh servers on a single box work correct

- POSTROUTING - perform NAT *after* rules kick in



To take packets bound for port 422 and forward them to 10.10.254.201 port 22:

`iptables -t nat -A PREROUTING -p tcp --dport 422 -j DNAT --to 10.10.254.201:22`



##IPFilter

Pre-iptables; mainly BSD distros.

Config in `/etc/ipf.conf` or `/etc/ipf/ipf.conf`

- Pro: Easier to understand rules



**To display**:

```console
ipfstat -in #displays inbound rules
ipfstat -on #displays outbound rules
```

**To make rules**:

```console
block in all # block all incoming traffic
```

**Rule Attributes:** *block, pass, on interface, proto, from, to, port, keep state* (statefulness)

*quick* ensures the rule is processes.

*map* will do NAT. Ex: `map eth0 10.10.10.0/24 -> 0/32`