## Installing Container in CentOS
```bash
yum install docker

vim /etc/containers/registries.conf
    # comment the following line
    unqualified-search-registries
    # add the following line
    unqualified-search-registries=["docker.io"]

docker --help
# disabling the podman and docker conflicts
touch /etc/containers/nodocker

docker --help
docker search alpine
docker search nginx
docker images

```
