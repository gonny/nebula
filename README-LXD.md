# LxdMosaic
https://github.com/turtle0x1/LxdMosaic

## Motivation
We'd like to have one management tool which is running in  LXC/LXD and can manage host/s cluster/s over the deployments and offer one view.
We'd like to use native containers, not microservices with minimal overhead. Our application is not so heavy to use k8s,docker or some other microservice/service provider.
We'd like to have precompiled images in our repository and server them to customers with their custom configs.
As much as possible work must be done from remote server with minimal human interaction. We can possibly use CDN or some remote Cloud storage to provide our images. Faster and automated deployment mean less problems during deployment and is lowering the costs. More reliability will be shifted to testing environment.
.
.
.
## Pros
### LXC/LXD
+ No iptables
+ Network is managed by LXD itself
+ Do not add non-system users to the containers because we'll use different ( container ) approach
### GUI
+ It is written in PHP / NodeJS ( Angular )
+ It uses Web API of LXD
+ It offers a lot of information and statistic about the running environment

## Cons
### LXC/LXD

### GUI
+ Network cannot be managed from within the GUI

# Useful guides
* [Remap user to LXD](https://ubuntuforums.org/showthread.php?t=2322924)
* [read-write storage](https://www.cyberciti.biz/faq/how-to-add-or-mount-directory-in-lxd-linux-container/)
* [LXD and cloud-init](https://linuxcontainers.org/lxd/docs/master/cloud-init)
* [LXD/LXC instance and profile configuration](https://github.com/lxc/lxd/blob/master/doc/instances.md)
* [Systemd interface custom DNS](https://blog.simos.info/how-to-use-lxd-container-hostnames-on-the-host-in-ubuntu-18-04/)
* [LXD config settings](https://github.com/lxc/lxd/blob/master/doc/instances.md#type-unix-block)
* [LXD API with authentication using CURL](https://stgraber.org/2016/04/18/lxd-api-direct-interaction/)
* [VXLAN Unicast overlay](https://lxadm.com/Unicast_VXLAN:_overlay_network_for_multiple_servers_with_dozens_of_containers)

# LXD lxc_custom_configuration
## What to test
* VLAN communication
* Resource limitation

# ~~nebula~~
~~OpenNebula deployment inside internal environment~~
* It seems that openNebula is not for our needs. OpenNebula is great tool for managing VM ( KVM ) instances and LXD implementation is done using opennebula LXD package which must be installed on host.

```
printf 'lxc.init.cmd = /docker-entrypoint.sh /usr/local/bin/redis-server /data/redis.conf\nlxc.signal.halt=SIGTERM\nlxc.init.uid=999'| lxc config set redisdb-build  raw.lxc - ; lxc stop redisdb-build --force
~ # cat /docker-entrypoint.sh
#!/bin/sh
set -e

# first arg is `-f` or `--some-option`
# or first arg is `something.conf`
if [ "${1#-}" != "$1" ] || [ "${1%.conf}" != "$1" ]; then
	set -- redis-server "$@"
fi

# allow the container to be started with `--user`
if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
	find . \! -user redis -exec chown redis '{}' +
	exec su-exec redis "$0" "$@"
fi

exec "$@"
```
*rc-update del crond
