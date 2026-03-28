---
title: "bat - Dracula Theme"
date: 2026-03-25
tags: ["linux", "CLI", "rust", "open source"]
categories: ["Terminal"]
---

## Puedes ver los temas disponibles con:
 ```bash
  bat --list-themes
  ```
## Y probar uno específico: En mi caso dracula

 ```bash
  bat --theme="Dracula" ~/.config/fish/config.fish
  ```
## Para hacerlo permanete lo ponemos `~/.config/fish/config.fish`

  ```bash
  set -Ux BAT_THEME Dracula
  ```
## Para saber cual theme esta en el momento

  ```bash
  echo $BAT_THEME
  ```
