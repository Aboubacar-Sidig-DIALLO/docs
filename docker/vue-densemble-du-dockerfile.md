# 📘 Vue d’ensemble du Dockerfile

Tout commence avec un **Dockerfile**.\
C’est un **fichier texte** qui décrit **comment construire une image Docker**, étape par étape.\
Docker lit ce fichier et exécute les instructions pour produire une image prête à l’emploi.

***

### 🛠️ Les instructions les plus courantes

| Instruction               | Description                                                                                                                                                                |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`FROM <image>`**        | Définit l’**image de base**. Exemple : `FROM node:18-alpine`                                                                                                               |
| **`RUN <command>`**       | Exécute une commande dans une nouvelle couche, puis enregistre le résultat. Exemple : `RUN apt-get update && apt-get install -y curl`                                      |
| **`WORKDIR <directory>`** | Définit le **répertoire de travail** pour les instructions suivantes (`RUN`, `CMD`, `COPY`, etc.). Exemple : `WORKDIR /app`                                                |
| **`COPY <src> <dest>`**   | Copie des fichiers/dossiers du **contexte local** vers le système de fichiers de l’image. Exemple : `COPY package.json /app/`                                              |
| **`CMD <command>`**       | Définit la **commande par défaut** exécutée au démarrage du conteneur. Exemple : `CMD ["node", "server.js"]`. Attention ⚠️ : seul le **dernier `CMD`** est pris en compte. |

***

### 🧩 Caractéristiques importantes

* Un **Dockerfile = la recette** de ton image.
* Chaque instruction crée une **nouvelle couche** (layer) dans l’image.
* Plus ton Dockerfile est optimisé, plus tes images seront **légères, rapides à construire et sécurisées**.
* Tu peux commencer avec un fichier **très simple**, puis l’enrichir pour gérer :
  * les dépendances,
  * la configuration,
  * les builds multi-étapes,
  * les optimisations de cache.

***

✅ En résumé :\
Le **Dockerfile** est le **point de départ incontournable** pour construire des images Docker.\
Il permet d’automatiser la construction de ton environnement et de l’exécuter **partout (local, cloud, CI/CD)**.

## 📘 Nom de fichier (Dockerfile)

*   **Nom par défaut** :\
    Le fichier doit s’appeler **`Dockerfile`** (sans extension).\
    👉 Avec ce nom par défaut, tu peux exécuter directement :

    ```bash
    docker build .
    ```

    sans avoir à préciser d’option supplémentaire.

***

### 📂 Cas où plusieurs Dockerfiles sont nécessaires

Dans certains projets, tu peux avoir besoin de **Dockerfiles distincts** pour des usages différents (ex : développement, production, tests).\
👉 Dans ce cas, une convention courante est de les nommer ainsi :

* `dev.Dockerfile`
* `prod.Dockerfile`
* `test.Dockerfile`

***

### ⚙️ Spécifier un Dockerfile particulier

Si ton fichier **ne s’appelle pas Dockerfile**, tu dois le préciser avec l’option `--file` :

```bash
docker build --file prod.Dockerfile -t mon-image:prod .
```

Ici :

* `--file prod.Dockerfile` → indique à Docker quel fichier utiliser,
* `-t mon-image:prod` → donne un nom et un tag à l’image,
* `.` → désigne le contexte de build (dossier courant).

***

### ✅ Recommandation officielle

👉 **Toujours utiliser `Dockerfile` comme fichier principal de ton projet**,\
et réserver les noms alternatifs (`prod.Dockerfile`, etc.) uniquement si tu as des besoins spécifiques.

## 🐳 Docker Images

Les **images Docker** sont constituées de **couches (layers)**.\
👉 Chaque couche est le résultat d’une instruction dans le **Dockerfile**.\
👉 Les couches sont **empilées séquentiellement** : chaque couche représente un **delta** (les changements appliqués par rapport à la précédente).

Cela rend les images :

* plus **efficaces** (réutilisation du cache des couches déjà construites),
* plus **rapides à transférer** (on ne télécharge que les couches manquantes).

***

### 📌 Exemple pratique : Application Flask (Hello World)

On prend une petite application **Python avec Flask** :

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"
```

***

### 🐳 Dockerfile pour packager l’app

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:22.04

# Installer Python et pip
RUN apt-get update && apt-get install -y python3 python3-pip

# Installer Flask
RUN pip install flask==3.0.*

# Copier l’application dans le conteneur
COPY hello.py /

# Définir la variable d’environnement pour Flask
ENV FLASK_APP=hello

# Exposer le port 8000
EXPOSE 8000

# Lancer l’application
CMD ["flask", "run", "--host", "0.0.0.0", "--port", "8000"]
```

***

### 🔎 Décomposition du Dockerfile

1.  **Dockerfile syntax**

    ```dockerfile
    # syntax=docker/dockerfile:1
    ```

    → Indique la version de syntaxe utilisée (bonne pratique).
2.  **Base image**

    ```dockerfile
    FROM ubuntu:22.04
    ```

    → On part de l’image officielle **Ubuntu 22.04**.
3.  **Installation des dépendances**

    ```dockerfile
    RUN apt-get update && apt-get install -y python3 python3-pip
    RUN pip install flask==3.0.*
    ```

    → Mise à jour des paquets, installation de **Python 3 + pip**, puis installation de **Flask**.
4.  **Ajout du code applicatif**

    ```dockerfile
    COPY hello.py /
    ```

    → On copie le fichier Python dans l’image.
5.  **Variables d’environnement**

    ```dockerfile
    ENV FLASK_APP=hello
    ```

    → Permet à Flask de savoir quel fichier exécuter.
6.  **Exposition du port**

    ```dockerfile
    EXPOSE 8000
    ```

    → Indique que le conteneur utilisera le **port 8000** (pour que l’extérieur puisse s’y connecter).
7.  **Commande par défaut (entrée du conteneur)**

    ```dockerfile
    CMD ["flask", "run", "--host", "0.0.0.0", "--port", "8000"]
    ```

    → Au démarrage du conteneur, Flask lance le serveur.

***

### 🚀 Build & Run

1.  **Construire l’image**

    ```bash
    docker build -t flask-hello .
    ```
2.  **Lancer le conteneur**

    ```bash
    docker run -p 8000:8000 flask-hello
    ```
3. **Tester dans le navigateur**\
   👉 Aller sur [http://localhost:8000](http://localhost:8000)\
   → Résultat : **Hello World!**

## 🐳 Dockerfile Syntax

👉 La **première ligne** d’un Dockerfile peut (optionnellement) contenir une **directive de syntaxe** (`# syntax=...`).

Cette directive **indique au moteur de build (BuildKit)** quelle version de la syntaxe Dockerfile utiliser.\
Elle permet aussi à des versions plus anciennes de Docker (avec BuildKit activé) d’utiliser un **frontend spécifique** pour comprendre correctement ton Dockerfile **avant de démarrer la construction**.

***

### 📌 Exemple de directive

```dockerfile
# syntax=docker/dockerfile:1
```

* `docker/dockerfile:1` → pointe toujours vers la **dernière version stable de la syntaxe v1**.
* BuildKit va automatiquement vérifier si une mise à jour de cette syntaxe est disponible **avant chaque build**.
* Ainsi, tu es toujours sûr d’utiliser la syntaxe la plus récente et compatible. ✅

***

### ⚠️ Règles importantes

1. La directive **doit être la première ligne** du Dockerfile.\
   (avant tout commentaire, espace ou instruction `FROM`, `RUN`, etc.).
2. Elle est **optionnelle**, mais fortement recommandée pour :
   * garantir la compatibilité,
   * profiter des dernières optimisations de BuildKit.

***

👉 **Bon réflexe** : toujours commencer ton Dockerfile par

```dockerfile
# syntax=docker/dockerfile:1
```

## 🐳 Base image (FROM)

👉 La ligne **juste après la directive `# syntax`** définit **l’image de base** de ton conteneur avec l’instruction :

```dockerfile
FROM ubuntu:22.04
```

***

### 📌 Rôle de l’instruction `FROM`

* Elle définit **l’environnement de départ** dans lequel toutes les instructions suivantes (`RUN`, `COPY`, `CMD`, etc.) vont être exécutées.
* Ici : ton conteneur part de **Ubuntu 22.04**, une distribution Linux stable et courante.
* Tout ce que tu ajoutes ensuite (dépendances, ton application, variables d’environnement, etc.) sera installé **par-dessus cette base**.

***

### 🔖 Notation `image:tag`

* `ubuntu` → **nom** de l’image.
* `22.04` → **tag** indiquant la version précise.
* Format général : `nom:tag`.

Exemples :

* `python:3.12` → image Python avec version 3.12.
* `node:20-alpine` → image Node.js 20 allégée (basée sur Alpine Linux).
* `debian:bullseye` → image Debian version Bullseye.

⚠️ Si tu omets le `:tag`, Docker utilisera **`:latest`** par défaut.\
Exemple : `FROM ubuntu` → équivaut à `FROM ubuntu:latest`.

***

### 🌍 Où trouver des images de base ?

* Sur **Docker Hub** : dépôt public officiel avec de nombreuses images maintenues par la communauté et les éditeurs.
* Sur des registres privés (GitHub Container Registry, GitLab, AWS ECR, GCP Artifact Registry, etc.) si tu veux contrôler tes propres images.

***

✅ En résumé :\
`FROM` est la **fondation** de ton image. Choisir une bonne base image te permet de :

* limiter la taille de ton conteneur,
* simplifier la maintenance,
* sécuriser ton application (en partant d’images minimales ou vérifiées).

## ⚙️ Environment setup

Après avoir défini une **image de base** avec `FROM`, on configure l’environnement dans lequel notre application va tourner. Cela se fait avec des instructions comme `RUN`.

Exemple :

```dockerfile
# installer les dépendances de l'application
RUN apt-get update && apt-get install -y python3 python3-pip
```

***

### 🔍 Que fait cette ligne ?

1. **`apt-get update`**
   * Met à jour l’index des paquets disponibles dans Ubuntu.
   * Nécessaire avant toute installation pour que `apt-get` connaisse les dernières versions.
2. **`apt-get install -y python3 python3-pip`**
   * Installe **Python 3** et **pip** (le gestionnaire de paquets Python).
   * L’option `-y` valide automatiquement les questions d’installation (pas d’interaction manuelle).
3. Cette commande est exécutée **dans le conteneur** basé sur l’image définie par `FROM ubuntu:22.04`.

***

### 🏗️ Pourquoi utiliser `RUN` ?

* Chaque **`RUN` crée une nouvelle couche** dans l’image Docker.
* Ces couches sont mises en cache, ce qui accélère les reconstructions ultérieures si la commande n’a pas changé.
* Permet d’installer **les dépendances système nécessaires** à ton application (ex. bibliothèques, compilateurs, utilitaires).

***

### ⚠️ Bonnes pratiques

* **Combiner plusieurs commandes** dans un seul `RUN` (comme dans l’exemple avec `&&`) pour réduire le nombre de couches.
* Nettoyer le cache APT après l’installation pour réduire la taille de l’image :

```dockerfile
RUN apt-get update && apt-get install -y python3 python3-pip \
    && rm -rf /var/lib/apt/lists/*
```

* Préférer des images spécialisées (ex : `python:3.12-slim`) plutôt que d’installer Python soi-même sur Ubuntu, afin d’obtenir une image plus légère et plus sécurisée.

***

✅ En résumé :\
L’instruction `RUN` sert à configurer l’environnement d’exécution de ton application en installant ses dépendances système. C’est l’étape où ton conteneur devient plus que juste une base : il se prépare à accueillir ton application.

## 📝 Commentaires dans un Dockerfile

Dans un **Dockerfile**, les commentaires commencent toujours par le symbole `#`.

Exemple :

```dockerfile
# installer les dépendances de l’application
RUN apt-get update && apt-get install -y python3 python3-pip
```

Ici, la ligne `# installer les dépendances de l’application` est **un commentaire**.

***

### 🎯 Pourquoi utiliser des commentaires ?

* 📖 **Documenter** ton Dockerfile pour faciliter la compréhension par d’autres développeurs (ou ton futur toi).
* 🛠️ **Expliquer des choix techniques**, par exemple pourquoi tu utilises une certaine version d’image ou un paquet spécifique.
* 🔍 **Clarifier des optimisations** (ex. nettoyage du cache APT, usage de multi-stage builds, etc.).

***

### ⚠️ Attention

* Le symbole `#` est aussi utilisé pour les **directives de syntaxe** (`# syntax=docker/dockerfile:1`) lorsqu’il est placé **tout en haut du fichier** et qu’il correspond à un **mot-clé reconnu**.
* Sinon, toute ligne commençant par `#` est simplement considérée comme un commentaire.

Exemple :

```dockerfile
# syntax=docker/dockerfile:1   # directive spéciale (doit être en première ligne)

# Ceci est juste un commentaire normal
FROM ubuntu:22.04
```

***

✅ En résumé :

* `#` = commentaire (sauf quand il s’agit d’une directive en première ligne).
* Les commentaires sont essentiels pour expliquer la logique et maintenir ton Dockerfile clair et compréhensible.

## 📦 Installation des dépendances

Après avoir configuré l’environnement de base (Ubuntu + Python + pip), on utilise une seconde instruction `RUN` pour installer les dépendances nécessaires à l’application.

Exemple :

```dockerfile
# installer Flask, nécessaire à l’application Python
RUN pip install flask==3.0.*
```

***

### 🔍 Explication

* `RUN pip install flask==3.0.*` → installe le framework **Flask** dans le conteneur.
* La notation `==3.0.*` signifie :
  * on veut **la version 3.0.x de Flask**
  * (cela inclut toutes les sous-versions comme 3.0.1, 3.0.2, etc.).

👉 Cela garantit une **compatibilité contrôlée** tout en permettant les mises à jour mineures.

***

### ⚠️ Pré-requis

* Le premier `RUN` a déjà installé **pip** avec :

```dockerfile
RUN apt-get update && apt-get install -y python3 python3-pip
```

Sans cette étape, la commande `pip install flask==3.0.*` échouerait, car `pip` n’existerait pas encore dans le conteneur.

***

✅ Résultat : l’image contiendra **Python + pip + Flask** prêts à exécuter ton application.

## 📂 Copie des fichiers dans l’image

L’instruction suivante ajoute ton code source dans le conteneur :

```dockerfile
COPY hello.py /
```

***

### 🔍 Explication

* `COPY <src> <dest>` → copie un fichier ou un dossier **du contexte de build local** (`<src>`) vers le système de fichiers **de l’image en cours de construction** (`<dest>`).

Ici :

* `hello.py` → fichier présent dans ton projet local (dans le _build context_).
* `/` → chemin de destination à la racine du conteneur.

Après cette commande, le fichier sera disponible **dans le conteneur**, accessible depuis `/hello.py`.

***

### 📌 Build context

👉 Le _build context_ correspond aux fichiers accessibles pour les instructions `COPY` ou `ADD`.\
Il est défini lors du `docker build`.\
Exemple :

```bash
docker build -t myapp .
```

Le `.` indique que **tout le dossier courant** est envoyé au démon Docker comme _context_.

⚠️ Attention : si ton dossier contient beaucoup de fichiers inutiles (logs, node\_modules, etc.), cela peut ralentir le build.\
➡️ C’est pourquoi on utilise souvent un `.dockerignore` pour **exclure** certains fichiers.

***

✅ Résultat : ton application `hello.py` est incluse dans l’image et pourra être exécutée par les prochaines instructions (`CMD`).

## 🌱 Définir des variables avec `ENV`

Dans ton exemple :

```dockerfile
ENV FLASK_APP=hello
```

***

### 🔍 Explication

* `ENV <clé>=<valeur>` → crée une variable d’environnement **disponible dans l’image** et dans tous les conteneurs démarrés à partir de cette image.

Ici :

* `FLASK_APP=hello`\
  → permet à Flask de savoir **quel fichier ou module Python** lancer quand on démarre l’application.

Sans ça, si tu faisais simplement `flask run`, Flask n’aurait pas su quel fichier exécuter.

***

### 📌 Points importants sur `ENV`

1. Les variables définies par `ENV` :
   * sont **permanentes** dans l’image.
   * sont disponibles pour **tous les processus du conteneur**.
2.  Tu peux en définir plusieurs :

    ```dockerfile
    ENV PYTHONUNBUFFERED=1 \
        APP_ENV=production \
        DEBUG=0
    ```
3.  Tu peux aussi les **surcharger à l’exécution** avec `docker run -e`:

    ```bash
    docker run -e FLASK_APP=other_app flask-image
    ```

***

✅ Dans ton Dockerfile, cette ligne prépare ton environnement pour que `CMD ["flask", "run", ...]` fonctionne correctement.

## 🔌 Exposed ports avec `EXPOSE`

Dans ton exemple :

```dockerfile
EXPOSE 8000
```

***

### 🔍 Explication

* `EXPOSE <port>` → indique que l’application à l’intérieur du conteneur **écoute** sur ce port.
* Ici : `8000` → Flask démarre son serveur web sur ce port.

⚠️ **Attention** :

* `EXPOSE` **ne publie pas** le port vers ta machine hôte.
* Il sert plutôt de **documentation intégrée** dans l’image et permet à Docker (et aux autres développeurs) de comprendre que ton service tourne sur ce port.

***

### 📌 Exemple pratique

#### Avec `EXPOSE` seulement

```bash
docker run flask-image
```

→ Le conteneur démarre Flask, mais tu ne peux pas y accéder depuis ton hôte directement.

#### Avec publication de port

```bash
docker run -p 8000:8000 flask-image
```

→ Ici :

* `-p 8000:8000` publie le port du conteneur (`8000`) vers le port `8000` de ta machine hôte.
* Tu peux alors accéder à ton app Flask via `http://localhost:8000`.

***

### ✅ Bonnes pratiques

* Toujours indiquer les ports avec `EXPOSE` → **documentation utile** pour l’équipe et les outils d’orchestration (ex. Docker Compose, Kubernetes).
* Publier les ports explicitement avec `-p` ou dans un `docker-compose.yml` → pour rendre le service accessible.

## 🚀 Starting the application avec `CMD`

Dans ton exemple :

```dockerfile
CMD ["flask", "run", "--host", "0.0.0.0", "--port", "8000"]
```

***

### 🔍 Explication

* **`CMD`** définit la **commande par défaut** exécutée quand on démarre un conteneur basé sur cette image.
* Ici, elle lance le serveur Flask en écoutant sur **toutes les interfaces (`0.0.0.0`)** au **port 8000**.

***

### ⚙️ Deux formes possibles

#### 1. **Forme exec (recommandée ✅)**

```dockerfile
CMD ["flask", "run", "--host", "0.0.0.0", "--port", "8000"]
```

* Chaque argument est séparé.
* Pas d’intermédiaire shell (`/bin/sh -c`).
* Meilleure gestion des **signaux système** (ex. `SIGTERM`, `SIGKILL`), utile pour arrêter proprement le conteneur.

#### 2. **Forme shell**

```dockerfile
CMD flask run --host 0.0.0.0 --port 8000
```

* Exécuté via `/bin/sh -c`.
* Plus flexible (on peut utiliser des variables shell, pipes `|`, redirections, etc.).
* Mais : moins bonne gestion des signaux (risque de processus « zombies » si mal géré).

***

### ✅ Bonnes pratiques

* Utiliser la **forme exec** par défaut.
* Garder **un seul `CMD`** dans ton Dockerfile (si plusieurs, seul le dernier est pris en compte).
* Si tu veux rendre ton image plus flexible (par ex. changer la commande au runtime), utilise `ENTRYPOINT` + `CMD` ensemble.

## 🛠️ **Construire l’image**

Commande :

```bash
docker build -t test:latest .
```

#### 🔍 Explications

* **`-t test:latest`** → donne un **nom** (`test`) et un **tag** (`latest`) à l’image.
* **`.` (point final)** → indique le **contexte de build**, ici le dossier courant.
  * Ce dossier doit contenir **Dockerfile** et **hello.py**.
  * Si l’un des deux manque → le build échoue.

***

## 🚀 **Exécuter le conteneur**

Commande :

```bash
docker run -p 127.0.0.1:8000:8000 test:latest
```

#### 🔍 Explications

* **`-p 127.0.0.1:8000:8000`** → publie le port interne `8000` du conteneur vers `http://localhost:8000` sur ta machine.
  * Format : `hôte:port_hôte:port_conteneur`.
* **`test:latest`** → image à lancer.

👉 Après ça, ton app Flask est accessible sur ton navigateur via :

```
http://127.0.0.1:8000
```

***

## 💡 Astuce

Dans **VS Code**, tu peux installer l’extension **Docker** pour :

* **linter** ton Dockerfile (détecter erreurs de syntaxe et optimisations possibles)
* faire de la **navigation de code** dans tes images
* scanner tes images pour détecter des **vulnérabilités**
