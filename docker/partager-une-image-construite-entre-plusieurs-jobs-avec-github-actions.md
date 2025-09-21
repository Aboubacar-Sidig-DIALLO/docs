# 🔗 Partager une image construite entre plusieurs jobs avec GitHub Actions

👉 Par défaut :

* Chaque **job** GitHub Actions s’exécute sur un **runner isolé** 🛑.
* Donc, une image construite dans un job **n’est pas disponible** dans un autre.

⚡ Solutions possibles :

* Utiliser un **self-hosted runner** (persistance locale).
* Utiliser **Docker Build Cloud**.
* OU → passer par les **artifacts GitHub Actions** pour exporter/importer l’image entre jobs.

***

### 📝 Exemple de workflow

#### 1️⃣ Job `build` → construire et exporter l’image

```yaml
name: ci

on:
  push:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # Configurer Buildx
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # Construire et exporter l’image en tarball
      - name: Build and export
        uses: docker/build-push-action@v6
        with:
          tags: myimage:latest
          outputs: type=docker,dest=${{ runner.temp }}/myimage.tar

      # Uploader l’image comme artifact
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: myimage
          path: ${{ runner.temp }}/myimage.tar
```

***

#### 2️⃣ Job `use` → télécharger et charger l’image

```yaml
  use:
    runs-on: ubuntu-latest
    needs: build
    steps:
      # Télécharger l’image exportée
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: myimage
          path: ${{ runner.temp }}

      # Charger l’image dans Docker
      - name: Load image
        run: |
          docker load --input ${{ runner.temp }}/myimage.tar
          docker image ls -a
```

***

### ⚡ Explication des étapes

* **Export de l’image** 🗄️
  * `outputs: type=docker,dest=...` → exporte l’image sous forme de fichier `.tar`.
* **Upload artifact** ⬆️
  * L’image `.tar` est stockée temporairement par GitHub Actions.
* **Download artifact** ⬇️
  * Dans le job suivant, on récupère ce fichier.
* **docker load** 📦
  * Recharge l’image dans le Docker local du runner.

***

### 🎯 Cas d’usage

* ✅ Tu veux **tester** une image dans un autre job après l’avoir buildée.
* ✅ Tu veux séparer le pipeline en **stages** (build → test → déploiement).
* ✅ Tu veux partager des images temporaires sans forcément les pousser dans un registre.
