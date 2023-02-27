# Setting Sticky Bit

- **SUID (o+t):** Ensures users can delete their own files.
- **SGID (g+s):** Ensures new files are created with group ownership of the directory.
- **SUID+SGID:** When set on executables they change the execution context.
- **Capabilities:** Adding a capability can allow the required activity but only the required activity. 

```bash
mkdir -p perms/dir1{1..4}
tree perms

chmod -v 1777 perms/dir1    # 1 signify user will only be able to delete their files not others

chmod -v 2777 perms/dir2    # 2 signify group will only be able to delete their files not others

# put SUID and SGID together
chmod -v 3777 perms/dir3

# Working with Linux Capabilities (Avoid Special Permissions)
getcap <filename>
setcap "capability_name" <filename>
    # For example: if user is not allowed to access ping command
    setcap "cap_net_raw+p" /path_to_ping_command    # /usr/bin/ping
capsh --print               # list all capabilities available in system
```

### Working with Sticky Bit
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
