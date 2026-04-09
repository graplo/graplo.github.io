---
title: "Jellyfin con Podman en CachyOS"
date: 2026-04-08T21:49:16-05:00
draft: false
summary: "Podman permite correr Jellyfin como contenedor bajo demanda, sin demonio ni servicios activos. Más ligero que el Flatpak y fácil de actualizar."
description: "Cómo instalar Jellyfin usando Podman en CachyOS, sin systemd ni servicios en background."
tags: ["linux", "podman", "jellyfin", "containers", "cli"]
categories: ["Terminal"]
---

Jellyfin está disponible como Flatpak, pero tiene un demonio que queda corriendo en background. Con Podman podés correrlo solo cuando lo necesitás, sin servicios activos ni overhead innecesario.

## Requisitos previos

Podman debe estar instalado. En CachyOS está en los repos oficiales:

```bash
paru -S podman
```

Por defecto Podman no tiene registros de búsqueda configurados. Para poder buscar y descargar imágenes de Docker Hub, agrega esta línea al inicio de `/etc/containers/registries.conf`:

```bash
sudo helix /etc/containers/registries.conf
```
> Helix es el editor que uso — podés usar cualquier otro: `nano`, `vim`, `gedit`, etc.

```toml
unqualified-search-registries = ["docker.io"]
```

Sin esto, Podman no sabe dónde buscar imágenes por nombre corto y hay que usar el nombre completo del registro en cada comando.

## Descargar la imagen

```bash
podman pull docker.io/jellyfin/jellyfin
```

Jellyfin tiene imagen oficial en Docker Hub mantenida por su propio equipo. El proceso descarga la imagen y la almacena localmente — no hace falta repetirlo salvo para actualizar.

## Crear las carpetas de configuración

```bash
mkdir -p ~/.config/jellyfin ~/.cache/jellyfin
```

Jellyfin guarda su configuración y base de datos en `~/.config/jellyfin`. El cache va en `~/.cache/jellyfin`. Ambas carpetas persisten entre reinicios del contenedor.

## Crear el contenedor

```bash
podman run -d \
  --name jellyfin \
  -p 8096:8096 \
  -v ~/.config/jellyfin:/config \
  -v ~/.cache/jellyfin:/cache \
  -v ~/Música:/media/musica \
  -v ~/Movies:/media/movies \
  docker.io/jellyfin/jellyfin
```

Este comando solo se ejecuta una vez. Los `-v` mapean carpetas del sistema al contenedor — la configuración y tu media quedan fuera del contenedor, así que si lo borras no pierdes nada.

El puerto `8096` queda mapeado al mismo puerto del host. Jellyfin es accesible en `http://localhost:8096`.

## Uso diario

Una vez creado el contenedor, para arrancarlo y pararlo:

```bash
podman start jellyfin
podman stop jellyfin
```

Para simplificarlo con una función Fish:

```fish
function jelly
    if podman ps | grep -q jellyfin
        podman stop jellyfin
        echo "Jellyfin apagado"
    else
        podman start jellyfin
        echo "Jellyfin encendido"
    end
end
```

Guárdala en `~/.config/fish/functions/jelly.fish` y desde cualquier terminal basta con escribir `jelly` para alternar el estado.

> Esta función es específica para Fish shell. Si usás Bash o Zsh, el equivalente sería un alias o función con sintaxis diferente.

## Actualizar Jellyfin

```bash
podman pull docker.io/jellyfin/jellyfin
podman stop jellyfin
podman rm jellyfin
podman run -d \
  --name jellyfin \
  -p 8096:8096 \
  -v ~/.config/jellyfin:/config \
  -v ~/.cache/jellyfin:/cache \
  -v ~/Música:/media/musica \
  -v ~/Movies:/media/movies \
  docker.io/jellyfin/jellyfin
```

El `podman rm` solo borra el contenedor, no los volúmenes. Tu configuración y media quedan intactas.

## Comandos útiles

Para verificar que el contenedor está corriendo:

```bash
podman ps
```

Para ver todas las imágenes descargadas:

```bash
podman images
```

Para ver los logs de Jellyfin si algo falla:

```bash
podman logs jellyfin
```

> Estos comandos son básicos en cualquier flujo de trabajo con Podman o Docker. `podman ps` sin argumentos muestra solo los contenedores activos — `podman ps -a` muestra todos, incluyendo los detenidos.
