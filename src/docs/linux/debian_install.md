# Debian - Basics and common info

## Tasks for new installs

This includes following steps:

* Install `vim` including my preferred config and theme
* Install [ble.sh](https://github.com/akinomyoga/ble.sh) - a fancy bash style including nice features
* Install `net-tools`, `mc`, `git`, `bind9-dnsutils`, `ripgrep`, `fzf`, `curl`, `htop`, `bat`, `ranger`

```bash title='Install & Configuration of common packages including a fancy bash'
wget -O - https://download.al1al7.de/vim/mydebian.sh | bash -s
```

## Upgrade Debian bookworm (12) to trixie (13)

### Hints

* dovecot version changes from 2.3.19 to 2.4.1

!!! danger "This update needs conversion of dovecot configuration scheme, otherwise dovecot will not start"

* Debian trixie will be shipped with Plasma 6 (6.3.6) and KDE 6.13
* Folder /tmp will be a RAM-based tmpfs file system (see info in upgrade section)
* APT sources list format will be changed to [deb822](https://wiki.debian.org/Deb822), a structured format for APT sources that improves readability and flexibility.
* OpenSSH no longer supports DSA keys

### Upgrade workflow

```bash title="Update to latest bookworm release"
apt update
apt upgrade
```

!!! info "Reboot system after last upgrade"

```bash title="Cleanup"
apt autoremove
apt clean
```

!!! danger "Check mariadb state after stop mariadb service"
    Shutdown of mariadb must be clean - recovery after a major release upgrade of mariadb is not possible

```bash title="Stop mariadb [optional]"
systemctl stop mariadb
# check log
journalctl -n 100 -f
```

```bash title="Switch to trixie sources"
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list
apt update
```

!!! info "Check folder `/etc/apt/sources.list.d` for manually installed sources"

```bash title="Minimal upgrade"
apt upgrade --without-new-pkgs
```

```bash title="Full upgrade"
apt full-upgrade
```

!!! info "Reboot after upgrade"

```bash title="Cleanup auto installed packages"
apt autoremove
```

```bash title="Cleanup orphaned and local installed packages"
apt list '~o'
# check before purge the packages
apt purge '~o'
```

!!! info "/tmp folder is now a RAM-based tmpfs - after upgrade old files must be cleaned"

```bash title="Bind mount to make old /tmp files visible"
mount --bind / /mnt
# delete old files
rm -rf /mnt/tmp/*
```
