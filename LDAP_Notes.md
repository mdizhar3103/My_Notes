## Configuring LDAP
- LDAP is the Light Weight Directory Access Protocol
- LDAP uses Port 389 for unencrypted connections
- Uses Port 636 for TLS based connections
- Used to provide a directory for authentication
    - Linux Authentication
    - DNS Entry Storage
    - Corporate White Pages
- ***Entries in LDAP are FQDN and are delimited by comma***

```bash
# Configuring Naming and Time
>>> hostnamectl
>>> hostnamectl -i
>>> vim sed.txt
    # Add the following Content
    # /^127.0.1.1/d
    # /^127.0.0.1/a <server_ip> server1.fqdn.com server_name1
    # /^127.0.0.1/a <server_ip> server2.fqdn.com server_name2
>>> sed -i -f sed.txt /etc/hosts
# Or directly add the entry to hostfiles

>>> head -n3 /etc/hosts

# Configuring and Installing Chrony
>>> yum install chrony -y
>>> systemctl enable chronyd --now
>>> systemctl status chronyd

>>> yum install -y openldap openldap-clients openldap-servers migrationtools
>>> firewall-cmd --permanent --add-service=ldap
>>> firewall-cmd --reload

# Configuring LDAP
>>> cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
>>> ls -l /var/lib/ldap
>>> slaptest
>>> ls -l /var/lib/ldap
>>> chown ldap.ldap /var/lib/ldap/*
>>> systemctl enable slapd --now
>>> systemctl status slapd 
>>> netstat -tnl
>>> netstat -lt

# schema describes what can be created in the directory
>>> cd /etc/openldap/schema
>>> ldapadd -Y EXTERNAL -H ldapi:/// -D "cn=config" -f cosine.ldif
    # -Y: use external authentication
    # -H: hostname as localhost
    # -D: use config
    # -f: read config files     (cosine.ldif defines the schema we are creating)

>>> ldapadd -Y EXTERNAL -H ldapi:/// -D "cn=config" -f nis.ldif

# using encrypted password for the directory administrator
>>> cd ~
>>> slappasswd -s Password@123 -n > rootpwd
>>> cat rootpwd
    {SSHA}RHZkKk025sCkaJ3IX6o/P8rlGKFnxiWM

# creating basic config of the server
>>> vi config.ldif
    # add the following data
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=izhar,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=Manager,dc=izhar,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: <output_from_the_slappasswd_command {SSHA}RHZk>

dn: cn=config
changetype: modify
replace: olcLogLevel
olcLogLevel: 0

dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=Manager,dc=izhar,dc=com" read by * none

>>> ldapmodify -Y EXTERNAL -H ldapi:/// -f config.ldif
```

### Creating the Directory Structure
```bash
>>> vim structure.ldif
    # add the following entries
dn: dc=izhar,dc=com
dc: izhar
objectClass: top
objectClass: domain

dn: ou=people,dc=izhar,dc=com
ou: people
objectClass: top
objectClass: organizationalUnit

dn: ou=group,dc=izhar,dc=com
ou: group
objectClass: top
objectClass: organizationalUnit

>>> ldapadd -x -W -D "cn=Manager,dc=izhar,dc=com" -f structure.ldif
    # -W: prompt for password (enter the password that we added in slappaswd)
    # -w: pass the password
>>> ldapsearch -x -W -D "cn=Manager,dc=izhar,dc=com" -b "dc=izhar,dc=com" -s sub "(objectclass=organizationalUnit)"
    # -b: base search 
```

### Create Groups and Users
```bash
>>> vim group.ldif
    # add the following data
dn: cn=ldapusers,ou=group,dc=izhar,dc=com
objectClass: posixGroup
cn: ldapusers
gidNumber: 4000

>>> ldapadd -x -W -D "cn=Manager,dc=izhar,dc=com" -f group.ldif

>>> cd /usr/share/migrationtools/
>>> vi migrate_common.ph
    # search for DEFAULT_MAIL_DOMAIN
    # change mail domain and default base also

>>> grep opc /etc/passwd > passwd
>>> cat passwd
>>> /usr/share/migrationtools/migrate_passwd.pl passwd user.ldif
>>> vi user.ldif
    # change the user, uid, cn to specific username, uidNumber, gidNumber (make as we added group.ldif), homeDirectory
>>> ldapadd -x -W -D "cn=Manager,dc=izhar,dc=com" -f user.ldif

```
