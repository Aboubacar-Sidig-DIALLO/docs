# 🏷️ Gérer les tags et labels avec GitHub Actions

👉 Si tu veux une **gestion automatique des tags** et des **labels conformes à la spécification OCI (Open Container Initiative)**, tu peux utiliser une étape dédiée avec l’action **Docker Metadata Action**.

Cette action génère automatiquement :

* les **tags** de l’image Docker 🏷️,
* les **labels** descriptifs 📝,\
  en fonction des **événements GitHub** et des **métadonnées Git**.

***

### 📝 Exemple de workflow

```yaml
name: ci

on:
  # Exécution programmée (cron)
  schedule:
    - cron: "0 10 * * *"
  # Sur push (branches + tags semver)
  push:
    branches:
      - "**"
    tags:
      - "v*.*.*"
  # Sur pull request
  pull_request:

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      # 1️⃣ Extraction des métadonnées Docker
      - name: Docker meta
        id: meta
        uses: docker/metadata-action@v5
        with:
          # Liste des images (base pour les tags)
          images: |
            name/app
            ghcr.io/username/app

          # Stratégies de génération des tags
          tags: |
            type=schedule                   # exécution planifiée
            type=ref,event=branch           # nom de branche
            type=ref,event=pr               # pull request
            type=semver,pattern={{version}} # tag semver complet (ex: v1.2.3)
            type=semver,pattern={{major}}.{{minor}} # ex: v1.2
            type=semver,pattern={{major}}   # ex: v1
            type=sha                        # hash de commit

      # 2️⃣ Login Docker Hub (sauf PR)
      - name: Login to Docker Hub
        if: github.event_name != 'pull_request'
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # 3️⃣ Login GitHub Container Registry (sauf PR)
      - name: Login to GHCR
        if: github.event_name != 'pull_request'
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.repository_owner }}
          password: ${{ secrets.GITHUB_TOKEN }}

      # 4️⃣ QEMU pour support multi-arch
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      # 5️⃣ Buildx
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # 6️⃣ Build et Push
      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

***

### ⚡ Explication des étapes

1. **Docker Metadata Action** ⚙️
   * Génère automatiquement :
     * `tags` : ex. `v1.2.3`, `v1.2`, `v1`, `sha-xxxx`
     * `labels` : conformes OCI (`org.opencontainers.image.title`, `description`, `revision`, etc.)
2. **Login aux registres** 🔑
   * Docker Hub et GHCR.
   * Désactivé pour les pull requests (pas de push).
3. **Build multi-plateformes** 🐳
   * Via QEMU et Buildx.
4. **Push conditionnel** 📤
   * Si `event_name != 'pull_request'`, l’image est poussée.
   * Sinon, seulement construite et testée.

***

### 🎯 Avantages

* ✅ **Tags cohérents** générés automatiquement (plus besoin de les gérer à la main).
* ✅ **Support des versions SemVer** (`major.minor.patch`).
* ✅ **Traçabilité Git** → chaque image est liée à un commit ou un tag Git.
* ✅ **Interopérabilité** → images conformes au format **OCI**.
