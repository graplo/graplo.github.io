---
title: "Impala Wifi"
date: 2026-06-27T22:59:53-05:00
draft: true
summary: ""
description: ""
tags: []
categories: []
---
## Introducción

En Linux existen varias formas de gestionar una conexión WiFi.

La configuración tradicional suele ser:

NetworkManager
|
└── wpa_supplicant

Otra alternativa moderna es usar iwd:

NetworkManager
|
└── iwd


iwd (iNet Wireless Daemon) fue desarrollado originalmente por Intel, pero funciona con tarjetas WiFi de diferentes fabricantes porque utiliza las interfaces estándar del kernel Linux.

Una vez configurado, herramientas como `iwctl` o `impala` pueden controlar la conexión WiFi usando iwd.

## Ver qué tarjeta WiFi tengo

Primero hay que saber cómo se llama nuestra interfaz WiFi.

El comando más simple:

```bash
ip link
```
En mi caso:

```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp9s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 10:7c:61:d5:22:ad brd ff:ff:ff:ff:ff:ff
    altname enx107c61d522ad
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc htb state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:2e:59:98 brd ff:ff:ff:ff:ff:ff
5: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DORMANT group default qlen 1000
    link/ether a8:41:f4:52:59:b3 brd ff:ff:ff:ff:ff:ff
```

### Ver servicios WiFi instalados

```bash
pacman -Qs iwd
pacman -Qs wpa
```

### Ver el estado de los servicios:

```bash
systemctl status iwd
systemctl status wpa_supplicant
systemctl status NetworkManager
```

## Ver qué backend usa NetworkManager

NetworkManager puede usar diferentes motores WiFi.

Comprobar configuración:

## Activar iwd como backend de NetworkManager

Editar el archivo:

```bash
sudo helix /etc/NetworkManager/NetworkManager.conf
```
Añadir:

```ini
[device]
wifi.backend=iwd
```
si lo quieres activar con el inicio de la pc (recomendado) pero consume 2.2 mb

```bash
sudo systemctl enable iwd
```

Guardar y reiniciar NetworkManager:

```bash
sudo systemctl restart NetworkManager
```
## Usar iwd desde la terminal

La herramienta oficial de iwd es iwctl.

Abrir:

```bash
iwctl
```

Dentro de la consola:

### Comandos útiles dentro de iwctl

| Acción | Comando |
|--------|--------|
| Ver dispositivos | `device list` |
| Escanear redes WiFi | `station wlan0 scan` |
| Ver redes disponibles | `station wlan0 get-networks` |
| Conectarse a una red | `station wlan0 connect NombreWifi` |
| Ejemplo de conexión | `station wlan0 connect Graplo` |
| Ver estado de conexión | `station wlan0 show` |
| Ejemplo de estado | `Connected network: Graplo` |
| Ver redes guardadas | `known-networks list` |

