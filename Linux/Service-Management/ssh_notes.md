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

#### Implementing SSH CA authority
```bash
# create a centralize known hosts file.
touch /etc/ssh/ssh_known_hosts

ssh-keygen -f <filename>_ca
    # provide a passphrase if you need
ls
cat <filename>_ca.pub
cat <filename>_ca.pub   >> /etc/ssh/ssh_known_hosts     # centralized file for known hosts

vim /etc/ssh/ssh_known_hosts
    # append the following line before the ssh-rsa to make it as certificate authority config
    @cert-authority  192.168.43.* 
    @cert-authority  <dns_domain>           # based on your requirement
    # For example: @cert-authority  192.168.43.*  ssh-rsa.......

# signing public key

ssh-keygen -s <filename>_ca -I izhar -h -n 192.168.43.72 -V +52w <filename>.pub
    # <filename>_ca here it is private key
    # -s signing request
    # -I identity name we need to assign, for login activity
    # -h host key
    # -n for name
    # -V for validity
    # <filename>.pub public key you need to sign

scp <filename>-cert.pub    username@ip:/tmp/ssh_host_rsa_key-cert.pub
    # login to server where the file is copied
    >>> cp /tmp/ssh_host_rsa_key-cert.pub /etc/ssh
    >>> echo "HostCertificate /etc/ssh/ssh_host_rsa_key-cert.pub" >> /etc/ssh/sshd_config
    >>> systemctl restart sshd

# For example:
# mkdir Izhar
# cd Izhar/
# ssh-keygen.exe -f izhar_ca
# ls
# cp izhar_ca.pub my_pub_key.pub
# ssh-keygen.exe -s izhar_ca -I mdizhar -h -n 192.168.1.5 -V +52w my_pub_key.pub

```
