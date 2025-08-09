# mdadm - Software Raid

## Basic commands dealing mdadm

```bash title="Create an array"
mdadm --create /dev/md/md_name --level=<RAID-Level> --raid-devices=<amount of physical partitions at array> /dev/sdX1 /dev/sdY1
```

```bash title="Check raid array"
mdadm --detail /dev/md/<LABEL>
# or
cat /proc/mdstat
```

```bash title="Remove raid array"
umount /dev/md/md_name
mdadm --stop /dev/md/md_name
mdadm --zero-superblock /dev/sdX1
mdadm --zero-superblock /dev/sdY1
```

```bash title="Restart sync of array"
mdadm --readwrite /dev/md/md_name
```

```bash title="Update raid config"
/usr/share/mdadm/mkconfig > /etc/mdadm/mdamd.conf
```

!!! info "Additional tasks, when device is needed for boot"

```bash
update-initramfs -u
update-grub2
grub-install /dev/sdX
```
