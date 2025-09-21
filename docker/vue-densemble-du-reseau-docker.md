# 🌐 Vue d’ensemble du réseau Docker

### 🔹 Qu’est-ce que le container networking ?

* Les **conteneurs Docker** ont le réseau activé par défaut.
* Ils peuvent **communiquer entre eux** ou avec des applications externes (hors Docker).
* Un conteneur voit uniquement :
  * une **interface réseau** avec une adresse IP,
  * une **gateway** (passerelle),
  * une **table de routage**,
  * un **service DNS**, etc.

👉 Le conteneur ne sait pas si ses pairs sont Docker ou non.\
👉 Sauf si on utilise le driver réseau **none**, qui coupe totalement le réseau.

***

### 🔹 Réseaux définis par l’utilisateur

* Tu peux créer tes propres réseaux Docker avec `docker network create`.
* Les conteneurs connectés à ce réseau peuvent communiquer :
  * par **IP**,
  * ou par **nom de conteneur** (résolution DNS interne).

#### Exemple :

```bash
# Créer un réseau bridge personnalisé
docker network create -d bridge my-net

# Lancer un conteneur dans ce réseau
docker run --network=my-net -itd --name=container3 busybox
```

***

### 🔹 Les drivers réseau Docker

Docker fournit plusieurs **pilotes de réseau** par défaut :

| Driver      | Description                                                                                           |
| ----------- | ----------------------------------------------------------------------------------------------------- |
| **bridge**  | Par défaut. Réseau local entre conteneurs sur un même hôte.                                           |
| **host**    | Supprime l’isolation → le conteneur partage la pile réseau de l’hôte.                                 |
| **none**    | Aucune connectivité réseau. Conteneur totalement isolé.                                               |
| **overlay** | Permet de relier plusieurs hôtes Docker (clusters Swarm ou multi-daemons).                            |
| **ipvlan**  | Donne un contrôle total sur l’adressage IPv4/IPv6 (sans NAT).                                         |
| **macvlan** | Attribue une **MAC unique** au conteneur, il apparaît comme un appareil physique sur le réseau local. |

👉 Pour plus de détails : **\[Network drivers overview]** (documentation Docker).

***

### 🔹 Connexion à plusieurs réseaux

Un conteneur peut être **membre de plusieurs réseaux en même temps**.

#### Cas pratique :

* **frontend** : connecté à un réseau **bridge** pour avoir accès à Internet,
* **backend** : connecté à un réseau **interne** (`--internal`) pour communiquer uniquement avec d’autres services internes.

#### Exemple avancé :

```bash
docker run \
  --network name=gwnet,gw-priority=1 \   # réseau principal avec priorité
  --network anet1 \                      # autre réseau secondaire
  --name myctr myimage
```

Puis tu peux aussi connecter un conteneur déjà lancé à un autre réseau :

```bash
docker network connect anet2 myctr
```

***

### 🔹 Gestion de la gateway par défaut

* Quand un conteneur est connecté à plusieurs réseaux, **Docker choisit automatiquement une gateway par défaut**.
* Ce choix peut changer si la configuration du réseau change.
* Pour forcer une gateway par défaut → utilise `gw-priority`.

👉 La gateway avec la priorité la plus haute (valeur la plus grande) devient la **gateway par défaut**.

Exemple :

```bash
docker run --network name=gwnet,gw-priority=1 --name app myimage
```

***

✅ En résumé :

* Les conteneurs communiquent par défaut via **bridge**.
* Tu peux définir des réseaux personnalisés (**user-defined networks**) pour isoler ou relier des services.
* Plusieurs drivers existent selon tes besoins (bridge, host, overlay, macvlan, etc.).
* Tu peux connecter un conteneur à **plusieurs réseaux simultanément** et gérer la **gateway par défaut** avec `gw-priority`.

## 🌐 Container networks

En plus des réseaux définis par l’utilisateur, Docker permet d’attacher un conteneur directement à la **pile réseau d’un autre conteneur**.

***

### 🔹 Utiliser le mode `container:`

* Flag : `--network container:<name|id>`
* Permet à un conteneur de **partager exactement le même namespace réseau** qu’un autre.
* Les deux conteneurs partagent **IP, interfaces et ports**.

⚠️ Dans ce mode, les options suivantes ne sont **pas supportées** :\
`--add-host`, `--hostname`, `--dns`, `--dns-search`, `--dns-option`, `--mac-address`, `--publish`, `--publish-all`, `--expose`.

#### Exemple :

```bash
# Lancer Redis lié à localhost
docker run -d --name redis example/redis --bind 127.0.0.1

# Lancer redis-cli en utilisant le réseau du conteneur "redis"
docker run --rm -it --network container:redis example/redis-cli -h 127.0.0.1
```

***

### 🔹 Published ports

Par défaut, les conteneurs **bridge** n’exposent aucun port.\
Tu dois utiliser `-p` / `--publish` pour exposer un port vers l’extérieur (host ou monde extérieur).

#### Exemples :

```bash
# Map 8080 sur l’hôte → 80 dans le conteneur (TCP)
-p 8080:80  

# Map sur une IP spécifique de l’hôte
-p 192.168.1.100:8080:80  

# Map UDP
-p 8080:80/udp  

# Map TCP et UDP
-p 8080:80/tcp -p 8080:80/udp
```

⚠️ **Sécurité** : publier un port rend le service accessible à tout le monde.\
👉 Pour limiter l’accès au seul hôte :

```bash
docker run -p 127.0.0.1:8080:80 -p '[::1]:8080:80' nginx
```

💡 Si tu veux qu’un conteneur accède à un autre, **il suffit de les mettre dans le même réseau** → pas besoin de publier de port.

***

### 🔹 IP address et hostname

* Par défaut : allocation **IPv4 activée**.
* Tu peux activer **IPv6** :

```bash
docker network create --ipv6 --ipv4=false v6net
```

* Chaque conteneur reçoit une **IP dans le sous-réseau du réseau Docker**.
* Tu peux connecter un conteneur à plusieurs réseaux :
  * au lancement → `--network` plusieurs fois,
  * ou après → `docker network connect`.
* Options :
  * `--ip` ou `--ip6` → forcer l’adresse IP.
  * `--hostname` → changer le hostname (par défaut = ID du conteneur).
  * `--alias` → donner un alias supplémentaire sur un réseau.

***

### 🔹 DNS services

* Par défaut : conteneurs utilisent les **DNS de l’hôte** (copiés depuis `/etc/resolv.conf`).
* Réseaux **bridge par défaut** → copie simple.
* Réseaux **user-defined** → utilisent le DNS embarqué de Docker.

#### Flags DNS possibles :

| Flag           | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| `--dns`        | IP d’un serveur DNS. Peut être répété.                       |
| `--dns-search` | Domaines de recherche pour compléter les noms non qualifiés. |
| `--dns-opt`    | Options DNS supplémentaires (ex : timeout, attempts).        |
| `--hostname`   | Définir le hostname du conteneur.                            |

***

### 🔹 Custom hosts

* Les conteneurs ont un `/etc/hosts` interne (avec `localhost`, nom du conteneur, etc.).
* Les entrées personnalisées de `/etc/hosts` de l’hôte **ne sont pas héritées**.
* Pour ajouter des entrées : utiliser `--add-host`.

***

### 🔹 Proxy server

Si ton conteneur doit passer par un **proxy HTTP/HTTPS**, tu peux configurer un proxy pour Docker (via variables d’environnement ou config du démon).

***

✅ En résumé :

* Mode `container:` = partager la pile réseau d’un autre conteneur.
* Publier un port expose le conteneur à l’extérieur → à manipuler avec prudence.
* Tu peux gérer IP, hostname, alias, et DNS de façon granulaire.
* Pour la communication interne entre conteneurs → utiliser un réseau Docker plutôt que de publier.
