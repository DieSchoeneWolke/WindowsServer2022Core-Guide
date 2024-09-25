# Windows Server 2022 Core Guide

This is a guide on how to install and configure `Windows Server 2022 Core` with an `Active Directory`!
The `Core` version comes with fewer pre-installed features and a slimmer GUI resulting in less hardware requirements and improved security and reliability.

## Download Windows Server 2022

https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022

Flash the image to an USB drive using [Rufus](https://rufus.ie/en/) or use [Ventoy](https://ventoy.net).
Please note that using these tools will erase all data on the USB drive, so make sure to back up any important files before proceeding.

## Install Windows Server 2022 Core

Follow the instructions and choose your desired version without the `Desktop experience` to install the `Core` version and partition your hard drive.
After a successful installation, your system will reboot, you'll need to press `Ctrl + Alt + Del`  and you need to provide a complex password for your `Administrator` account.

Note: To verify the password you need to press `TAB` not `Enter`.

## SConfig configuration

You are now in `SConfig` and want to configure your `Machine name` with a memorable name by typing `2` in the main menu - restart when you are finished with this block - and your `Network settings` afterwards by typing `8` in the main menu, select your primary `Network adapter` with the appropriate number and `S` to set a static IP address, subnet mask and a default gateway. Type `2` now to configure your DNS with a public DNS e.g. `1.1.1.1` or `8.8.8.8` or one provided from your Internet service provider.

## Install the SSH Server

```
dism /Online /Add-Capability /CapabilityName:OpenSSH.Server~~~~0.0.1.0

Set-Service -Name sshd -StartupType 'Automatic'

Start-Service sshd
```

If you want to configure the `SSH` server now, you can type:
`notepad C:\Programdata\ssh\sshd_config`

You can finally log in via `SSH` from another machine and type in commands less tediously!

## Download and install a newer Powershell version

```
Invoke-WebRequest -Uri https://github.com/PowerShell/PowerShell/releases/download/v7.2.0/PowerShell-7.2.0-win-x64.msi -OutFile $env:TEMP\PowerShell-7.2.0-win-x64.msi

Start-Process msiexec.exe -Wait -ArgumentList '/I', "$env:TEMP\PowerShell-7.2.0-win-x64.msi"
```
Note: You can start `PowerShell` with `powershell` if you are in `cmd` which is the default as of now if you connect via SSH!

## Set Powershell as default shell for SSH connections

`Set-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "C:\Program Files\PowerShell\7\pwsh.exe"`

## Download the Windows Admin Center

https://www.microsoft.com/de-de/windows-server/windows-admin-center

Push the installer to the Windows Server machine via `SCP`.

`scp ~/Downloads/WindowsAdminCenter2311.msi Administrator@<SERVER-IP>:C:/Users/Administrator/WindowsAdminCenter2311.msi`

Go back to the Windows Server and install the `Windows Admin Center` interactively with `.\WindowsAdminCenter2311.msi`. After a successful installation, you can now easily monitor and install new features on our Windows Server by visiting the IP address or machine name as a domain.
Note: `Windows Admin Center` will not work in `Firefox`! If you are using another DNS solution, you may want to use the IP address and set up the `A name record` pointing to your desired domain name.

## Backup

Now that you have a basic setup running, you can use SSH again and back up the system to another drive with:


`C:\Windows\System32\wbadmin.exe START SYSTEMSTATEBACKUP -backupTarget:<DRIVELETTER>:`

## Install Active Directory

One does not simply remove a domain controller from a server easily, that's why we backed up our system beforehand in case something goes wrong with our upcoming AD environment!
Go back to SConfig and change your DNS to `127.0.0.1`.

```
Install-WindowsFeature AD-Domain-Services

Install-ADDSForest -DomainName "ad.dieschoenewolke.com"
```

## Join the domain

From a Windows 10 client, ping the domain and if you get a response, open `Run` by pressing the `Windows key + R` on your keyboard and type in `sysdm.cpl` to open `System Properties` and click the second button labeled `Change...`. In the following window, change the radio button from `Workgroup` to `Domain` under `Member of:` and enter your domain name! After a few seconds a window should appear with the text `Welcome to the ad.dieschoenewolke.com domain.` After clicking ok, you'll be asked to restart your computer.


Note: If you don't get a successful ping, there may be a problem with your DNS or Firewall setup!
Make sure to set the IP address of the Windows Server as primary DNS in your Windows client.


After the restart log in with `AD/Administrator`. After the initialization of the user profile, press `Windows key + I` to open the `Settings` and search for `Add an optional feature`. Click on `Add a feature` at the top of the window and install `RSAT: Active Directory Domain Services and Lightweight Directory Services Tools`. Now open `Active Directory Users and Computers` and configure your `Active Directory` by clicking on your domain in the left pane and right click in the right pane, hover over `New` and click on `User` and fill out the form with your `First- and Last name`, `Initials` and a `User logon name`. Click on `Next >` and provide a password for the account. You can then log in with `AD/<User logon name>`.

If you want to use your existing settings from your local user account as an AD user, the easiest way is to copy over the files from your `C:\Users\<USERNAME>` directory as an `Administrator`. Press `Windows key + R`, type in `%USERPROFILE%` and click on `View` in the top bar of the `Explorer` and tick the checkbox `Hidden items`. Copy everything to your new profile and overwrite everything and skip some files when prompted. If you need to migrate many accounts you can use the [User State Migration Tool](https://learn.microsoft.com/en-us/windows/deployment/usmt/usmt-overview).

Note: If you need to log in with the local user account, you can do so by providing the username as `<Computername>/<local user name>`.

## Backup again

If everything is working fine, you'll back up your system again.
`C:\Windows\System32\wbadmin.exe START SYSTEMSTATEBACKUP -backupTarget:<DRIVELETTER>:`
