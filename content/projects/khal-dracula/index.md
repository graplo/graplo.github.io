---
title: "Khal Dracula"
draft: false
summary: "Theme Dracula para khal con soporte hex"
tags: ["khal", "dracula", "terminal", "linux", "configuracion"]
showDate: false
---

Khal soporta personalización de colores en su interfaz TUI (`ikhal`) mediante una sección `[palette]` en el config. Lo que descubrí es que el formato completo acepta hasta 5 valores por entrada, lo que permite usar colores hex exactos de Dracula en terminales modernos como Ghostty.
 
## El formato completo
 
La documentación oficial dice:
 
```
key = foreground, background, mono, foreground_high, background_high
```
 
Los dos primeros (`foreground`, `background`) son para terminales en "low color mode" — solo soportan los 16 colores ANSI básicos. Los últimos dos (`foreground_high`, `background_high`) son para "high color mode" donde se pueden usar colores hex. El campo `mono` es para terminales monocromáticos.
 
Los colores hex deben ir entre comillas simples: `'#bd93f9'`. Sin comillas el `#` se interpreta como comentario y el color se ignora silenciosamente.
 
## Keys disponibles
 
Los keys válidos están definidos en `khal/ui/colors.py`. Los más útiles:
 
```
header, footer, today, today focus, date header, date header focused,
date header selected, dayname, monthname, alert, mark, button,
button focused, eventcolumn, eventcolumn focus, reveal focus,
popupbg, caption, editbx, edit, edit focus
```
 
> Si no puedes cambiar el color de algún elemento, la documentación recomienda abrir un issue en GitHub — no todos los elementos son temizables desde el config.
 
## El color del calendario vs la palette
 
Hay dos sistemas de color independientes en khal:
 
- `color` en `[[calendario]]` — colorea los eventos de ese calendario. Acepta hex directamente sin comillas especiales. Este color se usa también como fondo del panel de detalle en ikhal.
- `[palette]` — controla los colores de la interfaz de ikhal (barra de fechas, encabezados, botones, etc.). Acepta los 16 colores ANSI en low color mode y hex en high color mode.
 
## Theme Dracula completo
 
```ini
[palette]
header = white, black, '', '#f8f8f2', '#282a36'
footer = white, black, '', '#f8f8f2', '#282a36'
today = light magenta, '', '', '#f8f8f2', '#bd93f9'
today focus = light magenta, '', '', '#282a36', '#ff79c6'
date header = light gray, black, '', '#6272a4', '#282a36'
date header focused = black, dark blue, '', '#282a36', '#bd93f9'
date header selected = light gray, dark gray, '', '#f8f8f2', '#44475a'
dayname = light cyan, black, '', '#8be9fd', '#282a36'
monthname = light magenta, '', '', '#bd93f9', ''
alert = white, dark red, '', '#f8f8f2', '#ff5555'
mark = black, light green, '', '#282a36', '#50fa7b'
button = black, light cyan, '', '#282a36', '#8be9fd'
button focused = white, light blue, '', '#282a36', '#bd93f9'
eventcolumn = light cyan, black, '', '#8be9fd', '#282a36'
eventcolumn focus = light cyan, dark gray, '', '#8be9fd', '#44475a'
reveal focus = black, light cyan, '', '#282a36', '#8be9fd'
popupbg = white, black, '', '#f8f8f2', '#282a36'
caption = white, black, '', '#f8f8f2', '#282a36'
editbx = light gray, black, '', '#f8f8f2', '#282a36'
edit = light gray, black, '', '#f8f8f2', '#282a36'
edit focus = black, dark blue, '', '#282a36', '#bd93f9'
```
 
## Colores de los calendarios
 
```ini
[[personal]]
color = '#8be9fd'
 
[[feriados]]
color = '#ffb86c'
```
 
## Nota sobre los colores ANSI en Ghostty con Dracula
 
Los 16 colores ANSI se renderizan diferente según el terminal. En Ghostty con paleta Dracula, `dark blue` (ANSI 4) aparece como el morado/azul Dracula — no como azul puro. Esto es útil para los campos de low color mode, pero puede sorprender si vienes de otro terminal.
 
Para ver cómo se mapean los 16 colores en tu terminal:
 
```fish
for i in (seq 0 15); printf "\033[48;5;%sm  %s  \033[0m\n" $i $i; end
```
 
