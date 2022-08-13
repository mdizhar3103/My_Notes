### Installing and Configuring Webserver
```bash
yum install httpd
systemctl enable httpd --now
systemctl status httpd
firewall-cmd --add-service=http 
firewall-cmd --add-service=https
firewall-cmd --reload

yum install mod_ssl
systemctl restart httpd
man httpd.conf

# primary config file
vim /etc/httpd/conf/httpd.conf

# creating own conf-file
vim /etc/httpd/conf/my-website.conf
    <VirtualHost *:80>
        ServerName   server_ip or server_full_domain_name
        DocumentRoot /var/www/mywebsite
    </VirtualHost>

    <VirtualHost *:80>
        ServerName   server_ip2 or server2_full_domain_name
        DocumentRoot /var/www/secondwebsite
    </VirtualHost>

    # we can replace * with server_ip also

apachectl configtest
systemctl restart httpd.service

# Configuring ssl connection
vim /etc/httpd/conf.d/ssl.conf
    <VirtualHost *:443>
        ServerName   server_ip or server_full_domain_name
        SSLEngine on
        SSLCertificateFile "path_to_cert_file.cert"
        SSLCertificateKeyFile "path_to_key_file.key"
    </VirtualHost>

vim /etc/httpd/conf.modules.d/00-mpm.conf
    # uncomment the LoadModule mpm_prefork_module

vim /etc/httpd/conf.modules.d/00-optional.conf
    # uncomment the modules that you need
vim /etc/httpd/conf.modules.d/00-base.conf
    # comment out the things we dont need

```

### Configuring Webserver Logs
```bash
vim /etc/httpd/conf/httpd.conf
    # Look for ErrorLog
    # Look for LogLevel
    # Look for Log format

# specifying LogFile for specific websites
vim /etc/httpd/conf/my-website.conf
    <VirtualHost *:80>
        ServerName   server_ip or server_full_domain_name
        DocumentRoot /var/www/mywebsite
        CustomLog /var/log/httpd/mywebsite_access.log combined
        ErrorLog /var/log/httpd/mywebsite_error.log
    </VirtualHost>

systemctl restart httpd.service
```

### Restrciting Access to a webpage
```bash
vim /etc/httpd/conf/httpd.conf
    <Directory "/var/www/html">
        Options Indexes FollowSymlinks
        # Indexes allows to view directory html data in browser like ftp does
        # If we remove the Indexes options, and browse the url we will get `forbidden Error`
        AllowOverride All
        Require all granted     
        # allow every user to access the website 
        # Require all denied
        # Require ip <ip_1> <ip_2> <cidr_block_also_accepted>
    </Directory>

    # Look for Files
    <Files ".ht*">
        Require all denied
    </Files>

    # Defining own file 
    <Files ".txt*">
        Require all denied
    </Files>

# Generating password file for web access
htpasswd -C /etc/httpd/passwords <username>

cat /etc/httpd/passwords 

# to add the new user in same file
htpasswd /etc/httpd/passwords <username>

# to delete password entry for specific user
htpasswd -D /etc/httpd/passwords <username>

vim /etc/httpd/conf/httpd.conf
    <Directory "/var/www/html">
        AuthType Basic
        AuthBasicProvider file
        AuthName "Restricted Page"
        AuthUserFile /etc/httpd/passwords
        Require valid-user
    </Directory>

systemctl restart httpd.service

```
