---
title: "Automatizar backups con Borg desde la terminal"
draft: true
summary: "Del comando largo que nunca recordás al script que corre solo. Cómo automatizar tus backups con Borg."
tags: ["linux", "borg", "backup", "bash", "terminal"]
categories: ["Terminal", "Homelab"]
showDate: false
---

Si usás Borg para tus backups, probablemente conocés bien ese comando largo que 
tenés guardado en algún lado para no olvidarlo:

```bash
borg create --progress --stats \
    /ruta/backup::"home-{now:%Y-%m-%d}" ~/ \
    --exclude ~/.cache/ \
    --exclude ~/.local/share/Trash \
    --exclude ~/.local/share/Steam \
    --exclude ~/Descargas
```

Funciona perfecto, pero escribirlo cada vez es tedioso y fácil de equivocarse.
La solución es simple — un script que lo haga por vos.

## El script

Creá el archivo en `~/.local/bin/borg_backup`:

```bash
#!/usr/bin/env bash
set -euo pipefail

BACKUP_PATH="/ruta/a/tu/repositorio/borg"
now=$(date +"%Y-%m-%d")

borg create --progress --stats "$BACKUP_PATH::home-${now}" \
    --exclude "$HOME/.cache/" \
    --exclude "$HOME/.local/share/Trash" \
    --exclude "$HOME/.local/share/Steam" \
    --exclude "$HOME/Descargas" \
    "$HOME"

borg prune --keep-last 7 "$BACKUP_PATH"

```

Hacelo ejecutable:
```bash
chmod +x ~/.local/bin/borg_backup
```

Ahora en vez del comando largo, solo escribís:
```bash
borg_backup
```

## Qué hace cada parte

`set -euo pipefail` hace que el script se detenga inmediatamente si algo falla, 
en vez de continuar silenciosamente con errores.

`now=$(date +"%Y-%m-%d")` captura la fecha del día automáticamente — cada backup 
queda nombrado como `home-2026-04-05`, `home-2026-04-06`, etc.

`borg prune --keep-last 7` limpia los backups viejos al terminar, conservando 
solo los 7 más recientes. Sin esto el repositorio crece indefinidamente.

## Ver los backups creados

Para listar todos los backups en el repositorio:

```bash
borg list /ruta/a/tu/repositorio/borg
```

Verás algo así:
home-2026-03-30   Sun, 2026-03-30 11:00:00
home-2026-03-31   Mon, 2026-03-31 11:00:00
home-2026-04-01   Tue, 2026-04-01 11:00:00
home-2026-04-05   Sat, 2026-04-05 12:34:17

Si tenés la variable `BACKUP_PATH` exportada en tu shell, podés usarla 
directamente:

```bash
borg list $BACKUP_PATH
```

## Por qué `$HOME` y no `~/`

Dentro de scripts bash, `~/` no siempre se expande correctamente cuando va 
entre comillas. `$HOME` es más explícito y fiable — apunta al mismo lugar 
pero sin ambigüedad.

---

En un próximo post veremos cómo instalar y configurar Borg desde cero para 
quien recién empieza.
