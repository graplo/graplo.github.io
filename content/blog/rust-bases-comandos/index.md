---
title: "Cargo: instalar y gestionar programas escritos en Rust"
date: 2026-04-06T19:51:38-05:00
draft: false
description: "Cómo instalar, actualizar y gestionar programas escritos en Rust usando Cargo y crates.io."
summary: "Cargo es el gestor de paquetes de Rust. Con él podés instalar programas de la comunidad directamente desde crates.io, sin repositorios ni PPAs."
tags: ["linux", "rust", "cargo", "terminal", "cli"]
categories: ["Terminal"]
---

Si usas Linux y te gusta el software de terminal, tarde o temprano vas a topar con un programa escrito en Rust que no está en los repos de tu distro. Ahí es donde entra Cargo.

Cargo es el gestor de paquetes oficial de Rust — equivalente a `pip` en Python o `npm` en Node. Permite instalar programas directamente desde [crates.io](https://crates.io), el repositorio central de la comunidad Rust, compilándolos en tu propia máquina.

## Instalar Rust

En CachyOS y Arch lo más limpio es instalar `rustup`, que gestiona las versiones de Rust:
```bash
paru -S rustup
rustup default stable
```

`rustup` te permite cambiar entre versiones estables, beta o nightly según lo necesites. Con `stable` tienes todo lo que necesitás para el uso normal.

## Instalar un programa
```bash
cargo install nombre
```

Cargo descarga el código fuente y lo compila en tu máquina. Dependiendo del programa y tu hardware puede tardar desde segundos hasta varios minutos. Los binarios quedan en `~/.cargo/bin/`, que debería estar en tu PATH.

## Listar los programas instalados
```bash
cargo install --list
```

Muestra todos los programas instalados via cargo con sus versiones. Útil para saber qué tenés y qué versión está activa.

## Actualizar todo

Cargo no tiene un comando de actualización nativo, pero existe `cargo-update` que lo añade. Solo necesitás instalarlo una vez:
```bash
cargo install cargo-update
```

A partir de ahí, para actualizar todos los programas instalados:
```bash
cargo install-update -a
```

El `-a` es de "all" — actualiza todo lo que encuentre desactualizado.

## Desinstalar
```bash
cargo uninstall nombre
```

Elimina el binario de `~/.cargo/bin/`. No uses `rm` directamente sobre los binarios — cargo lleva un registro interno y es mejor que lo gestione él.

## Ver la versión de cargo
```bash
cargo --version
```

---

Una cosa a tener en cuenta: instalar desde cargo compila el código en tu máquina, lo que significa que necesitás las dependencias de compilación instaladas. En la mayoría de casos cargo lo gestiona solo, pero algún programa puede pedir librerías del sistema que tendrás que instalar por separado.
