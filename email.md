## Configuring Email (POSTFIX service)
```bash
yum install postfix
systemctl enable postfix --now
systemctl status postfix
sendmail <username>@localhost <<< "Testing email service locally"
cat /var/spool/mail/<username>
```

### Email Aliases
```bash
vim /etc/aliases
    # add the following line
    <alias_name>: <username>
newaliases
sendmail <alias_name>@localhost <<< "Testing aliases"
cat /var/spool/mail/<username>

# add list of users
vim /etc/aliases
    <alias_name>: <username1>,<username2>

    # mailing to other server
    <alias_name>: <username>@<full_dns_name>
newaliases
```

### Configuring mail using ssmtp
```bash
Step1.  Install “ssmtp”

>>> yum install ssmtp

Step2.  Edit the ssmtp configuration file to add the mailhub server

>>> vim /etc/ssmtp/ssmtp.conf
    # Change the following parameters   
    mailhub=mailhub.accudyneindustries.com
    Hostname=ussdor05.accudyneindustries.com

Step3:  Test Email

>>> ssmtp -t

From:xyz@abc.com
To:Mohd.Izhar@abc.com
Subject:Test Email from DB

Hi,

This is a test email

#Press ctrl+D to send email
```
