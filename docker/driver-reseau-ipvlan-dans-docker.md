# 🚀 Driver réseau IPvlan dans Docker

Le driver **IPvlan** donne aux utilisateurs **un contrôle total sur l’adressage IPv4 et IPv6**.\
👉 Le driver **VLAN** s’appuie dessus pour fournir un contrôle complet du **tag VLAN (couche 2)** et même du **routage L3 avec IPvlan** pour ceux qui veulent intégrer leurs conteneurs à un réseau physique existant (**underlay network integration**).

➡️ Pour les déploiements en **overlay** qui masquent les contraintes physiques, voir le driver **overlay multi-hôtes**.

***

### 🔹 Présentation

* IPvlan est une **variante moderne de la virtualisation réseau**.
* Contrairement à l’approche classique avec un **bridge Linux** pour l’isolation, IPvlan s’appuie directement sur une **interface Ethernet Linux** (ou sous-interface) → ce qui réduit la complexité et améliore la performance.
* Résultat :
  * Meilleure **performance** (on évite le pont Linux).
  * Moins de composants intermédiaires.
  * Les interfaces des conteneurs sont **directement attachées à l’interface de l’hôte** → pas besoin de port-mapping pour exposer des services vers l’extérieur.

***

### 🔹 Options disponibles

Quand tu crées un réseau avec IPvlan, tu peux préciser :

| Option        | Valeur par défaut | Description                                          |
| ------------- | ----------------- | ---------------------------------------------------- |
| `ipvlan_mode` | `l2`              | Mode de fonctionnement : `l2`, `l3`, `l3s`           |
| `ipvlan_flag` | `bridge`          | Flag de mode IPvlan : `bridge`, `private`, `vepa`    |
| `parent`      | _(vide)_          | Interface parente à utiliser (ex: `eth0`, `eth0.10`) |

***

### 🔹 Prérequis

* Tous les exemples peuvent être faits sur un **seul hôte Docker**.
* Tu peux utiliser une interface existante (`eth0`) ou une **sous-interface VLAN** (`eth0.10`).
* Si `-o parent` est omis → Docker crée une **interface factice (dummy)**.
* **Version minimale du noyau Linux** : 4.2+ (les versions antérieures ont des bugs).

***

### 🌍 Exemple : IPvlan en mode **L2**

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

(ici, l’image montre la topologie IPvlan L2 simple avec conteneurs reliés à eth0)

Interface parente (`eth0`) :

```bash
ip addr show eth0
3: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 ...
    inet 192.168.1.250/24 brd 192.168.1.255 scope global eth0
```

Création du réseau :

```bash
docker network create -d ipvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o ipvlan_mode=l2 \
  -o parent=eth0 db_net
```

Lancer un conteneur dessus :

```bash
docker run --net=db_net -it --rm alpine /bin/sh
```

⚠️ **Note** : Les conteneurs ne peuvent **pas ping l’hôte directement**, car le noyau Linux bloque volontairement cela pour renforcer l’isolation.

***

### 🌍 Exemple multi-hôtes IPvlan L2

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

(on y voit plusieurs hôtes reliés au même segment L2 avec des conteneurs IPvlan)

Création d’un réseau IPvlan sans préciser de gateway :

```bash
docker network create -d ipvlan \
  --subnet=192.168.1.0/24 \
  -o parent=eth0 db_net_ipv
```

Lancer deux conteneurs et tester la résolution DNS Docker intégrée :

```bash
docker run --net=db_net_ipv --name=ipv1 -itd alpine /bin/sh
docker run --net=db_net_ipv --name=ipv2 -it --rm alpine /bin/sh
ping -c 4 ipv1
```

***

### 🔹 Réseaux isolés avec IPvlan

Deux moyens équivalents :

1. Omettre `-o parent` → Docker crée une interface factice.
2. Utiliser `--internal`.

Exemples :

```bash
docker network create -d ipvlan \
  --subnet=192.168.10.0/24 isolated1

docker network create -d ipvlan \
  --subnet=192.168.11.0/24 --internal isolated2

docker network create -d ipvlan isolated3
```

***

### 🌍 Exemple : IPvlan **802.1Q trunk L2**

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

(on y voit plusieurs VLAN séparés reliés à des conteneurs Docker via eth0.20 et eth0.30)

* VLAN ID 20 (parent = `eth0.20`) :

```bash
docker network create -d ipvlan \
  --subnet=192.168.20.0/24 \
  --gateway=192.168.20.1 \
  -o parent=eth0.20 ipvlan20
```

* VLAN ID 30 (parent = `eth0.30`) :

```bash
docker network create -d ipvlan \
  --subnet=192.168.30.0/24 \
  --gateway=192.168.30.1 \
  -o parent=eth0.30 \
  -o ipvlan_mode=l2 ipvlan30
```

Chaque conteneur peut ping un autre du même VLAN, mais pas entre VLAN → nécessite un routeur externe.

***

### 🌍 Exemple : IPvlan multi-sous-réseaux (L2)

(on y voit des conteneurs répartis sur deux subnets, connectés à un routeur externe)

Création réseau avec 2 sous-réseaux :

```bash
docker network create -d ipvlan \
  --subnet=192.168.114.0/24 --subnet=192.168.116.0/24 \
  --gateway=192.168.114.254 --gateway=192.168.116.254 \
  -o parent=eth0.114 \
  -o ipvlan_mode=l2 ipvlan114
```

***

### ✅ Conclusion

* **IPvlan** = performance + simplicité car pas de bridge intermédiaire.
* Parfait pour une **intégration native avec le réseau physique** (VLANs, routage L2/L3).
* Modes disponibles :
  * `l2` : partage du même domaine de broadcast que l’interface parente.
  * `l3` : chaque conteneur agit comme une interface routée (pas de broadcast).
  * `l3s` : variante simplifiée du L3.
* Compatible avec **VLAN tagging (802.1Q)** → très utile en entreprise.

## 🌐 Docker – Exemple d’utilisation du mode **IPvlan L3**

### 🔹 Principes

* En **mode L3**, IPvlan agit comme un **routeur** : chaque conteneur appartient à un **nouveau sous-réseau**.
* Les **routes doivent être distribuées** aux autres hôtes ou au réseau amont, sinon le trafic externe ne saura pas comment joindre ces sous-réseaux.
* Contrairement au mode L2 :
  * **Pas de broadcast** ni de multicast.
  * Chaque sous-réseau est indépendant → meilleure **scalabilité** et **prévisibilité**.
  * On évite les problèmes classiques de boucles de pontage (Bridge Port Data Units – BPDU).
* L3 réduit le domaine de panne : un bug ou une surcharge de broadcast reste **cantonné à l’hôte local**.

⚠️ L3 **nécessite un sous-réseau différent** de celui de l’interface parente (`eth0`).

***

### 🌍 Exemple : Création réseau IPvlan L3

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

(on y voit que le L2 partage le VLAN, alors que le L3 crée des sous-réseaux routés par l’hôte)

Interface parente (`eth0`) :

```bash
ip a show eth0
3: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 ...
    inet 192.168.1.250/24 brd 192.168.1.255 scope global eth0
```

Création réseau IPvlan L3 :

```bash
docker network create -d ipvlan \
  --subnet=192.168.214.0/24 \
  --subnet=10.1.214.0/24 \
  -o ipvlan_mode=l3 ipnet210
```

Lancement de conteneurs sur 2 sous-réseaux différents :

```bash
docker run --net=ipnet210 --ip=192.168.214.10 -itd alpine /bin/sh
docker run --net=ipnet210 --ip=10.1.214.10 -itd alpine /bin/sh
```

Tests de connectivité L3 entre sous-réseaux :

```bash
docker run --net=ipnet210 --ip=192.168.214.9 -it --rm alpine ping -c 2 10.1.214.10
docker run --net=ipnet210 --ip=10.1.214.9 -it --rm alpine ping -c 2 192.168.214.10
```

👉 **Pas besoin de gateway** en L3, Docker l’ignore.\
Dans un conteneur :

```bash
ip route
default dev eth0
192.168.214.0/24 dev eth0  src 192.168.214.10
```

⚠️ Pour accéder aux conteneurs depuis l’extérieur → il faut ajouter des **routes statiques** sur le réseau ou les autres hôtes.

***

### 🌍 Exemple : IPvlan L2 + IPv6 only

Création réseau IPv6 pur :

```bash
docker network create -d ipvlan \
  --ipv6 --subnet=2001:db8:abc2::/64 --gateway=2001:db8:abc2::22 \
  -o parent=eth0.139 v6ipvlan139
```

Lancement et affichage de l’interface IPv6 :

```bash
ip -6 route
2001:db8:abc2::/64 dev eth0
default via 2001:db8:abc2::22 dev eth0
```

Puis un deuxième conteneur :

```bash
ping6 2001:db8:abc2::1
```

✅ Les conteneurs échangent en IPv6 pur.

***

### 🌍 Exemple : Réseau **dual-stack IPv4 + IPv6** (mode L2)

Création réseau VLAN 140 avec 2 sous-réseaux IPv4 et 1 IPv6 :

```bash
docker network create -d ipvlan \
  --subnet=192.168.140.0/24 --subnet=192.168.142.0/24 \
  --gateway=192.168.140.1 --gateway=192.168.142.1 \
  --subnet=2001:db8:abc9::/64 --gateway=2001:db8:abc9::22 \
  -o parent=eth0.140 \
  -o ipvlan_mode=l2 ipvlan140
```

Un conteneur peut recevoir une IP v4 et une IP v6.\
Exemple :

```bash
docker run --net=ipvlan140 --ip6=2001:db8:abc2::51 -it --rm alpine /bin/sh
```

***

### 🌍 Exemple : Réseau **dual-stack IPv4 + IPv6** (mode L3)

Création réseau IPvlan L3 avec VLAN ID 118 :

```bash
docker network create -d ipvlan \
  --subnet=192.168.110.0/24 \
  --subnet=192.168.112.0/24 \
  --subnet=2001:db8:abc6::/64 \
  -o parent=eth0 \
  -o ipvlan_mode=l3 ipnet110
```

Lancement de plusieurs conteneurs (IPv4, IPv6, ou les deux) :

```bash
docker run --net=ipnet110 -it --rm alpine /bin/sh
docker run --net=ipnet110 --ip6=2001:db8:abc6::10 -it --rm alpine /bin/sh
docker run --net=ipnet110 --ip=192.168.112.30 -it --rm alpine /bin/sh
docker run --net=ipnet110 --ip6=2001:db8:abc6::50 --ip=192.168.112.50 -it --rm alpine /bin/sh
```

👉 En L3, la route par défaut est :

```bash
ip route
default dev eth0
```

***

### 🌍 Exemple : VLAN créé **manuellement**

Création d’une sous-interface VLAN 40 :

```bash
ip link add link eth0 name eth0.40 type vlan id 40
ip link set eth0.40 up
```

Création du réseau IPvlan sur ce VLAN :

```bash
docker network create -d ipvlan \
  --subnet=192.168.40.0/24 \
  --gateway=192.168.40.1 \
  -o parent=eth0.40 ipvlan40
```

Lancer deux conteneurs :

```bash
docker run --net=ipvlan40 -it --name ivlan_test5 --rm alpine /bin/sh
docker run --net=ipvlan40 -it --name ivlan_test6 --rm alpine /bin/sh
```

⚠️ Si tu crées une interface manuellement (`foo` au lieu de `eth0.40`), Docker ne la supprime pas quand tu fais `docker network rm`.\
Nettoyage :

```bash
ip link del foo
```

***

### ✅ Résumé

* **IPvlan L3** supprime broadcasts/multicasts → très adapté au **scale massif**.
* **Chaque conteneur = nouveau sous-réseau routé**.
* Nécessite **routes statiques** ou un routage distribué (BGP, etc.) pour être joignable de l’extérieur.
* **Dual-stack IPv4/IPv6** supporté avec même niveau de fonctionnalités.
* **VLANs** peuvent être gérés par Docker ou créés manuellement pour plus de contrôle.
