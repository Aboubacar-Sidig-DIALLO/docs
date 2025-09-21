# 🔄 Copier une image entre registres avec GitHub Actions

Quand tu construis une **image multi-plateforme** avec **Buildx**, tu peux la **copier d’un registre à un autre** sans avoir besoin de rebuilder l’image.\
👉 Cela se fait grâce à la commande **`docker buildx imagetools create`**.

***

### 📝 Exemple de workflow complet

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

      # 2️⃣ Connexion à GitHub Container Registry (GHCR)
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.repository_owner }}
          password: ${{ secrets.GITHUB_TOKEN }}

      # 3️⃣ Activer QEMU (nécessaire pour builds multi-arch)
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      
      # 4️⃣ Configurer Buildx
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # 5️⃣ Construire et pousser l’image sur Docker Hub
      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          platforms: linux/amd64,linux/arm64
          push: true
          tags: |
            user/app:latest
            user/app:1.0.0

      # 6️⃣ Copier l’image vers GHCR
      - name: Push image to GHCR
        run: |
          docker buildx imagetools create \
            --tag ghcr.io/user/app:latest \
            --tag ghcr.io/user/app:1.0.0 \
            user/app:latest
```

***

### ⚡ Explication étape par étape

1. 🔑 **Connexion à Docker Hub** → avec `DOCKERHUB_USERNAME` et `DOCKERHUB_TOKEN`.
2. 🔑 **Connexion à GHCR (GitHub Container Registry)** → avec `GITHUB_TOKEN`.
3. 🖥️ **QEMU activé** → permet les builds multi-architectures (`amd64`, `arm64`, etc.).
4. ⚙️ **Setup Buildx** → configure l’environnement de build avancé.
5. 🏗️ **Build + Push vers Docker Hub** → crée et publie l’image multi-platforms (`latest` + `1.0.0`).
6. 🔄 **Copie vers GHCR** → sans rebuild, via `imagetools create`.

***

### 🎯 Avantages

* 🚀 **Pas besoin de reconstruire l’image** → gain de temps et de ressources.
* 🔄 **Multi-registres** → tu peux publier la même image sur **Docker Hub**, **GHCR**, ou un registre privé.
* 📦 **Support multi-plateformes** → une seule image pour plusieurs environnements (x86\_64, ARM, etc.).
