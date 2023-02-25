### Configuring SSH server and client
- ssh_config is client file
- sshd_config is server file

```bash
# client configuration
cat /etc/ssh/ssh_config         # system wide config
cat ~/.ssh/config               # individual user config and key files (optional)
man ssh_config

# Server config
cat /etc/ssh/sshd_config       # system wide config
man sshd_config

ps -aux | grep key
ssh -v username@server_ip
ssh -l <login_name> server_ip -n "ps -aux --no-headers | sort -nrk 3 | head"
```

```bash
man sshd_config
    # Look for AddressFamily

vim /etc/ssh/sshd_config 
    # Uncomment: AddressFamily any (this allow all ip to connect)
    AddressFamily inet  # to listen on particular net card
    ListenAddress <server_ip>  

    # To run graphical system
    X11Forwarding yes

    # Disabling password Authentication
    PasswordAuthentication no

    # allowing specific user for Password Authentication
    Match User <username>
        PasswordAuthentication yes
    
systemctl restart sshd.service

man ssh_config

# configuring config in .ssh
vim ~/.ssh/config
    Host <hostname>
      HostName <serverIP>
      Port 22
      User <username>
    # Can add more options as
    # StrictHostKeyChecking no
    # PasswordAuthentication no
    # IdentityFile ~/.ssh/id_rsa

chmod 600 ~/.ssh/config
ssh <hostname_in_config_file>
```

**SSH Tunnelling**
```bash
ssh -L <local_port_number>:<remote_server_name/ip>:<port_number_to_access_on_remote_server> remote_server_name_or_ip
# ex: ssh -L 8080:my_server.com:80 my_server.com
# this will also give terminal to avoid getting terminal use -N also 
# ex: ssh -L 8080:my_server.com:80 my_server.com -N

vim /etc/ssh/sshd_config
    # Look for: GatewayPorts 
```

**ssh -X**
```bash
ssh -X username@server_name_or_ip
cat ~/.Xauthority
yum install xorg-x11-apps -y
xeyes       # to see open GUI

# in oracle linux
yum config-manager --enable ol8_codeready_builder   # for linux-8
```
