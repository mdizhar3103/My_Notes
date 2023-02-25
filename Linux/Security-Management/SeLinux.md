# Selinux Contexts
```bash
ls -lZ

# unconfined_u:object_r:user_home_t:s0
    |             |         |       |
    V             V         V       V
   user         role       type     level

# Selinux User and Roles
# developer_u: developer_r, docker_r
# guest_u: guest_r
# root: staff_r, sysadm_r, system_r, unconfined_r

ps axZ

getent passwd
getent passwd /etc/nsswtich.conf
getent group
getent network
getent services

gpasswd -M <user1>,<user2> <grp_name>       # to add user in group for a password

ausearch -m avc     # selinux module search
```

## Working with SeLinux
Three operational modes:
- Enforcing - Rules are enforced and violations logged
- Permissive - No Rules are enforced, violations logged
- Disbaled - SeLinux not operational and no logging

```bash
# get selinux mode
getenforce

# to get more details about SeLinux
sestatus

# selinux config file
cat /etc/selinux/config

# setting the SeLinux mode
setenforce 0                # disable temporary SeLinux
sestatus
setenforce 1                # enable temporary SeLinux

vim /etc/selinux/config     # change SELINUX=permissive     (to make persistent post reboot)

# to make SeLinux Disabled
vim /etc/selinux/config     # change SELINUX=disabled
```

## Preventing Runtime Changes to Mode
Setting bolean requires reboot of the system to change the SeLinux mode.

```bash
# list all selinux boolean
getsebool -a

# to get more information
semanage boolean --list

semanage boolean --list | grep secure_mode_policyload

# protect runtime changes
setsebool secure_mode_policyload on
setenforce 0                # will fail since we turned on the secure mode
setsebool secure_mode_policyload on -P  # -P to make persistent
```

## SeLinux Kernel Options
- selinux=0 : Disable SeLinux, generally the wrong idea
- enforcing=0 : Set SeLinux into permissive mode, great if SeLinux issues causing authentication issues.
- autorelabel=1 : Causes all files to receive the SeLinux labels that should be assigned to them, again can be good to resolve SeLinux boot and authentication issues.

**Use *chcon* to change a critical files SeLinux label**

```bash
ls -Z /etc/shadow                   # file used for authentication 
chcon -t user_home_t /etc/shadow    # changing the context of shadow file to test login    
ls -Z /etc/shadow

# reboot the server to make the changes persistent

# Now try login with in with same password you will get the permission denied.

# Now to fix the reboot the server and go to grub menu and add the following line, at the end of line ending with `rhgb quiet`
    enforcing=0
    # press ctrl and x to continue
    # now login with root and its password
    # once logged in run the below command to restore the selinux context type.
    >>> restorecon /etc/shadow
    >> ls -Z /etc/shadow
```

# SeLinux Users and controlling user access
```bash
# list selinux user mappings
semanage login -l           

# assigning account to logins
semanage login -m -s "user_u" -r s0 __default__ 

# adding/modifying default assignment
useradd -Z guest_u <username>          # adding user to specific selinux user
# changing user selinux again
semanage login -a -s user_u <username>
```

### SeLinux User Capabilities
| User | X-Windows | su/sudo | Execute /tmp ~ | Networking |
|------|-----|-----|---|---|
|sysadm_u|yes|both|yes|yes|
|staff_u|yes|sudo|yes|yes
|user_u|yes|no|yes|yes|
|guest_u|no|no|no|no|
|xguest_u|yes|no|no|Firefox|

```bash
# current user selinux context
id  -Z

# swtich to root user to check the root selinux context
su - 
id -Z

# list selinux manage users
semanage login -l

# changing for __default__
semanage login -m -s user_u -r s0 __default__
    # to test login with a normal user with sudo previleges
    # For ex: logged with izhar user
    >>> ping <ip>       # this will work
    >>> su -            # switch to root user, giving correct password will give authentication failure, because of above new semanage login policy which is `user_u`
    >>> sudo su -i      # will give another big error, same reason


# show local changes made to selinux users/semanage
semanage login -C -l

# assigning selinux user to another user
semanage login -a -s unconfined_u <username>        # will change the deafault assignment selinux user say (default user was root with unconfined_u, now it will become default with ,<username>)
    # deleting the default assigment for <username> will make it to got back to default user


# Setting for new user
useradd user1
semanage login -C -l

useradd user2 -Z guest_u
semanage login -C -l

# changing password
echo -e "user1:Password1\nuser2:Password2" | chpasswd

# Now login with above users and run few command to see diff.
    >>> ping
    >>> id -Z
```

### SeLinux Lable File System/Process

| User | Role | Type | MLS Level |
|------|------|------|-----------|
|system_u|object_r|httpd_sys_content_t|s0|
|system_u|system_r|httpd_t|s0|

```bash
>>> seinfo              # list all selinux details
yum install policycoreutils policycoreutils-devel setools-console httpd attr

systemctl enable --now httpd
firewall-cmd --add-service=http
firewall-cmd --runtime-to-permanent
pgrep http
ps -Z -p $(pgrep http)
ls -Zd /var/ww/html

getfattr --help
getfattr -n security.selinux /var/www/html  # to remove selinux label 

# list types of selinux
seinfo -t

seinfo -t | grep http

# allow selinux rule details
sesearch --allow  -s httpd_t -t httpd_sys_content_t | less
```

### Configuring Web Server with new Document Root
```bash
vim /etc/httpd/conf/httpd.conf
    # Look for DocumentRoot, add new document root or modify the existing /var/www/html
    DocumentRoot "/repos"

    <Directory "/repos">
        AllowOverride None
        Require all granted
    </Directory>

mkdir -m 2750 /repos                      # 2 - SGID (file create to be owned by group owner only)

ls -ld /repos
chgrp apache /repos
ls -ld /repos
echo "Hello" > /repos/index.html
ls -l /repos/index.html

systemctl restart httpd
curl localhost/index.html
    # gives the 403 Forbidden error
    # You dont have permission to access index.html on this server
    # this is due to selinux

# Configuring Selinux to allow access to custom directories
man semanage-fcontext               # look for examples
semanage-fcontext -a -t httpd_sys_content_t "/repos(/.*)?"  # copy from example and modify with your requirement/directory

ls -Zd /repos

# we made changes to policy but didn't udpated the filesystem policy
restorecon -R -v /repos
ls -Zd /repos

curl localhost/index.html

# Managing port through selinux port
    # Configuring webserver to listen on new port
    vim /etc/httpd/conf/httpd.conf
        # change the Listen from 80 to any port you want
        Listen 1000

systemctl restart httpd
    # might give error for denied

seinfo --portcon 80
semanage port -l | grep http_port_t
man semanage-port 
semanage port -a -t http_port_t -p tcp 1000
semanage port -l -C             # list changes made locally 
firewall-cmd --add-port=1000/tcp --permanent
firewall-cmd --reload

systemctl restart httpd
curl localhost:1000/index.html          # will work

```
