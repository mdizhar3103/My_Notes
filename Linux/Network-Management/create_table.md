```bash
systemctl stop firewalld
nft list ruleset
nft list tables
nft flush ruleset   # remove all the things created by default 
nft list tables
nft list ruleset

systemctl start firewalld
nft list tables
nft list table ip filter
```


# Creating table from scratch
```bash
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
