# 🌍 Images multi-plateformes avec GitHub Actions

Docker et Buildx permettent de construire des images **compatibles avec plusieurs architectures** (par ex. `linux/amd64` et `linux/arm64`).\
👉 C’est indispensable pour que ton image fonctionne aussi bien :

* sur des serveurs **x86 (Intel/AMD)**,
* que sur des machines **ARM (Raspberry Pi, AWS Graviton, Apple Silicon M1/M2, etc.)**.

***

### 📝 Exemple de workflow multi-plateformes

```yaml
name: ci

on:
  push:

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      # 1️⃣ Connexion à Docker Hub
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # 2️⃣ Activer QEMU (émulation pour ARM, etc.)
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      # 3️⃣ Configurer Buildx
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # 4️⃣ Build + Push de l’image multi-arch
      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          platforms: linux/amd64,linux/arm64
          push: true
          tags: user/app:latest
```

***

### ⚡ Explication

1. 🔑 **Login** → connexion au registre Docker Hub.
2. 🖥️ **QEMU** → active l’émulation pour construire ARM sur une machine x86 (ou inversement).
3. ⚙️ **Buildx** → moteur de build multi-plateformes.
4. 🚀 **Build + Push** → publie une seule image **compatible AMD64 + ARM64**.

📍 _Dans la doc officielle, une illustration montre un seul tag Docker (`user/app:latest`) qui regroupe plusieurs architectures._

***

### 📦 Charger une image multi-plateformes dans le runner

⚠️ **Problème par défaut** :\
Les runners GitHub Actions n’autorisent pas le **chargement direct** (`docker load`) d’images multi-plateformes dans le store Docker classique.

👉 Solution : utiliser **`docker/setup-docker-action`** pour activer le support **containerd**.

***

#### 📝 Exemple de workflow avec containerd activé

```yaml
name: ci

on:
  push:

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      # 1️⃣ Configurer Docker Engine avec containerd
      - name: Set up Docker
        uses: docker/setup-docker-action@v4
        with:
          daemon-config: |
            {
              "debug": true,
              "features": {
                "containerd-snapshotter": true
              }
            }

      # 2️⃣ Connexion à Docker Hub
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # 3️⃣ Activer QEMU
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      # 4️⃣ Construire + charger localement
      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          platforms: linux/amd64,linux/arm64
          load: true
          tags: user/app:latest
```

***

### 🎯 Avantages

* 🌍 **Portabilité totale** → une seule image pour plusieurs environnements.
* 🔄 **Compatibilité Apple Silicon, ARM servers, Raspberry Pi…**
* 🛠️ **Tests locaux** possibles avec `load: true` + containerd.
* 🚀 Déploiement simplifié (un seul tag = toutes les archis).

## ⚡ Distribuer un build sur plusieurs runners

👉 Dans un build multi-plateformes "classique", **toutes les architectures** (`amd64`, `arm64`, etc.) sont construites sur **le même runner**.\
➡️ Cela peut être **lent** ⏳ si :

* ton Dockerfile est complexe,
* ou si tu cibles beaucoup de plateformes.

💡 Solution : **utiliser une stratégie de matrice GitHub Actions** → chaque plateforme est construite **sur un runner distinct**, puis on assemble le tout dans un **manifest list** (multi-arch image).

***

### 📝 Exemple 1 : stratégie matrice + `imagetools create`

```yaml
name: ci

on:
  push:

env:
  REGISTRY_IMAGE: user/app

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        platform:
          - linux/amd64
          - linux/arm64
    steps:
      # Préparer les variables
      - name: Prepare
        run: |
          platform=${{ matrix.platform }}
          echo "PLATFORM_PAIR=${platform//\//-}" >> $GITHUB_ENV

      # Extraire métadonnées (tags, labels…)
      - name: Docker meta
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY_IMAGE }}

      # Login Docker Hub
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # QEMU + Buildx
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # Build par plateforme (push par digest uniquement)
      - name: Build and push by digest
        id: build
        uses: docker/build-push-action@v6
        with:
          platforms: ${{ matrix.platform }}
          labels: ${{ steps.meta.outputs.labels }}
          tags: ${{ env.REGISTRY_IMAGE }}
          outputs: type=image,push-by-digest=true,name-canonical=true,push=true

      # Exporter digest
      - name: Export digest
        run: |
          mkdir -p ${{ runner.temp }}/digests
          digest="${{ steps.build.outputs.digest }}"
          touch "${{ runner.temp }}/digests/${digest#sha256:}"

      # Uploader digest
      - name: Upload digest
        uses: actions/upload-artifact@v4
        with:
          name: digests-${{ env.PLATFORM_PAIR }}
          path: ${{ runner.temp }}/digests/*
          if-no-files-found: error
          retention-days: 1

  merge:
    runs-on: ubuntu-latest
    needs: build
    steps:
      # Télécharger les digests de chaque job
      - name: Download digests
        uses: actions/download-artifact@v4
        with:
          path: ${{ runner.temp }}/digests
          pattern: digests-*
          merge-multiple: true

      # Login Docker Hub
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # Setup Buildx
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # Docker meta (tags dynamiques)
      - name: Docker meta
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY_IMAGE }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}

      # Créer manifest list (multi-arch image)
      - name: Create manifest list and push
        working-directory: ${{ runner.temp }}/digests
        run: |
          docker buildx imagetools create $(jq -cr '.tags | map("-t " + .) | join(" ")' <<< "$DOCKER_METADATA_OUTPUT_JSON") \
            $(printf '${{ env.REGISTRY_IMAGE }}@sha256:%s ' *)

      # Inspecter l’image finale
      - name: Inspect image
        run: |
          docker buildx imagetools inspect ${{ env.REGISTRY_IMAGE }}:${{ steps.meta.outputs.version }}
```

***

### 📝 Exemple 2 : avec **Bake**

Bake simplifie les définitions de build et permet aussi de paralléliser par plateformes.

* Définition `docker-bake.hcl` :

```hcl
variable "DEFAULT_TAG" {
  default = "app:local"
}

target "docker-metadata-action" {
  tags = ["${DEFAULT_TAG}"]
}

group "default" {
  targets = ["image-local"]
}

target "image" {
  inherits = ["docker-metadata-action"]
}

target "image-local" {
  inherits = ["image"]
  output = ["type=docker"]
}

target "image-all" {
  inherits = ["image"]
  platforms = [
    "linux/amd64",
    "linux/arm/v6",
    "linux/arm/v7",
    "linux/arm64"
  ]
}
```

📍 Ensuite, le workflow GitHub utilise `bake-action` pour distribuer le build par plateformes, exporter les digests, et enfin créer un manifest multi-arch comme dans l’exemple précédent.

***

### 🎯 Avantages

* ⚡ **Builds plus rapides** → chaque architecture est construite en parallèle sur un runner distinct.
* 🌍 **Images multi-plateformes fiables** (pas seulement émulation via QEMU).
* 📦 **Manifest list** → un seul tag Docker (`user/app:latest`) qui contient toutes les archis.
* 🔑 **Intégration avec metadata-action** → tags/labels cohérents et automatisés.
