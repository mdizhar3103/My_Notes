# working with poolkit
- Kind of authenticator mechanism tool.

```bash
pkaction                # list the governance policies (who can do what)
cat /etc/polkit-1/rules.d/50-default.rules

pkexec <command to run>

pkexec cat /etc/shawdow
    # if didn't get any output (this is due to restricted file access, run the below command in new session to see the output of above command)
    >> echo $$                  # get the above command bash PID
    >>> pkttyagent --process <above_command_PID>    # to get the poolkit authenticator.    
```
