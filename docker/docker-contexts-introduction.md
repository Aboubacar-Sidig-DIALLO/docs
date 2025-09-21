# 🐳 Docker Contexts – Introduction

### 🔹 Qu’est-ce qu’un _Docker context_ ?

Un **contexte Docker** est un ensemble de paramètres qui permettent à ton **client Docker (`docker CLI`)** de savoir **quel démon Docker** (daemon) il doit utiliser et comment s’y connecter.

👉 Cela permet de gérer **plusieurs environnements** (local, distant, cloud, Swarm, etc.) **avec un seul client Docker**.

***

### 🔹 Exemple typique

Un même poste développeur peut avoir :

* un **contexte par défaut** (local, avec Docker Engine installé sur la machine).
* un **contexte distant** (un démon Docker exposé sur un autre serveur, ou dans un cluster Swarm/Kubernetes).

***

### 🔹 Commandes de base

1.  Vérifier si ton client supporte les contextes :

    ```bash
    docker context
    ```
2.  Voir les contextes disponibles :

    ```bash
    docker context ls
    ```
3.  Utiliser un contexte spécifique :

    ```bash
    docker context use <nom-du-context>
    ```

***

### 🔹 Cas d’usage

* Basculer rapidement entre **Docker local** et un **daemon distant**.
* Gérer un **serveur partagé** depuis ton poste de dev.
* Faciliter l’utilisation de **plusieurs environnements** (développement, staging, production).

***

⚡️ En résumé : les **Docker contexts** simplifient la vie quand tu dois jongler entre plusieurs démons Docker depuis un seul client CLI.

## 🐳 Anatomie d’un contexte Docker

Un **contexte Docker** est un ensemble de paramètres qui définissent **comment le client Docker (`docker CLI`) se connecte au démon Docker (`dockerd`)**.

***

### 🔹 Un contexte contient :

1. **Nom et description**
   * Exemple : `default` est créé automatiquement à l’installation.
   * Tu peux ajouter une description personnalisée.
2. **Configuration de l’endpoint (point de connexion)**
   * Indique _où_ se trouve le démon Docker.
   * Exemples :
     * Socket Unix local → `unix:///var/run/docker.sock`
     * TCP distant → `tcp://192.168.1.100:2376`
3. **Informations TLS**
   * Contient les certificats si tu sécurises ta connexion vers un démon distant avec TLS.

***

### 🔹 Lister les contextes

```bash
docker context ls
```

Exemple de sortie :

```
NAME        DESCRIPTION                               DOCKER ENDPOINT               ERROR
default *                                             unix:///var/run/docker.sock
```

* Ici, seul le contexte `default` existe.
* Le `*` indique que c’est le **contexte actif** : toutes les commandes `docker` pointent vers lui par défaut.

***

### 🔹 Inspecter un contexte

```bash
docker context inspect default
```

Exemple :

```json
[
  {
    "Name": "default",
    "Metadata": {},
    "Endpoints": {
      "docker": {
        "Host": "unix:///var/run/docker.sock",
        "SkipTLSVerify": false
      }
    },
    "TLSMaterial": {},
    "Storage": {
      "MetadataPath": "<IN MEMORY>",
      "TLSPath": "<IN MEMORY>"
    }
  }
]
```

👉 Explications :

* `"Host": "unix:///var/run/docker.sock"` → connexion locale via le socket Unix.
* `"SkipTLSVerify": false` → vérification TLS désactivée (inutile en local).
* `"TLSMaterial"` → contient les certificats si TLS est configuré.
* `"Storage"` → emplacement où sont stockées les infos du contexte (ici en mémoire).

***

⚡️ **En résumé :**\
Un **contexte Docker** regroupe le **nom, le point de connexion (Unix/TCP), et les infos TLS** nécessaires pour que ton `docker CLI` sache **où et comment envoyer ses commandes**.

## 🐳 Créer un nouveau contexte Docker

Tu peux créer de nouveaux contextes avec la commande :

```bash
docker context create
```

***

### 🔹 Exemple : créer un contexte `docker-test`

Ici, on définit un endpoint TCP vers un démon Docker distant :

```bash
docker context create docker-test --docker host=tcp://docker:2375
```

Sortie attendue :

```
docker-test
Successfully created context "docker-test"
```

***

### 🔹 Où Docker stocke le contexte ?

Chaque nouveau contexte est enregistré dans un fichier `meta.json` situé dans un sous-dossier dédié de :

```
~/.docker/contexts/
```

👉 Cela permet de gérer plusieurs contextes indépendamment.

***

### 🔹 Vérifier les contextes existants

```bash
docker context ls
```

Exemple de sortie :

```
NAME          DESCRIPTION                             DOCKER ENDPOINT               ERROR
default *                                             unix:///var/run/docker.sock
docker-test                                           tcp://docker:2375
```

* `default *` → contexte actif (local, via le socket Unix).
* `docker-test` → ton nouveau contexte, pointant vers `tcp://docker:2375`.

***

### 🔹 Inspecter un contexte

```bash
docker context inspect docker-test
```

Tu y verras les détails : host TCP, TLS activé ou non, chemin de stockage, etc.

***

👉 En résumé :

* `docker context create` = créer un nouveau contexte.
* Stockage = `~/.docker/contexts/`.
* `docker context ls` = lister les contextes.
* `docker context inspect` = voir les détails d’un contexte.

## 🐳 Utiliser un autre contexte Docker

Tu peux basculer entre différents contextes avec la commande :

```bash
docker context use <nom-du-contexte>
```

***

### 🔹 Exemple : utiliser le contexte `docker-test`

```bash
docker context use docker-test
```

Sortie attendue :

```
docker-test
Current context is now "docker-test"
```

***

### 🔹 Vérifier le contexte actif

```bash
docker context ls
```

Exemple de sortie :

```
NAME            DESCRIPTION                           DOCKER ENDPOINT               ERROR
default                                               unix:///var/run/docker.sock
docker-test *                                         tcp://docker:2375
```

👉 L’astérisque (`*`) indique quel contexte est actif. Ici, c’est `docker-test`.

***

### 🔹 Utiliser une variable d’environnement

Tu peux aussi définir le contexte via la variable `DOCKER_CONTEXT`, qui **prend le dessus** sur `docker context use`.

#### Sous **PowerShell** :

```powershell
$env:DOCKER_CONTEXT='docker-test'
```

#### Sous **Bash** :

```bash
export DOCKER_CONTEXT=docker-test
```

Puis vérifie :

```bash
docker context ls
```

***

### 🔹 Utiliser le flag `--context` (temporaire)

Tu peux aussi spécifier un contexte uniquement pour une commande, sans changer le contexte global :

```bash
docker --context production container ls
```

Ici, la commande cible le contexte `production`, mais le contexte actif global reste inchangé.

***

👉 Résumé :

* `docker context use <nom>` → change le contexte par défaut.
* `DOCKER_CONTEXT` → variable d’env qui écrase le choix précédent.
* `--context` → pour changer le contexte juste sur **une commande**.

## 🐳 Exporter, importer et mettre à jour des contextes Docker

Les contextes Docker peuvent être **partagés entre machines** en les exportant et les important.

***

### 🔹 Exporter un contexte

La commande :

```bash
docker context export <nom-du-contexte>
```

#### Exemple :

```bash
docker context export docker-test
```

👉 Cela génère un fichier `docker-test.dockercontext`.

Vérifier le contenu :

```bash
cat docker-test.dockercontext
```

***

### 🔹 Importer un contexte

Sur un autre hôte (ou la même machine), tu peux importer le fichier exporté :

```bash
docker context import <nom-du-contexte> <fichier>
```

#### Exemple :

```bash
docker context import docker-test docker-test.dockercontext
```

Sortie attendue :

```
docker-test
Successfully imported context "docker-test"
```

Vérifie avec :

```bash
docker context ls
```

***

### 🔹 Mettre à jour un contexte

Tu peux modifier certaines informations d’un contexte existant avec `docker context update`.

#### Exemple : ajouter une description

```bash
docker context update docker-test --description "Test context"
```

Sortie attendue :

```
docker-test
Successfully updated context "docker-test"
```

***

✅ **Résumé rapide :**

* `docker context export` → sauvegarde un contexte dans un fichier.
* `docker context import` → recrée un contexte depuis un fichier.
* `docker context update` → modifie un contexte existant.
