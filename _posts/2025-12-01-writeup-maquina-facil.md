---
layout: post
title: "Writeup: [Nombre de la Máquina] - Fácil"
date: 2025-12-01 10:00:00 -0300
categories: [Writeups/Hack The Box/Fáciles]
tags: [HTB, linux, web, fácil]
---

# 🎯 Objetivo: [Nombre de la Máquina]

Este documento detalla la explotación y escalada de privilegios en la máquina **[Nombre de la Máquina]** de Hack The Box.

## ⚙️ Reconocimiento (Nmap)

Comenzamos con un escaneo de puertos para identificar servicios abiertos.

```bash
# Comando usado para escanear
sudo nmap -sC -sV [IP de la máquina]
