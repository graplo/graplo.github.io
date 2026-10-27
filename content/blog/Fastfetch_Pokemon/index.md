---
title: "Fastfetch con Pokémon: sprites en tu terminal usando krabby"
date: 2026-10-26T22:47:56-05:00
draft: false
summary: "Cómo configurar fastfetch para que muestre un Pokémon aleatorio como logo usando krabby, con instalación en Arch, generación de la config y una línea de colores Dracula."
description: "Guía para instalar krabby y fastfetch en Arch Linux, generar la configuración de fastfetch e integrar sprites de Pokémon como logo con command-raw."
tags: ["linux", "terminal", "fastfetch", "pokemon", "rice", "cli", "arch"]
categories: ["Terminal"]
---

`krabby` imprime sprites de Pokémon en la terminal. Combinado con `fastfetch` puedes reemplazar el logo de la distro por un Pokémon distinto cada vez que abres una terminal, sin perder la información del sistema.

No es difícil, pero hay un par de detalles (el paquete está en el AUR y la integración usa un tipo de logo poco conocido) que vale la pena explicar paso a paso.

## Instalación

`fastfetch` está en los repositorios oficiales:

```bash
sudo pacman -S fastfetch
```

`krabby` solo está en el AUR, así que necesitas un helper como `paru`:

```bash
paru -S krabby-bin
```

Esto es lo que ofrece el AUR:

- `krabby-bin` — binario precompilado. Instalación rápida y sin compilar Rust. Es el que uso.
- `krabby-git` — versión de desarrollo, se compila al instalar.
- `kingler-bin` — un fork de krabby que además muestra información del Pokémon.

> `krabby-bin` y `krabby-git` entran en conflicto entre sí, así que solo se instala uno.

## Uso básico de krabby

Sin configurar nada todavía, ya puedes probarlo:

```bash
krabby random
```

**Comandos principales:**

- `krabby random` — muestra un Pokémon aleatorio
- `krabby random 1-3` — aleatorio de un rango de generaciones (del 1 al 9)
- `krabby random 1,3,6` — aleatorio de una lista de generaciones
- `krabby name pikachu` — selecciona un Pokémon por nombre
- `krabby list` — lista todos los Pokémon disponibles

Los nombres se escriben como en los juegos, con algunas excepciones como `nidoran-f`, `nidoran-m`, `mr-mime`, `farfetchd` o `type-null`. Si dudas, filtra la lista:

```bash
krabby list | grep -i nidoran
```

Por defecto krabby imprime el nombre del Pokémon encima del sprite. Para quitarlo se usa `--no-title`, que es lo que usaremos en fastfetch:

```bash
krabby random --no-title
```

Para ver todas las opciones de cada subcomando:

```bash
krabby random --help
```

## Generar la configuración de fastfetch

Fastfetch puede crear su propio archivo de configuración con un comando:

```bash
fastfetch --gen-config
```

Esto genera `~/.config/fastfetch/config.jsonc` con los valores por defecto, que luego puedes editar.

Si ya tenías una configuración y quieres empezar de cero, existe `--gen-config-force`, pero **sobrescribe el archivo actual**, así que haz una copia antes:

```bash
cp ~/.config/fastfetch/config.jsonc ~/.config/fastfetch/config.jsonc.bak
fastfetch --gen-config-force
```

## Integrar krabby en fastfetch

La clave está en el bloque `logo`. Fastfetch tiene un tipo llamado `command-raw`, que ejecuta un comando y usa su salida tal cual como logo, respetando los colores ANSI. Justo lo que necesita un sprite de krabby:

```jsonc
"logo": {
  "type": "command-raw",
  "source": "krabby random --no-title"
}
```

Cada vez que ejecutes `fastfetch`, se llama a `krabby` y aparece un Pokémon distinto.

Si prefieres limitarte a ciertas generaciones o fijar uno en concreto, solo cambia el `source`:

```jsonc
"source": "krabby random 1-3 --no-title"
```

```jsonc
"source": "krabby name charizard --no-title"
```

## Configuración completa

Esta es la configuración que uso. Cópiala en `~/.config/fastfetch/config.jsonc`:

```jsonc
{
  "$schema": "https://github.com/fastfetch-cli/fastfetch/raw/master/doc/json_schema.json",
  "logo": {
    "type": "command-raw",
    "source": "krabby random --no-title"
  },
  "modules": [
    "title",
    "separator",
    "os",
    "host",
    "kernel",
    "uptime",
    "packages",
    "shell",
    "display",
    "de",
    "wm",
    "wmtheme",
    "theme",
    "icons",
    "font",
    "cursor",
    "terminal",
    "terminalfont",
    "cpu",
    "gpu",
    "memory",
    "swap",
    "disk",
    "battery",
    "poweradapter",
    "localip",
    "locale",
    "break",
    {
      "type": "custom",
      "format": "\u001b[38;2;241;250;140m󰐝 \u001b[38;2;189;147;249m󰐝 \u001b[38;2;139;233;253m󰐝 \u001b[38;2;255;121;198m󰐝 \u001b[38;2;255;184;108m󰐝 \u001b[38;2;80;250;123m󰐝 \u001b[38;2;98;114;164m󰐝"
    }
  ]
}
```

## Módulos

La lista `modules` define qué información se muestra y en qué orden. Es bastante larga, así que puedes borrar o reordenar lo que no te interese. Por ejemplo, `battery` y `poweradapter` no tienen sentido en un PC de escritorio, y `wmtheme`, `icons` o `cursor` solo importan si te gusta mostrar tu setup visual.

Para ver todos los módulos disponibles:

```bash
fastfetch --list-modules
```

## Línea de colores Dracula

El último elemento de la lista es un módulo `custom` que dibuja una fila de iconos con la paleta Dracula usando colores RGB (`\u001b[38;2;R;G;Bm`). Es opcional, puedes quitarlo o cambiar los valores por los de tu tema.

> Los iconos `󰐝` son glifos de Nerd Font, así que necesitas una Nerd Font en tu terminal para verlos. Si al copiar y pegar el JSON los iconos se pierden o salen como un cuadro, reemplázalos por otro símbolo (como `●`) o escríbelos de nuevo con tu fuente.

## Ejecutarlo al abrir la terminal

Si quieres que aparezca cada vez que abres una terminal, añade esto al final de `~/.config/fish/config.fish`:

```fish
if status is-interactive
    fastfetch
end
```

En Bash o Zsh basta con poner `fastfetch` al final de `~/.bashrc` o `~/.zshrc`.

> Como los sprites tienen tamaños distintos, el diseño puede moverse un poco de un Pokémon a otro. Si algún sprite te queda demasiado grande o desalineado, vuelve a ejecutar `fastfetch` y saldrá otro.
