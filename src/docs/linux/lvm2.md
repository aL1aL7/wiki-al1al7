# LVM2

## Physical disks

```bash title="create/add physical disk"
pvcreate /dev/sdb
```

## Volume groups

```bash title="Create volume group"
vgcreate vg00 /dev/sdb1 /dev/sdc1
```

## Volumes

```bash title="Create volume"
# create by given size in GB
lvcreate -n data -L1G vg00
# create by given size of vg in %
lvcreate -n data -l100%VG vg00
# create by given free size of vg in %
lvcreate -n data -l100%FREE vg00
```
