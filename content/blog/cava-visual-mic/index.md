---
title: "CAVA con micrófono en PipeWire"
date: 2026-04-30T22:41:31-05:00
draft: true
summary: "CAVA puede visualizar el audio del micrófono además del audio de salida. Con PipeWire y EasyEffects es sencillo configurarlo en un archivo de config separado."
description: "Cómo configurar CAVA para capturar el micrófono usando PipeWire y EasyEffects en CachyOS."
tags: ["linux", "cava", "pipewire", "easyeffects", "audio", "cli"]
categories: ["Terminal"]
---

CAVA por defecto visualiza el audio de salida — lo que suena por los altavoces o auriculares. Pero también puede capturar el micrófono, útil para ver la respuesta del visualizador mientras grabas o hablas.

## Ver los dispositivos de audio disponibles

`pactl` es la herramienta de línea de comandos para controlar PipeWire (y PulseAudio). El nombre viene de **PulseAudio ConTroL** — aunque ahora funciona con PipeWire, mantiene el nombre por compatibilidad.

Para listar las fuentes de audio disponibles:

```bash
pactl list sources short
```

El argumento `short` muestra una línea por dispositivo en vez del detalle completo. El output muestra el número, nombre, servidor, formato y estado de cada fuente.

Un ejemplo de output relevante:


```bash
33      easyeffects_sink.monitor        PipeWire        float32le 2ch 48000Hz   SUSPENDED
34      easyeffects_source      PipeWire        float32le 2ch 48000Hz   SUSPENDED
89      alsa_input.usb-MOTU_M2_M2MA070DWT-00.HiFi__Mic1__source PipeWire        float32le 1ch 48000Hz   SUSPENDED
91      alsa_output.usb-MOTU_M2_M2MA070DWT-00.HiFi__Line__sink.monitor  PipeWire        s32le 2ch 48000Hz       SUSPENDED
95      alsa_output.pci-0000_0c_00.1.hdmi-stereo-extra1.monitor PipeWire        s32le 2ch 48000Hz       SUSPENDED
96      alsa_input.usb-046d_C922_Pro_Stream_Webcam_F55E727F-02.analog-stereo    PipeWire        s16le 2ch 32000Hz       SUSPENDED
97      alsa_output.pci-0000_0e_00.4.iec958-stereo.monitor      PipeWire        s32le 2ch 48000Hz       SUSPENDED
98      alsa_input.pci-0000_0e_00.4.analog-stereo       PipeWire        s32le 2ch 48000Hz       SUSPENDED
```
> esto es un ejemplo, depende de tus dispositivos de audio, en mi caso uso Eassy Effects para poner algunos filtros, y como ven tengo un MOTU M2, depende mucho de su hardware.

Hay dos opciones para capturar el micrófono:

- `alsa_input...Mic1__source` — señal cruda directamente del hardware, sin ningún procesamiento
- `easyeffects_source` — señal ya procesada por EasyEffects (gate, filtro de ruido, EQ, compresor, limitador)

Para visualizar la voz procesada lo correcto es usar `easyeffects_source`.

## Crear un config separado para el micrófono

En vez de modificar el config principal de CAVA, lo más limpio es crear un archivo aparte:

```bash
cp ~/.config/cava/config ~/.config/cava/config-mic
```

Edita `config-mic` y modifica la sección `[input]`:

```ini
[input]
method = pipewire
source = easyeffects_source
```

Y la sección `[output]` para ajustarlo a mono, ya que el micrófono es un dispositivo mono:

```ini
[output]
channels = mono
mono_option = average
```

Sin esto CAVA mostraría la mitad de las barras vacías al recibir señal mono en modo stereo.

## Lanzar CAVA con el config del micrófono

```bash
cava -p ~/.config/cava/config-mic
```

El flag `-p` indica el path al config a usar. Al cerrar CAVA y abrirlo normalmente sin `-p` vuelve al config por defecto capturando el audio de salida.

> El micrófono captura en mono (1 canal). `mono_option = average` toma el promedio del canal para distribuirlo uniformemente en las barras.

Para más detalles sobre cómo aplicar el tema Dracula en CAVA, mira el post anterior: [Aplicar tema Dracula en CAVA](/blog/cava-dracula/)
