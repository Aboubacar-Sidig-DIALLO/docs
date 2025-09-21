# 🔥 Filtrage de paquets et pare-feux (Docker + iptables)

### 🔹 Rôle des règles iptables créées par Docker

* Sur **Linux**, Docker configure automatiquement des règles **iptables** et **ip6tables**.
* Ces règles servent à :
  * isoler les réseaux de conteneurs (bridge networks),
  * publier des ports (`-p`),
  * filtrer le trafic.

⚠️ **Ne pas modifier les règles Docker** → elles sont nécessaires au bon fonctionnement.\
👉 Aucun ajout de règles pour les réseaux :

* `ipvlan`,
* `macvlan`,
* ou en mode `host`.

***

### 🔹 Chaînes iptables créées par Docker

Docker ajoute ses propres **chaînes personnalisées** :

| Chaîne                           | Rôle                                                                              |
| -------------------------------- | --------------------------------------------------------------------------------- |
| **DOCKER-USER**                  | Espace réservé aux règles de l’utilisateur. 📌 Traité **avant** celles de Docker. |
| **DOCKER-FORWARD**               | Première étape de traitement du trafic Docker.                                    |
| **DOCKER**                       | Gère le NAT et la publication des ports (`-p`).                                   |
| **DOCKER-ISOLATION-STAGE-1 / 2** | Isole les réseaux Docker entre eux.                                               |
| **DOCKER-INGRESS**               | Gère le trafic Swarm (mode cluster).                                              |

Dans la chaîne **FORWARD**, Docker redirige toujours vers :\
➡️ `DOCKER-USER → DOCKER-FORWARD → DOCKER-INGRESS`.

Dans la table **nat**, Docker ajoute une chaîne `DOCKER` pour :

* le **masquerading** (NAT),
* la **traduction de ports** (PAT).

***

### 🔹 Ajouter ses propres règles

* Les règles placées dans `DOCKER-USER` passent **avant** celles de Docker → recommandé pour renforcer la sécurité.
* Les règles ajoutées directement dans `FORWARD` passent **après** celles de Docker.

⚠️ Attention : quand un paquet arrive dans `DOCKER-USER`, il a déjà subi une **DNAT** (traduction d’adresse et de port).\
👉 Si tu veux filtrer selon l’adresse IP/port **original**, utilise l’extension **conntrack**.

#### Exemple :

```bash
# Autoriser les connexions déjà établies
sudo iptables -I DOCKER-USER -p tcp -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Autoriser en fonction de l’IP/port d’origine
sudo iptables -I DOCKER-USER -p tcp -m conntrack --ctorigdst 198.51.100.2 --ctorigdstport 80 -j ACCEPT
```

***

### 🔹 Publication et mappage de ports

* Par défaut, les ports des conteneurs **ne sont pas exposés** au monde extérieur.
* Avec `-p`, Docker crée une règle iptables pour rediriger un port du **host** vers le **conteneur**.

#### Exemple :

| Commande                   | Effet                                                               |
| -------------------------- | ------------------------------------------------------------------- |
| `-p 8080:80`               | Mappe le port **8080 du host** vers le port **80 du conteneur**.    |
| `-p 192.168.1.100:8080:80` | Même chose mais uniquement sur l’IP **192.168.1.100** du host.      |
| `-p 8080:80/udp`           | Mappe en UDP.                                                       |
| `-p 127.0.0.1:8080:80`     | Port accessible **uniquement depuis le host** (sécurité renforcée). |

⚠️ Publier un port l’expose par défaut à **tout l’internet** sauf si tu limites avec `127.0.0.1` ou iptables.

***

### 🔹 Restreindre l’accès externe aux conteneurs

Par défaut, tous les IP externes peuvent accéder aux ports publiés.\
👉 Tu peux restreindre avec `DOCKER-USER`.

#### Exemple : autoriser seulement 1 IP

```bash
iptables -I DOCKER-USER -i ext_if ! -s 192.0.2.2 -j DROP
```

#### Exemple : autoriser seulement un sous-réseau

```bash
iptables -I DOCKER-USER -i ext_if ! -s 192.0.2.0/24 -j DROP
```

#### Exemple : autoriser une plage d’IP

```bash
iptables -I DOCKER-USER -m iprange -i ext_if ! --src-range 192.0.2.1-192.0.2.3 -j DROP
```

#### Exemple : autoriser les connexions établies

```bash
iptables -I DOCKER-USER -m state --state RELATED,ESTABLISHED -j ACCEPT
```

***

### ✅ À retenir

* Docker **gère iptables automatiquement** pour ses réseaux.
* Tu peux ajouter tes règles de sécurité dans **DOCKER-USER** (jamais dans DOCKER).
* Publier un port (`-p`) = exposition externe (⚠️ risque sécurité).
* Utilise `127.0.0.1:PORT` ou iptables pour limiter l’accès.
* **conntrack** est nécessaire si tu veux matcher les IP/ports originaux avant NAT.

## 🌐 Docker – Routage direct (Direct Routing)

### 🔹 Qu’est-ce que le routage direct ?

* Par défaut, Docker **publie des ports** avec du **NAT (Network Address Translation)**.
* Cela signifie que l’adresse IP réelle du conteneur n’est pas routée directement, seul le port mappé sur le **host** est accessible depuis l’extérieur.
* Avec le **direct routing**, tu permets aux clients externes d’accéder **directement à l’IP du conteneur**, sans passer par une translation de ports.

👉 Ce mode est particulièrement utile avec **IPv6**, où le NAT est à éviter.

***

### 🔹 Accéder aux conteneurs via routage direct

Par défaut :

* Les **IP internes des conteneurs** (dans les réseaux bridge) ne sont pas accessibles depuis l’extérieur.
* Seuls les **ports publiés (`-p`)** sont atteignables via les adresses du host.

#### Pour activer le direct routing globalement :

Dans `/etc/docker/daemon.json` :

```json
{
  "allow-direct-routing": true
}
```

ou avec le flag `--allow-direct-routing`.

***

### 🔹 Routage direct sur une interface spécifique

Tu peux limiter le routage direct à certaines interfaces avec l’option :

```
-o com.docker.network.bridge.trusted_host_interfaces="vxlan.1:eth3"
```

#### Exemple

Créer un réseau bridge où seules `vxlan.1` et `eth3` permettent le routage direct :

```bash
docker network create \
  --subnet 192.0.2.0/24 \
  --ip-range 192.0.2.0/29 \
  -o com.docker.network.bridge.trusted_host_interfaces="vxlan.1:eth3" \
  mynet
```

Lancer un conteneur dans ce réseau :

```bash
docker run -d --ip 192.0.2.100 -p 127.0.0.1:8080:80 nginx
```

🔎 Résultat :

* Accessible via `http://127.0.0.1:8080` (port publié sur host).
* Accessible **directement** via `http://192.0.2.100:80` (IP du conteneur).
* Accessible aussi depuis les hôtes du réseau local qui ont une route vers `192.0.2.0/24`.

***

### 🔹 Modes de passerelle (Gateway modes)

Le driver `bridge` propose différents **modes de gateway** pour IPv4 et IPv6 :

| Mode                | Description                                                                                          |
| ------------------- | ---------------------------------------------------------------------------------------------------- |
| **nat** (défaut)    | NAT + masquerading. Les conteneurs sortent avec l’IP du host.                                        |
| **routed**          | Pas de NAT. Les paquets sortent avec l’IP du conteneur. Ports publiés restent protégés par iptables. |
| **nat-unprotected** | Comme nat, mais **tous les ports** sont accessibles par routage direct (⚠️ dangereux).               |
| **isolated**        | Réseau isolé (`--internal`). Pas d’adresse attribuée au bridge, pas d’accès externe.                 |

#### Exemple : réseau IPv6 routé

```bash
docker network create --ipv6 \
  --subnet 2001:db8::/64 \
  -o com.docker.network.bridge.gateway_mode_ipv6=routed \
  mynet
```

Lancer un conteneur avec port publié :

```bash
docker run --network=mynet -p 8080:80 myimage
```

🔎 Résultat :

* En **IPv6 (routed)** → le conteneur est joignable directement via son IP (`[2001:db8::...]:80`).
* En **IPv4 (nat)** → accessible uniquement via `host:8080`.

***

### 🔹 Configurer l’adresse par défaut de binding

Par défaut :

* Docker publie les ports sur **toutes les adresses du host** (`0.0.0.0` et `::`).

👉 Tu peux changer ça pour limiter les ports publiés à `127.0.0.1`.

#### Exemple : réseau user-defined

```bash
docker network create mybridge \
  -o "com.docker.network.bridge.host_binding_ipv4=127.0.0.1"
```

#### Exemple : réseau bridge par défaut

Dans `/etc/docker/daemon.json` :

```json
{
  "ip": "127.0.0.1"
}
```

Ensuite :

```bash
sudo systemctl restart docker
```

***

### ✅ À retenir

* **Direct routing** permet d’accéder directement à l’IP du conteneur (sans NAT).
* Tu peux l’activer globalement (`allow-direct-routing`) ou par réseau (`trusted_host_interfaces`).
* Les **modes gateway** (`nat`, `routed`, `isolated`) définissent le comportement réseau.
* En **IPv6 routé**, l’adresse du conteneur est accessible directement de l’extérieur.
* Attention à la **sécurité** : `nat-unprotected` expose tous les ports du conteneur.

## 🚦 Docker sur un routeur (Docker on a router)

### 🔹 IP Forwarding

* Pour que Docker fonctionne, il doit activer le **transfert IP (IP Forwarding)**.
* Au démarrage, Docker active automatiquement :
  * `net.ipv4.ip_forward=1`
  * `net.ipv6.conf.all.forwarding=1`

👉 Cela permet aux conteneurs de communiquer avec l’extérieur via l’hôte.

⚠️ Problème :\
Quand Docker active l’IP forwarding, il définit aussi la **politique par défaut de la chaîne FORWARD** d’iptables sur **DROP**.\
➡️ Résultat : ton hôte Docker ne peut plus agir comme routeur pour d’autres machines.\
➡️ C’est volontaire (mesure de sécurité recommandée).

***

### 🔹 Empêcher Docker de forcer DROP

Si tu veux que ton hôte Docker agisse comme **routeur Linux classique**, tu as 2 solutions :

#### 1. Configuration dans `daemon.json`

Ajoute dans `/etc/docker/daemon.json` :

```json
{
  "ip-forward-no-drop": true
}
```

Puis redémarre Docker :

```bash
sudo systemctl restart docker
```

#### 2. Option dockerd

Lancer Docker avec :

```bash
dockerd --ip-forward-no-drop
```

#### 3. Ajouter une règle `iptables`

Tu peux aussi garder la politique `DROP` mais autoriser explicitement certains flux avec la chaîne **DOCKER-USER** :

```bash
iptables -I DOCKER-USER -i src_if -o dst_if -j ACCEPT
```

(exemple : autoriser le trafic entre deux interfaces précises).

***

### 🔹 Différence selon les versions

* **Avant Docker 28.0.0** → Docker mettait **toujours** `DROP` sur la chaîne FORWARD IPv6.
* **Depuis 28.0.0** → Docker ne met `DROP` que s’il active lui-même le forwarding IPv6.

👉 Donc, si ton hôte active déjà IPv6 forwarding avant Docker, **c’est ta config système** qui décide de la sécurité.

***

### 🔹 Empêcher Docker de manipuler iptables

Tu peux désactiver la gestion des règles iptables par Docker :

Dans `/etc/docker/daemon.json` :

```json
{
  "iptables": false,
  "ip6tables": false
}
```

⚠️ Mais attention :

* Plus de règles de filtrage → tous les ports des conteneurs seront accessibles.
* Plus de mappage de ports → les `-p` ne fonctionnent plus.
* Tu devras gérer **manuellement** tout le firewall → très complexe et risqué.

👉 Donc réservé aux **experts réseaux** qui veulent une config firewall totalement custom.

***

### 🔹 Intégration avec firewalld

* Si tu utilises **firewalld** (souvent sur Fedora, RHEL, CentOS) :
  * Docker crée une **zone `docker`** avec `target=ACCEPT`.
  * Toutes les interfaces Docker (`docker0`, etc.) sont ajoutées dans cette zone.
  * Docker crée une règle de forwarding `docker-forwarding` qui autorise le trafic **de n’importe quelle zone → vers docker**.

👉 Résultat : les conteneurs sont accessibles selon les règles de firewalld, mais avec une **zone spéciale dédiée**.

***

### 🔹 Docker et UFW (Ubuntu/Debian)

* **UFW (Uncomplicated Firewall)** est un frontend iptables.
* Problème : Docker utilise la table `nat` pour rediriger le trafic **avant** que les règles UFW soient appliquées (dans `INPUT` et `OUTPUT`).\
  ➡️ Résultat : **les règles UFW sont ignorées** pour les conteneurs.\
  ➡️ Donc un conteneur peut être accessible depuis l’extérieur **même si UFW le bloque**.

⚠️ Cela rend **Docker et UFW incompatibles** sans configuration avancée.

***

### ✅ À retenir

* Docker active **IP Forwarding** et applique une politique de sécurité stricte (`DROP`) par défaut.
* Si tu veux que l’hôte agisse comme **routeur**, tu peux utiliser `ip-forward-no-drop` ou ajouter des règles dans `DOCKER-USER`.
* Avec **firewalld**, Docker crée une zone spéciale `docker`.
* Avec **UFW**, les règles sont **contournées** → il faut utiliser iptables directement ou un patch spécifique.
