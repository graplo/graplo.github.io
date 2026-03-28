---
title: "rmpc - instalación"
date: 2026-03-26
tags: ["linux", "TUI", "rust", "open source"]
categories: ["Terminal"]
---

{{<icon"music">}} Rmpc es un cliente moderno y rápido para Music Player Daemon (MPD). Escrito en Rust, está diseñado para funcionar en la terminal (TUI) de sistemas Linux. Se caracteriza por ser altamente configurable, tener soporte para carátulas de álbumes y ofrecer una experiencia de navegación de tres columnas tipo ranger.

<!--more-->

## Instalar paquetes necesarios {{<icon"dev">}} 

```bash
sudo pacman -S --needed mpd mpc rustup rmpc 
```
### Opcional (extras recomendados):

```bash
sudo pacman -S --needed cava ffmpeg yt-dlp ueberzugpp
```
## Crear configuración de MPD (modo usuario)

Crear carpeta:

```bash
mkdir -p ~/.config/mpd
```

Crear archivo:

> Yo uso Helix (hx) pero puede ser el q gustes como nano, o cualquier editor de texto

```bash
hx ~/.config/mpd/mpd.conf
```

### Contenido mínimo funcional:

```bash
music_directory    "~/Música"
playlist_directory "~/.config/mpd/playlists"

db_file            "~/.local/share/mpd/database"
log_file           "~/.local/share/mpd/log"
state_file         "~/.local/share/mpd/state"
sticker_file       "~/.local/share/mpd/sticker.sql"

bind_to_address    "127.0.0.1"
port               "6600"

auto_update        "yes"
restore_paused     "yes"

audio_output {
  type "pipewire"
  name "PipeWire Sound Server"
}
```
## Crear directorios y archivos necesarios

> Aun q existan no sobreescribe solo actualiza la fecha de modificación

```bash
mkdir -p ~/.config/mpd/playlists
mkdir -p ~/.local/share/mpd

touch ~/.local/share/mpd/database
touch ~/.local/share/mpd/log
touch ~/.local/share/mpd/state
touch ~/.local/share/mpd/sticker.sql
```
## Activar MPD como servicio de usuario

```bash
systemctl --user enable --now mpd
```

### Verificar:

```bash
systemctl --user status mpd
```

Debe mostrar:

```bash
Active: active (running)
```
## Activar MPD como servicio de usuario

```
systemctl --user enable --now mpd
```

### Verificar:

```bash
systemctl --user status mpd
```

Debe mostrar:

```bash
Active: active (running)
```
## Indexar música (primera vez)

```bash
mpc update
```

Probar conexión:

```bash
mpc status
```
## Ejecutar rmpc

```bash
rmpc
```
## Flujo final

```
MPD (daemon audio)
 ↓
mpc (control CLI)
 ↓
rmpc (interfaz TUI)
```

