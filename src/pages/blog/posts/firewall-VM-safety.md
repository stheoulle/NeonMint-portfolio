---
layout: /src/layouts/MarkdownPostLayout.astro
title: Firewall and VM Safety, A Comprehensive Guide
author: Sarah THEOULLE
description: "A detailed guide on how to secure your virtual machines with firewalls, ensuring safe and efficient operations in a virtualized environment."
image:
  url: "/images/posts/firewall.png"
  alt: ""
pubDate: 2025-07-01
tags:
  [
    "documentation",
    "tutorial",
    "firewall",
    "virtualization",
    "security",
  ]
languages: ["bash", "markdown", "yaml"]
---

### 🎯 Objectifs

- Limiter les requêtes ping
- Protéger l’accès SSH avec Fail2Ban
- Rediriger un port vers un service
- Logger les accès HTTP
- Bloquer les requêtes contenant certains mots-clés

---

## 🔧 Prérequis

- Une VM Linux (Debian/Ubuntu)
- Accès root (`sudo`)
- Un service tournant (ex: un serveur web sur le port 8080)

---

## 1. 🔐 Limiter le Ping (ICMP)

Limiter les requêtes ping entrantes à 1 par seconde :

```bash
sudo iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 30/minute -j ACCEPT
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```

✅ Vérifie avec :

```bash
ping -f <adresse_ip_de_ta_vm>

```

---

## 2. 🛡️ Sécuriser SSH avec Fail2Ban

### Installation

```bash
sudo apt update
sudo apt install fail2ban -y

```

### Configuration

```bash
sudo nano /etc/fail2ban/jail.local

```

Ajouter :

```bash
[sshd]
enabled = true
port    = ssh
logpath = %(sshd_log)s
backend = systemd
maxretry = 3
bantime  = 3600
findtime = 600

```

### Activation

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd

```

✅ Vérifie que Fail2Ban détecte bien des tentatives de connexion SSH anormales.

---

## 3. 🔁 Redirection du port 8000 vers un service (ex: 8080)

Si ton application écoute sur le port `8080`, redirige le trafic entrant sur `8000` :

```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 8000 -j REDIRECT --to-port 8080
sudo iptables -t nat -A OUTPUT -p tcp --dport 8000 -j REDIRECT --to-port 8080
```

✅ Teste avec :

```bash
curl http://localhost:8000

```

---

## 4. 📜 Logger les accès HTTP (port 80)

Ajouter une règle de log pour surveiller les accès au port 80 :

```bash
sudo iptables -A INPUT -p tcp --dport 80 -j LOG --log-prefix "HTTP ACCESS: "

```

✅ Voir les logs :

```bash
sudo tail -f /var/log/syslog

```

---

## 5. 🚫 Bloquer les requêtes contenant le mot "lamp"

Fonctionne uniquement avec du **trafic HTTP (non-HTTPS)** :

```bash
sudo iptables -A INPUT -p tcp --dport 80 -m string --string "lamp" --algo bm -j DROP

```

✅ Teste en envoyant une requête contenant ce mot (curl, telnet...).

---

## 🧠 Bonus : Rendre les règles persistantes

```bash
sudo apt install iptables-persistent -y
sudo netfilter-persistent save

```

---

## 📄 Vérification de l’état des règles

Lister les règles `iptables` :

```bash
sudo iptables -L -n -v

```

Lister les règles NAT :

```bash
sudo iptables -t nat -L -n -v

```

---

## 📦 Script Bash (optionnel)

Créer un script pour automatiser tout le TP :

```bash
nano tp_firewall.sh

```

Contenu :

```bash
#!/bin/bash
# TP Firewall

# Limiter le ping
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/second -j ACCEPT
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

# Redirection du port 8000 vers 8080
iptables -t nat -A PREROUTING -p tcp --dport 8000 -j REDIRECT --to-port 8080

# Log des accès sur le port 80
iptables -A INPUT -p tcp --dport 80 -j LOG --log-prefix "HTTP ACCESS: "

# Bloquer les requêtes contenant "lamp"
iptables -A INPUT -p tcp --dport 80 -m string --string "lamp" --algo bm -j DROP

# Sauvegarde
netfilter-persistent save

```

Puis rends-le exécutable :

```bash
chmod +x tp_firewall.sh
sudo ./tp_firewall.sh

```

---

## Ping delayed

```yaml
ping 192.168.122.242
PING 192.168.122.242 (192.168.122.242) 56(84) bytes of data.
64 bytes from 192.168.122.242: icmp_seq=1 ttl=64 time=0.877 ms
64 bytes from 192.168.122.242: icmp_seq=2 ttl=64 time=0.550 ms
64 bytes from 192.168.122.242: icmp_seq=3 ttl=64 time=0.656 ms
64 bytes from 192.168.122.242: icmp_seq=4 ttl=64 time=0.604 ms
[...]
64 bytes from 192.168.122.242: icmp_seq=111 ttl=64 time=0.554 ms
64 bytes from 192.168.122.242: icmp_seq=113 ttl=64 time=0.665 ms
^C
--- 192.168.122.242 ping statistics ---
114 packets transmitted, 62 received, 45.614% packet loss, time 115693ms
rtt min/avg/max/mdev = 0.321/0.575/0.930/0.138 ms
```

On a bien un paquet sur deux qui est drop, ce qui correspond à ma règle de 30 ping par minute

## Fail2Ban

```yaml
sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed: 3
|  `- Journal matches: _SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned: 1
   |- Total banned: 1
   `- Banned IP list: 192.168.122.1
```

## Server on 8080

```yaml
nohup python3 -m http.server 8080 >/dev/null 2>&1 &
[1] 2031
```

## Redirect 8080 on 8000

```yaml
sudo iptables -L -v -n
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
   62  5208 ACCEPT     1    --  *      *       0.0.0.0/0            0.0.0.0/0            icmptype 8 limit: avg 30/min burst 5
   52  4368 DROP       1    --  *      *       0.0.0.0/0            0.0.0.0/0            icmptype 8

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
heralys@debian:~$ sudo iptables -t nat -L -v -n
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 REDIRECT   6    --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:8000 redir ports 8080

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    1    60 REDIRECT   6    --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:8000 redir ports 8080

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination  
```

Ping le [localhost:8000](http://localhost:8000) et on a bien le serveur python du 8080 qui répond

## Requête lamp

```yaml
 curl -v "http://192.168.122.242/lamp"
*   Trying 192.168.122.242:80...
* Connected to 192.168.122.242 (192.168.122.242) port 80
> GET /lamp HTTP/1.1
> Host: 192.168.122.242
> User-Agent: curl/8.9.1
> Accept: */*
> 
* Request completely sent off
^C
ubuntu@vps-67b06a10:~$ curl -v "http://192.168.122.242"
*   Trying 192.168.122.242:80...
* Connected to 192.168.122.242 (192.168.122.242) port 80
> GET / HTTP/1.1
> Host: 192.168.122.242
> User-Agent: curl/8.9.1
> Accept: */*
> 
* Request completely sent off
* HTTP 1.0, assume close after body
< HTTP/1.0 200 OK
< Server: SimpleHTTP/0.6 Python/3.11.2
< Date: Mon, 16 Jun 2025 07:35:17 GMT
< Content-type: text/html; charset=utf-8
< Content-Length: 642
< 
<!DOCTYPE HTML>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href=".bash_history">.bash_history</a></li>
<li><a href=".bash_logout">.bash_logout</a></li>
<li><a href=".bashrc">.bashrc</a></li>
<li><a href=".cache/">.cache/</a></li>
<li><a href=".config/">.config/</a></li>
<li><a href=".face">.face</a></li>
<li><a href=".face.icon">.face.icon@</a></li>
<li><a href=".local/">.local/</a></li>
<li><a href=".profile">.profile</a></li>
<li><a href=".sudo_as_admin_successful">.sudo_as_admin_successful</a></li>
</ul>
<hr>
</body>
</html>
* shutting down connection #0
```

Si on a un lamp la requête est bien bloquée

```yaml
sudo iptables -L INPUT -v -n --line-numbers
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1       62  5208 ACCEPT     1    --  *      *       0.0.0.0/0            0.0.0.0/0            icmptype 8 limit: avg 30/min burst 5
2       52  4368 DROP       1    --  *      *       0.0.0.0/0            0.0.0.0/0            icmptype 8
3       38  3722 LOG        6    --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80 LOG flags 0 level 4 prefix "HTTP ACCESS: "
4       19  2546 DROP       6    --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80 STRING match  "lamp" ALGO name bm
```
