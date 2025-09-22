# 🔧 Définir, utiliser et gérer des variables dans un fichier Compose avec interpolation

Un fichier **Compose** peut utiliser des **variables** pour plus de flexibilité.\
👉 Cela permet par exemple :

* de **changer rapidement un tag d’image** afin de tester plusieurs versions 🐳,
* ou d’**ajuster la source d’un volume** selon votre environnement local,\
  ➡️ sans avoir à **modifier le fichier Compose à chaque fois**.

***

### 🔄 L’interpolation

L’**interpolation** permet d’**insérer des valeurs dans votre fichier Compose au moment de l’exécution**.\
Ces valeurs peuvent ensuite être transmises dans l’environnement du conteneur.

***

### 📝 Exemple simple

#### 📂 Fichier `.env`

```bash
cat .env
TAG=v1.5
```

#### 📂 Fichier `compose.yaml`

```yaml
services:
  web:
    image: "webapp:${TAG}"
```

***

### ▶️ Résultat

Lorsque vous exécutez :

```bash
docker compose up
```

👉 Le service **`web`** défini dans le fichier Compose interpolera la variable, ce qui donnera :

```
webapp:v1.5
```

➡️ Cette valeur vient directement du fichier **`.env`**.

***

### 🔍 Vérification avec `config`

Vous pouvez confirmer la valeur interpolée avec la commande suivante :

```bash
docker compose config
```

#### Résultat :

```yaml
services:
  web:
    image: 'webapp:v1.5'
```

***

✅ Grâce à l’interpolation, vous rendez vos fichiers `compose.yaml` **dynamiques, réutilisables et adaptés** à différents environnements, sans duplication inutile.

## ✨ Syntaxe d’interpolation dans Docker Compose

L’**interpolation** s’applique aux valeurs **non citées** (non entourées de guillemets) et **aux valeurs entre guillemets doubles**.

👉 Les deux formes sont supportées :

* **Avec accolades** : `${VAR}`
* **Sans accolades** : `$VAR`

***

### 📝 Formats supportés avec accolades

#### 🔹 1. Substitution directe

```yaml
${VAR}
```

➡️ remplacé par la valeur de **`VAR`**.

***

#### 🔹 2. Valeur par défaut

* Avec contrôle de valeur **non vide** :

```yaml
${VAR:-default}
```

➡️ prend la valeur de **`VAR`** si elle est définie **et non vide**, sinon utilise `default`.

* Sans contrôle de vide :

```yaml
${VAR-default}
```

➡️ prend la valeur de **`VAR`** si elle est définie (même vide), sinon utilise `default`.

***

#### 🔹 3. Valeur requise

* Avec contrôle de valeur **non vide** :

```yaml
${VAR:?error}
```

➡️ prend la valeur de **`VAR`** si elle est définie **et non vide**, sinon Compose quitte avec le message `error`.

* Sans contrôle de vide :

```yaml
${VAR?error}
```

➡️ prend la valeur de **`VAR`** si elle est définie (même vide), sinon Compose quitte avec le message `error`.

***

#### 🔹 4. Valeur alternative

* Avec contrôle de valeur **non vide** :

```yaml
${VAR:+replacement}
```

➡️ utilise `replacement` si **`VAR`** est défini **et non vide**, sinon une chaîne vide.

* Sans contrôle de vide :

```yaml
${VAR+replacement}
```

➡️ utilise `replacement` si **`VAR`** est défini (même vide), sinon une chaîne vide.

***

### 📚 Pour aller plus loin

👉 Voir la documentation officielle : **Interpolation dans la spécification Compose**

***

✅ Grâce à cette syntaxe d’interpolation, vos fichiers `compose.yaml` deviennent **dynamiques et flexibles**, tout en gérant :

* des **valeurs par défaut**,
* des **erreurs obligatoires**,
* des **remplacements conditionnels**.

## 🔄 Façons de définir des variables avec interpolation

**Docker Compose** peut interpoler (substituer) des variables dans votre fichier `compose.yaml` depuis **plusieurs sources**.

***

### ⚖️ Règles de priorité

Lorsque la **même variable** est déclarée dans plusieurs sources, un ordre de priorité s’applique (du plus fort au plus faible) :

1. **Variables de votre shell** 🖥️\
   ➝ définies dans l’environnement actuel du système (ex. `export VAR=valeur`).
2. **Fichier `.env` du répertoire de travail courant (PWD)** 📂\
   ➝ utilisé uniquement si **`--env-file` n’est pas défini**.
3. **Fichier défini par `--env-file` ou `.env` dans le répertoire du projet** 🗂️

***

### 🛠️ Vérification

Vous pouvez afficher les **variables et leurs valeurs effectives** utilisées par Compose pour interpoler votre modèle en exécutant :

```bash
docker compose config --environment
```

👉 Cette commande vous montre exactement **quelles valeurs sont injectées** dans votre configuration résolue.

***

✅ Grâce à ces règles, vous pouvez contrôler et surcharger vos variables facilement selon l’environnement (local, test, CI/CD, production).

## 📂 Le fichier `.env` dans Docker Compose

Un fichier **`.env`** est un simple fichier texte utilisé pour définir des **variables** qui seront disponibles pour l’**interpolation** lors de l’exécution de :

```bash
docker compose up
```

👉 Ce fichier contient généralement des **paires clé-valeur** (`VARIABLE=valeur`) et permet de **centraliser et gérer la configuration** en un seul endroit.

***

### 🎯 Utilité

* Le fichier `.env` est la **méthode par défaut** pour définir des variables.
* Il doit être placé à la **racine du projet**, à côté de votre fichier **`compose.yaml`**.
* Très pratique lorsque vous avez **plusieurs variables** à gérer.

📚 Pour plus de détails sur le format, voir la section : **Syntax for environment files**.

***

### 📝 Exemple simple

#### 📂 Fichier `.env`

```bash
# définit COMPOSE_DEBUG basé sur DEV_MODE, avec une valeur par défaut à "false"
COMPOSE_DEBUG=${DEV_MODE:-false}
```

#### 📂 Fichier `compose.yaml`

```yaml
services:
  webapp:
    image: my-webapp-image
    environment:
      - DEBUG=${COMPOSE_DEBUG}
```

***

### ▶️ Résultat en pratique

Si vous exécutez :

```bash
DEV_MODE=true docker compose config
```

👉 Le rendu final sera :

```yaml
services:
  webapp:
    environment:
      DEBUG: "true"
```

***

✅ Grâce au fichier `.env`, vous pouvez facilement gérer vos variables, définir des **valeurs par défaut**, et rendre vos fichiers Compose **dynamiques et réutilisables** selon vos environnements (développement, test, production).

## ℹ️ Informations supplémentaires sur le fichier `.env`

### 🔗 Référencement des variables dans `compose.yaml`

Si vous définissez une variable dans votre **fichier `.env`**, vous pouvez la référencer directement dans votre fichier `compose.yaml` via l’attribut **`environment`**.

#### 📝 Exemple

📂 `.env`

```bash
DEBUG=1
```

📂 `compose.yaml`

```yaml
services:
  webapp:
    image: my-webapp-image
    environment:
      - DEBUG=${DEBUG}
```

👉 Docker Compose remplacera `${DEBUG}` par la valeur trouvée dans `.env`, soit ici **`1`**.

***

### ⚠️ Important

* Soyez attentif à la **priorité des variables d’environnement** (Environment variables precedence) lorsque vous utilisez des variables `.env` qui se retrouvent aussi comme variables d’environnement de vos conteneurs.
*   Vous pouvez placer votre fichier `.env` **ailleurs que dans la racine du projet**, et préciser son chemin avec l’option CLI :

    ```bash
    docker compose --env-file chemin/vers/env up
    ```
* Votre fichier `.env` peut être **écrasé** par un autre fichier `.env` s’il est remplacé via `--env-file`.

***

⚠️ **Remarque importante**\
L’interpolation à partir des fichiers `.env` est une **fonctionnalité du CLI Docker Compose**.\
👉 Elle n’est **pas supportée par Swarm** lorsque vous exécutez :

```bash
docker stack deploy
```

***

## 📑 Syntaxe des fichiers `.env`

Les règles suivantes s’appliquent aux fichiers `.env` :

***

#### 1️⃣ Commentaires et lignes vides

* Les lignes commençant par `#` sont des **commentaires** et sont ignorées.
* Les lignes vides sont également ignorées.

***

#### 2️⃣ Interpolation et valeurs

* Les valeurs **non citées** ou **entre guillemets doubles (`"`)** sont interpolées.
* Chaque ligne représente une **paire clé-valeur** (`VAR=VAL`).
* Les valeurs peuvent être :
  * non citées,
  * entre guillemets doubles `"..."`,
  * entre guillemets simples `'...'`.

***

#### 3️⃣ Exemples simples

```bash
VAR=VAL       # -> VAL
VAR="VAL"     # -> VAL
VAR='VAL'     # -> VAL
```

***

#### 4️⃣ Commentaires en ligne

* Pour les **valeurs non citées**, les commentaires doivent être précédés d’un espace :

```bash
VAR=VAL # commentaire   -> VAL
VAR=VAL#pas un commentaire -> VAL#pas un commentaire
```

* Pour les **valeurs citées**, les commentaires doivent venir **après la fermeture des guillemets** :

```bash
VAR="VAL # pas un commentaire" -> VAL # pas un commentaire
VAR="VAL" # commentaire        -> VAL
```

***

#### 5️⃣ Variables entre guillemets simples `'...'`

* Les valeurs entre guillemets simples sont **prises littéralement** (pas d’interpolation).

```bash
VAR='$OTHER'      -> $OTHER
VAR='${OTHER}'    -> ${OTHER}
```

***

#### 6️⃣ Échappement des guillemets et séquences spéciales

* Vous pouvez échapper les guillemets avec `\` :

```bash
VAR='Let\'s go!' -> Let's go!
VAR="{\"hello\": \"json\"}" -> {"hello": "json"}
```

* Les **séquences d’échappement shell courantes** sont supportées dans les guillemets doubles :

```bash
VAR="some\tvalue" -> some    value
VAR='some\tvalue' -> some\tvalue
VAR=some\tvalue   -> some\tvalue
```

***

#### 7️⃣ Valeurs multi-lignes avec guillemets simples

Les valeurs entre guillemets simples peuvent s’étendre sur **plusieurs lignes** :

```bash
KEY='SOME
VALUE'
```

👉 Si vous exécutez ensuite :

```bash
docker compose config
```

Résultat :

```yaml
environment:
  KEY: |-
    SOME
    VALUE
```

***

✅ Grâce à cette syntaxe, vous pouvez rendre vos fichiers `.env` **lisibles, puissants et flexibles**, en gérant aussi bien des valeurs simples que complexes (multi-lignes, JSON, échappements).

## 📂 Substitution avec `--env-file`

Vous pouvez définir des **valeurs par défaut** pour plusieurs variables d’environnement dans un fichier **`.env`**, puis passer ce fichier comme **argument dans la CLI**.

👉 L’avantage de cette méthode est que vous pouvez **stocker ce fichier n’importe où** et le **nommer selon le contexte** (par ex. `config/.env.dev`, `config/.env.prod`).

Le chemin du fichier est relatif au répertoire courant (**PWD**) où la commande Docker Compose est exécutée.\
➡️ On utilise l’option **`--env-file`** :

```bash
docker compose --env-file ./config/.env.dev up
```

***

### ℹ️ Informations supplémentaires

Cette méthode est particulièrement utile si vous voulez **remplacer temporairement** un fichier `.env` déjà référencé dans votre `compose.yaml`.\
👉 Exemple courant : avoir différents fichiers `.env` pour **la production** (`.env.prod`) et pour **les tests** (`.env.test`).

***

### 📝 Exemple pratique

📂 `.env`

```bash
TAG=v1.5
```

📂 `./config/.env.dev`

```bash
TAG=v1.6
```

📂 `compose.yaml`

```yaml
services:
  web:
    image: "webapp:${TAG}"
```

***

#### ▶️ Sans option `--env-file`

```bash
docker compose config
```

Résultat :

```yaml
services:
  web:
    image: 'webapp:v1.5'
```

👉 Docker Compose charge par défaut le fichier **`.env`**.

***

#### ▶️ Avec option `--env-file`

```bash
docker compose --env-file ./config/.env.dev config
```

Résultat :

```yaml
services:
  web:
    image: 'webapp:v1.6'
```

👉 Ici, le fichier `.env.dev` **remplace le `.env` par défaut**.

***

#### ⚠️ En cas de chemin invalide

```bash
docker compose --env-file ./doesnotexist/.env.dev config
```

Résultat :

```
ERROR: Couldn't find env file: /home/user/./doesnotexist/.env.dev
```

👉 Compose renvoie une erreur si le fichier n’existe pas.

***

### 📑 Plusieurs fichiers `--env-file`

Vous pouvez passer **plusieurs fichiers** en utilisant plusieurs options `--env-file`.\
➡️ Docker Compose les lit **dans l’ordre**, et les fichiers suivants **peuvent écraser** les variables des fichiers précédents.

```bash
docker compose --env-file .env --env-file .env.override up
```

***

### 🎯 Surcharger des variables directement depuis la CLI

Vous pouvez aussi écraser des variables **spécifiques** en ligne de commande lors du démarrage :

```bash
docker compose --env-file .env.dev up -e DATABASE_URL=mysql://new_user:new_password@new_db:3306/new_database
```

👉 Ici, la variable `DATABASE_URL` surcharge celle définie dans le `.env.dev`.

***

✅ En résumé :

* `--env-file` permet de charger un ou plusieurs fichiers `.env` placés où vous voulez,
* l’ordre de lecture permet de **surcharger certaines valeurs**,
* et vous pouvez **encore écraser certaines variables à la volée** avec `-e`.

## 📂 Fichier `.env` local vs fichier `.env` du répertoire projet

Un fichier **`.env`** peut aussi être utilisé pour déclarer des **variables d’environnement prédéfinies** :

* pour **contrôler le comportement de Compose** ⚙️,
* et pour définir les **fichiers à charger** 📑.

***

### 🔎 Comportement par défaut

* Lorsque vous exécutez Docker Compose **sans option explicite `--env-file`** :
  1. Compose recherche un fichier **`.env`** dans votre répertoire de travail (**PWD**).
  2. Les valeurs de ce fichier sont chargées :
     * pour la **configuration interne de Compose**,
     * et pour l’**interpolation des variables**.
* Si ce fichier **définit la variable prédéfinie `COMPOSE_FILE`**, et que cela **change le répertoire projet** (par ex. en pointant vers un autre dossier),\
  👉 Compose cherchera alors un **deuxième fichier `.env`** dans ce **répertoire projet**.\
  ➡️ Ce deuxième fichier a une **priorité plus faible** que le `.env` initial du PWD.

***

### 🎯 Intérêt

Ce mécanisme permet de **réutiliser un projet Compose existant** avec un ensemble personnalisé de variables (comme des overrides)\
👉 sans avoir à passer manuellement toutes les variables via la ligne de commande.

***

### 📝 Exemple pratique

📂 `.env` (dans le répertoire courant **PWD**)

```bash
COMPOSE_FILE=../compose.yaml
POSTGRES_VERSION=9.3
```

📂 `../compose.yaml`

```yaml
services:
  db:
    image: "postgres:${POSTGRES_VERSION}"
```

📂 `../.env` (dans le répertoire projet)

```bash
POSTGRES_VERSION=9.2
```

***

#### ▶️ Exécution

```bash
docker compose config
```

#### Résultat

```yaml
services:
  db:
    image: "postgres:9.3"
```

👉 C’est bien la valeur **du `.env` local (PWD)** (`POSTGRES_VERSION=9.3`) qui l’emporte sur celle du `.env` du répertoire projet (`9.2`).

***

✅ En résumé :

* `.env` dans le **répertoire courant (PWD)** → priorité la plus haute.
* `.env` dans le **répertoire projet** → utilisé seulement si `COMPOSE_FILE` redéfinit la racine du projet, mais avec une priorité plus faible.

## 🖥️ Substitution depuis le shell

Vous pouvez utiliser des **variables d’environnement existantes** de votre machine hôte 🖥️ ou du **shell** dans lequel vous exécutez vos commandes `docker compose`.

👉 Cela permet d’**injecter dynamiquement** des valeurs dans votre configuration Docker Compose **au moment de l’exécution**.

***

### 📝 Exemple

Supposons que votre shell contienne la variable :

```bash
POSTGRES_VERSION=9.3
```

et que votre fichier `compose.yaml` contienne :

```yaml
db:
  image: "postgres:${POSTGRES_VERSION}"
```

***

### ▶️ Résultat

Lorsque vous exécutez :

```bash
docker compose up
```

* Compose cherche la variable **`POSTGRES_VERSION`** dans l’environnement du shell,
* puis substitue sa valeur.

👉 Dans cet exemple, l’image sera résolue en :

```
postgres:9.3
```

avant que la configuration ne soit exécutée.

***

### ⚠️ Si la variable n’est pas définie

* Compose substitue par une **chaîne vide**.
* Exemple avec `POSTGRES_VERSION` non défini :

```yaml
image: "postgres:"
```

➡️ Ce qui donne une référence invalide.

***

### 📝 Remarque importante

`postgres:` **n’est pas une image valide** ❌.

Docker attend :

* soit une référence **sans tag** (`postgres`) → par défaut, Docker prendra l’image **`latest`**,
* soit une référence **avec tag explicite** (`postgres:15`).

***

✅ En résumé :

* Les variables du **shell** peuvent être utilisées directement dans vos fichiers Compose,
* mais il est recommandé de **toujours définir une valeur par défaut** pour éviter d’obtenir des références invalides.
