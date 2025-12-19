---
layout: post
title: "SQL Injection"
date: 2025-11-30 18:30:00 -0300
categories: [Vulnerabilidad,SQL]
tags: [Vulnerabilidad, SQL, Web]
description: "Vulnerabilidad SQL Injection"
---

## Resumen de la Vulnerabilidad

SQL Injection (SQLi) es una vulnerabilidad web que permite a un atacante manipular consultas SQL enviadas a la base de datos de una aplicación, mediante la inyección de código malicioso a través de entradas del usuario.

Este ataque puede provocar desde la exposición de información sensible hasta la toma total del sistema, dependiendo del impacto.

## ¿Qué puede hacer un atacante con SQL Injection?

Un ataque SQLi exitoso puede permitir:

- Leer información sensible (usuarios, contraseñas, tarjetas)

- Modificar o eliminar datos

- Bypassear autenticación

- Obtener credenciales de administrador

- Ejecutar comandos en el servidor (en casos críticos)

## ¿Por qué ocurre SQL Injection?

SQL Injection ocurre cuando:

- La aplicación construye consultas SQL dinámicas

- Usa directamente entradas del usuario

- No valida ni sanitiza los datos

Ejemplo vulnerable:


```bash
SELECT * FROM users WHERE username = '$user' AND password = '$pass';
```

## Tipos de SQL Injection

## SQL Injection Clásico (In-Band)

El atacante obtiene la respuesta directamente en la aplicación.

Ejemplo:


```bash
' OR 1=1--
```
Subtipos:

- Error-Based

- Union-Based

## SQL Injection Ciego (Blind SQLi)

No se muestran errores ni resultados, pero se puede inferir información.

Ejemplos:

- Boolean-Based

- Time-Based

Payload ejemplo:


```bash
' OR IF(1=1, SLEEP(5), 0)--
```

## SQL Injection Fuera de Banda (Out-of-Band)

tiliza canales externos (DNS, HTTP) para exfiltrar datos.

Menos común, pero muy potente.

## Payloads comunes


```bash
' OR '1'='1'--
' UNION SELECT null, username, password FROM users--
' OR SLEEP(5)--
" OR "1"="1"--
```

## Cómo prevenir SQL Injection

**Buenas prácticas**

- Usar consultas preparadas (Prepared Statements)

- Emplear ORM seguros

- Validar y sanitizar entradas

- Principio de mínimos privilegios en la BD

- No mostrar errores SQL

Ejemplo seguro:


```bash
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$user]);
```

## Cómo detectar SQL Injection en pentesting

- Probar payloads simples (', ", --)

- Analizar errores SQL

- Probar delays (Time-Based)

Usar herramientas como:

- SQLMap

- Burp Suite

- OWASP ZAP

## Severidad

- Alta

- Crítica si permite acceso total a la base de datos o RCE

## Conclusión

La vulnerabilidad SQL Injection (SQLi) es una de las fallas de seguridad más críticas en aplicaciones web, ya que permite a un atacante interactuar directamente con la base de datos subyacente. A través de este tipo de ataque, es posible acceder, modificar o eliminar información sensible, comprometiendo gravemente la confidencialidad, integridad y disponibilidad de los datos.
