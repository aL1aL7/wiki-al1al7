# Proxmox Virtual Environment

## iSCSI

### Basics

Initiator name is stored in `/etc/iscsi/initiatorname.iscsi`.

Change some default values in `/etc/iscsi/iscsid.conf`:

```ini
# old value: 120 new value: 15
node.session.timeo.replacement_timeout = 15
```

Add the iSCSI LUN via Proxmox VE GUI:

1. Navigate to the Datacenter view.
2. Select the storage where you want to add the iSCSI LUN.
3. Click on "Add" and then "iSCSI".
4. Fill in the required fields, including the iSCSI target information. 

### iSCSI Multipath

Proxmox does not preinstall `multipath-tools`, so it have to be installed manually:

```bash
apt update
apt install multipath-tools
```

Add `/etc/multipath.conf` configuration file - try to get information about SAN from vendor to set correct values. Following example is for DELL ME4024 SAN

```bash title="Dell ME4024 /etc/multipath.conf"
##Default System Values
defaults {
   user_friendly_names yes
   find_multipaths strict
   max_fds 8192
   polling_interval 5
   queue_without_daemon no
}

## Universal Blacklist  (recommend white-listing)
blacklist {
   device {
      vendor '*'
      product '*'
   }
}

## Blacklist Exceptions
blacklist_exceptions {
   device {
      vendor "DellEMC"
      product "ME4"
   }
}

## Dell Device Configuration
devices {
   device {
      vendor "DellEMC"
      product "ME4"
      path_grouping_policy group_by_prio
      path_checker "tur"
      hardware_handler "1 alua"
      prio "alua"
      failback immediate
      rr_weight "uniform"
      path_selector "service-time 0"
   }
}
```

To identify the iSCSI LUNs, use the WWID. The command scsi_id can be used to determine the WWID of a device:

```bash
/lib/udev/scsi_id -g -u -d /dev/sdX
```

Replace `/dev/sdX` with the appropriate device identifier, which you can get with following command:

```bash
iscsiadm -m session -P3
```

Add WWIDs to the WWIDs file

```bash
multipath -a <WWID>
```

### Increase size of VG

1. Enlarge LUN on SAN
2. Rescan the iSCSI devices
   ```bash
   iscsiadm --mode session --rescan
   ```
3. Check size before increasing for comparison
   ```bash
   multipath -l /dev/sdX
   # or
   multipath -l /dev/mapper/mpathX
   ```
4. Resize multipath device
   ```bash
   multipathd resize map mpathX
   ```
5. Check size after resizing
   ```bash
   multipath -l /dev/sdX
   # or
   multipath -l /dev/mapper/mpathX
   # or
   lsblk /dev/mapper/mpathX
   ```
6. Resize of physical volume
   ```bash
   pvresize /dev/mapper/mpathX
   ```
