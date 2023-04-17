# Process Commands
```bash
ps 1
ps u 1
ps -U username         # get all process of specific user 
ps u -U username
ps l                    # to get nice values
ps lax
ps fax                  # tree view of processes
ps -ef --forest
# =====================================================================================

# changing nice value
nice -n 11 bash
renice 7 <PID>          # changing nice value
# =====================================================================================

# process tree
pstree
pstree -h
pstree -g               # group process
pstree <PID>            # to get specific process
pstree <PID> -s         # to get specific process with systemd
pstree <PID> -s -a      # to get specific process with systemd and arguments passed
pstree <PID> -s -a -p   # all information with process
# =====================================================================================

pmap <PID>
pwdx --help
# =====================================================================================

sar -u
sar -b
sar -n
sar -q 
# =====================================================================================

lsof -p <PID>           # list open files and directories of a process
lsof -p 1               # list all open files and directories of process 1
lsof <filename>
# =====================================================================================

journalctl -f           # follow
journalctl -p err       # show error logs
- info
- warning
- err
- crit
- debug
- notice
- emerg
- alert

journalctl -S 05:00               # show logs from time
journalctl -S 05:00 -U 14:00      # show logs between time
journalctl -b 0                   # logs of current boot
journalctl -b -1                  # logs of previoud boot
# =====================================================================================

last
lastlog

# =====================================================================================
# Running command in background mode using screen
>>> screen 
  >>> command               # run the command
  >>> ctrl + a + d          # detach the screen mode
>>> screen -list 
>>> screen -rd <id>         # go to detached running screen 
>>> screen -X -S <id> quit  # After completion of the command quit the screen

```
