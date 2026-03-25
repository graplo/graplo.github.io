---
title: "bat - Dracula Theme"
date: 2026-03-25
tags: ["linux", "CLI", "rust", "open source"]
---

1. ## Puedes ver los temas disponibles con:
 ```bash
  bat --list-themes
  ```
2. ## Y probar uno específico: En mi caso dracula

 ```bash
  bat --theme="Dracula" ~/.config/fish/config.fish
  ```
3. ## Para hacerlo permanete lo ponemos `~/.config/fish/config.fish`

  ```bash
  set -Ux BAT_THEME Dracula
  ```
4. ## Para saber cual theme esta en el momento

  ```bash
  echo $BAT_THEME
  ```
