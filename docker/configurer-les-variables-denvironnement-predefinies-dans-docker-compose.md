# ⚙️ Configurer les variables d’environnement prédéfinies dans Docker Compose

**Docker Compose** inclut plusieurs **variables d’environnement prédéfinies**.\
👉 Il hérite également des variables courantes de la CLI Docker, telles que :

* **`DOCKER_HOST`**
* **`DOCKER_CONTEXT`**

📚 Voir la référence officielle **Docker CLI environment variable reference** pour plus de détails.

***

### 📖 Cette page explique comment définir ou modifier les variables suivantes :

* **`COMPOSE_PROJECT_NAME`** 🏷️ – définit le nom du projet Compose.
* **`COMPOSE_FILE`** 📂 – définit quels fichiers `compose.yaml` utiliser.
* **`COMPOSE_PROFILES`** 🗂️ – définit les profils activés.
* **`COMPOSE_CONVERT_WINDOWS_PATHS`** 💻 – gère la conversion des chemins Windows.
* **`COMPOSE_PATH_SEPARATOR`** ➖ – définit le séparateur de chemins.
* **`COMPOSE_IGNORE_ORPHANS`** 🛑 – ignore les services orphelins non définis dans le fichier Compose.
* **`COMPOSE_REMOVE_ORPHANS`** ❌ – supprime les services orphelins.
* **`COMPOSE_PARALLEL_LIMIT`** ⚡ – limite le nombre d’opérations exécutées en parallèle.
* **`COMPOSE_ANSI`** 🎨 – contrôle la sortie ANSI (couleurs dans les logs).
* **`COMPOSE_STATUS_STDOUT`** 📊 – définit si le statut doit être affiché sur `stdout`.
* **`COMPOSE_ENV_FILES`** 📑 – liste les fichiers d’environnement utilisés.
* **`COMPOSE_DISABLE_ENV_FILE`** 🚫 – désactive complètement l’utilisation des `.env`.
* **`COMPOSE_MENU`** 📜 – active/désactive le menu interactif.
* **`COMPOSE_EXPERIMENTAL`** 🧪 – active les fonctionnalités expérimentales.
* **`COMPOSE_PROGRESS`** ⏳ – configure l’affichage de la progression (progress bar).

***

👉 Ces variables offrent une grande flexibilité pour personnaliser le comportement de **Docker Compose**, aussi bien en développement qu’en production.

## 🔄 Méthodes pour surcharger les variables d’environnement

| **Méthode**        | **Description**                                                               |
| ------------------ | ----------------------------------------------------------------------------- |
| **`.env` file** 📂 | Fichier situé dans le répertoire de travail du projet.                        |
| **Shell** 🖥️      | Variable définie dans le shell du système hôte (ex. `export VAR=valeur`).     |
| **CLI** ⌨️         | Variable transmise au moment de l’exécution avec les options `--env` ou `-e`. |

***

### ⚠️ Attention

Lorsque vous modifiez ou définissez des variables d’environnement, gardez en tête la règle de **priorité des variables d’environnement** (Environment variable precedence).\
👉 Cela détermine **quelle valeur finale** sera utilisée par Docker Compose lorsqu’une variable est définie dans plusieurs endroits.

## ⚙️ Détails de configuration

### 📂 Configuration du projet et des fichiers

#### 🏷️ **COMPOSE\_PROJECT\_NAME**

La variable **`COMPOSE_PROJECT_NAME`** permet de définir le **nom du projet**.\
👉 Cette valeur est **préfixée** au nom du service pour former le nom final du conteneur au démarrage.

***

### 📝 Exemple

Si votre projet s’appelle **`myapp`** et contient deux services :

* `db`
* `web`

alors Compose lancera des conteneurs nommés :

* **`myapp-db-1`** 🗄️
* **`myapp-web-1`** 🌐

***

### 🔄 Ordre de priorité (du plus fort au plus faible)

Docker Compose peut définir le nom du projet de plusieurs manières.\
Voici la hiérarchie appliquée :

1.  🏆 Le flag **`-p`** dans la ligne de commande

    ```bash
    docker compose -p monprojet up
    ```
2. 📌 La variable d’environnement **`COMPOSE_PROJECT_NAME`**
3. 📝 La clé **`name:`** au niveau supérieur du fichier `compose.yaml`\
   ➝ ou le dernier `name:` rencontré si plusieurs fichiers sont passés avec `-f`.
4. 📂 Le **nom du répertoire** du projet contenant le fichier `compose.yaml`\
   ➝ ou contenant le premier fichier indiqué avec `-f`.
5. 📂 Le **nom du répertoire courant** si aucun fichier de configuration n’est spécifié.

***

### ⚠️ Règles de validité des noms de projet

* Les noms doivent contenir uniquement :
  * des lettres minuscules (`a-z`),
  * des chiffres (`0-9`),
  * des tirets (`-`),
  * des underscores (`_`).
* Ils doivent **commencer** par une lettre minuscule ou un chiffre.

👉 Si le nom du répertoire du projet ou du répertoire courant **viole cette règle**, vous devez utiliser l’une des autres méthodes (ex. `-p` ou `COMPOSE_PROJECT_NAME`).

***

### 📚 Voir aussi

* La page **command-line options overview**
* L’option `-p` pour spécifier un nom de projet

## 📂 **COMPOSE\_FILE**

La variable **`COMPOSE_FILE`** permet de **spécifier le chemin vers un fichier Compose**.\
👉 Vous pouvez également indiquer **plusieurs fichiers**.

***

### ⚙️ Comportement par défaut

* Si **aucun fichier n’est précisé**, Compose recherche un fichier nommé **`compose.yaml`** dans le **répertoire courant**.
* S’il n’est pas trouvé, Compose **remonte récursivement dans les répertoires parents** jusqu’à trouver un fichier portant ce nom.

***

### 📑 Utilisation avec plusieurs fichiers

Lorsque plusieurs fichiers sont indiqués, le séparateur de chemin dépend du système :

* **Mac & Linux** 🐧 : `:` (deux-points)
* **Windows** 🪟 : `;` (point-virgule)

***

### 📝 Exemple

```bash
COMPOSE_FILE=compose.yaml:compose.prod.yaml
```

👉 Ici, deux fichiers sont utilisés :

1. `compose.yaml`
2. `compose.prod.yaml`

⚡ Ils sont **fusionnés** par Compose (le second pouvant **surcharger** des éléments du premier).

***

### 🔧 Personnalisation du séparateur

Le séparateur de chemins peut être personnalisé via la variable **`COMPOSE_PATH_SEPARATOR`**.

***

### 📚 Voir aussi

* La page **command-line options overview**
* L’utilisation de **`-f`** pour spécifier un ou plusieurs fichiers Compose (nom et chemin).

## 🗂️ **COMPOSE\_PROFILES**

La variable **`COMPOSE_PROFILES`** permet de spécifier **un ou plusieurs profils** à activer lorsque vous lancez :

```bash
docker compose up
```

***

### ⚙️ Fonctionnement

* Les services associés aux **profils activés** sont démarrés ✅.
* Les services **sans profil** sont toujours démarrés par défaut.

***

### 📝 Exemple simple

```bash
COMPOSE_PROFILES=frontend docker compose up
```

👉 Ici, Compose démarre :

* les services avec le profil **`frontend`**,
* ainsi que tous les services **sans profil**.

***

### 📝 Exemple avec plusieurs profils

```bash
COMPOSE_PROFILES=frontend,debug docker compose up
```

👉 Dans ce cas, Compose démarre :

* tous les services ayant le profil **`frontend`**,
* tous les services ayant le profil **`debug`**,
* et les services **sans profil**.

***

### 📚 Voir aussi

* La section **Using profiles with Compose**
* L’option de ligne de commande **`--profile`**

## ➖ **COMPOSE\_PATH\_SEPARATOR**

La variable **`COMPOSE_PATH_SEPARATOR`** permet de **spécifier un séparateur de chemin personnalisé** pour les éléments listés dans **`COMPOSE_FILE`**.

***

### ⚙️ Valeur par défaut

* Sur **macOS et Linux** 🐧🍏 → le séparateur est **`:`** (deux-points).
* Sur **Windows** 🪟 → le séparateur est **`;`** (point-virgule).

***

👉 Cette variable est utile si vous devez adapter ou harmoniser la gestion des chemins entre différents systèmes d’exploitation.

## 📑 **COMPOSE\_ENV\_FILES**

La variable **`COMPOSE_ENV_FILES`** permet de spécifier **quels fichiers d’environnement** Docker Compose doit utiliser **si l’option `--env-file` n’est pas fournie**.

***

### ⚙️ Utilisation avec plusieurs fichiers

Lorsque vous utilisez plusieurs fichiers, séparez-les par une **virgule**.

#### 📝 Exemple

```bash
COMPOSE_ENV_FILES=.env.envfile1,.env.envfile2
```

👉 Dans cet exemple, Compose charge les variables des fichiers :

1. `.env.envfile1`
2. `.env.envfile2`

***

### 📖 Comportement par défaut

* Si **`COMPOSE_ENV_FILES`** n’est pas défini **et** que vous n’utilisez pas `--env-file` dans la CLI,\
  ➡️ Docker Compose adopte son comportement par défaut : il recherche automatiquement un fichier **`.env`** dans le répertoire du projet.

***

✅ Cette variable est utile pour centraliser la gestion de plusieurs environnements (`.env.dev`, `.env.prod`, `.env.test`), sans devoir toujours préciser `--env-file`.

## 🚫 **COMPOSE\_DISABLE\_ENV\_FILE**

La variable **`COMPOSE_DISABLE_ENV_FILE`** permet de **désactiver l’utilisation du fichier `.env` par défaut**.

***

### ⚙️ Valeurs possibles

* **`true`** ou **`1`** → Compose **ignore** totalement le fichier `.env`.
* **`false`** ou **`0`** → Compose recherche un fichier **`.env`** dans le répertoire du projet.

***

### 📖 Valeur par défaut

* Par défaut : **`0`**\
  👉 Cela signifie que Compose **cherche automatiquement** un fichier `.env` dans le dossier du projet.

***

✅ Utile si vous ne souhaitez pas que Compose charge automatiquement des variables d’environnement depuis un fichier `.env`.

## 🌍 Gestion des environnements et cycle de vie des conteneurs

### 💻 **COMPOSE\_CONVERT\_WINDOWS\_PATHS**

Lorsque cette option est activée, **Docker Compose convertit automatiquement les chemins Windows en chemins Unix** dans les définitions de volumes.

***

### ⚙️ Valeurs possibles

* **`true`** ou **`1`** → conversion activée ✅
* **`false`** ou **`0`** → conversion désactivée ❌

***

### 📖 Valeur par défaut

* **`0`** (conversion désactivée par défaut).

***

👉 Cette option est particulièrement utile lorsque vous travaillez sous **Windows** 🪟 et que vous devez monter des volumes dans des conteneurs **Linux** 🐧 sans adapter manuellement les chemins (`C:\Users\...` → `/c/Users/...`).

## 🗑️ Gestion des conteneurs orphelins et parallélisme dans Compose

***

### 🛑 **COMPOSE\_IGNORE\_ORPHANS**

Lorsqu’elle est activée, cette variable indique à **Compose** de **ne pas essayer de détecter les conteneurs orphelins** liés au projet.

#### ⚙️ Valeurs possibles

* **`true`** ou **`1`** → détection désactivée ✅
* **`false`** ou **`0`** → détection activée ❌

#### 📖 Valeur par défaut

* **`0`** (Compose détecte les orphelins et avertit par défaut).

***

### 🗑️ **COMPOSE\_REMOVE\_ORPHANS**

Lorsqu’elle est activée, Compose **supprime automatiquement** les conteneurs orphelins lors de la mise à jour d’un service ou d’une stack.

👉 Un **conteneur orphelin** est un conteneur créé par une configuration précédente mais qui n’est plus défini dans le fichier **`compose.yaml`** actuel.

#### ⚙️ Valeurs possibles

* **`true`** ou **`1`** → suppression automatique activée 🧹
* **`false`** ou **`0`** → suppression automatique désactivée ❌ (Compose affiche alors un **avertissement** au lieu de supprimer).

#### 📖 Valeur par défaut

* **`0`** (Compose ne supprime pas automatiquement, mais affiche un avertissement).

***

### ⚡ **COMPOSE\_PARALLEL\_LIMIT**

Définit le **niveau maximal de parallélisme** pour les appels simultanés au moteur Docker.

👉 Utile pour contrôler la charge lorsque Compose démarre, arrête ou met à jour plusieurs services en parallèle.

## 🖨️ Sortie et affichage dans Compose

***

### 🎨 **COMPOSE\_ANSI**

Spécifie **quand afficher les caractères de contrôle ANSI** (utilisés pour la couleur et le formatage dans le terminal).

#### ⚙️ Valeurs possibles

* **`auto`** → Compose détecte si le mode **TTY** peut être utilisé. Sinon, il bascule en mode texte simple.
* **`never`** → toujours en mode texte simple (pas de couleurs).
* **`always`** ou **`0`** → toujours en mode **TTY** (forces les couleurs et le formatage).

#### 📖 Valeur par défaut

* **`auto`**

***

### 📊 **COMPOSE\_STATUS\_STDOUT**

Lorsqu’elle est activée, Compose écrit ses **messages internes de statut et de progression** dans **`stdout`** au lieu de **`stderr`**.

👉 Par défaut, la valeur est **`false`** pour bien séparer :

* les **messages Compose**
* des **logs de vos conteneurs**.

#### ⚙️ Valeurs possibles

* **`true`** ou **`1`** → activé ✅
* **`false`** ou **`0`** → désactivé ❌

#### 📖 Valeur par défaut

* **`0`** (désactivé).

***

### ⏳ **COMPOSE\_PROGRESS**

⚠️ **Nécessite Docker Compose 2.36.0 ou supérieur**

Définit le **type d’affichage de la progression**, sauf si l’option **`--progress`** est utilisée dans la CLI.

#### ⚙️ Valeurs possibles

* **`auto`** → détection automatique
* **`tty`** → affichage interactif avec barre de progression
* **`plain`** → affichage simple en texte brut
* **`json`** → sortie au format JSON
* **`quiet`** → désactive l’affichage de la progression

#### 📖 Valeur par défaut

* **`auto`**

***

👉 Ces variables permettent d’adapter l’affichage des logs et des statuts de Compose selon vos besoins : lisibilité, intégration CI/CD, ou debugging.

## 👤 Expérience utilisateur

***

### 📜 **COMPOSE\_MENU**

⚠️ **Disponible à partir de Docker Compose 2.26.0 et ultérieur**

Lorsqu’elle est activée, cette variable permet à Compose d’afficher un **menu de navigation** interactif où vous pouvez :

* ouvrir la stack Compose dans **Docker Desktop**,
* activer le **mode watch**,
* utiliser **Docker Debug**.

#### ⚙️ Valeurs possibles

* **`true`** ou **`1`** → activation ✅
* **`false`** ou **`0`** → désactivation ❌

#### 📖 Valeur par défaut

* **`1`** si vous utilisez Docker Compose via **Docker Desktop**
* **`0`** sinon (par défaut désactivé).

***

### 🧪 **COMPOSE\_EXPERIMENTAL**

⚠️ **Disponible à partir de Docker Compose 2.26.0 et ultérieur**

Il s’agit d’une variable **opt-out** (_désactivation volontaire_).\
👉 Lorsqu’elle est désactivée, elle **désactive toutes les fonctionnalités expérimentales**.

#### ⚙️ Valeurs possibles

* **`true`** ou **`1`** → activation des fonctionnalités expérimentales 🚀
* **`false`** ou **`0`** → désactivation ❌

#### 📖 Valeur par défaut

* **`1`** (fonctionnalités expérimentales activées par défaut).

***

✅ Ces deux variables influencent directement l’**expérience utilisateur** avec Compose :

* **`COMPOSE_MENU`** → interface et navigation
* **`COMPOSE_EXPERIMENTAL`** → accès aux nouvelles fonctionnalités encore en test

## ⛔ Non pris en charge dans Compose V2

Les variables d’environnement suivantes **n’ont aucun effet dans Compose V2**.\
👉 Pour plus d’informations, voir la page **Migrate to Compose V2**.

***

### 📋 Liste des variables obsolètes

#### 🛠️ **COMPOSE\_API\_VERSION**

* Par défaut, la version de l’API est **négociée automatiquement** avec le serveur.
* 👉 À la place, utilisez **`DOCKER_API_VERSION`**.
* 📚 Voir la page **Docker CLI environment variable reference**.

***

#### ⏳ **COMPOSE\_HTTP\_TIMEOUT**

Non pris en charge dans Compose V2.

***

#### 🔒 **COMPOSE\_TLS\_VERSION**

Non pris en charge dans Compose V2.

***

#### 💻 **COMPOSE\_FORCE\_WINDOWS\_HOST**

Non pris en charge dans Compose V2.

***

#### ⌨️ **COMPOSE\_INTERACTIVE\_NO\_CLI**

Non pris en charge dans Compose V2.

***

#### 🏗️ **COMPOSE\_DOCKER\_CLI\_BUILD**

À la place, utilisez **`DOCKER_BUILDKIT`** pour choisir entre :

* **BuildKit** (nouveau moteur de build),
* ou le **builder classique**.

👉 Si `DOCKER_BUILDKIT=0`, alors `docker compose build` utilise le builder classique pour construire les images.

***

✅ En résumé : certaines anciennes variables ne fonctionnent plus dans Compose V2 → il faut désormais utiliser les **équivalents du Docker CLI** (`DOCKER_API_VERSION`, `DOCKER_BUILDKIT`, etc.).
