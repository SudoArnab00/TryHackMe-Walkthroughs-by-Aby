## Room : [Windows Local Persistence - Backdooring RDP](https://tryhackme.com/room/windowslocalpersistence)

### With Sticky keys
We get the annoying sticky keys by pressing **shift** 5times.

To establish persistence using Sticky Keys, we will abuse a shortcut enabled by default in any Windows installation.

After pressing SHIFT 5 times, Windows will execute the binary in C:\Windows\System32\sethc.exe.

We will replace **sethc.exe** with a copy of **cmd.exe**

- To overwrite **sethc.exe**, we need to take ownership of the file and grant our current user permission to modify it - then we can replace it.

`takeown /f c:\Windows\System32\sethc.exe`

![WLP45](/img/WLP45.png)

- When user logs out or locked out, from lock screen you can press **shift** 5 times and you get the terminal.

![WLP46](/img/WLP46.png)

- Retrieve the `flag14.exe`

### With Utilman
Utilman is a built-in Windows application used to provide Ease of Access options during the lock screen. Bottom right of the screen beside the Power Button.
It executes **C:\Windows\System32\Utilman.exe** with SYSTEM privileges. 
We will replace Utilman.exe with a copy of our cmd.exe

-  To overwrite **utilman.exe**, we need to take ownership of the file and grant our current user permission to modify it - then we can replace it.

`takeown /f c:\Windows\System32\utilman.exe`

![WLP47](/img/WLP47.png)

- When user logs out or locked out, from lock screen you can press **shift** 5 times and you get the terminal.

![WLP48](/img/WLP48.png)

- Retrieve the `flag15.exe`