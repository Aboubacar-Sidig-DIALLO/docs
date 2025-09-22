# ⚙️ Automatisation avec Docker Content Trust

Il est **très courant** d’intégrer Docker Content Trust (DCT) dans des systèmes d’automatisation existants (par exemple : CI/CD).\
Pour permettre aux outils d’interagir avec Docker et de **pousser du contenu signé automatiquement**, on utilise des **variables d’environnement**.

👉 Ces variables contrôlent le comportement du client Docker et du client **Notary**.

***

### 🔑 Variables côté Docker (CLI)

Lorsqu’on active DCT pour l’automatisation, on utilise :

* `DOCKER_CONTENT_TRUST=1`\
  Active la vérification et la signature des images par défaut.
* `DOCKER_CONTENT_TRUST_SERVER`\
  Permet de spécifier l’URL d’un **serveur Notary personnalisé**.\
  (Par défaut : Docker Hub Notary server → `https://notary.docker.io`)

Exemple :

```bash
export DOCKER_CONTENT_TRUST=1
export DOCKER_CONTENT_TRUST_SERVER=https://my-notary.company.com
```

***

### 🔐 Variables côté Notary (client Notary direct)

Quand on utilise directement **Notary CLI** (au lieu de `docker trust`), il existe un autre ensemble de variables :

* `NOTARY_URL` → serveur Notary à utiliser.
* `NOTARY_ROOT_PASSPHRASE` → passphrase de la **root key**.
* `NOTARY_TARGETS_PASSPHRASE` → passphrase de la **clé de signature des tags**.
* `NOTARY_DELEGATION_PASSPHRASE` → passphrase des **clés de délégation**.

Exemple pour automatiser sans saisie manuelle :

```bash
export NOTARY_URL=https://my-notary.company.com
export NOTARY_ROOT_PASSPHRASE=rootpass123
export NOTARY_TARGETS_PASSPHRASE=targetpass456
export NOTARY_DELEGATION_PASSPHRASE=delegatepass789
```

***

### 🚀 Exemple dans un pipeline CI/CD

Dans un pipeline GitLab CI, Jenkins ou GitHub Actions :

```yaml
steps:
  - name: Build image
    run: docker build -t registry.example.com/app:1 .

  - name: Push signed image
    env:
      DOCKER_CONTENT_TRUST: 1
      NOTARY_URL: https://my-notary.company.com
      NOTARY_ROOT_PASSPHRASE: ${{ secrets.ROOT_PASSPHRASE }}
      NOTARY_TARGETS_PASSPHRASE: ${{ secrets.TARGET_PASSPHRASE }}
    run: docker push registry.example.com/app:1
```

👉 Ici, le push est **automatiquement signé** grâce aux variables, sans interaction manuelle.

***

✅ **Résumé :**

* Utilise `DOCKER_CONTENT_TRUST` et `DOCKER_CONTENT_TRUST_SERVER` pour le client Docker.
* Utilise les variables `NOTARY_*` pour le client Notary.
* Intègre ces variables dans ton pipeline CI/CD pour **signer automatiquement** tes images.

## 🔑 Ajouter une clé privée de délégation

Lorsqu’on veut **automatiser** l’importation d’une clé privée de délégation dans le **trust store local Docker** (`~/.docker/trust/`), il faut fournir une **passphrase**.\
👉 Cette passphrase sera demandée **chaque fois que cette clé signe un tag**.

***

### ✅ Étapes

1. **Définir la passphrase dans une variable d’environnement** :

```bash
export DOCKER_CONTENT_TRUST_REPOSITORY_PASSPHRASE="mypassphrase123"
```

> ⚠️ Remarque :
>
> * Dans un pipeline CI/CD, tu ne stockes pas cette passphrase en clair, mais dans un gestionnaire de secrets (GitHub Actions secrets, GitLab CI variables, Vault, etc.).

***

2. **Importer la clé privée dans le trust store Docker** :

```bash
docker trust key load delegation.key --name jeff
```

Résultat attendu :

```
Loading key from "delegation.key"...
Successfully imported key from delegation.key
```

***

### 📂 Où sont stockées les clés ?

* **Localement** : `~/.docker/trust/private/`
* Chaque clé est chiffrée avec ta **passphrase**.
* La clé publique correspondante est ajoutée au serveur Notary.

***

### 🛠 Exemple d’automatisation CI/CD

```yaml
steps:
  - name: Import delegation key
    env:
      DOCKER_CONTENT_TRUST_REPOSITORY_PASSPHRASE: ${{ secrets.REPO_KEY_PASSPHRASE }}
    run: |
      echo "${{ secrets.DELEGATION_KEY }}" > delegation.key
      docker trust key load delegation.key --name ci-signer
      rm delegation.key
```

👉 Ici, la clé est récupérée depuis un secret CI/CD et chargée automatiquement, prête à signer des images.

## 🔑 Ajouter une clé publique de délégation

Une **clé publique de délégation** permet d’associer un **signer** (par ex. `jeff`) à un dépôt d’images signé.\
C’est cette étape qui donne à un utilisateur (ou un système CI/CD) le droit de **signer des tags** pour un dépôt donné.

***

### ✅ Étapes

#### 1. Exporter les passphrases nécessaires

* Si le **repository n’a jamais été initialisé** :\
  tu dois fournir la **passphrase de la clé root locale** (Root Key).

```bash
export DOCKER_CONTENT_TRUST_ROOT_PASSPHRASE="rootpassphrase123"
```

* Si le **repository existe déjà** :\
  seule la passphrase du **repository** est nécessaire :

```bash
export DOCKER_CONTENT_TRUST_REPOSITORY_PASSPHRASE="repopassphrase123"
```

***

#### 2. Ajouter la clé publique au dépôt signé

Commande :

```bash
docker trust signer add --key delegation.crt jeff registry.example.com/admin/demo
```

Explication :

* `--key delegation.crt` → chemin vers la clé publique de délégation
* `jeff` → nom du **signer**
* `registry.example.com/admin/demo` → le dépôt d’images cible

***

#### 3. Résultat attendu

```
Adding signer "jeff" to registry.example.com/admin/demo...
Initializing signed repository for registry.example.com/admin/demo...
Successfully initialized "registry.example.com/admin/demo"
Successfully added signer: registry.example.com/admin/demo
```

👉 Cela confirme que :

* Le dépôt est **initialisé** pour DCT (si ce n’était pas déjà fait).
* Le **signer `jeff`** a bien été ajouté.

***

### 🔐 Résumé visuel du workflow des clés

* **Clé root (offline)** : utilisée uniquement pour initialiser de nouveaux dépôts.
* **Clé de repository** : utilisée pour signer globalement le dépôt.
* **Clés de délégation (pub/priv)** : utilisées par les développeurs/CI pour signer les **tags spécifiques**.

## 🖊️ Signer une image avec DCT

#### 1. Exporter la passphrase de la clé de délégation

Lorsque tu as ajouté une clé privée de délégation (avec `docker trust key load`), une **passphrase** a été définie.\
Tu dois l’exporter pour que Docker puisse l’utiliser automatiquement :

```bash
export DOCKER_CONTENT_TRUST_REPOSITORY_PASSPHRASE="mypassphrase123"
```

***

#### 2. Signer une image

On utilise la commande suivante :

```bash
docker trust sign registry.example.com/admin/demo:1
```

Explication :

* `docker trust sign` → signe et pousse les métadonnées de confiance (trust metadata).
* `registry.example.com/admin/demo:1` → image + tag à signer (`demo:1`).

***

#### 3. Résultat attendu

```
Signing and pushing trust data for local image registry.example.com/admin/demo:1, may overwrite remote trust data
The push refers to repository [registry.example.com/admin/demo]
428c97da766c: Layer already exists
2: digest: sha256:1a6fd470b9ce10849be79e99529a88371dff60c60aab424c077007f6979b4812 size: 524
Signing and pushing trust metadata
Successfully signed registry.example.com/admin/demo:1
```

👉 Cela veut dire que :

* L’image `demo:1` a bien été poussée (si nécessaire).
* Les **métadonnées signées** (trust metadata) ont été envoyées au serveur Notary attaché au registre.
* L’image est maintenant **signée et vérifiable** par tous les clients qui activent DCT.

***

✅ **Résumé :**

1. Importer une clé de délégation (`docker trust key load`).
2. Exporter la passphrase (`DOCKER_CONTENT_TRUST_REPOSITORY_PASSPHRASE`).
3. Signer avec `docker trust sign`.
4. L’image est prête à être tirée uniquement par des clients qui exigent DCT.

## 🏗️ Construire avec Docker Content Trust

#### 1. Activer DCT

Avant toute chose, il faut activer la vérification des signatures :

```bash
export DOCKER_CONTENT_TRUST=1
```

👉 Cela oblige Docker à n’utiliser que des images **signées** (ou déjà présentes localement).

***

#### 2. Exemple de Dockerfile

```dockerfile
# syntax=docker/dockerfile:1
FROM docker/trusttest:latest
RUN echo
```

Ici :

* `FROM docker/trusttest:latest` → base de l’image.
* `RUN echo` → instruction simple (juste pour le test).

***

#### 3. Construction réussie avec une image signée

Si `docker/trusttest:latest` est signé (et que DCT est activé) :

```bash
docker build -t docker/trusttest:testing .
```

Résultat attendu :

```
Using default tag: latest
latest: Pulling from docker/trusttest
b3dbab3810fc: Pull complete
a9539b34a6ab: Pull complete
Digest: sha256:d149ab53f871
```

👉 L’image est construite normalement, car la base (`latest`) est **signée**.

***

#### 4. Construction échouée avec une image non signée

Si le `FROM` pointe vers une image **non signée** (par exemple `notrust`), la build échoue :

```bash
docker build -t docker/trusttest:testing .
```

Résultat :

```
unable to process Dockerfile: No trust data for notrust
```

👉 Cela empêche l’utilisation d’images non fiables.

***

✅ **Résumé :**

* `DOCKER_CONTENT_TRUST=1` force la sécurité dès la build.
* Seules les **images signées** (ou locales) peuvent être utilisées dans un `Dockerfile`.
* Une build échoue si la base n’a pas de métadonnées de confiance.
