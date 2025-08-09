# iSCSI

```sh title="Search for iSCSI targets"
iscsiadm --mode discovery --portal XXX.XXX.XXX.XXX --type sendtargets
```

```sh title="Login"
iscsiadm -m node -T iqn.2024-11.local.... -p XXX.XXX.XXX.XXX:3260 --login
```

```sh title="Check iSCSI sessions"
iscsiadm -m session
```

```sh title="Logout of iSCSI session"
iscsiadm -m node -T iqn.2024-05.local.... -p XXX.XXX.XXX.XXX:3260 -u
```

```sh title="Remove iSCSI target"
iscsiadm -m node -o delete -T iqn.2024-05.local......
```
