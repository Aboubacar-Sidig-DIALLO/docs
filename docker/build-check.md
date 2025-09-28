# Build check

### 🔍 Qu’est-ce que les _Build Checks_ ?

* C’est une sorte de **linting avancé** et **analyse statique** de ton Dockerfile et de tes options de build.
* Permet de **vérifier ta configuration avant d’exécuter réellement le build** (un peu comme un _dry-run_ mais avec validation).
* Détecte des **erreurs potentielles**, des **mauvaises pratiques** ou des **incohérences** dans :
  * Ton **Dockerfile**
  * Les **flags `docker buildx build`**
  * Les **contextes et options de build**

***

### ⚡ Exemple d’utilisation

Tu peux lancer une commande de build avec **mode check** au lieu de lancer réellement l’image :

```bash
docker buildx build --check .
```

👉 Ça n’exécute pas le build, mais ça renvoie une série de diagnostics.

***

### ✅ Ce que ça peut vérifier

D’après la \[Build checks reference] :

* **FROM** → vérifier si ton image de base est à jour / vulnérable
* **RUN apt-get** → vérifier si tu utilises `apt-get update` séparé (mauvaise pratique)
* **ENV** → vérifier si tu conserves des secrets ou des valeurs sensibles
* **COPY/ADD** → avertir si tu copies des fichiers non nécessaires (absence de `.dockerignore`)
* **Versions d’images** → avertir si tu n’as pas _pinné_ une version ou digest
* **Multi-stage** → signaler si ton build peut être optimisé avec des stages
* **Labels** → vérifier présence de labels recommandés (vendor, version, etc.)

***

### 🛠️ Avantages

* Évite de lancer un build long qui échoue sur une bêtise.
* Donne une **baseline de bonnes pratiques** (style guide officiel Docker).
* Utile dans un **pipeline CI/CD** pour bloquer un merge si l’image ne respecte pas les standards.

***

### 🚀 Exemple complet dans un workflow CI

```yaml
name: Docker Build Checks

on: [push, pull_request]

jobs:
  lint-docker:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Run build checks
        run: docker buildx build --check .
```

👉 Ici, ton pipeline s’assure que ton Dockerfile est **valide, sécurisé et optimisé** avant de builder réellement.

***

⚖️ En gros : c’est comme **Hadolint** intégré directement dans Docker, mais avec plus de contexte (images, cache, options Buildx).

### 🔎 Comment fonctionnent les _build checks_ ?

#### 1. Mode normal (sans checks)

*   Quand tu lances un build classique avec :

    ```bash
    docker buildx build .
    ```

    👉 Docker exécute **chaque étape** de ton Dockerfile (`FROM`, `RUN`, `COPY`, etc.).\
    → Résultat : une image finale prête à tourner.

***

#### 2. Mode _build checks_

*   Quand tu ajoutes `--check` :

    ```bash
    docker buildx build --check .
    ```

    👉 **Aucune étape n’est réellement exécutée**.\
    👉 Docker va **analyser** :

    * Ton **Dockerfile**
    * Tes **options de build** (`--platform`, `--build-arg`, `--no-cache`, etc.)
    * Ton **contexte de build** (`.dockerignore`, fichiers copiés, etc.)

Et il retourne un **rapport** avec :

* ✅ bonnes pratiques respectées
* ⚠️ avertissements (anti-patterns, risques de sécurité, mauvaises pratiques)
* ❌ erreurs bloquantes

***

### ⚡ Utilité des build checks

1. **Validation préalable**\
   Vérifier que ton Dockerfile est **valide et cohérent** avant de lancer un build long.
2. **Alignement sur les bonnes pratiques Docker**
   * Utiliser `COPY` au lieu de `ADD` si tu n’as pas besoin de ses fonctionnalités avancées
   * Regrouper `apt-get update && apt-get install` dans une même couche
   * Nettoyer `/var/lib/apt/lists/*` pour réduire la taille de l’image
3. **Détection des anti-patterns**
   * Variables sensibles exposées avec `ENV`
   * Utilisation de tags flottants (`latest`, `3.21`) sans digest
   * Packages installés inutilement
4. **Gain en CI/CD**\
   Tu peux bloquer un **merge request / pull request** si ton image ne respecte pas les standards de sécurité et de performance.

***

### 🛠️ Exemple

Dockerfile problématique :

```dockerfile
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install -y curl
```

Check :

```bash
docker buildx build --check .
```

Rapport potentiel :

```
[WARNING] Using 'ubuntu:latest' is discouraged. Pin to a specific version or digest.
[ERROR] 'apt-get update' and 'apt-get install' should be combined to avoid cache issues.
[INFO] Consider cleaning up apt cache with 'rm -rf /var/lib/apt/lists/*'.
```

***

### 💡 Bonus

👉 Si tu veux du **linting enrichi dans VS Code**, le plugin **Docker VS Code Extension** t’apporte :

* Highlight des erreurs directement dans ton Dockerfile
* Suggestions de bonnes pratiques
* Scan de vulnérabilités

## ⚡ Build avec checks

#### 1. Prérequis

* **Buildx ≥ 0.15.0**
* **docker/build-push-action ≥ 6.6.0**
* **docker/bake-action ≥ 5.6.0**

👉 À partir de ces versions, les _build checks_ sont **activés par défaut** lors d’un build.

***

#### 2. Exemple en ligne de commande

```bash
docker build .
```

👉 Résultat : le build **s’exécute normalement**, mais **les checks sont évalués en parallèle**.\
Exemple de sortie :

```
[+] Building 3.5s (11/11) FINISHED
...

1 warning found (use --debug to expand):
  - Lint Rule 'JSONArgsRecommended': JSON arguments recommended for CMD to prevent unintended behavior related to OS signals (line 7)
```

💡 Ici, l’avertissement (`JSONArgsRecommended`) indique que `CMD` devrait être en syntaxe JSON array (`CMD ["executable", "arg"]`) au lieu de la forme shell (`CMD command arg`).

***

#### 3. Exemple en **GitHub Actions**

```yaml
name: Build and push Docker images
on:
  push:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build and push
        uses: docker/build-push-action@v6.6.0
```

👉 Dans ce cas :

* Les **checks sont exécutés automatiquement**
* Les **violations apparaissent directement dans la vue diff des Pull Requests** comme annotations\
  (⚠️ / ℹ️ directement visibles dans GitHub UI).

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### 🔎 Mode concis (par défaut)

Quand tu fais simplement :

```bash
docker build .
```

Tu obtiens quelque chose comme :

```
1 warning found (use --debug to expand):
  - Lint Rule 'JSONArgsRecommended': JSON arguments recommended for CMD to prevent unintended behavior related to OS signals (line 4)
```

👉 Résumé rapide : règle concernée, message, numéro de ligne.

***

### 📖 Mode détaillé avec `--debug`

```bash
docker --debug build .
```

Sortie beaucoup plus complète :

```
[+] Building 3.5s (11/11) FINISHED
...

 1 warning found:
 - JSONArgsRecommended: JSON arguments recommended for CMD to prevent unintended behavior related to OS signals (line 4)

JSON arguments recommended for ENTRYPOINT/CMD to prevent unintended behavior related to OS signals
More info: https://docs.docker.com/go/dockerfile/rule/json-args-recommended/

Dockerfile:4
--------------------
   2 |
   3 |     FROM alpine
   4 | >>> CMD echo "Hello, world!"
   5 |
--------------------
```

#### Ce que tu gagnes en `--debug` :

1. 📌 **Lien direct vers la doc** du lint concerné.
2. 📋 **Snippet du Dockerfile** avec la ligne incriminée mise en évidence (`>>>`).
3. 📝 **Explication détaillée** de pourquoi la règle existe.

***

👉 Ça en fait un vrai outil de **linting avancé pour Dockerfile**, pas juste un warning caché dans les logs.

## 🔍 Vérifier un build sans réellement construire l’image

### 📌 Contexte

Il est parfois utile de **contrôler la qualité d’un Dockerfile** sans exécuter le build complet.\
👉 C’est possible avec l’option **`--check`** de `docker build`.

***

### ✅ Exemple de commande

```bash
docker build --check .
```

* Cette commande **n’exécute pas les étapes du Dockerfile**.
* Elle lance uniquement les **vérifications (build checks)**.
* Les problèmes détectés sont listés dans la sortie du terminal.

***

### 📊 Exemple de sortie avec `--check`

```
[+] Building 1.5s (5/5) FINISHED
=> [internal] connecting to local controller
=> [internal] load build definition from Dockerfile
=> => transferring dockerfile: 253B
=> [internal] load metadata for docker.io/library/node:22
=> [auth] library/node:pull token for registry-1.docker.io
=> [internal] load .dockerignore
=> => transferring context: 50B

JSONArgsRecommended - https://docs.docker.com/go/dockerfile/rule/json-args-recommended/
JSON arguments recommended for ENTRYPOINT/CMD to prevent unintended behavior related to OS signals
Dockerfile:7
--------------------
5 |
6 |     COPY index.js .
7 | >>> CMD node index.js
8 |
--------------------
```

#### 📝 Explication

* La règle violée est **`JSONArgsRecommended`** → Docker recommande d’utiliser la **syntaxe JSON** pour `CMD` ou `ENTRYPOINT`.
* Ligne concernée : **7** (`CMD node index.js`).
* Lien vers la doc officielle est fourni.

***

### ⚠️ Différence avec un build normal

* **Build classique** : les violations apparaissent comme des **warnings**, mais le build continue (exit code = `0`).
* **Build avec `--check`** :
  * aucune image n’est construite,
  * si une violation est détectée → **le build échoue** (exit code ≠ `0`).

### 🚦 En résumé

* `docker build .` → exécute le build + affiche warnings.
* `docker --debug build .` → exécute le build + affiche warnings **détaillés**.
* `docker build --check .` → **pas de build**, uniquement vérifications, **échec si violation**.

👉 Parfait pour intégrer dans un pipeline **CI/CD** comme étape de validation avant build/push.

## 🚨 Échec du build en cas de violations de vérification (Build Checks)

### 📌 Contexte

Par défaut, lorsqu’on active les **build checks** dans Docker, les violations (mauvaises pratiques ou avertissements dans le Dockerfile) sont simplement signalées comme **warnings**.\
👉 Résultat : le build continue avec un **code de sortie 0** (succès).

***

### ✅ Transformer les avertissements en erreurs

#### 1. Dans le Dockerfile

Ajoute cette directive spéciale en haut de ton fichier :

```dockerfile
# syntax=docker/dockerfile:1
# check=error=true   👈 active le mode strict

FROM alpine
CMD echo "Hello, world!"
```

🔎 Résultat :

* **Sans `check=error=true`** → avertissement affiché mais le build réussit.
* **Avec `check=error=true`** → l’avertissement devient **erreur**, le build échoue (code de sortie 1).

***

#### 2. En ligne de commande (sans modifier le Dockerfile)

Tu peux activer le mode strict directement lors du build :

```bash
docker build --check --build-arg "BUILDKIT_DOCKERFILE_CHECK=error=true" .
```

***

### 🚦 Bonnes pratiques

* Active `check=error=true` dans tes **pipelines CI/CD** (GitHub Actions, GitLab CI, etc.) pour bloquer automatiquement un merge si le Dockerfile n’est pas conforme.
* Utilise `docker build --check` pour **analyser** un Dockerfile sans lancer la construction réelle.
* Combine le mode strict avec l’intégration dans **GitHub Actions** afin que les violations soient visibles dans la revue de code (annotations sur la PR).

## ⏭️ Ignorer des vérifications (Skip checks)

### 📌 Contexte

Par défaut, **tous les checks** sont exécutés lors d’un `docker build`.\
Mais tu peux choisir **d’ignorer certaines règles** si elles ne sont pas pertinentes pour ton projet.

***

### ✅ Ignorer des règles spécifiques dans le Dockerfile

Tu peux utiliser la directive **`# check=skip`** en haut de ton Dockerfile.\
Elle prend une **liste CSV** (séparée par des virgules) des identifiants de règles à ignorer.

#### Exemple :

```dockerfile
# syntax=docker/dockerfile:1
# check=skip=JSONArgsRecommended,StageNameCasing

FROM alpine AS BASE_STAGE
CMD echo "Hello, world!"
```

👉 Ici :

* La règle **`JSONArgsRecommended`** (qui recommande d’utiliser la syntaxe JSON pour `CMD`/`ENTRYPOINT`) est ignorée.
* La règle **`StageNameCasing`** (qui vérifie la casse correcte des noms de stages) est aussi ignorée.

➡️ Résultat : **aucune violation signalée** au build.

***

### ✅ Ignorer des règles via la CLI

Tu peux aussi passer les règles à ignorer directement dans la commande `docker build` grâce à l’argument **`BUILDKIT_DOCKERFILE_CHECK`**.

#### Exemple :

```bash
docker build --check \
  --build-arg "BUILDKIT_DOCKERFILE_CHECK=skip=JSONArgsRecommended,StageNameCasing" .
```

👉 Cela revient au même que la directive dans le Dockerfile.

***

### 🚫 Ignorer tous les checks

Si tu veux **désactiver toutes les vérifications**, utilise `skip=all`.

#### Exemple :

```dockerfile
# syntax=docker/dockerfile:1
# check=skip=all

FROM alpine
CMD echo "Hello, world!"
```

➡️ Ici, **aucune vérification n’est exécutée** pendant le build.

## ⚖️ Combiner `error` et `skip` dans les directives de vérification

### 📌 Contexte

* `skip` 👉 permet **d’ignorer certaines règles** de vérification.
* `error=true` 👉 transforme les avertissements (**warnings**) restants en **erreurs bloquantes**.

Tu peux combiner les deux pour :\
➡️ ignorer certaines règles **non pertinentes**,\
➡️ mais être strict sur toutes les autres.

***

### ✅ Exemple avec Dockerfile

```dockerfile
# syntax=docker/dockerfile:1
# check=skip=JSONArgsRecommended,StageNameCasing;error=true

FROM alpine
CMD echo "Hello, world!"
```

👉 Ici :

* Les règles `JSONArgsRecommended` et `StageNameCasing` sont **ignorées**.
* Toutes les autres règles actives deviennent **bloquantes** (le build échoue si elles ne passent pas).

***

### ✅ Exemple avec la CLI

Tu peux aussi faire la même chose via l’argument `BUILDKIT_DOCKERFILE_CHECK` :

```bash
docker build --check \
  --build-arg "BUILDKIT_DOCKERFILE_CHECK=skip=JSONArgsRecommended,StageNameCasing;error=true" .
```

***

### 📊 Résumé

| Paramètre                 | Effet                                                      |
| ------------------------- | ---------------------------------------------------------- |
| `skip=<rules>`            | Ignore les règles listées                                  |
| `skip=all`                | Ignore toutes les règles                                   |
| `error=true`              | Transforme les warnings restants en erreurs                |
| `skip=<rules>;error=true` | Ignore certaines règles, mais échoue sur toutes les autres |

## 🧪 Vérifications expérimentales (Experimental checks)

### 📌 Contexte

* Certaines règles de vérification sont encore **expérimentales** avant d’être promues stables.
* Par défaut 👉 **elles sont désactivées**.
* Tu peux choisir de les **activer toutes** ou seulement certaines.

***

### ✅ Activer **toutes** les vérifications expérimentales

En **CLI** :

```bash
docker build --check \
  --build-arg "BUILDKIT_DOCKERFILE_CHECK=experimental=all" .
```

En **Dockerfile** :

```dockerfile
# syntax=docker/dockerfile:1
# check=experimental=all

FROM alpine
CMD echo "Hello, world!"
```

***

### ✅ Activer seulement certaines vérifications expérimentales

Tu peux préciser les règles sous forme de **CSV** :

En **Dockerfile** :

```dockerfile
# syntax=docker/dockerfile:1
# check=experimental=JSONArgsRecommended,StageNameCasing
```

En **CLI** :

```bash
docker build --check \
  --build-arg "BUILDKIT_DOCKERFILE_CHECK=experimental=JSONArgsRecommended,StageNameCasing" .
```

***

### ⚠️ Important

👉 La directive `experimental` **a priorité** sur `skip`.\
Exemple :

```dockerfile
# syntax=docker/dockerfile:1
# check=skip=all;experimental=all
```

➡️ Ici, **toutes les règles stables sont ignorées**, mais **les règles expérimentales seront quand même exécutées**.
