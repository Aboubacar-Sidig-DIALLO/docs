# 📊 Collecter des métriques Docker avec Prometheus

👉 **Prometheus** est un outil open-source de **monitoring et d’alerting** très utilisé dans les environnements Cloud et DevOps.\
Tu peux configurer **Docker** comme une **cible Prometheus** pour collecter des métriques sur le démon Docker.

⚠️ **Attention** :

* Les **métriques disponibles** et leurs **noms** sont encore en **développement actif** → ils peuvent changer.
* Actuellement, Prometheus permet seulement de **surveiller Docker lui-même** (daemon, containers, ressources).\
  → Pas directement ton **application interne** (pour cela, il faut exporter toi-même des métriques).

***

### 🔹 Étape 1 : Activer les métriques dans Docker

Ajoute cette configuration au fichier `/etc/docker/daemon.json` :

```json
{
  "metrics-addr": "127.0.0.1:9323",
  "experimental": true
}
```

Ici :

* `metrics-addr` définit l’adresse/port où Docker expose ses métriques.
* `experimental: true` est nécessaire car la fonctionnalité est encore expérimentale.

🔄 Redémarre Docker pour appliquer les changements :

```bash
sudo systemctl restart docker
```

***

### 🔹 Étape 2 : Lancer Prometheus comme conteneur

Crée un fichier `prometheus.yml` pour configurer la **cible Docker** :

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'docker'
    static_configs:
      - targets: ['host.docker.internal:9323']
```

Puis lance Prometheus en conteneur :

```bash
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

***

### 🔹 Étape 3 : Vérifier

1. Ouvre Prometheus dans ton navigateur :\
   👉 [http://localhost:9090](http://localhost:9090)
2.  Dans le champ **Query**, essaye une métrique comme :

    ```
    engine_daemon_engine_info
    ```
3. Tu devrais voir les infos sur ton Docker Engine (version, architecture, etc.).

***

### 🔹 Étape 4 : Exemple de métriques exposées

Quelques métriques que tu peux collecter :

* `engine_daemon_engine_info` → infos sur le daemon.
* `engine_daemon_network_actions_total` → nombre d’actions réseaux.
* `engine_daemon_container_states_containers` → état des conteneurs (running, stopped…).
* `engine_daemon_images_total` → nombre total d’images Docker.

## ⚙️ Configurer le démon Docker comme cible Prometheus

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
