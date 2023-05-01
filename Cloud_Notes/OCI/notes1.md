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
