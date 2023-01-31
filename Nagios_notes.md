### Working with Nagios

| [Ref Link](https://support.nagios.com/kb/article/nagios-core-installing-nagios-core-from-source-96.html#Oracle_Linux)


```bash
sed -i 's/SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
setenforce 0
yum install -y gcc glibc glibc-common perl httpd php wget gd gd-devel
yum install openssl-devel
yum update -y
wget -O nagioscore.tar.gz https://github.com/NagiosEnterprises/nagioscore/archive/nagios-4.4.6.tar.gz
tar xzf nagioscore.tar.gz
cd nagioscore-nagios-4.4.6/
./configure
make all
make install-groups-users
usermod -a -G nagios apache
make install
make install-daemoninit
systemctl enable httpd
make install-commandmode
make install-config
make install-webconf
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --reload
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
systemctl start httpd
systemctl start nagios

http://<server_ip>/nagios

#------------------------------------
# Installing Nagios plugins
#------------------------------------

yum install -y yum-utils
yum-config-manager --enable ol8_optional_latest
cd /tmp
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -ihv epel-release-latest-7.noarch.rpm
yum install -y gcc glibc glibc-common make gettext automake autoconf wget openssl-devel net-snmp net-snmp-utils
yum install -y perl-Net-SNMP

wget --no-check-certificate -O nagios-plugins.tar.gz https://github.com/nagios-plugins/nagios-plugins/archive/release-2.3.3.tar.gz
tar zxf nagios-plugins.tar.gz
cd /tmp/nagios-plugins-release-2.3.3/
./tools/setup
./configure         # or ./configure --disable-ssl
make
make install

systemctl restart nagios
systemctl status nagios
```

### Configuring Remote Monitoring 

| [Ref Link](https://support.nagios.com/kb/article/nrpe-how-to-install-nrpe-8.html)

```bash
# steps to be performed on remote machine & nagios server
cd /tmp
wget http://assets.nagios.com/downloads/nagiosxi/agents/linux-nrpe-agent.tar.gz
tar xzf linux-nrpe-agent.tar.gz
cd linux-nrpe-agent
./fullinstall
systemctl status xinetd
netstat -tnlp | grep xinetd

# Test NRPE locally
/usr/local/nagios/libexec/check_nrpe -H 127.0.0.1

# Configure NRPE (if needed accordingly)
vi /usr/local/nagios/etc/nrpe.cfg

#----------------------------------------
# Confuring Nagios Server to talk to NRPE
#----------------------------------------

vi /usr/local/nagios/etc/nagios.cfg
    # uncomment the below line 
    cfg_dir=/usr/local/nagios/etc/servers 

# Create the directory that will store the configuration file for each server that you will monitor
mkdir /usr/local/nagios/etc/servers

# Add a new command to our Nagios configuration
vim /usr/local/nagios/etc/objects/commands.cfg
    # add the below lines at end
    define command{
 command_name check_nrpe
 command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}

systemctl restart nagios

# Monitoring a remote Linux Server (configuring on Nagios server)
- Add Host to Nagios Server Configuration
- Need to create a new configuration file for each of the remote hosts that we need monitoring.

cd /usr/local/nagios/etc/servers/
vim remoteServer.cfg
    # Add the below code.
    # Note hostname must be already configured

# To Monitor if the host is up or down :
define host { 
    use linux-server 
    host_name <server_hostname>
    alias RemoteLinux
    address <server_IP>
    max_check_attempts 5 
    check_period 24x7 
    notification_interval 30 
    notification_period 24x7 
}

# To Monitor the Ping : 
define service {
    use generic-service
    host_name <server_hostname>
    service_description PING
    check_command check_ping!100.0,20%!500.0,60%
}

# To Monitor SSH service 
define service {
    use generic-service
    host_name <server_hostname>
    service_description SSH
    check_command check_ssh
    notifications_enabled 0
}

# To Monitor Free Disk Space of SWAP
define service{
    use generic-service
    host_name <server_hostname>
    service_description Free Disk Space of SWAP
    check_command check_nrpe!check_hda2
}

# To Monitor Free Disk Space of /
define service{
    use generic-service
    host_name <server_hostname>
    service_description Free Disk Space of /
    check_command check_nrpe!check_hda3
}

# To Monitor Memory Utilization
define service{
    use generic-service
    host_name <server_hostname>
    service_description Memory Utilization
    check_command check_nrpe!check_mem
}

systemctl restart nagios

#---------------------------------------------------------------------------------------------------------------------
# Configure NRPE to monitor resource usage like disk usage , total processes , Memory Usage of Remote Linux Server.
#---------------------------------------------------------------------------------------------------------------------
vim /usr/local/nagios/etc/nrpe.cfg
    # uncomment the below line or add similar line according to your monitoring
    command[check_users]=/usr/local/nagios/libexec/check_users -w 5 -c 10
    command[check_load]=/usr/local/nagios/libexec/check_load -w 15,10,5 -c 30,25,20
    command[check_hda1]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/hda1
    command[check_hda2]=/usr/lib64/nagios/plugins/check_disk -w 20% -c 10% -p /dev/sda2
    command[check_hda3]=/usr/lib64/nagios/plugins/check_disk -w 20% -c 10% -p /dev/sda3
    command[check_zombie_procs]=/usr/local/nagios/libexec/check_procs -w 5 -c 10 -s Z
    command[check_total_procs]=/usr/local/nagios/libexec/check_procs -w 150 -c 200

# Now testing the configuration from Nagios Server (make sure you have configure the NRPE agent on Nagios server also)
- From the Nagios Server, test for Remote Linux Server Status

/usr/local/nagios/libexec/check_nrpe -H 10.99.50.18 -c check_total_procs 
/usr/local/nagios/libexec/check_nrpe -H 10.99.50.18 -c check_hda1
/usr/local/nagios/libexec/check_nrpe -H 10.99.50.18 -c check_mem

```
