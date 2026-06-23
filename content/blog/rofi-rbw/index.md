---
title: "rbw y rofi-rbw: Bitwarden en una caja"
date: 2026-06-22T20:27:26-05:00
draft: true
summary: "Configuración completa de rbw y rofi-rbw para acceder a Bitwarden con un atajo de teclado, sin abrir el navegador."
description: "Cómo instalar rbw como backend de Bitwarden y rofi-rbw como interfaz visual, incluyendo el error 400 al hacer login y por qué hay que registrar el dispositivo primero."
tags: ["linux", "rofi", "bitwarden", "niri", "rofi-rbw"]
categories: ["linux", "Niri"]
---

Quería poder llamar a mis contraseñas de Bitwarden con un atajo de teclado, igual que llamo a mis apps con drun o mi portapapeles con cliphist. Eso significa no depender de abrir el navegador o la app de escritorio de Bitwarden cada vez, sino tener un cuadro de rofi que copie usuario y contraseña directo, listo para pegar en cualquier programa, como Steam.

Para esto se necesitan dos piezas separadas: un backend que hable con Bitwarden (`rbw`), y una interfaz visual encima de ese backend (`rofi-rbw`). Son dos paquetes distintos, y el orden en que los configuras importa.

## Instalar rbw

`rbw` es un cliente no oficial de Bitwarden hecho específicamente para integrarse con herramientas como rofi. Se instala desde Pacman, y te instala ya como dependencia `rbw`:

```bash
sudo pacman -S rofi-rbw
```

Lo primero es decirle tu correo:

```bash
rbw config set email tu-correo@ejemplo.com
```

Si usas el Bitwarden oficial (bitwarden.com), no necesitas tocar nada más en la configuración. Si tuvieras un servidor self-hosted o Vaultwarden, ahí sí necesitarías además `rbw config set base_url https://tu-servidor.com`, pero no es mi caso.

## El error 400 al hacer login

Aquí es donde me trabé un buen rato. Después de configurar el correo, lo lógico parece ser correr directamente:

```bash
rbw login
```

Esto abre una ventana de pinentry pidiendo tu master password. El problema es que, aunque la contraseña sea correcta, el servidor de Bitwarden responde con un error genérico:

```
rbw login: failed to log in to bitwarden instance: api request returned error: 400
```

Pasé un rato sospechando de todo: 2FA, KDF, passkeys, el keyring de GNOME. Nada de eso era la causa. La razón real es que **el servidor oficial de Bitwarden trata el tráfico de línea de comandos como tráfico sospechoso**, y por eso pide registrar el dispositivo antes de poder loguearte con él.

El paso que faltaba es este, antes de `rbw login`:

```bash
rbw register
```

Este comando pide dos datos que no son tu master password: el `client_id` y el `client_secret`. Esos los sacas de tu cuenta de Bitwarden en el navegador, en **Configuración → Seguridad → Claves → Ver clave API**. Son credenciales separadas, pensadas justo para autenticar aplicaciones de línea de comandos.

Una vez que `rbw register` termina sin error, ahí sí `rbw login` funciona con tu master password normal. El orden correcto entonces es: configurar el email, registrar el dispositivo con la clave API, y solo después hacer login.

## Dónde queda guardado

Por curiosidad de seguridad, antes de meterme con dotfiles, revisé qué se guarda en texto plano. El archivo de configuración vive en `~/.config/rbw/config.json`, y solo contiene el correo y las URLs del servidor, nada sensible. El `client_id` y `client_secret` que metí durante el registro no quedaron ahí, se guardaron en el keyring de GNOME, así que están cifrados con la contraseña de sesión del sistema, no en un archivo legible.

## Correr rofi-rbw

Pero al correrlo por primera vez salió este error:

```
rofi_rbw.typer.typer.NoTyperFoundException: Could not find a valid way to type characters.
```

Esto pasa porque `rofi-rbw` necesita un "typer", el programa que usa para escribir texto directamente en otra ventana (la función de autotype). Como uso niri, que es Wayland puro, el que corresponde es `wtype`:

```bash
sudo pacman -S wtype
```

`wl-clipboard` también es necesario para copiar al portapapeles, pero ya lo tenía instalado de antes.

## El archivo de configuración no típico

Esta parte me sorprendió. Casi todos mis programas guardan su configuración en una carpeta propia dentro de `~/.config/nombre-programa/`. `rofi-rbw` no sigue esa convención, su archivo va directo en:

```
~/.config/rofi-rbw.rc
```

Sin carpeta intermedia. Dentro, las opciones se escriben con el nombre largo sin guiones dobles, una por línea:

```
action = copy
selector = rofi
```

La opción `action` es la que decide qué pasa al seleccionar una entrada. Por defecto viene en modo autotype, que escribe usuario y contraseña directamente en la ventana donde tengas el foco, como si fuera un teclado tecleando rápido. Para mi caso, donde quiero copiar y pegar manualmente entre programas como Steam, `copy` es mucho más predecible: copia al portapapeles en vez de escribir a ciegas donde sea que esté el cursor.

## Theme propio y atajo de teclado

Como con cualquier programa que invoque rofi, se le puede pasar un theme distinto al de tu launcher principal usando `--selector-args`. Ojo con la sintaxis exacta, porque tiene un bug conocido de argparse y hay que escribirlo con el signo igual pegado:

```bash
rofi-rbw --selector-args="-theme ~/.config/rofi/themes/rbw.rasi"
```

Metí esto en un script en `~/.local/bin/rofi-bw`, y le puse un atajo en niri:

```kdl
Mod+Alt+K hotkey-overlay-title="Bitwarden" { spawn "sh" "-c" "~/.local/bin/rofi-bw"; }
```

Con eso ya tengo mi vault completo accesible con un atajo, copiando al portapapeles, sin tener que abrir el navegador para algo tan simple como pegar una contraseña en Steam.

