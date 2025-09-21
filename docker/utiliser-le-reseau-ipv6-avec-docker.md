---
description: '⚠️ Important : L’IPv6 n’est supporté que sur les hôtes Linux.'
---

# 🌍 Utiliser le réseau IPv6 avec Docker

### 🔹 Créer un réseau IPv6

#### 1. Réseau IPv6 par défaut

```bash
docker network create --ipv6 ip6net
```

Cela active IPv6 mais sans préciser de sous-réseau.

***

#### 2. Réseau IPv6 avec un sous-réseau spécifique

```bash
docker network create --ipv6 --subnet 2001:db8::/64 ip6net
```

Ici, le réseau est créé avec un **préfixe IPv6 défini** (`2001:db8::/64`).

***

#### 3. Avec Docker Compose

```yaml
networks:
  ip6net:
    enable_ipv6: true
    ipam:
      config:
        - subnet: 2001:db8::/64
```

***

### 🔹 Lancer un conteneur sur le réseau IPv6

Exemple avec l’image de test **whoami** (traefik) :

```bash
docker run --rm --network ip6net -p 80:80 traefik/whoami
```

👉 Ici, le port 80 est publié à la fois en **IPv4** et en **IPv6**.

***

### 🔹 Vérifier la connectivité IPv6

Tester depuis l’hôte avec `curl` :

```bash
curl http://[::1]:80
```

Exemple de sortie :

```
Hostname: ea1cfde18196
IP: 127.0.0.1
IP: ::1
IP: 172.17.0.2
IP: 2001:db8::2
IP: fe80::42:acff:fe11:2
RemoteAddr: [2001:db8::1]:37574
GET / HTTP/1.1
Host: [::1]
User-Agent: curl/8.1.2
Accept: */*
```

👉 On voit que le conteneur est joignable via une **adresse IPv6 (`2001:db8::2`)**.

***

📌 Résumé :

* `--ipv6` active IPv6 sur un réseau Docker.
* `--subnet` permet de définir un bloc d’adresses IPv6.
* Les conteneurs connectés au réseau reçoivent une adresse IPv6.
* Les services exposés fonctionnent en **dual-stack (IPv4 + IPv6)**.

## 🌍 Activer IPv6 sur le réseau _bridge_ par défaut de Docker

### 1. Modifier la configuration du démon Docker

Édite le fichier `/etc/docker/daemon.json` (crée-le s’il n’existe pas) :

```json
{
  "ipv6": true,
  "fixed-cidr-v6": "2001:db8:1::/64"
}
```

#### Explication des options :

* **`"ipv6": true`** → active la prise en charge de l’IPv6 globalement.
* **`"fixed-cidr-v6": "2001:db8:1::/64"`** → définit un sous-réseau IPv6 pour le réseau `bridge`, permettant l’attribution automatique d’adresses IPv6 aux conteneurs.
* _(optionnel)_ **`"ip6tables": true`** → active les règles de filtrage et de NAT IPv6 pour assurer l’isolation réseau et le mapping de ports (par défaut activé).

***

### 2. Redémarrer Docker

Applique les changements en redémarrant le service Docker :

```bash
sudo systemctl restart docker
```

***

### 3. Lancer un conteneur

Exemple avec **traefik/whoami** :

```bash
docker run --rm -p 80:80 traefik/whoami
```

👉 Ce conteneur écoute sur **IPv4 et IPv6** via le réseau bridge.

***

### 4. Vérifier la connectivité IPv6

Teste avec `curl` :

```bash
curl http://[::1]:80
```

Exemple de sortie :

```
Hostname: ea1cfde18196
IP: 127.0.0.1
IP: ::1
IP: 172.17.0.2
IP: 2001:db8:1::242:ac12:2
IP: fe80::42:acff:fe12:2
RemoteAddr: [2001:db8:1::1]:35558
GET / HTTP/1.1
Host: [::1]
User-Agent: curl/8.1.2
Accept: */*
```

👉 On voit bien l’adresse IPv6 attribuée (`2001:db8:1::242:ac12:2`) au conteneur.

***

✅ Résumé :

* Activer IPv6 dans `/etc/docker/daemon.json`.
* Définir un préfixe avec `fixed-cidr-v6`.
* Redémarrer Docker.
* Les conteneurs du réseau _bridge_ obtiennent désormais une adresse IPv6 automatiquement.

## 🌍 Allocation dynamique de sous-réseaux IPv6 avec Docker

### 📌 Fonctionnement par défaut

* Quand tu crées un réseau utilisateur avec `docker network create` **sans préciser de `--subnet`**, Docker choisit une plage d’adresses automatiquement.
* Pour **IPv4**, cela vient des **default-address-pools** définis par le démon Docker.
* Pour **IPv6**, si tu actives IPv6 (`--ipv6` ou `enable_ipv6: true`), mais que tu ne donnes **aucune plage (`--subnet`)**, Docker génère une **ULA (Unique Local Address)** en `/64` avec un identifiant global aléatoire → cela garantit une forte probabilité d’unicité.

👉 Exemple d’adresse générée automatiquement :\
`fd00:1234:5678:abcd::/64`

***

### 📌 Définir tes propres pools d’adresses IPv6

Tu peux forcer Docker à utiliser des plages spécifiques pour IPv6 en configurant le fichier `/etc/docker/daemon.json` :

#### Exemple avec IPv4 par défaut + IPv6 personnalisé

```json
{
  "default-address-pools": [
    { "base": "172.17.0.0/16", "size": 16 },
    { "base": "172.18.0.0/16", "size": 16 },
    { "base": "172.19.0.0/16", "size": 16 },
    { "base": "172.20.0.0/14", "size": 16 },
    { "base": "172.24.0.0/14", "size": 16 },
    { "base": "172.28.0.0/14", "size": 16 },
    { "base": "192.168.0.0/16", "size": 20 },
    { "base": "2001:db8::/56", "size": 64 }
  ]
}
```

#### Explication :

* `"base": "2001:db8::/56"` → ta plage principale IPv6.
* `"size": 64` → Docker divise cette plage en sous-réseaux de `/64`.
* Avec un `/56`, tu as **256 sous-réseaux /64 disponibles** (`2001:db8::/64`, `2001:db8:0:1::/64`, etc.).

⚠️ **Note** :\
L’adresse `2001:db8::/56` est réservée pour la documentation.\
👉 Utilise plutôt une **vraie plage IPv6 ULA** (`fd00::/56`) ou une plage IPv6 fournie par ton opérateur.

***

### 📌 Exemple d’utilisation

1. Configure ton `daemon.json` avec la plage IPv6.
2.  Redémarre Docker :

    ```bash
    sudo systemctl restart docker
    ```
3.  Crée un réseau sans préciser de `--subnet` :

    ```bash
    docker network create --ipv6 mynet
    ```

    👉 Docker attribuera automatiquement un sous-réseau `/64` issu de `2001:db8::/56`.
4.  Vérifie le réseau :

    ```bash
    docker network inspect mynet
    ```

***

✅ En résumé :

* Sans configuration → Docker utilise des **ULAs aléatoires** en IPv6.
* Avec `default-address-pools` → tu maîtrises les **plages IPv6 dynamiques**.
* Cela évite les conflits et rend les adresses plus prévisibles.

## 🐳 Docker in Docker et IPv6

### 📌 Contexte

* **Docker in Docker (DinD)** = exécuter un démon Docker (`dockerd`) **à l’intérieur d’un conteneur**.
* Par défaut, le démon gère les règles réseau avec **iptables/nftables**.
* Pour **IPv6**, le module noyau **`ip6_tables`** doit être présent et chargé sur l’hôte.

### 🔹 Problème

* Docker utilise **iptables** (ou **nftables**) pour gérer les règles réseau.
* Quand on active **IPv6**, le module noyau **`ip6_tables`** doit être chargé pour permettre à Docker de créer des réseaux IPv6.
* Sur un hôte classique, ce module est chargé automatiquement au démarrage de Docker.
* Mais dans un environnement **Docker-in-Docker (DinD)** (souvent utilisé pour CI/CD avec l’image `docker:dind`), le module peut **ne pas être chargé**, surtout si l’image n’est pas basée sur une version récente.

👉 Résultat : impossible de créer un réseau IPv6 à l’intérieur du Docker-in-Docker.

***

### 🔹 Solutions possibles

#### ✅ 1. Charger le module `ip6_tables` sur l’hôte

Avant de démarrer le container DinD :

```bash
sudo modprobe ip6_tables
```

Ainsi, le module est disponible pour le Docker Engine tournant dans le conteneur DinD.

***

#### ✅ 2. Désactiver ip6tables dans DinD

Si tu ne veux pas utiliser IPv6 (ou si tu veux contourner le problème), tu peux désactiver `ip6tables` dans la configuration du démon Docker :

```bash
dockerd --ip6tables=false
```

Ou dans `/etc/docker/daemon.json` :

```json
{
  "ip6tables": false
}
```

***

#### ✅ 3. Utiliser une image DinD récente

Les images officielles `docker:dind` récentes gèrent mieux ces cas.\
Exemple :

```yaml
services:
  docker:
    image: docker:27-dind
    privileged: true
    environment:
      - DOCKER_TLS_CERTDIR=/certs
    volumes:
      - docker-certs-ca:/certs/ca
      - docker-certs-client:/certs/client
      - docker-data:/var/lib/docker
```

Cela réduit les problèmes liés aux modules noyau manquants.

***

### 🔹 En résumé

* Si tu veux **IPv6 actif dans Docker-in-Docker** → assure-toi que `ip6_tables` est bien chargé **sur l’hôte** (`modprobe ip6_tables`).
* Si tu ne veux **pas d’IPv6** → désactive avec `--ip6tables=false`.
* Si possible, utilise **une image DinD récente** pour éviter d’avoir à gérer ça manuellement.
