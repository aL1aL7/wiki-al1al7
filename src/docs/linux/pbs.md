# Proxmox BackupServer

## Create datastore at sshfs mount

1. mount path via sshfs e.g. `/mnt/backup`
2. use bindfs to ignore missing chown and chgrp for sshfs mount

    ```bash
    bindfs --chown-ignore --chgrp-ignore /mnt/backup /mnt/backup-bindfs
    ```

3. Create datastore via pbs UI with path /mnt/backup-bindfs
4. Modify `/etc/proxmox-backup/datastore.cfg` and replace `/mnt/backup-bindfs` with `/mnt/backup`
