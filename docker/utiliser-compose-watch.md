---
description: >-
  📋 Options de la page  Nécessite : Docker Compose 2.22.0 ou version
  ultérieure.
---

# 👀 Utiliser Compose Watch

### 🎯 Qu’est-ce que Compose Watch ?

L’attribut **`watch`** permet de :

* **mettre à jour automatiquement** vos services Compose en cours d’exécution,
* et de **prévisualiser les changements** dès que vous modifiez et sauvegardez votre code ✨.

👉 Pour de nombreux projets, cela permet d’avoir un **workflow de développement sans intervention manuelle** :

* vous lancez vos services avec Compose,
* puis ils se **mettent à jour automatiquement** au fur et à mesure que vous travaillez.

***

### 📂 Règles sur les chemins de fichiers surveillés

* Tous les chemins sont **relatifs au répertoire du projet**, sauf pour les motifs d’ignorés.
* Les **répertoires sont surveillés récursivement**.
* ❌ Les **glob patterns** (`*.js`, `**/*.ts`, etc.) ne sont pas pris en charge.
* Les règles de **`.dockerignore`** s’appliquent ✅.
* Vous pouvez utiliser l’option **`ignore`** pour définir des chemins supplémentaires à exclure (même syntaxe que `.dockerignore`).
* Les fichiers temporaires / de sauvegarde des IDE courants (**Vim, Emacs, JetBrains, etc.**) sont automatiquement ignorés 🛑.
* Les répertoires **`.git`** sont également ignorés automatiquement.

***

### 🛠️ Bonnes pratiques

* Vous n’avez **pas besoin d’activer `watch` pour tous vos services**.
* Dans de nombreux cas, seul un **sous-ensemble du projet** est pertinent pour la mise à jour automatique.\
  ➝ Exemple : le **frontend JavaScript** 🖥️ peut bénéficier de `watch`, mais pas forcément la base de données.

***

### ⚠️ Limites

* **Compose Watch** fonctionne uniquement avec les services construits à partir du code source local via l’attribut **`build`**.
* ❌ Il ne suit pas les changements pour les services qui utilisent une **image pré-construite** définie avec l’attribut **`image`**.

***

✅ En résumé :\
Compose Watch = **rechargement automatique des services** lors des changements de code → idéal pour un workflow **rapide, fluide et sans interruptions** 🎉.

## ⚖️ Compose Watch vs Bind Mounts

### 📂 Bind mounts (rappel)

Docker Compose permet de **partager un répertoire de l’hôte** à l’intérieur des conteneurs de service (via des **bind mounts**).

👉 Le mode **`watch`** ne remplace pas cette fonctionnalité.\
Au contraire, il agit comme un **complément** 🧩, spécialement conçu pour le **développement dans des conteneurs**.

***

### 🎯 Granularité plus fine avec Watch

L’avantage majeur de **`watch`** par rapport aux bind mounts est la **granularité** :

* Avec un bind mount → **tout le répertoire est partagé**, sans distinction.
* Avec `watch` → vous pouvez **ignorer certains fichiers ou dossiers** du répertoire surveillé.

***

### 📝 Exemple pratique

Dans un projet **JavaScript**, vous pourriez vouloir **ignorer le dossier `node_modules/`**.

#### ✨ Avantages :

1. **Performance** ⚡
   * Les dossiers contenant **beaucoup de petits fichiers** (comme `node_modules/`) peuvent causer une **forte charge d’I/O**.
   * Les exclure améliore la vitesse.
2. **Multi-plateforme** 🌍
   * Les artefacts compilés ne sont pas toujours compatibles entre l’**OS hôte** et l’**architecture du conteneur**.
   * Ignorer `node_modules/` évite des problèmes de portabilité.

***

### ⚠️ Cas spécifique à Node.js

Dans un projet **Node.js** :

* Il est **déconseillé de synchroniser le dossier `node_modules/`** avec un bind mount.
* Même si JavaScript est interprété, les **packages npm** peuvent contenir du **code natif compilé**,\
  ➝ ce qui rend ces fichiers **non portables** entre différentes plateformes.

***

✅ En résumé :

* **Bind mount** = partage direct d’un répertoire complet de l’hôte.
* **Compose Watch** = complément qui offre un **contrôle précis** (ignorer certains fichiers/dossiers).
* Cas d’usage parfait : **ignorer `node_modules/` dans Node.js** pour améliorer les performances et éviter les soucis multi-OS.

## ⚙️ Configuration de **Compose Watch**

### 📌 Définition

L’attribut **`watch`** définit une **liste de règles** qui contrôlent la **mise à jour automatique des services** en fonction des **modifications de fichiers locaux**.

***

### 📑 Structure des règles

Chaque règle doit contenir :

* **un chemin (path pattern)** à surveiller,
* **une action (action)** à exécuter lorsqu’une modification est détectée.

👉 Il existe **deux types d’actions possibles** dans `watch`.\
Selon l’action choisie, des **champs supplémentaires** peuvent être nécessaires ou obligatoires.

***

### 🌍 Polyvalence

Le mode **watch** peut être utilisé avec de nombreux **langages** et **frameworks** :

* Node.js / React ⚛️,
* Python 🐍,
* Go 🐹,
* Java ☕, etc.

👉 Les chemins et les règles varient d’un projet à l’autre, mais le **principe reste le même** :\
détecter un changement local ➝ appliquer l’action correspondante dans le conteneur.

***

✅ En résumé :

* `watch` = configuration par règles (path + action),
* actions possibles = 2 types principaux (avec options spécifiques),
* applicable à tous types de projets et stacks de développement.

### 📋 Prérequis

Pour fonctionner correctement, **`watch`** repose sur des exécutables standards.\
👉 Assurez-vous que l’image de votre service contienne :

* `stat`
* `mkdir`
* `rmdir`

⚠️ De plus, l’utilisateur du conteneur (**`USER`**) doit avoir les **droits d’écriture** ✍️ sur le chemin cible afin que `watch` puisse mettre à jour les fichiers.

***

### 📝 Bonnes pratiques Dockerfile

Un schéma courant consiste à copier le contenu initial dans le conteneur avec l’instruction **`COPY`**.\
➡️ Pour s’assurer que ces fichiers appartiennent bien à l’utilisateur configuré, utilisez l’option **`--chown`**.

#### Exemple Dockerfile

```dockerfile
# Exécution en tant qu’utilisateur non privilégié
FROM node:18
RUN useradd -ms /bin/sh -u 1001 app
USER app

# Installer les dépendances
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install

# Copier les fichiers sources dans le répertoire de l’application
COPY --chown=app:app . /app
```

👉 Ici, les fichiers copiés appartiennent à l’utilisateur `app` (UID `1001`), ce qui permet à **Compose Watch** de fonctionner correctement sans problèmes de permissions.

***

✅ En résumé :

* `watch` = liste de règles pour **déclencher des mises à jour automatiques**.
* Nécessite `stat`, `mkdir`, `rmdir` + permissions d’écriture utilisateur.
* Utilisez `COPY --chown` pour éviter les problèmes de droits ⚡.

## ⚙️ Actions dans **Compose Watch**

### 🔄 **1. Action : `sync`**

* Si l’action est définie sur **`sync`**, Compose s’assure que **toutes les modifications faites sur les fichiers de votre hôte** 🖥️ soient automatiquement **synchronisées** avec les fichiers correspondants dans le conteneur du service.
* ✅ Idéal pour les frameworks qui supportent le **Hot Reload** (ex. Node.js, React, Django, etc.).
* Plus généralement, `sync` peut remplacer les **bind mounts** dans de nombreux cas d’usage en développement.

***

### 🏗️ **2. Action : `rebuild`**

* Si l’action est définie sur **`rebuild`**, Compose reconstruit automatiquement une **nouvelle image avec BuildKit** ⚡ et **remplace le conteneur en cours d’exécution**.
* Ce comportement est équivalent à :

```bash
docker compose up --build <service>
```

* ✅ Idéal pour les **langages compilés** (Go, Java, Rust…) ou lorsque certaines modifications (ex. `package.json`) nécessitent une **reconstruction complète de l’image**.

***

### 🔄♻️ **3. Action : `sync+restart`**

* Si l’action est définie sur **`sync+restart`**, Compose :
  1. synchronise vos modifications,
  2. puis **redémarre le conteneur** concerné.
* ✅ Cas d’usage typique :
  * un **fichier de configuration** a changé (ex. `nginx.conf`, `database.yml`),
  * pas besoin de reconstruire l’image, mais juste de **relancer le processus principal**.

***

💡 **Astuce** : Optimisez vos `Dockerfile` pour permettre des rebuilds incrémentaux rapides 🚀 grâce au **caching des couches** et aux **multi-stage builds**.

***

## 📂 Autres attributs des règles

### 📌 `path` et `target`

* L’attribut **`target`** contrôle la façon dont le chemin local est **mappé dans le conteneur**.

Exemple : pour `path: ./app/html` et une modification sur `./app/html/index.html` :

```yaml
target: /app/html    ->  /app/html/index.html
target: /app/static  ->  /app/static/index.html
target: /assets      ->  /assets/index.html
```

***

### 🚫 `ignore`

* Les **patterns ignorés** sont relatifs au chemin (`path`) défini dans la règle `watch`, et **non au répertoire projet**.

👉 Exemple : si `path: ./web`, alors la règle `ignore` s’applique relativement à `./web`.

***

### 🗂️ `initial_sync`

* Lorsqu’on utilise une action de type **`sync+x`** (`sync+restart`, etc.),
* l’attribut **`initial_sync`** garantit que **les fichiers du chemin défini** sont bien **synchronisés avant de démarrer** une nouvelle session `watch`.

***

✅ En résumé :

* **`sync`** → hot reload pour le code source.
* **`rebuild`** → reconstruire l’image pour changements lourds.
* **`sync+restart`** → recharger le service quand seule une conf change.
* **`path`/`target`**, **`ignore`**, et **`initial_sync`** permettent de **contrôler précisément** ce qui est suivi, synchronisé ou ignoré.

## 📂 Exemples d’utilisation de **Compose Watch**

***

### 📝 Exemple 1 : Application **Node.js**

#### 📂 Structure du projet

```
myproject/
├── web/
│   ├── App.jsx
│   ├── index.js
│   └── node_modules/
├── Dockerfile
├── compose.yaml
└── package.json
```

#### 📑 `compose.yaml`

```yaml
services:
  web:
    build: .
    command: npm start
    develop:
      watch:
        - action: sync
          path: ./web
          target: /src/web
          initial_sync: true
          ignore:
            - node_modules/
        - action: rebuild
          path: package.json
```

***

#### ▶️ Explications

*   Quand vous exécutez :

    ```bash
    docker compose up --watch
    ```

    Compose lance un conteneur pour le service **`web`**, basé sur l’image construite depuis le `Dockerfile` à la racine du projet.
* La commande du service est **`npm start`**, qui démarre la version développement de l’application avec le **Hot Module Reload** activé dans l’outil de bundling (Webpack, Vite, Turbopack, etc.).
* Une fois le service démarré, le **mode watch** commence à surveiller les fichiers.

***

**🔄 Cas pratiques**

* Lorsqu’un fichier source du répertoire **`web/`** est modifié :
  * Compose **synchronise automatiquement** le fichier avec le conteneur → dans `/src/web`.
  * Exemple : `./web/App.jsx` est copié vers `/src/web/App.jsx`.
  * Le bundler détecte la modification et **met à jour l’application en direct sans redémarrer** le conteneur.
*   La règle **`ignore`** s’applique uniquement à :

    ```
    myproject/web/node_modules/
    ```

    et non à :

    ```
    myproject/node_modules/
    ```
* Quand le fichier **`package.json`** est modifié :
  * Compose reconstruit l’image et recrée le conteneur du service **`web`**, car l’ajout d’une dépendance ne peut pas être appliqué "à chaud".

***

#### 🌍 Autres langages

Ce modèle peut être appliqué à d’autres frameworks :

* **Python avec Flask** →
  * fichiers `.py` synchronisés en live,
  * mais une modification de `requirements.txt` déclenche un **rebuild complet**.

***

### 📝 Exemple 2 : Node.js + serveur Nginx + backend

#### 📑 `compose.yaml`

```yaml
services:
  web:
    build: .
    command: npm start
    develop:
      watch:
        - action: sync
          path: ./web
          target: /app/web
          ignore:
            - node_modules/
        - action: sync+restart
          path: ./proxy/nginx.conf
          target: /etc/nginx/conf.d/default.conf

  backend:
    build:
      context: backend
      target: builder
```

***

#### ▶️ Explications

Cet exemple illustre l’utilisation de l’action **`sync+restart`** :

* Les fichiers du répertoire **`web/`** sont synchronisés en live avec `/app/web`.
* Le fichier de configuration **`nginx.conf`** est surveillé :
  * lorsqu’il change, Compose le synchronise,
  * puis redémarre le service **web** pour appliquer la nouvelle configuration.

👉 Pendant ce temps, un service **backend** est construit via son propre contexte (`backend/`) et son stage `builder`.

***

#### ✅ Résultat

Cette configuration permet de développer et tester efficacement une application **Node.js avec frontend + backend** :

* les fichiers sources → **synchronisés à chaud**,
* les fichiers de configuration (comme `nginx.conf`) → **forcent un redémarrage contrôlé** du service,
* garantissant un cycle de développement **rapide et fluide** ⚡.

## 👀 Utiliser **watch** dans Docker Compose

### 🚀 Étapes d’utilisation

1. Ajoutez une section **`watch`** 📑 à un ou plusieurs services dans votre fichier **`compose.yaml`**.
2. Lancez votre projet avec :

```bash
docker compose up --watch
```

👉 Cela construit et démarre votre projet Compose, puis active le **mode surveillance des fichiers**.\
3\. Modifiez vos fichiers sources avec votre IDE ou éditeur préféré 🖋️ → Compose applique automatiquement les mises à jour aux services surveillés.

***

### 📝 Remarque

Vous pouvez aussi utiliser la commande dédiée :

```bash
docker compose watch
```

➡️ Utile si vous ne voulez pas mélanger :

* les **logs de l’application** 🐳
* avec les **logs de build** 🔨 et les **événements de synchronisation du système de fichiers** 📂.

***

### 💡 Astuce

👉 Découvrez des exemples concrets avec :

* **dockersamples/avatars** 🧑‍🎨
* ou la configuration locale des **Docker Docs** 📚

Ces projets démontrent parfaitement l’usage de **Compose Watch** en conditions réelles.

***

### 📚 Référence

👉 Voir la **Compose Develop Specification** pour plus de détails techniques.

***

✅ En résumé :

* Ajoutez `watch` → démarrez avec `docker compose up --watch`.
* Suivi automatique des modifications = workflow **fluide et rapide** ✨.
* Possibilité d’utiliser la commande séparée `docker compose watch` pour plus de clarté dans les logs.
