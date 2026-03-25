---
title: "Torrra - Primeros Pasos"
date: 2026-03-25
tags: ["linux", "TUI", "torrent", "open source"]
---

- TUI para buscar y descargar torrents desde la terminal, powered by Jackett.
- ## Instalación
  logseq.order-list-type:: number
  ```bash
  paru -S jackett-bin
  paru -S torrra
  ```
- ## Configuración de torrra en ~/.config/torrra/config.toml:
  logseq.order-list-type:: number
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
- ## Iniciar Jackett
  logseq.order-list-type:: number
  ```bash
  sudo systemctl start jackett
  ```
	- si queremos parar el demonio jackett
	  logseq.order-list-type:: number
	  ```bash
	  sudo systemctl stop jackett
	  ```
- ## Interfaz web de Jackett en http://localhost:9117:
  logseq.order-list-type:: number
	- Copiar API key al config.toml
	  logseq.order-list-type:: number
	- Agregar trackers (The Pirate Bay, YTS, etc.)
	  logseq.order-list-type:: number
- ## Usar torrra
  logseq.order-list-type:: number
  ```bash
  torrra
  ```
- ## Alternativa para no escribir toda la palabra **torrra**
  logseq.order-list-type:: number
  ```bash
  alias --save rrr="torrra"
  ```
	- Con eso solo se pone `rrr` en la terminal:
	  logseq.order-list-type:: number
	  ```bash
	  rrr
	  ```
- logseq.order-list-type:: number
