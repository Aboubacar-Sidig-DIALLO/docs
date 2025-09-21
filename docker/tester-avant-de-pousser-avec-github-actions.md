# 🧪 Tester avant de pousser avec GitHub Actions

👉 Dans certains cas, tu veux **valider que ton image fonctionne correctement** avant de la pousser dans un registre (Docker Hub, GHCR, etc.).

La stratégie :

1. 🏗️ **Construire et exporter l’image localement** (dans le runner GitHub).
2. ✅ **Lancer des tests** sur cette image.
3. 🌍 **Construire en multi-plateformes et pousser** dans le registre si les tests passent.

***

### 📝 Exemple de workflow

```yaml
name: ci

on:
  push:

env:
  TEST_TAG: user/app:test
  LATEST_TAG: user/app:latest

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

      # 2️⃣ QEMU pour build multi-arch
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      # 3️⃣ Buildx
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # 4️⃣ Construire et exporter localement
      - name: Build and export to Docker
        uses: docker/build-push-action@v6
        with:
          load: true
          tags: ${{ env.TEST_TAG }}

      # 5️⃣ Tester l’image
      - name: Test
        run: |
          docker run --rm ${{ env.TEST_TAG }}

      # 6️⃣ Build multi-arch et push
      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          platforms: linux/amd64,linux/arm64
          push: true
          tags: ${{ env.LATEST_TAG }}
```

***

### ⚡ Explication des étapes

1. **Build & Export** 🏗️
   * `load: true` → charge l’image directement dans le Docker local du runner.
   * Tag `user/app:test` utilisé pour les tests.
2. **Test de l’image** ✅
   * Ici, un simple `docker run`.
   * Tu peux y mettre des tests unitaires, d’intégration ou vérifier que ton app démarre bien.
3. **Build & Push multi-arch** 🌍
   * Une fois les tests validés → build final (`amd64` + `arm64`) et push sur le registre.
   * Tag `user/app:latest`.

***

### 🔎 Note importante

* L’image **linux/amd64** est construite **une seule fois**.
* Le cache interne de Buildx est réutilisé 👉 la deuxième étape **ne reconstruit que linux/arm64**.
* ⏱️ Résultat : builds plus rapides grâce au cache.

***

### 🎯 Cas d’usage

* ✅ Tu veux éviter de **pousser une image cassée** dans ton registre.
* ✅ Tu veux automatiser des **tests post-build** (ex. healthchecks, smoke tests).
* ✅ Tu veux **séparer build/test/push** proprement dans ton pipeline CI/CD.
