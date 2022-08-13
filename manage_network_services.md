# Managing Network Services
Understanding Booting Process [Link](https://mdizharblog.hashnode.dev/understanding-booting-process-and-runlevels-in-linux)

|feature | init | systemd |
|------|-------|--------|
|performance | serial | parallel |
| dependency management | folder based | Programmatic |
| On demand service startup | No | Dynamic |
| logging | /var/log/* | jorunald |
| process tracking | NA | control groups |

> Systemd is a replacement of initd daemon and can perform all of its functions.

## Systemd Components

- __Units:__ 
    - Used to describe information about a system resource or component
    - Implemented in unit files 
        - Unit files specify the behaviour and    configuration of object
        - Define in text files in locations:
            - /etc/systemd/system   - user/admin created
            - /run/systemd/system   - runtime created
            - /usr/lib/systemd/system - installed with packages
        - Precedence of files are from top to bottom 
    - Unit have __*states*__: *Active/Inactive* or *Activating/Deactivating*
    - Group of unit represent state of a system
    - Units can be *enabled or disabled*, and also have dependency as *Before/After* or *Requires/Conflicts*
    - __*Unit Types*__: 
        - __Service__: starts and control daemons
        - __Socket__: encapsulates IPC or network sockets
        - __Slice__: groups units heirarchically
        - __Scope__: externally started processes (like commands)
        - __Snapshot__: save state of units
        - __Device__: kernel devices
        - __Mount__: filesystem mount
        - __Swap__: swap partitions
        - __Automount__: on demand filesystem mounting
        - __Path__: file or directory
        - __Timer__: trigger activation of units based on time 

```bash
ls -la /sbin/init
pstree -np
ls -lat /var/run/systemd/system/        # to see the dynamically created session directory when system is booted
w                                       # match with above timestamp
ls -la /etc/systemd/system

systemctl status <service_name>
vim /usr/lib/systemd/system/<service_name>.service
systemctl show <service_name>           # get default setting of service
man systemd.unit

# checking file priority
systemctl status <service_name>
cp <path_from_above_command> /etc/systemd/system/<service_name> # if the above command doesn't point to this location
systemctl daemon-reload
systemctl restart <service_name>
systemctl status <service_name>

# modifying service file
vim /etc/systemd/system/<service_name>
    # in Service section add the following
    Restart=on-failure      # to start service automatically on failure

systemctl daemon-reload     # to re-read the config file
systemctl restart <service_name>
systemctl status <service_name>
    # note the PID of the service
killall <service_name>      # kill all services related to that service
systemctl status <service_name>
    # note the PID again
ps -aux | grep <service_name>

systemctl --type=socket 
systemctl --all
```

- __Target:__
    - Earlier called runlevels
    - A grouping of units
    - Defines a system state which units are running
    - __*Default Targets*__:
        - poweroff (0)
        - resuce (1)
        - multi-user (2, 3, 4): Non-graphical
        - Graphical (5)
        - reboot (6)
        - emergency: read only root
        - hibernate: saves state to disk
        - suspend: saves state in RAM

```bash
systemctl list-units --type target              # shows only active
systemctl list-units --type target --all

ls -la /etc/systemd/system/
cd /etc/systemd/system/
ls -lah multi-user.target.wants/

systemctl get-default
systemctl reboot
```

- __Control Groups:__
    - Processes are assigned to cgroups
    - Organized by service, scope and slice
    - Runtime performance information
    - Control resource consumption

```bash
systemd-cgls
systemd-cgtop

# Restricting memory limit for a service
systemctl show <service_name> | grep -i mem
vim /etc/systemd/system/<service_name>
    # add the following line
    MemoryLimit=512M

systemctl daemon-reload
systemctl reload <service_name>
systemctl restart <service_name>
systemctl show <service_name> | grep -i mem
systemctl show <service_name> -p MemoryLimit    # to get specific parameter
```


