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
id -Z
semanage login -l
semanage user -l
getenforce
setenforce <value>      # value: 0 to disable selinux, 1 to enable

getent passwd
getent passwd /etc/nsswtich.conf
getent group
getent network
getent services

gpasswd -M <user1>,<user2> <grp_name>       # to add user in group for a password

ausearch -m avc     # selinux module search

```
