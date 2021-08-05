### SMB

- Commands
	
	```
	smbclient -N -L 10.10.10.27
		Sharename       Type      Comment
		---------       ----      -------
		ADMIN$          Disk      Remote Admin
		backups         Disk
		C$              Disk      Default share
		IPC$            IPC       Remote IPC
	```
	
	```
	smbclient -N \\\\10.10.10.27\\backups
		smb: \> dir
			.                                   D        0  Mon Jan 20 04:20:57 2020
			..                                  D        0  Mon Jan 20 04:20:57 2020
			prod.dtsConfig                     AR      609  Mon Jan 20 04:23:02 2020
	```
	
- Only backups and IPC$ folders are mountable. All other files lack permissions
