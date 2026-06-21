---
title: "Stasis Lock Suspend"
date: 2026-06-20T19:45:02-05:00
draft: false
summary: "Cómo instalar y configurar Stasis, un gestor de inactividad para Wayland, en CachyOS con niri."
description: "Guía paso a paso para instalar Stasis desde AUR con paru, configurar el servicio systemd de usuario, y armar un archivo RUNE básico con perfiles para escritorio y gaming."
tags: ["linux", "wayland", "niri", "cachyos", "stasis"]
categories: ["Linux", "Niri"]
---

## Instalación de Stasis desde AUR con paru

Como usamos CachyOS (basado en Arch), instalamos Stasis desde AUR con `paru`.

### Primero buscamos el paquete:

```bash
paru -Ss stasis
```
```bash
aur/stasis 1.3.0-1 [+3 ~0.05] [Instalado]
    A modern Wayland idle manager designed for simplicity and effectiveness
```
> En mi caso está instalado por q el tutorial lo hice despues

### Antes de instalar revisamos la información del paquete:

```bash
paru -Si stasis
```

```bash
Repositorio               : aur
Nombre                    : stasis
Versión                   : 1.3.0-1
Descripción               : A modern Wayland idle manager designed for simplicity and effectiveness
URL                       : https://github.com/saltnpepper97/stasis
AUR URL                   : https://aur.archlinux.org/packages/stasis
Grupos                    : Ninguno
Licencias                 : MIT
Provee                    : Ninguno
Depende De                : systemd  dbus  libinput  wayland
Construir Dependencias    : cargo  rust
Comprobar Dependencias    : Ninguno
Dependencias Opcionales   : libnotify  pipewire-pulse  pulseaudio
En Conflicto Con          : stasis-git
Encargado                 : codedaemon
Votos                     : 3
Popularidad               : 0.047977
Subido por primera vez    : Thu, 25 Sep 2025 23:39:23
Modificado por última vez : Thu, 11 Jun 2026 14:39:42
Desactualizado            : No
```
> Dado el malware q se encontró en AUR no está demas ver si el paquete viene del desarrollador del mismo programa.

### Instalación

```bash
paru -S stasis
```
## Revisar el servicio systemd incluido

### Stasis servicio de usuario de systemd:

Comprobamos si el demon está corriendo

```bash
systemctl --user status stasis
```
nos aparece como `Active: inactive (dead)`

### Activamos el demonio

Con esto solo está activo para esta sesión

```bash
systemctl --user start stasis
```
con esto otro se queda ya siempre activo aun que apagues o reinicies

```bash
systemctl --user enable stasis
```
y conprobamos

```bash
systemctl --user status stasis
```
Nos sale este mensaje en terminal

```bash
● stasis.service - Stasis Wayland Idle Manager
     Loaded: loaded (/usr/lib/systemd/user/stasis.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-06-20 17:46:33 -05; 2h 18min ago
 Invocation: bf749f3e85a44201b5a882044a138fd7
   Main PID: 147991 (stasis)
      Tasks: 17 (limit: 38325)
     Memory: 5M (peak: 308.3M)
        CPU: 1min 31.333s
     CGroup: /user.slice/user-1000.slice/user@1000.service/app.slice/stasis.service
             └─147991 /usr/bin/stasis

jun 20 17:46:33 cachyos systemd[855]: Started Stasis Wayland Idle Manager.
```
## Archivo de configuración

### Creamos la carpeta y ponemos el archivo de ejemplo:

```bash
mkdir -p ~/.config/stasis
```
Copiamos su ejemplo para poder chismosear y ver como queda

```bash
cp /usr/share/stasis/examples/stasis.rune ~/.config/stasis/stasis.rune
```
Le hacemos un .bak para tener el ejemplo a modo de consulta
```bash
cp ~/.config/stasis/stasis.rune ~/.config/stasis/stasis.rune.bak
```
## Configuración básica usada

Para nuesta configuracion probamos con algo, básico por ahora

```ini
@author "Graplo"
@description "Configuración Stasis para CachyOS + Niri"

# ============================================================
# CONFIGURACIÓN BASE
#
# En un PC de escritorio este bloque funciona como el plan activo.
# En una laptop Stasis puede usar default.ac o default.battery.
# ============================================================

default:

  # ============================================================
  # OPCIONES GENERALES
  # ============================================================

  # No usamos integración directa con logind.
  enable_loginctl false

  # Respeta bloqueos de suspensión enviados por D-Bus.
  # Ejemplo: reproductores multimedia, presentaciones, etc.
  enable_dbus_inhibit true

  # Detecta reproductores multimedia mediante MPRIS.
  # Ejemplo: Haruna, mpv, etc.
  monitor_media true

  # Permite considerar también reproductores remotos.
  ignore_remote_media false

  # Evita falsos reinicios del contador por eventos rápidos.
  debounce_seconds 5


  # ============================================================
  # NOTIFICACIONES
  #
  # Stasis usa Desktop Notifications por D-Bus.
  # Como tienes mako, él será quien las muestre.
  # No necesitas ejecutar notify-send manualmente.
  # ============================================================

  notify_on_unpause true
  notify_before_action true


  # ============================================================
  # APLICACIONES QUE PUEDEN EVITAR EL BLOQUEO
  #
  # Estas son excepciones manuales.
  # La multimedia normalmente se detecta con monitor_media.
  # ============================================================

  inhibit_apps [
    "mpv"
    "vlc"
    r"kdenlive.*"
  ]


  # ============================================================
  # DESKTOP PLAN
  #
  # Este es el plan usado en tu PC de escritorio.
  #
  # Flujo:
  #   15 minutos sin actividad -> lock
  #   30 minutos sin actividad -> suspensión
  # ============================================================


  lock_screen:
    timeout 900

    # Ejecuta tu script:
    # ~/.local/bin/lock
    command "lock"

    # Aviso 10 segundos antes del bloqueo.
    notification "Locking screen in 10 seconds"
    notify_seconds_before 10
  end


  suspend:
    timeout 1800
    command "systemctl suspend"
  end

end


# ============================================================
# PERFIL GAMING
#
# NO ES UN PLAN.
# Es un perfil que modifica temporalmente el plan base.
#
# Se activa con:
#   stasis profile gaming
#
# Vuelve al normal con:
#   stasis profile
# ============================================================

gaming:

  mode "overlay"

  # Juegos de Steam y binarios Linux.
  # Mientras estén activos Stasis no actuará.
  inhibit_apps [
    r"steam_app_.*"
    r".*\.x86_64"
  ]

end
```

### Comandos útiles de `Stasis`

```bash
stasis --help
Stasis idle manager

Usage: stasis [OPTIONS] [COMMAND]

Commands:
  reload          Reload the configuration without restarting Stasis
  pause           Pause timers indefinitely, for a duration, or until a time
  resume          Resume timers after a pause
  list            List actions or profiles
  trigger         Manually trigger an idle action by name
  toggle-inhibit  Toggle manual idle inhibition
  tray            Run the optional system tray frontend
  stop            Stop all running Stasis instances
  info            Display current session information
  dump            Dump recent log lines
  profile         Switch profile or return to base config
  help            Print this message or the help of the given subcommand(s)

Options:
  -c, --config <FILE>
  -v, --verbose
      --no-console
      --timestamps
  -h, --help           Print help
  -V, --version        Print version
```
