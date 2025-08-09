# LUKS - Encryption

## Summary of basic commands

```bash title="Create encrypted partition"
cryptsetup -c aes-xts-plain64 -s 512 -h sha512 luksFormat /dev/sdX2
```

```bash title="Open encrypted partition as crypted\_sdX2"
cryptsetup luksOpen /dev/sdX2 crypted_sdX2
```

```console title="Add encryption key"
foo@bar:~$cryptsetup luksAddKey /dev/sdX2
```

!!! danger "Attention !!!"
    When you don’t have antoher key, you aren’t able to decrypt the partition afterwards</p>

```bash title="Remove encryption key"
cryptsetup luksRemoveKey /dev/sdX2
```
