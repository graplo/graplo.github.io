---
title: "Helix -  Salto De Linea"
date: 2026-03-29T23:36:16-05:00
draft: false
summary: "Evitar el Desplazamiento Horizontal"
tags: ["linux", "rust", "Open Source", "Editor de texto"]
categories: ["software"]
---

Helix es un editor de texto modal y moderno para terminal, escrito en Rust, diseñado para ser rápido y eficiente.

<!--more-->

## Objetivo

Evitar el desplazamiento horizontal (scroll lateral) y mejorar la lectura de líneas largas en el editor, haciendo que el texto "salte" automáticamente al borde de la ventana.

## Configuración aplicada en `~/.config/helix/config.toml`

```toml
[editor]

# Activa el ajuste de línea automático
soft-wrap.enable = true

# Define el carácter que aparece al inicio de la línea de continuación
soft-wrap.wrap-indicator = "↪ "

# Mantiene la sangría de la línea original en el salto
soft-wrap.wrap-at-text-width = true

text-width = 110
```
text-width = 110

> Puedes cambiar 'text-width = 110' al valor q te resulte comodo 
