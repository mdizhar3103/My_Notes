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
