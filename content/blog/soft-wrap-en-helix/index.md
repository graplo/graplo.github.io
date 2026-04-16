---
title: "Soft-wrap en Helix: adiós al scroll horizontal"
date: 2026-04-15T23:34:28-05:00
draft: true
summary: "Una configuración mínima en Helix que hace que las líneas largas salten solas al borde de la ventana, sin tocar el archivo."
description: "Cómo activar el ajuste de línea automático en Helix para evitar el desplazamiento horizontal y mejorar la lectura de líneas largas."
tags: ["helix", "editor", "configuracion", "terminal"]
categories: ["Terminal"]
---

Cuando trabajas con archivos de configuración o texto largo en Helix, las líneas que superan el ancho de la ventana te obligan a hacer scroll horizontal para leerlas. Molesto, innecesario, y fácil de resolver.

La solución es el **soft-wrap**: el editor muestra la línea como si "saltara" al siguiente renglón visual, pero sin insertar ningún salto de línea real en el archivo. El contenido no cambia, solo la forma en que se presenta.

## Configuración

En `~/.config/helix/config.toml`:

```toml
[editor]
soft-wrap.enable = true
soft-wrap.wrap-indicator = "↪ "
soft-wrap.wrap-at-text-width = true
```

- `soft-wrap.enable` activa el ajuste automático.
- `soft-wrap.wrap-indicator` define el carácter que aparece al inicio de cada línea de continuación. El `↪` deja claro visualmente que esa línea es parte de la anterior.
- `soft-wrap.wrap-at-text-width` respeta la sangría original de la línea al hacer el salto, lo que mantiene el código legible.

Con esto, Helix ajusta las líneas largas al ancho visible de la ventana sin modificar el archivo. Ideal para configs, Markdown o cualquier archivo con líneas que se van de pantalla.


```toml
[editor]
soft-wrap.enable = true
soft-wrap.wrap-indicator = "↪ "
soft-wrap.wrap-at-text-width = true
text-width = 110
```
> Si el área de lectura te parece demasiado estrecha o ancha, ajusta text-width a tu gusto. El valor por defecto es 80 columnas. Yo lo tengo en 110 para aprovechar mejor el ancho de mi monitor.
