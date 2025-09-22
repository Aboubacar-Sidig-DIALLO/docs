---
description: ✅ Prérequis  Docker Compose 2.34.0 ou plus récent
---

# 📦 Packager et déployer des applications Docker Compose comme artefacts OCI

Depuis cette version, Docker Compose prend en charge les **artefacts OCI** (_Open Container Initiative_).

👉 Cela signifie que tu peux :

* **packager** ton application Compose (tes fichiers `compose.yaml` + configuration associée)
* **la distribuer** via un registre de conteneurs (Docker Hub, GHCR, Harbor, etc.)
* **la déployer** directement depuis ce registre

Ainsi, tes fichiers Compose sont stockés **au même endroit que tes images Docker**, ce qui facilite :

* la **gestion des versions**
* le **partage** entre équipes
* le **déploiement cohérent** sur différents environnements

***

### 🚀 Pourquoi utiliser OCI pour Compose ?

* 📂 **Centralisation** → tes images et ton fichier Compose sont regroupés dans le registre.
* 🔄 **Versioning** → tu peux versionner ton application multi-conteneurs comme une seule unité.
* 🤝 **Collaboration** → plus facile de partager ton stack complet entre plusieurs équipes/projets.
* ⚡ **Déploiement simplifié** → un simple `docker compose up` depuis l’artefact suffit.

***

### 📝 Exemple concret

1. **Publier ton fichier Compose en tant qu’artefact OCI** :

```bash
docker compose push oci://docker.io/username/my-compose-app:1.0.0
```

2. **Déployer directement depuis le registre** :

```bash
docker compose -f oci://docker.io/username/my-compose-app:1.0.0 up -d
```

👉 Ici :

* `oci://` indique que tu utilises un artefact OCI.
* `docker.io/username/my-compose-app:1.0.0` → référence de ton package dans le registre.

***

### ✅ Cas d’usage typiques

* Partager un **stack complet** (ex. `frontend + backend + db`) avec toute l’équipe.
* Versionner ton application Compose **comme une image Docker**.
* Déployer rapidement ton projet sur un nouveau serveur ou un environnement CI/CD.

## 🚀 Publier une application Docker Compose comme artefact OCI

Docker Compose (>= **2.34.0**) permet de transformer ton fichier `compose.yaml` (et ses éventuelles extensions) en **artefact OCI** puis de le pousser dans un **registre conforme OCI** (Docker Hub, GHCR, Harbor, etc.).

Cela simplifie énormément le **partage et le déploiement** de ton stack.

***

### ✅ Étapes générales

1.  **Se placer dans le projet Compose**

    ```bash
    cd mon-projet-compose
    ```

    (ou utiliser `-f` si ton fichier n’est pas dans ce répertoire)
2.  **S’authentifier auprès d’un registre**\
    Exemple avec Docker Hub :

    ```bash
    docker login
    ```
3.  **Publier l’application**

    ```bash
    docker compose publish username/my-compose-app:latest
    ```

    👉 Cela crée un artefact OCI et le pousse vers `docker.io/username/my-compose-app:latest`.
4.  **Avec plusieurs fichiers Compose** (par ex. base + prod) :

    ```bash
    docker compose -f compose-base.yml -f compose-production.yml publish username/my-compose-app:latest
    ```

***

### ⚙️ Options avancées

* `--oci-version` → spécifie la version OCI (par défaut autodétectée).
* `--resolve-image-digests` → convertit les tags en digests immuables (plus sûr et reproductible).
* `--with-env` → inclut les variables d’environnement dans l’artefact.

⚠️ **Attention : risques de fuite de secrets**\
Si tu as des variables sensibles (`AWS_ACCESS_KEY_ID`, `GITHUB_TOKEN`, `PRIVATE_KEY`, etc.), Compose t’affiche un avertissement et te demande confirmation avant publication :

```bash
you are about to publish sensitive data within your OCI artifact.
please double check that you are not leaking sensitive data
AWS Client ID
"services.serviceA.environment.AWS_ACCESS_KEY_ID": xxxxxxxxxx
AWS Secret Key
"services.serviceA.environment.AWS_SECRET_ACCESS_KEY": xxxx
...
Are you ok to publish these sensitive data? [y/N]:
```

***

### 🔐 Bonnes pratiques

* ⚠️ **Ne publie jamais de secrets** dans ton `compose.yaml` → utilise plutôt les **secrets Compose**.
* ✅ Utilise des **fichiers spécifiques par environnement** (`compose.dev.yml`, `compose.prod.yml`) pour isoler les configs.
* 🔄 Active `--resolve-image-digests` en prod pour plus de sécurité et de stabilité.

***

👉 Ensuite, tu peux **déployer depuis l’artefact OCI** avec un simple :

```bash
docker compose -f oci://docker.io/username/my-compose-app:latest up -d
```

## ⚠️ Limitations de la publication Compose en artefacts OCI

Lorsque tu utilises `docker compose publish`, certaines configurations **ne sont pas supportées** :

#### ❌ Cas interdits

1.  **Services avec des bind mounts**\
    Exemple non supporté :

    ```yaml
    services:
      web:
        image: nginx
        volumes:
          - ./code:/app   # ❌ bind mount local → interdit
    ```
2.  **Services définis uniquement avec une section `build`**\
    Exemple non supporté :

    ```yaml
    services:
      app:
        build: .
        # pas d'image → ❌ impossible à publier
    ```
3.  **Inclure des fichiers locaux via `include`**\
    Exemple interdit :

    ```yaml
    include:
      - ./common-services.yaml   # ❌ fichier local → interdit
    ```

    👉 En revanche, les **includes distants** (via `oci://` ou `git://`) sont supportés.\
    Pour contourner cette limite, il faut **publier aussi les fichiers inclus** puis les référencer en remote :

    ```yaml
    include:
      - oci://docker.io/username/common-services:latest
    ```

***

✅ **En résumé** :

* Les **volumes bindés au host** → non supportés.
* Les services **uniquement buildés (sans image publiée)** → non supportés.
* Les **includes locaux** → non supportés (il faut les publier puis les référencer en **remote**).

## ▶️ Démarrer une application Docker Compose depuis un artefact OCI

Docker Compose permet de lancer directement une application stockée sous forme **d’artefact OCI** dans un registre (comme Docker Hub, GHCR, ou un registre privé).

***

### 🔹 Commande de base

On utilise le **préfixe `oci://`** avec l’option `-f` (`--file`) pour indiquer que le fichier `compose.yaml` doit être récupéré depuis un registre :

```bash
docker compose -f oci://docker.io/username/my-compose-app:latest up
```

👉 Ici :

* `oci://` → indique à Docker d’aller chercher le Compose dans un registre OCI.
* `docker.io/username/my-compose-app:latest` → référence de l’artefact publié.
* `up` → lance l’application normalement, comme si le `compose.yaml` était local.

***

### 🔹 Exemple pratique

1.  Une équipe publie son application en OCI avec :

    ```bash
    docker compose publish username/my-compose-app:latest
    ```
2.  Un autre utilisateur peut ensuite la lancer directement :

    ```bash
    docker compose -f oci://docker.io/username/my-compose-app:latest up -d
    ```
3. Les services définis dans cet artefact sont déployés **exactement comme s’ils étaient déclarés dans un fichier local**.

***

✅ **Avantage** : tu n’as plus besoin de cloner ou copier un `compose.yaml` → il suffit de connaître sa référence OCI.\
⚠️ **Attention** : les limitations vues avant s’appliquent (pas de bind mounts, pas de services uniquement `build`, pas d’`include` local).

## 🛠️ Dépannage lors de l’exécution d’une application Compose depuis un artefact OCI

Quand tu exécutes une application à partir d’un artefact **OCI**, Docker Compose peut afficher des **avertissements et demandes de confirmation** pour éviter de lancer une application malveillante à ton insu.

***

### 🔎 Vérification des variables utilisées

Compose liste toutes les **variables d’interpolation** détectées dans la configuration, avec leur **valeur**, leur **source** (ligne de commande, `.env`, défaut, etc.), et s’il s’agit d’une variable **requise** ou non.

Exemple :

```bash
$ REGISTRY=myregistry.com docker compose -f oci://docker.io/username/my-compose-app:latest up
```

Sortie :

```
Found the following variables in configuration:
VARIABLE     VALUE                SOURCE        REQUIRED    DEFAULT
REGISTRY     myregistry.com      command-line   yes         
TAG          v1.0                environment    no          latest
DOCKERFILE   Dockerfile          default        no          Dockerfile
API_KEY      <unset>             none           no          
```

👉 Compose te demande ensuite :

```
Do you want to proceed with these variables? [Y/n]:
```

***

### ⚠️ Vérification des ressources distantes

Si ton application OCI **inclut d’autres sources distantes** (par ex. via `include`), Compose t’avertit :

```
Warning: This Compose project includes files from remote sources:
- oci://registry.example.com/stack:latest
Remote includes could potentially be malicious. Make sure you trust the source.
Do you want to continue? [y/N]:
```

***

### 📂 Où sont stockées les ressources téléchargées ?

Si tu confirmes, Compose **télécharge et met en cache** l’artefact dans un répertoire local, par exemple :

```
Your compose stack "oci://registry.example.com/stack:latest" is stored in "~/Library/Caches/docker-compose/964e715660d6f6c3b384e05e7338613795f7dcd3613890cfa57e3540353b9d6d"
```

***

### 🚀 Exécution non-interactive

Si tu veux **éviter les prompts** (utile en CI/CD ou automatisation), tu peux ajouter l’option `-y` (`--yes`) :

```bash
docker compose publish -y username/my-compose-app:latest
```

Cela publie directement ton artefact **sans demander confirmation**.
