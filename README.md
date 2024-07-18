# Windows Server Core 2022 Guide
This is a guide on how to install and configure Windows Server Core 2022 with an Active Directory!

## Download Windows Server Core 2022
https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022

Flash the image to an USB drive using Rufus https://rufus.ie/en/ or use Ventoy https://ventoy.net.
Please note that using these tools will erase all data on the USB drive, so make sure to back up any important files before proceeding.

## Install Windows Server Core 2022

Follow the instructions and choose your desired version without the Desktop experience and partition your hard drive.

## SConfig configuration

After a successful installation, configure your machine name and network adapter. You may want to come back here later if you want to use this machine as your local DNS resolver after installing Active Directory, in which case you will enter 127.0.0.1. If you already have something like this, such as a Pi-Hole using dnsmasq as a DNS forwarder or unbound as resolver, you'll want to enter its IP here now.

## Install the SSH Server

First, start PowerShell with `powershell`!

Then type the following commands:

```
dism /Online /Add-Capability /CapabilityName:OpenSSH.Server~~~~0.0.1.0

Set-Service -Name sshd -StartupType 'Automatic'

Start-Service sshd
```

If you want to configure the SSH server now, you can type:
`notepad C:\Programdata\ssh\sshd_config`

We can finally log in via SSH from another machine and type in commands less tediously!

## Download and install a newer Powershell version
```
Invoke-WebRequest -Uri https://github.com/PowerShell/PowerShell/releases/download/v7.2.0/PowerShell-7.2.0-win-x64.msi -OutFile $env:TEMP\PowerShell-7.2.0-win-x64.msi

Start-Process msiexec.exe -Wait -ArgumentList '/I', "$env:TEMP\PowerShell-7.2.0-win-x64.msi"
```

## Set Powershell as default shell
`Set-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "C:\Program Files\PowerShell\7\pwsh.exe"`

## Download the Windows Admin Center
https://www.microsoft.com/de-de/windows-server/windows-admin-center

Push the installer to the Windows Server machine via SCP.

`scp ~/Downloads/WindowsAdminCenter2311.msi Administrator@<SERVER-IP>:C:/Users/Administrator/WindowsAdminCenter2311.msi`

Go back to the Windows Server and install the Admin Center interactively with `.\WindowsAdminCenter2311.msi`. After a successful installation, we can now easily monitor and install new features on our Windows Server by visiting the IP address or machine name as a domain.
Note: Windows Admin Center will not work in Firefox! If you are using another DNS solution, you may want to use the IP address and set up the domain there and visit `SConfig` again!

## Backup
Now that we have a basic setup running, it is a good idea to back it up to another drive via SSH again with:


`C:\Windows\System32\wbadmin.exe START SYSTEMSTATEBACKUP -backupTarget:<DRIVELETTER>:`

## Install Active Directory
One does not simply remove a domain controller from a server easily, that's why we back up our system beforehand to test our AD environment!

From a Windows client, ping the domain and if you get a response, log in with AD\Administrator.
From there you can install AD Tools and configure your directory with the OUs you want, etc.

Note: If you don't get a succesful ping there might be an issue with your DNS or firewall setup!
To install paste the following commands:

```
Install-WindowsFeature AD-Domain-Services

Install-ADDSForest -DomainName "ad.dieschoenewolke.com"
```

## Join the domain

From a Windows 10 client, ping the domain and if you get a response, type `env` in the search bar and click `Environment Variables`, then go to `Computer Name` in the tab bar, click the second button labeled `Change...` and change the radio button from `Workgroup` to `Domain` below `Member of:` and enter your domain name!

Then log out and log in with `AD/Administrator`. From there you can install the desired RSAT tools and configure your directory with the desired OUs, etc.

Note: If you don't get a successful ping, there may be a problem with your DNS or firewall setup!

## Backup again
`C:\Windows\System32\wbadmin.exe START SYSTEMSTATEBACKUP -backupTarget:<DRIVELETTER>:`