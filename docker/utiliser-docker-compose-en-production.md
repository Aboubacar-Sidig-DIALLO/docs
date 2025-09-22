# 🚀 Utiliser Docker Compose en production

Quand tu développes une application avec **Docker Compose**, tu peux réutiliser la même définition pour l’exécuter dans différents environnements :

* **CI/CD** (tests automatisés)
* **Staging** (pré-production)
* **Production** (serveurs réels)

***

### 1. Déploiement le plus simple : un seul serveur

Tu peux déployer ton application de la même manière qu’en développement, mais en adaptant ta configuration :

👉 Exemple classique :

```bash
docker compose -f compose.yaml -f compose.prod.yaml up -d
```

* `compose.yaml` → ta configuration de base.
* `compose.prod.yaml` → ton fichier d’override pour la production (optimisations, variables, sécurité…).
* `-d` → mode détaché (en arrière-plan).

***

### 2. Fichier de configuration dédié

En production, tu as souvent :

* **plusieurs fichiers Compose** (`compose.yaml`, `compose.override.yaml`, `compose.prod.yaml`).
* Des **.env spécifiques** (`.env.prod`, `.env.staging`, `.env.ci`).

👉 Exemple d’organisation :

```
project/
├── compose.yaml
├── compose.override.yaml  # utilisé en local/dev
├── compose.prod.yaml      # override production
├── .env                   # variables locales
├── .env.prod              # variables production
```

Et pour lancer en production :

```bash
docker compose --env-file .env.prod -f compose.yaml -f compose.prod.yaml up -d
```

***

### 3. Mise à l’échelle (scaling)

En production, tu peux **scaler** certains services.\
👉 Exemple : 3 instances pour ton backend :

```bash
docker compose up -d --scale backend=3
```

***

### 4. Déploiement sur un cluster (Swarm)

Si ton application doit tourner sur plusieurs serveurs (multi-hôtes), tu peux utiliser :

* **Docker Swarm** (natif Docker)
* ou bien **Kubernetes** (plus complexe, mais standard industriel).

👉 Exemple avec Swarm :

```bash
docker swarm init
docker stack deploy -c compose.yaml myapp
```

***

### 5. Bonnes pratiques en production

✔️ Utiliser **fichiers séparés** pour dev/staging/prod.\
✔️ Utiliser **secrets** au lieu des variables d’environnement pour les mots de passe/API keys.\
✔️ Ajouter des **healthchecks** pour que les services dépendent d’une base “prête” avant de démarrer.\
✔️ Configurer des **volumes persistants** pour la BDD (sinon les données sont perdues au redéploiement).\
✔️ Surveiller avec des outils comme **Prometheus + Grafana** ou **ELK stack**.\
✔️ Automatiser le déploiement avec **CI/CD pipelines** (GitHub Actions, GitLab CI, Jenkins…).

## 🛠️ Modifier ton fichier Compose pour la production

Quand tu passes de **développement → production**, il est essentiel d’ajuster ta configuration afin que ton application soit **plus robuste, sécurisée et performante**.

***

### ✅ Changements fréquents pour la production

1. **Supprimer les volumes liés au code**
   * En dev, tu montes ton code source avec un volume (`.:/app`) pour profiter du **hot reload**.
   * En prod, ton code doit être **intégré dans l’image**.
   * Cela empêche les modifications accidentelles depuis l’extérieur.

***

2. **Changer les ports d’exposition**
   * Exemple : en dev tu exposes ton backend sur `localhost:3000`, mais en prod tu voudras peut-être exposer sur `:80` ou derrière un **reverse proxy (nginx/traefik)**.

***

3. **Configurer les variables d’environnement différemment**
   * Activer/désactiver certains comportements :
     * Logs plus **verbeux** en dev.
     * Logs **minimisés** ou envoyés à un **système externe (ELK, Loki)** en prod.
   * Spécifier des **services externes** : ex. un serveur mail, des clés API, etc.

***

4.  **Ajouter une politique de redémarrage**

    ```yaml
    restart: always
    ```

    👉 permet d’éviter les interruptions si un service crash.

***

5. **Ajouter des services annexes**
   * Agrégateur de logs (ex. Fluentd, Loki, ELK).
   * Monitoring (Prometheus, Grafana).
   * Reverse proxy (Nginx, Traefik).

***

### 🗂️ Exemple : fichier Compose production

#### `compose.yaml` (base – dev)

```yaml
services:
  web:
    build: .
    volumes:
      - .:/app   # montage du code pour dev
    ports:
      - "3000:3000"
    environment:
      - DEBUG=true
```

#### `compose.production.yaml` (overrides – prod)

```yaml
services:
  web:
    ports:
      - "80:3000"         # expose sur le port 80
    volumes: []           # pas de montage du code en prod
    environment:
      - DEBUG=false       # logs réduits
      - MAIL_SERVER=smtp.example.com
    restart: always       # auto-redémarrage en cas de crash
```

***

### 🚀 Déploiement

Ensuite tu lances :

```bash
docker compose -f compose.yaml -f compose.production.yaml up -d
```

👉 Résultat :

* La config **de base** (`compose.yaml`) est appliquée.
* Les règles de **production** (`compose.production.yaml`) **écrasent ou complètent** les paramètres initiaux.

***

🔎 Conseil : tu peux même avoir plusieurs fichiers (ex. `compose.staging.yaml`, `compose.ci.yaml`) pour différents environnements.

## 🔄 Déployer des changements avec Docker Compose

Quand tu modifies ton **code applicatif** (par ex. ton backend ou ton frontend), il faut :

1. **Reconstruire l’image** mise à jour.
2. **Recréer le(s) conteneur(s)** correspondant(s) pour appliquer les modifications.

***

### 📝 Exemple : service `web`

```bash
docker compose build web
docker compose up --no-deps -d web
```

#### 🔍 Explication pas à pas :

1. **`docker compose build web`**
   * Reconstruit l’image pour le service `web`.
   * Utilise ton `Dockerfile` et le contexte défini dans `compose.yaml`.
2. **`docker compose up --no-deps -d web`**
   * Arrête, supprime puis recrée **uniquement** le service `web`.
   * Le flag `--no-deps` empêche Compose de redémarrer les services dont `web` dépend (ex. `db`).
   * L’option `-d` lance en **mode détaché** (en arrière-plan).

***

### ✅ Résumé

* Tu reconstruis ton image avec **build**.
* Tu redéploies uniquement ton service avec **up --no-deps -d**.
* Cela évite d’impacter inutilement les autres services (ex. ta base de données).

***

💡 Astuce : si tu veux **forcer la reconstruction + redéploiement** de toute l’app :

```bash
docker compose up --build -d
```

## 🖥️ Exécuter Docker Compose sur un serveur unique

### 🌍 Déployer à distance avec Docker Compose

Tu peux utiliser **Docker Compose** non seulement en local, mais aussi pour déployer ton application sur un **hôte Docker distant** (par ex. un serveur dans le cloud).

👉 Pour cela, tu dois configurer correctement certaines variables d’environnement afin que Docker CLI et Compose sachent **où envoyer les commandes**.

***

### ⚙️ Variables nécessaires

*   **`DOCKER_HOST`**\
    Définit l’adresse du démon Docker distant.\
    Exemple :

    ```bash
    export DOCKER_HOST=tcp://192.168.1.10:2376
    ```
*   **`DOCKER_TLS_VERIFY`**\
    Définit si la connexion doit utiliser TLS (sécurisée).

    * `1` → activer TLS
    * `0` → désactiver TLS

    Exemple :

    ```bash
    export DOCKER_TLS_VERIFY=1
    ```
*   **`DOCKER_CERT_PATH`**\
    Chemin vers les certificats TLS pour sécuriser la connexion.\
    Exemple :

    ```bash
    export DOCKER_CERT_PATH=$HOME/.docker/certs
    ```

***

### 🚀 Utilisation

Une fois les variables définies, tu peux exécuter **les mêmes commandes Docker Compose que d’habitude**, sans configuration supplémentaire.

👉 Exemple :

```bash
docker compose up -d
```

Cette commande va :

* contacter le démon Docker distant (grâce à `DOCKER_HOST`),
* déployer les services définis dans ton `compose.yaml`,
* lancer ton application **directement sur le serveur**.

***

✅ **Résumé :**

1. Configure tes variables (`DOCKER_HOST`, `DOCKER_TLS_VERIFY`, `DOCKER_CERT_PATH`).
2. Lance tes commandes Docker Compose comme en local.
3. Ton application tourne sur ton **serveur unique distant**.
