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
```
