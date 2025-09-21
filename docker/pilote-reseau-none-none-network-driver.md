# 🌐 Pilote réseau none (None network driver)

### 🔹 Définition

Si tu veux **isoler complètement la pile réseau d’un conteneur**, tu peux utiliser l’option :

```bash
--network none
```

lors du démarrage du conteneur.

👉 Dans ce mode, **aucune interface réseau** n’est créée dans le conteneur, **sauf la boucle locale (loopback)** `lo`.

***

### 🖥️ Exemple : vérifier les interfaces réseau

Lancer un conteneur Alpine avec le pilote réseau **none** et afficher les interfaces :

```bash
docker run --rm --network none alpine:latest ip link show
```

Sortie :

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

👉 On voit uniquement **l’interface loopback (`lo`)**.\
Aucune interface Ethernet (`eth0`) n’existe dans le conteneur.

***

### 🌍 IPv6 non configuré

Avec le pilote `none`, **aucune adresse IPv6** de boucle locale n’est créée.

Exemple :

```bash
docker run --rm --network none --name no-net-alpine alpine:latest ip addr show
```

Sortie :

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
```

👉 Seule l’adresse IPv4 `127.0.0.1` est disponible.\
Il n’y a **pas d’adresse IPv6 (::1)**.

***

### ✅ Résumé

* Le pilote réseau **none** = **conteneur totalement isolé** du réseau.
* Seule l’interface **loopback (`lo`)** existe.
* Pas de **connectivité externe** (pas d’IPv4, pas d’IPv6).
* Utile pour :
  * des **tests unitaires** où aucun accès réseau ne doit exister.
  * des conteneurs servant uniquement à des calculs locaux.
  * des environnements hautement sécurisés où l’accès réseau doit être interdit.
