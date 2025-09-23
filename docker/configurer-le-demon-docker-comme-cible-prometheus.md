# ⚙️ Configurer le démon Docker comme cible Prometheus

Pour que **Prometheus** puisse collecter des métriques de Docker, il faut configurer le démon Docker (`dockerd`) afin qu’il expose une **adresse de métriques**.

***

### 📍 Emplacement du fichier `daemon.json`

Selon ton système, le fichier de configuration du démon Docker se trouve par défaut à :

* **Linux** : `/etc/docker/daemon.json`
* **Windows Server** : `C:\ProgramData\docker\config\daemon.json`
* **Docker Desktop** : via l’UI → **Settings > Docker Engine**

👉 Si le fichier n’existe pas, crée-le.

***

### 📝 Exemple de configuration

Ajoute ceci dans `daemon.json` :

```json
{
  "metrics-addr": "127.0.0.1:9323"
}
```

***

### 🔄 Appliquer les changements

* **Linux** :

```bash
sudo systemctl restart docker
```

* **Docker Desktop (Mac/Windows)** :\
  Clique sur **Apply & Restart** après avoir modifié la configuration dans les **settings**.

***

### 🔎 Vérification

Docker expose désormais les métriques **compatibles Prometheus** sur :

```
http://127.0.0.1:9323/metrics
```

💡 Tu peux aussi configurer l’adresse sur `0.0.0.0:9323` pour que l’endpoint soit accessible depuis d’autres machines.\
⚠️ **Attention** : Cela expose les métriques au réseau → à utiliser uniquement si c’est sécurisé dans ton environnement.

## 📊 Créer une configuration Prometheus avec Docker comme cible

Voici un exemple complet de fichier `prometheus.yml` que tu peux enregistrer (par ex. dans `/tmp/prometheus.yml`).\
Il s’agit d’une configuration **standard de Prometheus**, avec l’ajout d’un **job pour Docker** afin de collecter ses métriques exposées sur le port **9323**.

***

### 📝 Exemple : `prometheus.yml`

```yaml
# 🌍 Configuration globale
global:
  scrape_interval: 15s          # Intervalle de collecte (par défaut 1 min)
  evaluation_interval: 15s      # Intervalle d’évaluation des règles
  # scrape_timeout reste à 10s par défaut

  # Labels ajoutés à toutes les séries temporelles / alertes
  external_labels:
    monitor: "codelab-monitor"

# 📌 Chargement des règles (optionnel)
rule_files:
  # - "first.rules"
  # - "second.rules"

# 📡 Configuration des jobs de collecte
scrape_configs:
  # Prometheus lui-même
  - job_name: prometheus
    static_configs:
      - targets: ["localhost:9090"]

  # 🚢 Docker daemon (métriques exposées via daemon.json)
  - job_name: docker
    static_configs:
      - targets: ["host.docker.internal:9323"]
```

***

### 🚀 Utilisation

1.  **Sauvegarde** le fichier :

    ```bash
    nano /tmp/prometheus.yml
    ```

    (ou un autre chemin de ton choix)
2.  **Lancer Prometheus** avec cette configuration :

    ```bash
    docker run -d \
      --name prometheus \
      -p 9090:9090 \
      -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml \
      prom/prometheus
    ```
3. Ouvre **l’UI Prometheus** dans ton navigateur :\
   👉 [http://localhost:9090/graph](http://localhost:9090/graph)
4.  Teste une requête, par exemple :

    ```
    engine_daemon_engine_info
    ```

    ou

    ```
    dockerd_builds_total
    ```

***

⚠️ Petite remarque :

* Sur **Linux**, si tu exécutes Prometheus directement sur la machine hôte, utilise `127.0.0.1:9323` comme target.
* Sur **Docker Desktop (Mac/Windows)**, il faut garder `host.docker.internal:9323`.

## 🚀 Lancer Prometheus dans un conteneur avec Docker

Une fois ton fichier **`prometheus.yml`** prêt, tu peux démarrer un conteneur Prometheus configuré pour collecter les métriques Docker.

***

### 📝 Commande Docker

```bash
docker run --name my-prometheus \
  --mount type=bind,source=/tmp/prometheus.yml,destination=/etc/prometheus/prometheus.yml \
  -p 9090:9090 \
  --add-host host.docker.internal=host-gateway \
  prom/prometheus
```

***

### ⚙️ Explications des options

* **`--name my-prometheus`** → Nom du conteneur.
* **`--mount type=bind,...`** → Monte ton fichier local `prometheus.yml` à l’intérieur du conteneur (Prometheus lira sa config depuis `/etc/prometheus/prometheus.yml`).
* **`-p 9090:9090`** → Expose l’interface web de Prometheus sur ton hôte à l’adresse [http://localhost:9090](http://localhost:9090).
* **`--add-host host.docker.internal=host-gateway`** → Ajoute la résolution de `host.docker.internal` vers l’IP de la machine hôte (utile sur Linux).\
  👉 Sur **Docker Desktop (Mac/Windows)**, cette option est déjà gérée automatiquement.

***

### ✅ Vérification

1. Lance la commande ci-dessus.
2. Ouvre ton navigateur et va sur :\
   👉 [http://localhost:9090](http://localhost:9090)
3.  Dans l’onglet **Graph**, exécute une requête comme :

    ```
    engine_daemon_engine_info
    ```

    pour voir les infos exposées par le démon Docker.

## 📊 Ouvrir le tableau de bord Prometheus et vérifier la cible Docker

Une fois ton conteneur Prometheus lancé :

***

### 🔎 Étape 1 : Accéder au tableau de bord

👉 Ouvre ton navigateur et va sur :

```
http://localhost:9090/targets/
```

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

### 🔎 Étape 2 : Vérifier la présence de la cible Docker

Sur cette page, tu dois voir une section :

* **Job : `docker`**
* **Endpoint : `host.docker.internal:9323/metrics`**
* **Status : `UP`** (si tout est bien configuré)

Cela confirme que Prometheus scrute correctement les métriques exposées par le démon Docker.

***

⚠️ **Attention :**\
Si tu utilises **Docker Desktop (Mac ou Windows)**, tu ne peux pas ouvrir directement les URLs d’endpoints listées dans cette page (`host.docker.internal:9323/metrics`). Ces liens sont destinés à Prometheus lui-même et non à un accès direct via ton navigateur.

## 📈 Utiliser Prometheus pour visualiser les métriques Docker

Une fois Prometheus démarré et accessible à [http://localhost:9090](http://localhost:9090), tu peux commencer à créer des **graphiques en temps réel** pour explorer les métriques Docker.

***

### 🔎 Étape 1 : Ouvrir l’onglet **Graph**

Dans l’UI Prometheus :

👉 Clique sur **Graph** (dans le menu du haut).\
👉 Dans la barre de recherche à côté du bouton **Execute**, choisis une métrique (par ex. `engine_daemon_network_actions_seconds_count`).

Exemple d’interface :

<figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

### 🔎 Étape 2 : Exécuter une requête

* Clique sur **Execute** pour afficher les données brutes.
* Clique ensuite sur **Graph** pour visualiser l’évolution dans le temps.

👉 Si ton Docker est **au repos**, tu verras une ligne plate, car il n’y a pas beaucoup d’activité.

***

### 🔎 Étape 3 : Générer de l’activité réseau Docker

Pour rendre le graphique plus intéressant, lance un conteneur qui effectue des actions réseau (téléchargements via un gestionnaire de paquets) :

```bash
docker run --rm alpine apk add git make musl-dev go
```

Ce conteneur télécharge et installe plusieurs paquets, ce qui génère du trafic réseau mesurable.

***

### 🔎 Étape 4 : Actualiser le graphique

Attends quelques secondes (⏳ l’intervalle de scrapping par défaut est de **15s**) puis recharge le graphique.

👉 Tu verras une **hausse nette** dans les métriques liées au réseau, reflétant l’activité du conteneur que tu viens de lancer.

Exemple attendu :

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

