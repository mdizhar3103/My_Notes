## Managing Docker Using Context

Inspecting Docker:
```bash
# grant docker access to a user
usermod -aG docker username         # Sometime after granting access you will still get permission error, so to avoid in some case either you need to logout and login again or reboot the machine

# listing the systemctl units
systemctl list-unit-files

cat /etc/systemd/system/multi-user.target.wants/docker.service

rpm -ql docker-ce-cli
```

#### Working with Docker Contexts
- Using Docker Contexts for Remote Access
- Command to Container
    - docker command
    - Unix socket (IPC) to API, root:docker
    - Docker Engine API (create container, list images, etc)
    - Docker Engine Daemon (dockerd) executes the incoming API task/query
    - containers (isolated processes) and associated resoruces like images, mounts, networking, cgroups, security, processes. 

```bash
# nsenter - to run program with namespaces of other processes
man nsenter

# Red Hat
# https://www.redhat.com/sysadmin/container-namespaces-nsenter

runc list
# lsns command to list the namespaces associated with a given process.
PID=$(docker inspect --format {{.State.Pid}} <container_name_or_ID>)
nsenter --target $PID --mount --uts --ipc --net --pid
lsns -p $PID
```

**Managing Connection with Named Contexts**
```bash
docker context

# get default context
docker context ls
docker version

        Docker
    +-----------------------+
    |  docker-cli           |
    |                       |
   VM           unix.socket |
    |                       |
    +-----------------------+

    
# Setting up docker-cli only
yum install docker-ce-cli

docker context ls
docker context inspect
docker version

        Docker-cli only
    +-----------------------+
    |  docker-cli           |
   VM                       |
    +-----------------------+

# ================================================================
# Connecting to socket remotely on ubuntu using 3rd part snappy
# creating binaries on other machine for docker API interaction

            Snappy
    +-----------------------+
    |  docker-cli           |------+
   VM                       |      |
    +-----------------------+      |
                                   |
            binaries               |
    +-------------------------+    |
    |                   socket <---+
    |                       | | 
    | dockerd ---- API <--->| |
    |                         |
    +-------------------------+


# refer to docker doc to download docker using binaries
# extract binaries
tar xzvf "docker-${VERSION}.tgz"

# copy or link binaries into directory in path already
# or modify path to include binaries folder
sudo cp docker/* /usr/bin/

# add a docker group and assign normal user membership
groupadd --system docker
useradd izhar docker

# start dockerd by hand in background:
dockerd --help
dockerd &
docker context ls
# kill the process now

systemctl status docker
systemctl stop docker
dockerd -H tcp://           # listen on default port 2375

docker context ls
docker -H tcp:// version    # passing arguments
docker -H tcp:// context ls

DOCKER_HOST=tcp:// docker context ls
DOCKER_HOST=tcp:// docker version

# On cli side configuring to connect to binaries vm
docker version
export DOCKER_HOST=tcp://binary_machine_ip:2375

# Running binaries on all interfaces (go to binary machine)
dockerd -H tcp://0.0.0.0:2375

# Now running on cli machine
docker version
docker container run --rm -d -p 8080:80 nginx

# now curl binray_machine_ip:8080
docker image ls
unset DOCKER_HOST
docker context ls
docker container ls
docker image ls

docker -H tcp://binary_ip:2375 image ls

docker context --help
docker context create --help

docker context create binaries --description "binary vm" --docker "host=tcp://binary_ip:2375"

docker context ls
docker context create locally --description "local unix socket"
docker context ls

# switching over contexts
docker context use binaries
docker images ls
docker context ls
docker context locallly
docker container ls
docker context inspect binaries

# exporting docker context
docker context export binaries

# check for docker context import
```

#### Securing Access to Docker
```bash
# on binary vm
sudo dockerd

# on cli vm
docker -H ssh://username@hostname_or_ip version
# In case of ssh error make sure you config of the remote machine in cli vm as:
Host binary_remote_ip
     HostName binary_remote_hostname
     User binary_username
     StrictHostKeyChecking no
     PasswordAuthentication no
     IdentityFile ~/pvt_key_path

docker context create binaries --docker "host=ssh://username@binary_ip"

docker context use binaries
docker context ls
docker version 
docker image ls
```

**Configuring Insecure Docker TCP**
```bash
# on binaries
systemctl cat docker.socket
systemctl edit docker.socket
    # add the following line
    [Socket]
    ListenStream=
    ListenStream=0.0.0.0:2375

systemctl status docker.socket
systemctl restart docker.socket

# on cli 
docker -H tcp:// version

# ----------------
watch -n 0.5 --color SYSTEMD_COLORS=1 systemctl status docker.service
# in another session run the following command and watch the output in above command
sudo systemctl stop docker.service
# Now from the outside vm access the following url
curl http://vm_ip:2375/info         # as soon as you hit this url the docket.socket will automatically start the docker daemon although we stopped the service
curl http://vm_ip:2375/info | jq
```

### Working with Containerd
- If docker is already installed containerd will also be present
- If not then manually download containerd from the official site

```bash
containerd --help
ctr --help

containerd config dump
containerd config default   # checking the default config

# starting containerd
containerd

ctr version
ls -l /run/containerd/      # check the ownership of files specially containerd.sock

# Configuring containerd
vim /etc/containerd/config.toml
    [grpc]
    # give user gid
    gid = 1000

# restart the containerd
containerd
ls -l /run/containerd/      # after re-running check the ownership of the socket file it will change according to your config
containerd config dump | grep gid

# checking the differences 
diff <(containerd config default) <(containerd config dump)

# Pulling image using ctr
ctr image pull docker.io/library/hello-world:latest
ctr image ls
ctr run --rm docker.io/library/hello-world:latest izhar_container       # if give error due to runc, we need to install that too
```
