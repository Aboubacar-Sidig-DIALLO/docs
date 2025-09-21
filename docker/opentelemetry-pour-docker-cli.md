# 📊 OpenTelemetry pour Docker CLI

### 🔹 Prérequis

* **Docker Engine ≥ 26.1.0**
* Docker CLI installé

À partir de cette version, **le CLI Docker peut émettre des métriques OpenTelemetry** sur les commandes que tu exécutes (`docker run`, `docker build`, etc.).

***

### 🔹 Fonctionnalité

* Par défaut : **désactivée** ✅
* Objectif : envoyer des métriques vers un **collecteur OpenTelemetry** (ex. Prometheus, Jaeger, Grafana Tempo, OTEL Collector).
* Permet :
  * de suivre l’usage du CLI Docker,
  * de voir combien de fois certaines commandes sont appelées,
  * de monitorer tes habitudes et flux d’utilisation Docker.

***

### 🔹 Comment activer l’export

Tu dois configurer **l’adresse du collecteur**.\
Exemple (Linux/macOS) :

```bash
export DOCKER_CLI_METRICS="otel:http://localhost:4318"
```

👉 Ici :

* `otel:` → dit au CLI d’utiliser OpenTelemetry,
* `http://localhost:4318` → correspond au port par défaut de l’**OTEL Collector (OTLP/HTTP exporter)**.

***

### 🔹 Vérification

Lance une commande Docker, par exemple :

```bash
docker ps
```

Puis vérifie dans ton collecteur OpenTelemetry que des métriques arrivent (ex. `otelcol`, Grafana Agent, etc.).

***

### 🔹 Ce que ça rapporte comme métriques

Les métriques typiques envoyées incluent :

* **Nom de la commande** (`docker build`, `docker run`, etc.)
* **Succès ou échec** de la commande
* **Durée d’exécution**
* **Arguments principaux** (pas tout, mais de quoi profiler)

***

### ⚠️ Attention

* **Opt-in uniquement** → rien n’est envoyé sans ta configuration.
* **Tu choisis ton endpoint** → tes données ne partent pas ailleurs.
* **Usage recommandé** → pour observabilité interne, intégration CI/CD, analytics d’usage Docker.

## 🔎 Qu’est-ce qu’OpenTelemetry (OTel) ?

**OpenTelemetry (OTel)** est un **framework open source** qui sert à collecter, générer et gérer des données d’observabilité.\
Ces données sont de trois types :

* **Traces** 🟣 → suivent le parcours d’une requête ou d’un événement à travers différents services (utile pour le tracing distribué).
* **Métriques** 🟢 → valeurs chiffrées comme CPU, mémoire, latence, nombre de requêtes, erreurs, etc.
* **Logs** 🔵 → messages textuels générés par les applications ou systèmes.

***

### 🎯 Objectifs d’OpenTelemetry

1. **Standardisation** :
   * Fournir un langage et un protocole communs pour la collecte de données.
   * Éviter de dépendre d’un outil ou d’un fournisseur spécifique.
2. **Interopérabilité** :
   * Fonctionne avec **beaucoup d’outils de monitoring et observabilité** comme Prometheus, Grafana, Jaeger, Elastic, Datadog, New Relic, etc.
3. **Instrumentation facile** :
   * Fournit des **librairies prêtes à l’emploi** pour différents langages (Java, Go, Python, JS, etc.).
   * Permet d’ajouter de la télémétrie sans réécrire ton application.

***

### 🛠️ Dans le contexte Docker CLI

Avec **le support OpenTelemetry**, le **CLI Docker peut émettre des métriques** (événements, commandes exécutées, durée, statut, etc.) selon le standard OTel.\
👉 Ça veut dire que tu peux suivre et analyser **comment tes commandes Docker sont utilisées**, en envoyant ces infos vers un collecteur OTel ou une plateforme d’observabilité.

***

### 📊 Exemple de flux OTel

1. **Application ou CLI** génère les données d’observabilité (Docker CLI dans ton cas).
2. Ces données passent via un **OTel Collector**.
3. Le collecteur exporte vers des systèmes comme **Prometheus, Jaeger ou Grafana** pour les visualiser.

***

✅ En résumé :\
**OpenTelemetry = le standard universel et open source pour collecter traces, métriques et logs, indépendamment du fournisseur d’outils.**

## ⚙️ Fonctionnement : Docker CLI + OpenTelemetry

1. **Par défaut → Pas de télémétrie**
   * Quand tu installes et utilises Docker CLI, il **n’envoie aucune donnée**.
   * C’est totalement **opt-in** → tu dois activer manuellement la télémétrie.

***

2.  **Activer l’export OpenTelemetry**

    * Tu définis une **variable d’environnement** :

    ```bash
    export DOCKER_CLI_OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
    ```

    Ici :

    * `DOCKER_CLI_OTEL_EXPORTER_OTLP_ENDPOINT` → indique où envoyer les métriques.
    * `http://localhost:4317` → correspond à l’adresse d’un **OTel Collector** qui écoute en local (port par défaut : 4317 pour gRPC, 4318 pour HTTP).

***

3. **Docker CLI génère la télémétrie**
   * Chaque fois que tu tapes une commande `docker run`, `docker build`, etc. →\
     des **métriques** (durée, statut, type de commande) sont créées.
   * Ces métriques suivent le **format OTel (OTLP)**.

***

4.  **Le rôle de l’OpenTelemetry Collector** 🛠️

    * Le **collector** est un composant central qui :
      * **reçoit** les données de télémétrie (depuis Docker CLI, ton app, etc.),
      * peut **les transformer/filtrer** (ex. : garder seulement certaines métriques),
      * puis les **exporte vers un backend**.

    Exemple de backends possibles :

    * **Prometheus / InfluxDB** → stockage de métriques.
    * **Jaeger / Zipkin** → visualisation des traces.
    * **Grafana** → dashboards interactifs.

***

5. **Visualisation des données** 📊
   * Si ton backend le permet (ex. Grafana branché sur Prometheus), tu peux avoir :
     * 📈 des graphes de **durée moyenne des commandes Docker**
     * 🔁 un suivi de la **fréquence des builds/run**
     * 🚨 des alertes si certaines commandes échouent souvent

***

✅ **Résumé simple :**

* Le Docker CLI **n’émet rien par défaut**.
* Tu actives l’export en définissant une variable d’env. (`DOCKER_CLI_OTEL_EXPORTER_OTLP_ENDPOINT`).
* Les données vont vers un **OTel Collector**.
* Le collector envoie ces données vers ton backend (Prometheus, InfluxDB, Grafana, etc.) pour analyse.

## ⚙️ Mise en place : OpenTelemetry pour Docker CLI

### 1. Préparer les fichiers de config

👉 **`compose.yaml`**

```yaml
name: cli-otel
services:
  prometheus:
    image: prom/prometheus
    command:
      - "--config.file=/etc/prometheus/prom.yml"
    ports:
      # Interface Prometheus : http://localhost:9091
      - 9091:9090
    restart: always
    volumes:
      - prom_data:/prometheus
      - ./prom.yml:/etc/prometheus/prom.yml

  otelcol:
    image: otel/opentelemetry-collector
    restart: always
    depends_on:
      - prometheus
    ports:
      - 4317:4317   # Port OTLP gRPC (où le Docker CLI enverra ses métriques)
    volumes:
      - ./otelcol.yml:/etc/otelcol/config.yaml

volumes:
  prom_data:
```

👉 **`otelcol.yml`** (collector)

```yaml
# Réception OTLP (depuis Docker CLI)
receivers:
  otlp:
    protocols:
      grpc:
      http:

# Export vers Prometheus
exporters:
  prometheus:
    endpoint: "0.0.0.0:8889"

service:
  pipelines:
    metrics:
      receivers: [otlp]
      exporters: [prometheus]
```

👉 **`prom.yml`** (Prometheus)

```yaml
scrape_configs:
  - job_name: "otel-collector"
    scrape_interval: 1s
    static_configs:
      - targets: ["otelcol:8889"]
```

***

### 2. Lancer l’infra

```bash
docker compose up -d
```

Cela démarre :

* **otelcol** → reçoit les métriques du Docker CLI (via OTLP gRPC sur `4317`)
* **prometheus** → scrape les métriques de l’OTel collector (`:8889`)
* **Prometheus UI** accessible sur 👉 [http://localhost:9091](http://localhost:9091)

***

### 3. Activer l’export OpenTelemetry dans le Docker CLI

```bash
export DOCKER_CLI_OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
```

⚠️ Cette variable doit être définie **dans ton shell avant d’exécuter `docker`**.\
(Tu peux aussi l’ajouter dans ton `~/.bashrc` ou `~/.zshrc` pour la rendre permanente).

***

### 4. Générer des métriques

Exécute une commande Docker, par exemple :

```bash
docker version
```

Cela envoie une métrique au collector.

***

### 5. Vérifier dans Prometheus

* Ouvre : [http://localhost:9091/graph](http://localhost:9091/graph)
* Dans le champ **Query**, tape :

```
command_time_milliseconds_total
```

Puis **Execute** → tu verras les métriques des commandes Docker (temps d’exécution, nombre d’invocations…).

***

✅ **Résumé** :

* Tu démarres l’infra avec `docker compose up`.
* Tu exportes `DOCKER_CLI_OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317`.
* Tu lances une commande Docker.
* Tu visualises la métrique `command_time_milliseconds_total` dans Prometheus.

## 📊 Métrique disponible dans Docker CLI (OpenTelemetry)

#### 🔹 Nom

`command.time`

#### 🔹 Description

Mesure le **temps d’exécution** d’une commande Docker (en millisecondes).

#### 🔹 Attributs associés

Chaque métrique est enrichie avec des **labels (attributs)** pour permettre une analyse plus fine :

| Attribut                | Description                                                                          |
| ----------------------- | ------------------------------------------------------------------------------------ |
| `command.name`          | Le nom de la commande exécutée (ex: `docker run`, `docker ps`, `docker build`, etc.) |
| `command.status.code`   | Le code de sortie de la commande (`0` si succès, autre valeur si erreur)             |
| `command.stderr.isatty` | `true` si `stderr` est connecté à un TTY (terminal interactif), sinon `false`        |
| `command.stdin.isatty`  | `true` si `stdin` est connecté à un TTY                                              |
| `command.stdout.isatty` | `true` si `stdout` est connecté à un TTY                                             |

***

#### 🔹 Exemple concret

Si tu exécutes :

```bash
docker ps
```

Alors dans **Prometheus**, tu pourrais voir une entrée comme :

```
command_time_milliseconds_total{
  command_name="ps",
  command_status_code="0",
  command_stderr_isatty="true",
  command_stdin_isatty="false",
  command_stdout_isatty="true"
} 45
```

👉 Cela veut dire :

* La commande `docker ps` a pris **45 ms** à s’exécuter.
* Elle s’est terminée avec un **exit code = 0** (succès).
* `stdout` et `stderr` étaient reliés à un terminal.

***

⚡ Ça permet ensuite de faire des **stats d’usage** :

* Temps moyen d’exécution par commande (`avg by(command_name)`).
* Taux d’erreurs (`rate(command_time_milliseconds_total{command_status_code!="0"}[5m])`).
* Profil d’utilisation (quelles commandes sont les plus utilisées).
