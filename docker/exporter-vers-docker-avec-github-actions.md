# 📦 Exporter vers Docker avec GitHub Actions

Il peut arriver que tu veuilles que le **résultat de ton build** soit disponible directement dans le **client Docker (`docker images`)**.\
👉 Cela permet ensuite d’utiliser cette image dans une **autre étape de ton workflow** (ex. tests, inspection, lancement de conteneur).

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
      # 1️⃣ Configuration de Buildx
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      # 2️⃣ Construction et export vers Docker local
      - name: Build
        uses: docker/build-push-action@v6
        with:
          load: true
          tags: myimage:latest
      
      # 3️⃣ Inspection de l’image
      - name: Inspect
        run: |
          docker image inspect myimage:latest
```

***

### ⚡ Explication des étapes

1. **Set up Docker Buildx** ⚙️
   * Configure Buildx comme moteur de build.
2. **Build avec `load: true`** 📥
   * Au lieu de pousser l’image dans un registre, le résultat du build est **chargé dans le Docker local** du runner GitHub Actions.
   * L’image apparaît donc dans la liste `docker images`.
3. **Inspecter l’image** 🔍
   * Avec `docker image inspect`, tu peux vérifier :
     * ses métadonnées,
     * ses labels,
     * ses couches,
     * ses configurations.

***

### 🎯 Cas d’usage

* ✅ Lancer des **tests fonctionnels** sur l’image fraîchement construite.
* ✅ Vérifier la **configuration** de l’image avant de la publier.
* ✅ Exécuter des conteneurs dans le workflow GitHub (ex. tests d’intégration avec `docker run`).
