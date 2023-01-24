# More Linux Commands 

**Linux commands**
```bash
stat <file_name>
```
**Hard Links (consume same space of origial file)**
```bash
cp -r <source> <destination>
 ```
**Hard links (to avoid extra space to be used in system)**
```bash
ln <path_to_target_file> <path_to_link_file>
ln <source> <destination>
stat <file_name>    # to check links
```
> Note: Only hardlinks for files not for folders

**Soft Links (symbolic links)**

ln -s <path_to_target_file> <path_to_link_file>
```bash
ls -l 
readlink <symbolic_link_filename>
```
> Note: Soft link, files to files, folder to folder, different filesystem as well

# Changing permissions
```bash
chmod u+w  <file_name>
    # for user: u
    # for gorup: g
    # for others: o
    # read, write, execute: rwx
    # To add permission use +, to remove use -, for exact use =

chmod u+rw,g=r,o= <file_name>       # chaining permissions

# Octal Permissions
chmod 664 <file_name>
    # read - 4
    # write - 2
    # execute - 1
```

# SUID, SGID, Sticky Bit
```bash
touch suidfile
chmod 4664 suidfile
ls -l
    # -rwSrw-r-- (assume the output)
    # SUID value: 4
    # S: signifies SUID no execution permission, and to execute file as that user only
    # s: permission enable for execution

touch sgidfile
chmod 2664 sgidfile
ls -l
    # -rw-rwSr--
    # S: no execute permission

chmod 2674 sgidfile
    # -rw-rwsr--
    # s: execute permission

# Finding SUID, SGID file
find . -perm /4000  # SUID
find . -perm /2000  # SGID

# Give both permission
chmod 6664 bothfile
ls -l
    # -rwSrwSr--
find . -perm /6000      # will list either SUID, SGID or both

# Sticky Bit - sets on directory, only user owner can remove 
mkdir stickydir  
ls -l
chmod +t stickydir
or
chmod 1777 stickydir
ls -ld
chmod 1666 stickydir
ls -ld
    # t: execute permission enabled
    # T: execute permission not enabled
```

# sed command
```bash
sed 's/<word_to_replace>/<new_word_that_will_replace>/g' <file_name>
    # g: to replace all (global)
sed 's/<word_to_replace>/<new_word_that_will_replace>'
    # only first occurrence are change not all

sed -i 's/<word_to_replace>/<new_word_that_will_replace>/g' --in-place
    # to modify the file directly
```

# Backup files to remote servers
```bash
rsync -a <folder_name> username@ip_address:/path_to_remote_system
rsync -a username@ip_address:/path_to_remote_system <folder_name>   # from remote to local
```

# IO Redirection
```bash
1>output.txt      # redirect only stdout to the file
2>&1              # redirect stderr to stdout

grep -r '^The' /etc/ 1>output.txt 2>&1      # redirect output to file and error to stdout
grep -r '^The' /etc/ 2>&1 1>output.txt      # first redirect error to stdout and output to file
```

# Boot Mode
```bash
systemctl get-default
systemctl set-default multi-user.target     # cli mode
reboot
systemctl isolate graphical.target          # switching to graphical mode
systemctl isolate emergency.target          # going to emergency mode, to load few program used in case of debugging and system is loaded in read-only
systemctl isolate rescue.target             # with root privileges prompt is given
```
> Note: to use emergency and rescue mode, root must have password, if doesn't have you can't use these options

# System Startup Processes
```bash
systemctl cat firewalld.service
systemctl edit --full firewalld.service     # to modify the service file
systemctl revert firewalld.service          # to restore back to default 
systemctl stop firewalld.service
systemctl start firewalld.service
systemctl status firewalld.service
systemctl restart firewalld.service
systemctl reload firewalld.service
systemctl reload-or-restart firewalld.service
systemctl enable firewalld.service
systemctl enable firewalld.service --now
systemctl disable firewalld.service
systemctl disable firewalld.service --now

systemctl mask firewalld.service            # to prevent start and enabling
systemctl unmask firewalld.service

systemctl list-units --type service --all   # list all service
```

# Process Commands
```bash
ps 1
ps u 1
ps -U username         # get all process of specific user 
ps u -U username
ps l                    # to get nice values

# changing nice value
nice -n 11 bash
ps lax
ps fax                  # tree view of processes

pmap <PID>
pwdx --help

sar -u
sar -b
sar -n
sar -q 

renice 7 <PID>          # changing nice value

lsof -p <PID>           # list open files and directories of a process
lsof -p 1               # list all open files and directories of process 1
lsof <filename>

journalctl -f           # follow
journalctl -p err       # show error logs
- info
- warning
- err
- crit
- debug
- notice
- emerg
- alert

journalctl -S 05:00               # show logs from time
journalctl -S 05:00 -U 14:00      # show logs between time
journalctl -b 0                   # logs of current boot
journalctl -b -1                  # logs of previoud boot

last
lastlog
```

# Scheduling Jobs
```bash
cat /etc/crontab
cat /var/log/cron
crontab -e                      # edit cron job
crontab -l                      # list cron jobs
crontab -e -u <username>        # edit cron job of specific user
crontab -r                      # to remove cron jobs
crontab -r -u <username>        # to remove cron jobs of specific user

# scheduling jobs in specific directory
ls /etc/cron.daily/
ls /etc/cron.hourly/
ls /etc/cron.monthly/
ls /etc/cron.weekly/
```

# Resource Management
```bash
df -h
du -sh <folder_name>
free -h
uptime
    # load average: last_min avg, last_5_min avg, last_15_min_avg
lscpu
lspci
xfs_repair -v /dev/partition_name
fsck.ext4 -vfp /dev/partition_name
    # f: force
    # v: verbose
    # p: prompt for yes or no

systemctl list-dependencies
```

# Kernel Parameters
```bash
sysctl -a
sysctl -w option_from_above=1       # to enable use 1 or 0 for disable
```

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
