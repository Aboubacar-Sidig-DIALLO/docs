# 📦 Build context (Contexte de build)

Les commandes **`docker build`** et **`docker buildx build`** permettent de construire des images Docker à partir :

* d’un **Dockerfile** (qui décrit les instructions de build),
* et d’un **contexte de build**.

***

### 🔍 Qu’est-ce que le _build context_ ?

Le **contexte de build** est **l’ensemble des fichiers et répertoires** accessibles par le processus de build.\
Ces fichiers sont envoyés au **Docker daemon** afin que les instructions comme `COPY` ou `ADD` puissent fonctionner.

***

### 📌 Définition du contexte

Le contexte est précisé par le dernier argument de la commande :

```bash
docker build [OPTIONS] PATH | URL | -
```

* **PATH** → chemin relatif ou absolu d’un répertoire local
* **URL** → dépôt Git, fichier tarball distant, ou Dockerfile brut
* **-** → entrée standard (_stdin_), par ex. un tarball envoyé via un pipe

***

### 📖 Exemples

#### 1. Contexte local

```bash
docker build -t monapp:latest .
```

* Contexte = le dossier courant (`.`).
* Tous les fichiers de ce dossier sont envoyés au démon Docker.

***

#### 2. Contexte via dépôt Git

```bash
docker build https://github.com/monuser/monrepo.git#main
```

* Clone le dépôt et utilise son contenu comme contexte.
* Supporte les branches, commits ou tags.

***

#### 3. Contexte via archive

```bash
docker build https://example.com/moncontext.tar.gz
```

* Télécharge l’archive, la décompresse et l’utilise comme contexte.

***

#### 4. Contexte via _stdin_

```bash
tar -cz . | docker build -t monapp -
```

* Le tarball est envoyé directement via un pipe.
* Utile pour maîtriser exactement ce qui est envoyé.

***

### ⚠️ Points importants

* Tout le contenu du répertoire de contexte est envoyé au démon Docker → attention aux fichiers lourds ou inutiles.
* Utilise **`.dockerignore`** pour exclure les fichiers inutiles (`node_modules`, `.git`, logs, etc.).
* Un mauvais contexte peut **ralentir** ton build et **gonfler** tes images.

## 🏗️ Qu’est-ce qu’un **build context** ?

Le **contexte de build** est l’ensemble des fichiers et répertoires auxquels Docker a accès lors de la construction d’une image.

Quand tu exécutes la commande **`docker build`**, le dernier argument que tu donnes à la commande définit **quel contexte utiliser**.

***

### 🔹 Format de la commande

```bash
docker build [OPTIONS] PATH | URL | -
```

👉 La partie `PATH | URL | -` indique à Docker où trouver le **contexte**.

***

### 📂 Types de contextes possibles

1. **Un répertoire local (PATH)**
   * Peut être relatif (`.`) ou absolu (`/home/utilisateur/projet`).
   *   Exemple :

       ```bash
       docker build -t monapp:latest .
       ```

       → Le répertoire courant est envoyé comme contexte de build.
2. **Un dépôt Git distant (URL)**
   *   Exemple :

       ```bash
       docker build https://github.com/exemple/monrepo.git#main
       ```

       → Docker clone le dépôt et utilise son contenu comme contexte.
3. **Une archive distante (tarball) ou un fichier texte**
   *   Exemple :

       ```bash
       docker build https://exemple.com/context.tar.gz
       ```
4. **L’entrée standard (`-`)**
   * Permet de passer un fichier tar ou du texte directement en flux (stdin).
   *   Exemple :

       ```bash
       tar -cz . | docker build -
       ```

***

✅ En résumé : le **build context** est **tout ce dont ton Dockerfile peut se servir** (par exemple avec `COPY` ou `ADD`).

## 📂 Contextes de build basés sur un système de fichiers (Filesystem contexts)

Quand tu définis ton **contexte de build** avec `docker build`, ce contexte devient **l’ensemble des fichiers disponibles pour la construction**.

Les instructions comme **`COPY`** et **`ADD`** peuvent ensuite utiliser **n’importe quel fichier ou dossier présent dans ce contexte**.

***

### 🔹 Règles générales

Un **filesystem build context** est traité de manière **récursive** :

* Si tu fournis un **répertoire local** → tous les sous-dossiers et fichiers sont inclus.
* Si tu fournis un **tarball (archive .tar/.tar.gz)** → tous les fichiers à l’intérieur de l’archive sont inclus.
* Si tu fournis un **dépôt Git distant** → le dépôt entier + ses **submodules** sont inclus.

***

### 📌 Types de contextes possibles

1.  **📁 Répertoire local**\
    Exemple :

    ```bash
    docker build -t monapp:latest .
    ```

    → Le répertoire courant (`.`) est envoyé comme contexte au démon Docker.
2.  **🌐 Dépôt Git**\
    Exemple :

    ```bash
    docker build https://github.com/utilisateur/monrepo.git#branche
    ```

    → Docker clone le dépôt et inclut aussi ses sous-modules.
3.  **📦 Archive distante (tarball)**\
    Exemple :

    ```bash
    docker build https://exemple.com/context.tar.gz
    ```

    → Docker télécharge l’archive, la décompresse et l’utilise comme contexte.

***

👉 Donc, le **context** agit comme une **“racine de fichiers”** pour ton Dockerfile.\
Si un fichier n’est **pas dans le contexte**, tu ne peux pas l’utiliser avec `COPY` ou `ADD`.

## 📄 Contextes de build basés sur un fichier texte (Text file contexts)

Quand ton **contexte de build** est un **fichier texte brut**, Docker l’interprète directement comme un **Dockerfile**.

👉 Dans ce cas :

* **Il n’y a pas de système de fichiers (filesystem context)** associé.
* Cela signifie que tu ne peux pas utiliser d’instructions comme `COPY` ou `ADD`, car il n’y a pas d’arborescence de fichiers disponible.

***

#### 📌 Exemple

Imaginons que tu as un Dockerfile minimaliste enregistré sous forme de **fichier texte** :

```bash
echo -e "FROM alpine\nCMD [\"echo\", \"Hello depuis Dockerfile inline!\"]" > Dockerfile.txt
```

Ensuite, tu peux lancer un build en passant ce fichier comme contexte :

```bash
docker build -f Dockerfile.txt - < Dockerfile.txt
```

Ici :

* `-f Dockerfile.txt` → indique le fichier à utiliser comme Dockerfile.
* `-` (le tiret) → précise qu’il n’y a **pas de contexte de fichiers**, juste ce Dockerfile.

***

#### 📖 Pour aller plus loin

Ce fonctionnement est lié à la notion de **empty build context** (contexte vide), qui est utile si :

* Tu veux juste tester un build **sans fichiers externes**.
* Tu as un Dockerfile qui ne dépend d’aucun fichier local.

## 📂 Contexte local (Local context)

Pour utiliser un **contexte de build local**, tu peux spécifier un chemin de fichier relatif ou absolu dans la commande `docker build`.

👉 Exemple :

```bash
docker build .
```

Ici, le `.` signifie que le contexte de build est le **répertoire courant**.

***

#### 🔎 Exemple de sortie

```
16 [internal] load build context
16 sha256:23ca2f94460dcbaf5b3c3edbaaa933281a4e0ea3d92fe295193e4df44dc68f85
16 transferring context: 13.16MB 2.2s done
```

➡️ Cela montre que :

* Docker a **chargé le contexte** du répertoire courant.
* Il a transféré 13,16 Mo de données vers le moteur de build.

***

#### ✅ Ce que ça implique

* Tous les **fichiers et dossiers** dans ton répertoire de travail sont accessibles au builder.
* Le moteur de build **ne charge les fichiers du contexte que lorsqu’ils sont nécessaires** (par exemple via une instruction `COPY` ou `ADD`).

***

#### 📦 Tarballs comme contexte local

Tu peux aussi utiliser une **archive tar** comme contexte de build en la transmettant à `docker build` :

```bash
tar -czf context.tar.gz . 
cat context.tar.gz | docker build -
```

Ici :

* `context.tar.gz` contient ton projet.
* `docker build -` prend le **contenu du tarball** comme contexte (au lieu d’un dossier).

## 📂 Répertoires locaux (Local directories)

Imaginons que tu aies la structure de projet suivante :

```
.
├── index.ts
├── src/
├── Dockerfile
├── package.json
└── package-lock.json
```

***

#### 📌 Utilisation dans un Dockerfile

Si tu utilises ce répertoire comme **contexte de build**, toutes ces ressources deviennent disponibles dans ton `Dockerfile`.

Exemple :

```dockerfile
# syntax=docker/dockerfile:1
FROM node:latest

# Définir le dossier de travail dans le conteneur
WORKDIR /src

# Copier uniquement les fichiers nécessaires pour installer les dépendances
COPY package.json package-lock.json .

# Installer les dépendances
RUN npm ci

# Copier le reste de l’application
COPY index.ts src .
```

***

#### 🚀 Commande de build

Pour construire l’image, exécute :

```bash
docker build .
```

Ici :

* Le `.` indique que le **répertoire courant** est utilisé comme **contexte de build**.
* Les instructions `COPY` font référence à des fichiers de ce répertoire (ex. `package.json`, `index.ts`, `src/`).

***

✅ Résultat : ton image Docker contient **Node.js**, les dépendances installées via `npm ci`, et ton code source (`index.ts` + `src/`).

## 📌 Contexte local avec un Dockerfile fourni via **stdin**

Normalement, Docker lit le `Dockerfile` dans le répertoire de contexte.\
Mais il est aussi possible de **fournir le Dockerfile directement depuis `stdin`** sans qu’il existe en tant que fichier sur disque.

***

#### 📝 Syntaxe

```bash
docker build -f- PATH
```

* `-f-` : indique à Docker de lire le **Dockerfile depuis l’entrée standard (stdin)**
* `PATH` : définit le **contexte de build** (par ex. `.` pour le répertoire courant)

***

#### 🚀 Exemple concret

1. Crée un dossier de test et ajoute un fichier :

```bash
mkdir example
cd example
touch somefile.txt
```

2. Lance un build avec un **Dockerfile passé directement dans la commande** :

```bash
docker build -t myimage:latest -f- . <<EOF
FROM busybox
COPY somefile.txt ./
RUN cat /somefile.txt
EOF
```

***

#### 🔎 Explication

* Le **contexte** est `.` → donc `somefile.txt` est disponible pour `COPY`.
* Le `Dockerfile` est fourni directement par le bloc **here-document** (`<<EOF ... EOF`).
* L’image finale :
  * est basée sur `busybox`,
  * copie `somefile.txt` dans `/`,
  * exécute `cat /somefile.txt`.

***

✅ Avantage : pas besoin de créer un fichier `Dockerfile` sur disque, pratique pour des tests rapides ou des scripts automatisés.

## 📌 Contexte local avec un **tarball**

Il est aussi possible de fournir à `docker build` un **contexte de build compressé** (`tar`, `tar.gz`, etc.) au lieu d’un répertoire.\
Dans ce cas, Docker décompresse le tarball et utilise son contenu comme **système de fichiers de contexte**.

***

#### 🗂 Exemple de projet

```
.
├── Dockerfile
├── Makefile
├── README.md
├── main.c
├── scripts/
├── src/
└── test.Dockerfile
```

***

#### 🔨 Étape 1 : Créer un tarball

```bash
tar czf foo.tar.gz *
```

Ici, `foo.tar.gz` contient tous les fichiers et dossiers du projet.

***

#### 🔨 Étape 2 : Construire avec ce tarball comme contexte

```bash
docker build - < foo.tar.gz
```

* Le `-` indique à Docker de lire le **contexte** depuis l’entrée standard (`stdin`).
* Le Dockerfile par défaut (`Dockerfile`) est recherché dans le tarball.

***

#### 🔨 Étape 3 : Spécifier un autre Dockerfile dans le tarball

Si tu veux utiliser `test.Dockerfile` présent dans l’archive :

```bash
docker build --file test.Dockerfile - < foo.tar.gz
```

Docker ira chercher **`test.Dockerfile` à la racine du tarball** comme fichier de build.

***

✅ **Cas d’usage pratique** :

* envoyer un projet complet compressé à `docker build` (ex. via un pipeline CI/CD).
* éviter de partager tout un répertoire, et ne transmettre que ce qui est nécessaire.

## 📌 Contexte distant (_Remote context_) dans Docker Build

Avec `docker build` ou `docker buildx build`, tu peux définir un **contexte de build distant** au lieu d’un répertoire local.\
Cela permet de construire directement depuis un **dépôt Git**, un **tarball** ou même un **fichier texte (Dockerfile)** accessible via une URL.

***

### 🔹 Types de contextes distants

1.  **Dépôts Git**

    * Docker clone automatiquement le dépôt spécifié.
    * Seul le dernier commit (_HEAD_) est récupéré (clone "léger").
    * Les **sous-modules** sont aussi clonés.

    ```bash
    docker build https://github.com/user/myrepo.git
    ```

    👉 Par défaut, c’est la branche principale (default branch) qui est utilisée.

***

2.  **Fragments d’URL Git (`#ref:dir`)**

    Tu peux préciser :

    * une branche,
    * un tag,
    * un commit hash,
    * et un sous-dossier comme contexte de build.

    Format :

    ```
    https://github.com/user/myrepo.git#ref:dir
    ```

    Exemple :

    ```bash
    docker build https://github.com/user/myrepo.git#container:docker
    ```

    Ici :

    * `container` = la branche,
    * `docker` = le sous-dossier utilisé comme contexte.

    ✅ Tableau des suffixes valides :

    | Syntaxe                        | Commit utilisé     | Contexte utilisé |
    | ------------------------------ | ------------------ | ---------------- |
    | `myrepo.git`                   | Branche par défaut | `/`              |
    | `myrepo.git#mytag`             | Tag `mytag`        | `/`              |
    | `myrepo.git#mybranch`          | Branche `mybranch` | `/`              |
    | `myrepo.git#pull/42/head`      | Pull request `#42` | `/`              |
    | `myrepo.git#:myfolder`         | Branche par défaut | `/myfolder`      |
    | `myrepo.git#master:myfolder`   | Branche `master`   | `/myfolder`      |
    | `myrepo.git#mytag:myfolder`    | Tag `mytag`        | `/myfolder`      |
    | `myrepo.git#mybranch:myfolder` | Branche `mybranch` | `/myfolder`      |

    ⚠️ Si tu utilises un **commit hash**, il doit être complet (40 caractères SHA-1).\
    Exemple :

    ```bash
    # ✅ OK
    docker build github.com/docker/buildx#d4f088e689b41353d74f1a0bfcd6d7c0b213aed2
    # ❌ Faux (hash tronqué à 7 caractères)
    docker build github.com/docker/buildx#d4f088e
    ```

***

3.  **Requêtes d’URL (méthode moderne avec `?`)**

    ⚡ Nécessite :

    * **Buildx ≥ 0.28.0**
    * **Dockerfile ≥ 1.18.0**
    * **Docker Desktop ≥ 4.46.0**

    Exemple :

    ```bash
    docker buildx build 'https://github.com/user/myrepo.git?branch=container&subdir=docker'
    ```

    ✅ Équivalents en tableau :

    | Syntaxe                               | Commit utilisé     | Contexte utilisé |
    | ------------------------------------- | ------------------ | ---------------- |
    | `myrepo.git`                          | Branche par défaut | `/`              |
    | `myrepo.git?tag=mytag`                | Tag `mytag`        | `/`              |
    | `myrepo.git?branch=mybranch`          | Branche `mybranch` | `/`              |
    | `myrepo.git?ref=pull/42/head`         | Pull request `#42` | `/`              |
    | `myrepo.git?subdir=myfolder`          | Branche par défaut | `/myfolder`      |
    | `myrepo.git?branch=master&subdir=...` | Branche `master`   | `/myfolder`      |

    Tu peux aussi ajouter un **checksum** pour valider que la référence pointe bien sur le commit attendu :

    ```bash
    docker buildx build 'https://github.com/moby/buildkit.git?tag=v0.21.1&checksum=66735c67'
    ```

    Si le hash ne correspond pas, la build échoue.

***

4.  **Conserver le dossier `.git`**

    Par défaut, BuildKit **supprime** le dossier `.git` lors du clonage.\
    Mais tu peux le garder si tu as besoin d’infos Git dans ton build :

    ```dockerfile
    # syntax=docker/dockerfile:1
    FROM alpine
    WORKDIR /src
    RUN --mount=target=. \
      make REVISION=$(git rev-parse HEAD) build
    ```

    Commande :

    ```bash
    docker build \
      --build-arg BUILDKIT_CONTEXT_KEEP_GIT_DIR=1 \
      https://github.com/user/myrepo.git#main
    ```

***

5.  **Dépôts privés**

    🔑 Deux options d’authentification :

    *   **Par SSH** (détecté automatiquement via `$SSH_AUTH_SOCK`) :

        ```bash
        docker buildx build --ssh default git@github.com:user/private.git
        ```
    *   **Par token** (via `--secret`) :

        ```bash
        GIT_AUTH_TOKEN=<token> docker buildx build \
          --secret id=GIT_AUTH_TOKEN \
          https://github.com/user/private.git
        ```

    ⚠️ **Ne jamais utiliser `--build-arg` pour transmettre des secrets** (pas sécurisé).

***

✅ **En résumé :**

* Tu peux construire directement depuis **Git**, **tarball** ou **Dockerfile distant**.
* Les fragments `#` et requêtes `?` permettent un contrôle fin (branche, tag, dossier, commit).
* Pour dépôts privés → SSH ou token.
* Avec BuildKit, tu peux même garder `.git` pour utiliser des infos de version dans ton image.

## 📌 Contexte distant avec un Dockerfile depuis `stdin`

Il est possible de **construire une image à partir d’un contexte distant** (par exemple un dépôt Git ou un tarball), tout en fournissant ton propre **Dockerfile via `stdin`**.

***

### 🔹 Syntaxe

```bash
docker build -f- URL
```

* `-f` ou `--file` → permet de spécifier le Dockerfile à utiliser.
* `-` (tiret) → indique à Docker de lire le Dockerfile depuis **l’entrée standard (`stdin`)**.

👉 Très utile si :

* le dépôt **ne contient pas de Dockerfile**,
* ou si tu veux utiliser un **Dockerfile personnalisé** sans créer un fork du projet.

***

### 🔹 Exemple pratique

Ici, on utilise le dépôt GitHub `hello-world`, qui contient un fichier `hello.c`, et on fournit un Dockerfile minimal depuis `stdin` :

```bash
docker build -t myimage:latest -f- https://github.com/docker-library/hello-world.git <<EOF
FROM busybox
COPY hello.c ./
EOF
```

* Le contexte vient du dépôt `hello-world` (téléchargé automatiquement).
* Le Dockerfile est lu **depuis `stdin`** (le bloc `EOF`).
* L’image finale est nommée `myimage:latest`.

***

## 📌 Contexte distant avec tarballs

Tu peux aussi utiliser directement un **tarball distant** comme contexte de build.

***

### 🔹 Exemple simple

```bash
docker build http://server/context.tar.gz
```

Résultat attendu :

```
1 [internal] load remote build context
1 DONE 0.2s

2 copy /context /
2 DONE 0.1s
...
```

* Le **fichier `context.tar.gz`** est téléchargé par **le démon BuildKit** (⚠️ pas forcément ta machine locale si tu utilises un builder distant).
* Le contenu du tarball est utilisé comme **contexte de build**.
* Le Dockerfile est recherché à la racine du tarball (sauf si tu précises `--file`).

***

### 🔹 Formats acceptés

Le tarball doit être conforme au format standard **Unix tar**, et peut être compressé avec :

* **gzip** (`.tar.gz`)
* **bzip2** (`.tar.bz2`)
* **xz** (`.tar.xz`)
* ou sans compression.

***

✅ **Résumé :**

* `docker build -f- URL` → permet d’utiliser un **Dockerfile personnalisé via `stdin`** avec un contexte distant.
* `docker build URL.tar.gz` → permet de construire depuis un **tarball distant**.
* Dans les deux cas, le téléchargement se fait côté **BuildKit**, pas forcément sur ton poste.

## 📌 Contexte vide & fichiers `.dockerignore`

### 🔹 Contexte vide

Quand tu utilises un **fichier texte** comme contexte de build, Docker l’interprète directement comme un **Dockerfile**.\
👉 Cela signifie que le build **n’a pas de contexte de système de fichiers**.

#### Exemple — construire sans contexte

Tu peux fournir le Dockerfile :

* via **stdin** (entrée standard),
* ou via l’URL d’un fichier texte distant.

**Exemple avec `stdin` (Linux) :**

```bash
docker build - < Dockerfile
```

**Exemple avec heredoc (bash) :**

```bash
ls
# -> main.c existe

docker build -<<< $'FROM scratch\nCOPY main.c .'
```

🔴 Résultat : erreur, car avec un **contexte vide**, tu ne peux pas utiliser `COPY` ou `ADD` pour inclure des fichiers locaux.

***

### 🔹 `.dockerignore`

Le fichier **`.dockerignore`** permet d’exclure certains fichiers/dossiers du contexte de build, ce qui :

* réduit la taille des données envoyées au démon Docker,
* accélère les builds,
* évite d’envoyer des fichiers inutiles ou sensibles (ex. `node_modules`, `.git`).

***

#### 📂 Exemple simple

```
# .dockerignore
node_modules
bar
```

➡️ Ni `node_modules/` ni `bar/` ne seront envoyés dans le contexte.

***

#### 📂 Exemple projet avec plusieurs Dockerfiles

Docker permet d’associer un fichier `.dockerignore` **spécifique** à un Dockerfile.\
Pour cela : nomme ton fichier `<Dockerfile>.dockerignore`.

Exemple :

```
.
├── index.ts
├── src/
├── docker/
│   ├── build.Dockerfile
│   ├── build.Dockerfile.dockerignore
│   ├── lint.Dockerfile
│   ├── lint.Dockerfile.dockerignore
│   ├── test.Dockerfile
│   └── test.Dockerfile.dockerignore
├── package.json
└── package-lock.json
```

➡️ Ici, `build.Dockerfile.dockerignore` s’applique uniquement au `build.Dockerfile`.\
⚠️ S’il y a à la fois un `.dockerignore` **global** et un \`.dockerignore spécifique\*\*, ce dernier **prend le dessus**.

***

#### 🔹 Syntaxe des règles `.dockerignore`

* Chaque ligne est un **motif** (pattern).
* Compatible avec les globs Unix (`*`, `?`, etc.).
* Les `/` initiaux ou finaux sont ignorés.
* Les lignes commençant par `#` sont des **commentaires**.

**Exemple**

```
/foo/bar/
/foo/bar
foo/bar/
/foo/bar
```

➡️ Toutes ces variantes excluent `foo/bar`.

***

#### 🔹 Exemple de matching

```
# commentaire
*/temp*
*/*/temp*
temp?
```

| Règle       | Effet                                                                                                 |
| ----------- | ----------------------------------------------------------------------------------------------------- |
| `*/temp*`   | Exclut `temp` et fichiers/dossiers commençant par `temp` à 1 niveau sous la racine (`/somedir/temp/`) |
| `*/*/temp*` | Exclut fichiers/dossiers `temp` à 2 niveaux sous la racine (`/somedir/subdir/tempX`)                  |
| `temp?`     | Exclut les fichiers `tempa`, `tempb`, … à la racine                                                   |

👉 Les règles utilisent l’API Go `filepath.Match`.\
👉 Docker ajoute aussi un support spécial pour `**` :

* `**/*.go` exclut **tous les fichiers `.go`** dans **tous les dossiers**.

***

#### 🔹 Règles avec exceptions (`!`)

Tu peux inverser une exclusion en ajoutant `!`.

**Exemple**

```
*.md
!README.md
```

➡️ Exclut tous les `.md` sauf `README.md`.

**Exemple avancé**

```
*.md
!README*.md
README-secret.md
```

➡️ Tous les `.md` sont exclus sauf les fichiers `README*`, sauf `README-secret.md`.

⚠️ **Ordre important** :\
👉 la **dernière règle applicable** détermine le comportement.

***

✅ **Résumé :**

* **Contexte vide** → utile si ton Dockerfile **ne dépend d’aucun fichier local**.
* **`.dockerignore`** → optimise et sécurise ton build en excluant des fichiers/dossiers inutiles.
* Tu peux définir un `.dockerignore` global ou spécifique à chaque Dockerfile.

## 📌 Contextes nommés dans Docker Build

En plus du **contexte par défaut** (celui que tu donnes en argument à `docker build`), tu peux fournir plusieurs **contexts nommés** à un build.\
👉 Cela permet d’inclure des fichiers et répertoires provenant de **sources multiples** (locales, distantes, images, etc.), tout en les gardant **logiquement séparés**.

***

### 🔹 Définir un contexte nommé

On utilise l’option `--build-context` suivie d’un **nom=valeur**.

Exemple :

```bash
docker build --build-context docs=./docs .
```

* `docs` est un contexte nommé qui pointe vers `./docs`.
* `.` est le contexte par défaut, ici le répertoire courant.

***

### 🔹 Utiliser les contexts nommés dans un Dockerfile

Dans ton Dockerfile, tu peux référencer ces contexts comme s’il s’agissait de **stages** dans un build multi-stage.

#### Exemple :

```dockerfile
# syntax=docker/dockerfile:1
FROM buildbase
WORKDIR /app

# Copier les fichiers depuis le contexte par défaut
COPY . /app/src
RUN make bin

# Monter les fichiers du contexte nommé "docs"
RUN --mount=from=docs,target=/app/docs \
    make manpages
```

***

### 🔹 Cas d’utilisation des contexts nommés

#### ✅ Exemple 1 : Combiner sources locales et distantes

Imagine que :

* le code de l’application est local,
* les scripts de déploiement sont dans un dépôt Git.

```bash
docker build --build-context scripts=https://github.com/user/deployment-scripts.git .
```

Dockerfile :

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine:latest

# Copier le code local
COPY . /opt/app

# Utiliser les scripts distants
RUN --mount=from=scripts,target=/scripts /scripts/main.sh
```

***

#### ✅ Exemple 2 : Builds dynamiques avec configs personnalisées

Tu veux injecter une configuration différente selon l’environnement (prod, dev…).

```bash
docker build --build-context config=./configs/prod .
```

Dockerfile :

```dockerfile
# syntax=docker/dockerfile:1
FROM nginx:alpine

# Utiliser la config "prod"
COPY --from=config nginx.conf /etc/nginx/nginx.conf
```

***

#### ✅ Exemple 3 : Pinner ou overrider une image

Tu peux remplacer une image directement au build, **sans modifier le Dockerfile**.

Dockerfile :

```dockerfile
FROM alpine:3.21
```

Commande :

```bash
docker buildx build --build-context alpine:3.21=docker-image://alpine:edge .
```

👉 Ici, `alpine:3.21` est remplacée par `alpine:edge`.

***

### ✅ Avantages des contexts nommés

* Plus de **flexibilité** (combine local, Git, tarballs, images…).
* Permet de séparer clairement les **sources de données**.
* Utile pour :
  * injecter des configurations dynamiques,
  * utiliser des dépôts privés ou publics,
  * tester différentes bases d’images **sans toucher au Dockerfile**.

## 📌 Contextes nommés avec **Bake**

👉 **Bake** est un outil intégré à `docker buildx` qui permet de gérer tes builds avec un fichier de configuration (`docker-bake.hcl` ou `docker-bake.json`).\
Il simplifie les **builds complexes**, en rendant la configuration **déclarative** et réutilisable.\
Bonne nouvelle : **Bake supporte pleinement les contextes nommés**.

***

### 🔹 Définir des contextes nommés dans un fichier Bake

Exemple `docker-bake.hcl` :

```hcl
target "app" {
  contexts = {
    docs = "./docs"
  }
}
```

👉 Équivalent en ligne de commande :

```bash
docker build --build-context docs=./docs .
```

***

### 🔹 Chaînage de cibles avec contextes nommés

L’un des gros avantages de **Bake** est de pouvoir définir des **pipelines de builds** où un build dépend directement d’un autre.

Imaginons que tu as **deux Dockerfiles** :

* `base.Dockerfile` → construit une image de base
* `app.Dockerfile` → construit ton application en utilisant l’image de base

***

#### 📝 Exemple `app.Dockerfile`

```dockerfile
FROM mybaseimage
```

Normalement, tu devrais :

1. Construire l’image base (`docker build -f base.Dockerfile -t mybaseimage .`)
2. La stocker en local ou la pousser dans un registre
3. Puis construire `app.Dockerfile`.

Avec **Bake**, plus besoin de ça. Tu peux déclarer une dépendance entre tes cibles.

***

#### 📝 Exemple `docker-bake.hcl`

```hcl
target "base" {
  dockerfile = "base.Dockerfile"
}

target "app" {
  dockerfile = "app.Dockerfile"
  contexts = {
    # le préfixe target: indique que "base" est une cible Bake
    mybaseimage = "target:base"
  }
}
```

***

### 🔹 Résultat

Quand tu construis `app`, Bake :

1. Recompile `base` si nécessaire
2. Passe directement son résultat comme contexte nommé `mybaseimage`
3. Utilise ce résultat dans `app.Dockerfile`

👉 Commande :

```bash
docker buildx bake app
```

***

### ✅ Avantages de Bake avec contextes nommés

* **Moins de scripts manuels** → tout est déclaré dans un seul fichier.
* **Automatisation des dépendances** → pas besoin de pousser ou recharger les images intermédiaires.
* **Builds reproductibles** → tu peux partager le `docker-bake.hcl` avec ton équipe.
* **Support multi-targets** → tu peux lancer `docker buildx bake` et tout reconstruire d’un coup (base, app, tests, etc.).
