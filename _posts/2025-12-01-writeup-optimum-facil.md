---
layout: post
title: "Optimum: Laboratorio Resuelto (HTB)"
date: 2025-12-01 10:00:00 -0300
categories: [WRITEUPS/Hack The Box/Fáciles/Optimum] # Nivel 1, 2, 3 y 4 (Máquina)
tags: [HTB, Optimum, fácil]
---

# 🎯 Objetivo: [Nombre de la Máquina]

Este documento detalla la explotación y escalada de privilegios en la máquina **[Nombre de la Máquina]** de Hack The Box.

## ⚙️ Reconocimiento (Nmap)

Comenzamos con un escaneo de puertos para identificar servicios abiertos.

```bash
# Comando usado para escanear
sudo nmap -sC -sV [IP de la máquina]
