# Boot Mode
```bash
systemctl get-default
systemctl set-default multi-user.target     # cli mode
reboot
systemctl isolate graphical.target          # switching to graphical mode
systemctl isolate emergency.target          # going to emergency mode, to load few program used in case of debugging and system is loaded in read-only
systemctl isolate rescue.target             # with root privileges prompt is given
```

> Note: to use emergency and rescue mode, root must have password, if doesn't have you can't use these options

# Kernel Parameters
```bash
sysctl -a
sysctl -w option_from_above=1       # to enable use 1 or 0 for disable
```
