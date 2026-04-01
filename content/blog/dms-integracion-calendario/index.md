---
title: "Dms Integracion con Calendario"
date: 2026-03-30T16:18:05-05:00
draft: false
summary: "Sincroniza tu calendario de Disroot con tu panel de DMS, sin bases de datos duplicadas."
tags: ["linux", "open source", "CLI"]
categories: ["DMS"]
---

Khal es un programa de calendario basado en la terminal (CLI) para sistemas operativos tipo Unix (Linux, macOS), diseñado para ser ligero, rápido y compatible con estándares iCalendar.

<!--more-->

## Paquetes necesarios

```bash
sudo pacman -S vdirsyncer khal cronie
```

- **vdirsyncer**: El motor que conecta con Disroot y descarga los archivos .ics.

- **khal**: La interfaz de calendario para tu terminal (el reemplazo limpio de calcurse).

- **cronie**: El demonio que se encarga de que todo se actualice solo cada 15 minutos.

## Configurar khal primero

Es importante configurar khal antes que vdirsyncer, porque khal decide dónde guardar los calendarios localmente y vdirsyncer tiene que apuntar a esa misma ruta.

```bash
khal configure
```

El asistente es interactivo:

> En mi caso hice un calendario de 0 con el asistente para q cree la ruta q al final fue en ~/.local/share/khal/calendars/private/personal

> En todo caso, khal crea su config en ~/.config/khal/config. Verifica la ruta exacta donde guarda los calendarios:

```bash
cat ~/.config/khal/config
```
 
Anota el `path` que aparece en `[[private]]` — esa es la ruta que necesitas para vdirsyncer.
 
### Si el wizard crashea, crea el config manualmente:
 
```bash
hx ~/.config/khal/config
```
 
```ini
[calendars]
[[private]]
path = /home/TU_USUARIO/.local/share/khal/calendars/private/personal
type = calendar
 
[locale]
timeformat = %H:%M
dateformat = %Y-%m-%d
longdateformat = %Y-%m-%d
datetimeformat = %Y-%m-%d %H:%M
longdatetimeformat = %Y-%m-%d %H:%M
 
[default]
default_calendar = private
```
## Configurar vdirsyncer
 
```bash
mkdir -p ~/.vdirsyncer
hx ~/.vdirsyncer/config
```
 
Usa la ruta que khal creó (con `/personal` al final, que es la colección que vdirsyncer va a crear):
 
```ini
[general]
status_path = "~/.local/share/vdirsyncer/status"
 
[pair personal_sync]
a = "disroot_remote"
b = "disroot_local"
collections = ["from a", "from b"]
conflict_resolution = "a wins"
metadata = ["color"]
 
[storage disroot_remote]
type = "caldav"
url = "https://cloud.disroot.org/remote.php/dav/"
username = "TU_USUARIO"
password = "TU_CONTRASEÑA_O_APP_PASSWORD"
 
[storage disroot_local]
type = "filesystem"
path = "/home/TU_USUARIO/.local/share/khal/calendars/private/personal"
fileext = ".ics"
```
 
> **Clave:** El `path` de `disroot_local` debe ser exactamente la misma ruta que tiene khal en su config. Si no coinciden, khal no va a ver los eventos.
 
 
## Sincronización inicial
 
```bash
vdirsyncer discover personal_sync
```
 
Va a preguntar si crear la colección `personal` localmente → responde **y**.
 
Luego sincroniza:
 
```bash
vdirsyncer sync
```
 
Debería descargar tus archivos `.ics` de Disroot a la carpeta local.

## Verificar con khal
 
```bash
khal list today 2026-12-31
```
 
> **Importante:** `khal list` sin argumentos solo muestra eventos de hoy. Si tus eventos son en días futuros, usa un rango de fechas como en el ejemplo de arriba.
 
Vista mensual:
 
```bash
khal calendar
```
 
## Auto-sync (opcional)
 
Para que vdirsyncer sincronice automáticamente cada 15 minutos:
 
```bash
crontab -e
```
 
Agrega:
 
```
*/15 * * * * /usr/bin/vdirsyncer sync
```
 
## Comandos útiles de khal
 
```bash
khal list                      # eventos de hoy
khal list today 2026-04-30     # eventos en un rango
khal calendar                  # vista mensual
khal new                       # crear evento interactivo
```

## Troubleshooting
 
- **Error 401 Unauthorized en vdirsyncer discover:**
Usa una App Password de Disroot → `https://cloud.disroot.org/settings/user/security`
 
- **khal list no muestra nada:**
Tus eventos probablemente son en fechas futuras. Usa `khal list today FECHA_FUTURA`.
 
- **khal list: "personal is not valid for default_calendar":**
El `default_calendar` debe coincidir con el nombre entre corchetes `[[nombre]]` en la sección `[calendars]` del config de khal.
 
- **Las rutas no coinciden:**
La causa más común de que khal no muestre eventos. Verifica que `path` en khal y `path` en vdirsyncer apunten exactamente a la misma carpeta.
