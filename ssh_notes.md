### Configuring SSH server and client
- ssh_config is client file
- sshd_config is server file

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
