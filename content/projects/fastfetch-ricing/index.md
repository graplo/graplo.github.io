---
title: "Fastfetch Ricing"
draft: false
summary: "La lógica completa detrás de un fastfetch personalizado: constants, placeholders, cajas con Nerd Font y módulos del sistema."
tags: ["linux", "fastfetch", "terminal", "dracula"]
showDate: false
categories: ["linux"]
---

Cuando abro la terminal me sale mi logo en ASCII con un degradado de cian a morado, y al lado toda la info de mi sistema organizada en cajas, también con su propio degradado. Eso lo hace fastfetch, un programa que lee un archivo de configuración y dibuja todo eso automáticamente cada vez que abres una terminal nueva.

La parte de convertir una imagen en arte ASCII es la única que no voy a explicar acá, porque honestamente esa parte se la dejé a Claude. Si quieren su propio logo en ASCII, esa es la recomendación: pídanselo a una IA, denle la imagen, y listo. Lo interesante, lo que sí vale la pena entender, es todo lo demás: cómo se arma el degradado, cómo se dibujan esas cajas con líneas, y cómo cada módulo le pide información al sistema.

## Lo primero: dónde viven los colores

Fastfetch usa un archivo de configuración, normalmente `~/.config/fastfetch/config.jsonc`. Es formato JSON, que es básicamente una forma de organizar datos con llaves `{}`, corchetes `[]`, y pares de "nombre": "valor".

Dentro de ese archivo hay una sección llamada `constants`, que es donde defines tu paleta de colores una sola vez:

```json
"constants": [
    "\u001b[38;2;139;233;253m",   // {$1}
    "\u001b[38;2;156;204;252m",   // {$4}
    "\u001b[38;2;189;147;249m",   // {$10}
    "┌──────",                    // {$11}
    "───────",                    // {$12}
    "──────┐"                     // {$13}
]
```

Ese código raro que empieza con `\u001b[38;2;` es la forma en que la terminal entiende "pinta lo que sigue de este color RGB específico". No hace falta memorizarlo, simplemente se reemplazan los tres números del medio (en este ejemplo 139, 233, 253) por el Rojo, Verde y Azul del color que quieras, y ese código va a pintar todo lo que venga después de él hasta que aparezca otro código de color.

Cada elemento de esa lista tiene una posición. El primero es la posición 1, el segundo la posición 2, y así. Fastfetch te deja "llamar" a cada uno usando `{$1}`, `{$2}`, `{$3}`... el número entre llaves y signo de dólar es simplemente "usa el color o texto que guardé en esa posición de la lista".

Fíjense que también metí ahí guiones (`┌──────`, `───────`, `──────┐`) en las posiciones 11, 12 y 13. Esto no es color, es texto literal, pero funciona exactamente igual: lo guardas una vez, y lo llamas con `{$11}` cuando lo necesitas. Esa es la base de todo lo que viene.

## El degradado: varios colores en vez de uno

Si solo pones un color en `constants`, todo lo que pintes con `{$1}` va a ser ese mismo color sólido. El truco del degradado es simplemente tener varios colores guardados (en mi caso, diez), donde cada uno es un paso intermedio entre mi cian y mi morado.

No los inventé a ojo, los calculé matemáticamente repartiendo la diferencia entre ambos colores en partes iguales. Si te interesa cómo se calcula eso exactamente, dale al final del post.

Con los diez colores ya guardados en `constants`, lo único que queda es decidir en qué orden los uso. Y ahí está la parte que de verdad vale la pena entender.

## Las cajas: cómo se dibuja eso de "┌──────┐"

Esas líneas con esquinas que separan "Hardware" de "Software" en mi fastfetch no son una imagen ni un dibujo especial, son simplemente caracteres Unicode que parecen líneas: `┌`, `─`, `┐`, `├`, `└`. Existen en cualquier fuente que tenga soporte de "box drawing characters", y como uso una Nerd Font (una tipografía que además incluye iconitos extra de programas, sistemas operativos, etc.), también puedo meter íconos como 󰣇 o 󰊬 directamente en el texto, igual que si fueran letras normales.

Mira cómo armo la línea de título de cada caja:

```json
{
    "type": "custom",
    "format": "{$1}{$11}{$2}{$12}{$3}{$12}{$4}{$12}{$5}{$6}{$12}{$7}{$12}{$8}{$12}{$9}{$12}{$10}{$13} Hardware"
}
```

Leyendo esto de izquierda a derecha: pinta con el color 1, dibuja la esquina izquierda (`{$11}` = `┌──────`), cambia al color 2, dibuja un tramo de relleno (`{$12}` = `───────`), cambia al color 3, otro tramo de relleno, y así sucesivamente, hasta terminar con la esquina derecha (`{$13}` = `──────┐`) y la palabra "Hardware" al final.

El resultado visual es una línea horizontal que va cambiando de color a medida que avanza, porque cada tramo de guiones usa un color distinto de los diez que tengo guardados. Eso es literalmente todo el secreto detrás del "degradado" de las cajas, no hay nada más sofisticado: es repetir guiones con colores distintos, uno detrás de otro.

El de al final de cada caja con esquina (`└`), es el mismo concepto pero usado para cerrar una lista de elementos. Mira un módulo individual:

```json
{ "type": "CPU", "key": "{$3}├󰻠 CPU" },
{ "type": "Memory", "key": "{$4}├󰑭 RAM" },
{ "type": "Disk", "key": "{$5}└󰋊 Disk" }
```

`├` (con la rama hacia la derecha) se usa para todos los elementos de en medio, y `└` (con la esquina hacia abajo) se usa solo en el último elemento de la lista, para "cerrar" visualmente esa sección. Es pura convención visual, nada obliga a usarlo así, pero es lo que le da esa sensación de árbol o de lista conectada.

## Los módulos: cómo fastfetch sabe tu RAM, tu CPU, etc

Cada bloque como este:

```json
{ "type": "CPU", "key": "{$3}├ 󰻠 CPU" }
```

Tiene dos partes. El `"type": "CPU"` le dice a fastfetch qué información buscar, fastfetch ya sabe internamente cómo consultar el procesador de tu máquina, no hay que programar nada de eso, simplemente existe una lista de tipos que fastfetch reconoce: `CPU`, `Memory`, `Disk`, `GPU`, `OS`, `Kernel`, `Shell`, `Wifi`, `Uptime`, y muchos más.

El `"key"` es solo la etiqueta visual, lo que se ve a la izquierda antes del valor real. Ahí es donde metes tu color con `{$3}`, tu carácter de árbol `├` o `└`, el ícono de Nerd Font, y el texto. El valor real (cuántos GB de RAM tienes, qué modelo de CPU, etc.) lo calcula fastfetch solo y lo pone automáticamente al lado.

Entonces armar una sección nueva es simplemente: elegir qué tipos de módulo quieres mostrar, ponerlos en orden dentro de la lista `modules`, y decorar cada uno con su color y su ícono.

## Donde está la magia

Todo lo que ven en mi fastfetch se reduce a tres ideas: una lista de colores guardados una vez y reutilizados muchas veces, caracteres de texto que dibujan líneas en vez de imágenes, y módulos que ya saben consultar tu sistema solos. No hay nada ahí que requiera saber programar de verdad, es entender la lógica de "guardo algo con un nombre corto, y después lo llamo cuando lo necesito".

La parte del ASCII art del logo sí es distinta, ahí sí se necesita un script que analice una imagen píxel por píxel, y esa parte se la dejo a Claude sin remordimiento. Pero el archivo de configuración completo, con degradado, cajas e información del sistema, eso lo armé yo entendiendo cada pieza, y por eso creo que cualquiera que use Linux con algo de paciencia puede hacerlo también.

## El `json` completo en mi caso

```bash
{
    "logo": {
        "type": "file",
        "source": "/home/graplo/.config/fastfetch/graplo-ansii3.txt",
        "color": {
            "1": "#bd93f9",
            "2": "#b79efa",
            "3": "#b0a8fa",
            "4": "#aab3fa",
            "5": "#a4befb",
            "6": "#9ec9fc",
            "7": "#98d4fc",
            "8": "#91defc",
            "9": "#8be9fd"
        }
    },
    "display": {
        "separator": " ",
"constants": [
    "\u001b[38;2;139;233;253m\u001B[1m",   // {$1} = #8be9fd (cian Dracula)
    "\u001b[38;2;146;215;253m\u001B[1m",   // {$2} = #92d7fd
    "\u001b[38;2;154;196;253m\u001B[1m",   // {$3} = #9ac4fd
    "\u001b[38;2;162;178;253m\u001B[1m",   // {$4} = #a2b2fd
    "\u001b[38;2;169;160;253m\u001B[1m",   // {$5} = #a9a0fd (morado Dracula)
    "\u001b[38;2;179;154;253m\u001B[1m",   // {$6} = #b39afd
    "\u001b[38;2;186;153;253m\u001B[1m",   // {$7} = #ba99fd
    "\u001b[38;2;192;163;253m\u001B[1m",   // {$8} = #c0a3fd
    "\u001b[38;2;198;167;253m\u001B[1m",   // {$9} = #c6a7fd
    "\u001b[38;2;205;173;252m\u001B[1m",   // {$10} = #cdadfc
    "┌──────",                              // {$11}
    "───────",                              // {$12}
    "──────┐"                               // {$13}
]
    },
    "modules": [
        "break",
        {
            "type": "custom",
            "format": "{$1}{$11}{$2}{$12}{$3}{$12}{$4}{$12}{$5}{$6}{$12}{$7}{$12}{$8}{$12}{$9}{$12}{$10}{$13} Hardware"
        },
        { "type": "Chassis", "key": "{$4}├󰌢 PC        " },
        { "type": "Board", "key": "{$5}├ Board     " },
        { "type": "CPU", "key": "{$6}├󰻠 CPU       ", "temp": true },
        { "type": "Memory", "key": "{$1}├󰑭 RAM       " },
        { "type": "Swap", "key": "{$2}├󰓡 Zram      " },
        { "type": "Disk", "key": "{$3}├󰋊 Disk      " },
        { "type": "GPU", "key": "{$4}├󰍛 GPU       " },
        { "type": "OpenGL", "key": "{$9}├󰍛 OpenGL    " },
        { "type": "Vulkan", "key": "{$8}├󰍛 Vulkan    " },
        { "type": "Display", "key": "{$7}└󰍹 Display(s)" },
                {
            "type": "custom",
            "format": "{$10}{$11}{$9}{$12}{$8}{$12}{$7}{$12}{$6}{$5}{$12}{$4}{$12}{$3}{$12}{$2}{$12}{$1}{$13} Software"
        },
        { "type": "OS", "key": "{$4}├󰣇 OS        " },
        { "type": "Kernel", "key": "{$5}├ Kernel    " },
        { "type": "Packages", "key": "{$6}├󰏖 Packages  ",  },
        { "type": "Shell", "key": "{$7}├ Shell     " },
        { "type": "DE", "key": "{$8}├󰊬 DE        " },
        { "type": "WM", "key": "{$9}├󰧨 WM        " },
        { "type": "LM", "key": "{$10}├󰧨 LM        " },
        { "type": "WMTheme", "key": "{$1}├󰉼 WM Theme  " },
        { "type": "Theme", "key": "{$2}├󰉼 Theme     " },
        { "type": "Icons", "key": "{$3}├󰸉 Icons     " },
        { "type": "Font", "key": "{$4}├ Font      " },
        { "type": "Terminal", "key": "{$5}└ Terminal  " },
                {
            "type": "custom",
            "format": "{$1}{$11}{$2}{$12}{$3}{$12}{$4}{$12}{$5}{$6}{$12}{$7}{$12}{$8}{$12}{$9}{$12}{$10}{$13} Connect"
        },
        { "type": "LocalIP", "key": "{$6}├󰩟 Local IP  " },
        { "type": "Wifi", "key": "{$7}└󰤨 Wifi      " },
                {
            "type": "custom",
            "format": "{$10}{$11}{$9}{$12}{$8}{$12}{$7}{$12}{$6}{$5}{$12}{$4}{$12}{$3}{$12}{$2}{$12}{$1}{$13} Time"
        },
        { "type": "Uptime", "key": "{$10}├󰅐 Uptime    " },
        { "type": "DateTime", "key": "{$8}└󰃭 Date      " },
        "break",
        {
          "type": "custom",
          "format": "\u001b[38;2;241;250;140m󰮯 \u001b[38;2;189;147;249m󰊠 \u001b[38;2;139;233;253m󰊠 \u001b[38;2;255;121;198m󰊠 \u001b[38;2;255;184;108m󰊠 \u001b[38;2;80;250;123m󰊠 \u001b[38;2;98;114;164m󱙝"
        }
    ]
}
```
## Mi logo en ascii q es solo un .txt

```txt
$1             .-+#%%%%#*+-:              
$1          .=#%@@@%%%%%@@@%%#+=-.        
$1        -*%@@%%%%%%%%%%%%@@@@%%*-       
$2      =#@@%%%%%%%%%%%%@@%#*=:.          
$2    :#@@%%%%%%%%%%%%@%#=:               
$2   +%@%%%%%%%%%%%%@%+.                  
$3  *@%%%%%%%%%%%%@%=.                 .:-
$3 +@%%%%%%%%%%%%@#.               .-+#%@%
$3:%%%%%%%%%%%%%@*             .-+#%@@@%%%
$4*@%%%%%%%%%%%@*          .-+#%@@@%%%%%%%
$4%%%%%%%%%%%%%%:          %@@%@%%%%%%%%%%
$4%%%%%%%%%%%%@#           ==..=%%%%%%%%%%
$5#%%%%%%%%%%%@#               .%%%%%%%%%%
$5+@%%%%%%%%%%%%:              :%%%%%%%%%%
$6.#@%%%%%%%%%%@#             .#%%%%%%%%%%
$6 :%@%%%%%%%%%%@#:          =%@%%%%%%%%%%
$6  -%@%%%%%%%%%%@%*-.  .:-*%@%@@%%%%%%%%%
$7   .#@@%%%%%%%%%%@@@%%#*+-:..-*%%%%%%%%%
$7     -#@@%%%%%%@@@%#+:.        %%%%%%%%%
$7       -*%@@@@%%*-.            %%%%%%@@%
$8         .-==-:   -+*#*      .*@%@@@%#=:
$8                -#@@@@*    :=%@@%#+-.   
$8               .%@%%%%%#*#%@@%*=:       
$9                *@@%%%%@@%#+:           
$9                 -*#%%%*=.
```
