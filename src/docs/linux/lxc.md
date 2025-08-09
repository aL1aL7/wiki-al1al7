# LXC Container - Requirements & Basics

## Requirements

```bash title='Install requirements for debian'
apt-get install lxc libvirt0 libpam-cgfs bridge-utils uidmap
```

## Basics

```bash title='Start Container'
lxc-start -n %NAME%
```

```bash title='Stop Container'
lxc-stop -n %NAME%
```

```bash title='Attach Container (get access to cli inside container)'
lxc-attach -n %NAME%
```

```bash title='List info/state of containers'
lxc-ls --fancy
```

```bash title='Create container (template downloaded for unprivileged container)'
lxc-create -n %NAME% -t download -- -r bookworm
```

```bash title='Destroy (delete) container - container have to be stopped before'
lxc-destroy -n %NAME%
```

## Creating unprivileged containers

When creating unprivileged containers as root with shared UID and GID the files `/etc/subuid` and `/etc/subgid` need some entries. Check content of both files before appending stuff to it! Furthermore the default lcx configuration file `/etc/lxc/default.conf` has to be extened.

```bash
echo "root:100000:65536" >>/etc/subuid
echo "root:100000:65536" >>/etc/subgid
echo "lxc.idmap = u 0 100000 65536" >>/etc/lxc/default.conf
echo "lxc.idmap = g 0 100000 65536" >>/etc/lxc/default.conf
```

## Autostart for containers

Add the following line to configuration file (`/var/lib/lxc/%%NAME%%/config`) of each container, which should be start automatically.

```ini
lxc.start.auto = 1
```

## Basic default configuration

```ini
xc.net.0.type = veth
lxc.net.0.link = lxcbr0
lxc.net.0.flags = up
lxc.net.0.hwaddr = 00:16:3e:xx:xx:xx

lxc.apparmor.profile = generated
lxc.apparmor.allow_nesting = 1

lxc.idmap = u 0 100000 65536
lxc.idmap = g 0 100000 65536
```
