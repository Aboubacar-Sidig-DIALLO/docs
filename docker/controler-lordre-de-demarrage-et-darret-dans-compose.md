---
description: >-
  📋 Options de la page  Vous pouvez contrôler l’ordre de démarrage et d’arrêt
  des services grâce à l’attribut depends_on.
---

# 🕹️ Contrôler l’ordre de démarrage et d’arrêt dans Compose

👉 Compose démarre et arrête toujours les conteneurs **selon leur ordre de dépendance**.\
Les dépendances peuvent être définies par :

* **`depends_on`**
* **`links`**
* **`volumes_from`**
* **`network_mode: "service:..."`**

***

### ⚠️ Problème classique sans gestion de l’ordre

Imaginons que votre application ait besoin d’accéder à une **base de données**.

*   Vous lancez **les deux services** avec :

    ```bash
    docker compose up
    ```
* Il existe un risque que l’application démarre **avant que la base de données soit prête** 🕑.
* Résultat ❌ : l’application tente d’envoyer des requêtes SQL, mais la base n’est pas encore en état de les traiter.

***

👉 C’est exactement pour éviter ce type de problème que **Compose permet de gérer l’ordre de démarrage** via `depends_on`.

## 🟢 Contrôler le démarrage des services

Au démarrage, **Compose n’attend pas** qu’un conteneur soit complètement **“prêt”** ✅.\
👉 Il attend seulement qu’il soit **en cours d’exécution**.

⚠️ Cela peut poser problème, par exemple avec une **base de données relationnelle** : celle-ci doit d’abord lancer ses propres services internes avant de pouvoir accepter des connexions entrantes.

***

### 🛠️ La solution : `condition`

Pour gérer l’état de disponibilité (_ready state_) d’un service, on utilise l’attribut **`condition`** avec l’une des options suivantes :

* **`service_started`** 🟢\
  ➝ le service dépendant démarre dès que le service requis est lancé (mais pas forcément prêt).
* **`service_healthy`** 💊\
  ➝ le service dépendant démarre seulement lorsque le service requis est **“healthy”** (c’est-à-dire déclaré sain via un **`healthcheck`**).
* **`service_completed_successfully`** ✅\
  ➝ le service dépendant démarre seulement une fois que le service requis a terminé son exécution avec succès.

***

### 📝 Exemple YAML

```yaml
services:
  web:
    build: .
    depends_on:
      db:
        condition: service_healthy
        restart: true
      redis:
        condition: service_started

  redis:
    image: redis

  db:
    image: postgres
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      retries: 5
      start_period: 30s
      timeout: 10s
```

***

### 📖 Explication

* Compose **crée les services dans l’ordre des dépendances** :\
  d’abord **`db`** et **`redis`**, puis **`web`**.
* Le service **`db`** doit être **“healthy”** (test réussi via `healthcheck`) avant que **`web`** ne démarre.
* Le service **`redis`** n’a qu’un `service_started`, donc il suffit qu’il soit en cours d’exécution.
* L’option **`restart: true`** assure que si **`db`** est redémarré (par ex. via `docker compose restart`), alors **`web`** sera aussi automatiquement redémarré 🔄 afin de rétablir correctement ses connexions.
*   Le **`healthcheck`** de **`db`** utilise la commande :

    ```bash
    pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}
    ```

    ➝ Cela permet de vérifier que PostgreSQL est bien prêt.\
    Le test est répété toutes les **10 secondes**, jusqu’à **5 tentatives maximum**, avec une période initiale d’attente de **30s**.
* Enfin, Compose supprime aussi les services **dans l’ordre inverse des dépendances** :\
  `web` est supprimé avant `db` et `redis`.

***

👉 Grâce à ce mécanisme, vous pouvez garantir que vos services démarrent et s’arrêtent dans le bon ordre, sans risquer de rencontrer des erreurs liées à des dépendances non prêtes.
