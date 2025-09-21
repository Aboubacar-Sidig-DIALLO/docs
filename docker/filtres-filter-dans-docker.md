# 🔎 Filtres (--filter) dans Docker

De nombreuses commandes Docker supportent l’option `--filter` afin de **restreindre les résultats** affichés.\
👉 Cela permet d’afficher uniquement les conteneurs, images, volumes, réseaux, etc. qui correspondent à certains critères.

***

### 📌 Exemple avec `docker ps`

Lister uniquement les conteneurs **qui utilisent l’image nginx** :

```bash
docker ps --filter ancestor=nginx
```

Lister uniquement les conteneurs **en cours d’exécution depuis plus de 10 minutes** :

```bash
docker ps --filter "status=running" --filter "since=10m"
```

Lister un conteneur par son **nom** :

```bash
docker ps --filter "name=webapp"
```

***

### 📌 Exemple avec `docker images`

Lister uniquement les images **dangling** (orphelines, sans tag) :

```bash
docker images --filter dangling=true
```

Lister uniquement les images créées avant une certaine image :

```bash
docker images --filter before=nginx:1.25
```

***

### 📌 Exemple avec `docker network`

Lister uniquement les réseaux de type **bridge** :

```bash
docker network ls --filter driver=bridge
```

***

### 📌 Exemple avec `docker volume`

Lister uniquement les volumes non utilisés :

```bash
docker volume ls --filter dangling=true
```

***

### 🎯 Points importants

* On peut **cumuler plusieurs `--filter`** pour affiner la recherche.
* La syntaxe est toujours :

```bash
docker <commande> --filter "clé=valeur"
```

* Les clés disponibles dépendent de la commande (`status`, `ancestor`, `dangling`, `label`, `name`, etc.).

## 🔎 Utilisation des filtres (`--filter`)

#### ✅ Syntaxe générale

```bash
docker <COMMAND> --filter "KEY=VALUE"
docker <COMMAND> --filter "KEY!=VALUE"   # négation possible
```

* **KEY** = champ à filtrer (ex: `reference`, `status`, `dangling`, `label`, …).
* **VALUE** = valeur ou motif recherché.
* **OPERATOR** = `=` (égal) ou `!=` (différent).

***

### 📌 Exemple avec `docker images`

Lister toutes les images **alpine** :

```bash
docker images --filter "reference=alpine"
```

👉 Résultat = toutes les images dont le nom est `alpine`.

Lister toutes les images **qui ne sont pas alpine** :

```bash
docker images --filter "reference!=alpine"
```

***

### 📌 Exemple avec `docker ps`

Lister uniquement les conteneurs **en cours d’exécution** :

```bash
docker ps --filter "status=running"
```

Lister les conteneurs **qui ne sont pas créés à partir de nginx** :

```bash
docker ps --filter "ancestor!=nginx"
```

***

### 📌 Exemple avec `docker network`

Lister les réseaux **bridge uniquement** :

```bash
docker network ls --filter "driver=bridge"
```

***

### 📌 Exemple avec `docker volume`

Lister les volumes **orphelins (dangling)** :

```bash
docker volume ls --filter "dangling=true"
```

***

### 🎯 Points importants

* **Chaque commande Docker a ses propres clés de filtre** (`docker ps` ≠ `docker images`).
* Certains filtres exigent une correspondance exacte (`dangling=true`), d’autres acceptent des **motifs partiels** (`reference=alpine` match aussi `alpine:3.21`).
* Tu peux **enchaîner plusieurs filtres** :

```bash
docker ps --filter "status=running" --filter "ancestor=nginx"
```

## 🔎 Combinaison des filtres (`--filter`)

#### ✅ Syntaxe

Tu peux passer **plusieurs `--filter`** dans une même commande :

```bash
docker <COMMAND> --filter "KEY=VALUE1" --filter "KEY=VALUE2"
```

👉 Cela correspond à un **OU logique (OR)**.\
Docker retourne **tous les éléments qui satisfont au moins un des filtres**.

***

### 📌 Exemple avec `docker images`

Lister toutes les images **alpine:latest** **OU** toutes les images **busybox** :

```bash
docker images --filter "reference=alpine:latest" --filter "reference=busybox"
```

➡️ Résultat :

* `alpine:latest`
* toutes les variantes de `busybox` (`uclibc`, `glibc`, …)

***

### 📌 Exemple avec `docker ps`

Lister les conteneurs **en cours d’exécution** **OU** créés à partir de `nginx` :

```bash
docker ps --filter "status=running" --filter "ancestor=nginx"
```

➡️ Tu obtiens :

* les conteneurs `status=running`
* et ceux `ancestor=nginx` (même si arrêtés).

***

### 📌 Exemple avec `docker volume`

Lister les volumes **orphelins** **OU** nommés `db-data` :

```bash
docker volume ls --filter "dangling=true" --filter "name=db-data"
```

***

### ⚠️ Attention

* Les filtres **ne s’additionnent pas en AND** (ce n’est pas une intersection).
* Si tu veux une logique **ET**, il faut souvent **filtrer côté shell** avec `grep`, `jq` ou `awk`.\
  Exemple :

```bash
docker ps --filter "status=running" | grep nginx
```

## 🚫 Filtres négatifs (`label!=...`)

#### ✅ Exemple simple

Si tu veux **supprimer tous les conteneurs qui n’ont pas le label `foo`** :

```bash
docker container prune --filter "label!=foo"
```

👉 Résultat :

* Les conteneurs **avec `label=foo`** sont conservés.
* Tous les autres (sans ce label) sont supprimés.

***

## 🧩 Plusieurs filtres négatifs

⚠️ **Attention :** quand tu combines plusieurs filtres `!=`, ils s’additionnent en **ET logique (AND)**.

Exemple :

```bash
docker container prune --filter "label!=foo" --filter "label!=bar"
```

👉 Cela veut dire :

* Garde uniquement les conteneurs qui ont **à la fois** `label=foo` **ET** `label=bar`.
* Tous les autres (même ceux qui ont seulement `foo` ou seulement `bar`) seront supprimés.

***

## 📊 Résumé logique

| Filtres appliqués           | Conteneur `label=foo` | Conteneur `label=bar` | Conteneur `label=foo,bar` | Conteneur sans label |
| --------------------------- | --------------------- | --------------------- | ------------------------- | -------------------- |
| `label!=foo`                | ✅ Conservé            | ❌ Supprimé            | ✅ Conservé                | ❌ Supprimé           |
| `label!=foo` + `label!=bar` | ❌ Supprimé            | ❌ Supprimé            | ✅ Conservé                | ❌ Supprimé           |

***

## 🔎 Cas d’usage pratique

*   **Exclure un seul label** :

    ```bash
    docker ps --filter "label!=test"
    ```

    (affiche tous les conteneurs sauf ceux avec `label=test`)
*   **Exclure tout sauf une combinaison de labels précise** :

    ```bash
    docker ps --filter "label!=env=prod" --filter "label!=team=backend"
    ```

    👉 Cela ne garde que les conteneurs **qui ont les deux labels en même temps**.

## 📖 Commandes supportant `--filter`

Voici la liste (avec ce que ça permet de filtrer) :

#### 🔹 **Conteneurs**

* `docker container ls` → filtrer la liste des conteneurs (ex: par statut, label, nom…)
* `docker container prune` → supprimer les conteneurs arrêtés selon un filtre (ex: labels ≠ prod)

#### 🔹 **Images**

* `docker image ls` → filtrer les images (ex: par référence, dangling, avant/depuis une date…)
* `docker image prune` → nettoyer les images inutilisées avec conditions

#### 🔹 **Volumes**

* `docker volume ls` → filtrer les volumes (par label, dangling…)
* `docker volume prune` → supprimer les volumes inutilisés selon filtres

#### 🔹 **Réseaux**

* `docker network ls` → filtrer les réseaux (par type, label…)
* `docker network prune` → supprimer des réseaux inutilisés avec conditions

#### 🔹 **Swarm (clusters)**

* `docker node ls` → filtrer les nœuds du cluster
* `docker node ps` → filtrer les tâches en cours d’un nœud
* `docker service ls` → filtrer les services Swarm
* `docker service ps` → filtrer les tâches liées à un service
* `docker stack ps` → filtrer les tâches d’une stack

#### 🔹 **Secrets et configs**

* `docker secret ls` → filtrer la liste des secrets
* `docker config ls` → filtrer les configs

#### 🔹 **Plugins**

* `docker plugin ls` → filtrer les plugins

#### 🔹 **Système**

* `docker system prune` → nettoyage global avec conditions
* `docker search` → rechercher des images sur Docker Hub avec des filtres (ex: étoiles, officiel, automatisé)

***

## ✅ Exemple pratique

Lister uniquement les conteneurs arrêtés depuis plus de 24h :

```bash
docker container ls -a --filter "status=exited" --filter "until=24h"
```

Lister uniquement les images _dangling_ (orphelines) :

```bash
docker image ls --filter dangling=true
```

Pruner uniquement les volumes avec un label précis :

```bash
docker volume prune --filter "label!=keep"
```

***

👉 En résumé : **presque toutes les commandes qui listent ou nettoient quelque chose supportent `--filter`** pour cibler plus précisément.
