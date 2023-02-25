## Working with Networking
```bash
ip a
ifconfig
ip r
cat /etc/resolv.conf
ls -l /etc/sysconfig/network-scripts/
vim /etc/sysconfig/network-scripts/ifcfg-<your_network_card>
hostname
cat /etc/hosts
nmcli c s

```

### More on Networking 
```bash
# managing Network Service on Boot
systemctl status NetworkManager
nmcli c modify <network_interface `enp0s3`> autoconnect yes

ss -tlnup
# OR
netstat -tnlp
```

### Statically routing IP traffic
```bash
# Two Servers on different network 
Server A (10.12.14.3)           Server B (192.168.43.5)
Network A (10.12.0.0)           Network B (192.168.43.0)

# Right now they both can't talk to each other
# To make them connect each other we need to bind them in a separate network together

Server A (10.12.0.1)   <=> Server C  <=>   Server B (192.168.43.5)
Network A (10.12.0.0)                       Network B (192.168.43.0)

ip route add 192.168.43.0/24 via 10.12.0.1
# specifying for specific network device
ip route add 192.168.43.0/24 via 10.12.0.1 dev ens33

# deleting the route
ip route del 192.168.43.0/24

# Gateway
ip route add default via 10.12.0.1
ip route del default via 10.12.0.1

# adding route to connection
nmcli c s
nmcli c modify ens33 +ipv4.routes "192.168.43.0/24 10.12.0.1"
nmcli device reapply ens33      # to immediately apply
ip route show

# to remove
nmcli c modify ens33 -ipv4.routes "192.168.43.0/24 10.12.0.1"
nmcli device reapply ens33      # to immediately apply
ip route show
```
