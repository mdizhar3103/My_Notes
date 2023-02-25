**TCP Wrappers**

- Host based access control
- Logging
- Use to secure services running at service level
- Limit access based on user, hostname, or network
- Reference order of files matter (allow, then deny), If entry is in both the files, hosts.allow wins
- If no entry is found in either of them, the request service is granted.
- Rules in file are processed top down and first match terminate the processing.

```bash
man hosts.allow
man hosts.deny

# TCP Wrappers Configuration
<daemon list>: <client_ip_or_hostname list> [: <option>:- <option>: ...]

cat /etc/hosts.allow
    # below are example entry to be added
    # sshd : server1.example.com
    # sshd : .example.com : DENY

cat /etc/hosts.deny
    # sshd : server2.example.com

```
