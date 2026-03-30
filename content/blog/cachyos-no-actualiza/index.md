---
title: "Cachyos No Actualiza"
date: 2026-03-30T01:03:00-05:00
draft: false
summary: ""
tags: ["linux", "CLI", "open source", "problemas"]
categories: ["cachyos"]
---

# El problema es el Keyring

En cachyos-keyring es un paquete esencial en Cahyos Linux que contiene las llaves PGP firmadas por los desarrolladores y mantenedores de la distribución. Pero en este caso parecia como q no las reconocía.

### Problema en cuestión

```bash
paru -Syu
error: cachyos: la firma de «CachyOS <admin@cachyos.org>» no es válida
[sudo] contraseña para graplo:
error: cachyos: la firma de «CachyOS <admin@cachyos.org>» no es válida
:: Sincronizando las bases de datos de los paquetes...
 cachyos-v3                                    127,4 KiB   357 KiB/s 00:00 [------------------------------------------] 100%
 cachyos-core-v3 está actualizado
 cachyos-extra-v3 está actualizado
 cachyos                                       528,0 KiB  1390 KiB/s 00:00 [------------------------------------------] 100%
 core está actualizado
 extra está actualizado
 multilib está actualizado
error: cachyos-v3: la firma de «CachyOS <admin@cachyos.org>» no es válida
error: cachyos: la firma de «CachyOS <admin@cachyos.org>» no es válida
error: no se han podido sincronizar todas las bases de datos (error inesperado)
```
## Se soluciona con:

```bash
sudo pacman -Sy cachyos-keyring
```
### Lo q me aparece en pantalla

```bash
sudo pacman -Sy cachyos-keyring
error: cachyos-v3: la firma de «CachyOS <admin@cachyos.org>» no es válida
error: cachyos: la firma de «CachyOS <admin@cachyos.org>» no es válida
:: Sincronizando las bases de datos de los paquetes...
 cachyos-v3                                    127,4 KiB  1053 KiB/s 00:00 [------------------------------------------] 100%
 cachyos-core-v3 está actualizado
 cachyos-extra-v3 está actualizado
 cachyos                                       528,0 KiB  2,86 MiB/s 00:00 [------------------------------------------] 100%
 core está actualizado
 extra está actualizado
 multilib está actualizado
advertencia: cachyos-keyring-20240331-1 está actualizado -- reinstalándolo
resolviendo dependencias...
buscando conflictos entre paquetes...

Paquete (1)              Versión antigua  Versión nueva  Diferencia neta

cachyos/cachyos-keyring  20240331-1       20240331-1            0,00 MiB

Tamaño total de la instalación:  0,00 MiB
Tamaño neto tras actualizar:     0,00 MiB

:: ¿Continuar con la instalación? [S/n] s
(1/1) comprobando las claves del depósito                                  [------------------------------------------] 100%
(1/1) verificando la integridad de los paquetes                            [------------------------------------------] 100%
(1/1) cargando los archivos de los paquetes                                [------------------------------------------] 100%
(1/1) comprobando conflictos entre archivos                                [------------------------------------------] 100%
:: Ejecutando los «hooks» de preinstalación...
(1/2) Performing snapper pre snapshots for the following configurations...
==> root: 652
(2/2) Waiting for limine-snapper-sync to finish...
:: Procesando los cambios de los paquetes...
(1/1) reinstalando cachyos-keyring                                         [------------------------------------------] 100%
==> Añadiendo las claves de cachyos.gpg...
==> Actualizando la base de datos de claves de confianza...
gpg: siguiente comprobación de base de datos de confianza el: 2026-10-21
:: Ejecutando los «hooks» de posinstalación...
(1/3) Arming ConditionNeedsUpdate...
(2/3) Checking which packages need to be rebuilt
(3/3) Performing snapper post snapshots for the following configurations...
==> root: 653
```
## Entonces q ha pasado:

>Ya está claro. La reinstalación del keyring fue lo que lo solucionó — al reinstalar corrió el hook de posinstalación que dice `Añadiendo las claves de cachyos.gpg` y `Actualizando la base de datos de claves de confianza`, eso es lo que regeneró las firmas locales.
