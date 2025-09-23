# 🐳 Docker Build

**Docker Build** est l’une des fonctionnalités les plus utilisées de **Docker Engine**.\
Chaque fois que tu crées une image, tu utilises `docker build`.

C’est une étape essentielle du cycle de développement logiciel, car elle permet de :\
✅ **packager ton application** avec toutes ses dépendances,\
✅ **standardiser l’environnement**,\
✅ **déployer ton code partout** (localement, en cloud, en CI/CD, etc.).

Mais **Docker Build** n’est pas seulement une commande, c’est tout un **écosystème d’outils** qui simplifie la création et la gestion d’images, depuis les cas simples jusqu’aux scénarios avancés.

***

### ⚙️ Les fonctionnalités principales

#### 📦 1. Packager ton application

* Crée une **image exécutable et portable** contenant ton application + ses dépendances.
* Tu peux l’exécuter **partout** : en local, sur un serveur, ou dans le cloud.
*   Exemple :

    ```bash
    docker build -t monapp:1.0 .
    docker run monapp:1.0
    ```

***

#### 🏗️ 2. Multi-stage builds

* Permet de **réduire la taille des images** en séparant les étapes (build, test, exécution).
* Seule la dernière étape est conservée → image plus **légère et sécurisée**.
*   Exemple :

    ```dockerfile
    FROM golang:1.20 AS build
    WORKDIR /app
    COPY . .
    RUN go build -o app

    FROM alpine:3.18
    COPY --from=build /app/app /app
    CMD ["/app/app"]
    ```

***

#### 🌍 3. Multi-platform images

* Construire et exécuter des images pour différentes **architectures CPU** (x86, ARM, etc.).
* Utile pour Raspberry Pi, Mac M1/M2, serveurs cloud hétérogènes.
*   Exemple :

    ```bash
    docker buildx build --platform linux/amd64,linux/arm64 -t monapp:multi .
    ```

***

#### ⚡ 4. BuildKit

* Le moteur moderne de construction d’images Docker.
* Avantages :
  * Builds **parallélisés et plus rapides**,
  * **meilleure gestion du cache**,
  * support natif des **multi-plateformes**,
  * **secrets** et **SSH forwarding** intégrés.
*   Activer :

    ```bash
    export DOCKER_BUILDKIT=1
    ```

***

#### 🖥️ 5. Build drivers

* Permettent de choisir **où et comment** exécuter le build :
  * en local,
  * dans un container,
  * dans un cluster distant.
* Exemple avec `docker buildx create --driver docker-container`.

***

#### 📤 6. Exporters

* Ne pas seulement générer des images Docker, mais aussi :
  * exporter en **archive tar**,
  * générer un **Dockerfile modifié**,
  * produire des **artefacts personnalisés**.

***

#### 🔄 7. Build caching

* Permet d’**éviter les reconstructions inutiles**.
* Docker réutilise les couches déjà construites si rien n’a changé.
* Gain de temps énorme, surtout pour l’installation de dépendances.

***

#### 🧩 8. Bake

* Outil avancé pour **orchestrer plusieurs builds**.
* Décrit les builds dans un fichier (`docker-bake.hcl` ou `docker-compose.yml`).
* Utile pour CI/CD et projets complexes.
*   Exemple :

    ```bash
    docker buildx bake
    ```

***

### ✅ En résumé

**Docker Build** n’est pas juste un `docker build .` :\
c’est un **écosystème complet** qui te permet de créer des images **plus rapides, plus légères, multi-plateformes et sécurisées**, tout en gardant une grande flexibilité (BuildKit, Bake, Exporters, caching).
