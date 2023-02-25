# Linux Host Security
System Logging

- rsyslog (Rocket Fast System log Processing)
- A way to route and store messages
- Logs local messages
- Logs to remote system (both client and server)

```bash
#  Configuring syslog and sending log to centralized logging server with rsyslog    

# Configuring Server
yum install rsyslog -y
systemctl enble rsyslog --now
systemctl status rsyslog

vim /etc/rsyslog.conf
    # Look for: 
    # imuxsock - provides support for local system logging using logger command
    # imjournal - provides access to systemd journal
    # uncomment the line
    $ModLoad imtcp
    $InputTCPServerRun 514

systemctl restart rsyslog
firewall-cmd --add-port 514/tcp --permanent
firewall-cmd --reload

# making selinux to allow this service
sudo semanage port -a -t syslogd_port_t -p tcp 514

# Configuring Client
vim /etc/rsyslog.conf
    # Go to End of file and add the following file
    *.* @@remote-host:514
    # Note: Single @ means udp protocol and double @@ means tcp protocol
    # Adding specific server hostname
    *.* @@fully_qualified_domain_name:514

systemctl restart rsyslog
logger "Sending Message to server"

# on server check the logs
tail -f /var/log/messages

```
