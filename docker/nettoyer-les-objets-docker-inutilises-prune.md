# 🧹 Nettoyer les objets Docker inutilisés (Prune)

Docker adopte une approche **conservatrice** concernant le nettoyage automatique :

* Les **images**, **conteneurs**, **volumes** et **réseaux** inutilisés ne sont **pas supprimés automatiquement**.
* C’est à toi de lancer explicitement les commandes de nettoyage (`prune`).

👉 Objectif : éviter d’accumuler des objets non utilisés qui consomment de l’espace disque.

***

### 🔹 Commandes prune spécifiques

#### 🗑️ Conteneurs arrêtés

Supprime tous les conteneurs **arrêtés** :

```bash
docker container prune
```

#### 🗑️ Réseaux non utilisés

Supprime les réseaux **non utilisés par au moins un conteneur** :

```bash
docker network prune
```

#### 🗑️ Images inutilisées

Supprime les images **dangling** (sans tag ou non référencées) :

```bash
docker image prune
```

➡️ Pour supprimer **toutes les images non utilisées** (pas seulement dangling) :

```bash
docker image prune -a
```

#### 🗑️ Volumes orphelins

Supprime les volumes **non utilisés par un conteneur** :

```bash
docker volume prune
```

***

### 🔹 Nettoyage global

Pour nettoyer **en une seule commande** les conteneurs arrêtés, réseaux inutilisés, images inutilisées et cache de build :

```bash
docker system prune
```

➡️ Pour inclure aussi les **volumes** :

```bash
docker system prune --volumes
```

***

### ⚠️ Attention

* Une fois supprimés, les objets **ne sont pas récupérables**.
* Utilise l’option `-f` ou `--force` pour éviter la confirmation interactive :

```bash
docker system prune -af --volumes
```

## 🗑️ `docker image prune` — Nettoyer les images inutilisées

La commande **`docker image prune`** permet de supprimer les **images Docker inutilisées**.

***

### 🔹 1. Supprimer uniquement les images _dangling_

Une image **dangling** = une image **sans tag** et **non référencée par un conteneur**.

```bash
docker image prune
```

Exemple d’avertissement :

```
WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N] y
```

***

### 🔹 2. Supprimer **toutes les images non utilisées**

Avec l’option `-a`, Docker supprime **toutes les images qui ne sont utilisées par aucun conteneur actif ou arrêté**.

```bash
docker image prune -a
```

⚠️ Attention, cela peut supprimer beaucoup d’images, y compris celles téléchargées récemment mais non encore utilisées.

***

### 🔹 3. Forcer la suppression (sans confirmation)

Pour éviter la demande de confirmation, ajoute `-f` ou `--force` :

```bash
docker image prune -af
```

***

### 🔹 4. Filtrer les images à supprimer

Tu peux restreindre la purge avec `--filter`.

Exemple : supprimer uniquement les images créées il y a plus de 24h :

```bash
docker image prune -a --filter "until=24h"
```

➡️ Autres filtres disponibles (comme sur `docker ps` ou `docker images`) : `label`, `before`, `since`, etc.

***

📌 En résumé :

* `docker image prune` ➝ supprime seulement les **dangling images**.
* `docker image prune -a` ➝ supprime toutes les images inutilisées.
* `--filter` ➝ contrôle fin (par date, labels, etc.).
* `-f` ➝ supprime sans confirmation.

## 🗑️ `docker container prune` — Nettoyer les conteneurs arrêtés

Quand tu arrêtes un conteneur, il **n’est pas supprimé automatiquement** (sauf si tu l’as lancé avec `--rm`).\
Résultat : avec `docker ps -a`, tu peux voir une longue liste de conteneurs (actifs + arrêtés), qui consomment quand même de l’espace disque.

👉 Pour supprimer les conteneurs arrêtés :

***

### 🔹 1. Supprimer tous les conteneurs arrêtés

```bash
docker container prune
```

Exemple de confirmation :

```
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
```

***

### 🔹 2. Supprimer sans confirmation

Ajoute `-f` ou `--force` pour **forcer la suppression** sans question :

```bash
docker container prune -f
```

***

### 🔹 3. Filtrer les conteneurs à supprimer

Avec `--filter`, tu peux cibler seulement certains conteneurs.

Exemple : supprimer uniquement ceux arrêtés **il y a plus de 24 heures** :

```bash
docker container prune --filter "until=24h"
```

***

📌 **Résumé :**

* `docker container prune` → supprime tous les conteneurs arrêtés.
* `-f` → pas de confirmation.
* `--filter` → supprimer selon une condition (`until`, `label`, etc.).

## 🗑️ `docker volume prune` — Nettoyer les volumes inutilisés

Les **volumes** servent à stocker des données persistantes et peuvent être utilisés par **un ou plusieurs conteneurs**.\
👉 Docker **ne les supprime jamais automatiquement**, car cela pourrait détruire des données importantes.

***

### 🔹 1. Supprimer tous les volumes inutilisés

Un volume est considéré comme inutilisé s’il n’est **associé à aucun conteneur**.

```bash
docker volume prune
```

Exemple de confirmation :

```
WARNING! This will remove all volumes not used by at least one container.
Are you sure you want to continue? [y/N] y
```

***

### 🔹 2. Supprimer sans confirmation

Ajoute `-f` ou `--force` pour ne pas avoir la demande de confirmation :

```bash
docker volume prune -f
```

***

### 🔹 3. Supprimer avec un filtre

Tu peux limiter la suppression avec `--filter`.

Exemple : supprimer tous les volumes **sauf ceux qui portent le label `keep`** :

```bash
docker volume prune --filter "label!=keep"
```

***

📌 **Résumé :**

* `docker volume prune` → supprime uniquement les volumes **non utilisés**.
* `-f` → pas de confirmation.
* `--filter` → cibler les volumes à garder ou supprimer.

## 🌐 `docker network prune` — Nettoyer les réseaux inutilisés

Les **réseaux Docker** ne consomment pas beaucoup d’espace disque, mais ils créent :

* des règles **iptables**,
* des **interfaces bridge**,
* et des **entrées de routage**.

Ces éléments peuvent encombrer ton système si tu ne les nettoies pas régulièrement.

***

### 🔹 1. Supprimer les réseaux inutilisés

Un réseau est considéré comme inutilisé s’il n’est **attaché à aucun conteneur**.

```bash
docker network prune
```

Exemple :

```
WARNING! This will remove all networks not used by at least one container.
Are you sure you want to continue? [y/N] y
```

***

### 🔹 2. Supprimer sans confirmation

Ajoute `-f` ou `--force` pour éviter la demande de confirmation :

```bash
docker network prune -f
```

***

### 🔹 3. Supprimer avec un filtre

Comme pour les autres objets (`container`, `image`, `volume`), tu peux limiter le nettoyage avec `--filter`.

Exemple : supprimer uniquement les réseaux créés il y a plus de 24h :

```bash
docker network prune --filter "until=24h"
```

***

📌 **Résumé :**

* `docker network prune` → supprime les réseaux inutilisés.
* `-f` → pas de confirmation.
* `--filter` → cibler selon l’âge, labels, etc.

## 🧹 `docker system prune` — Tout nettoyer d’un coup

La commande **`docker system prune`** est un **raccourci global** pour nettoyer plusieurs types d’objets Docker en une seule commande.

***

### 🔹 1. Nettoyage standard (sans volumes)

```bash
docker system prune
```

⚠️ Cela supprime :

* ✅ tous les **conteneurs arrêtés**,
* ✅ tous les **réseaux non utilisés** par au moins un conteneur,
* ✅ toutes les **images orphelines** (_dangling images_),
* ✅ tout le **cache de build inutilisé**.

Exemple :

```
WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all dangling images
        - unused build cache

Are you sure you want to continue? [y/N] y
```

***

### 🔹 2. Nettoyage avec volumes

Par défaut, les **volumes** ne sont pas supprimés (pour éviter la perte de données).\
👉 Ajoute `--volumes` si tu veux aussi les supprimer :

```bash
docker system prune --volumes
```

Cela supprime en plus :

* ✅ tous les **volumes inutilisés**.

Exemple :

```
WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all volumes not used by at least one container
        - all dangling images
        - all build cache
```

***

### 🔹 3. Supprimer sans confirmation

Ajoute `-f` ou `--force` :

```bash
docker system prune -f
```

***

### 🔹 4. Supprimer avec filtre

Exemple : supprimer uniquement les objets inutilisés depuis plus de 24h :

```bash
docker system prune --filter "until=24h"
```

***

📌 **Résumé :**

* `docker system prune` = tout nettoyer sauf les volumes.
* `docker system prune --volumes` = inclut les volumes.
* `-f` = pas de confirmation.
* `--filter` = cibler selon l’âge, labels, etc.
