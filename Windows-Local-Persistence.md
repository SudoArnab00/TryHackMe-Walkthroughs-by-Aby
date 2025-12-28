## Room : [Windows Local Persistence](https://tryhackme.com/room/windowslocalpersistence)

### Tampering with unprivileged accounts
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
7. 