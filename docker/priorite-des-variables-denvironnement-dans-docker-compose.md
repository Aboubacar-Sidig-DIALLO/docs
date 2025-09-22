# 🔄 Priorité des variables d’environnement dans Docker Compose

Lorsqu’une même variable d’environnement est définie dans **plusieurs sources**,\
👉 **Docker Compose applique une règle de priorité** pour déterminer **quelle valeur finale** sera utilisée dans l’environnement de votre conteneur.

Cette page explique comment Docker Compose choisit la valeur retenue lorsqu’une variable est définie à plusieurs endroits.

***

### 🏆 Ordre de priorité (du plus fort au plus faible)

1. **Définie via la CLI** avec `docker compose run -e`\
   ➝ la valeur définie dans la ligne de commande a toujours la priorité absolue.
2. **Définie avec `environment` ou `env_file` dans le Compose file**,\
   ➝ mais **valeur interpolée** depuis votre shell ou un fichier d’environnement (`.env` par défaut ou celui passé via `--env-file`).
3. **Définie uniquement avec l’attribut `environment`** dans le `compose.yaml`.
4. **Définie avec `env_file`** dans le `compose.yaml`.
5. **Définie dans l’image du conteneur** (`ENV` dans un `Dockerfile`).\
   ➝ Les directives `ARG` ou `ENV` dans un Dockerfile ne s’appliquent **que si aucune valeur n’est fournie** via `environment`, `env_file` ou `run --env`.

***

### ✅ En résumé

* 🎯 **La CLI domine** : si vous définissez une variable avec `-e`, c’est elle qui gagne.
* 📝 Ensuite, Compose regarde vos variables interpolées depuis le shell ou vos fichiers `.env`.
* ⚙️ Puis, il prend celles définies directement dans `compose.yaml` (`environment`).
* 📂 Enfin, il retient celles des `env_file`.
* 🐳 Et en dernier recours, il prend celles de l’image (Dockerfile).

***

👉 Cette hiérarchie vous permet de **gérer des valeurs par défaut** dans vos images et fichiers, tout en pouvant facilement les **surcharger au moment de l’exécution**.

## 📝 Exemple simple

Dans l’exemple ci-dessous, une même variable d’environnement (**`NODE_ENV`**) reçoit **deux valeurs différentes** :

* Une valeur définie dans un fichier **`.env`**,
* Une autre valeur définie directement dans l’attribut **`environment`** du fichier `compose.yaml`.

***

### 📂 Contenu du fichier `.env`

```bash
cat ./webapp.env
NODE_ENV=test
```

***

### 📂 Contenu du fichier `compose.yaml`

```yaml
services:
  webapp:
    image: 'webapp'
    env_file:
     - ./webapp.env
    environment:
     - NODE_ENV=production
```

***

### 📖 Explication

* La variable `NODE_ENV` est définie dans deux endroits :
  * `webapp.env` ➝ valeur = `test`
  * `environment` ➝ valeur = `production`

👉 Dans ce cas, **la valeur définie dans `environment` prend le dessus** ✅.

***

### ▶️ Vérification avec la commande

```bash
docker compose run webapp env | grep NODE_ENV
```

#### Résultat :

```
NODE_ENV=production
```

***

✅ Cela illustre bien la règle de **priorité des variables d’environnement** :\
`environment` > `env_file` > `ENV` du Dockerfile.

## 🧩 Exemple avancé

Dans cet exemple, on utilise une variable d’environnement nommée **`VALUE`**, qui définit la version d’une image.

***

### 🛠️ Comment lire le tableau

* Chaque **colonne** représente un **contexte** où une valeur peut être définie ou substituée pour la variable `VALUE`.
  * **docker compose run** (`--env`)
  * **environment attribute** (`environment` dans `compose.yaml`)
  * **env\_file attribute** (`env_file` dans `compose.yaml`)
  * **Image ENV** (`ENV` dans le `Dockerfile`)
  * **Host OS environment** (variable définie dans votre shell)
  * **.env file** (fichier `.env` à la racine du projet)

⚠️ Les colonnes **Host OS environment** et **.env file** sont listées **uniquement à titre illustratif** :\
👉 à elles seules, elles ne définissent pas de variable dans le conteneur.\
Elles doivent être utilisées avec **`environment`** ou **`env_file`** dans Compose pour être injectées.

* Chaque **ligne** représente une **combinaison de contextes** où la variable `VALUE` est définie.
* La colonne **Result** indique la **valeur finale** de `VALUE` dans le conteneur.

***

### 📊 Tableau de priorité

| #  | docker compose run | environment attribute | env\_file attribute | Image ENV | Host OS env | .env file | Résultat final |
| -- | ------------------ | --------------------- | ------------------- | --------- | ----------- | --------- | -------------- |
| 1  | -                  | -                     | -                   | -         | VALUE=1.4   | VALUE=1.3 | -              |
| 2  | -                  | -                     | VALUE=1.6           | VALUE=1.5 | VALUE=1.4   | -         | **1.6**        |
| 3  | -                  | VALUE=1.7             | -                   | VALUE=1.5 | VALUE=1.4   | -         | **1.7**        |
| 4  | -                  | -                     | -                   | VALUE=1.5 | VALUE=1.4   | VALUE=1.3 | **1.5**        |
| 5  | --env VALUE=1.8    | -                     | -                   | VALUE=1.5 | VALUE=1.4   | VALUE=1.3 | **1.8**        |
| 6  | --env VALUE        | -                     | -                   | VALUE=1.5 | VALUE=1.4   | VALUE=1.3 | **1.4**        |
| 7  | --env VALUE        | -                     | -                   | VALUE=1.5 | -           | VALUE=1.3 | **1.3**        |
| 8  | -                  | -                     | VALUE               | VALUE=1.5 | VALUE=1.4   | VALUE=1.3 | **1.4**        |
| 9  | -                  | -                     | VALUE               | VALUE=1.5 | -           | VALUE=1.3 | **1.3**        |
| 10 | -                  | VALUE                 | -                   | VALUE=1.5 | VALUE=1.4   | VALUE=1.3 | **1.4**        |
| 11 | -                  | VALUE                 | -                   | VALUE=1.5 | -           | VALUE=1.3 | **1.3**        |
| 12 | --env VALUE        | VALUE=1.7             | -                   | VALUE=1.5 | VALUE=1.4   | VALUE=1.3 | **1.4**        |
| 13 | --env VALUE=1.8    | VALUE=1.7             | -                   | VALUE=1.5 | VALUE=1.4   | VALUE=1.3 | **1.8**        |
| 14 | --env VALUE=1.8    | -                     | VALUE=1.6           | VALUE=1.5 | VALUE=1.4   | VALUE=1.3 | **1.8**        |
| 15 | --env VALUE=1.8    | VALUE=1.7             | VALUE=1.6           | VALUE=1.5 | VALUE=1.4   | VALUE=1.3 | **1.8**        |

***

### 📖 À retenir

* 🎯 **La CLI (`--env`) domine toujours**.
* Ensuite viennent les définitions via **`environment`** et **`env_file`**.
* Puis celles de l’image (`ENV` dans le Dockerfile).
* Enfin, les valeurs des fichiers `.env` et de l’environnement du shell ne s’appliquent qu’**indirectement** via interpolation.

***

👉 Cet exemple avancé illustre parfaitement l’importance de connaître l’**ordre de priorité** pour éviter les surprises lors du déploiement.

## 🔎 Comprendre les résultats de priorité des variables

Voici une explication des résultats du tableau précédent, ligne par ligne :

***

#### 🟢 Résultat 1

L’environnement local (Host OS) définit une valeur, mais le fichier Compose **ne la réplique pas** dans le conteneur.\
👉 Donc, la variable **n’est pas définie** dans l’environnement du conteneur.

***

#### 🟢 Résultat 2

L’attribut **`env_file`** du fichier Compose définit explicitement une valeur pour `VALUE`.\
👉 L’environnement du conteneur prend donc cette valeur.

***

#### 🟢 Résultat 3

L’attribut **`environment`** du fichier Compose définit explicitement une valeur pour `VALUE`.\
👉 L’environnement du conteneur prend donc cette valeur.

***

#### 🟢 Résultat 4

La directive **`ENV`** dans l’image (Dockerfile) déclare `VALUE`.\
👉 Comme le fichier Compose ne redéfinit pas cette variable, c’est **la valeur de l’image** qui est retenue.

***

#### 🟢 Résultat 5

La commande **`docker compose run`** utilise l’option `--env` avec une valeur explicite.\
👉 Cela **écrase** la valeur définie dans l’image.

***

#### 🟢 Résultat 6

La commande **`docker compose run`** utilise `--env VALUE` sans valeur explicite.\
👉 Cela réplique la valeur depuis l’environnement **du Host OS**, qui a la priorité.

***

#### 🟢 Résultat 7

Même cas que ci-dessus (`--env VALUE`), mais comme aucune valeur n’existe dans le Host OS,\
👉 c’est la valeur définie dans le fichier **`.env`** qui est utilisée.

***

#### 🟢 Résultat 8

L’attribut **`env_file`** du Compose file est configuré pour répliquer `VALUE` depuis l’environnement local.\
👉 La valeur du **Host OS** prend la priorité et est injectée dans le conteneur.

***

#### 🟢 Résultat 9

Même cas, mais comme aucune valeur n’est disponible dans le Host OS,\
👉 c’est la valeur issue du **fichier `.env`** qui est utilisée.

***

#### 🟢 Résultat 10

L’attribut **`environment`** du Compose file est configuré pour répliquer `VALUE` depuis l’environnement local.\
👉 La valeur du **Host OS** a la priorité et est injectée dans le conteneur.

***

#### 🟢 Résultat 11

Même cas, mais si la variable n’est pas définie dans le Host OS,\
👉 la valeur du **fichier `.env`** est choisie.

***

#### 🟢 Résultat 12

Le flag **`--env`** a une priorité plus élevée que `environment` et `env_file`.\
👉 Comme il est configuré pour répliquer la valeur depuis l’environnement local,\
➡️ c’est la valeur du **Host OS** qui est utilisée.

***

#### 🟢 Résultats 13 à 15

Le flag **`--env`** a toujours la priorité sur `environment` et `env_file`.\
👉 Il impose directement la valeur spécifiée.

***

✅ En résumé :

* **`--env` (CLI)** domine toujours,
* ensuite viennent **`environment`** et **`env_file`**,
* puis enfin les valeurs définies dans l’image (`ENV`).
