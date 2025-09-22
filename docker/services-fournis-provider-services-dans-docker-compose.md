# ⚙️ Services Fournis (Provider Services) dans Docker Compose

### 🔎 Définition

Un **provider service** est un service dont le **cycle de vie n’est pas géré par Compose lui-même**, mais par un composant tiers (ex. une plateforme cloud, un orchestrateur externe, ou une stack déjà déployée).

👉 Cela permet d’**intégrer des services externes** dans ton fichier `compose.yaml` sans que Compose essaie de les construire, lancer ou arrêter directement.

***

### ✅ Objectif

* Référencer un service **existant** (base de données, broker de messages, cache, etc.) sans que Compose ait besoin de le gérer.
* Éviter d’avoir à dupliquer manuellement la configuration réseau et de dépendances.
* **Faciliter l’intégration multiplateforme** : Compose peut “consommer” un service externe comme s’il faisait partie du projet.

***

### 📌 Exemple simple

Imaginons que ton application ait besoin d’un **PostgreSQL** qui est déjà déployé dans une autre infrastructure (ex. un cloud provider).

```yaml
services:
  app:
    image: myapp:latest
    depends_on:
      - db
    environment:
      DATABASE_URL: postgres://user:pass@db:5432/mydb

  db:
    provider: true
```

#### Explication

* `db` est marqué comme un **provider service** (`provider: true`)
* Compose **ne tente pas de créer** un conteneur `db`.
* Mais le service `app` peut quand même déclarer `depends_on: db` et utiliser `db` comme endpoint.

***

### 🔄 Cas d’usage typiques

* Bases de données managées (**AWS RDS, GCP Cloud SQL, Azure Database**).
* Services partagés entre plusieurs projets (ex. un cluster Kafka déjà existant).
* Intégration avec des stacks externes (par ex. Swarm, Kubernetes, ou une autre instance Compose déjà en cours).

***

### ⚠️ Points à retenir

* `provider` remplace la définition classique (`image`, `build`, etc.).
* Tu ne peux pas lancer ou arrêter le service via `docker compose up` ou `down`.
* Il sert uniquement de **référence logique** dans ton graphe de dépendances et pour la configuration réseau.

## 🔹 Qu’est-ce que les _Provider Services_ dans Docker Compose ?

### 📌 Définition

Les _provider services_ sont un **type particulier de service** dans Docker Compose.\
👉 Contrairement aux services classiques, ils **ne représentent pas un conteneur**, mais une **capacité offerte par la plateforme** (Docker Desktop, Swarm, Kubernetes, etc.).

Cela veut dire que Compose ne lance pas un conteneur, mais demande à la plateforme de **fournir et configurer une ressource externe** pour que vos services puissent l’utiliser.

***

### ✅ Objectif

* Déclarer des **dépendances vers des fonctionnalités externes** (base de données managée, GPU, stockage, etc.).
* Intégrer des services dont le **cycle de vie est géré par la plateforme** et non par Compose.
* Garder un fichier `compose.yaml` **déclaratif et portable**.

***

### ⚙️ Fonctionnement

1. Vous déclarez un _provider service_ dans votre `compose.yaml`.
2. Docker Compose **ne crée pas de conteneur** pour lui.
3. Il demande à la **plateforme** de provisionner cette ressource.
4. Vos autres services peuvent alors s’y connecter (réseau, variables d’environnement, dépendances).

***

### 📖 Cas d’usage typiques

* **Base de données managée** (Postgres, MySQL, Redis, etc.).
* **File de messages** (Kafka, RabbitMQ) déjà fournie par la plateforme.
* **Accès GPU** ou stockage persistant.
* **Services cloud natifs** (gestion des secrets, monitoring, logs).

***

### 📝 Exemple dans un `compose.yaml`

```yaml
services:
  app:
    image: mon-app:latest
    depends_on:
      - database

  database:
    x-provider: true   # ← indique un provider service
```

Ici :

* `app` est un service classique (un conteneur).
* `database` est un _provider service_ → Docker Compose ne lance pas Postgres lui-même, mais suppose que la base est fournie par la plateforme.

## 🔹 Utiliser les _Provider Services_ dans Docker Compose

Pour utiliser un service _provider_ dans ton fichier `compose.yaml`, tu dois suivre **4 étapes principales** :

1. **Définir un service** avec l’attribut `provider`.
2. **Spécifier le type** de provider (par ex. un cloud ou un service externe).
3. **Configurer les options spécifiques** au provider.
4. **Déclarer les dépendances** de tes services applicatifs vers ce provider.

***

### 📖 Exemple basique

```yaml
services:
  database:
    provider:
      type: awesomecloud         # type de provider (ex. un cloud externe)
      options:
        type: mysql              # type de service fourni (ici MySQL)
        foo: bar                 # options spécifiques au provider

  app:
    image: myapp
    depends_on:
      - database                 # l’app dépend de "database"
```

***

### 🔎 Explication pas à pas

* **`provider`**\
  → Indique que ce service n’est pas un conteneur Docker classique, mais une **ressource fournie par la plateforme** (ex. une base de données managée).
* **`type: awesomecloud`**\
  → Spécifie le **fournisseur** (ici un exemple fictif `awesomecloud`).
* **`options`**\
  → Ce sont des paramètres propres au provider choisi. Dans cet exemple :
  * `type: mysql` → base de données MySQL
  * `foo: bar` → une option personnalisée
* **`depends_on`**\
  → L’application `app` déclare qu’elle **dépend** du provider `database`.\
  → Cela permet à Compose de s’assurer que la ressource `database` est disponible **avant de lancer `app`**, et d’injecter les infos nécessaires (connexion, credentials, etc.).

***

### ✅ En résumé

* Les _provider services_ remplacent les conteneurs par des **ressources externes gérées par une plateforme**.
* Ils sont utiles pour intégrer des services **cloud managés** (BDD, cache, file de messages, etc.).
* L’attribut clé est `provider`, qui décrit le type de provider et ses options.
* Les autres services peuvent déclarer une dépendance avec `depends_on`.

## 🔹 Comment fonctionnent les _provider services_

Quand tu exécutes `docker compose up` :

1. **Compose détecte** les services qui utilisent un `provider`.
2. Il **demande au provider** (par ex. un cloud, une base de données managée, etc.) de provisionner la ressource.
3. Le provider renvoie à Docker Compose les **informations de connexion** à cette ressource (URL, jeton d’authentification, etc.).
4. **Compose injecte automatiquement** ces infos dans les services dépendants (via des variables d’environnement).

***

### 📖 Convention de nommage des variables

Les variables injectées suivent la règle :

```
<PROVIDER_SERVICE_NAME>_<VARIABLE_NAME>
```

👉 Exemple : si ton service provider s’appelle **`database`**, Compose peut créer automatiquement :

* `DATABASE_URL` → l’URL de connexion à la base de données
* `DATABASE_TOKEN` → un jeton d’authentification
* D’autres variables propres au provider (mot de passe, région cloud, etc.)

***

### 📌 Exemple concret

```yaml
services:
  database:
    provider:
      type: awesomecloud
      options:
        type: postgres

  app:
    image: myapp
    depends_on:
      - database
```

➡️ Quand tu fais :

```bash
docker compose up
```

Ton service `app` recevra automatiquement dans son environnement :

```bash
DATABASE_URL=postgres://user:password@host:5432/dbname
DATABASE_TOKEN=abc123xyz
```

Ainsi, ton application n’a **rien besoin de configurer manuellement** : elle peut directement lire ces variables (`process.env.DATABASE_URL` en Node.js par ex.) et se connecter à la base fournie par le provider.

## 🔹 Types de _providers_ dans Docker Compose

Le champ **`type`** d’un service provider indique le **nom du provider** à utiliser.

👉 Concrètement, Docker Compose va chercher :

1. **Un plugin Docker CLI** qui commence par `docker-<type>`
   * Exemple : `docker-model` si `type: model`
2. **Ou bien un binaire dans le PATH** avec le nom donné
   * Exemple : `model`

***

### 📌 Exemple

```yaml
services:
  ai-runner:
    provider:
      type: model   # Cherche un plugin docker-model ou un binaire "model"
      options:
        model: ai/example-model
```

Dans cet exemple :

* `type: model` signifie que Compose va chercher un outil appelé **`docker-model`** ou **`model`**.
* Les **options** (`model: ai/example-model`) sont envoyées à ce provider pour qu’il comprenne **ce qu’il doit provisionner**.

***

### 🚀 Ce que fait le provider

Le plugin ou binaire est responsable de :

1. **Lire les options** fournies dans le `provider`.
2. **Créer la ressource demandée** (ex. une base de données, un service cloud, un modèle IA).
3. **Retourner les informations nécessaires** (URL, credentials, tokens, etc.).

Ces infos sont ensuite **injectées automatiquement** dans les services dépendants sous forme de **variables d’environnement** (comme `DATABASE_URL`, `AI_RUNNER_HOST`, etc.).

***

💡 **Astuce** :\
Si tu travailles spécifiquement avec des **modèles IA**, Docker recommande d’utiliser l’élément **`models`** au niveau **top-level** du fichier Compose (au lieu de passer par un provider générique).

***

## 🔹 Avantages d’utiliser les _provider services_ dans Docker Compose

L’utilisation des **services providers** dans vos applications Compose apporte plusieurs bénéfices :

#### 1. 🎯 Configuration simplifiée

👉 Plus besoin de configurer et gérer manuellement certaines fonctionnalités de la plateforme (par ex. bases de données managées, modèles IA, services externes).\
Docker Compose s’occupe d’orchestrer tout ça via le provider.

***

#### 2. 📜 Approche déclarative

👉 Tu peux **déclarer toutes les dépendances** de ton application (containers + services externes/provisionnés) **au même endroit**, dans ton `compose.yaml`.\
➡ Ça rend la config plus lisible et maintenable.

***

#### 3. 🔄 Workflow cohérent

👉 Tu continues à utiliser les **mêmes commandes Docker Compose** (`up`, `down`, `logs`, etc.) pour gérer :

* Tes **containers locaux**
* Tes **services providers** (ex. cloud, IA, DB managée, etc.)

➡ Cela évite de devoir mélanger plusieurs outils ou scripts.

***

💡 En résumé : **les providers te permettent de gérer des ressources externes comme si elles faisaient partie de ton stack Compose.**

## 🔹 Créer ton propre _provider_ dans Docker Compose

Si tu veux **étendre Docker Compose** avec des capacités personnalisées, tu peux créer **ton propre provider**.

#### ⚙️ Comment ça fonctionne ?

* Les providers sont gérés via le **mécanisme d’extensions de Compose**.
* Tu peux développer un **plugin Compose** qui enregistre de nouveaux types de providers.
* Ainsi, tu ajoutes des fonctionnalités que Compose n’a pas nativement (exemple : un provider interne pour ton entreprise, un service cloud spécifique, un moteur IA maison, etc.).

***

#### 📌 Étapes principales

1. **Développer un plugin Compose**
   * Le plugin peut être un **Docker CLI plugin** (`docker-myplugin`)
   * ou un **binaire** disponible dans ton `$PATH`.
2. **Enregistrer ton provider**
   * Tu définis un **type de provider** (ex. `type: mycloud`) que Compose pourra reconnaître.
3. **Implémenter la logique**
   * Le provider doit :
     * lire les `options` définies dans le `compose.yaml`,
     * provisionner la ressource (DB, IA, service externe, etc.),
     * retourner à Compose les infos nécessaires (URL, tokens, credentials).
4. **Compose injecte les infos**
   * Ces infos deviennent des **variables d’environnement** accessibles dans tes services dépendants.

***

#### 📚 Documentation

Pour les détails complets : consulte la **documentation officielle Compose Extensions**, qui explique comment :

* développer un plugin,
* enregistrer de nouveaux providers,
* et les utiliser dans un projet Docker Compose.
