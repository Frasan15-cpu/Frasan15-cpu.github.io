---
layout: post
title: "Detectar_os"
date: 2025-11-30 18:30:00 -0300
categories: [Proyectos-Herramientas,Detectar_os]
tags: [Proyectos, Detectar_os]
---

Script de Reconocimiento – Detección de Sistema Operativo por TTL
Este script permite identificar el sistema operativo de un host remoto utilizando su valor TTL en la respuesta del comando ping.
 La técnica funciona porque distintos sistemas operativos establecen valores TTL iniciales diferentes:
Linux suele usar valores entre 0 y 64
Windows utiliza valores entre 65 y 128
El script realiza los siguientes pasos:
Recibe una dirección IP como argumento.
Ejecuta un ping hacia esa IP usando subprocess.
Extrae el valor TTL de la respuesta utilizando expresiones regulares.
Clasifica el sistema operativo según el TTL obtenido.
Imprime la dirección IP, el TTL detectado y el sistema operativo estimado.
Es útil durante la fase de reconocimiento para tener una idea inicial del tipo de sistema al que estás apuntando.

SCRIPT:

```bash
#!/usr/bin/python3
#coding: utf-8

import re, sys, subprocess

if len(sys.argv) != 2:
    print("\n[*] Uso: python3 " + sys.argv[0] + " <dirección-ip>\n")
    sys.exit(1)

def get_ttl(ip_address):

    proc = subprocess.Popen(["/usr/bin/ping -c 1 %s" % (ip_address, "")], stdout=subprocess.PIPE, shell=True)
    (out, err) = proc.communicate()

    out = out.split()
    out = out[12].decode('utf-8')

    ttl_value = re.findall(r"\d{1,3}", out)[0]

    return ttl_value

def get_os(ttl):

    ttl = int(ttl)

    if ttl >= 0 and ttl <= 64:
        return "Linux"
    elif ttl >= 65 and ttl <= 128:
        return "Windows"
    else:
        return "Not Found"

if __name__ == "__main__":

    ip_address = sys.argv[1]

    ttl = get_ttl(ip_address)

    os_name = get_os(ttl)

    print("\n%s (ttl -> %s): %s" % (ip_address, ttl, os_name))

```
