### Implementing OpenLDAP and SSSD
SSSD: Provides access to identity management solutions and requires secure communications.

```bash
# Configuring Ubuntu machine as LDAP server
# LDAP Client - SSSD

# ensure domain name is set
hostnamectl set-hostname ubuntu.example.com
vim /etc/hosts 
    Ip.x.x.x ubuntu.example.com ubuntu

apt update
apt install slpad ldap-utils -y
    # will prompt to enter password for LDAP admin

# in ubuntu services are started by default
systemctl status slapd

netstat -ntlp           # look for port 389
ldapwhoami -x           # check the default user

ldapwhoami -Q -Y EXTERNAL -H ldapi:///
    # -Q: dont display more info
    # -Y: use external authentication with current logged in user
    # -H: URI to connect

ldapsearch -x -b dc=example,dc=com cn
    # -x: connect with anonymous user
    # -b: base
    # cn: common name

dpkg-reconfigure slapd          # to reconfigure

```

### LDIF Files
**LDIF:** Lightweight Directory Interchange Format

```bash
# creating a user
vim user.ldif
    # add the following content
    dn: ou=People,dc=example,dc=com
    objectClass: organizationalUnit
    ou: People

    dn: ou=Groups,dc=example,dc=com
    objectClass: organizationalUnit
    ou: Groups

    dn: cn=staff,ou=Groups,dc=example,dc=com
    objectClass: posixGroup
    cn: staff
    gidNumber: 5000

    dn: uid=john,ou=People,dc=example,dc=com
    objectClass: inetOrgPerson
    objectClass: posixAccount
    objectClass: shadowAccount
    uid: izhar
    sn: Ahmad
    givenName: Izhar
    cn: Izhar Ahmad
    displayName: John Doe
    uidNumber: 10000
    gidNumber: 5000
    userPassword: <give_your_passowrd>
    gecos: Izhar Ahmad              
    loginShell: /bin/bash
    homeDirectroy: /home/izhar

# uploading the file in LDAP
ldapadd -x -D cn=admin,dc=example,dc=com -W -f user.ldif
    # -D: default admin user    (otherwise operation can't be performed)
    # -W: prompt for admin password

ldapsearch -x -b dc=example,dc=com '(objectClass=inetorgPerson)' cn
ldapsearch -x -b dc=example,dc=com '(objectClass=posixGroup)' 

# setting the ldap user password
ldappasswd -x -D cn=admin,dc=example,dc=com -S uid=izhar,ou=People,dc=example,dc=com -W
    # -S: Set password details
    # first it will prompt for to enter user password then LDAP admin password
```

#### Configuring TLS in LDAP
```bash
# make sure your certificate already exist
cp /etc/ssl/certs/{my_ca,ubuntu}.crt /etc/ldap

cp /etc/ssl/private/ubuntu.key /etc/ldap
chgrp openldap /etc/ldap ubuntu.key
chmod =v 640 /etc/ldap ubuntu.key

vim tls.ldif
    # file content
    # - is used to add multiple attributes
    dn: cn=config
    add: olcTLSCACertificateFile
    olcTLSCACertificateFile: /etc/ldap/my_ca.crt
    -
    add: olcTLSCertificateFile
    olcTLSCertificateFile: /etc/ldap/ubuntu.crt
    -
    add: olcTLSCertificateKeyFile
    olcTLSCertificateKeyFile: /etc/ldap/ubuntu.key

ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f tls.ldif

ldapwhoami -x -ZZ -H ldap://ubuntu/         # give error since tls is enforced
    # -Z: use StartTLS
    # -ZZ: force StartTLS

```

### Installing SSSD on Ubuntu
```bash
apt install -y sssd-ldap
pam-auth-update --enable mkhomedir

cp /usr/lib/x86_64-linux-gnu/sssd/conf/sssd.conf /etc/sssd/sssd.conf
chmod -v 600 /etc/sssd/sssd.conf 
vim /etc/sssd/sssd.conf
    # add/modify the following content
    [sssd]
    domains = shadowutils,example.com

    [nss]

    [pam]

    [domain/example.com]
    id_provider = ldap
    auth_provider = ldap
    ldap_uri = ldap://ubuntu
    cache_credentials = True
    ldap_search_base = dc=example,dc=com
    ldap_id_use_start_tls = true

    # this is default values
    [domain/shadowutils]
    id_provider = files
    auth_provider = proxy
    proxy_pam_target = sssd-shadowutils

    proxy_fast_alias = True

systemctl restart sssd

# login to ldap create user
su - izhar
    # su - <ldap_user>
pwd
id
```

### Configuring SSSD on client
```bash
# on redhat/centos/oracle linux based
yum install sssd sssd-ldap oddjob oddjob-mkhomedir

# make user ldap server entry exist 
vim /etc/hosts
    # add ldap server entry

authselect select sssd with-mkhomedir
    # use incase of error: authselect select sssd with-mkhomedir --force

systemctl enable oddjob --now

vim /etc/sssd/sssd.conf
    # add the following content
    [sssd]
    domains = example.com
    services = nss, pam

    [nss]

    [pam]

    [domain/example.com]
    id_provider = ldap
    auth_provider = ldap
    ldap_uri = ldap://<ldap_server_name_or_ip>
    cache_credentials = True
    ldap_search_base = dc=example,dc=com
    ldap_id_use_start_tls = true

chmod -v 600 /etc/sssd/sssd.conf 
systemctl enable sssd --now

# testing the ldap user login on client
su - izhar
    # su - <ldap_user>
pwd
id
```
