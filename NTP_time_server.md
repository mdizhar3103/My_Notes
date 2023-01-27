## Configuring NTP server
```bash
>>> yum whatprovides chrony
>>> yum install chrony -y
>>> firewall-cmd --permanent --add-service=ntp
>>> firewall-cmd --reload
>>> systemctl start chronyd
>>> systemctl enable chronyd
>>> vim /etc/chrony.conf
    # modify accordingly
>>> systemctl restart chronyd
>>> chronyc sources

```
## Configuring ntp clients
```bash
>>> yum install chrony -y
>>> firewall-cmd --permanent --add-service=ntp
>>> firewall-cmd --reload

>>> systemctl start chronyd
>>> systemctl enable chronyd
>>> vim /etc/chrony.conf
    # adding the ntp server entry
    server <ntp_server>.<domain>.<com> iburst prefer
>>> systemctl restart chronyd
>>> chronyc sources
>>> chronyc sources -v
    # MS (Source Mode/State)
    # Source mode  '^' = server, '=' = peer, '#' = local clock.
    # Source state '*' = current synced, '+' = combined, '-' = not combined, '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
```

> [Ref Link](https://docs.oracle.com/en/operating-systems/oracle-linux/8/network/network-ConfiguringNetworkTime.html#ol-chrony-configfile)
