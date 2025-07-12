---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - HTTP
  - JohnTheRipper
  - MD5
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
- [[#Explotación|Explotación]]
	- [[#Explotación#HTTP|HTTP]]
	- [[#Explotación#John The Ripper|John The Ripper]]

---
# Resolviendo la máquina Corridor

>En esta publicación, comparto cómo resolví la máquina [Corridor](https://tryhackme.com/room/corridor) de [TryHackMe](https://tryhackme.com/).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 10.10.119.85`

![[Pasted image 20250712190702.png]]

*TTL=63* -> **Linux**
### Nmap

Escaneo inicial de puertos.

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.119.85 -oG allPorts`

![[Pasted image 20250712191030.png]]

`nmap -p80 -sCV 10.10.119.85 -oN targeted`

![[Pasted image 20250712191211.png]]

---
## Explotación
### HTTP

Al analizar el sitio web, observamos que los títulos parecen estar codificados en **MD5**.

![[Pasted image 20250712193608.png]]
### John The Ripper

Utilizamos **John the Ripper** para descifrar uno de los hashes.

```
echo "c4ca4238a0b923820dcc509a6f75849b" > hash
```

```
john --format=Raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt hash
```

![[Pasted image 20250712193906.png]]

El hash corresponde al número **1**.

Descifrando todos los títulos, obtenemos una secuencia del **1 al 13**.

Para probar con el **número 0**, generamos su hash MD5 manualmente.

`echo -n "0" | md5sum`

![[Pasted image 20250712194102.png]]

`http://10.10.119.85/cfcd208495d565ef66e7dff9f98764da`

Se encuentra la flag.

---