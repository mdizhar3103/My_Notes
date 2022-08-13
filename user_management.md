# User Management 
```bash
useradd username
ls -a /etc/skel         # skeleton directory
useradd --defaults

cat /etc/login.defs
passwd username
userdel username        # to delete user and also remove the username group
useradd -s /bin/sh -d /home/new_homepath username   # to provide specific login shell and home directory together

cat /etc/passwd
cat /etc/shadow
cat /etc/group
cat /etc/gshadow
id 
whoami
who
w

# moving home directory to new home directory
usermod -d /new/home/dir_path -m username

# renaming username
usermod -l username new_name_for_user

# changing user login shell
usermod -s /bin/sh username

# locking the user
usermod -L username

# Unlock the user
usermod -U username

# expiring the user after specific date
usermod -e 2021-05-01

# making a script to show lastlogin
vim /etc/profile.d/lastlogin.sh
    echo "Your last login was at: " > $HOME/lastlogin.log
    date >> $HOME/lastlogin.log
logout
# login-again
ls
cat lastlogin.log
```

## User Template Environement
```bash
vim /etc/skel/README
    # Please dont run CPU-intensive processes between 9AM and 6PM
useradd <username>  
ls -al          # you will notice all the files in this location are copied from /etc/skel
```

## Managing User Resource Limit
```bash
# to restrict resources for user, we special file
cat /etc/security/limits.conf
    # <doamin>: username
    # <domain>: @groupname
    # <domain>: * for all everone
    # <type>: hard, soft, -
        # hard: value is not overwritten by regular user, cant go above the specified limit
        # soft: can go above specified limit but if specified with hard that is the last value, need to manually raise the limit if needed by user
        # -: Both hard and soft limit
    # <item>:
        # nproc: maximum processes can open in user sesssion
        # fsize: maximum file size user can create in this session, size is specified in KB
        # cpu: values specifies in minutes
        # maxlogins: maximum login allowed

ulimit -a       # to list limits of current session
ulimit -u  10   # decrease the limit of user process
```
> Note: User can descrease the limit but can't increase


### User Privileges
```bash
groups          # list groups of current user
visudo          # open sudoers file
    # understanding sudoers entry
    # username ALL=(ALL)         ALL
    # %groupname ALL=(ALL)       ALL       # to run as command as non-admin and admin
    # username   ALL=(username1, username2) ALL    # to run sudo command with user only
    # username   ALL=ALL
    # username   ALL=(ALL)  /bin/ls, /bin/cat      # to restrict user to run specific command


# managing PAM 
ls /etc/pam.d/
man pam                 # press 2 times tab to list options available

```

## Advance File management
```bash
echo "Hello from Izhar" > my_file.txt
chown root:opc my_file.txt
ls -l my_file.txt
echo "Adding New Data" >> my_file.txt
cat my_file.txt

# only specific user can access file
setfacl --modify user:<username>:rw my_file.txt
echo "Adding New Data" >> my_file.txt
ls -l my_file.txt           # you will see a + sign in permission this signifies ACL exist 
cat my_file.txt

getfacl my_file.txt         # to see ACL of a file
setfacl --modify user:<username>:--- my_file.txt
setfacl --remove user:<username> my_file.txt

# speficifying ACL on directories
setfacl --recursive --modify user:<username>:rwx dir_name
setfacl --recursive --remove user:<username> dir_name


# change attribute
echo "Old data" > newfile
chattr [option] <filename>
chattr +a newfile               # to append data
echo "Replace data" > newfile   # gives error
echo "Replace data" >> newfile
cat newfile
chattr -a newfile               # remove append option

# freezing the file (making it immutable)
chatta +i newfile               # can't be changed, renamed, deleted, editted and not even root can modify it

lsattr <filename>               # to list Attributes
lsattr newfile
chattr -i newfile
```
