# 🌍 Définir des variables d’environnement dans l’environnement d’un conteneur

⚠️ L’environnement d’un conteneur n’est pas défini **par défaut**.\
👉 Il faut une **entrée explicite** dans la configuration du service pour le mettre en place.

Avec **Docker Compose**, vous disposez de **deux façons** pour définir des variables d’environnement dans vos conteneurs via le fichier `compose.yml`.

***

### 💡 Astuce

🚫 N’utilisez pas les variables d’environnement pour transmettre des informations sensibles (comme des **mots de passe** 🔑).\
➡️ Préférez plutôt l’utilisation des **secrets** pour gérer ce type de données sensibles en toute sécurité 🔒.

## ⚙️ Utiliser l’attribut `environment`

Vous pouvez définir des **variables d’environnement** directement dans l’environnement de votre conteneur à l’aide de l’attribut **`environment`** dans votre fichier `compose.yaml`.

***

### 📝 Exemple (syntaxe mapping)

```yaml
services:
  webapp:
    environment:
      DEBUG: "true"
```

***

### 📝 Exemple équivalent (syntaxe liste)

```yaml
services:
  webapp:
    environment:
      - DEBUG=true
```

***

👉 Les deux syntaxes produisent le **même résultat** ✅.

📚 Consultez la section **`environment` attribute** de la documentation pour voir davantage d’exemples et d’options d’utilisation.

## ℹ️ Informations supplémentaires sur `environment`

#### 🔄 Passer des variables directement depuis votre shell

Vous pouvez choisir de **ne pas définir de valeur** et de transmettre directement les variables d’environnement de votre **shell** vers vos conteneurs.

👉 Cela fonctionne de la même manière que la commande :

```bash
docker run -e VARIABLE ...
```

***

### 📝 Exemple

```yaml
services:
  web:
    environment:
      - DEBUG
```

➡️ Dans ce cas, la valeur de la variable **`DEBUG`** dans le conteneur sera prise depuis la valeur de la même variable dans le **shell** où vous lancez `docker compose`.

⚠️ Attention : si la variable **`DEBUG`** n’est pas définie dans l’environnement du shell, **aucun avertissement** ne sera affiché.

***

#### 🔄 Utiliser l’interpolation

Vous pouvez aussi profiter de l’**interpolation** pour plus de sécurité et de clarté.

***

### 📝 Exemple avec interpolation

```yaml
services:
  web:
    environment:
      - DEBUG=${DEBUG}
```

👉 Ici, le comportement est similaire à l’exemple précédent.\
➡️ Mais différence importante : **Compose vous prévient** si la variable `DEBUG` n’est pas définie dans l’environnement du shell **ou** dans un fichier `.env` du répertoire du projet.

***

✅ Cela permet d’éviter les oublis et de sécuriser davantage votre configuration.

## 📂 Utiliser l’attribut `env_file`

L’environnement d’un conteneur peut aussi être défini en utilisant des fichiers **`.env`** combinés à l’attribut **`env_file`**.

***

### 📝 Exemple

```yaml
services:
  webapp:
    env_file: "webapp.env"
```

***

### 📖 Avantages de l’utilisation des fichiers `.env`

1. ✅ Vous pouvez réutiliser le **même fichier `.env`** :
   *   avec une commande classique :

       ```bash
       docker run --env-file ...
       ```
   * ou bien dans plusieurs services Compose, sans dupliquer un long bloc `environment` dans le YAML.
2. ✅ Cela permet de **séparer les variables d’environnement** de votre fichier principal `compose.yaml`.
   * ➝ Organisation plus claire 🗂️
   * ➝ Plus de sécurité 🔒 : vous pouvez stocker vos secrets ailleurs, sans devoir les placer en clair dans la config principale.
3. ✅ L’attribut **`env_file`** supporte l’utilisation de **plusieurs fichiers `.env`** au sein de la même application Compose.
4. ✅ Les chemins indiqués dans `env_file` sont **relatifs** à l’emplacement de votre fichier `compose.yaml`.

***

### ⚠️ Important

* L’**interpolation** dans les fichiers `.env` est une **fonctionnalité du CLI Docker Compose**.
*   Elle n’est **pas supportée** si vous utilisez directement :

    ```bash
    docker run --env-file ...
    ```

***

👉 En résumé : `env_file` rend la gestion des variables d’environnement plus **simple, réutilisable, sécurisée et organisée**.

## ℹ️ Informations supplémentaires sur `env_file`

#### 🔄 Gestion de plusieurs fichiers

Si plusieurs fichiers `.env` sont spécifiés, ils sont **évalués dans l’ordre** 📑.\
👉 Les valeurs définies dans les fichiers suivants **peuvent écraser** celles définies dans les fichiers précédents.

***

#### ⚙️ Fichiers `.env` optionnels (depuis Compose 2.24.0)

Depuis **Docker Compose 2.24.0**, vous pouvez définir un fichier `.env` comme étant **optionnel** grâce au champ **`required`**.

* Si `required: true` (par défaut), l’absence du fichier génère une erreur ❌.
* Si `required: false`, et que le fichier est manquant, Compose **ignore silencieusement** cette entrée ✅.

***

### 📝 Exemple

```yaml
env_file:
  - path: ./default.env
    required: true  # comportement par défaut
  - path: ./override.env
    required: false # fichier optionnel
```

***

#### 📂 Formats alternatifs (depuis Compose 2.30.0)

Depuis **Docker Compose 2.30.0**, vous pouvez utiliser un **format alternatif** pour les fichiers définis par `env_file`, grâce à l’attribut **`format`**.\
👉 Pour plus de détails, voir la section **format** de la documentation.

***

#### 🖥️ Priorité CLI

Les valeurs définies dans vos fichiers `.env` peuvent être **surchargées depuis la ligne de commande** en utilisant :

```bash
docker compose run -e VARIABLE=valeur
```

***

✅ En résumé :

* Vous pouvez **enchaîner plusieurs `.env`**,
* définir certains comme **optionnels**,
* utiliser différents **formats**,
* et **écraser les valeurs à la volée** via le CLI.

## ⚡ Définir des variables d’environnement avec `docker compose run --env`

De la même manière que la commande classique **`docker run --env`**,\
👉 vous pouvez définir des **variables d’environnement temporaires** avec :

* **`docker compose run --env`**
* ou sa forme abrégée **`docker compose run -e`**

***

### 📝 Exemple

```bash
docker compose run -e DEBUG=1 web python console.py
```

***

### 📖 Explication

* Ici, la variable d’environnement **`DEBUG`** est définie uniquement pour l’exécution de cette commande.
* Le service **`web`** est lancé avec `DEBUG=1` puis exécute le script `console.py` avec Python.
* ⚡ Cette variable **n’est pas persistante** : elle disparaît après l’arrêt du conteneur.

***

✅ Utile pour tester rapidement un comportement, activer temporairement un mode debug 🐛, ou exécuter un script avec une configuration différente sans modifier vos fichiers `compose.yaml` ni vos `.env`.

## ℹ️ Informations supplémentaires sur `--env`

Vous pouvez également transmettre une variable depuis votre **shell** ou vos **fichiers d’environnement** sans lui donner de valeur explicite.

***

### 📝 Exemple

```bash
docker compose run -e DEBUG web python console.py
```

***

### 📖 Explication

* Dans cet exemple, la variable **`DEBUG`** n’a pas de valeur définie directement dans la commande.
* 👉 La valeur de `DEBUG` dans le conteneur est alors prise depuis :
  1. La valeur de la variable **dans le shell** où vous exécutez `docker compose`,
  2. Ou bien depuis vos **fichiers d’environnement** (`.env` ou `env_file`).

***

✅ Cela vous permet de **réutiliser automatiquement vos variables existantes** sans avoir à les réécrire manuellement dans la commande.
