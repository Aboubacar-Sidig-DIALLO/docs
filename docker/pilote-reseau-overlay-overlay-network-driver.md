# 🌐 Pilote réseau Overlay (Overlay network driver)

### 🔹 Définition

Le pilote **overlay** permet de créer un **réseau distribué entre plusieurs hôtes Docker** (plusieurs démons Docker).

* Ce réseau est une **couche virtuelle** qui s’ajoute par-dessus les réseaux spécifiques à chaque hôte.
* Les conteneurs connectés à ce réseau peuvent communiquer **sécurisé**ment entre eux (si le chiffrement est activé).
* Docker gère automatiquement le **routage des paquets** vers le bon hôte et le bon conteneur.

👉 Les réseaux **overlay** sont principalement utilisés pour connecter des **services Swarm**, mais on peut aussi les utiliser pour relier des **conteneurs indépendants** situés sur différents hôtes (dans ce cas, il faut activer **Swarm mode**).

***

### 🔧 Créer un réseau Overlay

Avant de créer un réseau Overlay, il faut s’assurer que **tous les nœuds participants** peuvent communiquer entre eux.\
Les ports suivants doivent être ouverts entre hôtes :

| Port                   | Description                                                                                   |
| ---------------------- | --------------------------------------------------------------------------------------------- |
| **2377/tcp**           | Port par défaut du plan de contrôle Swarm (modifiable avec `docker swarm join --listen-addr`) |
| **4789/udp**           | Port utilisé pour le trafic overlay (modifiable avec `docker swarm init --data-path-addr`)    |
| **7946/tcp, 7946/udp** | Communication interne entre nœuds (non configurable)                                          |

Exemple de création d’un réseau overlay :

```bash
docker network create -d overlay --attachable my-attachable-overlay
```

👉 L’option `--attachable` permet :

* aux **conteneurs indépendants** d’utiliser le réseau,
* aux **services Swarm** de s’y connecter.

Sans `--attachable`, seul Swarm peut l’utiliser.

***

### 🔒 Chiffrement du trafic

Pour chiffrer les données qui transitent sur un réseau Overlay, on utilise l’option `--opt encrypted` :

```bash
docker network create \
  --opt encrypted \
  --driver overlay \
  --attachable \
  my-attachable-multi-host-network
```

Cela active le chiffrement **IPsec** au niveau du **VXLAN**.

⚠️ Attention :

* Le chiffrement a un **impact sur les performances** → à tester avant en production.
* **Non supporté sous Windows** :
  * Les conteneurs Windows ne peuvent pas communiquer avec les conteneurs Linux.
  * Le trafic entre conteneurs Windows n’est pas chiffré.

***

### 🔗 Attacher un conteneur à un réseau Overlay

Une fois le réseau Overlay créé, on peut y attacher des conteneurs pour qu’ils communiquent **sans configuration réseau manuelle** entre hôtes.

Exemple :

```bash
docker run --network multi-host-network busybox sh
```

👉 ⚠️ Cela ne marche que si le réseau a été créé avec `--attachable`.

***

### 🔍 Découverte des conteneurs

* Les conteneurs sur un réseau Overlay peuvent se **découvrir entre eux via DNS**, en utilisant leur **nom**.
* Exemple : un conteneur `web` pourra être résolu simplement avec `ping web`.

#### Publication de ports

Tu peux exposer un port d’un conteneur Overlay avec `-p` :

| Exemple                         | Description                                                                   |
| ------------------------------- | ----------------------------------------------------------------------------- |
| `-p 8080:80`                    | Mappe le port **80 TCP** du conteneur vers le port **8080** du réseau Overlay |
| `-p 8080:80/udp`                | Mappe le port **80 UDP** du conteneur vers le port **8080**                   |
| `-p 8080:80/sctp`               | Mappe le port **80 SCTP** du conteneur vers le port **8080**                  |
| `-p 8080:80/tcp -p 8080:80/udp` | Mappe **TCP et UDP** sur le même port                                         |

***

### ⚠️ Limite de connexion

Comme pour les réseaux **bridge**, il existe une **limite de 1000 conteneurs par hôte** sur un réseau Overlay (limitation du noyau Linux).\
Au-delà, le réseau devient instable et la communication inter-conteneurs peut casser.

***

✅ **Résumé :**

* Le réseau Overlay connecte des conteneurs **sur plusieurs hôtes Docker**.
* Il nécessite **Swarm mode**.
* Supporte le **chiffrement IPsec (VXLAN)**.
* Permet la **découverte DNS entre conteneurs**.
* Limite : **1000 conteneurs par hôte**.
