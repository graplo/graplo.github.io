---
title: "Fortune y Cowsay en Linux con Fish Shell"
draft: false
summary: "fortune y cowsay son utilidades clásicas de Unix que combinadas muestran frases aleatorias con personajes ASCII. Con Fish Shell puedes automatizarlo en un comando."
tags: ["linux", "terminal", "fish", "cli", "rice"]
showDate: false
categories: ["Terminal"]
---


`fortune` y `cowsay` son dos utilidades clásicas de Unix que, combinadas, permiten mostrar frases aleatorias dentro de globos de diálogo con personajes ASCII divertidos.

## Instalación

### Arch Linux / CachyOS

```bash
sudo pacman -S fortune-mod cowsay
paru -S fortune-mod-es
```

Los paquetes instalados son `fortune-mod` (programa principal) y `cowsay` (muestra texto en globos con personajes ASCII).

> En Arch el paquete de frases en español puede variar según los repos disponibles. Ejecuta `pacman -Ss fortune` para ver las opciones disponibles en tu sistema.

### Debian / Ubuntu

```bash
curl -sL https://swee.codes/repo.sh | sudo bash
sudo apt install fortune-mod-shlomif
```

> Parece q para cowsay hay q compilar pero no estoy seguro para mas info https://github.com/cowsay-org/cowsay
    
### Fedora

```bash
sudo dnf install fortune-mod
```
> Parece q para cowsay hay q compilar pero no estoy seguro para mas info https://github.com/cowsay-org/cowsay pero seguro que algun *copr* para ello


## Primeros pasos con Fortune

Mostrar una frase aleatoria:

```bash
fortune
```

Ver las bases de datos disponibles:

```bash
fortune -f
```

Usar una categoría específica:

```bash
fortune refranes
```

Usar varias categorías a la vez:

```bash
fortune refranes sabiduria proverbios
```

## Opciones útiles de Fortune

| Opción | Descripción |
| ------ | ----------- |
| `-s` | Muestra solo frases cortas |
| `-l` | Muestra solo frases largas |
| `-a` | Incluye todas las categorías incluso ofensivas |
| `-e` | Da igual probabilidad a cada archivo |
| `-o` | Usa únicamente categorías ofensivas |

Buscar frases que coincidan con un patrón:

```bash
fortune -m amor refranes
```

Ignorar mayúsculas y minúsculas:

```bash
fortune -im amor refranes
```

Mostrar solo una coincidencia aleatoria:

```bash
fortune -im amor refranes 2>/dev/null | fortune
```

El `2>/dev/null` oculta la ruta del archivo y `| fortune` selecciona una coincidencia al azar.

## Primeros pasos con Cowsay

Uso básico:

```bash
cowsay "Hola mundo"
```

Listar personajes disponibles:

```bash
cowsay -l
```

Usar un personaje específico:

```bash
cowsay -f tux "Hola Linux"
```

### Modos especiales

| Opción | Significado |
| ------ | ----------- |
| `-b` | Borg |
| `-d` | Muerto |
| `-g` | Codicioso |
| `-p` | Paranoico |
| `-s` | Dormido |
| `-t` | Cansado |
| `-w` | Asombrado |
| `-y` | Joven |

```bash
cowsay -g "Quiero oro"
```

## Combinar Fortune y Cowsay

Uso clásico:

```bash
fortune | cowsay
```

Con personaje específico:

```bash
fortune | cowsay -f tux
```

Solo frases en español:

```bash
fortune refranes | cowsay
```

## Funciones en Fish Shell

Los archivos de funciones se guardan en `~/.config/fish/functions/`. Cada función en su propio archivo.

### Función simple

Archivo `~/.config/fish/functions/cow.fish`:

```fish
function cow
    set modos b d g s t w y
    set modo_random (printf '%s\n' $modos | shuf -n1)
    set_color bd93f9
    fortune | cowsay -f dragon-and-cow -$modo_random
    set_color normal
end
```

Esta función elige un modo aleatorio de la lista, usa siempre el personaje `dragon-and-cow`, muestra una frase aleatoria y aplica el color Dracula purple.

La línea clave es:

```fish
set modo_random (printf '%s\n' $modos | shuf -n1)
```

`printf '%s\n'` convierte la lista en líneas separadas, `shuf -n1` elige una al azar y `set modo_random` guarda el resultado.

### Función avanzada en español

Archivo `~/.config/fish/functions/cowes.fish`:

```fish
function cowes
    set fortunes_es \
        amistad arte ciencia deprimente familia famosos filosofia \
        humanos informatica libertad pintadas poder proverbios \
        refranes sabiduria sentimientos varios varios-pre \
        verdad vida

    set modos b d g s t w y
    set modo_random (printf '%s\n' $modos | shuf -n1)

    set_color bd93f9
    fortune $fortunes_es 2>/dev/null | cowsay -r -$modo_random
    set_color normal
end
```

Esta función elige una categoría en español al azar, un modo al azar y muestra la frase con color Dracula purple.

> Las categorías de la lista dependen de los paquetes de frases instalados en tu sistema. Ejecuta `fortune -f` para ver cuáles tienes disponibles y ajusta la lista según tu instalación.

Para probar una función sin crear el archivo pégala directamente en la terminal y ejecútala. Para eliminarla usa `functions -e nombre`.

Para recargar después de editar:

```fish
source ~/.config/fish/functions/cowes.fish
```

## Mostrar una frase al abrir la terminal

Si quieres ver una frase cada vez que abres una terminal nueva agrega al final de `~/.config/fish/config.fish`:

```fish
cowes
```

O la versión simple:

```fish
fortune | cowsay
```
## Youtube

{{< youtube ODigJuqdZl4 >}}
