# 🔹 Qu’est-ce que Docker Engine ?

Docker Engine est le **moteur open source** qui permet de créer et gérer des **conteneurs**.\
Il fonctionne selon une **architecture client-serveur** composée de trois éléments principaux :

1. **dockerd (le démon Docker)**
   * Processus serveur qui tourne en arrière-plan.
   * Responsable de la création et gestion des objets Docker (images, conteneurs, réseaux, volumes).
2. **API Docker**
   * Interface qui permet aux programmes de communiquer avec le démon.
   * Peut être utilisée par des outils tiers ou directement via des scripts.
3. **CLI Docker (`docker`)**
   * Le client en ligne de commande que tu utilises tous les jours.
   * Exemple : `docker run`, `docker ps`, `docker build`.
   * Ces commandes envoient des requêtes à `dockerd` via l’API.

📌 En résumé : **`docker` (CLI) → API → `dockerd` (daemon) → objets Docker (images, conteneurs, réseaux, volumes).**

***

## 🔹 Principales fonctionnalités liées à Docker Engine

1. **Installation**
   * Tu peux installer Docker Engine sur Linux (Ubuntu, Debian, CentOS, Fedora, etc.).
   * Sur Windows/Mac, tu utilises généralement **Docker Desktop**, qui embarque Docker Engine.
2. **Stockage (Volumes et Bind Mounts)**
   * Permet de **garder les données persistantes** même si le conteneur est supprimé.
   * Exemple : une base de données PostgreSQL qui garde ses données après un `docker rm`.
3. **Réseaux**
   * Gestion de la communication entre conteneurs.
   * Types : `bridge` (par défaut), `host`, `overlay`, `none`.
4. **Logs de conteneurs**
   *   Tu peux consulter les journaux d’un conteneur avec :

       ```bash
       docker logs <container_id>
       ```
5. **Prune (nettoyage)**
   * Supprime les ressources non utilisées pour libérer de l’espace.
   *   Exemple :

       ```bash
       docker system prune -a
       ```

       (⚠️ Supprime toutes les images non utilisées, attention en prod).
6. **Configurer le démon**
   * Personnaliser `dockerd` via le fichier `daemon.json`.
   * Exemple : changer l’adresse du registre miroir, activer l’API distante, etc.
7. **Rootless mode**
   * Permet de lancer Docker **sans droits root** → plus sécurisé.
   * Utile dans des environnements multi-utilisateurs.
8. **Fonctionnalités dépréciées**
   * Docker retire progressivement certaines options ou syntaxes obsolètes.
   * Toujours consulter la documentation avant d’utiliser de vieilles commandes.
9. **Release notes**
   * Permet de suivre les nouveautés et changements à chaque version.

***

## 🔹 Exemple d’utilisation basique

Créer un conteneur Nginx avec Docker Engine :

```bash
docker run -d -p 8080:80 nginx
```

* `docker` = client
* `run` = instruction
* `-d` = mode détaché
* `-p` = mapping port 8080 → 80
* `nginx` = image demandée à Docker Hub

👉 Le CLI envoie la commande à **dockerd** qui télécharge l’image, crée le conteneur, et le lance.

### Fonctionnalités principales

* **Stockage (Storage)**\
  Tu peux persister les données des conteneurs avec des **volumes** ou des **bind mounts**, pour que les données survivent à l’arrêt/suppression des conteneurs.
* **Réseau (Networking)**\
  Docker crée des **réseaux virtuels** pour permettre aux conteneurs de communiquer entre eux ou avec l’extérieur (bridge, host, overlay, etc.).
*   **Logs (Container logs)**\
    Chaque conteneur génère des logs accessibles via :

    ```bash
    docker logs <container_id>
    ```
*   **Nettoyage (Prune)**\
    Supprimer les ressources inutilisées (conteneurs stoppés, images non utilisées, volumes orphelins) :

    ```bash
    docker system prune
    ```
* **Configurer le démon (`dockerd`)**\
  Tu peux personnaliser son comportement via `/etc/docker/daemon.json` (exemple : drivers de stockage, options réseau, registry miroir…).
* **Mode rootless**\
  Permet d’exécuter Docker **sans privilèges root**, augmentant la sécurité.

***

### Cycle de vie des objets Docker

Docker Engine gère plusieurs types d’objets :

* **Images** → modèles immuables (ex. `python:3.10-alpine`)
* **Conteneurs** → instances en cours d’exécution créées à partir des images
* **Volumes** → stockage persistant
* **Réseaux** → communication entre conteneurs

***

### Points importants

* **Docker Engine est le cœur** de toute la plateforme Docker.
* **Compose, Swarm, Kubernetes** reposent dessus pour orchestrer les conteneurs.
* Il est **open source** et peut être utilisé sans Docker Desktop.
