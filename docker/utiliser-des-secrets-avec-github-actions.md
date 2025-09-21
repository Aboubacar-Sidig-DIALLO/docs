# 🔑 Utiliser des secrets avec GitHub Actions

Un **secret** est une information sensible utilisée pendant un build :

* un **mot de passe**,
* un **token API**,
* une **clé SSH**.

👉 Docker Build prend en charge deux formes de secrets :

1. **Secret mounts** → montés comme fichiers dans le conteneur de build (par défaut sous `/run/secrets`).
2. **SSH mounts** → ajoutent des sockets ou clés SSH dans le conteneur de build.

***

### 📂 Secret mounts

#### 1️⃣ Exemple avec un **Dockerfile**

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
RUN --mount=type=secret,id=github_token,env=GITHUB_TOKEN ...
```

👉 Ici, le secret est nommé **`github_token`**.

***

#### 2️⃣ Exemple de workflow GitHub Actions

```yaml
name: ci

on:
  push:

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build
        uses: docker/build-push-action@v6
        with:
          platforms: linux/amd64,linux/arm64
          tags: user/app:latest
          secrets: |
            "github_token=${{ secrets.GITHUB_TOKEN }}"
```

👉 Ici, le secret GitHub Actions **`GITHUB_TOKEN`** est exposé au build comme fichier secret.

***

#### ℹ️ Notes utiles

* Tu peux aussi exposer un **fichier secret** :

```yaml
secret-files: |
  "MY_SECRET=./secret.txt"
```

* Si ton secret est **multi-lignes** (ex. clé GPG, JSON…), tu dois l’entourer de **guillemets** :

```yaml
secrets: |
  "MYSECRET=${{ secrets.GPG_KEY }}"
  GIT_AUTH_TOKEN=abcdefghi,jklmno=0123456789
  "MYSECRET=aaaaaaaa
  bbbbbbb
  ccccccccc"
  FOO=bar
  "EMPTYLINE=aaaa

  bbbb
  ccc"
  "JSON_SECRET={""key1"":""value1"",""key2"":""value2""}"
```

➡️ Exemple de rendu final :

| Key              | Value                                          |
| ---------------- | ---------------------------------------------- |
| `MYSECRET`       | \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* |
| `GIT_AUTH_TOKEN` | `abcdefghi,jklmno=0123456789`                  |
| `MYSECRET`       | `aaaaaaaa\nbbbbbbb\nccccccccc`                 |
| `FOO`            | `bar`                                          |
| `EMPTYLINE`      | `aaaa\n\nbbbb\nccc`                            |
| `JSON_SECRET`    | `{"key1":"value1","key2":"value2"}`            |

⚠️ Attention : les **guillemets** dans JSON nécessitent une **double échappement** (`""`).

***

### 🔐 SSH mounts

Les **SSH mounts** permettent de s’authentifier avec des serveurs SSH, par exemple pour :

* faire un **git clone** d’un dépôt privé,
* télécharger des dépendances depuis un repo privé.

***

#### 1️⃣ Exemple Dockerfile avec SSH

```dockerfile
# syntax=docker/dockerfile:1

ARG GO_VERSION="1.24"

FROM golang:${GO_VERSION}-alpine AS base
ENV CGO_ENABLED=0
ENV GOPRIVATE="github.com/foo/*"
RUN apk add --no-cache file git rsync openssh-client
RUN mkdir -p -m 0700 ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts
WORKDIR /src

FROM base AS vendor
# Vérifie que la clé SSH est chargée
RUN --mount=type=ssh <<EOT
  set -e
  echo "Setting Git SSH protocol"
  git config --global url."git@github.com:".insteadOf "https://github.com/"
  (
    set +e
    ssh -T git@github.com
    if [ ! "$?" = "1" ]; then
      echo "No GitHub SSH key loaded exiting..."
      exit 1
    fi
  )
EOT
# Télécharge les modules Go
RUN --mount=type=bind,target=. \
    --mount=type=cache,target=/go/pkg/mod \
    --mount=type=ssh \
    go mod download -x

FROM vendor AS build
RUN --mount=type=bind,target=. \
    --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache \
    go build ...
```

***

#### 2️⃣ Workflow GitHub Actions avec SSH

On utilise une action tierce **`MrSquaare/ssh-setup-action`** pour initialiser la clé SSH sur le runner GitHub.

```yaml
name: ci

on:
  push:

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Set up SSH
        uses: MrSquaare/ssh-setup-action@2d028b70b5e397cf8314c6eaea229a6c3e34977a # v3.1.0
        with:
          host: github.com
          private-key: ${{ secrets.SSH_GITHUB_PPK }}
          private-key-name: github-ppk

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          ssh: default
          push: true
          tags: user/app:latest
```

👉 Ici :

* Le secret GitHub **`SSH_GITHUB_PPK`** contient ta clé privée.
* L’action l’ajoute dans l’agent SSH du runner.
* Docker Build utilise directement le socket SSH (`SSH_AUTH_SOCK`).

***

### 📌 En résumé

* **Secret mounts** → secrets injectés sous forme de fichiers `/run/secrets/*`.
* **SSH mounts** → accès SSH sécurisé pour cloner ou récupérer des dépendances privées.
* ⚠️ Toujours utiliser **GitHub Secrets** pour stocker les infos sensibles.
* ✅ Combine avec `build-push-action` ou `bake-action` pour un workflow complet CI/CD sécurisé.
