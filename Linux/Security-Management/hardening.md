### Hardening Linux
```bash
# Ubuntu Server I am using
systemctl status                        # list default services

netstat -tnlpa

# remove unncessary services
yum -S <package name>

# disabling icmp from server
sysctl -a
sysctl -ar 'icmp'
ping localhost

# temporary changing the setting
sysctl -w net.ipv4.icmp_echo_ignore_all=1
sysctl -ar 'icmp'
ping localhost

# creating number file (to make changes permanent) (higher the number lately file will be read)
vim /etc/sysctl.d/99-icmp.conf      
    # add the following line
    net.ipv4.icmp_echo_ignore_all=1

# to make system to read all files
sysctl --system             
sysctl -ar 'icmp'
ping localhost
```

### Locking the failed login accounts
```bash
man pam_tally2
vim /etc/pam.d/common-auth
    # the entry in file is read line by line if one fails further no entry is checked.

    # success=1 (validate the password)
    auth [success=1 default=ignore] pam_unix.so nullok
    
    # adding the below line to lock the account for certain duration in seconds, and deny after number of failed attempts
    auth required                   pam_tally2.so deny=3 unlock_time=300
    
    auth required                   pam_deny.so 
    auth required                   pam_permit.so 
    auth optional                   pam_cap.so 

# test by login with failed password for 3 times
```
