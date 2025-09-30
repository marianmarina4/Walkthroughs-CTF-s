
## Enumeración (nmap)

```bash
nmap -sV -sC --open $IP -oN scan.txt
```

**Comentario:** `-sV` detecta versiones de servicios; `-sC` ejecuta scripts por defecto; `--open` muestra solo los puertos abiertos; `-oN` guarda la salida en un archivo normal.

**Resultado**
```bash
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.1.31
|_http-title: ColddBox | One more machine
|_http-server-header: Apache/2.4.18 (Ubuntu)
```

**Resumen**
* 80/tcp open http (Apache/2.4.18)

---

  gobuster dir -u http:/$IP -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt
