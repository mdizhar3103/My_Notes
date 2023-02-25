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
semanage user -l

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
