# 🔗 Liens de conteneurs hérités (Legacy container links)

### ⚠️ Avertissement

L’option `--link` est une **fonctionnalité héritée** de Docker.\
➡️ Elle pourrait être supprimée dans le futur.

👉 Il est fortement recommandé d’utiliser les **réseaux définis par l’utilisateur** (**user-defined networks**) plutôt que `--link`.

* Les réseaux définis par l’utilisateur permettent la communication entre conteneurs sans configuration compliquée.
* La seule fonctionnalité que `--link` supporte encore et qui n’existe pas dans les réseaux définis par l’utilisateur est le **partage de variables d’environnement** entre conteneurs.
* Pour cela, tu peux utiliser des alternatives comme :
  * des **volumes** pour partager un fichier de configuration,
  * ou **Docker Compose** avec des variables définies.

👉 Voir aussi **Différences entre ponts définis par l’utilisateur et pont par défaut**.

***

### 🌉 Contexte

Avant l’introduction des **réseaux Docker**, on utilisait `--link` pour :

* permettre aux conteneurs de se découvrir entre eux,
* transférer de manière sécurisée des informations d’un conteneur à un autre.

Aujourd’hui :

* `--link` fonctionne encore, mais son comportement diffère selon qu’on utilise :
  * le **réseau bridge par défaut** (créé automatiquement avec Docker),
  * ou un **réseau défini par l’utilisateur**.

***

### 🔌 Connexion via le **mappage de ports**

Exemple : tu lances une appli Python Flask dans un conteneur :

```bash
docker run -d -P training/webapp python app.py
```

* L’option `-P` publie automatiquement les ports du conteneur vers un **port aléatoire haut** de l’hôte.
* Exemple avec `docker ps` :

```bash
docker ps nostalgic_morse
```

```
CONTAINER ID  IMAGE                   COMMAND       PORTS
bc533791f3f5  training/webapp:latest  python app.py 0.0.0.0:49155->5000/tcp
```

➡️ Ici, le port **5000 (conteneur)** est lié au port **49155 (hôte)**.

***

#### Spécifier un port fixe

```bash
docker run -d -p 80:5000 training/webapp python app.py
```

👉 Cela lie le **port 80 de l’hôte** au **port 5000 du conteneur**.\
⚠️ Mais cela limite l’utilisation (un seul conteneur peut utiliser ce port).

***

#### Spécifier une **plage de ports**

```bash
docker run -d -p 8000-9000:5000 training/webapp python app.py
```

➡️ Le port 5000 du conteneur sera lié à un port disponible entre **8000 et 9000**.

***

#### Restreindre à une interface spécifique

```bash
docker run -d -p 127.0.0.1:80:5000 training/webapp python app.py
```

➡️ Accessible seulement depuis `localhost` (pas depuis l’extérieur).

***

#### Lier avec protocole UDP ou SCTP

```bash
docker run -d -p 127.0.0.1:80:5000/udp training/webapp python app.py
```

***

#### Vérifier les ports publiés

```bash
docker port nostalgic_morse 5000
```

Exemple de sortie :

```
127.0.0.1:49155
```

***

### 🔗 Connexion avec le système de liens (`--link`)

⚠️ Partie **héritée**. Fonctionne uniquement avec le **réseau bridge par défaut**.

Les liens permettent à un conteneur de :

* recevoir des infos d’un autre conteneur (IP, variables, etc.),
* utiliser directement son **nom** pour le joindre.

***

#### 📛 Importance des noms

* Chaque conteneur peut avoir un **nom** (`--name`).
* Docker s’en sert comme référence dans les liens.

Exemple :

```bash
docker run -d -P --name web training/webapp python app.py
```

```bash
docker ps -l
```

```
CONTAINER ID  IMAGE                  COMMAND        PORTS                  NAMES
aed84ee21bde  training/webapp:latest python app.py  0.0.0.0:49154->5000/tcp  web
```

➡️ Le conteneur est nommé **web**.

⚠️ Les noms doivent être **uniques**.

*   Si tu veux réutiliser le même nom, supprime l’ancien conteneur :

    ```bash
    docker rm web
    ```
* Ou lance avec `--rm` pour supprimer automatiquement à l’arrêt.

***

✅ **Résumé :**

* `--link` = ancien mécanisme, **déconseillé**.
* Préfère les **réseaux personnalisés (user-defined networks)**.
* Le mappage de ports `-p` reste valable pour exposer un service au monde extérieur.

## 🔗 Communication entre conteneurs via `--link` (héritée)

Les **liens Docker (`--link`)** permettent à un conteneur :

* de **découvrir** un autre conteneur,
* de **recevoir des informations** (IP, variables d’environnement, alias),
* sans avoir besoin d’exposer les ports du conteneur source au réseau externe.

C’est une **ancienne méthode** (avant les réseaux définis par l’utilisateur).

***

### 🛠 Exemple pratique : `web` ↔ `db`

#### 1. Créer un conteneur base de données

```bash
docker run -d --name db training/postgres
```

➡️ Conteneur `db` lancé avec une base PostgreSQL.

***

#### 2. Supprimer l’ancien conteneur web

```bash
docker container rm -f web
```

***

#### 3. Créer un conteneur web lié au `db`

```bash
docker run -d -P --name web --link db:db training/webapp python app.py
```

* `--link db:db` → le conteneur **web** est lié au conteneur **db**.
*   Syntaxe :

    ```
    --link <nom|id>:alias
    ```

    * `db` = conteneur source
    * `alias` = nom que verra le conteneur cible

⚠️ Si tu omets `:alias`, l’alias = nom du conteneur (`db`).

***

#### 4. Vérifier le lien

```bash
docker inspect -f "{{ .HostConfig.Links }}" web
```

Résultat :

```
[/db:/web/db]
```

👉 Le conteneur **web** est bien lié au conteneur **db**.

***

### 📦 Que fait réellement `--link` ?

Quand un conteneur est lié à un autre, Docker fournit au conteneur cible (ici `web`) des infos sur le conteneur source (ici `db`) par deux mécanismes :

1. **Variables d’environnement**
2. **Mise à jour du fichier `/etc/hosts`**

***

### 🌍 1. Variables d’environnement générées

Docker crée automatiquement des variables d’environnement dans le conteneur lié.

Exemple :

```bash
docker run --rm --name web2 --link db:db training/webapp env
```

Sortie :

```
DB_NAME=/web2/db
DB_PORT=tcp://172.17.0.5:5432
DB_PORT_5432_TCP=tcp://172.17.0.5:5432
DB_PORT_5432_TCP_PROTO=tcp
DB_PORT_5432_TCP_PORT=5432
DB_PORT_5432_TCP_ADDR=172.17.0.5
```

👉 Explications :

* `DB_NAME` → chemin du lien (`/web2/db`)
* `DB_PORT` → URL de connexion (tcp://IP:PORT)
* `DB_PORT_5432_TCP_ADDR` → IP du conteneur `db`
* `DB_PORT_5432_TCP_PORT` → port exposé (5432)

⚠️ **Attention :**

* Toutes les variables d’environnement du conteneur source (`ENV` du Dockerfile, `-e`, etc.) sont exposées au conteneur cible.
* Cela peut représenter un **risque de sécurité** si elles contiennent des données sensibles.

⚠️ Autre limite :

* Si le conteneur source redémarre, **les IP dans les variables d’environnement ne sont pas mises à jour**.
* C’est pourquoi il vaut mieux utiliser `/etc/hosts`.

***

### 📒 2. Mise à jour automatique de `/etc/hosts`

Docker ajoute des entrées dans le fichier `/etc/hosts` du conteneur lié.

Exemple :

```bash
docker run -t -i --rm --link db:webdb training/webapp /bin/bash
cat /etc/hosts
```

Sortie :

```
172.17.0.7  aed84ee21bde
...
172.17.0.5  webdb 6e5cdeb2d300 db
```

👉 Explications :

* `webdb` = alias défini dans `--link db:webdb`
* `db` = nom original du conteneur source
* `6e5cdeb2d300` = hostname interne

Tu peux tester la connectivité avec :

```bash
ping webdb
```

***

### 🔄 Gestion lors d’un redémarrage

Si le conteneur `db` est redémarré :

```bash
docker restart db
```

➡️ L’IP change.\
➡️ Docker met automatiquement à jour `/etc/hosts` dans les conteneurs liés (`web`).

***

### ✅ Résumé

* `--link` crée une **liaison privée** entre deux conteneurs.
* Le conteneur cible reçoit :
  * des **variables d’environnement** (⚠️ non mises à jour après redémarrage),
  * des **entrées /etc/hosts** (✅ mises à jour automatiquement).
* Plus besoin d’exposer le conteneur source au réseau (`-p`).
* ⚠️ Mais : `--link` est une **fonctionnalité obsolète**, remplacée par les **réseaux définis par l’utilisateur**.
