---
title: "Impala Wifi: Gestionando redes inalámbricas con iwd en Linux"
date: 2026-06-27T22:59:53-05:00
draft: false
summary: "Configuración de iwd como backend WiFi de NetworkManager y uso de Impala como interfaz TUI para administrar conexiones inalámbricas desde la terminal."
description: "Guía para instalar y configurar iwd en Linux, reemplazar wpa_supplicant y utilizar Impala como una interfaz basada en teclado para gestionar redes WiFi."
tags: ["linux", "wifi", "iwd", "networkmanager", "Impala", "terminal", "tui"]
categories: ["Linux", "Redes"]
---
## Introducción

En Linux existen varios componentes para gestionar una conexión WiFi.

En la mayoría de distribuciones, NetworkManager utiliza **wpa_supplicant** como backend para controlar la interfaz inalámbrica.

```ini
NetworkManager
|
└── wpa_supplicant
```
Una alternativa moderna es utilizar iwd (iNet Wireless Daemon), que puede reemplazar a wpa_supplicant como backend.

```ini
NetworkManager
|
└── iwd
```

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
En mi equipo consume aproximadamente 2.2 MiB de memoria, aunque el valor puede variar según la distribución y la versión instalada.

```bash
sudo systemctl enable iwd
```

Guardar y reiniciar NetworkManager:

```bash
sudo systemctl restart NetworkManager
```
## Desactivar wpa_supplicant

Si anteriormente NetworkManager utilizaba `wpa_supplicant`, es recomendable detenerlo y deshabilitarlo antes de cambiar a `iwd`.

Primero comprobamos su estado:

```bash
systemctl status wpa_supplicant
```

Si todo está correcto, debería aparecer algo similar a:

```text
○ wpa_supplicant.service - WPA supplicant
     Loaded: loaded (/usr/lib/systemd/system/wpa_supplicant.service; disabled; preset: disabled)
     Active: inactive (dead)
```

Esto indica que el servicio está instalado, pero no se está ejecutando.

En caso de que estuviera activo, podemos detenerlo:

```bash
sudo systemctl stop wpa_supplicant
```

Y evitar que se inicie automáticamente:

```bash
sudo systemctl disable wpa_supplicant
```

También es posible que exista una instancia asociada directamente a la interfaz WiFi:

```bash
sudo systemctl stop wpa_supplicant@wlan0
sudo systemctl disable wpa_supplicant@wlan0
```

Después verificamos que `iwd` esté funcionando:

```bash
systemctl status iwd
```

El servicio debe aparecer como:

```text
Active: active (running)
```

Finalmente reiniciamos NetworkManager para aplicar el nuevo backend:

```bash
sudo systemctl restart NetworkManager
```

La arquitectura final queda así:

```text
NetworkManager
      |
      └── iwd
             |
             └── tarjeta WiFi


Impala / iwctl
      |
      └── D-Bus
             |
             └── iwd
```

De esta forma, `iwd` se encarga de la gestión inalámbrica y herramientas como `iwctl` o `impala` pueden comunicarse con él mediante D-Bus.

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

## ¿Qué es Impala?

Impala es una aplicación TUI (Terminal User Interface) para administrar conexiones WiFi mediante **iwd**.

A diferencia de `iwctl`, que funciona como una consola interactiva donde los comandos deben escribirse manualmente, Impala presenta una interfaz visual que facilita la administración de redes inalámbricas directamente desde la terminal.

Internamente utiliza la API de **iwd** a través de D-Bus, por lo que no reemplaza a iwd, sino que actúa como un cliente gráfico para la terminal.

Gracias a esto, resulta especialmente útil en gestores de ventanas (Window Managers) o entornos minimalistas donde no se dispone de un applet gráfico de NetworkManager.

### Instalación

En Arch Linux y distribuciones derivadas como CachyOS, Impala se encuentra en los repositorios oficiales, por lo que puede instalarse con:

```bash
sudo pacman -S impala
```

Una vez instalado, basta con ejecutar:

```bash
impala
```

Si `iwd` está correctamente configurado y NetworkManager utiliza este backend, Impala detectará automáticamente la interfaz inalámbrica y mostrará las redes WiFi disponibles.

---

### Funciones

Entre las principales características de Impala se encuentran:

- Escaneo automático de redes WiFi disponibles.
- Conexión a redes abiertas y protegidas.
- Solicitud de contraseña cuando es necesaria.
- Visualización de la intensidad de la señal.
- Identificación del tipo de seguridad de cada red.
- Desconexión de la red actual.
- Eliminación de redes guardadas (Known Networks).
- Actualización de la lista de redes sin salir de la aplicación.
- Interfaz completamente basada en teclado.
- Integración directa con iwd mediante D-Bus.

---

### Atajos de teclado

Impala está pensado para utilizarse completamente con el teclado.

| Tecla | Acción |
|--------|--------|
| ↑ / ↓ | Moverse entre las redes disponibles |
| Enter | Conectarse a la red seleccionada |
| Tab | Cambiar entre paneles |
| r | Volver a escanear redes |
| d | Desconectarse de la red actual |
| Delete | Eliminar una red guardada |
| q | Salir de Impala |
| Esc | Cerrar cuadros de diálogo o volver atrás |

---

### Impala frente a iwctl

| iwctl | Impala |
|--------|---------|
| Basado en comandos | Interfaz TUI |
| Hay que recordar la sintaxis | Navegación mediante teclado |
| Ideal para scripting | Ideal para el uso diario |
| Incluido con iwd | Aplicación independiente |
