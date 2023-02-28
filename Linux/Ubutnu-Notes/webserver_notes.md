### Configuring Webserver (HTTPS based) on Ubuntu
```bash
hostnamectl
apt update
apt install -y apache2 w3m
    # w3m - command line based webbrowser

systemctl start apache2
systemctl enable apache2
systemctl status apache2

w3m http://localhost

# Configuring SSL
tree /etc/apache2

# enabling a module
a2enmod --help
a2enmod ssl                 # enable SSL module

# list enabled modules
ls -l /etc/apache2/mods-enabled

# creating a self signed certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/mi_pvt.key -out /etc/ssl/certs/mi_pub.crt 
    # its not node (nodes), its (no des, means we are not putting a password)

    # Enter values that prompted for, but dont forget common name (hostname, FQDN, etc)

ls /etc/ssl/private
ls /etc/ssl/certs

vim /etc/apache2/sites-available/default-ssl.conf
    # add the following lines under <VirtualHost _default_:443>
    ServerName <hostname, or FQDN >
    SSLCertificateFile  /etc/ssl/certs/<above_filename>.crt
    SSLCertificateKeyFile  /etc/ssl/provate/<above_filename>.key

# enable the site (since we made changes in original file)
a2ensite default-ssl                # this will create a symbolic link

systemctl restart apache2
netstat -tnlp

w3m https://<hostname_or_FQDN>:443
    # accept certificate type yes 2 times


# Redirecting HTTP to HTTPS
apachectl configtest
vim /etc/apache2/sites-available/000-default.conf
    # add/modify the following lines under <VirtualHost *:80>
    ServerName <hostname, or FQDN>
    Redirect / https://<hostname_or_FQDN>/

systemctl restart apache2

```

### View Certificate information from commandline
```bash
openssl s_client -connect FQDN:443
    # example: openssl s_client -connect izhar.com:443
    # press enter to to get more details on prompt

openssl s_client -connect FQDN:443 -showcerts
openssl s_client -connect FQDN:443 -showcerts | less
```

### comparing the password with openssl
```bash
getent /etc/shadow <username>
awk -F$ '/<username>/ {print "algorithm: " $2 "\nsalt: " $3 "\nPassword: " $4}' /etc/shadow

openssl passwd -<algorithm_number> -salt <saltvalue>
# For example
openssl passwd -6 -salt MADUJB/IUBD/ <your_password>
```
