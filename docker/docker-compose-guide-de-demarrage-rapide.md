# ⚡ Docker Compose – Guide de démarrage rapide

Ce tutoriel introduit les **concepts fondamentaux de Docker Compose** en développant une simple application web Python avec **Flask** et un compteur de visites stocké dans **Redis**.

👉 Même si tu ne connais pas Python, l’objectif est surtout de comprendre **comment Compose orchestre plusieurs conteneurs**.

***

### ✅ Prérequis

* Avoir installé **Docker** et **Docker Compose** (plugin ou standalone).
* Avoir des notions de base sur Docker (images, conteneurs).

***

### 🥇 Étape 1 : Préparer le projet

Crée un dossier de travail :

```bash
mkdir composetest
cd composetest
```

#### 1. Fichier `app.py`

Crée un fichier **`app.py`** :

```python
import time
import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return f'Hello World! I have been seen {count} times.\n'
```

👉 Ici :

* **Flask** = serveur web minimaliste
* **Redis** = stockage des compteurs (le host `redis` correspond au service défini dans `compose.yaml`).
* La fonction `get_hit_count()` gère les tentatives de reconnexion si Redis n’est pas dispo.

***

#### 2. Fichier `requirements.txt`

Crée un fichier **`requirements.txt`** :

```
flask
redis
```

***

#### 3. Fichier `Dockerfile`

Crée un fichier **`Dockerfile`** :

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.10-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run", "--debug"]
```

👉 Ce Dockerfile :

* Utilise **Python 3.10** basé sur Alpine (léger).
* Installe les dépendances de compilation (`gcc`, etc.).
* Installe les libs Python (`flask`, `redis`).
* Expose le port **5000**.
* Lance l’appli Flask.

⚠️ Attention : vérifie que le fichier s’appelle bien `Dockerfile` (sans `.txt`).

***

### 🥈 Étape 2 : Créer le fichier Compose

Crée un fichier **`compose.yaml`** :

```yaml
services:
  web:
    build: .
    ports:
      - "8000:5000"
  redis:
    image: "redis:alpine"
```

👉 Explications :

* `web` : construit l’image à partir du `Dockerfile` local, expose **5000** (Flask) vers **8000** sur ta machine.
* `redis` : utilise l’image officielle **Redis Alpine** depuis Docker Hub.

***

### 🥉 Étape 3 : Construire et lancer l’application

Depuis le dossier du projet, lance :

```bash
docker compose up
```

Tu verras :

* Un réseau créé automatiquement (`composetest_default`).
* Deux conteneurs : `web` (Flask) et `redis`.

Logs attendus :

```
web_1    |  * Running on http://0.0.0.0:5000/
redis_1  |  * Ready to accept connections
```

***

### 🌍 Étape 4 : Tester

Ouvre un navigateur sur [http://localhost:8000](http://localhost:8000)

👉 Résultat attendu :

```
Hello World! I have been seen 1 times.
```

Chaque rafraîchissement de la page incrémente le compteur stocké dans Redis. 🎉

***

### 🛑 Arrêter les services

Pour arrêter et nettoyer les conteneurs :

```bash
docker compose down
```

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Si tu rafraîchis la page [http://localhost:8000](http://localhost:8000) :

👉 Redis incrémente la clé `hits`, donc le compteur augmente à chaque visite.

Par exemple :

```
Hello World! I have been seen 2 times.
```

➡️ Si tu continues à rafraîchir :

* 3e visite → `Hello World! I have been seen 3 times.`
* 4e visite → `Hello World! I have been seen 4 times.`
* etc.

C’est Redis qui garde la donnée persistante tant que le conteneur `redis` est actif.

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

#### ✅ Étape 3 – Vérification des images

Quand tu exécutes :

```bash
docker image ls
```

Tu vois :

* `composetest_web` → ton image Python/Flask construite à partir de ton `Dockerfile`.
* `redis:alpine` → image officielle tirée de Docker Hub.
* L’image de base `python:3.10-alpine` (ou une autre selon ton Dockerfile).

👉 Ça confirme que tes services sont bien construits et prêts.

***

#### ✅ Étape 4 – Activation de **Compose Watch**

Tu modifies `compose.yaml` :

```yaml
services:
  web:
    build: .
    ports:
      - "8000:5000"
    develop:
      watch:
        - action: sync
          path: .
          target: /code
  redis:
    image: "redis:alpine"
```

🔎 Ce que ça change :

* Dès que tu modifies un fichier dans ton projet, il est **synchronisé automatiquement** dans le conteneur (`/code`).
* Flask est lancé en mode `--debug` → il recharge automatiquement ton code Python.
* Pas besoin de refaire `docker compose up` à chaque modification.

***

#### ✅ Étape 5 – Relancer avec Watch

Commande :

```bash
docker compose watch
```

Tu vois :

```
⦿ watch enabled
```

👉 Ton app se reconstruit au besoin et surveille tes fichiers.

Tu peux ouvrir ton navigateur :\
➡️ `http://localhost:8000`\
Le compteur s’incrémente toujours.

***

#### ✅ Étape 6 – Tester la mise à jour à chaud

Change ton `app.py` :

```python
return f'Hello from Docker! I have been seen {count} times.\n'
```

Sauvegarde → Compose Watch détecte le changement → Flask recharge → sans reconstruire l’image, ton app affiche&#x20;

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

(ou autre chiffre selon ton compteur Redis).

***

👉 Quand tu as fini, tu arrêtes tout proprement avec :

```bash
docker compose down
```

### 🔹 Étape 7 – Scinder les services

👉 Objectif : séparer tes services dans plusieurs fichiers pour rendre ton projet **plus lisible et maintenable**.

#### 1. Nouveau fichier `infra.yaml`

Tu crées un fichier à la racine du projet :

```yaml
services:
  redis:
    image: "redis:alpine"
```

Ici, tu n’as que **Redis**, isolé dans un fichier d’infrastructure.

***

#### 2. Mise à jour du `compose.yaml`

Tu ajoutes l’inclusion de ton fichier d’infra et gardes ton service `web` :

```yaml
include:
  - infra.yaml

services:
  web:
    build: .
    ports:
      - "8000:5000"
    develop:
      watch:
        - action: sync
          path: .
          target: /code
```

🔎 Résultat : Compose fusionne automatiquement les deux fichiers → ton app a toujours `web` et `redis`, mais maintenant chaque partie peut être gérée séparément.

C’est très utile pour des **gros projets** où une équipe s’occupe du backend et une autre de la base de données par exemple.

***

#### 3. Lancer l’application

```bash
docker compose up
```

➡️ Tu devrais revoir ton `Hello World` avec compteur dans le navigateur.\
&#xNAN;_(Compose va construire `web` et lancer `redis` à partir d’`infra.yaml`.)_

***

### 🔹 Étape 8 – Expérimenter d’autres commandes

#### Démarrage en arrière-plan

```bash
docker compose up -d
```

👉 Tu ne vois plus les logs dans ton terminal, mais les conteneurs tournent.

***

#### Vérifier l’état des services

```bash
docker compose ps
```

Exemple de sortie :

```
       Name                      Command               State           Ports         
-------------------------------------------------------------------------------------
composetest_redis_1   docker-entrypoint.sh redis ...   Up      6379/tcp              
composetest_web_1     flask run                        Up      0.0.0.0:8000->5000/tcp
```

***

#### Arrêter sans supprimer

```bash
docker compose stop
```

➡️ Conteneurs arrêtés mais toujours présents (`docker ps -a` les montre encore).

***

#### Tout supprimer (containers, réseaux, etc.)

```bash
docker compose down
```

➡️ Ici tu reviens à zéro, sans casser tes images (`docker image ls` les garde).

***

✨ Tu viens de voir comment **structurer ton projet Compose** et **gérer son cycle de vie complet**.
