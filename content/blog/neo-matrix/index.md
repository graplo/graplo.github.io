---
title: "Neo-matrix: La lluvia digital de Matrix en tu terminal"
date: 2026-05-03T17:36:13-05:00
draft: false
summary: "neo-matrix recrea el efecto de lluvia digital de The Matrix en la terminal, con colores personalizables, mensajes y decenas de opciones para ajustarlo a tu gusto."
description: "Cómo usar neo-matrix en Linux para simular la lluvia digital de The Matrix, con flags principales y colores Dracula en un script."
tags: ["linux", "terminal", "matrix", "rice", "cli", "fish"]
categories: ["Terminal"]
---

`neo-matrix` recrea el efecto de lluvia digital de The Matrix — específicamente la escena donde Cypher le explica el código a Neo. Incluye detalles como los caracteres katakana de media anchura, colores desiguales, glitching y parpadeo.

En CachyOS y Arch el binario se llama `neo-matrix`, no `neo` como aparece en la documentación oficial.

## Instalación

```bash
paru -S neo-matrix
```

## Uso básico

Sin argumentos ya funciona con los valores por defecto:

```bash
neo-matrix
```

## Flags principales

**Velocidad y movimiento:**

- `-S NUM` / `--speed=NUM` — velocidad de scroll (default: 8.0)
- `-a` / `--async` — cada columna scrollea a velocidad independiente
- `-d NUM` / `--density=NUM` — cantidad de droplets en pantalla (default: 1.0, rango recomendado: 0.25–4.0)
- `-r NUM` / `--rippct=NUM` — porcentaje de droplets que se detienen antes de llegar al fondo (default: 33%)
- `--shortpct=NUM` — porcentaje de droplets acortados (default: 50%)

**Colores:**

- `-c COLOR` / `--color=COLOR` — color predefinido: `green`, `green2`, `green3`, `yellow`, `orange`, `red`, `blue`, `cyan`, `gold`, `rainbow`, `purple`, `pink`, `pink2`, `vaporwave`, `gray`
- `-C FILE` / `--colorfile=FILE` — archivo de colores personalizado
- `-D` / `--defaultbg` — usa el color de fondo del terminal
- `--colormode=NUM` — modo de color: 0 (mono), 16, 256, 32 (32-bit)

**Caracteres:**

- `--charset=LANG` — juego de caracteres: `ascii`, `katakana`, `greek`, `cyrillic`, `arabic`, `hebrew`, `braille`, `runic`, `binary`, `hex`, entre otros
- `--chars=NUM1,NUM2` — rango de caracteres Unicode en hexadecimal

**Efectos:**

- `-G NUM` / `--glitchpct=NUM` — porcentaje de caracteres con glitch (default: 10%)
- `-g NUM1,NUM2` / `--glitchms=NUM1,NUM2` — intervalo entre glitches en ms (default: 300,400)
- `--noglitch` — desactiva el glitching
- `-l NUM1,NUM2` / `--lingerms=NUM1,NUM2` — tiempo que permanecen los caracteres tras terminar el scroll (default: 1,3000)
- `-M NUM` / `--shadingmode=NUM` — modo de sombreado: 0=aleatorio (default), 1=gradiente
- `-b NUM` / `--bold=NUM` — caracteres en negrita: 0=off, 1=aleatorio (default), 2=todos

**Mensaje:**

- `-m "STR"` / `--message="STR"` — muestra un mensaje en el centro que se va descubriendo con la lluvia. Solo acepta ASCII. Para que aparezca rápido:

```bash
neo-matrix -m "Wake up, Graplo..." --speed=12 --density=3 --lingerms=1,1 --rippct=0
```

**Otros:**

- `-f NUM` / `--fps=NUM` — framerate objetivo (default: 60)
- `-s` / `--screensaver` — sale al primer keypress
- `-F` / `--fullwidth` — dos columnas por carácter (útil para katakana completo)

## Controles en tiempo real

Mientras corre puedes presionar teclas para ajustar el comportamiento:

- `ESC` / `q` — salir
- `SPACE` — limpiar pantalla
- `UP` / `DOWN` — aumentar/disminuir velocidad
- `+` / `-` — más/menos droplets
- `p` — pausar
- `a` — alternar velocidad asíncrona
- `TAB` — alternar modo de sombreado
- `1`–`9` y símbolos — cambiar color en tiempo real

## Archivo de colores personalizado

El formato del archivo es simple — primera línea es el fondo, las siguientes son los colores del gradiente en orden de brillo ascendente. Cada línea tiene cuatro valores: código de color 16/256, R, G, B (escala 0–1000):
```
neo_color_version 1
0 0 0 0          # Fondo negro
61 98 114 164    # Morado oscuro (estela)
141 189 147 249  # Dracula Purple
117 139 233 253  # Dracula Cyan
231 248 248 242  # Punta blanca
```
## Script con colores Dracula

Para tenerlo todo en un solo archivo sin depender de un txt externo:

```bash
#!/bin/bash
TMPFILE=$(mktemp)
cat > "$TMPFILE" << 'EOF'
neo_color_version 1
0 0 0 0
61 98 114 164
141 189 147 249
117 139 233 253
231 248 248 242
EOF
neo-matrix -C "$TMPFILE" -D -m "Wake up, Graplo..." --speed=12 --density=3 --lingerms=1,1 --rippct=0
rm "$TMPFILE"
```

Guárdalo en `~/.local/bin/matrix` y dale permisos:

```bash
chmod +x ~/.local/bin/matrix
```

Desde cualquier terminal basta con escribir `matrix`.

## Rendimiento

Si hay stuttering o alto uso de CPU, algunas opciones para aliviar la carga:

```bash
neo-matrix --fps=30 -d 0.5 --speed=5 --noglitch --colormode=0 --bold=0 --charset=ascii
```

> neo-matrix funciona mejor con terminales con aceleración GPU como Ghostty o Alacritty. Sin aceleración puede consumir uno o varios cores de CPU.
