---
tags:
  - Cybersecurity
  - Labs
  - TryHackMe
  - Easy
  - HashIdentifier
  - JohnTheRipper
  - Hashcat
---
- [[#Level 1|Level 1]]
	- [[#Level 1#Hash 1 - MD5|Hash 1 - MD5]]
	- [[#Level 1#Hash 2 - SHA-1|Hash 2 - SHA-1]]
	- [[#Level 1#Hash 3 - SHA-256|Hash 3 - SHA-256]]
	- [[#Level 1#Hash 4 - Bcrypt|Hash 4 - Bcrypt]]
	- [[#Level 1#Hash 5 - MD4|Hash 5 - MD4]]
- [[#Level 2|Level 2]]
	- [[#Level 2#Hash 1 - SHA-256|Hash 1 - SHA-256]]
	- [[#Level 2#Hash 2 - NTLM|Hash 2 - NTLM]]
	- [[#Level 2#Hash 3 - SHA512|Hash 3 - SHA512]]
	- [[#Level 2#Hash 4 -  SHA-1 (HMAC)|Hash 4 -  SHA-1 (HMAC)]]


---
# Resolviendo la máquina Crack the hash

>En esta publicación, comparto cómo resolví la máquina **Crack the hash** de [TryHackMe](https://tryhackme.com/room/crackthehash).

---
## Level 1
### Hash 1 - MD5

**Hash:** `48bb6e862e54f2a795ffc4e541caed4d`.

Se usa la herramienta `hash-identifier` para averiguar el *hash*.

![[Pasted image 20250804175544.png]]

Se usa `john` para realizar fuerza bruta.

`vim md5_hash.txt`

`john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt md5_hash.txt`

![[Pasted image 20250804175757.png]]

`john --show --format=raw-md5 md5_hash.txt`

![[Pasted image 20250804175806.png]]
### Hash 2 - SHA-1

**Hash:** `CBFDAC6008F9CAB4083784CBD1874F76618D2A97`.

Se usa la herramienta `hash-identifier` para averiguar el *hash*.

![[Pasted image 20250804175847.png]]

Se usa `john` para realizar fuerza bruta.

`vim sha-1_hash.txt`

`john --format=raw-sha1 --wordlist=/usr/share/wordlists/rockyou.txt sha-1_hash.txt`

![[Pasted image 20250804180258.png]]
### Hash 3 - SHA-256

**Hash:** `1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032`.

Se usa la herramienta `hash-identifier` para averiguar el *hash*.

![[Pasted image 20250804180341.png]]

Se usa `john` para realizar fuerza bruta.

`vim sha256_hash.txt`

`john --format=raw-sha256 --wordlist=/usr/share/wordlists/rockyou.txt sha256_hash.txt`

![[Pasted image 20250804180632.png]]

`john --show --format=raw-sha256 sha256_hash.txt`

![[Pasted image 20250804180643.png]]
### Hash 4 - Bcrypt

**Hash:** `$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom`.

Se usa la herramienta `hash-identifier` para averiguar el *hash*.

![[Pasted image 20250804180743.png]]

No se identifica ningún *hash*

Se identifica el *hash* con [Cipher Identifier](https://www.dcode.fr/cipher-identifier).

![[Pasted image 20250804180937.png]]

Se usa `john` para realizar fuerza bruta.

`vim bcrypt_hash.txt`

`john --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt --fork=4 bcrypt_hash.txt`

Este *hash* tarda bastante.

`bleh`
### Hash 5 - MD4

**Hash:** `279412f945939ba78ce0758d3fd83daa`.

Se usa la herramienta `hash-identifier` para averiguar el *hash*.

![[Pasted image 20250804181350.png]]

Se utiliza [MD5 Hashing](https://md5hashing.net/) para desencriptarlo.

![[Pasted image 20250804182211.png]]
## Level 2
### Hash 1 - SHA-256

**Hash:** `F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85`.

Se usa la herramienta `hash-identifier` para averiguar el *hash*.

![[Pasted image 20250804183007.png]]

`vim sha256_hash.txt`

`john --format=raw-sha256 --wordlist=/usr/share/wordlists/rockyou.txt sha256_hash.txt`

![[Pasted image 20250804183315.png]]
### Hash 2 - NTLM

**Hash:** `1DFECA0C002AE40B8619ECF94819CC1B`.

Se usa la herramienta `hash-identifier` para averiguar el *hash*.

![[Pasted image 20250804183349.png]]

Se usa `john` para realizar fuerza bruta.

`vim ntlm_hash.txt`

`john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt ntlm_hash.txt `

![[Pasted image 20250804183804.png]]
### Hash 3 - SHA512

**Hash:** `$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02`.

Se usa la herramienta `hash-identifier` para averiguar el *hash*.

![[Pasted image 20250804183922.png]]

Se identifica el *hash* es **SHA512**.

Este *hash* tarda bastante.

`vim SHA-512_hash.txt`

`hashcat -m 1800 SHA-512_hash.txt /usr/share/wordlists/rockyou.txt --session sha512`

![[Pasted image 20250805181115.png]]
### Hash 4 -  SHA-1 (HMAC)

**Hash:** `e5d8870e5bdd26602cab8dbe07a942c8669e56d6`.

Se usa la herramienta `hash-identifier` para averiguar el *hash*.

![[Pasted image 20250804184047.png]]

`hashcat -m 1100 SHA-1_hash.txt /usr/share/wordlists/rockyou.txt`

Este *hash* tarda bastante.

`481616481616`