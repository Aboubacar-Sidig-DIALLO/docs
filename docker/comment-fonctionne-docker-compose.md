# 🐳 Comment fonctionne Docker Compose

👉 **Docker Compose** est un outil qui permet de gérer des applications **multi-conteneurs** avec un simple fichier de configuration **YAML** (`compose.yaml`).

Au lieu de lancer tes conteneurs un par un avec `docker run`, tu peux décrire **toute ton application** (services, réseaux, volumes, secrets, etc.) dans un fichier, puis tout démarrer avec une seule commande :

```bash
docker compose up
```

***

### 🧩 Le modèle d’application Compose

#### 🔹 Services

* Ce sont les **briques de calcul** de ton application.
* Un service correspond en pratique à **un ou plusieurs conteneurs** créés à partir de la **même image** et configuration.
* Exemple :
  * `web` (serveur Node.js)
  * `db` (base PostgreSQL)
  * `redis` (cache mémoire)

***

#### 🔹 Réseaux

* Les services doivent **communiquer entre eux**.
* Compose crée automatiquement un **réseau par défaut** pour que les services puissent se voir par leur **nom**.
* Exemple : le service `web` peut contacter la base avec `db:5432` (sans IP fixe).

***

#### 🔹 Volumes

* Utilisés pour stocker et partager des **données persistantes**.
* Contrairement aux conteneurs (éphémères), un **volume** conserve les données même si tu supprimes le conteneur.
* Exemple : volume pour `/var/lib/postgresql/data`.

***

#### 🔹 Configs

* Données de configuration spécifiques au **runtime** ou à la plateforme.
* Montées dans les conteneurs comme des fichiers (un peu comme un volume).
* Exemple : un fichier `nginx.conf`.

***

#### 🔹 Secrets

* Similaires aux `configs`, mais dédiés aux **données sensibles** (mots de passe, clés API, certificats).
* Disponibles uniquement de manière sécurisée dans les conteneurs (montés comme fichiers).
* Exemple : un mot de passe MySQL.

***

### 📦 Le projet (project)

Un **projet** est une instance d’une application définie par ton `compose.yaml`.

* Le **nom du projet** (`name:` en haut du fichier) sert à **grouper et isoler** toutes les ressources.
* Cela permet d’avoir plusieurs déploiements de la **même application** sur une infrastructure, avec seulement un nom différent.
* Par défaut, le nom du projet correspond au **nom du dossier** où se trouve le fichier `compose.yaml`.

👉 Exemple :

* Projet `app1` → crée des conteneurs `app1_web`, `app1_db`
* Projet `app2` (même compose.yaml mais lancé avec `-p app2`) → crée `app2_web`, `app2_db`

***

### ⚡ En résumé

* **Services** → définissent les conteneurs.
* **Réseaux** → permettent la communication.
* **Volumes** → stockent les données persistantes.
* **Configs & Secrets** → fournissent des fichiers de config et infos sensibles.
* **Projet** → groupe toutes les ressources et permet plusieurs déploiements isolés.

## 📄 Le fichier Compose

Le fichier principal utilisé par **Docker Compose** est le **`compose.yaml`** (forme recommandée).\
👉 Les noms suivants restent supportés pour compatibilité :

* `compose.yml`
* `docker-compose.yaml`
* `docker-compose.yml`

⚠️ Si plusieurs fichiers existent, Compose choisira **en priorité** `compose.yaml`.

***

### 🧩 Organisation et extensions

Pour garder ton fichier clair et facile à maintenir :

* Tu peux utiliser des **fragments** et **extensions** YAML.
* Tu peux combiner **plusieurs fichiers Compose** :
  * Les attributs simples ou maps sont **écrasés** par le dernier fichier.
  * Les listes sont **fusionnées par ajout**.
  * Les chemins relatifs sont résolus à partir du **premier fichier**.

👉 Cela permet de séparer des morceaux d’application (exemple : dev, prod, CI/CD) ou de **réutiliser** des Compose files via `include`.

***

### 🖥️ CLI (Interface en ligne de commande)

Le client **Docker CLI** permet de piloter ton application multi-conteneurs avec :

#### Commandes principales ⚡

* **Démarrer tous les services** :

```bash
docker compose up
```

* **Arrêter et supprimer les services** :

```bash
docker compose down
```

* **Voir les logs en direct** (utile pour déboguer) :

```bash
docker compose logs
```

* **Lister les services et leur état** :

```bash
docker compose ps
```

👉 La liste complète des commandes est dispo dans la documentation de référence.

***

### 🌐 Exemple illustratif

Imaginons une application avec :

* Un **frontend** (app web)
* Un **backend** (base de données)
* Un **secret** (certificat HTTPS)
* Une **config** (fichier HTTP externe)
* Un **volume** (stockage persistant pour la DB)
* Deux **réseaux** (communication sécurisée interne et externe)

#### 📝 Exemple de `compose.yaml`

```yaml
services:
  frontend:
    image: example/webapp
    ports:
      - "443:8043"
    networks:
      - front-tier
      - back-tier
    configs:
      - httpd-config
    secrets:
      - server-certificate

  backend:
    image: example/database
    volumes:
      - db-data:/etc/data
    networks:
      - back-tier

volumes:
  db-data:
    driver: flocker
    driver_opts:
      size: "10GiB"

configs:
  httpd-config:
    external: true

secrets:
  server-certificate:
    external: true

networks:
  # Définir les réseaux suffit
  front-tier: {}
  back-tier: {}
```

***

### 🚀 Exécution

1. **Démarrer l’application** :

```bash
docker compose up
```

Cela :

* Lance `frontend` et `backend`
* Crée les réseaux `front-tier` et `back-tier`
* Monte le volume `db-data`
* Injecte le `httpd-config` et `server-certificate` dans le frontend

2. **Lister les services** :

```bash
docker compose ps
```

#### Exemple de sortie :

```
NAME                IMAGE                COMMAND                  SERVICE    CREATED        STATUS       PORTS
example-frontend-1  example/webapp       "nginx -g 'daemon of…"   frontend   2 minutes ago  Up 2m        0.0.0.0:443->8043/tcp
example-backend-1   example/database     "docker-entrypoint.s…"   backend    2 min
```
