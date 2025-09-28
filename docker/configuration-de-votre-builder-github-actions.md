# ⚙️ Configuration de votre builder GitHub Actions

Cette page explique comment configurer vos instances **BuildKit** lorsque vous utilisez l’action **Setup Buildx**.

📍 _(Dans la doc officielle, des schémas illustrent la relation entre Buildx et BuildKit.)_

***

### 📌 Version pinning

Par défaut :

* l’action utilise la **dernière version disponible** de **Buildx** (le client de build),
* et la **dernière release de BuildKit** (le serveur de build).

👉 Vous pouvez cependant **verrouiller une version spécifique**.

* Exemple : **Buildx v0.10.0**

```yaml
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3
  with:
    version: v0.10.0
```

* Exemple : **BuildKit v0.11.0**

```yaml
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3
  with:
    driver-opts: image=moby/buildkit:v0.11.0
```

📍 _(Dans la doc officielle, une capture montre la différence entre versions récentes et versions épinglées.)_

***

### 📜 Logs du conteneur BuildKit

Pour afficher les logs du conteneur BuildKit avec le driver **docker-container** :

* Activez le **debug logging** dans GitHub Actions,
* ou passez l’option `--debug` à **buildkitd**.

Exemple :

```yaml
name: ci

on:
  push:

jobs:
  buildx:
    runs-on: ubuntu-latest
    steps:
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
        with:
          buildkitd-flags: --debug
      
      - name: Build
        uses: docker/build-push-action@v6
```

👉 Les logs apparaîtront à la fin du job.

<figure><img src="../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

***

### 🔧 Configuration de BuildKit Daemon

Vous pouvez fournir une configuration à votre builder **si vous utilisez le driver docker-container** (par défaut).

Deux options :

* `buildkitd-config-inline` (bloc TOML directement dans le workflow),
* `config` (fichier TOML dans votre dépôt).

***

#### 🌐 Exemple : configurer un _registry mirror_

```yaml
name: ci

on:
  push:

jobs:
  buildx:
    runs-on: ubuntu-latest
    steps:
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
        with:
          buildkitd-config-inline: |
            [registry."docker.io"]
              mirrors = ["mirror.gcr.io"]
```

***

#### ⚡ Exemple : limiter le parallélisme

* Avec un fichier `.github/buildkitd.toml` :

```toml
# .github/buildkitd.toml
[worker.oci]
  max-parallelism = 4
```

* Dans le workflow :

```yaml
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3
  with:
    config: .github/buildkitd.toml
```

📍 _(Une illustration compare un build avec parallélisme élevé vs limité sur une machine faible.)_

***

### 🌍 Ajouter des nœuds supplémentaires

Buildx peut exécuter des builds sur **plusieurs machines** → idéal pour :

* les **multi-plateformes** (ex. ARM64 + AMD64),
* la **distribution de la charge** sur différents hôtes.

Exemple avec `append` :

```yaml
name: ci

on:
  push:

jobs:
  buildx:
    runs-on: ubuntu-latest
    steps:
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
        with:
          driver: remote
          endpoint: tcp://oneprovider:1234
          append: |
            - endpoint: tcp://graviton2:1234
              platforms: linux/arm64
            - endpoint: tcp://linuxone:1234
              platforms: linux/s390x
        env:
          BUILDER_NODE_0_AUTH_TLS_CACERT: ${{ secrets.ONEPROVIDER_CA }}
          BUILDER_NODE_0_AUTH_TLS_CERT: ${{ secrets.ONEPROVIDER_CERT }}
          BUILDER_NODE_0_AUTH_TLS_KEY: ${{ secrets.ONEPROVIDER_KEY }}
          BUILDER_NODE_1_AUTH_TLS_CACERT: ${{ secrets.GRAVITON2_CA }}
          BUILDER_NODE_1_AUTH_TLS_CERT: ${{ secrets.GRAVITON2_CERT }}
          BUILDER_NODE_1_AUTH_TLS_KEY: ${{ secrets.GRAVITON2_KEY }}
          BUILDER_NODE_2_AUTH_TLS_CACERT: ${{ secrets.LINUXONE_CA }}
          BUILDER_NODE_2_AUTH_TLS_CERT: ${{ secrets.LINUXONE_CERT }}
          BUILDER_NODE_2_AUTH_TLS_KEY: ${{ secrets.LINUXONE_KEY }}
```

***

### 🔐 Authentification pour remote builders

#### 🔑 Via SSH

Configure une clé privée SSH dans ton runner GitHub :

```yaml
- name: Set up SSH
  uses: MrSquaare/ssh-setup-action@v3.1.0
  with:
    host: graviton2
    private-key: ${{ secrets.SSH_PRIVATE_KEY }}
    private-key-name: aws_graviton2

- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3
  with:
    endpoint: ssh://me@graviton2
```

***

#### 🔑 Via TLS

En utilisant `remote driver` + variables d’environnement :

```yaml
- name: Set up Docker Build
  uses: docker/setup-buildx-action@v3
  with:
    driver: remote
    endpoint: tcp://graviton2:1234
  env:
    BUILDER_NODE_0_AUTH_TLS_CACERT: ${{ secrets.GRAVITON2_CA }}
    BUILDER_NODE_0_AUTH_TLS_CERT: ${{ secrets.GRAVITON2_CERT }}
    BUILDER_NODE_0_AUTH_TLS_KEY: ${{ secrets.GRAVITON2_KEY }}
```

***

### 🛰️ Mode standalone

Si Docker CLI n’est pas installé sur le runner :\
👉 Buildx est invoqué directement (utile pour le driver Kubernetes).

```yaml
jobs:
  buildx:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
        with:
          driver: kubernetes
      
      - name: Build
        run: |
          buildx build .
```

***

### 🛠️ Builders isolés

Tu peux définir plusieurs builders dans un même workflow → pratique pour les **monorepos** où certains packages :

* nécessitent plus de puissance,
* ou des capacités matérielles spécifiques.

Exemple :

```yaml
jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Set up builder1
        uses: docker/setup-buildx-action@v3
        id: builder1
      
      - name: Set up builder2
        uses: docker/setup-buildx-action@v3
        id: builder2
      
      - name: Build against builder1
        uses: docker/build-push-action@v6
        with:
          builder: ${{ steps.builder1.outputs.name }}
          target: mytarget1
      
      - name: Build against builder2
        uses: docker/build-push-action@v6
        with:
          builder: ${{ steps.builder2.outputs.name }}
          target: mytarget2
```

***

### ✅ En résumé

* 🔧 **Pinning** → choisir une version spécifique de Buildx/BuildKit.
* 📜 **Logs** → activer `--debug` pour déboguer BuildKit.
* 🌐 **Config TOML** → registry mirror, parallélisme, etc.
* 🌍 **Multi-nœuds** → distribuer le build sur plusieurs hôtes/architectures.
* 🔑 **Auth** → SSH ou TLS pour remote builders.
* 🛰️ **Standalone** → exécuter Buildx sans Docker CLI.
* 🛠️ **Builders isolés** → utile pour monorepos complexes.
