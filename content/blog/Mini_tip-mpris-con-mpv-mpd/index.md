---
title: "Mini-tip Mpris Con Mpv Mpd"
date: 2026-08-10T12:46:35-05:00
draft: false
summary: "MPRIS es un protocolo que muchos reproductores no implementan por sí solos. MPD y mpv necesitan un puente aparte para que playerctl, SwayNC o cualquier widget de escritorio los reconozca."
description: "Por qué MPD y mpv no aparecen en herramientas MPRIS como playerctl o SwayNC, y cómo instalar mpd-mpris y mpv-mpris para solucionarlo en Arch/CachyOS."
tags: ["linux", "mpris", "mpd", "mpv", "cli", "audio"]
categories: ["Terminal"]
---

Si alguna vez armaste un widget de reproductor multimedia (en eww, Waybar, SwayNC o similar) y notaste que Firefox o Spotify aparecen sin problema pero MPD o mpv no muestran nada, no es un bug de tu configuración. Es que esos dos reproductores no hablan MPRIS por sí solos.

## Qué es MPRIS

MPRIS es un protocolo estándar sobre D-Bus que permite controlar reproductores multimedia y leer sus metadatos (título, artista, portada, progreso) desde cualquier programa que sepa consumirlo. `playerctl`, SwayNC, DankMaterialShell, o un widget propio en eww son todos "clientes" MPRIS: escuchan D-Bus y muestran lo que encuentren.

El problema no está en el cliente que usas para mostrarlo. Está en si el reproductor en sí se registra en D-Bus como fuente MPRIS. Firefox, Chromium, VLC y la app oficial de Spotify lo traen integrado de fábrica. MPD y mpv, no.

## Por qué MPD y mpv son casos distintos

MPD es un servidor que corre en segundo plano de forma indefinida, reproduzcas o no algo con un cliente como `rmpc` en ese momento. Como es un proceso persistente separado del cliente, el puente hacia MPRIS también necesita ser un proceso persistente aparte, vigilando constantemente los cambios de estado del daemon.

mpv, en cambio, no es un servidor. Vive y muere junto a su propia ventana. Su soporte MPRIS se resuelve con un plugin que se carga dentro del propio proceso de mpv al arrancar, sin necesitar ningún servicio separado.

## Puente para MPD

```bash
sudo pacman -S mpd-mpris
```

Como MPD sigue vivo aunque cierres el cliente, `mpd-mpris` necesita arrancar junto con tu sesión y quedarse corriendo en background:

```bash
systemctl --user enable --now mpd-mpris
```

A partir de ahí, cualquier cambio de estado en MPD (reproducir, pausar, cambiar de canción) se refleja automáticamente en `playerctl` y en cualquier widget que lo consuma.

## Plugin para mpv

```bash
sudo pacman -S mpv-mpris
```

Acá no hace falta ningún `systemctl`. El paquete deja el plugin en la carpeta de scripts de mpv (`~/.config/mpv/scripts/` o `/etc/mpv/scripts/`), y mpv lo carga solo la próxima vez que abras el reproductor. Mientras esa ventana de mpv esté reproduciendo algo, aparece en `playerctl -l`; en cuanto la cierras, desaparece, igual que pasa con Firefox.

> Si tenés dudas de si tu reproductor de turno necesita algo similar, `playerctl -l` te lista los reproductores MPRIS activos en cualquier momento — es la forma más rápida de confirmar si algo se está registrando o no.

## Resultado

Con los dos puentes instalados, MPD y mpv quedan a la par de Firefox y Spotify: cualquier cliente MPRIS que ya tengas configurado (playerctl, SwayNC, tu propio widget) los reconoce sin más cambios de tu parte.
