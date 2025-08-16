# Exchange SE - Management Shell snippets

## Check OAuth

[Documentation by Microsoft](https://docs.microsoft.com/en-us/exchange/troubleshoot/administration/exchange-oauth-authentication-could-not-find-the-authorization)

```powershell title="Is certificate configured?"
Get-AuthConfig | Format-List CurrentcertificateThumbPrint
```

```powershell title="Yes -> get certificate"
Get-ExchangeCertificate
```

```powershell title="No - generate new certificate"
New-ExchangeCertificate -KeySize 2048 -SubjectName "cn=Microsoft Exchange Auth Certificate for Me" -FriendlyName "Microsoft Exchange Server Auth Certificate for Me" -PrivateKeyExportable $true -Services SMTP -DomainName <yourdomain>
```

```powershell title="No - already generated, configure it for OAuth"
# Replace <ThumbprintFromStep4A> with the Thumbprint value obtained from the 'Get-ExchangeCertificate' command above
Set-AuthConfig -NewCertificateThumbprint <ThumbprintFromGetExchangeCertificate> -NewCertificateEffectiveDate $(get-date)
```

```powershell title="Publish certificate"
Set-AuthConfig -PublishCertificate
```

```powershell title="Map certificates to services"
# Replace <Thumbprint> with your certificate's actual thumbprint
Enable-ExchangeCertificate -Thumbprint <Thumbprint> -Services POP,IMAP,IIS,SMTP
```

## Health state

```powershell title="Query state"
Get-ServerHealth myexchange.mydomain | ?{$_.HealthSetName -eq "OWA.Proxy"}
```

```powershell title="Query component state"
(Get-ServerComponentState -Identity myexchange.mydomain -Component OwaProxy).LocalStates
```

```powershell title="Enable/Disable component"
Set-ServerComponentState myexchange.mydomain -Component OwaProxy -State Active -Requester HealthApi
```

!!! info "Take care of Requester"

```powershell title="List all unhealthy components"
Get-ServerHealth myexchange.mydomain | ? { $_.AlertValue -eq "UnHealthy" }
```

## Mailbox rules

```powershell title="Create new rule"
New-InboxRule "Category HR" -HeaderContainsWords "X-CUSTOM-FW-RULE: DENY RULE HR" -ApplyCategory "HR" -Mailbox myuser
```

## Query mailbox state & repair mailbox

[Documentation by  New-MailboxRepairRequest](https://docs.microsoft.com/en-us/powershell/module/exchange/new-mailboxrepairrequest?view=exchange-ps "New-MailboxRepairRequest")

[Documentation by  Get-MailboxRepairRequest](https://docs.microsoft.com/en-us/powershell/module/exchange/get-mailboxrepairrequest?view=exchange-ps "Get-MailboxRepairRequest")

```powershell title="Check only"
New-MailboxRepairRequest -Mailbox myuser -CorruptionType ProvisionedFolder,SearchFolder,FolderView,AggregateCounts -DetectOnly
```

```powershell title="Repair mailbox"
New-MailboxRepairRequest -Mailbox myuser -CorruptionType ProvisionedFolder,SearchFolder,FolderView,AggregateCounts
```

```powershell title="Get state of repair request for mailbox"
Get-MailboxRepairRequest -Mailbox myuser
```

```powershell title="Get state of repair request for full database"
Get-MailboxRepairRequest -Database MailboxDatabase01
```

```powershell title="Show only errors"
Get-MailboxRepairRequest -Database MailboxDatabase01 | Format-list Tasks,Corruption*
```

## Move mailbox

```powershell title="Move mailbox to new database"
# Optional -BadItemLimits (number of bad objects, which are accepted)
New-MoveRequest -Identity 'myuser@mydomain' -TargetDatabase "NEWDB"
```

```powershell title="State of move request"
Get-MoveRequest -ResultSize Unlimited | Get-MoveRequestStatistics
```

```powershell title="Delete finished mailbox move requests"
Get-MoveRequest | where {$_.status -eq "Completed"} | Remove-MoveRequest
```

## Managed Folder Assistant

Managed folder assistant can be used to start archiving of mailbox.

```powershell
Start-ManagedFolderAssistant -Identity myuser
```

## Renew certificates

On the Exchange Console (launchems), create a CSR (e.g., read the thumbprint of the certificate to be renewed with the healthchecker.ps script):

```powershell
$txtrequest = Get-ExchangeCertificate -Thumbprint 3EB742B253689947017397C250BB31D65DB748F1 | New-ExchangeCertificate -GenerateRequest 
[System.IO.File]::WriteAllBytes('c:\custom\exch.req', [System.Text.Encoding]::Unicode.GetBytes($txtrequest))
```

Copy the file exch.req you just created to the domain controller and, as Administrator, execute the following on the DC

```powershell
certreq -attrib "CertificateTemplate:WebServer"
```

In the user interface, select the previously copied file, the CA, and the storage location

The newly signed certificate must be copied to the Exchange server and imported (launchems)

```powershell
Import-ExchangeCertificate -FileData ([System.IO.File]::ReadAllBytes('c:\custom\exch-2024.cer'))
```

Afterwards, assign the corresponding certificate in the EAC.

## Queue-Viewer

Exchange management tools must be installed to get an user interface for Exchange queues. The tools can be installed at Exchange servers or clients.

You can launch the Exchange Queue Viewer from the Run dialog (`Win + R`) or Command Prompt with:

```powershell
%ProgramFiles%\Microsoft\Exchange Server\V15\Bin\Exchange Queue Viewer.msc
```

Or, from PowerShell, use:

```powershell
Start-Process "$env:ProgramFiles\Microsoft\Exchange Server\V15\Bin\Exchange Queue Viewer.msc"
```
