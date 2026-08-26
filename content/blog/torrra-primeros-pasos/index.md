---
title: "Torrra - Primeros Pasos"
date: 2026-03-25
tags: ["linux", "TUI", "torrent", "open source"]
categories: ["Terminal"]
---

TUI para buscar y descargar torrents desde la terminal, powered by Jackett.
## Instalación

 ```bash
 paru -S jackett-bin
 paru -S torrra
 ```
## Configuración de torrra en ~/.config/torrra/config.toml:

```toml
 [general]
 download_path = "/home/graplo/Descargas"
 download_in_external_client = false
 use_transmission = false
 transmission_user = ""
 transmission_pass = ""
 theme = "dracula"
 timeout = 10
 max_retries = 3
 use_cache = true
 cache_ttl = 300
 
 [indexers]
 default = "jackett"
 
 [indexers.jackett]
 url = "http://localhost:9117"
 api_key = "TU_API_KEY"
```
## Iniciar Jackett

 ```bash
 sudo systemctl start jackett
 ```
### si queremos parar el demonio jackett

  ```bash
  sudo systemctl stop jackett
  ```
## Interfaz web de Jackett en http://localhost:9117:

- Copiar API key al `config.toml`
- Agregar trackers (The Pirate Bay, YTS, etc.)

## Usar torrra

 ```bash
 torrra
 ```
### Alternativa para no escribir toda la palabra **torrra**

 ```bash
 alias rrr="torrra"
 funcsave rrr
 ```
> con funcsave es para que se quede el comando permanente.

### Con eso solo se pone `rrr` en la terminal:

  ```bash
  rrr
  ```
> **Actualización — 26/08/2026:** Después de utilizar Torrra durante un tiempo, encontré algunos detalles adicionales relacionados con Jackett y los indexadores. Los dejo aquí para complementar esta guía.

## Algunas cosas a tener en cuenta

Después de usar Torrra con Jackett durante un tiempo, encontré un par de detalles que conviene conocer.

### Indexadores que fallan aunque pasen el Test

Que un indexador aparezca correctamente y pase el botón **Test** de Jackett no garantiza que vaya a funcionar durante una búsqueda real.

Por ejemplo, un indexador puede cambiar la estructura de su página y hacer que Jackett ya no pueda interpretar los resultados, o simplemente puede tardar demasiado en responder.

En mi caso, Jackett mostraba errores como:

```text
Error while parsing field=category
Selector "td.col-cat" didn't match
```
El resultado era que Torrra mostraba Failed indexer al realizar una búsqueda.

La solución fue entrar en la interfaz web de Jackett, probar los indexadores individualmente y desactivar los que estaban fallando. Los demás continuaron funcionando normalmente.

Por eso, si Torrra muestra Failed indexer, no significa necesariamente que Torrra o Jackett estén mal configurados. Conviene revisar el log de Jackett:

```bash
journalctl -u jackett --since "5 minutes ago" --no-pager
```
También podemos filtrar solamente errores:

```bash
journalctl -u jackett --since "5 minutes ago" --no-pager | grep -i -E "error|fail|exception|timeout"
```

### API key de Jackett

La API key se encuentra en:

```bash
/var/lib/jackett/ServerConfig.json
```
dentro del campo:

```json
"APIKey": "TU_API_KEY"
```
La clave generada por Jackett puede utilizarse directamente en Torrra. También es posible establecer manualmente una clave propia en ServerConfig.json.

Después de cambiarla, hay que utilizar la misma clave en Torrra:

```toml
[indexers.jackett]
url = "http://localhost:9117"
api_key = "torrra-jackett-2026"
```
En mi caso mantengo la clave aleatoria generada por Jackett, ya que no tengo ninguna razón para cambiarla.

### ¿Qué indexadores utilizar?

No es necesario añadir muchos trackers. Es preferible tener unos pocos que funcionen correctamente a tener una lista enorme de indexadores con varios que fallen.

Si un indexador empieza a producir errores constantemente, puede desactivarse desde la interfaz web de Jackett y comprobar después si las búsquedas de Torrra funcionan correctamente con los restantes.

