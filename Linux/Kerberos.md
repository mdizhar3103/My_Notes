## Kerbros Authentication

```bash
>>> yum install krb5-libs krb5-server krb5-workstation pam_krb5 -y
>>> cd /var/kerberos/krb5kdc
>>> vim kadm5.acl
  */admin@IZHAR.COM   *
  
>>> vim kdc.conf 

# client configuration file
>>> vim /etc/krb5.conf
    #Change the following parameters:
    default_realm = IZHAR.COM
    [realms]
    IZHAR.COM = {
    KDC = dnsserver.izhar.com
    Admin_server = dnsserver.izhar.com
    }

    [domain_realm]
    .izhar.com = IZHAR.COM
    izhar.com = IZHAR.COM

# creating realm
>>> kdb5_util create -s -r IZHAR.COM

# starting services
>>> systemctl start krb5kdc
>>> systemctl start kadmin
>>> systemctl enable krb5kdc
>>> systemctl enable kadmin

>>> netstat -ntlp

# Allow incoming TCP connections to ports 88, 464, and 749 and UDP datagrams on UDP port 88, 464, and 749:
>>> firewall-cmd --add-service=kpasswd --permanent 
>>> firewall-cmd --add-service=kerberos --permanent
>>> firewall-cmd --add-port=749/tcp --permanent  
>>> firewall-cmd --reload

# Create a principal for each user who should have the admin instance
>>> kadmin.local
    >> listprincs
    >> addprinc <username>/admin

# Add principals for users and the Kerberos server and cache the key for the server's host principal in /etc/kadm5.keytab by using either kadmin.local or kadmin
>>> kadmin.local -q "addprinc bob"
>>> kadmin.local -q "addprinc -randkey host/krbsvr.mydom.com"
>>> kadmin.local -q "ktadd -k /etc/kadm5.keytab host/krbsvr.mydom.com" 

# Cache the keys that kadmind uses to decrypt administration Kerberos tickets in /etc/kadm5.keytab:
>>> kadmin.local -q "ktadd -k /etc/kadm5.keytab kadmin/admin"
>>> kadmin.local -q "ktadd -k /etc/kadm5.keytab kadmin/changepw" 
```

### Enabling Kerberos Authentication
```bash
# client config
>>> vim /etc/ssh/ssh_config
    # uncomment below parameters and change to yes
    GSSAPIAuthentication yes
    GSSAPIAuthentication yes
    GSSAPIDelegateCredentials yes

>>> systemctl restart sshd
>>> authconfig --enablekrb5 --update

>>> klist
>>> kinit <principle_user_Added>@<DOMAIN.XXX>
>>> klist

# testing the connection
>>> ssh <user_added_in_principle>@<kerberos_server>.domain.com
    # ssh opc@dnsserver.izhar.com
    >>> exit

>>> klist

# removing the tokens
>>> kdestroy
>>> klist

```

### Configuring client
```bash
>>> yum install -y krb5-workstation pam_krb5
>>> vim /etc/krb5.conf
    # copy and paste the server configuration

>>> kadmin
    >> listprincs
    >> addprinc -randkey host/<server2>.izhar.com
    >> listprincs
    >> ktadd host/<server2>.izhar.com
    >> quit

>>> vim /etc/ssh/ssh_config
    # uncomment below parameters and change to yes
    GSSAPIAuthentication yes
    GSSAPIAuthentication yes
    GSSAPIDelegateCredentials yes

>>> authconfig --enablekrb5 --update
>>> systemctl restart sshd
>>> klist
>>> kinit <username>@<domain>
    >> klist

>>> ssh <user_in_principle>@server1.domain.com
```


> [Ref Link](https://docs.oracle.com/en/operating-systems/oracle-linux/7/userauth/userauth-AuthenticationConfiguration.html#ol7-s17-auth)
