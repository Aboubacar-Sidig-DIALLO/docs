# 🗄️ Registre local avec GitHub Actions

Dans certains cas (tests, validation CI, debug), tu peux avoir besoin de **créer un registre Docker local** et d’y pousser tes images.\
👉 Cela permet de simuler un environnement réel sans publier l’image sur un registre public.

***

### 📝 Exemple de workflow avec un registre local

```yaml
name: ci

on:
  push:

jobs:
  docker:
    runs-on: ubuntu-latest
    services:
      # 1️⃣ Service registre local
      registry:
        image: registry:2
        ports:
          - 5000:5000

    steps:
      # 2️⃣ QEMU pour builds multi-arch
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      
      # 3️⃣ Buildx avec option réseau host (pour accéder au registre local)
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
        with:
          driver-opts: network=host
      
      # 4️⃣ Construire et pousser dans le registre local
      - name: Build and push to local registry
        uses: docker/build-push-action@v6
        with:
          push: true
          tags: localhost:5000/name/app:latest
      
      # 5️⃣ Inspecter l’image dans le registre local
      - name: Inspect
        run: |
          docker buildx imagetools inspect localhost:5000/name/app:latest
```

***

### ⚡ Explication étape par étape

1. **Créer le service `registry`** 🗄️
   * Utilise l’image officielle `registry:2`.
   * Expose le port `5000`.
   * Le registre local est accessible sur `localhost:5000`.
2. **Configurer QEMU** 🖥️
   * Pour builds multi-arch (ex. `amd64` + `arm64`).
3. **Setup Buildx avec `network=host`** 🌐
   * Permet au builder de communiquer avec le registre local via `localhost`.
4. **Build + Push dans le registre local** 📦
   * L’image est taggée avec `localhost:5000/name/app:latest`.
   * Elle est stockée dans ton registre local.
5. **Inspecter l’image** 🔍
   * Vérifie les métadonnées de l’image avec `imagetools inspect`.

***

### 🎯 Cas d’usage

* ✅ **Tests de CI/CD** sans publier sur Docker Hub.
* ✅ Vérification que ton image est correctement construite et pushée.
* ✅ Simulation d’un **environnement privé** avant mise en prod.
* ✅ Débogage rapide d’images multi-plateformes.
