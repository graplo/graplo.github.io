---
title: "Swaylock y awww"
draft: false
summary: "Implementación de un lock screen sincronizado en Niri usando awww para obtener dinámicamente el wallpaper actual y swaylock-effects para aplicar blur y estilo, mediante un script wrapper que automatiza la extracción de la imagen activa y su integración con swaylock sin perder la configuración visual existente."
tags: ["lockscreen", "terminal", "swaylock", "scripts", "awww"]
showDate: false
categories: ["Niri"]
---

## Lock screen sincronizado con wallpaper en Niri + awww + swaylock-effects

### El problema

En un entorno como Niri, donde el wallpaper puede cambiar dinámicamente (por ejemplo con `awww`), el lock screen de `swaylock-effects` normalmente no está sincronizado con el fondo actual.

Esto genera un desajuste visual:

- El escritorio muestra un wallpaper
- El lock screen puede usar otro (o incluso un screenshot del estado anterior)
- El resultado es inconsistente y poco limpio

La idea es resolver esto haciendo que el lock screen siempre use el wallpaper actual del sistema, sin intervención manual.


### La solución

La solución consiste en crear un pequeño wrapper que:

1. Obtiene el wallpaper actual desde `awww`
2. Extrae la ruta de la imagen
3. La pasa a `swaylock-effects`
4. Aplica blur automáticamente


## Dependencias

### Arch Linux / CachyOS

Instalar swaylock-effects:

```bash
sudo pacman -S swaylock-effects awww
```
## Obtener el wallpaper actual

El comando base que expone el wallpaper es:

```bash
awww query
```
Ejemplo de salida:

```bash
DP-2: 2560x1440, scale: 1, currently displaying: image: /home/user/Imágenes/wallpapers/fondo.png
```
Para extraer únicamente la ruta de la imagen:  

```basch
awww query | sed -n 's/.*image: //p'
```
Esto devuelve:

```bash
/home/user/Imágenes/wallpapers/fondo.png
```

## Script del lock screen

Creamos un script personalizado:
En mi caso **hx** es **Helix** pero usa tu editor de texto faborito 

```bash
hx ~/.local/bin/lock
```
Contenido:

```bash
  #!/usr/bin/env bash
# ┏┓       ┓    ┓                  
# ┗┓┓┏┏┏┓┓┏┃ ┏┓┏┃┏  ━━  ┏┓┓┏┏┓┏┏┓┏┏
# ┗┛┗┻┛┗┻┗┫┗┛┗┛┗┛┗      ┗┻┗┻┛┗┻┛┗┻┛
#         ┛                        

WALL=$(awww query | sed -n 's/.*image: //p')

swaylock \
  -i "$WALL" \
  --effect-blur 7x5

  ```
  Dar permisos de ejecución:

  ```bash
chmod +x ~/.local/bin/lock
  ```
## Configuración del keybind en Niri

  En lugar de llamar directamente a swaylock, ahora llamamos al script:

  ```ini
Mod+Alt+L hotkey-overlay-title="Lock Screen" { spawn "lock"; }
  ```
## Configuración de swaylock-effects

Lo encontramos en:

```bash
hx ~/.config/swaylock/config
```

La configuración visual se mantiene separada del script:

```ini
#          ▜     ▌   ▐▘▌     
# ▛▘▌▌▌▀▌▌▌▐ ▛▌▛▘▙▘  ▜▘▙▘▛▘▛▘
# ▄▌▚▚▘█▌▙▌▐▖▙▌▙▖▛▖  ▐ ▛▖▌ ▄▌
#        ▄▌                  
# ----------------------------
# Background
# ----------------------------

# screenshots
# effect-blur=20x2
# effect-vignette=0.5:0.5
fade-in=0.2

# ----------------------------
# Clock
# ----------------------------

clock
timestr=%H:%M
datestr=%a, %d %B %Y

font=Ubuntu
font-size=45

# ----------------------------
# Indicator
# ----------------------------

indicator
indicator-radius=130
indicator-thickness=10

# ----------------------------
# Dracula Theme
# ----------------------------

inside-color=282A36ff
ring-color=BD93F9ff
line-color=282A36ff
separator-color=282A36ff

text-color=F8F8F2ff

# Verifying
inside-ver-color=50FA7Bcc
ring-ver-color=50FA7Bff
text-ver-color=50FA7Bff
line-ver-color=50FA7Bff

# Wrong password
inside-wrong-color=FF5555cc
ring-wrong-color=FF5555ff
text-wrong-color=FF5555ff
line-wrong-color=FF5555ff

# Caps Lock
inside-caps-lock-color=FFB86Ccc
ring-caps-lock-color=FFB86Cff
text-caps-lock-color=FFB86Cff
line-caps-lock-color=FFB86Cff

# Key press highlight
key-hl-color=50FA7Bff
bs-hl-color=FF79C6ff

# ----------------------------
# Misc
# ----------------------------

# show-keyboard-layout
indicator-idle-visible
```

## Resultado

Con esta configuración:

- El wallpaper actual de awww se usa automáticamente
- El lock screen siempre está sincronizado con el escritorio
- No es necesario actualizar manualmente nada al cambiar fondo
- El blur se aplica de forma consistente (7x5 en este caso)

## Conclusión

Este enfoque separa claramente responsabilidades:

- awww → gestiona el wallpaper
- script lock → conecta estado del sistema con swaylock
- swaylock-effects → renderiza UI del lock screen

El resultado es un sistema limpio, automático y coherente visualmente.

## tip para tomar foto de ello

```bash
sleep 5 && grim ~/lockscreen.png
```
> tienes q tener grim para esto.

Con este ejemplo le damos 5 segundos para que te de tiempo de poner el **`swaylock`** en mi caso uso las teclas `super+alt+l`.
