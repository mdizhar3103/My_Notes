### Configuring the caching DNS server
```bash
yum install bind bind-utils
cat /etc/named.conf
man named.conf

vim /etc/named.conf
    # adding multiple listeners
    listen-on port 53 { 127.0.0.1; other_ip; };
    allow-query { localhost; other_ip/netmask; };
    # to allow from all replace all above ip with `any`
    listen-on port 53 { any; };
    allow-query { any; };
    
    recursion yes; # make DNS server to look for other if not available in cache

systemctl enable named -=now
# add the rule to firewalld
firewall-cmd --add-service=dns --permanent
firewall-cmd --reload

# testing the server
dig @localhost google.com

```

#### Configuring DNS Zone
```bash
vim /etc/named.conf
    # creating our own zone
    zone "test.com" IN {
        type master;
        file "test.com.zone"
    };

vim /var/named/named.localhost
cp -p=ownership /var/named/named.localhost /var/named/test.com.zone
vim /var/named/test.com.zone
    # TTL - Time To Live
    # IN - internet
    # NS - Nameserver
    # CNAME - Canonical Name
    # MX - Mail Exchanger Record
    # TXT - Text Record
    $TTL 1H   
    test.com. A 
    @ IN SOA @ admin.test.com. {
        0       ; serial (to refresh local copy )
        1D      ; refresh (check for any change)
        1H      ; retry (how long to wait before trying to refresh again after last failed attempt)
        1W      ; expire (if dont get respone from bind server)
        3H }    ; minimum (minimum time to cache negative responses)
    @      NS  dns1.test.com.
    @      NS  dns2.test.com.   ; second backup named server
    dns1    A       ip_of_server1
    dns2    A       ip_of_server2
    @       A       ip_of_server
    www     A       ip_of_server
    www     CNAME   ip_of_server
    test.com MX 10  m1.test.com.
             MX 20  m2.test.com.
    m1      A       ip_of_server
    m2      A       ip_of_server
    server1 AAAA    ipv6_of_server
    test.com. TXT   "Write your own test here any text"

systemctl restart named.service
dig @localhost test.com
dig @localhost m1.test.com
dig @localhost test.com ANY
```

-----------------------
# Configuring DNSMasq
```bash
yum install -y dnsmasq
groupadd -r dnsmasq 
useradd -rg dnsmasq dnsmasq
cp /etc/dnsmasq.conf /etc/dnsmasq.conf_bak
ls -l /etc/dnsmasq.d/
vim /etc/dnsmasq.conf
    listen-address=127.0.0.1
    port=53
    bind-interfaces
    user=dnsmasq
    group=dnsmasq
    pid-file=/var/run/dnsmasq.pid
    resolv-file=/etc/resolv.dnsmasq
    no-hosts

vim /etc/resolv.dnsmasq
    # add the following line
    nameserver 8.8.8.8
    nameserver 8.8.4.4

systemctl restart dnsmasq
systemctl enable dnsmasq --now
systemctl status dnsmasq

# adding domain server
vim /etc/dnsmasq.conf
    listen-address=127.0.0.1,<dnsserver_ip>
    domain-needed
    bogus-priv
    dns-forward-max=100
    cache-size=500
    port=53
    bind-interfaces
    user=dnsmasq
    group=dnsmasq
    pid-file=/var/run/dnsmasq.pid
    resolv-file=/etc/resolv.dnsmasq
    no-poll
    no-hosts

systemctl restart dnsmasq
systemctl status dnsmasq
netstat -tnlp
```
