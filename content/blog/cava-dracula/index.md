---
title: "Aplicar tema Dracula en CAVA"
date: 2026-04-29T12:41:32-05:00
draft: false
summary: "CAVA permite cargar temas de color desde archivos externos. Con un solo ajuste en el config se puede aplicar Dracula sin tocar los colores manualmente."
description: "Cómo aplicar un tema de color externo en CAVA usando la opción theme del archivo de configuración."
tags: ["linux", "cava", "dracula", "rice", "cli"]
categories: ["Terminal"]
---

CAVA es un visualizador de audio para terminal. Por defecto usa los colores del tema actual, pero permite cargar paletas de color desde archivos externos ubicados en `~/.config/cava/themes/`.

## Descargar el tema

El tema Dracula para CAVA está disponible en varios repositorios de la comunidad. El archivo tiene extensión `.cava` y contiene solo la sección `[color]` con los gradientes definidos en hex.

Un ejemplo del contenido:

```ini
[color]
gradient = 1
gradient_color_1 = '#8BE9FD'
gradient_color_2 = '#9AEDFE'
gradient_color_3 = '#CAA9FA'
gradient_color_4 = '#BD93F9'
gradient_color_5 = '#FF92D0'
gradient_color_6 = '#FF79C6'
gradient_color_7 = '#FF6E67'
gradient_color_8 = '#FF5555'
```

Guárdalo en:

```bash
~/.config/cava/themes/dracula.cava
```

## Activar el tema

Abre el archivo de configuración principal, en mi caso mi editor de texto es Helix pero puede ser cualquiera:

```bash
helix ~/.config/cava/config
```

Busca la sección `[color]` y agrega esta línea — asegúrate de que no esté comentada con `;`:

```ini
[color]
theme = dracula.cava
```

> Solo el nombre del archivo, sin ruta ni comillas. CAVA busca automáticamente en `~/.config/cava/themes/`.

## Verificar

Ejecuta CAVA con música sonando:

```bash
cava
```

Las barras deberían mostrar el gradiente de Dracula de azul a rojo. Si algo falla, CAVA muestra el error directamente en terminal indicando qué archivo no pudo abrir.


