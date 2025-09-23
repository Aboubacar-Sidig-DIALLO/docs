# 🔹 Variables de build dans Docker

Lorsqu’on construit une image avec `docker build`, on peut **passer des informations dynamiques** au moment du build. Cela rend les builds **flexibles et paramétrables**.\
Pour ça, Docker propose deux types de variables principales :

1. **`ARG` (Build arguments)** → utilisées uniquement **pendant la construction** de l’image.
2. **`ENV` (Variables d’environnement)** → utilisées **pendant la construction ET à l’exécution** du conteneur.

***

### 1️⃣ `ARG` → Variables pour le **build uniquement**

* Définies dans le **Dockerfile** avec `ARG`.
* Peuvent être passées au moment du build avec `--build-arg`.
* **Visibles uniquement pendant le build** → ne sont pas disponibles dans le conteneur final, sauf si on les réinjecte avec `ENV`.

Exemple :

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:22.04

# Déclare une variable de build
ARG APP_VERSION=1.0.0

# Utilise la variable pendant le build
RUN echo "Building version $APP_VERSION"
```

Commande :

```bash
docker build --build-arg APP_VERSION=2.3.1 -t myapp:2.3.1 .
```

➡️ Ici, la variable `APP_VERSION` vaut `2.3.1` seulement **pendant la construction**.\
➡️ Si tu exécutes le conteneur, cette variable **n’existe plus**.

***

### 2️⃣ `ENV` → Variables pour le **runtime et le build**

* Définies dans le **Dockerfile** avec `ENV`.
* Disponibles **pendant la construction** ET **dans le conteneur final**.
* Peuvent être surchargées au moment du `docker run -e`.

Exemple :

```dockerfile
FROM python:3.11

# Déclare une variable d’environnement
ENV APP_ENV=production

# Utilise la variable dans le build
RUN echo "Building for $APP_ENV"

CMD ["python", "app.py"]
```

Commande :

```bash
docker run -e APP_ENV=development myapp
```

➡️ Dans ce cas, la variable `APP_ENV` sera visible **dans le conteneur** avec la valeur `development`.

***

### 3️⃣ Différences clés entre `ARG` et `ENV`

| Caractéristique          | `ARG` 🏗️ (Build uniquement)        | `ENV` 🌍 (Build + Runtime)                |
| ------------------------ | ----------------------------------- | ----------------------------------------- |
| Scope                    | Visible seulement pendant le build  | Visible pendant le build ET à l’exécution |
| Défaut                   | Peut avoir une valeur par défaut    | Peut avoir une valeur par défaut          |
| Override                 | `--build-arg` pendant le build      | `-e` ou `--env` pendant `docker run`      |
| Persistance dans l’image | ❌ Non (sauf si transformé en `ENV`) | ✅ Oui                                     |
| Sécurité (secrets)       | ❌ Inadapté                          | ❌ Inadapté                                |

***

### ⚠️ Sécurité : Attention aux secrets

Docker **ne recommande pas** de mettre des secrets (tokens, mots de passe, clés privées) dans `ARG` ou `ENV`, car :

* Ils apparaissent dans l’historique de build.
* Ils peuvent être retrouvés avec `docker history`.
* Ils restent stockés dans l’image finale.

👉 Pour les secrets sensibles → il faut utiliser les **`secret mounts`** ou les **`SSH mounts`** (fonctionnalités de BuildKit).

## 🔹 Similarités et différences entre ARG et ENV

#### ✅ Points communs

* **Déclarés dans le Dockerfile** (`ARG` ou `ENV`).
* **Peuvent être définis lors du build** avec des options (`--build-arg` pour ARG, valeur codée pour ENV).
* **Servent à paramétrer les builds** pour les rendre plus flexibles.

***

#### 🔸 **ARG (Arguments de build)**

* **Portée** : uniquement pendant la **construction de l’image**.
* **Utilité** : personnaliser les instructions du Dockerfile (ex. choisir la version d’une dépendance).
* **Disponibilité** : non accessibles dans le conteneur final (sauf si transférés dans un `ENV`).
* **Persistance** : peuvent apparaître dans les **métadonnées** de l’image (`docker history`).
* **Sécurité** : ⚠️ pas adaptés aux secrets, car visibles dans l’historique de build.
* **Exemple d’usage** : choisir la version de Node.js ou d’une dépendance.

```dockerfile
# Déclare un argument de build
ARG NODE_VERSION=20

FROM node:${NODE_VERSION}
RUN echo "Using Node.js ${NODE_VERSION}"
```

Commande :

```bash
docker build --build-arg NODE_VERSION=18 -t myapp:18 .
```

***

#### 🔸 **ENV (Variables d’environnement)**

* **Portée** : valables pendant le build **et** dans le conteneur final.
* **Utilité** : configurer l’environnement du build et définir des variables **par défaut pour le conteneur**.
* **Disponibilité** : toujours présentes dans le conteneur à l’exécution.
* **Override** : peut être surchargé au **runtime** avec `docker run -e`.
* **Exemple d’usage** : définir une configuration (`APP_ENV`, `DB_HOST`, etc.).

```dockerfile
FROM python:3.11

ENV APP_ENV=production
RUN echo "Building for $APP_ENV"

CMD ["python", "app.py"]
```

Commande :

```bash
docker run -e APP_ENV=development myapp
```

***

#### 🔀 **Combinaison ARG + ENV**

Une bonne pratique est d’utiliser `ARG` pour **injecter une valeur au moment du build** et la propager dans un `ENV` pour la rendre **persistante à l’exécution**.

```dockerfile
FROM alpine:latest

# Déclare un argument de build
ARG APP_VERSION=1.0.0

# Transforme en variable d’environnement persistante
ENV APP_VERSION=$APP_VERSION

CMD ["sh", "-c", "echo Running app version $APP_VERSION"]
```

Build :

```bash
docker build --build-arg APP_VERSION=2.3.1 -t myapp:2.3.1 .
```

Run :

```bash
docker run myapp:2.3.1
# → Running app version 2.3.1
```

***

#### ⚖️ Résumé rapide

| Aspect        | **ARG** 🏗️ (Build uniquement) | **ENV** 🌍 (Build + Runtime) |
| ------------- | ------------------------------ | ---------------------------- |
| Portée        | Construction seulement         | Construction + Exécution     |
| Override      | `--build-arg` au build         | `-e` au run                  |
| Persistance   | ❌ Non                          | ✅ Oui                        |
| Secrets       | ❌ Non recommandé               | ❌ Non recommandé             |
| Usage typique | Versions, options de build     | Config runtime par défaut    |

## 🛠️ Exemple d’utilisation de `ARG` dans un Dockerfile

#### 📌 Pourquoi utiliser `ARG` ?

* Définir des **versions de dépendances** (Node, Alpine, Python, etc.) sans modifier le Dockerfile à chaque fois.
* Rendre ton Dockerfile **réutilisable et maintenable**.
* Pouvoir **surcharger les valeurs au moment du build** avec `--build-arg`.
* Réutiliser la même valeur à plusieurs endroits (ex. une version unique d’Alpine utilisée pour plusieurs images).

***

#### 📄 Exemple pratique

```dockerfile
# syntax=docker/dockerfile:1

# Déclaration des arguments de build avec valeurs par défaut
ARG NODE_VERSION="20"
ARG ALPINE_VERSION="3.21"

# Étape de base : Node.js sur Alpine
FROM node:${NODE_VERSION}-alpine${ALPINE_VERSION} AS base
WORKDIR /src

# Étape de build : installation et compilation
FROM base AS build
COPY package*.json ./
RUN npm ci
RUN npm run build

# Étape de production : image minimale
FROM base AS production
COPY package*.json ./
RUN npm ci --omit=dev && npm cache clean --force
COPY --from=build /src/dist/ .
CMD ["node", "app.js"]
```

***

#### 🚀 Comment ça marche ?

* `ARG NODE_VERSION="20"` → par défaut, Node.js 20 est utilisé.
* `ARG ALPINE_VERSION="3.21"` → par défaut, Alpine 3.21 est utilisé.
* La base devient donc `node:20-alpine3.21`.

👉 Tu peux réutiliser ces valeurs dans toutes les étapes du build.

***

#### ⚡ Construire avec les valeurs par défaut

```bash
docker build -t myapp:latest .
```

Résultat → Image basée sur `node:20-alpine3.21`.

***

#### ⚡ Construire en surchargeant les arguments

```bash
docker build \
  --build-arg NODE_VERSION=22 \
  --build-arg ALPINE_VERSION=3.20 \
  -t myapp:22-alpine3.20 .
```

Résultat → Image basée sur `node:22-alpine3.20`.

***

#### 📊 Avantages

✅ Un seul Dockerfile, plusieurs variantes d’images.\
✅ Maintenance simplifiée (changer la valeur de l’ARG suffit).\
✅ Bonne pratique : définir les `ARG` **au début du fichier** pour plus de lisibilité.

## 🌍 Exemple d’utilisation de `ENV` dans un Dockerfile

#### 📌 Rappel

* `ENV` définit une **variable d’environnement**.
* Elle est disponible pour toutes les instructions suivantes du build (`RUN`, `COPY`, `CMD`, etc.).
* Elle persiste aussi **dans le conteneur en exécution** → ton application peut l’utiliser.

***

#### 📄 Exemple simple

Ici, on définit `NODE_ENV=production` pour que `npm` n’installe pas les dépendances de développement.

```dockerfile
# syntax=docker/dockerfile:1

FROM node:20
WORKDIR /app

# Copier les fichiers de dépendances
COPY package*.json ./

# Définir la variable d’environnement
ENV NODE_ENV=production

# Installer uniquement les dépendances de prod
RUN npm ci && npm cache clean --force

# Copier le reste du code
COPY . .

# Démarrer l’application
CMD ["node", "app.js"]
```

🔎 Ici :

* `ENV NODE_ENV=production` est appliqué avant l’installation → `npm` ignore les devDependencies.
* Cette variable reste disponible au runtime → ton application peut vérifier `process.env.NODE_ENV`.

***

#### ⚡ Rendre `ENV` configurable au build

Par défaut, `ENV` est figé.\
Si tu veux changer sa valeur **au moment du build**, combine **`ARG`** et **`ENV`** :

```dockerfile
# syntax=docker/dockerfile:1

FROM node:20

# Déclaration d’un argument de build
ARG NODE_ENV=production

# Définition d’une variable d’environnement en fonction de l’ARG
ENV NODE_ENV=$NODE_ENV

WORKDIR /app
COPY package*.json ./
RUN npm ci && npm cache clean --force
COPY . .
CMD ["node", "app.js"]
```

***

#### 🚀 Construire avec valeur par défaut

```bash
docker build -t myapp:prod .
```

➡️ `NODE_ENV=production`

#### 🚀 Construire en surchargeant

```bash
docker build --build-arg NODE_ENV=development -t myapp:dev .
```

➡️ `NODE_ENV=development`

***

#### ⚠️ Attention

* `ENV` reste dans l’image et dans les conteneurs créés à partir de cette image.
* Cela peut provoquer des **effets de bord** si l’app lit cette variable et qu’elle est mal définie.

## 📌 Scoping (portée) des variables dans Dockerfile

#### 1️⃣ Variables `ARG` en **portée globale**

Si tu déclares un `ARG` **avant** tout `FROM`, il est global, mais **non accessible directement** dans une étape de build tant que tu ne le redéclares pas.

```dockerfile
# syntax=docker/dockerfile:1

# Déclaré globalement
ARG NAME="joe"

FROM alpine
# Ici, NAME n’est pas accessible directement
RUN echo "hello ${NAME}!"
```

👉 Résultat : `hello !` (car `$NAME` est vide).

***

#### 2️⃣ Héritage dans un **stage**

Pour qu’un `ARG` global soit utilisable dans une étape, tu dois le redéclarer dans ce stage.

```dockerfile
# syntax=docker/dockerfile:1

ARG NAME="joe"   # global

FROM alpine
ARG NAME         # consommé/redéclaré dans le stage
RUN echo "hello $NAME"
```

👉 Résultat : `hello joe`

***

#### 3️⃣ Héritage dans les **multi-stage builds**

Une fois un `ARG` déclaré ou redéclaré dans un stage, il est **automatiquement hérité** par les stages enfants.

```dockerfile
# syntax=docker/dockerfile:1

FROM alpine AS base
ARG NAME="joe"

FROM base AS build
# Hérité automatiquement de `base`
RUN echo "hello $NAME!"
```

👉 Résultat : `hello joe`

***

#### 4️⃣ Schéma d’héritage simplifié

* `ARG` global : doit être redéclaré pour être utilisé dans un stage.
* `ARG` déclaré dans un stage : hérité par les stages suivants.
* `ENV` déclaré dans un stage : persiste dans le conteneur final (et dans les stages suivants si copiés).

***

#### 5️⃣ Exemple combinant `ARG` et `ENV`

```dockerfile
# syntax=docker/dockerfile:1

# ARG global
ARG NODE_ENV=production

FROM node:20 AS base
# Consommation de l’ARG dans ce stage
ARG NODE_ENV
# Conversion en ENV pour persistance
ENV NODE_ENV=$NODE_ENV
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM base AS build
# Ici NODE_ENV est encore accessible
RUN echo "Environnement = $NODE_ENV"
COPY . .
RUN npm run build
```

👉 Ici :

* `ARG NODE_ENV` est défini globalement.
* Chaque stage qui veut l’utiliser doit le redéclarer.
* On le transforme en `ENV` pour qu’il soit **disponible à l’exécution** du conteneur.

***

⚠️ Attention :

* `ARG` = disponible **pendant le build uniquement** (sauf si converti en `ENV`).
* `ENV` = persiste dans l’image finale et est disponible à l’exécution du conteneur.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## ⚙️ Arguments de build pré-définis dans Docker

Docker fournit certains **`ARG` spéciaux** accessibles dans tous les builds, sans que tu aies besoin de les déclarer dans ton Dockerfile.\
Ils se classent en deux catégories principales :

***

### 1️⃣ Arguments multi-plateformes

Ces arguments décrivent **la plateforme de build** (où BuildKit s’exécute) et **la plateforme cible** (l’image que tu veux construire).

#### Plateforme de build (`BUILD*`)

Ils reflètent **l’OS et l’architecture de la machine qui exécute le build** :

* `BUILDPLATFORM` → plateforme complète (ex. `linux/amd64`)
* `BUILDOS` → système d’exploitation (ex. `linux`, `windows`)
* `BUILDARCH` → architecture CPU (ex. `amd64`, `arm64`)
* `BUILDVARIANT` → variante de l’architecture (ex. `v7` pour ARMv7)

#### Plateforme cible (`TARGET*`)

Ils reflètent **la plateforme que tu veux générer**, définie via `--platform` lors du build :

* `TARGETPLATFORM`
* `TARGETOS`
* `TARGETARCH`
* `TARGETVARIANT`

👉 Exemple pratique pour du **cross-compiling** :

```dockerfile
# syntax=docker/dockerfile:1

# Ici, on force le builder à utiliser sa propre plateforme
FROM --platform=$BUILDPLATFORM golang AS builder

# Héritage explicite des variables dans le stage
ARG TARGETOS
ARG TARGETARCH

# Compilation Go ciblant la plateforme voulue
RUN GOOS=$TARGETOS GOARCH=$TARGETARCH go build -o ./app .
```

Et lors du build :

```bash
docker build --platform=linux/arm64 -t myapp:arm64 .
```

***

### 2️⃣ Arguments de proxy

Docker gère aussi automatiquement les **proxies réseau** lors des builds.\
Tu n’as pas besoin de les déclarer dans ton Dockerfile : un simple `--build-arg` suffit.

Variables prises en charge (insensibles à la casse) :

* `HTTP_PROXY`
* `HTTPS_PROXY`
* `FTP_PROXY`
* `ALL_PROXY`
* `NO_PROXY`

👉 Exemple :

```bash
docker build \
  --build-arg HTTP_PROXY=http://proxy.example.com:8080 \
  --build-arg HTTPS_PROXY=https://proxy.example.com:8080 .
```

#### 🔐 Note importante

* Les arguments de proxy sont **exclus du cache** et de l’historique (`docker history`) par défaut → sécurité accrue.
* Si tu les **références explicitement** dans ton Dockerfile (`ENV HTTP_PROXY=$HTTP_PROXY`), ils finissent dans le cache et peuvent être visibles dans l’image → ⚠️ à éviter sauf besoin spécifique.

***

✅ En résumé :

* `BUILD*` = machine qui construit
* `TARGET*` = machine cible
* `*_PROXY` = configuration réseau pour le build

## ⚙️ Variables de configuration de Buildx et BuildKit

Ces variables d’environnement **ne sont pas utilisées dans le conteneur** et **n’ont aucun rapport avec l’instruction `ENV` du Dockerfile**.\
Elles servent uniquement à **configurer le client Buildx** (côté CLI) et/ou le **démon BuildKit** (côté exécution des builds).

***

### 📋 Liste des principales variables

| Variable                                  | Type             | Description                                                                             |
| ----------------------------------------- | ---------------- | --------------------------------------------------------------------------------------- |
| **`BUILDKIT_COLORS`**                     | String           | Définit les couleurs du texte dans la sortie terminal.                                  |
| **`BUILDKIT_HOST`**                       | String           | Définit l’hôte à utiliser pour des builders distants.                                   |
| **`BUILDKIT_PROGRESS`**                   | String           | Configure le type de sortie du progrès (`auto`, `plain`, `tty`).                        |
| **`BUILDKIT_TTY_LOG_LINES`**              | String           | Définit le nombre de lignes de logs affichées dans le mode TTY pour les étapes actives. |
| **`BUILDX_BAKE_FILE`**                    | String           | Spécifie le ou les fichiers de définition pour `docker buildx bake`.                    |
| **`BUILDX_BAKE_FILE_SEPARATOR`**          | String           | Définit le séparateur de chemin de fichier pour `BUILDX_BAKE_FILE`.                     |
| **`BUILDX_BAKE_GIT_AUTH_HEADER`**         | String           | Définit le schéma d’authentification HTTP pour les fichiers Bake distants.              |
| **`BUILDX_BAKE_GIT_AUTH_TOKEN`**          | String           | Définit le token d’authentification HTTP pour les fichiers Bake distants.               |
| **`BUILDX_BAKE_GIT_SSH`**                 | String           | Configure l’authentification SSH pour les fichiers Bake distants.                       |
| **`BUILDX_BUILDER`**                      | String           | Définit l’instance de builder à utiliser.                                               |
| **`BUILDX_CONFIG`**                       | String           | Spécifie l’emplacement de la configuration, des états et des logs.                      |
| **`BUILDX_CPU_PROFILE`**                  | String           | Génère un profil CPU (`pprof`) à l’emplacement indiqué.                                 |
| **`BUILDX_EXPERIMENTAL`**                 | Booléen          | Active les fonctionnalités expérimentales.                                              |
| **`BUILDX_GIT_CHECK_DIRTY`**              | Booléen          | Active la détection de _dirty Git checkout_ (modifications locales non validées).       |
| **`BUILDX_GIT_INFO`**                     | Booléen          | Supprime les infos Git dans les attestations de provenance.                             |
| **`BUILDX_GIT_LABELS`**                   | String / Booléen | Ajoute des labels Git de provenance aux images.                                         |
| **`BUILDX_MEM_PROFILE`**                  | String           | Génère un profil mémoire (`pprof`) à l’emplacement indiqué.                             |
| **`BUILDX_METADATA_PROVENANCE`**          | String / Booléen | Personnalise les infos de provenance incluses dans le fichier metadata.                 |
| **`BUILDX_METADATA_WARNINGS`**            | String           | Inclut les avertissements de build dans le fichier metadata.                            |
| **`BUILDX_NO_DEFAULT_ATTESTATIONS`**      | Booléen          | Désactive les attestations de provenance par défaut.                                    |
| **`BUILDX_NO_DEFAULT_LOAD`**              | Booléen          | Empêche de charger automatiquement les images dans le store local après le build.       |
| **`EXPERIMENTAL_BUILDKIT_SOURCE_POLICY`** | String           | Définit un fichier de _BuildKit source policy_.                                         |

***

### 🔑 Notes importantes

* Ces variables permettent de **modifier le comportement de Buildx/BuildKit**, pas celui du conteneur final.
* Elles ne persistent pas dans l’image Docker.
* Les valeurs booléennes peuvent être exprimées de plusieurs façons (`true`, `1`, `T` → vrai).
* BuildKit propose aussi des **arguments de build intégrés** (build args) pour configurer certains comportements côté exécution → cf. section _BuildKit built-in build args_.

***

✅ En résumé :

* Ces variables = **leviers de configuration avancée** de Docker Build.
* Elles sont utiles pour ajuster la **sortie logs**, gérer des **builders distants**, activer les **fonctionnalités expérimentales**, ou encore contrôler les **infos de provenance** dans les images.

## 🎨 `BUILDKIT_COLORS`

La variable d’environnement **`BUILDKIT_COLORS`** permet de changer les couleurs utilisées dans la sortie du terminal par BuildKit.\
C’est utile pour personnaliser l’affichage des logs lors d’un build, par exemple pour mettre plus en évidence les erreurs, warnings ou étapes spécifiques.

***

### 📌 Format attendu

Tu dois fournir une chaîne CSV (valeurs séparées par `:`) avec la forme suivante :

```
<type>=<valeur couleur>
```

Exemple :

```bash
export BUILDKIT_COLORS="run=123,20,245:error=yellow:cancel=blue:warning=white"
```

Dans cet exemple :

* `run=123,20,245` → la couleur des étapes en cours (`run`) est définie par un code RGB (Rouge=123, Vert=20, Bleu=245).
* `error=yellow` → les erreurs s’affichent en **jaune**.
* `cancel=blue` → les étapes annulées apparaissent en **bleu**.
* `warning=white` → les avertissements apparaissent en **blanc**.

***

### 🎨 Valeurs possibles

* **Codes RGB** (séparés par des virgules → `R,G,B`) → ex. `255,0,0` pour rouge.
* **Noms de couleurs prédéfinies par BuildKit** → `red`, `green`, `blue`, `yellow`, `white`, etc.

***

### 🔕 Désactiver les couleurs

Si tu veux complètement désactiver la coloration :

```bash
export NO_COLOR=1
```

Ceci suit la recommandation de [no-color.org](https://no-color.org), qui propose une convention standard pour désactiver la coloration dans les terminaux.

***

✅ En résumé :

* `BUILDKIT_COLORS` = personnalisation des couleurs des logs.
* Tu peux utiliser **RGB** ou **noms prédéfinis**.
* `NO_COLOR` = désactive toutes les couleurs.

## 🌐 `BUILDKIT_HOST`

La variable d’environnement **`BUILDKIT_HOST`** permet de définir l’adresse d’un **démon BuildKit distant** que Docker doit utiliser comme builder.

C’est une alternative pratique à l’argument positionnel de la commande `docker buildx create`.

***

### 📌 Version requise

👉 Disponible à partir de **Docker Buildx 0.9.0** et versions ultérieures.

***

### ⚙️ Utilisation

Tu peux l’exporter ainsi :

```bash
export BUILDKIT_HOST=tcp://localhost:1234
docker buildx create --name=remote --driver=remote
```

* Ici, `tcp://localhost:1234` désigne l’adresse d’un démon BuildKit tournant sur le port `1234`.
* La commande `docker buildx create` crée un **builder nommé `remote`** qui se connecte à ce démon.

***

### 🔑 Priorité

* Si tu définis à la fois **`BUILDKIT_HOST`** et que tu passes aussi une adresse en **argument positionnel** lors du `docker buildx create`,\
  ➡️ **l’argument positionnel prend le dessus** sur la variable d’environnement.

***

✅ En résumé :

* `BUILDKIT_HOST` = indiquer l’adresse d’un BuildKit distant.
* Utile pour configurer rapidement ton environnement sans répéter l’option `--driver=remote`.
* Si tu donnes aussi une adresse en argument → **l’argument gagne**.

## ⚡ `BUILDKIT_PROGRESS`

La variable d’environnement **`BUILDKIT_PROGRESS`** définit le **type d’affichage des logs et de la progression** lors de l’exécution d’un build avec BuildKit.

***

### 📌 Valeurs possibles

| Valeur                    | Description                                                                                                            |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **`auto`** _(par défaut)_ | BuildKit choisit automatiquement le meilleur affichage selon le terminal (par ex. `tty` si interactif, sinon `plain`). |
| **`plain`**               | Affiche les logs sous forme **texte brut**, simple et compatible avec la redirection vers des fichiers ou CI/CD.       |
| **`tty`**                 | Affiche les logs avec une **interface interactive** (progress bars, couleurs, mise en page dynamique).                 |
| **`quiet`**               | Mode silencieux : n’affiche quasiment rien pendant le build, seulement les erreurs éventuelles.                        |
| **`rawjson`**             | Produit une sortie en **JSON brut**, utile pour une intégration avec d’autres outils ou un traitement automatisé.      |

***

### ⚙️ Exemple d’utilisation

```bash
# Force l'affichage en mode "plain"
export BUILDKIT_PROGRESS=plain

# Lancer un build
docker buildx build .
```

***

✅ En résumé :

* **`auto`** = choix intelligent (par défaut).
* **`plain`** = pratique pour les logs CI/CD.
* **`tty`** = idéal pour un usage interactif en local.
* **`quiet`** = minimal, pour scripts silencieux.
* **`rawjson`** = destiné aux outils qui consomment directement la sortie.

## ⚡ `BUILDKIT_TTY_LOG_LINES`

La variable d’environnement **`BUILDKIT_TTY_LOG_LINES`** permet de définir **le nombre de lignes de logs visibles** pour chaque étape active lorsqu’on utilise BuildKit en mode **TTY** (progression interactive avec barres et couleurs).

👉 Par défaut, BuildKit affiche **6 lignes** pour chaque étape active.

***

### ⚙️ Exemple d’utilisation

```bash
# Changer le nombre de lignes visibles en mode TTY à 8
export BUILDKIT_TTY_LOG_LINES=8

# Lancer un build
docker buildx build .
```

***

### 📌 Résumé

* **Variable** : `BUILDKIT_TTY_LOG_LINES`
* **Valeur attendue** : un nombre entier (par ex. 4, 6, 10, etc.)
* **Valeur par défaut** : `6`
* **Effet** : plus la valeur est élevée ➝ plus tu verras de lignes de logs par étape **pendant l’exécution interactive** (`tty`).

## ⚡ `EXPERIMENTAL_BUILDKIT_SOURCE_POLICY`

Cette variable d’environnement permet de **spécifier un fichier de politique de sources (source policy)** pour BuildKit.\
👉 Le but est de rendre les builds **reproductibles et sécurisés**, en **verrouillant les dépendances** (pinning des images ou fichiers par hash).

***

### ⚙️ Exemple d’utilisation

```bash
export EXPERIMENTAL_BUILDKIT_SOURCE_POLICY=./policy.json
docker buildx build .
```

Ici, `policy.json` contient les règles de sécurité et de versionnement des sources.

***

### 📝 Exemple de fichier `policy.json`

```json
{
  "rules": [
    {
      "action": "CONVERT",
      "selector": {
        "identifier": "docker-image://docker.io/library/alpine:latest"
      },
      "updates": {
        "identifier": "docker-image://docker.io/library/alpine:latest@sha256:4edbd2beb5f78b1014028f4fbb99f3237d9561100b6881aabbf5acce2c4f9454"
      }
    },
    {
      "action": "CONVERT",
      "selector": {
        "identifier": "https://raw.githubusercontent.com/moby/buildkit/v0.10.1/README.md"
      },
      "updates": {
        "attrs": {
          "http.checksum": "sha256:6e4b94fc270e708e1068be28bd3551dc6917a4fc5a61293d51bb36e6b75c4b53"
        }
      }
    },
    {
      "action": "DENY",
      "selector": {
        "identifier": "docker-image://docker.io/library/golang*"
      }
    }
  ]
}
```

***

### 🔎 Décryptage des règles

* **Rule 1 (CONVERT alpine:latest)**
  * Sélecteur : `alpine:latest`
  * Action : **convertir** le tag flottant `latest` en **digest SHA-256** ➝ garantit que le build utilisera **toujours la même image exacte**, même si `latest` évolue.
* **Rule 2 (CONVERT fichier HTTP)**
  * Sélecteur : un fichier distant (`README.md`)
  * Action : impose un **checksum SHA-256** ➝ BuildKit vérifiera que le fichier téléchargé correspond bien au hash attendu (évite les modifications malveillantes).
* **Rule 3 (DENY golang)**
  * Sélecteur : toutes les images `golang*`
  * Action : **refuser** leur utilisation dans le build.

***

### 📌 Résumé

* Variable : `EXPERIMENTAL_BUILDKIT_SOURCE_POLICY`
* Valeur : chemin vers un fichier JSON (`policy.json`)
* Objectif :\
  ✅ Garantir des **builds reproductibles**\
  ✅ Protéger contre des dépendances non maîtrisées (tags `latest`)\
  ✅ Refuser des images ou sources interdites\
  ✅ Vérifier l’intégrité via **hashs SHA-256**

## BUILDX\_BAKE\_FILE 🧩

### ⚙️ Prérequis

**Docker Buildx 0.26.0 ou version ultérieure**.

### 🎯 Rôle

`BUILDX_BAKE_FILE` permet d’indiquer **un ou plusieurs fichiers de définition** à utiliser avec `docker buildx bake`.\
C’est l’**équivalent** de l’option en ligne de commande **`-f` / `--file`**, mais sous forme **de variable d’environnement**.

***

### 🧪 Utilisation de base

Vous pouvez référencer **plusieurs fichiers** en les séparant avec le **séparateur de chemin du système** :

* **Linux / macOS** : `:`
* **Windows** : `;`

**Exemple (shell POSIX) :**

```bash
export BUILDX_BAKE_FILE=file1.hcl:file2.hcl
```

> 💁‍♀️ Les exemples ci-dessus utilisent `export` (shells POSIX).\
> Sous **PowerShell**, vous utiliseriez par ex. :
>
> ```powershell
> $env:BUILDX_BAKE_FILE = "file1.hcl;file2.hcl"
> ```
>
> (Notez le **`;`** pour Windows.)

***

### 🔀 Définir un séparateur personnalisé

Vous pouvez choisir votre **propre séparateur** via `BUILDX_BAKE_FILE_SEPARATOR`, puis l’utiliser dans `BUILDX_BAKE_FILE`.

**Exemple :**

```bash
export BUILDX_BAKE_FILE_SEPARATOR=@
export BUILDX_BAKE_FILE=file1.hcl@file2.hcl
```

***

### 🧭 Priorité avec `-f`

Si **`BUILDX_BAKE_FILE`** _et_ l’option **`-f`** sont tous deux fournis, **seuls les fichiers passés via `-f` sont pris en compte**.

> ✅ Retenez : `-f` **écrase** `BUILDX_BAKE_FILE`.

***

### 🚫 Gestion des erreurs

Si l’un des fichiers listés **n’existe pas** ou est **invalide**, `bake` **renvoie une erreur**.

***

### 🗺️ Résumé rapide

* 🧩 Variable : `BUILDX_BAKE_FILE` — alternative à `-f/--file`.
* 🧷 Multi-fichiers : séparer avec `:` (Linux/macOS) ou `;` (Windows).
* 🧰 Séparateur custom : définir `BUILDX_BAKE_FILE_SEPARATOR`.
* 🥇 Priorité : `-f` > `BUILDX_BAKE_FILE`.
* 🛑 Erreurs : fichier manquant/invalide ⇒ échec de `bake`.

## BUILDX\_BAKE\_FILE\_SEPARATOR 🔀

### ⚙️ Prérequis

**Docker Buildx 0.26.0 ou version ultérieure**

### 🎯 Rôle

Cette variable contrôle le **séparateur** utilisé entre les chemins de fichiers dans la variable d’environnement **`BUILDX_BAKE_FILE`**.

👉 Utile si vos chemins de fichiers contiennent déjà le caractère de séparation par défaut, ou si vous souhaitez **standardiser** les séparateurs entre différentes plateformes.

***

### 🧪 Exemple d’utilisation

```bash
export BUILDX_BAKE_PATH_SEPARATOR=@
export BUILDX_BAKE_FILE=file1.hcl@file2.hcl
```

***

## BUILDX\_BAKE\_GIT\_AUTH\_HEADER 🔐

### ⚙️ Prérequis

**Docker Buildx 0.14.0 ou version ultérieure**

### 🎯 Rôle

Définit le **schéma d’authentification HTTP** à utiliser lorsque vous chargez une définition **Bake distante** dans un dépôt Git privé.

C’est l’équivalent du secret **`GIT_AUTH_HEADER`**, mais il permet une **authentification préalable (pre-flight)** lors du chargement du fichier Bake distant.

* Valeurs supportées :
  * `bearer` _(par défaut)_
  * `basic`

***

### 🧪 Exemple d’utilisation

```bash
export BUILDX_BAKE_GIT_AUTH_HEADER=basic
```

***

## BUILDX\_BAKE\_GIT\_AUTH\_TOKEN 🔑

### ⚙️ Prérequis

**Docker Buildx 0.14.0 ou version ultérieure**

### 🎯 Rôle

Définit le **token d’authentification HTTP** lorsque vous utilisez une définition **Bake distante** dans un dépôt Git privé.

C’est l’équivalent du secret **`GIT_AUTH_TOKEN`**, mais facilite l’authentification préalable lors du chargement du fichier Bake.

***

### 🧪 Exemple d’utilisation

```bash
export BUILDX_BAKE_GIT_AUTH_TOKEN=$(cat git-token.txt)
```

***

## BUILDX\_BAKE\_GIT\_SSH 🔗

### ⚙️ Prérequis

**Docker Buildx 0.14.0 ou version ultérieure**

### 🎯 Rôle

Permet de spécifier une **liste de chemins vers des sockets d’agent SSH** à transférer à Bake pour s’authentifier auprès d’un serveur Git lorsqu’on utilise une définition Bake distante dans un dépôt privé.

👉 C’est similaire aux **montages SSH** pour les builds, mais appliqué à l’authentification préalable de Bake lors de la résolution de la définition de build.

***

### 📝 Remarques importantes

* Par défaut, Bake utilise déjà le socket défini dans **`SSH_AUTH_SOCK`**.
* Vous n’avez besoin de définir cette variable **que si vous voulez utiliser un autre socket** (chemin différent).
* Vous pouvez spécifier **plusieurs chemins** en les séparant par des **virgules**.

***

### 🧪 Exemple d’utilisation

```bash
export BUILDX_BAKE_GIT_SSH=/run/foo/listener.sock,~/.creds/ssh.sock
```

***

### 🗺️ Résumé visuel

* 🔀 **BUILDX\_BAKE\_FILE\_SEPARATOR** : définit le séparateur de chemins dans `BUILDX_BAKE_FILE`.
* 🔐 **BUILDX\_BAKE\_GIT\_AUTH\_HEADER** : schéma d’auth Git (`bearer` ou `basic`).
* 🔑 **BUILDX\_BAKE\_GIT\_AUTH\_TOKEN** : token d’auth Git pour dépôt privé.
* 🔗 **BUILDX\_BAKE\_GIT\_SSH** : sockets SSH personnalisés pour l’authentification.

## BUILDX\_BUILDER 🏗️

### ⚙️ Rôle

`BUILDX_BUILDER` permet de **surcharger** l’instance de **builder** configurée par défaut.

👉 C’est l’**équivalent exact** de l’option en ligne de commande :

```bash
docker buildx --builder
```

En clair : cette variable vous permet de dire à Docker Buildx **quel builder utiliser** sans avoir à préciser l’option `--builder` à chaque fois.

***

### 🧪 Exemple d’utilisation

```bash
export BUILDX_BUILDER=my-builder
```

Ici, `my-builder` est le **nom de l’instance de builder** que vous avez créé ou configuré auparavant.

***

### 🗺️ Résumé rapide

* 🏗️ **Variable** : `BUILDX_BUILDER`
* 🔁 **Équivaut à** : `--builder` en CLI
* 🎯 **But** : choisir/surcharger l’instance de builder utilisée par défaut
* 🧪 **Exemple** : `export BUILDX_BUILDER=my-builder`

## BUILDX\_CONFIG ⚙️📂

### 🎯 Rôle

La variable **`BUILDX_CONFIG`** permet de spécifier le **répertoire** à utiliser pour :

* la **configuration** de Buildx,
* l’**état** (state),
* et les **logs**.

***

### 🔍 Ordre de recherche des répertoires

Lorsque Docker Buildx cherche son répertoire de configuration, il suit l’ordre suivant :

1. 📌 **`$BUILDX_CONFIG`** (si défini)
2. 📌 **`$DOCKER_CONFIG/buildx`**
3. 📌 **`~/.docker/buildx`** _(valeur par défaut)_

***

### 🧪 Exemple d’utilisation

```bash
export BUILDX_CONFIG=/usr/local/etc
```

Ici, le répertoire `/usr/local/etc` sera utilisé par Buildx pour stocker sa configuration, ses états et ses journaux.

***

### 🗺️ Résumé rapide

* 📂 **Variable** : `BUILDX_CONFIG`
* 🛠️ **But** : définir où Buildx stocke configs, états et logs
* 📌 **Ordre de priorité** :
  1. `$BUILDX_CONFIG`
  2. `$DOCKER_CONFIG/buildx`
  3. `~/.docker/buildx` _(par défaut)_
* 🧪 **Exemple** : `export BUILDX_CONFIG=/usr/local/etc`

## BUILDX\_CPU\_PROFILE 🧠⚡

### ⚙️ Prérequis

**Docker Buildx 0.18.0 ou version ultérieure**

### 🎯 Rôle

Si cette variable est définie, Buildx génère un **profil CPU `pprof`** à l’emplacement indiqué.

⚠️ **Important** :

* Cette option est **uniquement utile pour le développement de Buildx lui-même**.
* Les données de profilage générées **ne servent pas** à analyser les performances d’un build.

***

### 🧪 Exemple d’utilisation

```bash
export BUILDX_CPU_PROFILE=buildx_cpu.prof
```

***

## BUILDX\_EXPERIMENTAL 🧪✨

### 🎯 Rôle

Active les **fonctionnalités expérimentales** de Buildx.

👉 À utiliser si vous souhaitez tester des **nouvelles fonctionnalités non encore stabilisées**.

***

### 🧪 Exemple d’utilisation

```bash
export BUILDX_EXPERIMENTAL=1
```

***

## BUILDX\_GIT\_CHECK\_DIRTY 🧾🔍

### ⚙️ Prérequis

**Docker Buildx 0.10.4 ou version ultérieure**

### 🎯 Rôle

Quand cette variable est définie sur `true` (ou `1`), Buildx vérifie si l’état du dépôt Git est **“sale”** (_dirty state_ = modifications locales non validées).

👉 Cette vérification est utilisée pour les **attestations de provenance** (supply chain / SLSA).

***

### 🧪 Exemple d’utilisation

```bash
export BUILDX_GIT_CHECK_DIRTY=1
```

***

### 🗺️ Résumé rapide

* 🧠 **BUILDX\_CPU\_PROFILE** : génère un profil CPU `pprof` (utile pour dev Buildx, pas pour vos builds).
* 🧪 **BUILDX\_EXPERIMENTAL** : active les fonctionnalités expérimentales.
* 🔍 **BUILDX\_GIT\_CHECK\_DIRTY** : vérifie l’état “dirty” du dépôt Git pour les attestations de provenance.

## BUILDX\_GIT\_INFO 🧾🚫

### ⚙️ Prérequis

**Docker Buildx 0.10.0 ou version ultérieure**

### 🎯 Rôle

Quand cette variable est définie à **`false` (ou `0`)**, Buildx **supprime les informations de contrôle de source (Git)** dans les **attestations de provenance**.

👉 Utile si vous souhaitez générer des images **sans fuite d’informations Git**.

***

### 🧪 Exemple d’utilisation

```bash
export BUILDX_GIT_INFO=0
```

***

## BUILDX\_GIT\_LABELS 🏷️📦

### ⚙️ Prérequis

**Docker Buildx 0.10.0 ou version ultérieure**

### 🎯 Rôle

Ajoute des **labels de provenance** aux images générées, en se basant sur les **informations Git** du projet.

Ces labels peuvent inclure :

* 📝 **`com.docker.image.source.entrypoint`** → Chemin du `Dockerfile` relatif à la racine du projet.
* 🔢 **`org.opencontainers.image.revision`** → Révision Git (commit SHA).
* 🌍 **`org.opencontainers.image.source`** → Adresse SSH ou HTTPS du dépôt Git.

***

### 📌 Exemple de labels générés

```json
"Labels": {
  "com.docker.image.source.entrypoint": "Dockerfile",
  "org.opencontainers.image.revision": "5734329c6af43c2ae295010778cd308866b95d9b",
  "org.opencontainers.image.source": "git@github.com:foo/bar.git"
}
```

***

### ⚙️ Modes disponibles

* `BUILDX_GIT_LABELS=1` → inclut **l’entrée du Dockerfile** et la **révision Git**.
* `BUILDX_GIT_LABELS=full` → inclut **tous les labels** (entrypoint, révision, source).

👉 Si le dépôt est dans un état **dirty** (modifications locales non commit), la révision se verra suffixée par **`-dirty`**.

***

### 🧪 Exemples d’utilisation

Inclure seulement les labels essentiels (entrypoint + révision) :

```bash
export BUILDX_GIT_LABELS=1
```

Inclure **tous** les labels (entrypoint + révision + source) :

```bash
export BUILDX_GIT_LABELS=full
```

***

### 🗺️ Résumé rapide

* 🚫 **BUILDX\_GIT\_INFO** : désactive l’inclusion des infos Git dans les attestations.
* 🏷️ **BUILDX\_GIT\_LABELS** : ajoute des labels Git aux images (`entrypoint`, `revision`, `source`).
* ⚠️ Si dépôt “dirty” → révision suffixée par `-dirty`.

## BUILDX\_MEM\_PROFILE 🧠📊

### ⚙️ Prérequis

**Docker Buildx 0.18.0 ou version ultérieure**

### 🎯 Rôle

Si définie, cette variable permet à Buildx de générer un **profil mémoire `pprof`** à l’emplacement indiqué.

⚠️ **Note importante** :

* Cette option est **uniquement utile pour le développement de Buildx** lui-même.
* Les données de profilage générées **ne servent pas** à analyser les performances d’un build.

***

### 🧪 Exemple d’utilisation

```bash
export BUILDX_MEM_PROFILE=buildx_mem.prof
```

***

## BUILDX\_METADATA\_PROVENANCE 🧾🔍

### ⚙️ Prérequis

**Docker Buildx 0.14.0 ou version ultérieure**

### 🎯 Rôle

Par défaut, Buildx inclut une **provenance minimale** dans le fichier de métadonnées (via l’option `--metadata-file`).

Avec `BUILDX_METADATA_PROVENANCE`, vous pouvez personnaliser le **niveau d’information de provenance** ajouté :

* `min` → provenance minimale _(par défaut)_.
* `max` → provenance complète.
* `disabled`, `false` ou `0` → **aucune provenance**.

***

## BUILDX\_METADATA\_WARNINGS ⚠️📑

### ⚙️ Prérequis

**Docker Buildx 0.16.0 ou version ultérieure**

### 🎯 Rôle

Par défaut, Buildx **n’inclut pas** les **avertissements de build** dans le fichier de métadonnées (`--metadata-file`).

En définissant cette variable à **`1` ou `true`**, vous activez leur inclusion.

***

## BUILDX\_NO\_DEFAULT\_ATTESTATIONS 🚫📝

### ⚙️ Prérequis

**Docker Buildx 0.10.4 ou version ultérieure**

### 🎯 Rôle

Depuis BuildKit **v0.11**, des **attestations de provenance** sont automatiquement ajoutées aux images que vous construisez.

👉 En définissant cette variable à **`1`**, vous désactivez cette inclusion par défaut.

***

### 🧪 Exemple d’utilisation

```bash
export BUILDX_NO_DEFAULT_ATTESTATIONS=1
```

***

## BUILDX\_NO\_DEFAULT\_LOAD 🚫📦

### 🎯 Rôle

Lorsque vous construisez une image avec le **driver Docker**, celle-ci est automatiquement **chargée dans le store d’images local** une fois le build terminé.

👉 En définissant cette variable, vous pouvez **désactiver ce chargement automatique**.

***

### 🧪 Exemple d’utilisation

```bash
export BUILDX_NO_DEFAULT_LOAD=1
```

***

### 🗺️ Résumé rapide

* 🧠 **BUILDX\_MEM\_PROFILE** → génère un profil mémoire `pprof` (dev Buildx uniquement).
* 🧾 **BUILDX\_METADATA\_PROVENANCE** → contrôle le niveau de provenance (`min`, `max`, `disabled`).
* ⚠️ **BUILDX\_METADATA\_WARNINGS** → inclut les warnings dans le fichier de métadonnées.
* 🚫 **BUILDX\_NO\_DEFAULT\_ATTESTATIONS** → désactive les attestations de provenance par défaut.
* 🚫📦 **BUILDX\_NO\_DEFAULT\_LOAD** → empêche le chargement auto des images dans le store local.
