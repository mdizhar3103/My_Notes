### Firewalls, Iptables and TCP Wrappers
Securing Hosts and services 

- Firewall filters network traffic based on rules specified.
- Firewall filters based on information in the packet Source IP, Destination IP, and Destination Port 

```bash
# Firewall Architecture

    +-----------+----------+
    | firewall  | service  |
    +-----------+----------+
    | iptables  | command  |
    +-----------+----------+
    | netfilter | kernel   |
    +-----------+----------+

```

**Netfilter**

- A collection of table, and a table is a collection of chain, chain are evalutaion point where we define set of rules, rules are collection policy which are based on match criteria. And Match specifies the target
- Each table has a chain, each chain has rules, each rules has a match and target.
- Rules are processed from top to bottom, if there are no Rule match found there is another rule ***default***
- Tables - collection of chain based on a purpose
- Chains - Determine what time in the packet flow it's evaluated 
- __PREROUTING__: Before routing decision
- __INPUT__: delivered to local processes
- __FORWARD__: routed, non-local packets
- __OUTPUT__: sent packets, from local processes
- __POSTROUTING__: after routing decision
- Matches: describe interesting packets
- __Targets__ 
    - ACCEPT: Allow packet
    - REJECT: Replies prohibited
    - DROP: drops, no replies
    - LOG: record info to file
- __States__:
    - NEW
    - ESTABLISHED
    - RELATED
- Default Tables and their Chains


| Tables | Chains |
|--------|--------|
|Filter | INPUT, FORWARD, OUTPUT |
| NAT | PREROUTING, OUTPUT, POSTROUTING |
| MANGLE | PREROUTING, INPUT, FORWARD, OUTPUT, POSTROUTING |



Packet capturing
----------------
```bash
tcpdump -D
  # -D: to show you the available interfaces you can collect packet data from

sudo tcpdump -i eth0 -s0 -w sample.pcap
  # -i: Captures packets coming from and going to interface 
  # -s: Sets the snaplengh to the maximum size
  # -w: Writes the capture into a packet capture (pcap) format.

tcpdump -r sample.pcap
  # -r: to read and output the contents of the pcap file

tcpdump -r sample.pcap host 172.31.37.100 and port 80
  # filter based on host and port 
```
