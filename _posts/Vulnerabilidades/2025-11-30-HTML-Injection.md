---
layout: post
title: "Cross-Site-Scripting"
date: 2025-11-30 18:30:00 -0300
categories: [Vulnerabilidad,XSS]
tags: [Vulnerabilidad, XSS, Web]
description: "Vulnerabilidad XSS (Cross Site Scripting)"
---
## Resumen de la Vulnerabilidad

Cross-Site Scripting (XSS) es una vulnerabilidad web que permite a un atacante inyectar y ejecutar código JavaScript malicioso en el navegador de otros usuarios.

El ataque no compromete directamente el servidor, sino a los usuarios que visitan la aplicación vulnerable.

## ¿Qué puede hacer un atacante con XSS?

Un ataque XSS exitoso puede permitir:

- Robar cookies de sesión

- Secuestrar cuentas de usuario

- Realizar acciones en nombre de la víctima

- Redirigir a sitios maliciosos

- Registrar pulsaciones del teclado (keylogging)

- Mostrar contenido falso (phishing)


## ¿Por qué ocurre XSS?

XSS ocurre cuando una aplicación web:

- Acepta datos del usuario

- No los valida ni escapa correctamente

- Los muestra directamente en el navegador

Ejemplo vulnerable:


```bash
<p>Bienvenido, <?php echo $_GET['name']; ?></p>
```

## TIPOS DE XSS

##1 XSS Reflejado (Reflected XSS)

El payload se envía en la petición HTTP y se refleja en la respuesta.

Ejemplo:

```bash
https://victima.com/buscar?q=<script>alert(1)</script>
```

**Características:**

- No se almacena en el servidor

- Requiere que la víctima haga clic en un enlace

- Muy común en parámetros GET


##2 XSS Almacenado (Stored XSS)

El código malicioso se guarda en la base de datos y se ejecuta cada vez que alguien visita la página.

Ejemplos comunes:

- Comentarios

- Foros

- Perfiles de usuario

Payload típico:

```bash 
<script>alert('XSS')</script> 
```

**Características:**

- Más peligroso

- Afecta a múltiples usuarios

- Persistente


##3 XSS Basado en DOM (DOM-Based XSS)

Ocurre completamente en el navegador del usuario.

Ejemplo vulnerable:


```bash 
document.getElementById("msg").innerHTML = location.hash;
```
**Características:**

- No pasa por el servidor

- Difícil de detectar

- Depende del JavaScript del cliente

## Ejemplos de payloads comunes


```bash 
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
"><script>alert(1)</script>
```

## Cómo prevenir XSS

**Buenas prácticas**

- Escapar correctamente la salida (HTML encoding)

- Validar y sanear entradas del usuario

- Usar Content-Security-Policy (CSP)

- Evitar innerHTML, usar textContent

- Cookies con HttpOnly y Secure

Ejemplo seguro:


```bash 
echo htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
```
## Cómo detectar XSS en pentesting

- Probar entradas reflejadas

- Revisar parámetros GET y POST

- Analizar JavaScript del cliente

Usar herramientas como:

- Burp Suite

- OWASP ZAP

- XSStrike


## Clasificación de la vulnerabilidad

**Tipo:** Cross-Site Scripting (XSS)

**OWASP Top 10:** Injection/Cross-Site Scripting (XSS) (A03:2021 – Injection)

**Severidad:** Media a Alta

Crítica si permite:

Robo de sesión

Account Takeover

Ejecución persistente (Stored XSS)

Acciones en nombre del usuario


## Conclusión

La vulnerabilidad Cross-Site Scripting (XSS) representa un riesgo significativo para la seguridad de las aplicaciones web, ya que permite a un atacante ejecutar código malicioso en el navegador de los usuarios afectados. Aunque el servidor no sea comprometido directamente, el impacto sobre la confidencialidad, integridad y confianza del usuario puede ser grave.
