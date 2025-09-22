# 🔀 Fusionner plusieurs fichiers Compose

### 🛠️ Principe

Docker Compose permet de **fusionner** et **surcharger** plusieurs fichiers Compose afin de créer un **fichier composite** 📑.

***

### 📌 Comportement par défaut

Par défaut, Compose lit automatiquement **deux fichiers** :

* **`compose.yaml`** → contient la **configuration de base** de votre application.
* **`compose.override.yaml`** (optionnel) → contient les **surcharges** ou des services supplémentaires.

👉 Par convention :

* le fichier **principal** = configuration standard (commune à tous les environnements),
* le fichier **override** = ajustements spécifiques (par ex. pour le développement).

***

### ⚡ Exemple concret

* Vous définissez un service **`web`** dans `compose.yaml`.
* Vous ajoutez un **volume** ou changez la commande du conteneur dans `compose.override.yaml`.
* Quand vous exécutez :

```bash
docker compose up
```

Compose fusionne les deux fichiers et applique les règles de surcharge.

***

### 📑 Règle de fusion

Si un même service est défini dans **les deux fichiers** (`compose.yaml` et `compose.override.yaml`) :

* Compose **fusionne les configurations** en suivant les règles décrites dans :
  * la documentation officielle **Compose Specification**.

👉 En résumé :

* Les définitions de `compose.override.yaml` **complètent ou remplacent** celles de `compose.yaml`.

***

✅ En résumé :

* `compose.yaml` = base 🏗️
* `compose.override.yaml` = modifications ✏️
* Les deux sont automatiquement fusionnés → résultat final utilisé par Compose.

## 🌀 Comment fusionner plusieurs fichiers Compose

### 📌 Méthodes possibles

Pour utiliser plusieurs fichiers d’override, ou un fichier d’override avec un autre nom :

1. Vous pouvez utiliser la variable d’environnement **`COMPOSE_FILE`** (pré-définie).
2. Ou bien utiliser l’option **`-f`** en ligne de commande afin de spécifier la liste des fichiers à fusionner.

***

### ⚡ Règle de fusion

* Les fichiers sont **fusionnés dans l’ordre où ils sont indiqués** en CLI.
* Chaque fichier suivant peut :
  * **fusionner** avec le précédent,
  * **surcharger** certains champs,
  * ou **ajouter** de nouveaux champs/services.

***

### 🖥️ Exemple pratique

```bash
docker compose -f compose.yaml -f compose.admin.yaml run backup_db
```

👉 Ici :

* **`compose.yaml`** définit la configuration de base.
* **`compose.admin.yaml`** apporte des ajouts ou des overrides.

***

#### 🔹 Exemple `compose.yaml`

```yaml
webapp:
  image: examples/web
  ports:
    - "8000:8000"
  volumes:
    - "/data"
```

#### 🔹 Exemple `compose.admin.yaml`

```yaml
webapp:
  environment:
    - DEBUG=1
```

***

### ✅ Résultat fusionné

```yaml
webapp:
  image: examples/web
  ports:
    - "8000:8000"
  volumes:
    - "/data"
  environment:
    - DEBUG=1
```

👉 Les champs qui existaient déjà (`image`, `ports`, `volumes`) sont conservés.\
👉 Les nouveaux champs (`environment`) sont ajoutés à la configuration.

## 🔀 Règles de fusion des fichiers Compose

Quand vous utilisez plusieurs fichiers **Compose** avec `-f`, Docker Compose applique des règles précises pour fusionner et interpréter les configurations.

***

### 📂 Chemins relatifs

👉 **Tous les chemins doivent être relatifs au fichier de base** (le **premier fichier** spécifié avec `-f`).

* Les fichiers d’override n’ont pas besoin d’être des fichiers Compose complets : ils peuvent ne contenir que des **fragments**.
* Pour éviter toute confusion (par exemple : savoir si `./data` se réfère au chemin du fichier principal ou de l’override), **Compose impose que tout soit relatif au fichier de base**.

⚡ **Astuce** : utilisez `docker compose config` pour afficher la configuration fusionnée et éviter les problèmes liés aux chemins.

***

### 📌 Principe général

* Les options d’un service définies dans le fichier d’override **remplacent** ou **étendent** celles du fichier principal.
* Cela dépend si l’option est une **valeur unique** ou une **valeur multiple/collection**.

***

### 1️⃣ Options à **valeur unique**

Exemples : `image`, `command`, `mem_limit`.\
👉 La valeur de l’override **remplace** celle de la configuration originale.

#### Exemple

**Fichier de base** :

```yaml
services:
  myservice:
    command: python app.py
```

**Fichier override** :

```yaml
services:
  myservice:
    command: python otherapp.py
```

**Résultat fusionné** :

```yaml
services:
  myservice:
    command: python otherapp.py
```

***

### 2️⃣ Options à **valeurs multiples concaténées**

Exemples : `ports`, `expose`, `external_links`, `dns`, `dns_search`, `tmpfs`.\
👉 Les valeurs des deux fichiers sont **concaténées**.

#### Exemple

**Fichier de base** :

```yaml
services:
  myservice:
    expose:
      - "3000"
```

**Fichier override** :

```yaml
services:
  myservice:
    expose:
      - "4000"
      - "5000"
```

**Résultat fusionné** :

```yaml
services:
  myservice:
    expose:
      - "3000"
      - "4000"
      - "5000"
```

***

### 3️⃣ Options à **fusion intelligente**

Exemples : `environment`, `labels`, `volumes`, `devices`.\
👉 Compose fusionne les entrées et applique une **règle de priorité** :

* Pour **`environment`** et **`labels`** → les clés identiques sont remplacées par la valeur de l’override.
* Pour **`volumes`** et **`devices`** → les chemins de montage dans le conteneur (`/path/in/container`) déterminent la priorité.

***

#### 🔹 Exemple avec `environment`

**Fichier de base** :

```yaml
services:
  myservice:
    environment:
      - FOO=original
      - BAR=original
```

**Fichier override** :

```yaml
services:
  myservice:
    environment:
      - BAR=local
      - BAZ=local
```

**Résultat fusionné** :

```yaml
services:
  myservice:
    environment:
      - FOO=original
      - BAR=local
      - BAZ=local
```

👉 La clé `BAR` est remplacée par `local`, tandis que `FOO` est conservé et `BAZ` est ajouté.

***

#### 🔹 Exemple avec `volumes`

**Fichier de base** :

```yaml
services:
  myservice:
    volumes:
      - ./original:/foo
      - ./original:/bar
```

**Fichier override** :

```yaml
services:
  myservice:
    volumes:
      - ./local:/bar
      - ./local:/baz
```

**Résultat fusionné** :

```yaml
services:
  myservice:
    volumes:
      - ./original:/foo
      - ./local:/bar
      - ./local:/baz
```

👉 Le montage `/bar` est remplacé par celui du fichier override.

***

### 📖 Pour aller plus loin

Pour connaître l’ensemble des règles officielles de fusion :\
👉 Merge and override in the Compose Specification.

## 📘 Informations supplémentaires sur l’utilisation de plusieurs fichiers Compose

### ✅ Utilisation par défaut (sans `-f`)

* L’option **`-f`** est **facultative**.
* Si vous ne la fournissez pas, **Compose recherche automatiquement** :
  * un fichier **`compose.yaml`** (obligatoire),
  * et un fichier **`compose.override.yaml`** (optionnel).
* Si les deux fichiers sont trouvés au même niveau de répertoire, **Compose les combine** en une seule configuration.

***

### 📥 Lecture depuis **stdin**

Vous pouvez utiliser `-f -` pour lire un fichier Compose **depuis l’entrée standard (stdin)**.

#### Exemple :

```bash
docker compose -f - <<EOF
webapp:
  image: examples/web
  ports:
    - "8000:8000"
  volumes:
    - "/data"
  environment:
    - DEBUG=1
EOF
```

👉 Ici, les **chemins** utilisés dans la configuration sont **relatifs au répertoire courant** (`PWD`).

***

### 📂 Utiliser un fichier Compose situé ailleurs

Avec `-f`, vous pouvez fournir un chemin vers un fichier `compose.yaml` qui **n’est pas dans le répertoire courant**.

* Cela peut se faire directement en CLI avec `-f`.
* Ou bien en définissant la variable d’environnement **`COMPOSE_FILE`** (dans votre shell ou un fichier `.env`).

#### Exemple :

Imaginons que vous avez un projet **Rails** avec un `compose.yaml` dans `~/sandbox/rails/`.\
Pour télécharger l’image Postgres du service `db` depuis **n’importe quel répertoire** :

```bash
docker compose -f ~/sandbox/rails/compose.yaml pull db
```

Sortie :

```
Pulling db (postgres:latest)...
latest: Pulling from library/postgres
...
Status: Downloaded newer image for postgres:latest
```

***

### 🛠 Exemple pratique : multi-fichiers pour différents environnements

#### 🔹 1. Fichier de base (services communs) → `compose.yaml`

```yaml
services:
  web:
    image: example/my_web_app:latest
    depends_on:
      - db
      - cache

  db:
    image: postgres:latest

  cache:
    image: redis:latest
```

***

#### 🔹 2. Configuration développement → `compose.override.yaml`

```yaml
services:
  web:
    build: .
    volumes:
      - '.:/code'
    ports:
      - 8883:80
    environment:
      DEBUG: 'true'

  db:
    command: '-d'
    ports:
      - 5432:5432

  cache:
    ports:
      - 6379:6379
```

👉 Lors d’un `docker compose up`, Compose lit **automatiquement** ce fichier et surcharge la configuration.

***

#### 🔹 3. Configuration production → `compose.prod.yaml`

```yaml
services:
  web:
    ports:
      - 80:80
    environment:
      PRODUCTION: 'true'

  cache:
    environment:
      TTL: '500'
```

***

#### 🚀 Déploiement en production

Pour lancer avec la configuration **de base + production** (sans le dev override) :

```bash
docker compose -f compose.yaml -f compose.prod.yaml up -d
```

👉 Cela déploie `web`, `db` et `cache` avec :

* la config de **`compose.yaml`**
* la surcharge de **`compose.prod.yaml`**\
  ❌ mais **pas** la config de `compose.override.yaml`.

***

📖 Pour aller plus loin : Utiliser Compose en production

## ⚠️ Limitations de Docker Compose avec plusieurs fichiers

Docker Compose prend en charge les **chemins relatifs** pour de nombreuses ressources incluses dans le modèle d’application, par exemple :

* le **contexte de build** pour les images des services 🏗️,
* l’emplacement du fichier contenant les **variables d’environnement** 🌍,
* le chemin vers un **répertoire local** utilisé dans un volume monté 🔗.

👉 Avec cette contrainte, l’organisation du code dans un **monorepo** peut devenir compliquée :

* Une organisation naturelle consisterait à créer des **dossiers dédiés par équipe ou par composant**.
* Mais dans ce cas, les chemins relatifs définis dans les fichiers Compose risquent de **ne plus correspondre** correctement, rendant leur utilisation complexe.

***

### 📖 Références utiles

* 🔗 Merge rules – pour les règles de fusion entre plusieurs fichiers Compose.
