---
title: "Migrando Jellyfin de podman run a podman-compose"
date: 2026-04-15T00:02:58-05:00
draft: true
summary: "Cómo migrar Jellyfin de un podman run directo a podman-compose, incluyendo el compose.yml, una función Fish para arrancar y apagar con un solo comando, y el flujo para actualizar la imagen sin perder configuración."
description: "Los contenedores de Podman son inmutables — si querés cambiar un volumen o puerto tenés que recrearlo desde cero. Migré Jellyfin a podman-compose para tener toda la configuración en un archivo YAML editable, más fácil de mantener y documentar."
tags: ["podman", "jellyfin", "self-hosting", "linux", "containers"]
categories: ["Terminal", "Contenedores"]
---

Tenía Jellyfin corriendo con un `podman run` directo que funcionaba bien, pero llegó el momento de mover la biblioteca de medios a un nuevo disco. Ahí descubrí el problema: los contenedores de Podman son **inmutables una vez creados**. No podés editar volúmenes, puertos ni nombre sin borrar y recrear el contenedor desde cero.
 
La solución es `podman-compose` y un archivo `compose.yml` que se convierte en la fuente de verdad del contenedor.
 
## El problema con podman run
 
Cuando creás un contenedor con `podman run`, las rutas y configuración quedan fijas. Si querés cambiar algo — un volumen, un puerto, el nombre — tenés que hacer:
 
```bash
podman stop jellyfin
podman rm jellyfin
podman run ... # con los cambios
```
 
Además, si en seis meses no recordás con qué flags lo creaste, tenés que hacer `podman inspect` para recuperar el comando original. No es práctico.
 
## La solución: podman-compose
 
Con `podman-compose` toda la configuración vive en un archivo YAML. Querés cambiar algo, editas el archivo y en la próxima ejecución toma los cambios.
 
### Instalación
 
```bash
paru -S podman-compose
```
 
### El archivo compose.yml
 
Creo el archivo en una ubicación ordenada:
 
```bash
mkdir -p ~/.config/containers/jellyfin
```
 
El contenido en `~/.config/containers/jellyfin/compose.yml`:
 
```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    ports:
      - "8096:8096"
    volumes:
      - /home/graplo/.config/jellyfin:/config
      - /home/graplo/.cache/jellyfin:/cache
      - /mnt/vault/Music:/media/music
      - /mnt/vault/Movies:/media/movies
```
 
Noto que no uso `restart: unless-stopped`. Jellyfin en mi caso lo arranco bajo demanda — no necesito que corra como demonio permanente consumiendo ~1GB de RAM cuando no lo uso.
 
### Comandos básicos
 
Arrancar:
```bash
podman-compose -f ~/.config/containers/jellyfin/compose.yml up -d
```
 
Apagar:
```bash
podman-compose -f ~/.config/containers/jellyfin/compose.yml down
```
 
El flag `-d` es *detached* — corre en segundo plano sin bloquear la terminal.
 
> **Importante:** `down` para el contenedor y lo borra, pero **no toca la imagen**. No hay que volver a descargar los 1.55GB de Jellyfin cada vez.
 
## Función Fish para uso diario
 
El comando completo es largo para escribirlo cada vez. Lo envuelvo en una función Fish:
 
```fish
function jelly
    set compose_file ~/.config/containers/jellyfin/compose.yml
    if podman ps | grep -q jellyfin
        podman-compose -f $compose_file down
        echo "Jellyfin apagado"
    else
        podman-compose -f $compose_file up -d
        echo "Jellyfin encendido"
    end
end
```
 
Ahora con solo escribir `jelly` el contenedor se levanta o apaga según su estado actual.
 
## Flujo para actualizar Jellyfin
 
```bash
podman pull jellyfin/jellyfin  # descarga la imagen más reciente
jelly                          # apaga si está corriendo
jelly                          # levanta con la nueva imagen
```
 
Para limpiar imágenes viejas:
```bash
podman image prune
```
 
## Por qué vale la pena
 
La ventaja principal no es técnica sino de mantenimiento: el `compose.yml` es documentación. En seis meses — o en una reinstalación — abrís ese archivo y sabés exactamente cómo está configurado Jellyfin sin tener que reconstruir el comando original de memoria.
 
