---
title: "Khal Feriados Peru"
date: 2026-03-31T20:28:34-05:00
draft: false
summary: "Calendario de feriados en Khal"
tags: ["khal", "calendario", "linux", "CLI"]
categories: ["Terminal"]
---

Khal puede manejar varios calendarios a la vez, cada uno con su propio color. Esto permite tener los feriados nacionales separados de los eventos personales, sin necesidad de sincronizarlos con Disroot.

## Descargar el .ics de Thunderbird

Thunderbird mantiene calendarios de feriados para muchos países en formato `.ics` listo para importar:

```bash
curl -o ~/Descargas/PeruHolidays.ics "https://www.thunderbird.net/media/caldata/autogen/PeruHolidays.ics"
```

La lista completa de países está en [thunderbird.net/en-US/calendar/holidays](https://www.thunderbird.net/en-US/calendar/holidays).

## Crear la carpeta local

```bash
mkdir -p ~/.local/share/khal/calendars/private/feriados
```

## Agregar el calendario en khal

En `~/.config/khal/config`, dentro de la sección `[calendars]`:

```ini
[[feriados]]
path = /home/graplo/.local/share/khal/calendars/private/feriados
type = calendar
color = '#ffb86c'
```

## Importar el .ics

```bash
khal import --batch -a feriados ~/Descargas/PeruHolidays.ics
```

La opción `-a` especifica el calendario destino. Sin `--batch` khal pregunta confirmación por cada evento — con feriados son muchos, así que `--batch` ahorra tiempo.

## Por qué no sincroniza con Disroot

Vdirsyncer solo conoce las carpetas que están en su config (`~/.vdirsyncer/config`). Como `/feriados` no está ahí, nunca la toca. Los feriados quedan locales, lo cual está bien — son datos estáticos que no cambian frecuentemente.

Disroot ya tiene los feriados integrados como suscripción interna en su interfaz web, pero esa suscripción no es accesible vía CalDAV para vdirsyncer. La solución local con el `.ics` de Thunderbird es más simple y funciona igual de bien.

## Actualizar los feriados

El `.ics` de Thunderbird cubre 2025-2028. Cuando expire, solo hay que repetir el proceso: descargar el nuevo archivo e importarlo. Khal preguntará si actualizar los eventos existentes — con `--batch` los sobreescribe automáticamente.

