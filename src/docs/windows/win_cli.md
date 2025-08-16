# PowerShell snippets

## DISM

```powershell title="Clean up component store (Windows update files)"
dism.exe /online /cleanup-Image /AnalyzeComponentStore
dism.exe /online /cleanup-Image /StartComponentCleanup
```

## SSH

```powershell title="Generate ssh key pair"
cd ~
ssh-keygen -t rsa
```

## Alternatives to Linux tools

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

## Fancy PowerShell

### PowerShell profile pathes

|Scope|Path|Profile variable|
|---|---|---|
|Current user – Current host|$Home\My Documents\PowerShell\Microsoft.PowerShell_profile.ps1|$profile|
|Current user – All hosts   |$Home\My Documents\PowerShell\Profile.ps1|$profile.CurrentUserAllHosts|
|All Users – Current Host|$PSHOME\Microsoft.PowerShell_profile.ps1|$profile.AllUsersCurrentHost|
|All Users –  All Hosts|$PSHOME\Profile.ps1|$profile.AllUsersAllHosts|

```powershell title="Install module PSReadline"
Install-Module PSReadLine
```

```powershell title="Install module CompletionPredictor"
Install-Module CompletionPredictor
```

```powershell title="Install OhMyPosh"
winget install JanDeDobbeleer.OhMyPosh
```

[:octicons-download-16: Download OhMyPosh configuration file](https://download.al1al7.de/vim/posh_config.omp.json)

### Example for Profile.ps1 configuration

```powershell title="Sample for PowerShell profile"
using namespace System.Management.Automation
# Shows navigable menu of all options when hitting Tab
Set-PSReadlineKeyHandler -Key Tab -Function MenuComplete

# Autocompletion for arrow keys
Import-Module CompletionPredictor
Set-PSReadlineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadlineKeyHandler -Key DownArrow -Function HistorySearchForward
Set-PSReadLineOption -HistorySearchCursorMovesToEnd
Set-PSReadLineOption -PredictionSource HistoryAndPlugin
Set-PSReadLineOption -PredictionViewStyle ListView
Set-PSReadLineOption -EditMode Windows

# OhMyPosh configuration
oh-my-posh init pwsh --config "C:\OhMyPosh\posh_config.omp.json" | Invoke-Expression

# Read ssh configuration and enable autocompletion for ssh
Register-ArgumentCompleter -CommandName ssh,scp,sftp -Native -ScriptBlock {
	param($wordToComplete, $commandAst, $cursorPosition)

	function Get-SSHHostList($sshConfigPath) {
		Get-Content -Path $sshConfigPath `
		| Select-String -Pattern '^Host ' `
		| ForEach-Object { $_ -replace 'Host ', '' } `
		| ForEach-Object { $_ -split ' ' } `
		| Sort-Object -Unique `
		| Select-String -Pattern '^.*[*!?].*$' -NotMatch
	}
	
	function Get-SSHConfigFileList ($sshConfigFilePath) {
		$sshConfigDir = Split-Path -Path $sshConfigFilePath -Resolve -Parent
	
		$sshConfigFilePaths = @()
		$sshConfigFilePaths += $sshConfigFilePath
	
		$pathsPatterns = @()
		Get-Content -Path $sshConfigFilePath `
		| Select-String -Pattern '^Include ' `
		| ForEach-Object { $_ -replace 'Include ', '' }  `
		| ForEach-Object { $_ -replace '~', $Env:USERPROFILE } `
		| ForEach-Object { $_ -replace '\$Env:USERPROFILE', $Env:USERPROFILE } `
		| ForEach-Object { $_ -replace '\$Env:HOMEPATH', $Env:USERPROFILE } `
		| ForEach-Object { 
		$sshConfigFilePaths += $(Get-ChildItem -Path $sshConfigDir\$_ -File -ErrorAction SilentlyContinue -Force).FullName `
		| ForEach-Object { Get-SSHConfigFileList $_ } 
		}
	
		if (($sshConfigFilePaths.Length -eq 1) -and ($sshConfigFilePaths.item(0) -eq $sshConfigFilePath) ) {
			return $sshConfigFilePath
		}
	
		return $sshConfigFilePaths | Sort-Object -Unique
	}

	$sshPath = "$Env:USERPROFILE\.ssh"
	$hosts = Get-SSHConfigFileList "$sshPath\config" `
	| ForEach-Object { Get-SSHHostList $_ } `

	# For now just assume it's a hostname.
	$textToComplete = $wordToComplete
	$generateCompletionText = {
		param($x)
		$x
	}
	if ($wordToComplete -match "^(?<user>[-\w/\\]+)@(?<host>[-.\w]+)$") {
		$textToComplete = $Matches["host"]
		$generateCompletionText = {
			param($hostname)
			$Matches["user"] + "@" + $hostname
		}
	}

	$hosts `
	| Where-Object { $_ -like "${textToComplete}*" } `
	| ForEach-Object { [CompletionResult]::new((&$generateCompletionText($_)), $_, [CompletionResultType]::ParameterValue, $_) }
}
```
