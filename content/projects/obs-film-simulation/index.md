---
title: "LUTs de Emulación de Película para OBS Studio"
draft: true
summary: "Cómo convertí una colección de LUTs de RawTherapee al formato que usa OBS, y cómo lo pueden hacer ustedes con cualquier LUT que tengan."
tags: ["linux", "obs", "luts", "streaming"]
showDate: false
categories: ["projects"]
---

Hace un tiempo empecé a jugar con LUTs para mejorar el color de mi cámara en OBS, y me encontré con la [colección de RawTherapee](https://github.com) de Pat David, Pavlov Dmitry y Michael Ezra: una serie de emulaciones de película análoga (Kodak, Fuji, Polaroid, blanco y negro) hechas originalmente para edición de fotos. El problema es que OBS no lee ese formato directamente, así que tuve que convertirlas, y de paso armé un repo con todo el proceso y el resultado ya listo para usar: [OBS Film Simulation LUTs](https://github.com/graplo/obs-film-luts).

## Qué es en realidad un LUT

LUT significa "Lookup Table", tabla de búsqueda. La idea es simple: por cada color que entra, la tabla dice qué color debe salir. Un LUT de "Kodak Portra", por ejemplo, no es más que una tabla enorme que dice "este verde apagado, conviértelo en este otro verde más cálido", repetido para prácticamente todos los colores posibles.

El formato "Hald CLUT" es una forma de guardar esa tabla como si fuera una imagen normal. Se toma una imagen cuadrada que contiene, en orden, todos los colores posibles (o una muestra representativa), se le aplica el efecto que se quiere lograr, y el resultado —esa imagen ya modificada— es el LUT. Cuando un programa "aplica" el LUT, en realidad está comparando cada píxel de tu video con esa imagen de referencia y haciendo el reemplazo de color correspondiente.

Las de RawTherapee vienen en formato cuadrado de 512x512. OBS, en cambio, espera el mismo tipo de tabla pero acomodada en una tira horizontal. Es la misma información, solo que reorganizada en otra forma.

## Convertir un LUT para que OBS lo entienda

Acá es donde entra ImageMagick, una herramienta de línea de comandos que ya casi todo el mundo en Linux tiene instalada o puede instalar con un solo paquete. ImageMagick trae un modo especial llamado `-hald-clut` que hace exactamente esa reorganización: toma un LUT cuadrado como el de RawTherapee y lo redibuja como una tira horizontal, tomando como base una imagen de referencia neutra que ya trae OBS instalada en su propia carpeta de plugins.

El comando completo es este:

```bash
magick /usr/share/obs/obs-plugins/obs-filters/LUTs/original.png input_hald.png -hald-clut output_obs.png
```

Desglosado: `original.png` es la imagen de referencia neutra de OBS (la "identidad", el mapa de colores sin ningún efecto aplicado), `input_hald.png` es el LUT cuadrado que quieres convertir, y `output_obs.png` es el nombre que le quieres dar al resultado ya convertido. ImageMagick usa la referencia de OBS para saber exactamente en qué orden y tamaño tiene que acomodar los colores de tu LUT para que la tira final sea compatible.

Si tienen su propio LUT en formato Hald cuadrado (de cualquier fuente, no solo el de RawTherapee), ese mismo comando les sirve para pasarlo a OBS.

## Convertir muchos LUTs de una sola vez

Cuando tuve que convertir la colección entera, que trae decenas de archivos organizados por carpetas (monochrome, polaroid, kodak, etc.), hacerlo uno por uno no tenía sentido. Uso la shell Fish, y ahí un bucle `for` recorre todos los archivos de una carpeta y les aplica el mismo comando:

```fish
for f in *.png
    magick /usr/share/obs/obs-plugins/obs-filters/LUTs/original.png "$f" -hald-clut "/home/graplo/Imágenes/OBS/LUT_pack/graplo_obs_luts/monochrome/polaroid/obs_$f"
end
```

La lógica es: por cada archivo `.png` en la carpeta donde estoy parado, guárdalo en la variable `$f`, corre el mismo comando de conversión que vimos arriba, y guarda el resultado con el prefijo `obs_` en la carpeta de destino. Si están en Bash en vez de Fish, la sintaxis del bucle cambia un poco (`for f in *.png; do ... done`), pero el comando de ImageMagick de adentro es idéntico.

Lo único que hay que tener claro antes de correrlo es en qué carpeta están parados cuando lo ejecutan, porque `*.png` toma todos los PNG de esa carpeta, y la ruta de salida hay que ajustarla a donde quieran guardar los resultados.

## Cómo usarlos en OBS

Una vez que tienen el PNG convertido (o descargan directamente los que ya dejé listos en el repo), en OBS es cuestión de clic derecho sobre la fuente de su cámara, entrar a Filtros, agregar un filtro de efecto nuevo, elegir "Apply LUT", y seleccionar el archivo PNG. El slider de "Amount" controla qué tan fuerte se aplica el efecto; yo lo dejo entre 0.5 y 0.8, porque al 100% algunos LUTs se ven demasiado marcados para video en vivo.

## Créditos

Toda la emulación de color viene de la [RawTherapee Film Simulation Collection](https://github.com) original, del 2015, hecha por Pat David, Pavlov Dmitry y Michael Ezra, bajo licencia Creative Commons Attribution-ShareAlike 4.0. Lo único que hice fue la conversión de formato para que sea compatible con OBS. Los nombres de películas que aparecen (Kodak, Fuji, Polaroid, etc.) son solo para identificar qué stock se está aproximando, no hay ninguna afiliación con esas marcas.

Si tienen curiosidad por cómo se ve cada uno antes de instalarlo, o quieren aportar sus propias conversiones, el repo está abierto: [OBS Film Simulation LUTs](https://github.com/graplo/obs-film-simulation-luts).
