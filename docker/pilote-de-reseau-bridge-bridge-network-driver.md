# 🌉 Pilote de réseau Bridge (Bridge network driver)

### 🔹 Qu’est-ce qu’un bridge réseau ?

En termes réseaux, un **bridge** (pont) est un périphérique de couche 2 (Link Layer) qui transfère le trafic entre différents segments réseau.\
Ce pont peut être **matériel** (switch) ou **logiciel** (dans le noyau de la machine hôte).

Dans Docker, un réseau **bridge** est un pont logiciel qui permet à des conteneurs **connectés au même réseau bridge** de communiquer entre eux, tout en étant isolés des conteneurs qui ne sont pas connectés à ce réseau.

👉 Le pilote **bridge** installe automatiquement des règles dans l’hôte afin que les conteneurs de réseaux différents ne puissent pas communiquer directement.

***

### 🔹 Utilisation

* Les réseaux bridge s’appliquent aux conteneurs qui tournent sur le **même démon Docker (même hôte)**.
* Pour communiquer entre conteneurs sur des hôtes différents :
  * Soit tu configures le routage au niveau de l’OS,
  * Soit tu utilises un réseau **overlay** (plus pratique).

***

### 🔹 Le réseau bridge par défaut

Quand tu démarres Docker :

* Un réseau par défaut nommé **bridge** est créé automatiquement.
* Tous les nouveaux conteneurs y sont connectés **sauf si tu spécifies un autre réseau**.
* Tu peux aussi créer tes propres réseaux bridge personnalisés (**user-defined bridges**), ce qui est recommandé.

***

### 🔹 Différences entre le **bridge par défaut** et les **bridges définis par l’utilisateur**

#### ✅ Résolution DNS

* **Bridge par défaut** : les conteneurs ne peuvent se joindre **qu’avec leur adresse IP**, sauf si tu utilises l’option **--link** (⚠️ obsolète).
* **Bridge défini par l’utilisateur** : les conteneurs peuvent se joindre par **nom** ou **alias**.\
  👉 Exemple : un conteneur `web` peut se connecter à `db` en utilisant simplement `db` comme hostname.

***

#### ✅ Isolation

* **Bridge par défaut** : tous les conteneurs sans `--network` sont connectés dessus → risque que des services/applications non liés puissent communiquer.
* **Bridge défini par l’utilisateur** : seul un groupe de conteneurs **dans le même réseau** peut communiquer → meilleure isolation.

***

#### ✅ Connexion/déconnexion dynamique

* **Bridge par défaut** : pour changer de réseau, tu dois **arrêter le conteneur et le recréer**.
* **Bridge défini par l’utilisateur** : tu peux **connecter/déconnecter** un conteneur à chaud (`docker network connect` / `disconnect`).

***

#### ✅ Configuration

* **Bridge par défaut** : configurable mais toutes les applis utilisent les mêmes paramètres (MTU, iptables…).\
  ⚠️ Modifier sa config nécessite un **redémarrage de Docker**.
* **Bridge défini par l’utilisateur** : configurable **indépendamment** via `docker network create`.

***

#### ✅ Variables d’environnement partagées

* **Bridge par défaut** : initialement, tu pouvais partager des variables avec `--link`.
* **Bridge défini par l’utilisateur** : pas de `--link`, mais meilleures alternatives existent :
  * Monter un volume partagé,
  * Utiliser **docker-compose** pour définir des variables partagées,
  * Utiliser **Swarm services** avec secrets/configs.

***

👉 **Rappel** :

* Les conteneurs connectés au **même réseau bridge personnalisé** exposent **tous leurs ports entre eux**.
* Pour exposer un port au monde extérieur ou à d’autres réseaux, il faut utiliser **-p / --publish**.

***

### 🔹 Options disponibles avec le driver bridge

Quand tu crées un réseau personnalisé avec `docker network create -d bridge`, tu peux utiliser **--opt** pour configurer :

| Option                                                            | Défaut       | Description                                                |
| ----------------------------------------------------------------- | ------------ | ---------------------------------------------------------- |
| `com.docker.network.bridge.name`                                  |              | Nom de l’interface Linux à créer pour le bridge            |
| `com.docker.network.bridge.enable_ip_masquerade`                  | true         | Activer le masquerading IP (NAT)                           |
| `com.docker.network.bridge.gateway_mode_ipv4 / gateway_mode_ipv6` | nat          | Contrôle la connectivité externe (voir règles firewall)    |
| `com.docker.network.bridge.enable_icc`                            | true         | Autoriser ou non la communication entre conteneurs         |
| `com.docker.network.bridge.host_binding_ipv4`                     | toutes       | Adresse IP par défaut pour la publication de ports         |
| `com.docker.network.driver.mtu`                                   | 0 (illimité) | Taille MTU pour les paquets réseau des conteneurs          |
| `com.docker.network.container_iface_prefix`                       | eth          | Préfixe personnalisé pour les interfaces réseau conteneurs |
| `com.docker.network.bridge.inhibit_ipv4`                          | false        | Empêcher Docker d’assigner une IP au bridge                |

***

### 🔹 Flags équivalents dans `dockerd`

Certaines options existent aussi comme **flags du démon Docker** :

| Option                 | Flag équivalent |
| ---------------------- | --------------- |
| `enable_ip_masquerade` | `--ip-masq`     |
| `enable_icc`           | `--icc`         |
| `host_binding_ipv4`    | `--ip`          |
| `mtu`                  | `--mtu`         |

Tu peux aussi utiliser `--bridge` pour définir ton **propre bridge docker0** (utile si plusieurs démons Docker tournent sur le même hôte).

***

✅ En résumé :

* **bridge par défaut** = simple mais limité.
* **bridges définis par l’utilisateur** = isolation, flexibilité, DNS entre conteneurs, configuration indépendante.\
  👉 Toujours privilégier les **user-defined bridge networks**.

## 🌍 Adresse de liaison par défaut de l’hôte (Default host binding address)

#### 🔹 Comportement par défaut

Quand tu publies un port sans préciser d’adresse hôte, par exemple :

```bash
docker run -p 80 nginx
docker run -p 8080:80 nginx
```

👉 Le port 80 du conteneur est exposé sur **toutes les adresses de l’hôte**, en IPv4 et IPv6 (`0.0.0.0` et `::`).

***

#### 🔹 Modifier l’adresse par défaut

Tu peux utiliser l’option du driver bridge :

```json
com.docker.network.bridge.host_binding_ipv4
```

* Permet de définir **l’adresse par défaut** où les ports publiés s’attacheront.
* Malgré son nom, tu peux aussi y mettre une **adresse IPv6**.

***

#### 🔹 Exemples

* Si tu définis `127.0.0.1`, les ports publiés ne seront accessibles **que localement**.
* Si tu définis `::`, seuls les ports IPv6 seront accessibles.
* Si tu définis `0.0.0.0`, les ports seront accessibles **en IPv4 et IPv6**.

👉 Pour limiter explicitement à l’IPv4, tu dois le préciser dans l’option `-p` :

```bash
docker run -p 0.0.0.0:8080:80 nginx
```

***

## 🔧 Gérer un réseau bridge défini par l’utilisateur

#### 🔹 Créer un réseau

```bash
docker network create my-net
```

👉 Tu peux ajouter des options (`--subnet`, `--gateway`, `--ip-range`, etc.)

***

#### 🔹 Supprimer un réseau

```bash
docker network rm my-net
```

⚠️ Tu dois **d’abord déconnecter les conteneurs** liés à ce réseau.

***

#### 🔹 Connecter un conteneur

Lors de la création :

```bash
docker create --name my-nginx \
  --network my-net \
  --publish 8080:80 \
  nginx:latest
```

Sur un conteneur existant :

```bash
docker network connect my-net my-nginx
```

***

#### 🔹 Déconnecter un conteneur

```bash
docker network disconnect my-net my-nginx
```

***

## 🌐 Utiliser IPv6 avec un réseau bridge

#### 🔹 Activer IPv6

```bash
docker network create --ipv6 --subnet 2001:db8:1234::/64 my-net
```

* Si aucun `--subnet` n’est donné, Docker choisit automatiquement un préfixe **ULA** (fd00::/8).

***

#### 🔹 Réseau IPv6 uniquement

```bash
docker network create --ipv6 --ipv4=false v6net
```

⚠️ Impossible de désactiver IPv4 sur le réseau bridge **par défaut**.

***

## 🏗️ Utiliser le réseau bridge par défaut

#### 🔹 Conteneurs connectés automatiquement

Si tu ne précises pas de réseau avec `--network`, le conteneur est placé dans le **bridge par défaut**.\
👉 Ils communiquent **par IP uniquement**, sauf avec l’option obsolète `--link`.

***

#### 🔹 Configurer le bridge par défaut

Cela se fait via `/etc/docker/daemon.json`. Exemple :

```json
{
  "bip": "192.168.1.1/24",
  "fixed-cidr": "192.168.1.0/25",
  "mtu": 1500,
  "default-gateway": "192.168.1.254",
  "dns": ["10.20.1.2","10.20.1.3"]
}
```

* `bip` : IP et masque du bridge
* `fixed-cidr` : plage IP réservée aux conteneurs
* `mtu` : taille max des paquets
* `default-gateway` : passerelle des conteneurs
* `dns` : serveurs DNS utilisés par défaut

***

#### 🔹 Activer IPv6 sur le bridge par défaut

Toujours dans `daemon.json` :

```json
{
  "ipv6": true,
  "bip6": "2001:db8::1111/64",
  "fixed-cidr-v6": "2001:db8::/64",
  "default-gateway-v6": "2001:db8:abcd::89"
}
```

* `ipv6`: active IPv6
* `bip6`: adresse et sous-réseau du bridge IPv6
* `fixed-cidr-v6`: pool d’adresses pour les conteneurs
* `default-gateway-v6`: passerelle par défaut

👉 Si rien n’est précisé, Docker choisit un préfixe ULA automatiquement.

⚠️ Il faut **redémarrer Docker** après modification.

***

## 🚧 Limitations des réseaux bridge

* Un réseau bridge devient instable au-delà de **1000 conteneurs connectés** (limitation du noyau Linux).
* Voir issue : [moby/moby#44973](https://github.com/moby/moby/issues/44973).

***

## 🔒 Désactiver l’IP par défaut du bridge

Option :

```bash
com.docker.network.bridge.inhibit_ipv4=true
```

👉 Crée un réseau sans IP par défaut attribuée au bridge.\
Utile si tu veux gérer manuellement le **gateway** (exemple : ajout d’une interface physique dans le bridge).

⚠️ Sans configuration manuelle, le trafic **entrant/sortant** du bridge ne fonctionnera pas.

***

✅ En résumé :

* Par défaut, les ports publiés sont exposés partout (`0.0.0.0` et `::`).
* Tu peux restreindre ça globalement (via `daemon.json`) ou par conteneur (avec `-p`).
* Toujours préférer **les réseaux bridge définis par l’utilisateur** pour plus de contrôle, IPv6 et isolation.
