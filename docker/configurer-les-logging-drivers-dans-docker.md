# ⚙️ Configurer les logging drivers dans Docker

Docker propose plusieurs mécanismes pour gérer et collecter les logs des conteneurs, appelés **logging drivers**.

Chaque **daemon Docker** possède un **logging driver par défaut**.\
➡️ Tous les conteneurs utilisent ce driver sauf si on leur en attribue un autre explicitement.

***

### 🔹 1. Le logging driver par défaut

* **`json-file`** est le driver utilisé par défaut.
  * Les logs des conteneurs sont stockés sous forme **JSON** sur le disque.
  * Aucun **log-rotation** n’est activé par défaut ⚠️.
  * Risque : un conteneur très bavard peut remplir tout le disque → 💥 _disk exhaustion_.

***

### 🔹 2. Logging driver recommandé

👉 **`local`** est recommandé pour un usage standard, car :

* Active la **rotation des logs par défaut**,
* Utilise un format de fichier plus efficace,
* Réduit le risque de saturation disque.

***

### 🔹 3. Autres drivers disponibles

Docker propose plusieurs drivers adaptés à différents usages :

* **`json-file`** → format JSON (par défaut, pas de rotation).
* **`local`** → optimisé, rotation automatique.
* **`syslog`** → envoie les logs vers le démon **syslog** du système.
* **`journald`** → envoie les logs vers **systemd journal**.
* **`gelf`** → envoie vers des systèmes centralisés comme Graylog ou Logstash.
* **`fluentd`** → envoie vers **Fluentd**.
* **`awslogs`** → envoie vers **Amazon CloudWatch Logs**.
* **`splunk`** → envoie vers **Splunk**.
* **`etwlogs`** → (Windows) envoie vers **Event Tracing for Windows (ETW)**.
* **`gcplogs`** → envoie vers **Google Cloud Logging**.
* **`none`** → désactive complètement la collecte des logs.

👉 Tu peux aussi développer ou installer des **plugins de logging driver**.

***

### 🔹 4. Conseils pratiques

* ✅ Utilise **`local`** plutôt que **`json-file`** si tu veux une gestion automatique et efficace.
* ✅ Pour la centralisation des logs (prod), configure un driver comme `fluentd`, `gelf`, `awslogs`, `gcplogs` ou `splunk`.
* ❌ Évite d’utiliser `json-file` en production sans rotation → risque de **disque plein**.

## ⚙️ Configurer le **logging driver par défaut** dans Docker

Tu peux définir quel **logging driver** sera utilisé par défaut par tous les nouveaux conteneurs, via le fichier de configuration du démon **`daemon.json`**.

***

### 🔹 1. Emplacement du fichier `daemon.json`

* **Linux** → `/etc/docker/daemon.json`
* **Windows Server** → `C:\ProgramData\docker\config\daemon.json`
* **Docker Desktop (Mac/Windows)** → via l’UI : _Settings → Docker Engine_

***

### 🔹 2. Exemple : utiliser `local` comme driver par défaut

C’est recommandé pour éviter la saturation disque.

```json
{
  "log-driver": "local"
}
```

***

### 🔹 3. Exemple : configurer `json-file` avec options

Si tu veux garder `json-file` mais ajouter de la rotation et des filtres :

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",             // limite de taille des logs (10 Mo)
    "max-file": "3",               // nombre de fichiers de rotation
    "labels": "production_status", // inclure ce label dans les logs
    "env": "os,customer"           // inclure ces variables d’environnement
  }
}
```

⚠️ **Important :** toutes les valeurs doivent être des **chaînes de caractères** (même les nombres ou booléens).

***

### 🔹 4. Appliquer la configuration

Après modification, redémarre Docker pour que la config soit prise en compte :

```bash
sudo systemctl restart docker
```

👉 Cette modification ne concerne que les **nouveaux conteneurs** créés après le changement.\
Les conteneurs existants gardent leur logging driver d’origine.

***

### 🔹 5. Vérifier le logging driver actuel

Tu peux vérifier le driver par défaut avec :

```bash
docker info --format '{{.LoggingDriver}}'
```

Exemple de sortie :

```
json-file
```

***

📌 Résumé :

* Par défaut → `json-file` (sans rotation).
* Meilleure pratique → utiliser `local` en dev/prod simple.
* Pour un système de logs centralisé → utiliser `fluentd`, `gelf`, `awslogs`, `gcplogs`, etc.

## 📝 Configurer le **logging driver** pour un conteneur spécifique

Tu peux forcer un conteneur à utiliser un **logging driver différent** de celui défini par défaut dans `daemon.json` en utilisant l’option `--log-driver` lors de la création.

***

### 🔹 1. Exemple : désactiver complètement les logs

```bash
docker run -it --log-driver none alpine ash
```

👉 Ici, le conteneur **n’écrit aucun log**. Utile pour les jobs très verbeux qui pollueraient les logs.

***

### 🔹 2. Exemple : utiliser `json-file` avec options personnalisées

```bash
docker run -d \
  --name webapp \
  --log-driver json-file \
  --log-opt max-size=5m \
  --log-opt max-file=2 \
  nginx
```

✅ Les logs de ce conteneur sont stockés localement en JSON avec **rotation automatique** (5 Mo max, 2 fichiers).

***

### 🔹 3. Exemple : envoyer les logs vers Fluentd

```bash
docker run -d \
  --name app-fluentd \
  --log-driver=fluentd \
  --log-opt fluentd-address=localhost:24224 \
  --log-opt tag="docker.{{.Name}}" \
  my-app
```

👉 Ici les logs sont envoyés directement à **Fluentd**, ce qui permet un traitement centralisé (ex: ELK stack).

***

### 🔹 4. Vérifier le driver de logs utilisé par un conteneur

Utilise `docker inspect` pour savoir quel logging driver est actif :

```bash
docker inspect -f '{{.HostConfig.LogConfig.Type}}' <CONTAINER>
```

Exemple de sortie :

```
json-file
```

Tu peux aussi voir les **options configurées** :

```bash
docker inspect -f '{{json .HostConfig.LogConfig.Config}}' <CONTAINER>
```

***

📌 Résumé :

* `--log-driver` permet de définir un driver spécifique à un conteneur.
* `--log-opt` permet de passer des options (rotation, tags, endpoint, etc.).
* Vérification possible avec `docker inspect`.

## ⚙️ Configurer le **mode de livraison des logs** (Docker → Logging driver)

Docker permet de choisir **comment les logs sont transmis** du conteneur vers le driver de logs :

***

### 🔹 1. Modes disponibles

#### ✅ Mode par défaut : **blocking (direct)**

* Chaque message est transmis directement au driver.
* **Avantage** : aucun log n’est perdu.
* **Inconvénient** : si le driver est lent ou surchargé, le flux `STDOUT/STDERR` de l’application peut **se bloquer** → risque de comportements inattendus.

***

#### 🚀 Mode alternatif : **non-blocking**

* Les logs passent par un **buffer intermédiaire par conteneur**.
* **Avantage** : évite que l’application se bloque à cause de la pression des logs.
* **Inconvénient** : si le buffer est plein, les **nouveaux messages sont perdus** (droppés).

***

### 🔹 2. Taille du buffer (`max-buffer-size`)

* Défaut : `1m` (1 Mo ≈ 1 million d’octets).
* Peut être ajusté avec des suffixes comme `KiB`, `MiB`, `g`, etc.
  * `1KiB` = 1024 octets
  * `4m` = 4 millions d’octets
  * `2g` = 2 milliards d’octets

***

### 🔹 3. Exemple : Conteneur avec buffer 4 MB en **mode non-blocking**

```bash
docker run -it \
  --log-opt mode=non-blocking \
  --log-opt max-buffer-size=4m \
  alpine ping 127.0.0.1
```

👉 Ici :

* Les logs sont mis en tampon jusqu’à 4 Mo.
* Si le buffer se remplit, les logs supplémentaires seront **ignorés** au lieu de bloquer l’application.

***

### 🔹 4. Ajouter des **variables d’environnement** ou **labels** aux logs

Certains drivers (ex. `json-file`, `fluentd`, `gelf`) peuvent enrichir les logs avec les `--env` et `--label` du conteneur.

Exemple :

```bash
docker run -dit \
  --label production_status=testing \
  -e os=ubuntu \
  alpine sh
```

Extrait des logs générés par **json-file** :

```json
"attrs": {
  "production_status": "testing",
  "os": "ubuntu"
}
```

👉 Cela permet d’ajouter du **contexte** aux logs (environnement, version, statut, etc.).

***

📌 **Résumé** :

* `mode=blocking` = fiable mais peut bloquer l’app.
* `mode=non-blocking` = fluide mais risque de pertes si buffer plein.
* `max-buffer-size` permet de contrôler la taille de ce tampon.
* Les **labels** et **variables d’env.** enrichissent les logs avec des métadonnées.

## 📋 Les drivers de logs supportés par Docker

Docker propose plusieurs **drivers de logging** qui déterminent **où et comment les logs de conteneurs sont stockés ou transmis**.

***

### 🔹 Liste des drivers disponibles

| **Driver**    | **Description**                                                                                                                              |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| **none**      | Aucun log n’est conservé. `docker logs` ne retourne rien. Utile pour réduire l’overhead si tu n’as pas besoin des logs.                      |
| **local**     | Stockage local dans un format optimisé, faible overhead et rotation activée par défaut. 🔥 **Recommandé** pour limiter la saturation disque. |
| **json-file** | Stockage en JSON (driver **par défaut**). Simple mais peut vite remplir le disque car pas de rotation automatique.                           |
| **syslog**    | Envoie les logs vers le **démon syslog** du système hôte.                                                                                    |
| **journald**  | Envoie les logs vers **journald** (systemd).                                                                                                 |
| **gelf**      | Envoie les logs au format **GELF** vers un endpoint (ex : Graylog, Logstash).                                                                |
| **fluentd**   | Envoie les logs vers **Fluentd** (`forward input`). Le démon fluentd doit tourner sur l’hôte.                                                |
| **awslogs**   | Envoie les logs vers **Amazon CloudWatch Logs**.                                                                                             |
| **splunk**    | Envoie les logs vers **Splunk** via HTTP Event Collector.                                                                                    |
| **etwlogs**   | Écrit les logs comme événements **ETW (Event Tracing for Windows)**. Disponible **uniquement sur Windows**.                                  |
| **gcplogs**   | Envoie les logs vers **Google Cloud Platform (GCP) Logging**.                                                                                |

***

### 🔹 Limitations générales des logging drivers

* **Rotation des logs** : la lecture de logs implique parfois de **décompresser** des fichiers de rotation → augmentation temporaire de la **CPU** et de l’usage **disque**.
* **Capacité limitée par le stockage** : les logs sont bornés par la capacité du disque hébergeant le répertoire de données Docker (`/var/lib/docker`).
* Certains drivers nécessitent un **service externe en fonctionnement** (ex. `fluentd`, `journald`, `syslog`).
* `none` = gain de performance, mais **aucune trace disponible**.

***

👉 Recommandation pratique :

* **Dév/local** → `local` (rapide, rotation auto).
* **Prod avec centralisation** → `fluentd`, `gelf`, `awslogs`, `gcplogs`, `splunk` selon ton écosystème.
* **Linux avec systemd** → `journald` pour intégration native.
