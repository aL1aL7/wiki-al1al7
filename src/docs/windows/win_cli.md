# PowerShell snippets

## Clean component store

```powershell title="Clean Up Component Store (Windows update files)"
dism.exe /online /cleanup-Image /AnalyzeComponentStore
dism.exe /online /cleanup-Image /StartComponentCleanup
```

## SSH

```powershell title="Generate ssh key pair"
cd ~
ssh-keygen -t rsa
```

## Alternatives to Linux Tools

```powershell title="Replacement for missing ssh-copy-id on Windows"
type C:\Users\username\.ssh\id_rsa.pub | ssh user@192.168.0.1 "cat >> .ssh/authorized_keys"
```

```powershell title="Replacement for grep"
findstr /b "<?php" *.php
```

Useful parameters for `findstr`

- `/b - match string must be at the begin`
- `/e - match string must be at the end`
- `/s - recursive`

## Windows services

```powershell title="Get services"
Get-Service | where -Filterscript {$_.StartType -eq 'Disabled'} | ft 'Name','StartType',"Status"
```

```powershell title="Set startup type of service"
Set-Service -StartupType Automatic -Name <ServiceName>
```

```powershell title="Start service"
Start-Service <ServiceName>
```

## Active Directory

```powershell title="Sync all domain controllers (PUSH)"
repadmin /syncall /AdeP
```

```powershell title="Force DFSR sync"
dfsrdiag pollad
```