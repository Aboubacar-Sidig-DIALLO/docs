# 🌍 Builds multi-plateformes

### 🎯 Définition

Un **build multi-plateforme** désigne une **invocation unique** de build qui cible plusieurs combinaisons différentes de :

* système d’exploitation (OS),
* architecture processeur (CPU).

***

### 🛠️ Objectif

Lors de la création d’images Docker, un build multi-plateforme vous permet de produire une **image unique** capable de s’exécuter sur différentes plateformes, par exemple :

* 🖥️ `linux/amd64` (classique PC/serveur x86-64)
* 📱 `linux/arm64` (Raspberry Pi, Mac M1/M2, serveurs ARM dans le cloud)
* 💻 `windows/amd64` (Windows Server ou containers Windows)

***

### ✅ Intérêt

* Une seule image → plusieurs variantes sous le capot.
* Le client Docker choisit **automatiquement la bonne variante** selon la plateforme de la machine hôte.
* Simplifie la distribution et le déploiement d’applications sur des environnements hétérogènes.

## ❓ Pourquoi les builds multi-plateformes ?

### 🚀 Le problème initial

Docker résout le problème du fameux _“ça marche sur ma machine”_ en empaquetant les applications et leurs dépendances dans des **conteneurs**.\
👉 Cela permet d’exécuter la **même application** dans différents environnements (développement, test, production…).

***

### ⚠️ Mais la conteneurisation seule ne suffit pas

Les conteneurs partagent le **noyau (kernel) de l’hôte**.\
➡️ Cela signifie que le code qui s’exécute dans le conteneur doit être **compatible avec l’architecture de l’hôte**.

Exemples :

* ❌ Impossible d’exécuter directement une image `linux/amd64` sur un hôte `arm64` (sauf en utilisant de l’émulation).
* ❌ Impossible d’exécuter un conteneur Windows sur un hôte Linux.

***

### ✅ La solution : les builds multi-plateformes

Les builds multi-plateformes résolvent ce problème en empaquetant **plusieurs variantes** d’une même application dans **une seule image**.

👉 Ainsi, la même image peut tourner sur différents types de matériel, par exemple :

* une machine de développement en **x86-64**,
* une instance cloud basée sur **ARM** (comme Amazon EC2 `arm64`),
* etc.

Le tout **sans avoir recours à l’émulation**.

***

## 🔍 Différence entre images mono-plateforme et multi-plateformes

### 📦 Image mono-plateforme

* Contient **un seul manifeste**
* Ce manifeste pointe vers :
  * une seule configuration
  * un seul ensemble de couches (_layers_)

### 🌍 Image multi-plateforme

* Contient une **liste de manifestes (manifest list)**
* Cette liste pointe vers plusieurs manifestes, chacun correspondant à :
  * une configuration différente
  * un ensemble de couches différent

***

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

***

## 🔄 Fonctionnement lors du push / pull

* Lorsque vous **poussez** (`docker push`) une image multi-plateforme vers un registre :
  * le registre stocke la **manifest list**,
  * ainsi que **tous les manifestes individuels**.
* Lorsque vous **tirez** (`docker pull`) cette image :
  * le registre renvoie la manifest list,
  * Docker sélectionne automatiquement la **bonne variante** selon l’architecture de l’hôte.

Exemples :

* Sur un **Raspberry Pi ARM**, Docker choisit la variante `linux/arm64`.
* Sur un **ordinateur portable x86-64**, Docker choisit la variante `linux/amd64` (si vous utilisez des conteneurs Linux).

## ✅ Prérequis pour les builds multi-plateformes

Avant de construire des images multi-plateformes, vous devez vous assurer que votre environnement Docker est configuré pour les supporter.\
Deux méthodes principales s’offrent à vous :

1. 🔄 **Basculer du “classic image store” vers le&#x20;**_**containerd image store**_
2. 🛠️ **Créer et utiliser un builder personnalisé**

***

### 📂 1. Le _classic image store_ vs le _containerd image store_

* Le **classic image store** (mécanisme historique de Docker Engine) **ne prend pas en charge** les images multi-plateformes.
* En revanche, le **containerd image store** permet à Docker Engine de :
  * **pousser** (`push`),
  * **tirer** (`pull`),
  * et **construire** (`build`)\
    des images multi-plateformes.

👉 Basculer sur le _containerd image store_ est donc recommandé si vous voulez une prise en charge complète dans Docker Engine.

***

### 🏗️ 2. Créer un builder personnalisé

Vous pouvez aussi créer un **builder** basé sur un **driver compatible multi-plateforme**, comme le driver :

```
docker-container
```

Cela permet de réaliser des builds multi-plateformes **sans changer d’image store**.

⚠️ Limite :

* Vous **ne pourrez pas charger** les images multi-plateformes créées directement dans l’image store de Docker Engine.
* Mais vous pouvez tout à fait les **pousser dans un registre** avec :

```bash
docker buildx build --push ...
```

***

## 🖼️ Illustrations à insérer ici

* **Illustration 1 : containerd image store**
* **Illustration 2 : Custom builder**

***

### ⚙️ Étapes d’activation du _containerd image store_

Les étapes dépendent de votre environnement :

* 🖥️ **Docker Desktop**\
  → Activez le _containerd image store_ depuis les **paramètres Docker Desktop**.
* ⚙️ **Docker Engine standalone**\
  → Activez-le via le **fichier de configuration du démon** (`daemon.json`).

***

### 🐧 Cas particulier : émulation avec QEMU

Si vous utilisez **Docker Engine standalone** et que vous devez construire des images multi-plateformes avec **émulation**, vous devez aussi installer **QEMU**.

👉 Voir la documentation : _Install QEMU manually_.

## 🏗️ Construire des images multi-plateformes

### 🎯 Définir les plateformes cibles

Lors du déclenchement d’un build, utilisez l’option **`--platform`** pour définir les plateformes cibles de la sortie du build.\
Exemple :

```bash
docker buildx build --platform linux/amd64,linux/arm64 .
```

👉 Ici, une seule commande génère une image compatible à la fois avec **linux/amd64** (PC/serveurs x86-64) et **linux/arm64** (ARM : Raspberry Pi, Mac M1/M2, serveurs cloud ARM).

***

## 🔀 Stratégies possibles pour les builds multi-plateformes

Selon vos besoins et contraintes, vous pouvez choisir entre **trois approches principales** :

***

### 🐧 1. Utiliser l’émulation (via QEMU)

* QEMU permet d’**émuler une architecture différente** de celle de votre machine hôte.
* Vous pouvez ainsi construire et tester des images pour `arm64` sur un hôte `amd64`, et inversement.
* ✅ Simple à mettre en place.
* ⚠️ Mais les performances peuvent être **significativement plus lentes** pour des builds lourds.

***

### 🌐 2. Utiliser un builder avec plusieurs nœuds natifs

* Configurez un **builder Buildx** qui orchestre plusieurs machines/nœuds, chacun disposant de son architecture native (amd64, arm64, etc.).
* Chaque nœud construit l’image pour son architecture.
* ✅ Plus rapide, car pas d’émulation.
* ⚠️ Nécessite une infrastructure avec accès à plusieurs environnements différents.

***

### ⚙️ 3. Utiliser la cross-compilation (multi-stage builds)

* Certaines plateformes de développement (Go, Rust, etc.) supportent la **cross-compilation** : compiler pour une architecture différente directement depuis la machine hôte.
* Avec des builds multi-étapes, vous pouvez combiner compilation croisée et packaging Docker.
* ✅ Rapide et efficace quand l’écosystème supporte la cross-compilation.
* ⚠️ Peut nécessiter des ajustements manuels selon les langages ou dépendances.

## 🐧 QEMU — Émulation pour builds multi-plateformes

### 🚀 Principe

Construire des images multi-plateformes sous émulation avec **QEMU** est la méthode la plus simple pour commencer, à condition que votre builder la supporte déjà.

* ✅ Aucun changement nécessaire dans votre **Dockerfile**.
* ⚙️ BuildKit détecte automatiquement les architectures disponibles via l’émulation.

***

### ⚠️ Remarque importante

L’émulation avec QEMU peut être **beaucoup plus lente** qu’un build natif, surtout pour des tâches lourdes comme :

* la compilation,
* la compression/décompression.

👉 Si possible, privilégiez l’utilisation de **nœuds natifs multiples** ou la **cross-compilation**.

***

### 🖥️ Cas Docker Desktop

Docker Desktop prend en charge **par défaut** l’exécution et la construction d’images multi-plateformes via QEMU.

* Aucune configuration supplémentaire nécessaire.
* Le builder utilise directement le QEMU inclus dans la **VM Docker Desktop**.

***

### 🐧 Installer QEMU manuellement

Si vous utilisez un builder **en dehors de Docker Desktop**, par exemple :

* Docker Engine sur Linux,
* un builder distant personnalisé,

👉 Vous devez installer QEMU et enregistrer les types d’exécutables sur l’OS hôte.

#### 📌 Prérequis

* Linux kernel **v4.8 ou supérieur**
* `binfmt-support` **v2.1.7 ou supérieur**
* Les binaires QEMU doivent être **statiquement compilés** et enregistrés avec le flag `fix_binary`.

***

### 🛠️ Installation rapide avec l’image `tonistiigi/binfmt`

Vous pouvez installer QEMU et enregistrer les types d’exécutables en une seule commande :

```bash
docker run --privileged --rm tonistiigi/binfmt --install all
```

👉 Cette commande :

* installe les binaires QEMU,
* les enregistre avec `binfmt_misc`,
* permet ainsi à QEMU d’exécuter des formats binaires non natifs pour l’émulation.

***

### ✅ Vérification de l’installation

Une fois QEMU installé et les types d’exécutables enregistrés, tout fonctionne de manière **transparente à l’intérieur des conteneurs**.

Pour vérifier, consultez les flags dans :

```bash
cat /proc/sys/fs/binfmt_misc/qemu-*
```

Si la lettre **`F`** figure parmi les flags, alors le registre est correct.

## 🌐 Multiple native nodes

### 🚀 Pourquoi utiliser plusieurs nœuds natifs ?

* ✅ Meilleur support pour les cas complexes que QEMU ne peut pas gérer.
* ✅ Meilleures performances que l’émulation.
* ⚠️ Introduit une certaine complexité dans la mise en place et la gestion des clusters de builders.

***

### ⚙️ Créer un builder multi-nœuds

Vous pouvez ajouter plusieurs nœuds à un builder avec le flag **`--append`**.

Exemple :\
Ici, on suppose que vous avez déjà créé deux contextes Docker : `node-amd64` et `node-arm64`.

```bash
docker buildx create --use --name mybuild node-amd64
mybuild
docker buildx create --append --name mybuild node-arm64
docker buildx build --platform linux/amd64,linux/arm64 .
```

👉 Résultat : un seul builder (`mybuild`) utilise deux contextes différents pour construire en **amd64** et **arm64**.

***

### ☁️ Alternative : Docker Build Cloud

Si vous ne souhaitez pas gérer vous-même vos clusters de builders :

* **Docker Build Cloud** propose des builders multi-nœuds managés, fournis par l’infrastructure Docker.
* Vous obtenez directement des builders natifs **ARM** et **x86**, sans avoir à les maintenir.
* Avantages supplémentaires : cache de build partagé, simplicité de configuration.

#### Exemple d’utilisation :

```bash
docker buildx create --driver cloud ORG/BUILDER_NAME
cloud-ORG-BUILDER_NAME

docker build \
  --builder cloud-ORG-BUILDER_NAME \
  --platform linux/amd64,linux/arm64,linux/arm/v7 \
  --tag IMAGE_NAME \
  --push .
```

👉 Après inscription à Docker Build Cloud, ajoutez simplement le builder à votre environnement local et lancez vos builds.

***

## ⚙️ Cross-compilation

Selon votre projet, si le langage utilisé supporte bien la **cross-compilation** (par ex. Go, Rust…), vous pouvez tirer parti des **multi-stage builds** pour compiler vos binaires pour différentes plateformes **depuis l’architecture native du builder**.

***

### 🔑 Variables utiles

BuildKit fournit automatiquement deux arguments prédéfinis :

* `$BUILDPLATFORM` → plateforme native du builder
* `$TARGETPLATFORM` → plateforme cible du build

***

### 🧪 Exemple : cross-compilation en Go

```dockerfile
# syntax=docker/dockerfile:1
FROM --platform=$BUILDPLATFORM golang:alpine AS build
ARG TARGETPLATFORM
ARG BUILDPLATFORM
RUN echo "I am running on $BUILDPLATFORM, building for $TARGETPLATFORM" > /log

FROM alpine
COPY --from=build /log /log
```

* Ici, la première étape (`build`) est **épinglée** à la plateforme native (`--platform=$BUILDPLATFORM`) pour éviter que QEMU ne s’active.
* Les variables `$BUILDPLATFORM` et `$TARGETPLATFORM` sont interpolées dans l’instruction `RUN`.
* Dans cet exemple, elles sont juste écrites dans un fichier `/log`, mais en pratique vous les passeriez au compilateur pour générer un binaire cross-compilé.

***

## 🧪 Exemples de builds multi-plateformes

### 1️⃣ Build simple avec QEMU (émulation)

Exemple de build basique qui crée une image multi-plateforme contenant un simple fichier indiquant l’architecture du conteneur.

#### 📌 Prérequis

* Docker Desktop **ou** Docker Engine avec QEMU installé
* _containerd image store_ activé

#### 🛠️ Étapes

1.  Créer un dossier vide et s’y déplacer :

    ```bash
    mkdir multi-platform
    cd multi-platform
    ```
2.  Créer un **Dockerfile** :

    ```dockerfile
    # syntax=docker/dockerfile:1
    FROM alpine
    RUN uname -m > /arch
    ```
3.  Construire l’image pour `linux/amd64` et `linux/arm64` :

    ```bash
    docker build --platform linux/amd64,linux/arm64 -t multi-platform .
    ```
4.  Lancer le conteneur et afficher l’architecture :

    ```bash
    docker run --rm multi-platform cat /arch
    ```

    *   Sur une machine **x86-64**, le résultat sera :

        ```
        x86_64
        ```
    *   Sur une machine **ARM**, le résultat sera :

        ```
        aarch64
        ```

## 📝 Exemple 1 : Build multi-plateforme de Neovim avec **Docker Build Cloud**

### 🚀 Contexte

Cet exemple montre comment lancer un **build multi-plateforme** avec Docker Build Cloud pour compiler et exporter les binaires de **Neovim** pour les plateformes `linux/amd64` et `linux/arm64`.

👉 **Avantage clé** : Docker Build Cloud fournit des builders multi-nœuds managés, supportant nativement les builds multi-plateformes sans émulation.\
✅ Résultat : compilation **beaucoup plus rapide**, surtout pour les tâches CPU-intensives comme la compilation.

***

### 📌 Prérequis

* Vous êtes inscrit à **Docker Build Cloud**.
* Vous avez créé un **builder cloud**.

***

### 🛠️ Étapes

#### 1. Créer un dossier de travail

```bash
mkdir docker-build-neovim
cd docker-build-neovim
```

#### 2. Créer un **Dockerfile** pour compiler Neovim

```dockerfile
# syntax=docker/dockerfile:1
FROM debian:bookworm AS build
WORKDIR /work
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && apt-get install -y \
    build-essential \
    cmake \
    curl \
    gettext \
    ninja-build \
    unzip
ADD https://github.com/neovim/neovim.git#stable .
RUN make CMAKE_BUILD_TYPE=RelWithDebInfo

FROM scratch
COPY --from=build /work/build/bin/nvim /
```

#### 3. Lancer le build avec Docker Build Cloud

```bash
docker build \
  --builder <cloud-builder> \
  --platform linux/amd64,linux/arm64 \
  --output ./bin .
```

👉 Cette commande utilise le builder cloud pour compiler et exporte les binaires dans le dossier local `./bin`.

***

### ✅ Vérification

Listez le contenu du répertoire :

```bash
tree ./bin
```

Résultat attendu :

```
./bin
├── linux_amd64
│   └── nvim
└── linux_arm64
    └── nvim

3 directories, 2 files
```

👉 Vous avez maintenant les binaires **Neovim** compilés pour **amd64** et **arm64** 🎉

***

## 📝 Exemple 2 : Cross-compiler une application Go

### 🚀 Contexte

Cet exemple montre comment **cross-compiler** une application Go simple (un serveur HTTP qui affiche l’architecture du conteneur) pour plusieurs plateformes avec **multi-stage builds**.

ℹ️ Même si l’exemple utilise **Go**, les mêmes principes s’appliquent à d’autres langages supportant la cross-compilation.

***

### 📌 Prérequis

* Docker Desktop **ou** Docker Engine installé

***

### 🛠️ Étapes

#### 1. Créer un dossier de travail

```bash
mkdir go-server
cd go-server
```

#### 2. Créer un premier Dockerfile (non encore multi-plateforme)

```dockerfile
# syntax=docker/dockerfile:1
FROM golang:alpine AS build
WORKDIR /app
ADD https://github.com/dvdksn/buildme.git#eb6279e0ad8a10003718656c6867539bd9426ad8 .
RUN go build -o server .

FROM alpine
COPY --from=build /app/server /server
ENTRYPOINT ["/server"]
```

👉 Ce Dockerfile compile bien l’application, mais ne gère pas encore la cross-compilation.\
👉 Si vous tentez un build multi-plateforme, Docker utilisera **QEMU** par défaut → moins performant.

***

#### 3. Ajouter la cross-compilation

On modifie le Dockerfile pour tirer parti des **arguments prédéfinis** fournis par BuildKit :

* `$BUILDPLATFORM` → plateforme native du builder
* `$TARGETPLATFORM` → plateforme cible
* `$TARGETOS` et `$TARGETARCH` → déduits automatiquement, utilisés pour la compilation Go

***

#### ✅ Dockerfile mis à jour avec cross-compilation

```dockerfile
# syntax=docker/dockerfile:1
FROM --platform=$BUILDPLATFORM golang:alpine AS build
ARG TARGETOS
ARG TARGETARCH
WORKDIR /app
ADD https://github.com/dvdksn/buildme.git#eb6279e0ad8a10003718656c6867539bd9426ad8 .
RUN GOOS=${TARGETOS} GOARCH=${TARGETARCH} go build -o server .

FROM alpine
COPY --from=build /app/server /server
ENTRYPOINT ["/server"]
```

* Ici, l’image `golang:alpine` est **épinglée** au `BUILDPLATFORM` pour éviter l’émulation.
* On ajoute `ARG TARGETOS` et `ARG TARGETARCH` → passés au compilateur via `GOOS` et `GOARCH`.
* Résultat : génération d’un binaire compilé pour la plateforme cible.

***

#### 4. Construire l’image pour deux plateformes

```bash
docker build --platform linux/amd64,linux/arm64 -t go-server .
```

👉 Vous obtenez une image multi-plateforme de votre serveur Go 🎉

***

### 💡 Astuce

Pour simplifier la cross-compilation dans vos projets, vous pouvez aussi utiliser [**xx**](https://github.com/tonistiigi/xx) → une image Docker contenant des scripts utilitaires facilitant la cross-compilation dans les builds Docker.
