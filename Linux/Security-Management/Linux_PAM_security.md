### Working with PAM in Linux
```bash
# PAM Security
ls /lib64/security    
ls /etc/security
vim /etc/login.defs

# configuring authentication
yum list oddjob*
yum install oddjob
systemctl start oddjobd
systemctl enable oddjobd
authconfig --enablemkhomedir --update
cd /etc/pam.d
grep mkdhomedir *
cat /etc/pam.d/system-auth              # check for pwquality PAM module
cat /etc/security/pwqualtiy.conf        # checks password strength        

pwscore             # to check password score & strength

# Limiting Access
ulimit -a
vim /etc/security/limits.conf

# Adding time restrictions
vim /etc/pam.d/sshd
    # add the following line
    account     required    pam_time.so
cd /etc/security
vim time.conf
    # add the following line
    *;*;<usernames>|<username2>;Wk0800-1800
    | |            |             |        
    | |            |             +--> time zone range to specify (Wk - weekday)
    | |            +--> or for and use &
    | +----> all terminals
    +----> all services

```
