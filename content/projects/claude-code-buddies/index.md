---
title: "Claude Code Buddies en WebM"
draft: true
summary: ""
tags: ["python", "cli", "open-source", "dracula"]
categories: ["Linux", "Open Source"]
showDate: false
---
 
El 1 de abril de 2026 Anthropic lanzó una sorpresa escondida en Claude Code — un sistema de mascotas virtuales llamado `/buddy`. Por un error accidental, el código fuente de Claude Code se filtró en npm un día antes, y la comunidad encontró rápidamente la carpeta `src/buddy/` con todos los sprites ASCII de los 18 animales.

<!--more-->
 
Un usuario en GitHub publicó el código de los sprites en un [gist](https://gist.github.com/zmxv/7f83671f860c15be02f45b07fee207fc), y a partir de ahí surgió la idea — convertir esos ASCII animados en archivos WebM con fondo transparente para poder usarlos en cualquier contexto.
 
El proceso fue vibe coding es decir (no se programar): yo hacía preguntas, pedía cosas, y Claudia (mi nombre para Claude) escribía el código. Python con Pillow renderiza cada frame del sprite como PNG, y ffmpeg los convierte a WebM con codec VP9 y canal alpha para la transparencia. Los colores son Dracula purple `#bd93f9`, claro.
 
El resultado son 18 archivos WebM de entre 14 y 16 KB cada uno — muy ligeros para ser videos animados con transparencia real.
 
---
<style>
  /* Solo afecta a los <small> dentro de #buddies-table */
  #buddies-table small {
    color:#8be9fd;
    font-size: 1rem;
    font-weight: normal;
  }
</style>


<div id="buddies-table" style="display: grid; grid-template-columns: repeat(3, 1fr); gap: 1rem; text-align: center;">
 
<div>
<video src="/img/buddies/duck.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>duck</small>
</div>
 
<div>
<video src="/img/buddies/goose.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>goose</small>
</div>
 
<div>
<video src="/img/buddies/blob.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>blob</small>
</div>
 
<div>
<video src="/img/buddies/cat.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>cat</small>
</div>
 
<div>
<video src="/img/buddies/dragon.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>dragon</small>
</div>
 
<div>
<video src="/img/buddies/octopus.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>octopus</small>
</div>
 
<div>
<video src="/img/buddies/owl.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>owl</small>
</div>
 
<div>
<video src="/img/buddies/penguin.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>penguin</small>
</div>
 
<div>
<video src="/img/buddies/turtle.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>turtle</small>
</div>
 
<div>
<video src="/img/buddies/snail.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>snail</small>
</div>
 
<div>
<video src="/img/buddies/ghost.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>ghost</small>
</div>
 
<div>
<video src="/img/buddies/axolotl.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>axolotl</small>
</div>
 
<div>
<video src="/img/buddies/capybara.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>capybara</small>
</div>
 
<div>
<video src="/img/buddies/cactus.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>cactus</small>
</div>
 
<div>
<video src="/img/buddies/robot.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>robot</small>
</div>
 
<div>
<video src="/img/buddies/rabbit.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>rabbit</small>
</div>
 
<div>
<video src="/img/buddies/mushroom.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>mushroom</small>
</div>
 
<div>
<video src="/img/buddies/chonk.webm" autoplay loop muted playsinline style="width:50%"></video>
<small>chonk</small>
</div>
 
</div>
