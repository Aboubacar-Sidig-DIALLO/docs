# 🏷️ Named Contexts avec GitHub Actions

👉 Les **Named Contexts** (contextes nommés) permettent de définir des contextes de build supplémentaires et de les utiliser dans ton `Dockerfile` avec :

* `FROM name`
* ou `--from=name`.

⚡ Si le Dockerfile définit déjà un stage avec le même nom → il est **écrasé**.

💡 Utile dans GitHub Actions pour :

* réutiliser les résultats d’autres builds,
* **pinner** une image sur un tag précis (au lieu d’un `:latest` risqué).

***

### 📌 Exemple 1 : Pinner une image sur un tag

Au lieu de :

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
RUN echo "Hello World"
```

On la remplace par une version **fiable** :

```yaml
name: ci

on:
  push:

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build
        uses: docker/build-push-action@v6
        with:
          build-contexts: |
            alpine=docker-image://alpine:3.21
          tags: myimage:latest
```

✅ Ici, `alpine:3.21` est utilisé explicitement comme contexte, au lieu du `latest`.

***

### 🔁 Exemple 2 : Réutiliser une image entre étapes

Par défaut, avec le driver `docker-container` (utilisé par Buildx), les images **ne sont pas chargées automatiquement** dans le store Docker local.\
➡️ Mais avec un contexte nommé, tu peux réutiliser une image construite dans une étape précédente 👇

```yaml
# syntax=docker/dockerfile:1
FROM alpine
RUN echo "Hello World"

name: ci

on:
  push:

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      # Utilisation du driver docker
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
        with:
          driver: docker

      # 1️⃣ Construire une image de base
      - name: Build base image
        uses: docker/build-push-action@v6
        with:
          context: "{{defaultContext}}:base"
          load: true
          tags: my-base-image:latest

      # 2️⃣ Réutiliser l’image construite comme contexte
      - name: Build
        uses: docker/build-push-action@v6
        with:
          build-contexts: |
            alpine=docker-image://my-base-image:latest
          tags: myimage:latest
```

***

### 📦 Exemple 3 : Avec un registre local

⚠️ Problème : le driver `docker-container` est **isolé** et ne peut pas charger une image directement du store local.\
👉 Solution : utiliser un **registre local** pour pousser/puller l’image entre étapes.

```yaml
# syntax=docker/dockerfile:1
FROM alpine
RUN echo "Hello World"

name: ci

on:
  push:

jobs:
  docker:
    runs-on: ubuntu-latest
    services:
      registry:
        image: registry:2
        ports:
          - 5000:5000
    steps:
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
        with:
          driver-opts: network=host  # nécessaire pour pousser sur le registre local

      # 1️⃣ Construire et pousser l’image de base vers le registre local
      - name: Build base image
        uses: docker/build-push-action@v6
        with:
          context: "{{defaultContext}}:base"
          tags: localhost:5000/my-base-image:latest
          push: true

      # 2️⃣ Réutiliser l’image depuis le registre local
      - name: Build
        uses: docker/build-push-action@v6
        with:
          build-contexts: |
            alpine=docker-image://localhost:5000/my-base-image:latest
          tags: myimage:latest
```

***

### 🎯 Quand utiliser les Named Contexts ?

* ✅ Quand tu veux **pinner** une base image à une version précise.
* ✅ Quand tu veux **chaîner plusieurs builds** dans un workflow GitHub Actions.
* ✅ Quand tu veux **partager une image temporaire** via un registre local.
* ✅ Quand tu travailles avec des **multi-stages Dockerfile** où certains stages dépendent d’images custom.
