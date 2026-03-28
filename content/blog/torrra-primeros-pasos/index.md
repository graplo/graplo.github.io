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
