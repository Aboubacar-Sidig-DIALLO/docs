# 🏷️ Ajouter des annotations d’image avec GitHub Actions

Les **annotations** permettent d’ajouter des **métadonnées arbitraires** aux composants d’une image OCI :

* **manifests**,
* **indexes**,
* **descriptors**.

👉 Cela sert à enrichir vos images avec des informations supplémentaires (version, commit, build info…), de manière **standardisée et compatible OCI**.

***

### ⚙️ Ajouter des annotations dans un workflow

Pour annoter vos images lors de leur construction avec GitHub Actions, utilisez :

* **`docker/metadata-action`** → génère automatiquement des annotations conformes OCI,
* **`docker/build-push-action`** (ou **`bake-action`**) → applique ces annotations aux images.

***

#### 📝 Exemple basique de workflow avec annotations

```yaml
name: ci

on:
  push:

env:
  IMAGE_NAME: user/app

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.IMAGE_NAME }}

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          tags: ${{ steps.meta.outputs.tags }}
          annotations: ${{ steps.meta.outputs.annotations }}
          push: true
```

👉 Dans ce workflow :

* **`docker/metadata-action`** génère des tags + annotations,
* **`docker/build-push-action`** construit et pousse l’image avec ces annotations intégrées.

***

### 🎚️ Configurer le niveau des annotations

Par défaut, les annotations sont appliquées aux **manifests**.\
Mais vous pouvez définir le(s) niveau(x) sur lesquels les annotations seront placées.

➡️ Utilisez la variable d’environnement **`DOCKER_METADATA_ANNOTATIONS_LEVELS`** dans l’étape `metadata-action`.

* Valeurs possibles :
  * `manifest` → annotations appliquées aux manifests (par défaut),
  * `index` → annotations appliquées aux indexes,
  * `descriptor` ou `index-descriptor` → annotations appliquées aux descripteurs.
* Vous pouvez fournir une liste séparée par des virgules pour annoter plusieurs niveaux.

***

#### 📝 Exemple avec annotations sur **manifest + index**

```yaml
name: ci

on:
  push:

env:
  IMAGE_NAME: user/app

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.IMAGE_NAME }}
        env:
          DOCKER_METADATA_ANNOTATIONS_LEVELS: manifest,index

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          tags: ${{ steps.meta.outputs.tags }}
          annotations: ${{ steps.meta.outputs.annotations }}
          push: true
```

***

⚠️ **Important** :

* Le build doit produire les composants que vous voulez annoter.
  * Exemple : si vous voulez annoter un **index**, le build doit générer un **index**.
  * Si le build ne produit qu’un **manifest** et que vous essayez d’annoter `index`, le workflow échouera ❌.

***

👉 Avec ça, tu sais comment :

* générer des annotations OCI conformes,
* les appliquer au bon niveau (manifest, index, descriptor),
* sécuriser et enrichir tes images Docker.
