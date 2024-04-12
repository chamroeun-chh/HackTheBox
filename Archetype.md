# HTB — Archetype: Hacking Without Reverse Shell

# Enumeration

It all starts with gathering information about the target. We used a tool called **nmap** to scan the target’s IP address like this:

```
nmap {target IP} -Pn -sV
```

![](https://miro.medium.com/v2/resize:fit:875/1*9v4DM-bhzkemNxZ4JStEPQ.png)

Our scan showed that a few doors were open on the target — RPC, SMB, and MSSQL. We decided to focus on the SMB door first and used **smbclient** to explore it:

```
smbclient -L {target IP}
```

![](https://miro.medium.com/v2/resize:fit:875/1*8eI93EPMrCbOO2_7VSV5LQ.png)

We tried to access the ADMIN$ and C$ shares, but we were denied access. Fortunately, we found an accessible share called “backups,” and we started our exploration from there:

```
smbclient \\\\10.129.20.146\\backups - no-pass
```

![](https://miro.medium.com/v2/resize:fit:875/1*eIWhjCt4q-3yKi7G1ADBxw.png)

In this share, we stumbled upon a file named ‘prod.dtsConfig.’ It seemed like a valuable configuration file, so we downloaded it to our local machine using the ‘get’ command.

![](https://miro.medium.com/v2/resize:fit:875/1*S2hPP2nEYn_gtVBy9BH5ag.png)

Upon examining the configuration file, we discovered the login credentials for a user named ‘sql_svc.’ These credentials included the username ‘sql_svc,’ the hostname ‘Archetype,’ and the password ‘M3g4c0rp123.’ We planned to make use of these credentials by using a Python script called ‘mssqlclient’ from the Impacket toolkit:

```
./mssqlclient.py -windows-auth ARCHETYPE/sql_svc@10.129.20.146
```

![](https://miro.medium.com/v2/resize:fit:875/1*BWbz5Jl-1leXMdcxxD5YWQ.png)

We successfully logged into the Microsoft SQL Server and started exploring its features by using the ‘help’ command. Our main goal was to execute commands from within the MSSQL server, so we enabled the ‘xp_cmdshell’ feature:

![](https://miro.medium.com/v2/resize:fit:875/1*21n86H5oq77JO7Bp2PuzMw.png)

```
xp_cmdshell  
exec sp_configure 'show advanced option'  
exec sp_configure 'show advanced option',1  
reconfigure  
exec sp_configure 'xp_cmdshell',1  
reconfigure
```

![](https://miro.medium.com/v2/resize:fit:875/1*Aj-TxQl5SDbtwepgyg1Vbw.png)

Enabling ‘xp_cmdshell’ in the SQL Server gave us access to this handy feature. With it enabled, we looked around the file directory and found the user flag:
```
xp_cmdshell (powershell -c type C:/Users/sql_svc/Desktop/user.==txt==)
```

![](https://miro.medium.com/v2/resize:fit:875/1*O7gWibMtlP1dsE0BhNq-lQ.png)

> Here’s the interesting part — we managed to get the admin’s credentials without using a reverse shell by running this command:

```
xp_cmdshell "powershell -c cat (Get-PSReadlineOption).HistorySavePath"
```

![](https://miro.medium.com/v2/resize:fit:875/1*acgBznbyDIVbqz6c1b60vQ.png)

Now that we had the admin’s credentials, we used a tool called ‘psexec.py’ from the Impacket toolkit to gain access to the admin’s shell and got the root flag :)

```
./psexec.py administrator@10.129.20.146
```

![](https://miro.medium.com/v2/resize:fit:875/1*74Vh6BReOzAcc_kuSdkbVw.png)

**Summary**

In this lab, I successfully utilized a shortcut method that didn’t require the use of a reverse shell. 🚀 This approach allowed for efficient and effective execution of tasks, and I hope that my contribution benefits the community in some way. 🙌