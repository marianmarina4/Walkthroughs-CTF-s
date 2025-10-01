# ColddBox — Writeup

**Fecha:** 1-09-2025\
**Plataforma:** TryHackMe

## Enumeration (nmap)

```bash
nmap -sV -sC --open $IP -oN scan.txt
```

**Comment:** `-sV` detecta versiones de servicios; `-sC` ejecuta scripts por defecto; `--open` muestra solo los puertos abiertos; `-oN` guarda la salida en un archivo normal.

**Result**
```bash
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.1.31
|_http-title: ColddBox | One more machine
|_http-server-header: Apache/2.4.18 (Ubuntu)
```

**Summary**
* 80/tcp open http (Apache/2.4.18)

---
## Web Enumeration (Gobuster)

```bash
  gobuster dir -u http://$IP -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt
```

## Users Enumeration

```bash
  wpscan --url http://$IP --enumerate u 
```

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

```
## Foothold
**Reverse Shell**

**Upload Script**

**Listen Port**

**Upgrade Shell**
