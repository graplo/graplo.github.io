---
title: "Miniaturas_nautilus_mkv_solucion"
date: 2026-07-29T07:25:59-05:00
draft: false
summary: "Cómo hacer que Nautilus muestre las vistas previas de tus grabaciones de OBS en formato .mkv configurando ffmpegthumbnailer correctamente."
description: "Guía paso a paso para solucionar la falta de miniaturas de archivos Matroska (.mkv) en GNOME/Nautilus corrigiendo los tipos MIME y limpiando la caché desde Fish."
tags: ["linux", "gnome", "nautilus", "mkv", "obs", "cachyos", "fish"]
categories: ["Desktop"]
---

## El problema de las miniaturas fantasma

Si grabas con OBS Studio, lo más probable es que tus videos se guarden en formato `.mkv` (Matroska). Al abrir Nautilus para organizar tus clips, en lugar de ver la captura del video te encuentras con un mar de iconos genéricos (como los de Papirus), mientras que los `.mp4` se ven perfectamente.

Aunque muchos asumen que el problema es el peso del archivo —Nautilus sí tiene un límite de tamaño por defecto para miniaturas—, la causa real suele ser otra: GNOME no sabe cómo relacionar el archivo Matroska con el generador de miniaturas correcto, o la configuración entra en conflicto con su sistema de aislamiento (sandbox).

## La raíz del problema: los tipos MIME

Cuando consultas el tipo de archivo de un `.mkv` desde la terminal, el sistema te devuelve `video/x-matroska`. Sin embargo, las reglas de GNOME y muchos paquetes por defecto a veces buscan la variante estándar `video/matroska`. Si el archivo de configuración no declara ambas, Nautilus falla en silencio.

## Instalación

En CachyOS o cualquier otra distribución basada en Arch, primero necesitamos un generador ligero y rápido. `ffmpegthumbnailer` es la mejor opción:

```bash
sudo pacman -S ffmpegthumbnailer
```
## Configurar el Thumbnailer

### Crear una copia

Solo por prevención no está mal crear una copia, para esto desde terminal:

```bash
sudo cp /usr/share/thumbnailers/ffmpegthumbnailer.thumbnailer /usr/share/thumbnailers/ffmpegthumbnailer.thumbnailer.bak
```
### Sobreescrivo los cambios

Vamos a sobrescribir la regla de GNOME para asegurarnos de que cubra absolutamente todas las variantes de Matroska y use una configuración que no rompa el sandbox.

Ejecuta este bloque completo en tu terminal (te pedirá contraseña de sudo):

```bash
sudo bash -c 'cat <<EOF> /usr/share/thumbnailers/ffmpegthumbnailer.thumbnailer
[Thumbnailer Entry]
TryExec=ffmpegthumbnailer
Exec=ffmpegthumbnailer -i %i -o %o -s %s -q 7
MimeType=video/3gpp;video/3gpp2;video/annodex;video/dv;video/isivideo;video/mj2;video/mp2t;video/mp4;video/mpeg;video/ogg;video/quicktime;video/vnd.avi;video/vnd.mpegurl;video/vnd.radgamettools.bink;video/vnd.radgamettools.smacker;video/vnd.rn-realvideo;video/vnd.vivo;video/vnd.youtube.yt;video/wavelet;video/webm;video/x-anim;video/x-flic;video/x-flv;video/x-javafx;video/x-matroska;video/matroska;video/x-matroska-3d;video/x-mjpeg;video/x-mng;video/x-ms-wmv;video/x-nsv;video/x-ogm+ogg;video/x-sgi-movie;video/x-theora+ogg;application/mxf;application/vnd.ms-asf;application/vnd.rn-realmedia;application/x-matroska;application/ogg;
EOF'
```
Detalle visual: El comando usa -q 7 (calidad de imagen). Por defecto, ffmpegthumbnailer decora las miniaturas añadiendo unos bordes laterales que simulan una cinta de película (filmstrip), lo cual queda bastante bien. Si prefieres las imágenes totalmente limpias, puedes añadir la bandera -t al final de la línea Exec.

## Limpiar la caché y reiniciar

Para que Nautilus aplique los cambios, hay que borrar todas las miniaturas fallidas que ya almacenó.

Si usas Bash o Zsh, un simple rm -rf ~/.cache/thumbnails/* funcionaría. Pero si usas Fish, el comodín * te dará error si alguna subcarpeta está vacía o hay archivos ocultos conflictivos. La forma correcta y a prueba de fallos en Fish es especificar los directorios exactos y matar el proceso de Nautilus en la misma línea:

```bash
rm -rf ~/.cache/thumbnails/{fail,normal,large,x-large} ; nautilus -q
```

Al abrir nuevamente la carpeta de tus grabaciones, Nautilus procesará los archivos y finalmente verás las miniaturas generadas.
