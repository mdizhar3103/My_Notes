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
    # a below similar data will be visible modify accordingly
    dn: uid=izhar,ou=People,dc=izhar,dc=com
    uid: izhar
    cn: izhar
    objectClass: account
    objectClass: posixAccount
    objectClass: top
    objectClass: shadowAccount
    userPassword: {crypt}$6$Pwnn2OZp$Ic5kVTtxftMXVbyepnPMuMmYWWdlpTzFU.H5NFWCQBZlEfqFYMVc9VBqQbZ8iOMdYAFaeggjH3zdH/kydbqsF/
    shadowLastChange: 19377
    shadowMin: 0
    shadowMax: 99999
    shadowWarning: 7
    loginShell: /bin/bash
    uidNumber: 4000
    gidNumber: 4000
    homeDirectory: /home/opc
    gecos: izhar local User
>>> ldapadd -x -W -D "cn=Manager,dc=izhar,dc=com" -f user.ldif

```

### LDAP Client
 
```bash
# adding DNS Server / LDAP server entry
>>> vim /etc/hosts
    <dnserver_ip/ldap_server_ip> dnsserver.izhar.com dnsserver
    clientserver_ip clientserver.izhar.com clientserver
>>> yum install oddjob oddjob-mkhomdir -y
>>> systemctl start oddjobd
>>> systemctl enable oddjobd
>>> systemctl status oddjobd

>>> yum install openldap-clients nss-pam-ldapd -y
>>> authconfig-tui
    # a gui will be open
    # or instead we can user command line for the setup too as below
>>> authconfig --enableldap --ldapserver=dnsserver.izhar.com --ldapbasedn="dc=izhar,dc=com" --enablemkhomedir --update
>>> grep passw /etc/nsswitch.conf
>>> getent passwd
    # you will see the user we created in LDAP (izhar uid=4000)

>>> su - izhar # ldapuser test
    >>> id

# listing group
>>> getent group
>>> grep ldap /etc/nsswitch.conf

# getting LDAP informations
>>> ldapsearch -x -H ldap://dnsserver.izhar.com -b dc=izhar,dc=com

```

### LDAP - olclogLevel

- any: All debugging
- none: No logging
- conns: Log connections
- filter: Search filters
- stats: Conns, Ops, Results

> Note: Sometimes 'stats' are useful, default is none.

```bash
# list current log level
>>> ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config olcLogLevel 
>>> journalctl -f -n 0 -u slapd
>>> ldapsearch -x -LLL -H ldapi:/// -b dc=izhar,dc=com dn  

# modify loglevel
>>> vim loglevel.ldif
dn: cn=config
changetype: modify
replace: olcLogLevel
olcLogLevel: any

>>> ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f loglevel.ldif
```

### LDAP - oldDBIndex
- Help in searching directories effectiviely
- Search based on user UID
- More index creation will cause more memory utilizations

```bash
>>> ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config olcDbIndex

# search in specific index
>>> ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase={1}mdb olcDbIndex

# Default Index List
olcDBIndex: objectClass eq
olcDBIndex: cn,uid eq
olcDBIndex: uidNumber,gidNumber eq
olcDBIndex: member,memberUid eq
    # eq = equality ('cn=admin')
    # approx ('cn~=admyn')
    # sub ('cn=adm*')

# Deleting and Adding olcDbIndex
# ------------------------------

# list all attribute 
>>> ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase={1}mdb 

# list only index attributes
>>> ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase={1}mdb olcDbIndex

##### Note #####
-L  : No comment
-LL : No Summary
-LLL: No Version

>>> vim cnindex.ldif
    dn: olcDatabase={1}mdb,cn=config
    changetype: modify
    delete: olcDbIndex
    olcDbIndex: cn,uid eq

    dn: olcDatabase={1}mdb,cn=config
    changetype: modify
    add: olcDbIndex
    olcDbIndex: uid eq
    olcDbIndex: cn eq,approx

>>> ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f cnindex.ldif
>>> ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase={1}mdb olcDbIndex

# now searching based on the above indexes created
>>> ldapsearch -x -LLL -w -D cn=admin,dc=izhar,dc=com -b dc=izhar,dc=com "(cn~=admnin)"
>>> ldapsearch -x -LLL -w -D cn=admin,dc=izhar,dc=com -b dc=izhar,dc=com "(cn=adm*)"
```

### LDAP - openLDAP users Authentication in Linux
- Can be done by adding objectClasses **posixAccount** and **shadowAccount**.
- If Linux Authentication is not required then not needed.
- If Linux User belong to an LDAP Group then Posix Group must exits

```bash
>>> vim notRequireLinAuth.ldif
    dn: cn=mdizhar,ou-people,dc=izhar,dc=com
    objectClass: inetOrgPerson
    sn: MdIzhar
    givenName: Mohd
    cn: mdizhar
    userPassword: UserPassword1

# Posix Group
>>> vim posixgrp.ldif
    dn: cn=ldapusers,ou=people,dc=izhar,dc=com
    objectClass: posixGroup
    cn: ldapusers
    gidNumber: 4000

# creating a Posix User
>>> vim createPosixUser.ldif
    dn: uid=mdizhar,ou=people,dc=izhar,dc=com
    objectClass: inetOrgPerson
    objectClass: posixAccount
    objectClass: shadowAccount
    uid: mdizhar
    sn: Ahmad
    givenName: Mohd
    cn: mdizhar
    uidNumber: 4002
    gidNumber: 4002
    userPassword: Password#123
    loginShell: /bin/bash
    homeDirectory: /home/mdizhar

# adding attribute that doesn't exist
>>> vim addUserAttributes.ldif
    dn: cn=mdizhar,ou=people,dc=izhar,dc=com
    changetype: modify
    add: mail
    mail: izhar@email.com

# replacing attribute 
>>> vim replaceUserAttributes.ldif
    dn: cn=mdizhar,ou=people,dc=izhar,dc=com
    changetype: modify
    replace: mail
    mail: mdizhar@email.com

# removing attribute 
>>> vim removeUserAttributes.ldif
    dn: cn=mdizhar,ou=people,dc=izhar,dc=com
    changetype: modify
    delete: mail

# deleting a user
>>> ldapdelete -x -D cn=admin,dc=izhar,dc=com -w "cn=mdizhar,ou=people,dc=izhar,dc=com" 
```
