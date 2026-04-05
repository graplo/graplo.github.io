---
title: "Jellyfin Puerto Bloqueado Cachyos"
date: 2026-04-04T20:53:51-05:00
draft: false
summary: "Jellyfin funciona pero no es accesible desde la red local. El problema no es Jellyfin, es el firewall."
tags: ["linux", "jellyfin", "firewall", "red", "gui"]
categories: ["Linux", "Homelab"]
---


Si instalaste Jellyfin en CachyOS y funciona bien desde tu computadora 
pero no podés acceder desde el celular o la TV, el problema probablemente 
no es Jellyfin — es el firewall.

## Verificar que Jellyfin está corriendo

Primero confirmamos que Jellyfin realmente está escuchando en la red:

```bash
ss -tulpn
```

Si ves algo así en la salida:
0.0.0.0:8096 users:(("jellyfin"...))

Jellyfin está activo y escuchando en todas las interfaces de red. 
El problema está en otro lado.

## El culpable: el firewall

CachyOS usa nftables como firewall, y por defecto tiene una política 
bastante estricta. Podés verlo con:

```bash
sudo nft list ruleset
```

Vas a encontrar algo como:
chain input {
type filter hook input priority 0;
policy drop;
}

Ese `policy drop` significa que todo el tráfico entrante que no tenga 
una regla explícita que lo permita se bloquea automáticamente. 
El flujo era este:

Celular / TV
↓
Puerto 8096
↓
Firewall (nftables)
↓
policy drop → bloqueado

Jellyfin nunca recibía la conexión.

## La solución

Usamos `ufw`, una herramienta que simplifica la gestión del firewall, 
para abrir el puerto 8096 que usa Jellyfin:

```bash
sudo ufw allow 8096/tcp
```

Para verificar que el cambio quedó activo:

```bash
sudo ufw status
```

## Probar desde otro dispositivo

Desde el celular o la TV abrí el navegador y entrá a:
http://192.168.X.X:8096

Reemplazá la IP por la de tu computadora. Un detalle importante: 
Jellyfin usa HTTP por defecto, no HTTPS. Asegurate de no poner `https://` 
o no va a conectar.
