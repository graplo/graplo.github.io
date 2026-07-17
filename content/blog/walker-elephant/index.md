---
title: "Walker Elephant"
date: 2026-07-07T10:54:47-05:00
draft: true
summary: ""
description: ""
tags: []
categories: []
---

## Intalar walker + Elephant
```bash
sudo pacman -S walker
paru -S 
```

## Servicios creados en ~/.config/systemd/user
Elephant (backend de Walker)

### Archivo:

```bash
~/.config/systemd/user/elephant.service
```

### Contenido:

```bash
[Unit]
Description=Elephant
After=graphical-session.target

[Service]
Type=simple
ExecStart=elephant
Restart=on-failure

[Install]
WantedBy=graphical-session.target
```
¿Qué hace?

Elephant es el motor de búsqueda. Mantiene los providers activos:

desktopapplications → aplicaciones
files → archivos
clipboard → portapapeles
calc → calculadora
archlinuxpkgs → paquetes Arch/CachyOS
wireplumber → audio
playerctl → multimedia
bookmarks
websearch
etc.

No tiene interfaz gráfica.

### Activación:

Se creó con:

```bash
elephant service enable
```
y se inició con:

```bash
systemctl --user start elephant.service
```
Estado:

```bash
systemctl --user status elephant.service
```
Debe quedar: Bueno parecido este es mi sistema

```bash
● elephant.service - Elephant
     Loaded: loaded (/home/graplo/.config/systemd/user/elephant.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-07-07 10:09:06 -05; 57min ago
 Invocation: 61130d5954f043378e023c93421099f1
   Main PID: 117503 (elephant)
      Tasks: 23 (limit: 38322)
     Memory: 314.8M (peak: 720.1M)
        CPU: 5.755s
     CGroup: /user.slice/user-1000.slice/user@1000.service/app.slice/elephant.service
             ├─117503 elephant
             └─117568 wl-paste --watch echo clipboard-changed
```
## Después de crear Walker

### Archivo:

```bash
~/.config/systemd/user/walker.service
```
### Contenido recomendado:

```bash
[Unit]
Description=Walker Launcher Service
After=graphical-session.target

[Service]
Type=simple
ExecStart=/usr/bin/walker --gapplication-service
Restart=on-failure

[Install]
WantedBy=graphical-session.target
```
¿Qué hace?

Walker queda cargado en memoria esperando órdenes.

Cuando ejecutas:

```bash
walker
```
o: Por ejemplo para portapapeles

```bash
walker -m clipboard
```
no tiene que arrancar todo desde cero, por eso abre instantáneo.

### Después de crear Walker

### Recargar systemd:

```bash
systemctl --user daemon-reload
```
### Activar e iniciar:

```bash
systemctl --user enable --now walker.service
```
### Comprobar:

```bash
systemctl --user status walker.service
```
En mi caso se ve así:

```bash
● walker.service - Walker Launcher Service
     Loaded: loaded (/home/graplo/.config/systemd/user/walker.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-07-07 10:36:55 -05; 47min ago
 Invocation: bb3bf6f1b4d7471680fde814edd31dec
   Main PID: 129911 (walker)
      Tasks: 16 (limit: 38322)
     Memory: 39.9M (peak: 87.1M)
        CPU: 2.613s
     CGroup: /user.slice/user-1000.slice/user@1000.service/app.slice/walker.service
             └─129911 /usr/bin/walker --gapplication-service
```


## Tus binds en Niri

Niri solamente abre acciones, no arranca servicios.

Ejemplo: Claro tiene q esta dentro de `binds { Ejemplo }`

```ini
    //Walker
    Mod+Space   hotkey-overlay-title="Walker Launcher"  { spawn "walker"; }
    Mod+V       hotkey-overlay-title="Walker Clipboard" { spawn "walker" "-m" "clipboard"; }
    Mod+slash   hotkey-overlay-title="Walker Files"     { spawn "walker" "-m" "files"; }
    Mod+Shift+C hotkey-overlay-title="Walker Calc"      { spawn "walker" "-m" "calc"; }

```

## Flujo:

Pulsas Mod+V
        ↓
Niri ejecuta walker -m clipboard
        ↓
Walker ya está cargado
        ↓
Walker pregunta a Elephant
        ↓
Elephant entrega historial clipboard
Comandos útiles para recordar

### Ver servicios:

```bash
systemctl --user list-units
```
### Reiniciar Elephant:

```bash
systemctl --user restart elephant
```
### Reiniciar Walker:

```bash
systemctl --user restart walker
```
### Ver logs:

```bash
journalctl --user -u elephant -f
journalctl --user -u walker -f
```
## Desactivar si algún día no quieres que arranquen:

```bash
systemctl --user disable elephant
systemctl --user disable walker
```
