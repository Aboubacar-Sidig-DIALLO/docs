---
description: ✅ Prérequis  Nécessite Docker Compose 2.20.3 ou version ultérieure.
---

# 📦 Utiliser include dans Docker Compose

### 📌 Qu’est-ce que `include` ?

L’instruction **`include`** vous permet **d’incorporer directement un fichier `compose.yaml` séparé** dans votre fichier `compose.yaml` principal.

👉 Objectifs :

* **Modulariser** des applications complexes en sous-fichiers Compose.
* Rendre les configurations plus **simples** et plus **explicites**.
* Refléter l’**organisation des équipes** dans la structure du fichier (chaque équipe peut gérer son propre sous-fichier Compose).
* Résoudre le problème des **chemins relatifs** que posent `extends` et `merge`.

***

### ⚙️ Fonctionnement

* Chaque fichier listé dans **`include`** est chargé comme un **modèle d’application Compose indépendant**, avec son propre **répertoire projet** pour résoudre les chemins relatifs.
* Une fois chargé, **toutes les ressources** (services, volumes, réseaux, etc.) de ce fichier sont **copiées** dans l’application Compose courante.
* **Récursivité** : si un fichier inclus contient lui-même un `include`, les fichiers mentionnés sont également chargés.

***

### 🖥️ Exemple simple

**compose.yaml** :

```yaml
include:
  - my-compose-include.yaml  # contient la définition de serviceB

services:
  serviceA:
    build: .
    depends_on:
      - serviceB  # utilisable comme s’il était défini ici
```

**my-compose-include.yaml** :

* Gère `serviceB` avec ses propres détails (réplicas, UI web, réseaux isolés, volumes persistants, etc.).
* L’application principale qui dépend de `serviceB` n’a pas besoin de connaître ces détails internes.
* Elle consomme ce fichier comme un **bloc de construction réutilisable**.

👉 Avantages :

* L’équipe qui gère `serviceB` peut **faire évoluer** son service (ex. ajouter une base de données) sans impacter les équipes dépendantes.
* Les équipes consommatrices n’ont pas besoin d’ajouter des options `-f` supplémentaires à chaque commande `docker compose`.

***

### 🌍 Exemple avec un fichier distant

Vous pouvez également référencer des fichiers Compose depuis des **sources distantes**, comme :

* un **artefact OCI**,
* ou un **dépôt Git**.

```yaml
include:
  - oci://docker.io/username/my-compose-app:latest  # fichier Compose stocké comme artefact OCI

services:
  serviceA:
    build: .
    depends_on:
      - serviceB
```

👉 Ici, **`serviceB`** est défini dans un fichier Compose stocké sur **Docker Hub**.

***

### 🎯 En résumé

* `include` est plus moderne et puissant que `extends`.
* Il permet de **structurer les projets complexes** en modules indépendants, facilement réutilisables.
* Il facilite la **collaboration entre équipes** et réduit la complexité de gestion dans les projets multi-services.

## 📝 Overrides avec `include` dans Docker Compose

### ⚠️ Problème de base

* Quand tu utilises **`include`**, si une ressource (ex. service, volume, réseau) définie dans ton `compose.yaml` **entre en conflit** avec une ressource du fichier inclus → **Compose renvoie une erreur**.
* C’est un mécanisme de sécurité pour éviter d’écraser par erreur la configuration écrite par l’auteur du fichier inclus.

***

### ✅ Solution 1 : Override local dédié par include

Tu peux personnaliser un fichier inclus en ajoutant un **fichier d’override dédié** dans la directive `include` :

```yaml
include:
  - path: 
      - third-party/compose.yaml
      - override.yaml  # override spécifique au modèle inclus
```

👉 Ici :

* `third-party/compose.yaml` est un fichier externe (par ex. fourni par une autre équipe ou un package).
* `override.yaml` est ton fichier local qui modifie certains paramètres **spécifiquement pour ce modèle inclus**.

⚠️ Limite :

* Tu dois maintenir **un fichier override par include**.
* Pour un gros projet avec beaucoup de `include`, ça peut vite devenir lourd.

***

### ✅ Solution 2 : `compose.override.yaml` global

Autre approche → utiliser un **fichier global d’override** : `compose.override.yaml`.

Exemple :

#### `compose.yaml` principal

```yaml
include:
  - team-1/compose.yaml   # déclare service-1
  - team-2/compose.yaml   # déclare service-2
```

#### `compose.override.yaml` global

```yaml
services:
  service-1:
    # Override du service-1 pour exposer un port de debug
    ports:
      - 2345:2345

  service-2:
    # Override du service-2 pour monter un dossier local de test
    volumes:
      - ./data:/data
```

👉 Ici :

* `compose.yaml` importe les définitions des équipes 1 et 2.
* `compose.override.yaml` les **modifie globalement après fusion**, sans conflit.

***

### 🎯 Résultat

Cette combinaison te permet de :

* **Réutiliser des composants tiers** (ex. stacks d’équipe, Compose externes).
* **Adapter la config à tes besoins** sans toucher aux fichiers tiers.
* Garder une logique claire :
  * **include** → pour importer des blocs de services.
  * **override** → pour personnaliser ton environnement (debug, volumes locaux, etc.).
