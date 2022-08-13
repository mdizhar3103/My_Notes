## Configuring IMAP Service
```bash
yum install dovecot
systemctl enable dovecot --now
systemctl status dovecot
firewall-cmd --add-service=imap  --permanent
firewall-cmd --add-service=imaps --permanent
firewall-cmd --reload

vim /etc/dovecot/dovecot.conf
    # add the following line
    protocols = imap 
    listen = ip_address

ls -l /etc/dovecot/conf.d/
vim /etc/dovecot/conf.d/10-master.conf
    # modify the following changes
    # uncomment: inet_listener imaps {
        port 993
    }

vim /etc/dovecot/conf.d/10-mail.conf
    # uncomment the following line
    mail_location = mbox:~/mail:INBOX=/var/mail/%u
    # save and exit

vim /etc/dovecot/conf.d/10-ssl.conf
    # look for ssl = required and change it as
    ssl = yes       # to connect without proper ssl configuration
    # save and exit

```
