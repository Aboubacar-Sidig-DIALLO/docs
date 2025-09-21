# 🚀 Pousser une image vers plusieurs registres avec GitHub Actions

👉 Il est fréquent de vouloir publier une image Docker sur **plusieurs registres** (par exemple **Docker Hub** et **GitHub Container Registry (GHCR)**) pour des raisons de :

* **sécurité** 🔒 (backup et redondance),
* **performance** ⚡ (accès plus rapide selon la région),
* **interopérabilité** 🌍 (chaque équipe peut choisir son registre).

Avec GitHub Actions, tu peux te connecter à plusieurs registres et pousser l’image sur tous en une seule étape.

***

### 📝 Exemple de workflow

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

      # 3️⃣ Activer QEMU (si multi-arch)
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      # 4️⃣ Configurer Buildx
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # 5️⃣ Construire et pousser vers plusieurs registres
      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          platforms: linux/amd64,linux/arm64
          push: true
          tags: |
            user/app:latest
            user/app:1.0.0
            ghcr.io/user/app:latest
            ghcr.io/user/app:1.0.0
```

***

### ⚡ Explication des étapes

1. **Connexion à Docker Hub** 🔑
   * Utilisation de `docker/login-action`.
   * Les identifiants sont stockés dans `vars.DOCKERHUB_USERNAME` et `secrets.DOCKERHUB_TOKEN`.
2. **Connexion à GHCR (GitHub Container Registry)** 🐙
   * Utilisation du `secrets.GITHUB_TOKEN` (fourni automatiquement par GitHub Actions).
3. **QEMU + Buildx** 🛠️
   * Permet de builder pour plusieurs plateformes (`amd64`, `arm64`, etc.).
4. **Push multi-tags et multi-registres** 📦
   * Ici, on pousse **les mêmes images** vers **Docker Hub** (`user/app`) et **GHCR** (`ghcr.io/user/app`).
   * Avec plusieurs tags : `latest` et `1.0.0`.

***

### 🎯 Cas d’usage

* ✅ Tu veux **héberger ton image publique sur Docker Hub** mais **ton CI interne utilise GHCR**.
* ✅ Tu veux assurer la **redondance** de ton image (si un registre tombe en panne, l’autre reste dispo).
* ✅ Tu veux publier une image à la fois pour **usage public et usage privé**.
