---
layout: post
title: "Blue"
date: 2025-11-30 18:30:00 -0300
categories: [Writeups,Blue]
tags: [Htb, Blue, Windows]
description: "Writeup de la máquina Blue de Hackthebox"
image: /assets/images/hackthebox.png
---
## Resumen de la resolución

**Blue** es una maquina **Windows** de dificultad **Easy** de la plataforma **HackTheBox**. Blue es una máquina vulnerada por el famoso exploit EternalBlue (MS17-010).
Es una de las más clásicas y perfectas para entender explotación de SMB. 

---
## Enumeración

En primer lugar, debemos desplegar la máquina para poder obtener la **Dirección IP** todo ello desde la web de **HackTheBox** y luego desde la **terminal** debemos conectarnos a la VPN usando el fichero correspondiente de la siguiente forma:


```bash
openvpn lab_Frasan15.opvn
```

Luego le lanzamos un **ping** para comprobar si la maquina esta activa. Verificamos que responde correctamente devolviendo el paquete enviado. A continuacion, utilizamos un script propio que, a partir del valor del  **ttl**  permite identificar el tipo de sistema operativo: **Linux (TTL 64 )** o **Windows (TTL 128)**, en este caso, observamos un **TTL** próximo a 128 (**128**) por lo que se trata de una máquina **Windows**

> El motivo por el cual el **TTL** es de **127** es porque el paquete pasa por unos intermediarios (routers) antes de llegar a su destino (máquina atacante). Esto podemos comprobarlo con el comando `ping -c 1 -R 10.129.24.16`.

### Nmap

En segundo lugar, realizaremos un escaneo por **TCP** usando **Nmap** para ver que puertos de la máquina víctima se encuentra abiertos.

```bash
nmap 10.129.24.16 -p- --open -T5 -v -n -oG allPorts
```

Observamos como nos reporta que se encuentran abiertos un montón de puertos.

Ahora, gracias a la utilidad **extractPorts** definida en nuestra **.zshrc** podremos copiarnos cómodamente todos los puerto abiertos de la máquina víctima a nuestra **clipboard**.

A continuación, volveremos a realizar un escaneo con **Nmap**, pero esta vez se trata de un escaneo más exhaustivo pues lanzaremos unos script básicos de reconocimiento, además de que nos intente reportar la versión y servicio que corre para cada puerto.


```bash
nmap -sC -sV -p21,22,139,445,3632 10.129.24.16 -oN targeted
```
En el segundo escaneo de **Nmap**, lo primero que llama la atención es que la salida SMB dice que es **Windows 7 Professional**.

![](/assets/images/bluenmap.png)

___
### Puerto 445 (SMB)

Como ya sabemos nmap tiene vulnscripts que buscan vulnerabilidades conocidad dentro del servicio por lo que lo ejecutamos y encontramos una vulnerabilidad **MS-17-010**


```bash
 nmap -p 445 -script vuln -oA scans/nmap-smbvulns 10.129.24.16
```

![](/assets/images/NMAPVULN.png)

### Explotacion de vulnerabilidad

MS-17-010 es una vulnerabilidad conocida cono **eternalblue**, es una vulnerabilidad de ejecución de código remoto no autenticado en Windows SMB. Para explotar dicha vulnerabilidad utilizaremos metaexploit.

El primer paso es abrir la consola con el siguiente comando:


```bash
msfconsole
```
Una vez con la consola abierta buscamos el exploit de esta forma:

```bash
search ms17-010
```
![](/assets/images/buscamosexploir.png)

Al ver todos los exploits elegimos el adecuado, lo configuramos y lo lanzamos.

![](/assets/images/configyrun.png)

### Explotacion exitosa

Una vez que corremos el exploit logramos ver que nos otorga acceso a la consola.
Al estar dentro de la consola con el comando **dir** vemos las carpetas que se encuentran en el sistema.

Luego con el comando siguiente obtendremos la flag de usuario:


```bash
type user.txt
```
Y para finalizar y obtener la flag de root utilizamos el siguiente comando:


```bash
type administratot\desktop\root.txt
```
La máquina Blue me pareció una de las más accesibles y perfectas para quienes están comenzando en el pentesting real. La explotación fue directa gracias a la vulnerabilidad EternalBlue (MS17-010), lo que permitió comprender muy bien cómo funciona un exploit real a nivel de SMB y memoria.
















