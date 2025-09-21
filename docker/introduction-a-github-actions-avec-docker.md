# ⚡ Introduction à GitHub Actions avec Docker

Ce guide propose une **introduction à la création de pipelines CI** (Intégration Continue) en utilisant **Docker** et **GitHub Actions**.

👉 Vous allez apprendre à utiliser les **Actions GitHub officielles de Docker** pour :

* construire votre application sous forme d’**image Docker**,
* puis la **pousser sur Docker Hub**.

À la fin du guide, vous disposerez d’une configuration **GitHub Actions simple et fonctionnelle** pour vos builds Docker.

* Vous pouvez l’utiliser telle quelle ✅,
* ou bien l’**étendre** pour l’adapter à vos besoins.

***

### ✅ Prérequis

Avant de suivre le guide, assurez-vous d’avoir :

* 🔑 **Un compte Docker**.
* 📄 Une **familiarité avec les Dockerfiles**.

⚠️ Ce guide suppose que vous connaissez les notions de base de Docker, mais fournit des explications spécifiques sur son usage avec **GitHub Actions**.

***

### 📦 Obtenir l’application d’exemple

Ce guide est **agnostique du projet** → il part du principe que vous avez déjà une application avec un **Dockerfile**.

👉 Si vous n’avez pas encore de projet :

* Vous pouvez utiliser une **application d’exemple** fournie (incluant un Dockerfile).
* Ou bien :
  * utiliser votre propre projet GitHub,
  * créer un nouveau dépôt à partir d’un template.

***

#### 📝 Exemple de `Dockerfile`

Voici un exemple de **Dockerfile multi-étapes** pour une application **Node.js** :

```dockerfile
#syntax=docker/dockerfile:1

# Étape 1 : builder → installe les dépendances et build l’app
FROM node:lts-alpine AS builder
WORKDIR /src
RUN --mount=src=package.json,target=package.json \
    --mount=src=package-lock.json,target=package-lock.json \
    --mount=type=cache,target=/root/.npm \
    npm ci
COPY . .
RUN --mount=type=cache,target=/root/.npm \
    npm run build

# Étape 2 : release → crée l’image finale prête à l’exécution
FROM node:lts-alpine AS release
WORKDIR /app
COPY --from=builder /src/build .
EXPOSE 3000
CMD ["node", "."]
```

👉 Explication :

* La première étape (`builder`) installe les dépendances et construit l’application.
* La deuxième (`release`) contient uniquement le build final → ce qui rend l’image plus **légère et optimisée**.

***

### 🔐 Configurer votre dépôt GitHub

Le workflow de ce guide pousse l’image que vous construisez vers **Docker Hub**.\
➡️ Pour cela, vous devez vous **authentifier** avec vos identifiants Docker (**nom d’utilisateur + access token**).

#### Étapes :

1. Créez un **Docker access token** (voir doc officielle : _Create and manage access tokens_).
2. Dans votre dépôt GitHub, ajoutez vos identifiants comme secrets/variables :
   * ⚙️ Ouvrez **Settings** du dépôt.
   * Allez dans **Security > Secrets and variables > Actions**.
   * Sous **Secrets**, créez un nouveau secret nommé **`DOCKER_PASSWORD`**, contenant votre **access token**.
   * Sous **Variables**, créez une variable **`DOCKER_USERNAME`**, contenant votre **nom d’utilisateur Docker Hub**.

✅ Ainsi, vos credentials seront disponibles dans vos workflows GitHub Actions de manière sécurisée.

## ⚡ Mise en place du workflow GitHub Actions

Un **workflow GitHub Actions** définit une série d’étapes automatisées (ex. build, tests, déploiement) qui s’exécutent en réponse à des **déclencheurs** (push, pull request, etc.).

👉 Ici, l’objectif est :

* automatiser la **construction d’images Docker**,
* exécuter des tests pour vérifier que l’application conteneurisée fonctionne bien,
* puis la **pousser sur Docker Hub** si tout est OK ✅.

***

### 📂 Créer le fichier de workflow

Crée un fichier nommé **`docker-ci.yml`** dans le dossier **`.github/workflows/`** de ton dépôt.

Commence par la configuration de base :

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - main
  pull_request:
```

👉 Explication :

* Le workflow s’exécute :
  * sur chaque **push** vers la branche **main**
  * et sur chaque **pull request**.
* Cela permet de vérifier que l’image Docker se construit correctement **avant de fusionner une PR**.

***

### 🏷️ Extraire les métadonnées (tags & annotations)

Première étape → utiliser l’action officielle **`docker/metadata-action`** pour générer des **tags et annotations** à appliquer à ton image Docker.

Ajoute ceci dans ton fichier workflow :

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Extract Docker image metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ vars.DOCKER_USERNAME }}/my-image
```

👉 Détails :

* **Checkout** : clone le dépôt Git dans l’environnement du workflow.
* **Extract metadata** :
  * récupère des infos Git (branche, commit SHA, etc.),
  * génère des **tags** et **annotations** pour l’image (ex. `latest`, `main-<commit-sha>`).

***

### 🔐 Authentification au registre Docker

Avant de pousser l’image, il faut **s’authentifier** auprès de Docker Hub avec tes identifiants.

Ajoute ce step :

```yaml
      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
```

👉 Explication :

* On utilise les identifiants configurés dans **Settings > Secrets and variables**.
* **`DOCKER_USERNAME`** est une variable publique,
* **`DOCKER_PASSWORD`** est un secret contenant le **token d’accès**.

***

### 🏗️ Construire et pousser l’image

Enfin, construis et pousse l’image sur le registre :

```yaml
      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          annotations: ${{ steps.meta.outputs.annotations }}
```

👉 Ce qu’il faut retenir :

* **`push: ${{ github.event_name != 'pull_request' }}`**
  * L’image est **seulement poussée** quand l’événement n’est pas une PR.
  * Ainsi :
    * Pour une **PR** → l’image est construite et testée ✅, mais **pas publiée**.
    * Pour un **push sur main** → l’image est construite **et poussée** 🚀.
* **`tags`** et **`annotations`** utilisent les valeurs générées par `docker/metadata-action` → cohérence garantie pour tes images.

***

### 🧩 Workflow final (récapitulatif)

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Extract Docker image metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ vars.DOCKER_USERNAME }}/my-image

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          annotations: ${{ steps.meta.outputs.annotations }}
```

***

👉 Avec ça, tu as un pipeline **CI/CD minimaliste et efficace** pour :

* construire tes images,
* tester leur validité,
* et les pousser automatiquement sur Docker Hub 🐳.

## 🛡️ Attestations (SBOM & provenance)

Les **attestations SBOM** (_Software Bill of Materials_) et **provenance** améliorent :

* 🔒 la **sécurité**,
* 🔍 la **traçabilité**,

en garantissant que tes images respectent les exigences modernes des **chaînes d’approvisionnement logicielles** (_software supply chain_).

👉 Avec un peu de configuration supplémentaire, tu peux demander à **`docker/build-push-action`** de générer :

* un **SBOM** (liste complète des composants logiciels inclus dans l’image),
* une **attestation de provenance** (informations sur l’origine et la construction de l’image).

Ces métadonnées sont générées **au moment du build**.

***

### ⚙️ Modifications nécessaires

Pour activer SBOM & provenance :

1️⃣ **Avant l’étape du build**, ajoute une étape qui configure **Docker Buildx** (client Docker avancé pour les builds).

* Buildx offre des fonctionnalités supplémentaires que le client par défaut ne supporte pas.

2️⃣ Mets à jour l’étape **Build and push Docker image** pour inclure les options `provenance: true` et `sbom: true`.

***

#### 📝 Exemple mis à jour :

```yaml
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          annotations: ${{ steps.meta.outputs.annotations }}
          provenance: true
          sbom: true
```

👉 Résultat : ton image Docker sera accompagnée de métadonnées **fiables et vérifiables**, ce qui facilite les audits et la conformité.

***

## ✅ Workflow complet (récapitulatif final)

Voici le workflow final intégrant toutes les bonnes pratiques vues jusqu’ici 👇

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Extract Docker image metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ vars.DOCKER_USERNAME }}/my-image

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          annotations: ${{ steps.meta.outputs.annotations }}
          provenance: true
          sbom: true
```

***

### 📌 Conclusion

🎯 Ce workflow applique les **meilleures pratiques** pour :

* construire des images Docker optimisées,
* automatiser leur publication sur Docker Hub,
* assurer leur **sécurité et traçabilité** via SBOM & provenance.

⚡ Tu peux l’utiliser **tel quel** ou l’étendre selon tes besoins (par ex. : builds multi-plateformes, tests automatisés, cache de build pour plus de rapidité…).
