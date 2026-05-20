---
title: "PyRadio: radio en tu terminal con estilo"
date: 2026-05-19T20:44:36-05:00
draft: false
summary: "Cómo instalar PyRadio, agregar tus emisoras favoritas y personalizarlo con temas Dracula desde la terminal."
description: "Guía práctica de PyRadio en CachyOS: instalación, agregar emisoras desde RadioBrowser o URL directa, atajos esenciales y cómo aplicar temas."
tags: ["linux", "terminal", "audio", "tui", "dracula"]
categories: ["Terminal"]
---

## ¿Qué es PyRadio?

PyRadio es un reproductor de radio por internet que corre directamente en la terminal. Está escrito en Python, usa mpv como backend de audio y tiene navegación estilo vim. Sin interfaz gráfica, sin anuncios, sin complicaciones.

## Instalación

En CachyOS o cualquier distro basada en Arch:

```bash
paru -S pyradio
```

## Agregar emisoras

PyRadio guarda las emisoras en un archivo CSV simple en `~/.config/pyradio/stations.csv`. Cada línea es una emisora con nombre y URL:

Doble Nueve, https://conectperu.com:7000/stream?icy=http
Filarmonía, https://c22.radioboss.fm:18100/live

### ¿De dónde saco las URLs?

La forma más fácil es buscar tu emisora en [RadioBrowser](https://www.radio-browser.info) — una base de datos colaborativa y de código abierto con más de 300,000 emisoras. Desde la página de detalles copia el campo `url` directamente al CSV.

Si la emisora usa un codec como AAC+ que algunos reproductores rechazan, la URL directa del stream suele funcionar sin problemas. Para verificar antes de agregarla:

```bash
mpv https://tu-url-del-stream
```

## RadioBrowser integrado

PyRadio tiene RadioBrowser integrado directamente. Presiona `O` para abrirlo — por defecto carga las 100 emisoras más votadas de la plataforma.

### Buscar emisoras

Dentro de RadioBrowser presiona `s` para abrir la ventana de búsqueda. Puedes filtrar por nombre, país, idioma, bitrate y más. Para buscar emisoras peruanas por ejemplo pon `PE` en el campo Country.

### Historial de búsquedas

PyRadio guarda las búsquedas que haces en RadioBrowser. Para navegar por ellas dentro de la ventana de búsqueda (`s`):

- `Ctrl+N / Ctrl+P` — moverte entre búsquedas anteriores y siguientes
- `Ctrl+B` — marcar la búsqueda actual como predeterminada

### Guardar una emisora a tu playlist

1. Navega hasta la emisora que quieras
2. Presiona `y` para copiarla (yank, como en vim)
3. Presiona `Enter` para guardarla en el registro
4. Sal de RadioBrowser con `q`
5. Presiona `p` para pegarla en tu playlist actual

### Guardar cambios en la playlist

Después de pegar o modificar cualquier emisora presiona `s` para guardar. 
PyRadio debería guardar automáticamente al salir pero por si acaso mejor hacerlo manual.

### Cerrar RadioBrowser

Con `q`, `Escape` o `\\` (doble backslash) para volver al historial de playlists.

## Gestión de playlists

### Crear una nueva playlist

1. Presiona `o` para abrir el selector de playlists
2. Desde ahí presiona `n` para crear una nueva
3. Le pones nombre y listo

También puedes crearla directamente como CSV:

```bash
touch ~/.config/pyradio/clasica.csv
```

### Borrar una emisora

Selecciónala y presiona `del` — pide confirmación antes de borrar.

### Agregar a favoritos

Presiona `*` sobre cualquier emisora para agregarla a la playlist de favoritos. Presiona `*` de nuevo para quitarla. Para ver tus favoritos presiona `o` y selecciona la lista `favorites`.

### Borrar una playlist

Desde la terminal:

```bash
rm ~/.config/pyradio/nombre-playlist.csv
```

## Atajos básicos

### Navegación
- `j / k` o flechas — bajar/subir en la lista
- `g / G` — ir al inicio/final
- `Enter` — reproducir emisora seleccionada
- `p` — ir a la emisora que está sonando
- `q` — salir

### Reproducción
- `Space` — detener/reanudar
- `+ / -` — subir/bajar volumen
- `m` — silenciar
- `r` — reproducir una emisora aleatoria

### Playlists
- `o` — abrir selector de playlists
- `s` — guardar playlist actual
- `R` — recargar playlist desde disco
- `*` — agregar/quitar de favoritos

### Búsqueda
- `/` — buscar en la lista actual
- `n / N` — siguiente/anterior resultado
- `O` — abrir RadioBrowser

## Temas

PyRadio trae varios temas incluidos, entre ellos Dracula y Catppuccin. Para cambiar el tema:

1. Presiona `t` para abrir el selector de temas
2. Navega con `j/k` o las flechas hasta el que quieras
3. Presiona `Enter` para previsualizar
4. Si te convence presiona `Space` para dejarlo como predeterminado

El asterisco `*` en la lista indica cuál es el tema activo por defecto.

## Evitar que mpv abra ventana

Por defecto mpv puede abrir una ventana visual al reproducir. Para que corra en background cuando lo llama pyradio, edita `~/.config/mpv/mpv.conf`:

```
  force-window=yes
[pyradio]
force-window=no
```
El perfil `[pyradio]` lo activa pyradio automáticamente, así el resto de usos de mpv no se ven afectados.
