# 📛 Spécifier un nom de projet dans Docker Compose

👉 Par défaut, **Docker Compose** attribue un **nom de projet** basé sur le **nom du répertoire** qui contient ton fichier `docker-compose.yml`.

✅ Mais tu peux **surcharger ce nom** de plusieurs manières, selon ton besoin.

***

### 🔹 Pourquoi personnaliser le nom de projet ?

* **Éviter les conflits** : si tu exécutes plusieurs projets Compose avec les mêmes noms de services/réseaux/volumes.
* **Organiser les environnements** : ex. `myapp_dev`, `myapp_test`, `myapp_prod`.
* **Déployer plusieurs instances** de la même application côte à côte.

***

### 🔹 Méthodes pour définir un nom de projet

#### 1. Avec l’option CLI `-p`

```bash
docker compose -p customname up -d
```

➡️ Le projet portera le nom `customname`.

***

#### 2. Avec la variable d’environnement `COMPOSE_PROJECT_NAME`

```bash
export COMPOSE_PROJECT_NAME=myproject
docker compose up -d
```

***

#### 3. Avec le champ `name:` dans `docker-compose.yml`

```yaml
name: "myproject"

services:
  web:
    image: nginx
```

***

#### 4. Avec l’option `--project-directory`

```bash
docker compose --project-directory /path/to/dir up -d
```

➡️ Utilise ce dossier comme **base du projet**, ce qui influence le nom.

***

### 🔹 Ordre de priorité

Quand plusieurs méthodes sont utilisées, voici l’ordre de **précédence** (du plus fort au plus faible) :

1. **Option CLI `-p`**
2. **Variable d’environnement `COMPOSE_PROJECT_NAME`**
3. **Champ `name:` dans le `docker-compose.yml`**
4. **Nom du répertoire contenant le fichier** (valeur par défaut)

***

⚠️ **Note :**

* Le **répertoire du projet par défaut** est celui du fichier `docker-compose.yml`.
* Tu peux le changer avec `--project-directory`.

## 📌 Cas d’utilisation des noms de projet dans Docker Compose

👉 Docker Compose utilise le **nom de projet** pour **isoler les environnements** les uns des autres.\
Cela évite les conflits entre services, réseaux ou volumes qui portent le même nom.

Voici quelques **contextes où un nom de projet est particulièrement utile** :

***

### 🔹 1. Sur une machine de développement

➡️ **Créer plusieurs copies d’un même environnement**.

* Utile si tu veux lancer une version stable et une version expérimentale en parallèle.
* Exemple : une copie de l’application par **branche Git** (`feature/login`, `feature/api`, etc.).

👉 Chaque copie aura ses propres réseaux et volumes, donc elles ne s’interfèrent pas.

***

### 🔹 2. Sur un serveur CI/CD

➡️ **Éviter les interférences entre builds**.

* Chaque exécution de pipeline peut définir le **nom de projet** en fonction du numéro de build ou de commit.
* Exemple : `myapp_build_1023`.

👉 Ainsi, deux builds parallèles n’écrasent pas leurs conteneurs, réseaux ou volumes respectifs.

***

### 🔹 3. Sur une machine partagée ou un serveur de dev

➡️ **Isoler différents projets** qui utilisent des services avec les mêmes noms (`web`, `db`, `redis`, etc.).

* Par défaut, deux projets distincts mais sans nom personnalisé pourraient partager ou entrer en conflit sur leurs ressources.
* Exemple :
  * Projet A → réseau `projA_default`, conteneur `projA_web`
  * Projet B → réseau `projB_default`, conteneur `projB_web`

👉 Cela garantit que **chaque projet reste indépendant**.

***

⚡ En résumé : le **nom de projet est une clé d’isolation**.\
Il permet d’avoir plusieurs environnements Compose **sans collisions** entre conteneurs, volumes et réseaux.

## ⚙️ Définir un nom de projet dans Docker Compose

Un **nom de projet** doit respecter certaines règles :

* uniquement **lettres minuscules**, **chiffres**, **tirets (-)** et **underscores (\_)**
* doit **commencer par une lettre minuscule ou un chiffre**

👉 Si le nom du répertoire ou du projet ne respecte pas ces contraintes, il existe d’autres mécanismes pour définir un nom personnalisé.

***

### 📌 Ordre de priorité (du plus fort au plus faible)

Quand plusieurs méthodes sont utilisées pour définir le nom de projet, **Docker Compose applique la plus prioritaire** :

1.  **L’option `-p` en ligne de commande**

    ```bash
    docker compose -p monprojet up -d
    ```

    → Le projet sera nommé `monprojet` même si ton dossier s’appelle autrement.

***

2.  **La variable d’environnement `COMPOSE_PROJECT_NAME`**

    ```bash
    export COMPOSE_PROJECT_NAME=monprojet
    docker compose up -d
    ```

    → Pratique pour CI/CD ou pour éviter de répéter `-p` à chaque fois.

***

3.  **L’attribut `name:` dans ton `docker-compose.yml`** (au niveau racine)

    ```yaml
    name: monprojet
    services:
      web:
        image: nginx
    ```

    → ⚠️ Si tu combines plusieurs fichiers (`-f`), c’est le **dernier fichier** qui peut définir le `name:` utilisé.

***

4. **Le nom du répertoire qui contient ton `docker-compose.yml`**\
   → Par défaut, si tu n’as rien défini, le **nom du dossier** devient le nom du projet.\
   Exemple : si ton fichier est dans `/home/user/monapp/`, alors le projet sera `monapp`.

***

5. **Le nom du répertoire courant si aucun `docker-compose.yml` n’est spécifié**\
   → Si tu lances la commande dans un dossier sans préciser de fichier Compose, Docker prend le **nom du dossier courant** comme nom de projet.

***

✅ En résumé :

* Besoin d’un **nom temporaire** → utilise `-p`.
* Besoin d’un **nom global pour tout le projet** → mets `name:` dans ton `docker-compose.yml`.
* En **CI/CD** ou script → préfère `COMPOSE_PROJECT_NAME`.
