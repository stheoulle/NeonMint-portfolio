---
layout: /src/layouts/MarkdownPostLayout.astro
title: Proxyfying a VM
author: Sarah THEOULLE
description: "Learn how to set up a proxy server with HAProxy and Squid."
image:
  url: "/images/posts/VM.png"
  alt: "Screenshot of NeonMint, a modern and minimalist web template built with Astro and TailwindCSS. It displays various sections such as blog, portfolio, work experience, and Markdown guide, in both light and dark modes with mint green accents."
pubDate: 2025-04-04
tags: ["documentation", "VM", "proxy", "haproxy", "squid"]
languages: ["bash", "markdown", "yaml", "html"]
---

Deployer un HA proxy avec deux backends (on change de backend au F5)

Un squid en local avec le téléphone branché sur le laptop avec un wireshark

---

## 📁 Structure du projet

Crée un dossier (ex: `proxy-lab/`) avec ces fichiers :

```bash
proxy-lab/
├── docker-compose.yml
├── haproxy.cfg
├── squid.conf
└── backend1/index.html
└── backend2/index.html

```

---

## 📦 1. `docker-compose.yml`

```yaml
version: '3.8'

services:
  backend1:
    image: nginx:alpine
    volumes:
      - ./backend1:/usr/share/nginx/html:ro
    networks:
      - proxy_net
    ports:
      - "9001:80"

  backend2:
    image: nginx:alpine
    volumes:
      - ./backend2:/usr/share/nginx/html:ro
    networks:
      - proxy_net
    ports:
      - "9002:80"

  haproxy:
    image: haproxy:latest
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro
    networks:
      - proxy_net
    ports:
      - "8080:8080"

  squid:
    image: sameersbn/squid:3.5.27-2
    volumes:
      - ./squid.conf:/etc/squid/squid.conf:ro
    ports:
      - "3128:3128"
    networks:
      - proxy_net

networks:
  proxy_net:
    driver: bridge

```

---

## 🧰 2. `haproxy.cfg`

```bash
global
    daemon
    maxconn 256
    log stdout format raw

defaults
    mode http
    timeout connect 5s
    timeout client  50s
    timeout server  50s

frontend http-in
    bind *:8080
    default_backend web_servers

backend web_servers
    balance roundrobin
    option httpchk
    server web1 backend1:80 check
    server web2 backend2:80 check

```

---

## 🧰 3. `squid.conf`

```bash
http_port 3128

acl localnet src 0.0.0.0/0
http_access allow localnet
http_access allow localhost
http_access deny all

cache deny all
forwarded_for off
via off

# Rediriger tout vers haproxy
cache_peer haproxy parent 8080 0 no-query originserver
never_direct allow all

```

---

## 🖼️ 4. Fichiers HTML des backends

### `backend1/index.html`

```html
<h1>Hello from Backend 1</h1>

```

### `backend2/index.html`

```html
<h1>Hello from Backend 2</h1>

```

---

## 🚀 Lancer le tout

Dans le dossier `proxy-lab` :

```bash
docker-compose up --build

```

---

## ✅ Tester

1. Configure ton téléphone avec un **proxy HTTP** :
    - **IP** : IP de ton laptop
    - **Port** : `3128`
2. Ouvre un navigateur sur le téléphone → tu dois voir alternativement les deux backends.
3. Lance **Wireshark** sur ton interface réseau locale pour capturer le trafic.

---

![image.png](attachment:1bdc034a-ac6c-4df3-b5d3-6e0c0a7213e3:image.png)

## Tunnel IP IP avec VPN

Voici un tutoriel complet pour créer un **tunnel IP-IP** (protocole 4) entre deux machines virtuelles (VM) dans un même VPS, formant un VPN simple de type **IPIP**.

---

## 🧠 **Prérequis**

- 2 VM (appelons-les `VM1` et `VM2`), toutes deux avec un système GNU/Linux (ex : Debian, Ubuntu).
- Droits `root` ou sudo sur les deux.
- Le **module `ipip`** du noyau doit être disponible (souvent inclus par défaut).

---

## 🌐 **Étape 1 : Informations réseau**

|  | VM1 | VM2 |
| --- | --- | --- |
| IP ext | 192.168.122.242 | 192.168.122.163 |
| IP tun | `10.0.0.1` | `10.0.0.2` |

> ⚠️ Remplace ces adresses IP selon ta configuration.
>

---

## 🧩 **Étape 2 : Charger le module IPIP**

Sur les **deux VM** :

```bash
sudo modprobe ipip

```

Pour le charger automatiquement au boot, ajoute `ipip` dans `/etc/modules-load.d/ipip.conf` :

```bash
echo ipip | sudo tee /etc/modules-load.d/ipip.conf

```

---

## 🔧 **Étape 3 : Créer l’interface tunnel IPIP**

### Sur **VM1**

```bash
sudo ip tunnel add tun0 mode ipip remote 192.168.122.163 local 192.168.122.242 ttl 255
sudo ip link set tun0 up
sudo ip addr add 10.0.0.1/30 dev tun0

```

### Sur **VM2**

```bash
sudo ip tunnel add tun0 mode ipip remote 192.168.122.242 local 192.168.122.163 ttl 255
sudo ip link set tun0 up
sudo ip addr add 10.0.0.2/30 dev tun0

```

---

## 🧪 **Étape 4 : Tester la connectivité**

Depuis **VM1** :

```bash
ping 10.0.0.2

```

Depuis **VM2** :

```bash
ping 10.0.0.1

```

---

## 🛣️ **Étape 5 : Ajouter des routes si nécessaire**

Tu peux acheminer du trafic à travers ce tunnel :

Par exemple, si tu veux que **VM1** puisse accéder à un réseau `192.168.10.0/24` derrière VM2 :

```bash
sudo ip route add 192.168.10.0/24 via 10.0.0.2 dev tun0

```

Et l’inverse sur VM2 si nécessaire.

---

## 🔐 **Sécurité (Optionnel mais recommandé)**

Le tunnel IPIP **n’est pas chiffré**. Tu peux :

- Le **restreindre via iptables** (firewall) pour n'autoriser que le trafic IPIP :

```bash
sudo iptables -A INPUT -p 4 -s 192.168.0.2 -j ACCEPT

```

- Ou encapsuler le tout dans une **connexion IPSec** si besoin de chiffrement.

---

## 🔁 **Rendre le tunnel persistant**

Créer un script dans `/etc/network/if-up.d/ipip-tunnel` ou utiliser `systemd` pour activer au démarrage :

```bash
#!/bin/sh
ip tunnel add tun0 mode ipip remote 192.168.0.2 local 192.168.0.1 ttl 255
ip link set tun0 up
ip addr add 10.0.0.1/30 dev tun0

```

Rends-le exécutable :

```bash
sudo chmod +x /etc/network/if-up.d/ipip-tunnel

```

Fais de même sur l’autre VM avec l’adresse inverse.

---

## ✅ Résultat attendu

- Un tunnel IPIP opérationnel.
- Chaque VM peut joindre l’autre via les adresses `10.0.0.X`.
- Le trafic routé via ce tunnel est encapsulé en IPIP (protocole 4).

---

Souhaites-tu maintenant :

- Ajouter **du chiffrement** (avec IPSec) ?
- Automatiser ça avec **Netplan** ou **NetworkManager** ?
- Voir comment le monitorer avec `tcpdump` ?
