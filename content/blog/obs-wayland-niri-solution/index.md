---
title: "Cambiar escenas de OBS con Niri"
date: 2026-05-03T21:47:08-05:00
draft: false
summary: "En Wayland los atajos globales no funcionan como en X11. Con gobs-cli y el WebSocket de OBS podemos cambiar escenas desde cualquier app, sin tener OBS en foco."
description: "Cómo configurar atajos globales para cambiar escenas de OBS en niri usando gobs-cli y el WebSocket de OBS."
tags: ["linux", "obs", "niri", "wayland", "cli", "streaming"]
categories: ["Terminal"]
---

En X11 cualquier app podía capturar atajos de teclado globalmente. En Wayland esto cambió por razones de seguridad — cada app solo recibe eventos de teclado cuando está en foco. Esto afecta directamente a OBS: si estás grabando y tienes otra app en foco, los atajos de escena de OBS no funcionan.

Esto aplica a compositors tiling como niri o Hyprland. Entornos como GNOME o Plasma tienen su propio sistema de atajos globales integrado y no tienen este problema.

La solución es usar el WebSocket de OBS junto con `gobs-cli` — una herramienta CLI que controla OBS remotamente — y configurar los atajos directamente en el compositor.

## Instalar gobs-cli

```bash
paru -S gobs-cli-bin
```

> Hay otros paquetes similares como `obs-cli` pero usan el protocolo pre-5.0.0 que no es compatible con las versiones actuales de OBS. `gobs-cli` soporta WebSocket v5.

## Activar el WebSocket en OBS

En OBS ve a **Herramientas → Configuración del servidor WebSocket** y activa:

- Habilitar servidor WebSocket
- Habilitar autenticación

El puerto por defecto es `4455`. Al hacer click en **Mostrar información de conexión** verás tu IP local, puerto y contraseña.

> La IP que muestra OBS es tu IP local de red — solo accesible dentro de tu red doméstica, no desde internet.

## Probar la conexión

```bash
gobs-cli -p "tu-contraseña" sc sw "nombre de tu escena"
# Sustituye "tu-contraseña" por la contraseña del WebSocket
# Sustituye "nombre de tu escena" por el nombre exacto de tu escena en OBS
```

El nombre de la escena debe coincidir exactamente con el que aparece en OBS, incluyendo mayúsculas y espacios.

Si la conexión funciona OBS cambiará a esa escena inmediatamente.

## Configurar los atajos en niri

Los atajos globales en niri van en `~/.config/niri/config.kdl` o en el archivo de binds de tu configuración. En mi caso uso DankMaterialShell que los gestiona en `~/.config/niri/dms/binds.kdl`.

Agrega dentro del bloque `binds {}`:

```kdl
F1 hotkey-overlay-title="OBS: Full Camera" { spawn "sh" "-c" "gobs-cli -p \"tu-contraseña\" sc sw \"1 Full camera\""; }
F2 hotkey-overlay-title="OBS: Escritorio y cam" { spawn "sh" "-c" "gobs-cli -p \"tu-contraseña\" sc sw \"2 Escritorio y cam\""; }
F3 hotkey-overlay-title="OBS: Cam Circular" { spawn "sh" "-c" "gobs-cli -p \"tu-contraseña\" sc sw \"3 Escena\""; }
```

> Sustituye `tu-contraseña` por tu contraseña real del WebSocket. Sustituye los nombres de escena por los tuyos. Las teclas F1, F2, F3 también puedes cambiarlas por cualquier combinación que prefieras.

Si usas Hyprland la sintaxis del atajo es diferente pero el comando `gobs-cli` es exactamente el mismo

## Resultado

Desde cualquier app, sin importar cuál esté en foco, F1, F2 y F3 cambian la escena activa en OBS al instante.
