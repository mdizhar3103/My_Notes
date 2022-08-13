# Linux Host and Network Security
System Logging

- rsyslog (Rocket Fast System log Processing)
- A way to route and store messages
- Logs local messages
- Logs to remote system (both client and server)

```bash
#  Configuring syslog and sending log to centralized logging server with rsyslog    

# Configuring Server
yum install rsyslog -y
systemctl enble rsyslog --now
systemctl status rsyslog

vim /etc/rsyslog.conf
    # Look for: 
    # imuxsock - provides support for local system logging using logger command
    # imjournal - provides access to systemd journal
    # uncomment the line
    $ModLoad imtcp
    $InputTCPServerRun 514

systemctl restart rsyslog
firewall-cmd --add-port 514/tcp --permanent
firewall-cmd --reload

# making selinux to allow this service
sudo semanage port -a -t syslogd_port_t -p tcp 514

# Configuring Client
vim /etc/rsyslog.conf
    # Go to End of file and add the following file
    *.* @@remote-host:514
    # Note: Single @ means udp protocol and double @@ means tcp protocol
    # Adding specific server hostname
    *.* @@fully_qualified_domain_name:514

systemctl restart rsyslog
logger "Sending Message to server"

# on server check the logs
tail -f /var/log/messages

```

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


```bash
iptables
    # table: -t defaults to FILTER
    # chain: INPUT/FORWARD/OUTPUT
    # command: -I (insert), -A (append), -D (delete)
    # match: -m (IP, Protocol, port)
    # target: -j (ACCEPT/REJECT/DROP/LOG)

iptables -t filter -I INPUT -m tcp -p tcp --dport 80 -j ACCEPT
man iptables

# To configure iptables
# Default is firewalld now, remove- or disable it and install and enable iptables

cat /etc/sysconfig/iptables

systemctl status firewalld
systemctl stop firewalld
systemctl disable firewalld

yum install iptables-services -y
systemctl enable iptables --now
systemctl status iptables

iptables -L     # list iptables
iptables -Lv    # list verbose output of iptables
iptables -Lvn   # list numeric values

# blocking traffic from a particular network
iptables -I INPUT -s <ip_adress_or_cidr>/netmask -j REJECT

# allowing from specific ip
iptables -I INPUT -s 192.168.43.5 -p tcp -m tcp --dport 22 -j ACCEPT

iptables -Lvn
iptables -L --line-number

# remove the rule
iptables -D INPUT <rule_number>
iptables -L --line-number

iptables -F     # flush iptables

# reloading the rules back
systemctl restart iptables   # Note: this will not add the rules we added 
iptables -L

# making the rules permanent, add the rules and run the following command
iptables -L
service iptables save
cat /etc/sysconfig/iptables 

```

**TCP Wrappers**

- Host based access control
- Logging
- Use to secure services running at service level
- Limit access based on user, hostname, or network
- Reference order of files matter (allow, then deny), If entry is in both the files, hosts.allow wins
- If no entry is found in either of them, the request service is granted.
- Rules in file are processed top down and first match terminate the processing.

```bash
man hosts.allow
man hosts.deny

# TCP Wrappers Configuration
<daemon list>: <client_ip_or_hostname list> [: <option>:- <option>: ...]

cat /etc/hosts.allow
    # below are example entry to be added
    # sshd : server1.example.com
    # sshd : .example.com : DENY

cat /etc/hosts.deny
    # sshd : server2.example.com

```

### Firewalld
- High level of security 
- Dynamically manage firewall with support for zones
- Uses XML based config files
- Hold open existing connection on config change
- Behind the scene, uses iptables command to communicate with netfilter
- Permanent and runtime config

**Managing Firewall Zone**

- Define the level of trust for a network connection
- Many interfaces in different zones
- Interface can only be in one zone
- Define the things available in that zone, Ports, protocols, and services
- ICMP reachability, NAT Masquerading, Port forwarding
- Default Zones:
    - __public__: default for all network interfaces
    - __block__: blocks all incoming traffic
    - __drop__: rejects the incoming traffic without sending response
    - __dmz__: for the services that need to be available partially
    - __external__: used for external based router configurations, useful in masquerading scenarios
    - __home, internal, work__: these are trusted incoming network accepted with little variations
    - __trusted__: accept all incoming connections

```bash
man firewalld.zones
man firewalld.zone
man firewalld.service

# global config file
cat /etc firewalld/firewalld.conf

# Default config file
ls /usr/lib/firewalld/
    # Zone config file, Service config

# System specific config file
ls /etc/firewalld/

systemctl status firewalld
systemctl enable firewalld --now

firewall-cmd --state

# zones
firewall-cmd --get-zones
firewall-cmd --list--all -zones
firewall-cmd --zone=<zone_name> --list-all
firewall-cmd --get-services
firewall-cmd --zone=<zone_name> --list-services

# add service
firewall-cmd --zone=public --add-service=http

# ports
firewall-cmd --list-ports
firewall-cmd --zone=<zone_name> --remove-port=<port_num>/tcp
firewall-cmd --zone=<zone_name> --add-port=<port_num>/tcp

firewall-cmd --change-interface=<interface_name[ens33]> --zone=internal

# making changes permanent
firewall-cmd --rutime-to-permanent
```

### Port Forwarding
```bash
firewall-cmd --permanent --zone=external --add-forward-port=port=2223:proto=tcp:toport=22:toaddr=192.168.43.5
firewall-cmd --permanent --zone=external --add-forward-port=port=2222:proto=tcp:toport=22:toaddr=
firewall-cmd --reload
firewall-cmd --zone=external --list-forward-port
ip addr

# This should work
ssh username@server_ip -p 2223
ssh username@server_ip -p 2222
```

## SSH Configuration
```bash
# client configuration
cat /etc/ssh/ssh_config         # system wide config
cat ~/.ssh/config               # individual user config and key files (optional)
man ssh_config

# Server config
cat /etc/ssh/sshd_config       # system wide config
man sshd_config

ps -aux | grep key
ssh -v username@server_ip
ssh --help
ssh -l <login_name> server_ip -n "ps -aux --no-headers | sort -nrk 3 | head"
```

**SSH Tunnelling**
```bash
ssh -L <local_port_number>:<remote_server_name/ip>:<port_number_to_access_on_remote_server> remote_server_name_or_ip
# ex: ssh -L 8080:my_server.com:80 my_server.com
# this will also give terminal to avoid getting terminal use -N also 
# ex: ssh -L 8080:my_server.com:80 my_server.com -N

vim /etc/ssh/sshd_config
    # Look for: GatewayPorts 
```

**ssh -X**
```bash
ssh -X username@server_name_or_ip
cat ~/.Xauthority
yum install xorg-x11-apps -y
xeyes       # to see open GUI

# in oracle linux
yum config-manager --enable ol8_codeready_builder   # for linux-8
```

### More on Configuring Firewalls

```bash
firewall-cmd --list-all 
firewall-cmd --list-all --permanent
firewall-cmd --state
ls /etc/firewalld
firewall-cmd --get-default-zone
firewall-cmd --info-zone=public
firewall-cmd --info-service=ssh
firewall-cmd --add-service=http
firewall-cmd --runtime-to-permanent
firewall-cmd --remove-service=http
firewall-cmd --list-all --zone=internal
firewall-cmd --add-source=192.168.1.72/24 --zone=internal
firewall-cmd  --add-port=443/tcp --permanent
firewall-cmd --reload 

journalctl --unit sshd --since "14:00"

systemctl stop firewalld
nft list ruleset
nft list tables
nft flush ruleset   # remove all the things created by default 
nft list tables
nft list ruleset

systemctl start firewalld
nft list tables
nft list table ip filter

# Creating table from scratch
systemctl stop firewalld --now 
nft flush ruleset
nft list tables
nft list ruleset
nft add table inet filter
nft list ruleset
nft add chain inet filter INPUT { type filter hook input priority 0 \; policy accept \; }
nft list ruleset
nft add rule inet filter INPUT iif lo accept
nft add rule inet filter INPUT ct state established, related accept 
nft add rule inet filter INPUT tcp dport 22 accept
nft add rule inet filter INPUT counter drop
nft list ruleset
```
