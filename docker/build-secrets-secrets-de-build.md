# 🔐 Build secrets (Secrets de build)

### 🎯 Qu’est-ce qu’un _build secret_ ?

Un **build secret** est toute information **sensible** utilisée au cours du processus de build de votre application.\
Exemples courants :

* 🔑 un mot de passe,
* 🛡️ un token d’API,
* 📜 une clé privée,
* etc.

***

### ⚠️ Mauvaises pratiques

Il peut être tentant d’utiliser des **arguments de build** (`--build-arg`) ou des **variables d’environnement** pour transmettre ces secrets.

👉 **Problème** : ces informations sont **persistées dans l’image finale** ❌.\
Elles deviennent donc accessibles à quiconque inspecte l’image, ce qui compromet la sécurité.

***

### ✅ Bonne pratique : les _secret mounts_ et _SSH mounts_

Docker recommande d’utiliser :

* 🔐 **Secret mounts** : permettent de monter temporairement vos secrets pendant le build, sans qu’ils soient stockés dans l’image finale.
* 🔑 **SSH mounts** : exposent une clé SSH de manière sécurisée, utile pour interagir avec des dépôts Git privés ou des services nécessitant une authentification SSH.

👉 Ces mécanismes garantissent que vos secrets restent **éphémères et protégés** lors du build.

***

### 🗺️ Résumé rapide

* 📌 Un _build secret_ = information sensible (mot de passe, token, clé…).
* ❌ **Ne pas utiliser** `ARG` ou variables d’environnement → secrets exposés dans l’image finale.
* ✅ **Utiliser** les **secret mounts** ou **SSH mounts** → sécurité renforcée, secrets non persistés.

## 🗂️ Types de _build secrets_

Il existe plusieurs façons de transmettre des secrets à vos builds selon vos besoins.

***

### 🔐 Secret mounts (montages de secrets)

* Ce sont des montages **généralistes** qui permettent de **passer des secrets dans le build**.
* Un secret mount prend un **secret fourni par le client de build** et le rend disponible **temporairement** à l’intérieur du conteneur de build.
* ⏳ Le secret n’existe que le temps de l’instruction de build concernée, puis disparaît.
* ✅ Cas d’usage typique :
  * Le build doit communiquer avec un **serveur privé d’artifacts** ou une **API protégée**.

***

### 🔑 SSH mounts (montages SSH)

* Ce sont des montages **spécialisés** destinés à rendre accessibles des **sockets SSH ou des clés SSH** à l’intérieur du build.
* Très utilisés lorsqu’il faut récupérer des **dépôts Git privés** pendant un build.
* 🔒 Les clés ne sont pas persistées dans l’image finale → sécurité garantie.

***

### 🌍 Git authentication for remote contexts (authentification Git pour contextes distants)

* Dans le cas où vous construisez à partir d’un **contexte Git distant** qui est **privé**, Docker Buildx fournit un ensemble de **secrets pré-définis**.
* Ces secrets sont appelés **“pre-flight secrets”** ✈️ :
  * Ils ne sont **pas consommés directement dans les instructions de build**.
  * Ils servent uniquement à fournir au builder les **identifiants nécessaires** pour récupérer le contexte Git distant.

***

### 🗺️ Résumé rapide

* 🔐 **Secret mounts** → secrets temporaires (API, serveurs privés).
* 🔑 **SSH mounts** → clés/agents SSH pour accès Git privé.
* 🌍 **Git authentication (remote contexts)** → secrets “pre-flight” pour donner accès au builder avant même le début du build.

## 🛠️ Utilisation des _build secrets_

Pour les **secret mounts** et les **SSH mounts**, l’utilisation des secrets se fait en **deux étapes** :

***

### 1️⃣ Passer le secret à la commande `docker build`

Lorsque vous lancez un build, vous devez **transmettre le secret** au builder.\
Cela se fait avec l’option :

* `--secret` dans **`docker build`** (CLI)
* ou l’option équivalente avec **Bake**

#### 📌 Exemple (CLI)

```bash
docker build --secret id=aws,src=$HOME/.aws/credentials .
```

👉 Ici :

* `id=aws` → identifiant du secret (nom utilisé dans le Dockerfile).
* `src=$HOME/.aws/credentials` → chemin du fichier contenant le secret (credentials AWS).

***

### 2️⃣ Consommer le secret dans le `Dockerfile`

Une fois le secret transmis au build, vous devez le **monter dans une instruction** pour qu’il soit accessible.\
Cela se fait avec l’option :

```dockerfile
RUN --mount=type=secret,id=aws \
    AWS_SHARED_CREDENTIALS_FILE=/run/secrets/aws \
    aws s3 cp ...
```

👉 Explication :

* `--mount=type=secret,id=aws` → monte le secret identifié comme `aws`.
* Le secret est disponible **temporairement** dans le conteneur sous `/run/secrets/aws`.
* On configure la variable `AWS_SHARED_CREDENTIALS_FILE` pour pointer vers ce fichier.
* Ensuite, la commande `aws s3 cp ...` peut utiliser ces credentials.

***

### 🗺️ Résumé rapide

1. **Transmettre le secret au build** avec `--secret id=...,src=...`.
2. **Utiliser le secret dans le Dockerfile** avec `RUN --mount=type=secret,id=...`.
3. ✅ Les secrets sont éphémères → jamais persistés dans l’image finale.

## 🔐 Secret mounts

### 🎯 Rôle

Les **secret mounts** exposent vos secrets dans les conteneurs de build :

* soit sous forme de **fichiers**,
* soit sous forme de **variables d’environnement**.

👉 Cela permet de passer des informations sensibles à vos builds (API tokens, mots de passe, clés SSH…), **sans les stocker dans l’image finale**.

***

### 📌 Sources possibles des secrets

Un secret peut provenir de :

* 📂 un **fichier**,
* 🌍 une **variable d’environnement**.

Lorsque vous utilisez le **CLI** (`docker build`) ou **Bake** :

* Le type (`file` ou `env`) est généralement **détecté automatiquement**.
* Mais vous pouvez aussi le spécifier explicitement avec :
  * `type=file`
  * `type=env`

***

### 🧪 Exemple 1 — Monter une variable d’environnement comme fichier

Ici, on monte la variable d’environnement `KUBECONFIG` dans le conteneur de build,\
sous forme de fichier `/run/secrets/kube` (secret ID = `kube`) :

```bash
docker build --secret id=kube,env=KUBECONFIG .
```

***

### 🧪 Exemple 2 — Monter une variable d’environnement automatiquement

Si vous utilisez un secret depuis une **variable d’environnement**, vous pouvez **omettre le paramètre `env`**.

👉 Dans ce cas, Docker crée automatiquement un fichier dans `/run/secrets/<NOM_DE_LA_VARIABLE>`.

Exemple avec `API_TOKEN` :

```bash
docker build --secret id=API_TOKEN .
```

Résultat :

* La valeur de la variable `API_TOKEN` est montée dans le conteneur,
*   et disponible dans le fichier :

    ```
    /run/secrets/API_TOKEN
    ```

***

### 🗺️ Résumé rapide

* 🔐 **Secret mounts** = secrets montés comme fichiers/variables d’env.
* 📂 Source possible : fichier ou variable d’env.
* 🧪 Exemple 1 → `--secret id=kube,env=KUBECONFIG` → monté dans `/run/secrets/kube`.
* 🧪 Exemple 2 → `--secret id=API_TOKEN` → monté dans `/run/secrets/API_TOKEN`.

## 🎯 Target — Consommer un secret dans un `Dockerfile`

### 📌 Rappel

Par défaut, lorsqu’un secret est consommé dans un `Dockerfile`, il est monté comme **fichier** dans le conteneur de build.

*   📂 Chemin par défaut :

    ```
    /run/secrets/<id>
    ```

👉 Vous pouvez personnaliser la façon dont les secrets sont montés en utilisant les options **`target`** et **`env`** avec le flag `--mount` dans vos instructions `RUN`.

***

### 🧪 Exemple 1 — Montage simple par défaut

On prend le secret avec l’ID `aws`, monté par défaut dans :

```
/run/secrets/aws
```

```dockerfile
RUN --mount=type=secret,id=aws \
    AWS_SHARED_CREDENTIALS_FILE=/run/secrets/aws \
    aws s3 cp ...
```

***

### 🧪 Exemple 2 — Changer le chemin du fichier (option `target`)

On monte le secret `aws` dans un chemin personnalisé :

```dockerfile
RUN --mount=type=secret,id=aws,target=/root/.aws/credentials \
    aws s3 cp ...
```

👉 Ici, le secret est accessible comme s’il s’agissait du fichier standard `credentials` d’AWS.

***

### 🧪 Exemple 3 — Monter un secret comme **variable d’environnement** (option `env`)

Au lieu d’un fichier, on peut exposer le secret directement comme **variable d’environnement** dans le conteneur :

```dockerfile
RUN --mount=type=secret,id=aws-key-id,env=AWS_ACCESS_KEY_ID \
    --mount=type=secret,id=aws-secret-key,env=AWS_SECRET_ACCESS_KEY \
    --mount=type=secret,id=aws-session-token,env=AWS_SESSION_TOKEN \
    aws s3 cp ...
```

👉 Chaque secret est ici injecté directement comme **env var** attendue par l’outil AWS CLI.

***

### 🧪 Exemple 4 — Utiliser `target` + `env` en même temps

Vous pouvez **combiner** les deux options pour qu’un même secret soit monté :

* comme **fichier**,
* et comme **variable d’environnement**.

Cela donne plus de flexibilité selon les besoins de vos outils dans le build.

***

### 🗺️ Résumé rapide

* 📂 Par défaut → secret monté dans `/run/secrets/<id>`.
* 🎯 `target` → changer le chemin du fichier cible.
* 🌍 `env` → exposer un secret comme variable d’environnement.
* 🔀 `target + env` → possible de combiner les deux.

## 🔑 SSH mounts

### 🎯 Rôle

Si l’identifiant ou la clé que vous souhaitez utiliser pendant votre build est un **socket d’agent SSH** ou une **clé SSH**, vous devez utiliser un **SSH mount** au lieu d’un _secret mount_.

👉 Cas d’usage le plus courant :\
📂 **Cloner un dépôt Git privé** directement pendant le build.

***

### 🧪 Exemple — Cloner un dépôt Git privé avec un SSH mount

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
ADD git@github.com:me/myprivaterepo.git /src/
```

👉 Ici, le dépôt privé est cloné directement depuis GitHub, grâce au montage SSH.

***

### ⚙️ Passer un socket SSH au build

Pour transmettre votre **socket SSH** au builder, vous utilisez :

* le flag **`--ssh`** avec `docker buildx build`
* ou son équivalent dans **Bake**

#### Exemple (CLI) :

```bash
docker buildx build --ssh default .
```

👉 Ici, `default` correspond au **socket SSH par défaut** exposé par votre environnement.

***

### 🗺️ Résumé rapide

* 🔑 **SSH mounts** → pour exposer des sockets ou clés SSH dans un build.
* 📌 Cas typique : cloner un dépôt Git privé.
* ⚙️ Commande → `docker buildx build --ssh <socket> .`
* ✅ Sécurisé : les clés/agents ne sont pas persistés dans l’image finale.

## 🌍 Authentification Git pour contextes distants

Quand vous utilisez un **contexte distant Git privé** (par exemple un dépôt privé comme source de votre build ou bien des `ADD`/`COPY` pointant vers un dépôt Git), BuildKit propose deux _secrets prédéfinis_ pour gérer l’authentification HTTP :

* `GIT_AUTH_TOKEN`
* `GIT_AUTH_HEADER`\
  [Docker Documentation](https://docs.docker.com/build/building/secrets/?utm_source=chatgpt.com)

***

### 🔐 Scénarios d’usage

Ces secrets sont utiles dans deux cas de figure fréquents :

1. **Utiliser un dépôt Git privé comme contexte de build** (par exemple : `docker build https://…` avec un dépôt privé).
2. **Récupérer des dépôts Git privés** via une instruction `ADD` ou `COPY` dans le Dockerfile.\
   [Docker Documentation](https://docs.docker.com/build/building/secrets/?utm_source=chatgpt.com)

***

### 🧪 Exemple concret

Supposons que vous avez un projet GitLab privé à l’adresse :

```
https://gitlab.com/example/todo-app.git
```

Si vous lancez :

```bash
docker build https://gitlab.com/example/todo-app.git
```

Vous obtiendrez une erreur du type :

> `fatal: could not read Username for 'https://gitlab.com': terminal prompts disabled`\
> [Docker Documentation](https://docs.docker.com/build/building/secrets/?utm_source=chatgpt.com)

Car le builder ne dispose pas des droits pour accéder au dépôt Git privé.

***

### ✅ Solution : fournir le token d’authentification

1. Créez un **token GitLab** valide (ou pour le service Git privé que vous utilisez).
2. Chargez ce token dans `GIT_AUTH_TOKEN` (variable d’environnement).
3. Passez ce secret au build via l’option `--secret id=GIT_AUTH_TOKEN`.

Par exemple :

```bash
GIT_AUTH_TOKEN=$(cat gitlab-token.txt) docker build \
  --secret id=GIT_AUTH_TOKEN \
  https://gitlab.com/example/todo-app.git
```

Ainsi, BuildKit peut authentifier la requête HTTP pour accéder au dépôt Git.\
[Docker Documentation](https://docs.docker.com/build/building/secrets/?utm_source=chatgpt.com)

***

### 🛠️ Fonctionne aussi avec ADD/COPY

Si, dans votre Dockerfile, vous faites :

```dockerfile
FROM alpine
ADD https://gitlab.com/example/todo-app.git /src
```

Le secret `GIT_AUTH_TOKEN` sera aussi utilisé pour permettre à l’instruction `ADD` de récupérer le dépôt privé.\
[Docker Documentation](https://docs.docker.com/build/building/secrets/?utm_source=chatgpt.com)

***

### ⚙️ Schéma d’authentification HTTP

*   Par défaut, l’authentification se fait en mode **Bearer** :

    ```
    Authorization: Bearer <GIT_AUTH_TOKEN>
    ```

    [Docker Documentation](https://docs.docker.com/build/building/secrets/?utm_source=chatgpt.com)
*   Si vous voulez utiliser le mode **Basic** (nom d’utilisateur + mot de passe ou token), vous pouvez définir aussi le secret `GIT_AUTH_HEADER` à `basic`.\
    Exemple :

    ```bash
    export GIT_AUTH_TOKEN=$(cat gitlab-token.txt)
    export GIT_AUTH_HEADER=basic
    docker build \
      --secret id=GIT_AUTH_TOKEN \
      --secret id=GIT_AUTH_HEADER \
      https://gitlab.com/example/todo-app.git
    ```

    [Docker Documentation](https://docs.docker.com/build/building/secrets/?utm_source=chatgpt.com)

***

### 🌐 Hôtes multiples

Vous pouvez aussi utiliser ces secrets **par hôte**, pour couvrir plusieurs dépôts Git privés différents, chacun avec ses propres credentials.

Exemple :

```bash
export GITLAB_TOKEN=$(cat gitlab-token.txt)
export GERRIT_TOKEN=$(cat gerrit-credentials.txt)
export GERRIT_SCHEME=basic

docker build \
  --secret id=GIT_AUTH_TOKEN.gitlab.com,env=GITLAB_TOKEN \
  --secret id=GIT_AUTH_TOKEN.gerrit.internal.example,env=GERRIT_TOKEN \
  --secret id=GIT_AUTH_HEADER.gerrit.internal.example,env=GERRIT_SCHEME \
  https://gitlab.com/example/todo-app.git
```

Dans cet exemple :

* `GIT_AUTH_TOKEN.gitlab.com` → token pour gitlab.com
* `GIT_AUTH_TOKEN.gerrit.internal.example` → token pour un hôte Gerrit interne
* `GIT_AUTH_HEADER.gerrit.internal.example` → schéma d’authentification pour Gerrit (ici `basic`)\
  [Docker Documentation](https://docs.docker.com/build/building/secrets/?utm_source=chatgpt.com)

***

### ⚠️ À retenir

* Ces secrets sont des **secrets “pre-flight”** : ils ne sont pas consommés **dans** vos instructions de build (RUN, etc.), mais servent pour l’**étape préalable** de récupération du contexte distant. [Docker Documentation](https://docs.docker.com/build/building/secrets/?utm_source=chatgpt.com)
* Ils permettent de sécuriser l’accès aux dépôts privés sans exposer vos credentials dans l’image ou Dockerfile.

## 🔐 Schéma d’authentification HTTP

Par défaut, l’authentification Git via HTTP utilise le schéma **Bearer** :

```
Authorization: Bearer <GIT_AUTH_TOKEN>
```

Si vous devez utiliser le schéma **Basic** (nom d’utilisateur + mot de passe), vous pouvez définir le _build secret_ `GIT_AUTH_HEADER` avec la valeur `basic`.\
Exemple :

```bash
export GIT_AUTH_TOKEN=$(cat gitlab-token.txt)
export GIT_AUTH_HEADER=basic

docker build \
  --secret id=GIT_AUTH_TOKEN \
  --secret id=GIT_AUTH_HEADER \
  https://gitlab.com/example/todo-app.git
```

> ℹ️ BuildKit ne prend actuellement en charge que les schémas **Bearer** et **Basic**. [Docker Documentation](https://docs.docker.com/build/building/secrets/?utm_source=chatgpt.com)

***

## 🌐 Plusieurs hôtes (Multiple hosts)

Vous pouvez définir `GIT_AUTH_TOKEN` et `GIT_AUTH_HEADER` **individuellement pour chaque hôte**.\
Cela permet d’utiliser des paramètres d’authentification différents selon le domaine Git (gitlab.com, gerrit.internal, etc.).

Pour cela, on ajoute le nom d’hôte comme **suffixe** à l’identifiant du secret.

**Exemple :**

```bash
export GITLAB_TOKEN=$(cat gitlab-token.txt)
export GERRIT_TOKEN=$(cat gerrit-username-password.txt)
export GERRIT_SCHEME=basic

docker build \
  --secret id=GIT_AUTH_TOKEN.gitlab.com,env=GITLAB_TOKEN \
  --secret id=GIT_AUTH_TOKEN.gerrit.internal.example,env=GERRIT_TOKEN \
  --secret id=GIT_AUTH_HEADER.gerrit.internal.example,env=GERRIT_SCHEME \
  https://gitlab.com/example/todo-app.git
```

* `GIT_AUTH_TOKEN.gitlab.com` → token pour **gitlab.com**
* `GIT_AUTH_TOKEN.gerrit.internal.example` → token pour **gerrit.internal.example**
* `GIT_AUTH_HEADER.gerrit.internal.example` → schéma `basic` pour cet hôte particulier [Docker Documentation](https://docs.docker.com/build/building/secrets/?utm_source=chatgpt.com)

