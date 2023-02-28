```bash
sudo -l

username ALL=(root) NOPASSWD: /usr/bin/passwd, !/usr/bin/passwd root 
                |               |                   |
                |               |                   +-----> dont allow to run this command
                |               +----------> run only this command
                +---> only sudo as root
```
