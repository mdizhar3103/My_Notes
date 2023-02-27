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

**Soft Links (symbolic links)**
```bash
ln -s <path_to_target_file> <path_to_link_file>
ls -l 
readlink <symbolic_link_filename>
```

> Note: Only hardlinks for files not for folders

> Soft link, files to files, folder to folder, different filesystem as well


### Backup files to remote servers
```bash
rsync -a <folder_name> username@ip_address:/path_to_remote_system
rsync -a username@ip_address:/path_to_remote_system <folder_name>   # from remote to local
```

### IO Redirection
```bash
1>output.txt      # redirect only stdout to the file
2>&1              # redirect stderr to stdout

grep -r '^The' /etc/ 1>output.txt 2>&1      # redirect output to file and error to stdout
grep -r '^The' /etc/ 2>&1 1>output.txt      # first redirect error to stdout and output to file
```

### Changing permissions
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

# Advance File management
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
chattr +i newfile               # can't be changed, renamed, deleted, editted and not even root can modify it

lsattr <filename>               # to list Attributes
lsattr newfile
chattr -i newfile
```
