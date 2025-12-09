---
layout: post
title: "PortScan"
date: 2025-11-30 18:30:00 -0300
categories: [Proyectos-Herramientas,PortScan]
tags: [Proyectos, PortScan]
---

Proyecto: portScan.sh - Escáner Básico de Puertos

🎯 Descripción del Proyecto
portScan.sh es una herramienta de scripting minimalista escrita íntegramente en Bash, diseñada para realizar un escaneo rápido y asíncrono de todos los puertos TCP (del 1 al 65535) en una dirección IP objetivo.

El objetivo principal de este script es demostrar el concepto fundamental del escaneo de puertos utilizando únicamente las capacidades y redirecciones nativas del sistema operativo Linux, sin depender de herramientas externas como Nmap o Netcat. Es una prueba de concepto para la automatización de tareas básicas de reconocimiento.

⚙️ ¿Cómo Funciona el Script?
El script utiliza una técnica conocida como "conexión ciega" a través del dispositivo especial /dev/tcp.

Validación de Uso:

Comienza verificando si el usuario proporcionó una dirección IP como argumento (if [ $1 ]; then). Si no lo hizo, muestra el mensaje de uso y sale.

Bucle Completo:

Si se proporciona la IP, inicia un bucle (for port in $(seq 1 65535); do...) que intenta probar todos los números de puerto, desde el 1 hasta el 65535.

Intento de Conexión Asíncrona (El Núcleo):

timeout 1: Establece un límite de tiempo de 1 segundo para la conexión. Esto evita que el script se quede colgado si un puerto no responde.

bash -c "echo '' > /dev/tcp/$ip_address/$port": Este es el truco de Bash. Intenta abrir una conexión TCP a la IP y el puerto especificados. Si el puerto está abierto, la conexión se establece.

2>/dev/null: Descarta todos los mensajes de error (stderr) generados por el sistema, que normalmente indican que el puerto está cerrado o es inalcanzable.

&& echo "[+] Port $port - OPEN": Si la conexión es exitosa (&&), solo entonces imprime el mensaje indicando que el puerto está abierto.

&: Esta es la clave de la velocidad. Este símbolo ejecuta el proceso de escaneo de cada puerto en segundo plano (de forma asíncrona), permitiendo que miles de intentos de conexión se realicen simultáneamente.

Espera Final:

wait: Una vez que el bucle de puertos ha terminado de enviar todos los intentos, el comando wait pausa la ejecución del script hasta que todos los procesos en segundo plano (las 65535 conexiones) finalicen, asegurando que todos los resultados de los puertos abiertos se muestren.


SCRIPT:

#!/bin/bash

# Uso:
#   ./portScan.sh <ip-address>

if [ $1 ]; then
    ip_address=$1

    for port in $(seq 1 65535); do
        timeout 1 bash -c "echo '' > /dev/tcp/$ip_address/$port" 2>/dev/null \
        && echo "[+] Port $port - OPEN" &
    done
    wait

else
    echo -e "\n[*] Uso: ./portScan.sh <ip-address>\n"
    exit 1
fi

COMO SE USA:
Primero que nada le damos permisos: chmod u+x ./portScan.sh
Luego escribimos en la terminal ./portScan.sh <ip>

