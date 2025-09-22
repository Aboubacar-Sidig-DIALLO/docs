# 🔐 Gérer les secrets en toute sécurité avec Docker Compose

### 🧐 Qu’est-ce qu’un _secret_ ?

Un **secret** est toute donnée sensible que vous devez protéger, par exemple :

* un **mot de passe** 🔑,
* un **certificat** 📜,
* ou une **clé API** 🔒.

👉 Ces données ne doivent **jamais** être :

* transmises en clair sur un réseau 🌐,
* stockées sans chiffrement dans un **Dockerfile** 🐳,
* ou intégrées directement dans le **code source** de votre application 💻.

***

### 🚫 Limites des variables d’environnement

Beaucoup de développeurs utilisent encore des **variables d’environnement** (`ENV`) pour injecter des mots de passe ou des clés API. Mais cette pratique présente plusieurs risques ⚠️ :

* Les **variables d’environnement** sont accessibles par **tous les processus** d’un conteneur.
* Il est **difficile de tracer** qui y accède réellement.
* Elles peuvent être **imprimées accidentellement dans les logs** 📝 lors d’un débogage.

👉 Résultat : vos informations sensibles peuvent être **exposées sans que vous le sachiez**.

***

### ✅ La solution : les _secrets_ dans Compose

Docker Compose offre une méthode sécurisée pour gérer vos secrets :

* Les secrets sont définis **hors du code et du Dockerfile**.
* Les services y accèdent uniquement si vous les déclarez explicitement via l’attribut **`secrets`** au niveau du service.
* Cela permet de **réduire les risques d’exposition accidentelle** de vos informations sensibles.

***

✨ En résumé :

* ❌ N’utilisez pas les variables d’environnement pour vos mots de passe ou clés API.
* ✅ Utilisez les **secrets Compose**, qui offrent un **contrôle explicite** et plus de **sécurité** 🔐.

## 🔐 Utiliser les secrets dans Docker Compose

### 📂 Où sont stockés les secrets ?

Les **secrets** sont **montés comme des fichiers** à l’intérieur du conteneur, dans le chemin suivant :

```
/run/secrets/<nom_du_secret>
```

***

### 🛠️ Étapes pour injecter un secret dans un conteneur

1. **Définir le secret** ➝ dans l’élément **top-level `secrets`** de votre fichier `compose.yaml`.
2. **Attribuer le secret** ➝ à un service en le déclarant via l’attribut **`secrets`** dans la configuration de ce service.

👉 Ainsi, Compose **accorde l’accès aux secrets service par service** (granularité fine).\
👉 À l’intérieur du conteneur, l’accès peut ensuite être contrôlé via les **permissions classiques du système de fichiers** 🔒.

***

### 🧩 Exemples

#### ✅ Exemple 1 : Injection d’un secret dans un seul service

```yaml
services:
  myapp:
    image: myapp:latest
    secrets:
      - my_secret

secrets:
  my_secret:
    file: ./my_secret.txt
```

➡️ Dans ce cas :

* Le service **`myapp`** accède au secret **`my_secret`**.
*   Dans le conteneur, le secret est monté sous :

    ```
    /run/secrets/my_secret
    ```
* Le contenu du fichier `./my_secret.txt` est disponible dans ce chemin.

***

#### ✅ Exemple 2 : Partage de secrets entre plusieurs services (exemple WordPress + MySQL)

```yaml
services:
   db:
     image: mysql:latest
     volumes:
       - db_data:/var/lib/mysql
     environment:
       MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD_FILE: /run/secrets/db_password
     secrets:
       - db_root_password
       - db_password

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD_FILE: /run/secrets/db_password
     secrets:
       - db_password

secrets:
   db_password:
     file: db_password.txt
   db_root_password:
     file: db_root_password.txt

volumes:
   db_data:
```

***

#### 🔎 Explication de cet exemple avancé

* Chaque service (ici `db` et `wordpress`) déclare **explicitement** les secrets dont il a besoin via l’attribut **`secrets`**.
* La section **top-level `secrets`** définit :
  * `db_password` (contenu de `db_password.txt`)
  * `db_root_password` (contenu de `db_root_password.txt`)
*   Lors du déploiement, Docker crée un **bind mount** sous :

    ```
    /run/secrets/<nom_du_secret>
    ```

    avec les valeurs contenues dans les fichiers.

***

### 💡 Note importante

L’utilisation des variables d’environnement terminant par **`_FILE`** (ex. `MYSQL_PASSWORD_FILE`) est une **convention** utilisée par certaines images officielles Docker, comme **MySQL** et **Postgres**.\
👉 Cela permet d’indiquer à l’image de lire le secret directement depuis un fichier monté.

***

✅ En résumé :

* Déclarez vos secrets en haut du `compose.yaml`.
* Attribuez-les uniquement aux services qui en ont besoin.
* Docker monte les secrets comme **fichiers sécurisés** dans `/run/secrets/`.
* Cette approche réduit considérablement les risques liés aux variables d’environnement.

## 🏗️ Build secrets (Secrets de build)

### 🔑 À quoi servent les _build secrets_ ?

Les **build secrets** permettent de fournir des informations sensibles **uniquement au moment de la construction de l’image** (avec `docker compose build`).\
👉 Exemple typique : un **jeton NPM** (`NPM_TOKEN`) pour télécharger des dépendances privées lors du `npm install`.

⚠️ Contrairement aux variables d’environnement classiques, les build secrets ne sont **pas intégrés dans l’image finale**, ce qui améliore la **sécurité** 🔐.

***

### 📑 Exemple

```yaml
services:
  myapp:
    build:
      secrets:
        - npm_token
      context: .

secrets:
  npm_token:
    environment: NPM_TOKEN
```

***

#### ▶️ Explication

* Le secret **`npm_token`** est défini au niveau **top-level `secrets`**.
* Sa valeur est prise depuis la variable d’environnement **`NPM_TOKEN`** disponible sur la machine hôte.
* Lors de la construction de l’image (`docker compose build`), le secret est rendu accessible au conteneur de build.
* L’image construite peut alors, par exemple, exécuter `npm install` avec accès à des packages privés protégés par ce token.

👉 Mais une fois le build terminé :

* le secret **n’apparaît pas dans l’image finale**,
* il reste temporaire et utilisé uniquement au moment du build.

***

### 📚 Ressources associées

* 🔗 Élément top-level `secrets`
* 🔗 Attribut `secrets` pour les services
* 🔗 Build secrets

***

✅ En résumé :

* Les **build secrets** = secrets accessibles **uniquement pendant la construction** d’une image.
* Sécurité renforcée : **aucune fuite** dans l’image finale.
* Exemple courant : **tokens API** ou **clés privées** nécessaires pour télécharger des dépendances.
