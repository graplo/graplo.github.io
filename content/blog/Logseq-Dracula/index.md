---
title: "Logseq - Dracula theme - arreglo"
date: 2026-03-25
tags: ["linux", "GUI", "markdown", "open source"]
categories: ["Software"]
---
Logseq es una
aplicación de toma de notas y gestión del conocimiento de código abierto, centrada en la privacidad, que funciona sobre archivos locales de texto plano (Markdown o Org-mode). Destaca por su sistema de enlaces bidireccionales, lo que permite conectar ideas y visualizar relaciones entre notas en forma de grafo. 

<!--more-->

## Instalar el theme Dracula

  - Usar los tres puntos he instalar el tema de la tienda.
  - Nada dificil, pero el tema esta imcompleto, varias partes del programa estan mal.
  

## Donde ponerlo

    ```bash
    cd ~/Documentos/logseq/logseq/custom.css
    ```
 igual depende de donde tengan su su Logseq.

## El custom css

```css
:root {
  --ls-font-family: "Ubuntu", sans-serif;
  --ls-font-family-ui: "Ubuntu", sans-serif;
  --ls-font-family-code: "JetBrainsMonoNL Nerd Font Mono", monospace;
}

.cm-s-default,
.CodeMirror,
.CodeMirror pre {
  font-family: "JetBrainsMonoNL Nerd Font Mono", monospace !important;
}

/* 2. CONFIGURACIÓN INTEGRAL TEMA DRACULA */
:root,
.dark-theme,
[data-theme="dark"] {
  --ls-primary-background-color: #282a36 !important; /* Centro */
  --ls-secondary-background-color: #21222c !important; /* Laterales más oscuros */
  --ls-tertiary-background-color: #383a59 !important;
  --ls-primary-text-color: #f8f8f2 !important;
  --ls-secondary-text-color: #bd93f9 !important; /* Púrpura Dracula */
  --ls-selection-background-color: #44475a !important;
  --ls-selection-text-color: #ffffff !important;
  --ls-block-bullet-color: #6272a4 !important;
}

/* 3. ARREGLO DE BARRAS LATERALES (Izquierda y Derecha) */
#left-sidebar, 
#left-sidebar .left-sidebar-inner,
.left-menu-item,
#right-sidebar,
.left-sidebar-inner {
  background-color: #21222c !important;
}

#left-sidebar .header,
#left-sidebar .item-content,
#left-sidebar .nav-content-item,
#right-sidebar .sidebar-item {
  color: #f8f8f2 !important;
}

#left-sidebar .nav-content-item:hover {
  background-color: #44475a !important;
}

/* 4. REFUERZO PARA EL EDITOR Y CUERPO CENTRAL */
.cp__sidebar-main-content,
#main-content-container {
  background-color: #282a36 !important;
}

/* REPARACIÓN DEL TÍTULO (Para que vuelva a ser Púrpura) */
.title, 
h1.title, 
.ls-page-title,
h1 {
  color: #bd93f9 !important; /* Aquí devolvemos el morado a "Audio - set up" */
  font-weight: bold !important;
}

.ls-block, 
.ls-block-properties, 
.block-content {
  color: #f8f8f2 !important;
}

/* Color para los links [[ ]] y tags # */
.page-ref, .ls-tag {
  color: #8be9fd !important; /* Cyan */
}

/* Color para los TODOs */
.todo {
  color: #ff79c6 !important; /* Rosa */
}

/* BOTÓN COPY CODE - ESTILO DRACULA */
.copy-code-button,
button[title="Copy code"] {
  background-color: #44475a !important;
  color: #f8f8f2 !important;
  border: 1px solid #6272a4 !important;
  border-radius: 4px !important;
  padding: 4px 8px !important;
  font-size: 12px !important;
  transition: all 0.2s ease !important;
}

.copy-code-button:hover,
button[title="Copy code"]:hover {
  background-color: #bd93f9 !important;
  color: #282a36 !important;
  border-color: #bd93f9 !important;
}

/* BLOQUES DE CÓDIGO - MEJOR CONTRASTE */
.CodeMirror,
pre code {
  background-color: #21222c !important;
  color: #f8f8f2 !important;
  border: 1px solid #44475a !important;
  border-radius: 4px !important;
  padding: 8px !important;
}
```
> cada bloque está explicado
  
  
