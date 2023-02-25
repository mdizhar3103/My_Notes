## Random Number Entropy

While generating encrypted data we need entropy to create key pairs.

```bash
yum install -y rng-tools
systemctl enable rngd
vim /usr/lib/systemd/system/rngd.service
    # modify the line
    ExecStart=/sbin/rngd -f -r /dev/urandom

systemctl daemon-reload 
systemctl start rngd
systemctl status rngd
```
