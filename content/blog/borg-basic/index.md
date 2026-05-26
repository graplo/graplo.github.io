---
title: "Borg: backups serios desde la terminal"
date: 2026-05-24T17:39:37-05:00
draft: false
summary: "Guía práctica de Borg: deduplicación, compresión, encriptación y automatización con script para no volver a perder datos."
description: "Aprende a usar Borg Backup desde cero: init, create, mount, restore, prune y cómo automatizarlo todo con un script en Bash"
tags: ["linux", "terminal", "backup", "cli", "bash", "Borg"]
categories: ["Terminal"]
---
## ¿Qué es Borg?

Borg es una herramienta de backup para terminal con tres características que la hacen destacar: deduplicación, compresión y encriptación. La deduplicación significa que solo guarda lo que cambió desde el último backup — si tienes 116GB de datos pero solo cambiaron 3GB, Borg solo guarda esos 3GB. El resultado es que puedes tener meses de backups diarios ocupando una fracción del espacio original.

Es de esas herramientas que entre más la usas más la respetas. Por fuera parece simple pero por dentro tiene mucho trabajo bien hecho.

## Instalación

En CachyOS o cualquier distro basada en Arch:

```bash
sudo pacman -S borg
```

## Conceptos básicos

Antes de empezar hay dos términos importantes:

- **Repositorio** — la carpeta donde Borg guarda todo. Tiene su propia estructura interna, nunca la toques directamente con `rm` o `mv`.
- **Archive** — cada backup individual dentro del repositorio. Puedes tener cientos de archives en un solo repositorio.

## Init — una sola vez

El repositorio se inicializa una sola vez:

```bash
borg init --encryption=repokey /ruta/a/tu/backup
```

`--encryption=repokey` activa encriptación con contraseña — te la pedirá al crear y en cada operación posterior. Para pruebas puedes usar `--encryption=none`.

## Crear un backup

```bash
borg create --progress --stats --compression zstd,3 \
    /ruta/backup::"home-{now:%Y-%m-%d_%H-%M-%S}" ~/
```

Las llaves `{}` generan el timestamp automáticamente. En Fish necesitas comillas dobles para que no interprete las llaves.

### Compresión

Borg ofrece varias opciones de compresión:

| Algoritmo | Velocidad | Compresión | Uso recomendado |
|-----------|-----------|------------|-----------------|
| `none` | Máxima | Ninguna | Sin prioridad de espacio |
| `lz4` | Muy rápida | Moderada | Backups frecuentes |
| `zstd,3` | Rápida | Buena | Uso general — la más recomendada |
| `zlib` | Media | Moderada | Alternativa a lz4 |
| `lzma` | Lenta | Máxima | Archivado a largo plazo |

El nivel de zstd va de 1 a 22 — el 3 es un buen balance entre velocidad y compresión.

Lo interesante es que puedes cambiar de algoritmo entre backups sin romper el repositorio — cada archive recuerda con qué compresión fue creado.

### Flags útiles

| Flag | Para qué sirve |
|------|----------------|
| `--progress` | Muestra los archivos mientras los procesa |
| `--stats` | Muestra resumen al final: tamaño, compresión, deduplicación |
| `--dry-run` | Simula el backup sin escribir nada |
| `--exclude` | Excluir rutas |

## Listar backups

Ver todos los archives del repositorio:

```bash
borg list /ruta/backup
```

Ver el contenido de un archive específico:

```bash
borg list /ruta/backup::home-2026-05-24
```

## Montar y explorar

Puedes montar un archive como si fuera una carpeta normal y explorarla con yazi o Nautilus — es lo que hace Pika Backup por debajo:

```bash
mkdir ~/borg/mount
borg mount /ruta/backup::home-2026-05-24 ~/borg/mount

# cuando termines
borg umount ~/borg/mount
```

Los archivos no son copias, se leen directo del repositorio. Para tener copias reales usa `extract`.

## Restaurar archivos

Siempre hacer `cd` primero a un directorio temporal:

```bash
mkdir -p ~/borg/restore
cd ~/borg/restore
borg extract /ruta/backup::home-2026-05-24
```

Para restaurar solo un archivo o carpeta específica:

```bash
borg extract /ruta/backup::home-2026-05-24 home/graplo/.config/fish
```

Sin `/` inicial — Borg usa rutas relativas.

> **Importante:** `borg extract` no acepta una ruta de destino como argumento — siempre extrae en el directorio actual, por eso el `cd` previo es obligatorio. El argumento opcional al final no es el destino sino el archivo o carpeta específica que quieres extraer:
> 
> ```bash
> # extrae todo el archive en ~/borg/restore
> borg extract /ruta/backup::home-2026-05-24
> 
> # extrae solo una carpeta específica
> borg extract /ruta/backup::home-2026-05-24 home/graplo/.config/fish
> ```

## Prune y compact — limpiar backups viejos

`prune` marca los archives a eliminar según las reglas que definas, `compact` los borra físicamente. Siempre van juntos:

```bash
# primero prueba con --dry-run para ver qué borraría
borg prune --keep-daily=7 --keep-weekly=4 --dry-run /ruta/backup

# si todo se ve bien, ejecuta sin --dry-run
borg prune --keep-daily=7 --keep-weekly=4 /ruta/backup
borg compact /ruta/backup
```

## Automatizar con un script

El verdadero poder de Borg viene cuando lo automatizas. Este es un script funcional para backup del home completo:

```bash
#!/usr/bin/env bash
# set -e: muere si hay error / -u: muere si una variable no existe / -o pipefail: atrapa errores en tuberías
set -euo pipefail

BACKUP_PATH="/run/media/graplo/db2/backup"
now=$(date +"%Y-%m-%d_%H-%M-%S")

borg create --progress --stats --compression auto,zstd,3 "$BACKUP_PATH::home-${now}" \
    --exclude "$HOME/.cache/" \
    --exclude "$HOME/.local/share/Trash" \
    --exclude "$HOME/.local/share/Steam" \
    --exclude "$HOME/Descargas" \
    --exclude "$HOME/.var/app/*/cache" \
    --exclude "$HOME/.local/share/containers" \
    --exclude "$HOME/.ollama" \
    "$HOME"

borg prune --stats --keep-last 7 --glob-archives "home-*" "$BACKUP_PATH"
borg compact "$BACKUP_PATH"

```

`set -euo pipefail` hace que el script muera si hay cualquier error en vez de continuar silenciosamente. Las exclusiones evitan backupear cachés, la papelera, imágenes de Steam, Flatpak, podman y modelos de ollama — todo lo que pesa mucho y se puede volver a descargar.

## Pika Backup y la compresión

Si venías de Pika Backup, tus backups anteriores no tienen compresión — Pika la desactiva por defecto para simplificar la interfaz. Puedes seguir usando el mismo repositorio con zstd desde ahora sin problema, Borg maneja cada archive de forma independiente y no se confunde.

## YouTube

{{< youtube E1sIWAT4VsA >}}
