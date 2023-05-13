### Compute

**Creating Custom Images:**
- Go to existing boot volume, click on more action and select create custom image.

**Using Custom Image in another Region:**
- Go to custom image under compute section, click on 3 dot icon and you can export, to do so you need a bucket where you can export

**Importing on-premise image:**
- First you need to upload that image in a bucket then.
- Go to custom images under compute section, click on import image.

**DR solution:**
- Take backup of boot volume and restore it.
- Create custom image and upload in a bucket from bucket use pre-authenticated request and in another region use this URL to import in custom image section
- Boot Volume Replication is another option which data is constantly replicated

### Networking
**Load Balancer important points:**
- Load Balancing policy - tells the load balancer how to distribute incoming traffic to backend servers
- Backend Server - application server responsible for generating content in reply to the incoming TCP or HTTP traffic
- Health Checks - a test to confirm the availability of backend servers
- Backend Set - List of backend servers, a load balancing policy, and a health check policy 
- Listener - entity that checks for incoming traffic on the load balancer's IP address

# Block Storage In OCI

Volume Groups
-------------

- Just contains metadata of collection of boot/block volumes
- Can be used to create clone of grouped boot/block volumes in single click
- Can be used to create a backups of grouped boot/block volumes co-ordinated lifecylcle (Means: 10 boot/block volumes are attached to single volumes group and when backup policy is initiated all boot/block volumes will backup at same time.)
- Feature with no additional charge.
- Policy based backups also available

Cross-Region Replica of Block Volumes
-------------------------------------

- Once cross Region replica is on for particular block storage, **size of the block storage can't be increased at current time**
- To increase the size of replicated block storage disable the cross-region replication option and then increase the size. **Note: disabling the cross region feature will delete the replica in another region, enable it again to start new replica.**
- Incur Additional cost

Block Storage Access Type
--------------------------

- **Shared Multi-Attach**
    - Ability to attach same volume to multiple compute instances on the same availability domain.
    - Block volume service doesn't provide con-current writes.
    - Users must install and configure clustered file system to avoid data corruption (ocsf2)

    - Go to instances, click on attached volumes, you will get the option of Access- Read/Write, Read/Write Shareable, Read Only Shareable

    **Note: if selecting read/write shareable, it is your responsibility to configure ocfs2 so that locking can be done during write operation at volume.**


Block Volume Performance
------------------------
- There are 3 aspects to the block vol performance.
    - **IOPS**: Number of Input operation can be done
    - **Throughput**: How much data can be send
    - **Latency**: Delay in getting data
- Throughput performance on VM instance is dependent on network bandwidth.
- IOPS performance is dependent of the instance type or shape, so applicable to all VM/BM shapes, for iscsi attached volumes.
- ISCSI gives performance while paravirtualized is simple service.

File Storage Service
--------------------
- NFS storage implementation
- AD specific services
- Access from any AD (although it is AD specific service)
- Access from outside the VCN with below option
    - Private connectivity with VCN peering
    - On-premise connectivity with VPN or fast connect
- Subnet security list must allow traffic to access from particular location/server.
- FSS snapshot are available in mount points under mountpoint/.snapshot
    - Snapshot consume no additional usage in the file system
    - Snapshot data usage is metered against differentiated data only.
    - File system reports metered utilization
    - Space is allocated for metadata
- **Use df or du command** to view usage information.


# Troubleshooting and Disaster Recovery

SSH Connection
- Client Side:
    - Key Error
    - Persmission Error
    - Firewall Issue
    - Security List Issue
- Server Side:
    - Public Key should be available in .ssh under user home directory with read all permission (chmod +r public_key)


Load Balancer Health Status
- Health Check is misconfigured
- Listener is misconfigured
- Security rule is misconfigured
- Backends unhealthy
- Compute instances have misconfigured route tables
- NSG or security list block traffic


Block Storage Multi-Attach
- Multi Attach is to allow multiple instances to share 1 data volume (boot volume is not shareable)
- User can config a block volume as shareable during attach call
- Common Error messages:
    - To attach a volume to more than one instance, all volume attachment must be configured as shareable.
    - Read-Write attachment type conflict.
    - Volume shared by too many instances.
    - 8 instances can attach a single block volume as of now


# Identity & Access Management
- To give fine-grained access control
- Authentication (AuthN - Who are you?)
- Authorization (AuthZ - What permission do you have?)
- **Identity Domain is a way to segeregate the user**
- Identity is Role Based Access Control, For example: a set of user grouped in a particular group and that group is then assigned a policy or set of policies which decides where to grant access and to which the resources.
- **Identity Domains represents user configurations and security settings**
    - Act As a container to manage users, roles, Federation, SSO, MFA, etc.
    - Usecase: We have Dev env and Prod env, to keep the identity seperate, this  concept can be used.
    - Identity Domain types:
        - Oracle Apps Premium (helps in manage Oracle Workload types application like JDE, peoplesoft, etc.)
        - Free
        - Premium (Helps in Manage Oracle Apps as well as Non-oracle Apps)
        - External User (helps manage consumer and non-emplyee use case)
    
OCI Federation
--------------

- User Login to OCI, request goes to Service Provider (SP) and if federation is setup request is redirected to Identity provider for authentication, once validated Identity Provider(IdP) return assertion to user login and IdP Authentication Assertion used to login in Service Provider.
- OCI provides federation with **Oracle IDCS, Microsoft Active Directory, and identity provider that supports the Security Assertion Markup Language (SAML) 2.0 protocol**

Security Services in OCI
------------------------

- Infrastructure Protection
    - DDoS Protection
    - Network Security Controls
    - Virtual Firewalls (DRG)
    - Filter malicious web traffic

- IAM
    - AuthN and AuthZ
    - MFA
    - Federation
    - Audit

- OS and workload Protection
    - Managed Bastion
    - OSMS
    - Secured Boot
    - Dedicated Host

- Data Protection
    - Certifcates
    - Data Safe
    - Vault Secret Management
    - Vault Key Management

- Detection and Remediation
    - Cloud Guard
        - Problems Can be:
            1. Remediated: Fixed by using Cloud Guard Responder.
            2. Resolved: Fixed by other process.
            3. Dismissed: Ignored/Closed. 
    - Security Zones
        - An association between a compartment and security zone recipe (a recipe is collection of policies), validate the operation according to policy, and deny access if violated.
        - Security Zone Policies are for example, Deny public access, Restrict Resource Movement/Association, Use Oracle Approved Configuration, Require Encryption, etc.
    - Security Advisor
    - VSS

## Encryption in OCI
- **Encryption at rest:** The data is encrypted with key for example harddisk, storage, etc. so if attackers get the physical disk they retrieve data.
- **Encryption in-transit:** For example HTTPS, encryption between client and server.

***Types of Encryption:***
- **Symmetric Encryption:** A single Key is used for Encryption/Decryption. Drawback is mulitple users have access to same key, one using for encryption and other for decryption. (AES Based keys)
- **Asymmetric Encryption:** It is where different keys are used for encryption and decryption. (Public Key for encryption and Private Key for decryption. RSA based keys)
- **ECDSA Keys**, can be used for digital signing, not for encryption and decryption

### Master and Data Encryption Keys
- **Master encryption keys (MEK)**
    - Can create or import MEKs into Vault
    - MEKs are used to generate data encryption keys
    - MEK are always created in vault.
    - Protection mode indicates how MEK persists and where cryptographic operations are performed.
    - __Protection Modes:__
        1. HSM - stored in an HSM, can't be exported from HSM, all cryptographic operation happen inside HSM.
        2. Software - Stored on server, can be exported to perform cryptograhic operations, software protected at rest, encrypted by a root key on HSM.

- **Data encryption keys (DEK)**
    - Generated by the master encryption key, used to encrypt data.
    - DEK is encrypted with MEK, known as envelope encryption.
    - OCI services dont have access to plaintest DEK.


> Note: When we create a vault a wrapping keys is included vy default, this key can't deleted, created, or rotated. Use the **public wrapping- when you need to wrap key material for import into the vault service.**


## WAF
WAF is a specific form of application firewall that filters, monitors, and blocks HTTP/S traffic to and from web traffic.
- Inspects HTTP/S traffic and can prevent attacks exploiting a web application's know vulnerabilities like **SQL Injection, Cross-site scripting (XSS)**
- Protects from DDoS Attacks
- Typical responses from WAF will be to allow the request to pass through it audit loggin the request, or blocking the request by respinding with an error page.
- OCI WAF is a cloud based, Payment Card Industry PCI-compliant, global security service.
- Use Cases:
    - Protects from internet facing endpoints from cyber attacks and malicious actors.
    - XSS
    - Bot management- - dynamically block bad bots
    - Protects againts layer 7 DDoS attacks
    - Aggregated threat intelligence from multiple sources including Webroot BrightCloud.
- **OWASP Rules in OCI WAF**
    - OCI WAF uses OWASP ModSecurity Core Rules Set to protect against most CVE.
    - OCI WAF comes preconfigured with protection against most important threat on the internet as defined by OWASP Top 10
        - A1: Injections (SQL, LDAP, OS, etc)
        - A2: Broken Authentication and Session Management
        - A3: XSS
        - A4: Insecure Direct Object Reference
        - A6: Sensitive Data Exposure
        - A7: Missing Function-level Access Control

- **WAF Service Components**
    - WAF Policy 
    - Origin is an endpoint (IP address of application)
        - Can be Load balancer public IP
        - Mulitple origins can be defined, but only a single origin can be active at WAF.
        - Can set HTTP headers for outbound traffic
    - Access Conrtols
        - Whitelist IP Address
        - Criteria type like URL, IP, Country, Region, etc.
    - Caching Rules
    - Protection Rules (allow or block traffic)
    - Bot Management (allow detecting bot traffic to your web applications and either allow or block traffic)

> Note: WAF works at port 80/443



**Bastion Jump Host Tunneling Command**
```bash
ssh -i /home/user/pvt.key -o ProxyCommand="ssh  -W %h:%p -i /home/user/pvt.key opc@<bastion_ip" opc@<instance_internal_ip>
```

KVM in OEL-8 (Kernel Based VM)
------------
```bash
# Detect hardware
grep -e 'vmx' /proc/cpuinfo         # for intel based
grep -e 'svm' /proc/cpuinfo         # for AMD based
grep -e 'vendor id' /proc/cpuinfo   # cpu type
lsmod | grep kvm 
# install virtualization software
yum install virt 
yum info virt 
yum install virt-install virt-viewer    # installing other dependencies
virt-host-validate                      # verfiying virtualization (if any check fail follow instruction to correct it)
systemctl start libvirtd
systemctl enable libvirtd
systemctl status libvirtd
```
