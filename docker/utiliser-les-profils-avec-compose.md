---
description: >-
  📋 Options de la page  Les profils vous permettent d’adapter votre application
  Compose à différents environnements ou cas d’usage en activant certains
  services sélectivement.
---

# 🗂️ Utiliser les profils avec Compose

### ⚙️ Fonctionnement des profils

* Chaque **service** peut être assigné à **un ou plusieurs profils**.
* 👉 Les services **sans profil** sont démarrés/arrêtés **par défaut**.
* 👉 Les services **avec profil** ne sont démarrés/arrêtés **que lorsque leur profil est activé**.

***

### 🎯 Intérêt

Cette configuration permet d’inclure dans un **seul fichier `compose.yml`** :

* des services spécifiques au **débogage** 🐛,
* des services destinés au **développement** 💻,
* ou encore des services optionnels utilisés dans certains environnements.

➡️ Ils ne seront **activés que si nécessaire**, évitant ainsi de multiplier les fichiers de configuration.

***

✨ Exemple concret à venir : définir un profil pour n’activer un outil de debug **que dans l’environnement de développement**, tout en gardant la même base `compose.yml`.

## 🏷️ Assigner des profils aux services

Les services sont associés à des profils grâce à l’attribut **`profiles`**,\
👉 qui accepte un **tableau de noms de profils**.

***

### 📝 Exemple YAML

```yaml
services:
  frontend:
    image: frontend
    profiles: [frontend]

  phpmyadmin:
    image: phpmyadmin
    depends_on: [db]
    profiles: [debug]

  backend:
    image: backend

  db:
    image: mysql
```

***

### 📖 Explication

* Le service **`frontend`** est associé au profil **`frontend`**.
* Le service **`phpmyadmin`** est associé au profil **`debug`**.\
  👉 Résultat : ces deux services ne seront démarrés **que lorsque leurs profils respectifs sont activés**.

⚡ Les services **sans attribut `profiles`** (ici **`backend`** et **`db`**) sont **toujours activés par défaut**.\
➡️ Dans ce cas, exécuter simplement :

```bash
docker compose up
```

ne démarrera **que `backend` et `db`**.

***

### 🔑 Règles de nommage

Les noms de profils valides suivent ce format **regex** :

```
[a-zA-Z0-9][a-zA-Z0-9_.-]+
```

✅ Cela signifie :

* ils doivent commencer par une lettre ou un chiffre,
* puis peuvent contenir lettres, chiffres, `_` (underscore), `.` (point) ou `-` (tiret).

***

### 💡 Astuce

Les **services principaux** de votre application (ceux nécessaires à son fonctionnement de base) ne devraient **pas être assignés à un profil**.\
➡️ Ainsi, ils seront **toujours activés et démarrés automatiquement**.

## ⚡ Démarrer des profils spécifiques

Pour démarrer un profil particulier, vous pouvez utiliser :

1. L’option **`--profile`** dans la ligne de commande
2. Ou bien la variable d’environnement **`COMPOSE_PROFILES`**

***

### 📝 Exemples

#### ▶️ Avec l’option `--profile`

```bash
docker compose --profile debug up
```

#### ▶️ Avec la variable d’environnement

```bash
COMPOSE_PROFILES=debug docker compose up
```

***

### 📖 Explication

Dans les deux cas, le profil **`debug`** est activé ✅.

👉 En reprenant l’exemple du fichier `compose.yaml` précédent :

* Les services démarrés seront :
  * `db` 🗄️
  * `backend` ⚙️
  * `phpmyadmin` 🐘

➡️ Car `phpmyadmin` appartient au profil `debug`, et `db` + `backend` n’ont pas de profil (donc toujours activés par défaut).

## 🗂️ Démarrer plusieurs profils

Il est également possible d’**activer plusieurs profils en même temps**.

Par exemple :

```bash
docker compose --profile frontend --profile debug up
```

➡️ Ici, les profils **`frontend`** et **`debug`** seront activés simultanément ✅.

***

### 📝 Méthodes possibles

#### ▶️ Avec plusieurs options `--profile`

```bash
docker compose --profile frontend --profile debug up
```

#### ▶️ Avec la variable d’environnement (liste séparée par des virgules)

```bash
COMPOSE_PROFILES=frontend,debug docker compose up
```

***

### 🔄 Activer **tous les profils**

Si vous souhaitez activer **tous les profils définis** dans votre fichier `compose.yaml`, vous pouvez exécuter :

```bash
docker compose --profile "*"
```

***

👉 Cela vous donne une grande flexibilité pour adapter rapidement votre stack Compose à différents contextes (dev, debug, test, staging, production, etc.) sans multiplier les fichiers de configuration.

## 🔄 Démarrage automatique des profils et résolution des dépendances

Lorsque vous ciblez explicitement un **service** en ligne de commande et que ce service est associé à un ou plusieurs **profils**,\
👉 vous n’avez **pas besoin d’activer manuellement le profil** : **Compose exécute ce service quoi qu’il arrive** ✅.

C’est très pratique pour lancer des **services ponctuels** ou des **outils de débogage** 🛠️.

***

### ⚙️ Règle de fonctionnement

* Seul le **service ciblé** (et ses dépendances déclarées via `depends_on`) est démarré.
* Les autres services du même profil ne seront **pas démarrés** sauf si :
  1. Ils sont eux aussi explicitement ciblés 🎯,
  2. Ou bien si le profil est activé manuellement avec `--profile` ou `COMPOSE_PROFILES`.

***

### 📖 Exemple

```yaml
services:
  backend:
    image: backend

  db:
    image: mysql

  db-migrations:
    image: backend
    command: myapp migrate
    depends_on:
      - db
    profiles:
      - tools
```

***

#### ▶️ Cas 1 : démarrage classique

```bash
docker compose up -d
```

👉 Ici, seuls `backend` et `db` démarrent (aucun profil n’est activé).

***

#### ▶️ Cas 2 : exécution ciblée d’un service avec profil

```bash
docker compose run db-migrations
```

👉 Le service `db-migrations` démarre même s’il appartient au profil **`tools`**.\
➡️ Pourquoi ? Parce qu’il est **explicitement ciblé**.

👉 De plus, comme il dépend de `db`, ce dernier est également démarré automatiquement 🔄.

***

### ⚠️ Attention aux dépendances profilées

Si un service ciblé a des dépendances qui sont elles aussi derrière un profil, vous devez vous assurer que ces dépendances soient :

1. Dans le **même profil** 🗂️,
2. Ou **démarrées séparément** manuellement,
3. Ou **sans profil assigné**, afin qu’elles soient toujours activées par défaut.

***

✅ En résumé : cibler un service le rend exécutable directement, peu importe son profil. Mais les dépendances profilées demandent une attention particulière.

## ⏹️ Arrêter une application et des services avec des profils spécifiques

De la même manière que pour le démarrage de profils spécifiques, vous pouvez utiliser :

1. L’option **`--profile`** en ligne de commande
2. Ou bien la variable d’environnement **`COMPOSE_PROFILES`**

***

### 📝 Exemples

#### ▶️ Avec l’option `--profile`

```bash
docker compose --profile debug down
```

#### ▶️ Avec la variable d’environnement

```bash
COMPOSE_PROFILES=debug docker compose down
```

***

### 📖 Explication

Ces deux commandes vont :

* **arrêter** et **supprimer** les services du profil `debug`,
* ainsi que les services **sans profil**.

👉 En reprenant l’exemple `compose.yaml` suivant :

```yaml
services:
  frontend:
    image: frontend
    profiles: [frontend]

  phpmyadmin:
    image: phpmyadmin
    depends_on: [db]
    profiles: [debug]

  backend:
    image: backend

  db:
    image: mysql
```

➡️ L’exécution des commandes ci-dessus arrêtera les services :

* `db` 🗄️
* `backend` ⚙️
* `phpmyadmin` 🐘

⚡ Par contre, `frontend` ne sera pas concerné puisqu’il dépend d’un autre profil (`frontend`).

***

### 🎯 Arrêter un seul service

Si vous souhaitez uniquement arrêter **un service précis** (par exemple `phpmyadmin`), vous pouvez utiliser :

```bash
docker compose down phpmyadmin
```

ou bien :

```bash
docker compose stop phpmyadmin
```

***

### ⚠️ Note importante

Lancer simplement :

```bash
docker compose down
```

➡️ n’arrête **que les services sans profil** (`backend` et `db` dans cet exemple).

***

### 📚 Informations de référence

* **profiles** : permettent d’activer ou de désactiver certains services selon le contexte.
