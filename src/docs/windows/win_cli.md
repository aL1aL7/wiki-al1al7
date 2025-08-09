# PowerShell snippets

```powershell title="Generate ssh key pair"
cd ~
ssh-keygen -t rsa
```

```powershell title="Replacement for missing ssh-copy-id on Windows"
type C:\Users\username\.ssh\id_rsa.pub | ssh user@192.168.0.1 "cat >> .ssh/authorized_keys"
```
