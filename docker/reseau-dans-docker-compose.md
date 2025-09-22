# 🌐 Réseau dans Docker Compose

### ⚠️ Important

La documentation Docker fait référence aux fonctionnalités de **Compose V2**.

* Depuis **juillet 2023**, **Compose V1** n’est plus mis à jour ❌ et n’est plus inclus dans les nouvelles versions de **Docker Desktop**.
* **Compose V2** est désormais la version par défaut et est intégré dans toutes les versions actuelles de Docker Desktop.\
  👉 Pour plus d’informations, voir : **Migrate to Compose V2**.

***

### 🔧 Fonctionnement par défaut

Par défaut, **Compose crée un seul réseau** 🕸️ pour votre application.

* Chaque **conteneur** d’un service rejoint ce **réseau par défaut**.
* Les conteneurs peuvent alors :
  * **communiquer entre eux** 🌍,
  * et être **découverts via le nom du service** 🏷️.

***

### 📝 Note sur le nom du réseau

Le réseau de votre application reçoit un nom basé sur le **nom du projet**.

* Le nom du projet est dérivé du **nom du répertoire** qui contient le fichier `compose.yaml`.
* Vous pouvez le **surcharger** avec :
  * l’option `--project-name` dans la ligne de commande, ou
  * la variable d’environnement **`COMPOSE_PROJECT_NAME`**.

***

### 📑 Exemple

Supposons que votre application soit dans un répertoire nommé **`myapp`**, avec ce `compose.yaml` :

```yaml
services:
  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres
    ports:
      - "8001:5432"
```

#### ▶️ Quand vous exécutez :

```bash
docker compose up
```

Voici ce qui se passe :

1. Un réseau nommé **`myapp_default`** est créé.
2. Un conteneur est créé à partir du service **web** → il rejoint le réseau sous le nom **`web`**.
3. Un conteneur est créé à partir du service **db** → il rejoint le réseau sous le nom **`db`**.

👉 Chaque conteneur peut maintenant résoudre les noms des services (**web** ou **db**) en adresses IP correspondantes.

***

### 📡 Communication entre services

Dans cet exemple :

* Le code de l’application **web** peut se connecter à la base de données via :

```
postgres://db:5432
```

***

### 🔎 Distinction importante : `HOST_PORT` vs `CONTAINER_PORT`

* **`CONTAINER_PORT`** = port interne exposé dans le conteneur (ici **5432**, port par défaut de Postgres).
* **`HOST_PORT`** = port mappé sur la machine hôte (ici **8001**).

👉 Communication **entre services Compose** = utilise toujours le **`CONTAINER_PORT`**.\
👉 Communication **depuis l’extérieur (machine hôte)** = utilise le **`HOST_PORT`**.

***

#### Exemple concret

*   Depuis **web** (dans le conteneur) → connexion :

    ```
    postgres://db:5432
    ```
*   Depuis votre **machine hôte** (local) → connexion :

    ```
    postgres://localhost:8001
    ```

    (ou `postgres://{DOCKER_IP}:8001` si Docker tourne sur une VM).

***

✅ En résumé :

* Compose crée automatiquement un réseau privé pour vos services.
* Les conteneurs communiquent entre eux via leurs **noms de service**.
* `CONTAINER_PORT` = communication interne, `HOST_PORT` = accès externe.

## 🔄 Mise à jour des conteneurs sur le réseau

### ⚙️ Comportement lors d’une mise à jour

*   Si vous modifiez la configuration d’un service puis exécutez :

    ```bash
    docker compose up
    ```

    ➝ l’ancien conteneur est **supprimé** ❌ et remplacé par un **nouveau conteneur** ✅.
* Ce nouveau conteneur :
  * rejoint le **même réseau**,
  * avec une **nouvelle adresse IP**,
  * mais **conserve le même nom de service** 🏷️.

***

### 🔎 Conséquences réseau

* Les autres conteneurs peuvent toujours se connecter à ce service via son **nom**.
* Mais l’**ancienne adresse IP** n’est plus valide 🚫.

👉 Si un conteneur avait une connexion **ouverte vers l’ancien conteneur**, cette connexion est **fermée** automatiquement.\
👉 C’est alors à l’application dans le conteneur de :

1. détecter que la connexion est perdue,
2. rechercher à nouveau le service par son **nom**,
3. et se reconnecter 🔁.

***

### 💡 Astuce

➡️ **Toujours référencer les conteneurs par leur nom de service et non par leur adresse IP**.\
Sinon, vous devrez **mettre à jour en permanence les adresses IP** utilisées lors de chaque redéploiement.

***

✅ En résumé :

* Un nouveau conteneur prend la place de l’ancien lors d’une mise à jour.
* Le **nom du service reste stable**, mais l’**IP change**.
* Utilisez toujours le **nom du service** (ex. `db`, `web`) pour des connexions fiables.

## 🔗 Lier des conteneurs avec **links**

### 🧐 Qu’est-ce que les _links_ ?

Les **links** permettent de définir des **alias supplémentaires** 🔀 grâce auxquels un service peut être accessible depuis un autre service.

👉 Attention :

* Les links ne sont **pas nécessaires** pour permettre aux services de communiquer entre eux.
* Par défaut, **tout service** peut joindre **tout autre service** via son **nom** (ex. `db`, `web`).

***

### 📑 Exemple

```yaml
services:

  web:
    build: .
    links:
      - "db:database"

  db:
    image: postgres
```

***

#### ▶️ Explication

* Ici, le service **`db`** est accessible depuis **`web`** :
  * avec le nom **`db`** (nom du service),
  * mais aussi avec l’alias **`database`** (grâce au link).

Ainsi, depuis le conteneur `web`, vous pouvez vous connecter à la base Postgres en utilisant soit :

```
database:5432
```

ou

```
db:5432
```

***

### 📚 Référence

👉 Voir la documentation officielle : **links reference** pour plus de détails.

***

✅ En résumé :

* `links` = ajoute des **alias de connexion** entre services.
* Non requis pour la communication (déjà possible via le nom du service).
* Utile pour donner des **noms plus explicites** ou rétro-compatibles 🔖.

## 🌍 Réseau multi-hôtes avec Docker Compose

### 🔧 Fonctionnement

Lorsque vous déployez une application Compose sur un **Docker Engine avec le mode Swarm activé** 🐝, vous pouvez utiliser le **driver overlay intégré** pour permettre la **communication entre plusieurs hôtes** 🖥️↔️🖥️.

***

### 📌 Caractéristiques des réseaux overlay

* Les réseaux **overlay** sont toujours créés comme **attachables** par défaut.
* Vous pouvez cependant définir la propriété **`attachable: false`** si vous souhaitez empêcher les conteneurs autonomes (non gérés par Compose ou Swarm) de se connecter à ce réseau.

***

### 📚 Ressources associées

👉 Consultez :

* La section **Swarm mode** pour apprendre à configurer un **cluster Swarm** 🐳.
* Le guide **Getting started with multi-host networking** pour comprendre en détail le fonctionnement des **réseaux overlay multi-hôtes**.

***

✅ En résumé :

* **Swarm + overlay networks** = permet la **communication transparente entre plusieurs machines hôtes**.
* Par défaut → les réseaux overlay sont **attachables**.
* Idéal pour les déploiements distribués à grande échelle.

## 🌐 Définir des réseaux personnalisés dans Docker Compose

### 🛠️ Pourquoi utiliser des réseaux personnalisés ?

Par défaut, Compose crée un **réseau unique** pour votre application (par ex. `myapp_default`).\
Mais vous pouvez définir vos **propres réseaux** avec la clé top-level **`networks`**.

👉 Cela permet :

* de créer des **topologies réseau plus complexes**,
* de définir des **drivers de réseau personnalisés** et leurs options,
* ou encore de connecter des services à des **réseaux externes** non gérés par Compose.

***

### 📌 Définition des réseaux par service

Chaque service peut indiquer à quels réseaux il se connecte grâce à l’attribut **`networks`** (au niveau du service).\
Cet attribut prend une liste de noms qui correspondent aux réseaux définis sous la clé top-level **`networks`**.

***

### 📝 Exemple : deux réseaux distincts

```yaml
services:
  proxy:
    build: ./proxy
    networks:
      - frontend

  app:
    build: ./app
    networks:
      - frontend
      - backend

  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
    # Exemple avec un driver et des options
    driver: bridge
    driver_opts:
      com.docker.network.bridge.host_binding_ipv4: "127.0.0.1"
  backend:
    # Exemple avec un driver personnalisé
    driver: custom-driver
```

***

#### ▶️ Explication

* `proxy` est connecté uniquement au réseau **frontend**.
* `db` est connecté uniquement au réseau **backend**.
* `app` est connecté **aux deux réseaux** → il sert donc de "pont" entre `proxy` et `db`.

👉 Résultat :

* `proxy` et `db` sont **isolés** (aucune communication directe).
* Seul `app` peut communiquer avec les deux.

***

### 🌍 Options supplémentaires

#### 1. 📌 Adresses IP statiques

Vous pouvez attribuer des **adresses IP fixes** dans vos réseaux :

```yaml
services:
  app:
    networks:
      frontend:
        ipv4_address: 172.16.238.10
```

***

#### 2. 🏷️ Nom personnalisé pour un réseau

Vous pouvez donner un **nom explicite** à vos réseaux :

```yaml
services:
  # ...
networks:
  frontend:
    name: custom_frontend
    driver: custom-driver-1
```

➡️ Ici, le réseau créé s’appelle `custom_frontend` (au lieu de `projectname_frontend`).

***

✅ En résumé :

* La clé **`networks`** (top-level) vous permet de définir vos propres réseaux.
* Chaque service déclare à quels réseaux il appartient.
* Cela permet d’isoler, de sécuriser et de mieux organiser la communication entre services.
* Possibilité de personnaliser : **drivers, options, IPs statiques, noms de réseaux**.

## ⚙️ Configurer le réseau par défaut dans Docker Compose

### 🧐 Rappel

* Par défaut, Compose crée un **réseau unique** pour toute votre application (souvent nommé `<nom_projet>_default`).
* Tous les services définis dans votre `compose.yaml` rejoignent automatiquement ce réseau, sauf si vous spécifiez d’autres réseaux personnalisés.

***

### 🔧 Personnalisation du réseau par défaut

Au lieu de définir uniquement vos propres réseaux personnalisés, vous pouvez aussi **modifier les paramètres du réseau par défaut**.

👉 Pour cela, ajoutez une entrée **`default`** sous la section **top-level `networks`**.

***

### 📑 Exemple

```yaml
services:
  web:
    build: .
    ports:
      - "8000:8000"

  db:
    image: postgres

networks:
  default:
    # Utiliser un driver réseau personnalisé
    driver: custom-driver-1
```

***

#### ▶️ Explication

* Les services **`web`** et **`db`** rejoignent automatiquement le réseau **par défaut**.
* Dans cet exemple, ce réseau utilise un **driver personnalisé** (`custom-driver-1`) au lieu du driver **bridge** standard.

***

✅ En résumé :

* Tous les services rejoignent le réseau **`default`** sauf si vous les rattachez à d’autres réseaux.
* Vous pouvez **personnaliser** ce réseau par défaut (driver, options, etc.) via :

```yaml
networks:
  default:
    driver: ...
    driver_opts:
      ...
```

## 🌉 Utiliser un réseau existant avec Docker Compose

### 🧐 Cas d’usage

Il est parfois utile de connecter vos services Compose à un **réseau Docker déjà créé manuellement**, par exemple avec :

```bash
docker network create my-pre-existing-network
```

👉 Cela permet de partager un réseau entre plusieurs projets Compose, ou d’intégrer Compose dans une infrastructure Docker déjà en place.

***

### 🔧 Comment faire ?

Pour rattacher vos services à un réseau externe, vous devez :

1. Déclarer le réseau sous la section **`networks`** de votre `compose.yaml`.
2. Indiquer qu’il est **externe** avec l’option `external: true`.

***

### 📑 Exemple

```yaml
services:
  # ...
networks:
  network1:
    name: my-pre-existing-network
    external: true
```

***

#### ▶️ Explication

* Normalement, Compose crée automatiquement un réseau par défaut nommé **`[nom_projet]_default`**.
* Ici, au lieu de créer ce réseau, Compose cherche un réseau déjà existant appelé **`my-pre-existing-network`**.
* Les conteneurs de votre application sont alors directement connectés à ce réseau externe.

***

### 📚 Références associées

Pour plus de détails sur la configuration réseau dans Compose, consultez :

* 🔗 Élément top-level `networks`
* 🔗 Attribut service-level `networks`

***

✅ En résumé :

* `external: true` permet d’utiliser un réseau Docker déjà existant.
* Idéal pour partager un réseau entre plusieurs applications ou intégrer Compose dans une infra existante.
