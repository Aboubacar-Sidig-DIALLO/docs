# ✅ Validation de la configuration de build avec GitHub Actions

Les **build checks** permettent de **valider la configuration de ton build Docker** sans exécuter réellement la construction de l’image.\
👉 C’est une étape de contrôle qualité : elle détecte les erreurs ou avertissements avant de lancer un build complet.

***

### 🏗️ Exécuter des checks avec `docker/build-push-action`

Pour valider la configuration avec `build-push-action`, il suffit d’ajouter l’option :

```yaml
with:
  call: check
```

➡️ Si des avertissements ou erreurs de configuration sont détectés, le workflow **échoue immédiatement** ❌.

***

#### 📝 Exemple de workflow avec `build-push-action`

```yaml
name: ci

on:
  push:

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Validate build configuration
        uses: docker/build-push-action@v6
        with:
          call: check

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          push: true
          tags: user/app:latest
```

👉 Explication :

* **Validate build configuration** → vérifie la configuration (Dockerfile, arguments, etc.).
* **Build and push** → ne s’exécute que si la validation réussit ✅.

***

### 🧩 Exécuter des checks avec `docker/bake-action`

Si tu utilises **Docker Bake** (`bake-action`) pour gérer tes builds multi-cibles, la validation est intégrée directement.

* Il suffit de définir une **target** spéciale dans ton fichier `docker-bake.hcl`.
* Cette target appelle la méthode `check`.

***

#### 📝 Exemple `docker-bake.hcl`

```hcl
target "build" {
  dockerfile = "Dockerfile"
  args = {
    FOO = "bar"
  }
}

target "validate-build" {
  inherits = ["build"]
  call = "check"
}
```

***

#### 📝 Exemple de workflow avec `bake-action`

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

      - name: Validate build configuration
        uses: docker/bake-action@v6
        with:
          targets: validate-build

      - name: Build
        uses: docker/bake-action@v6
        with:
          targets: build
          push: true
```

👉 Explication :

* **validate-build** → hérite de la config de `build` mais appelle uniquement le **check**.
* **build** → exécute ensuite la vraie construction et le push.

***

### 📌 En résumé

* 🔎 `call: check` (avec **build-push-action**) → valide la config avant de builder.
* 🧩 `validate-build` (avec **bake-action**) → crée une target spéciale de validation.
* 🚫 Si une erreur est détectée → le build s’arrête immédiatement → gain de temps et fiabilité accrue.
