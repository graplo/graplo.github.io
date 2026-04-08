---
title: "Imagemagick Basic Comands Example"
date: 2026-04-07T20:01:05-05:00
draft: true
summary: "Usando el comando magik, en una situación real. De paso repasamos algunos q son interesantes"
tags: ["ImageMagick", "Linux", "cli"]
categories: ["Terminal"]
---

# Generar favicons y avatars con ImageMagick (magick)

Cuando trabajas con sitios estáticos (como Hugo), necesitas varias versiones de un mismo logo: favicons, iconos para Android, Apple, etc.

Aquí dejo los comandos básicos usando `magick` (ImageMagick) y un flujo real.

---

## Idea general

* Partes de una imagen grande (ej: `2000x2000`)
* Generas versiones más pequeñas
* Usas un buen filtro (`Lanczos`) para mantener calidad

---

## Comandos esenciales

### Resize

```bash
magick input.png -resize 512x512 output.png
```

---

### Filtro (calidad)

```bash
-filter Lanczos
```

Mejora la nitidez al reducir imágenes.

---

### Quitar metadata

```bash
-strip
```

Reduce peso del archivo.

---

## Ejemplo real: generar favicons

Partiendo de:

```
g-logo.png (2000x2000)
```

---

### Error común

NO hagas esto:

```bash
magick g-logo.png -resize 192x192 a.png -resize 512x512 b.png
```

Esto encadena los resize y pierde calidad.

---

## Forma correcta (una sola ejecución)

```bash
magick g-logo.png -filter Lanczos \
  \( -clone 0 -resize 192x192 -write android-chrome-192x192.png \) \
  \( -clone 0 -resize 512x512 -write android-chrome-512x512.png \) \
  \( -clone 0 -resize 194x194 -write apple-touch-icon.png \) \
  \( -clone 0 -resize 16x16 -write favicon-16x16.png \) \
  \( -clone 0 -resize 32x32 -write favicon-32x32.png \) \
  \( -clone 0 -resize 48x48 -write favicon-48x48.png \) \
  \( -clone 0 -resize 48x48 -write favicon.ico \) \
  null:
```

---

## Explicación

* `-clone 0` → copia la imagen original
* `-resize` → genera el tamaño
* `-write` → guarda el resultado
* `null:` → evita salida final innecesaria

> Cada imagen se genera desde el original (sin pérdida de calidad)

---

## Crear favicon.ico con múltiples tamaños

```bash
magick favicon-16x16.png favicon-32x32.png favicon-48x48.png favicon.ico
```

---

## Generar avatar

```bash
magick g-logo-recortado.png -filter Lanczos -resize 512x512 avatar.png
magick g-logo-recortado.png -filter Lanczos -resize 512x512 -quality 90 avatar.webp
```

---

## Tip importante

Si tu logo tiene halo blanco:

1. Recórtalo antes (ej: en GIMP)
2. Luego recién haces resize

> evita bordes sucios al reducir

---

## Resumen

* Usa siempre el original para cada tamaño
* `Lanczos` para mejor calidad
* `-clone` para múltiples salidas
* `-write` para guardar resultados intermedios

---

## Resultado

Con estos comandos puedes generar:

* Favicons
* Iconos web
* Avatares

todo desde terminal, rápido y reproducible.

