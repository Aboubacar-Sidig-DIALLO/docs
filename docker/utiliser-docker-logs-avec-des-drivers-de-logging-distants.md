# 📑 Utiliser docker logs avec des drivers de logging distants

### 🔹 Vue d’ensemble

Normalement, lorsque tu configures un **driver de logging distant** (ex. : `fluentd`, `awslogs`, `gcplogs`, `splunk`, plugin externe, etc.), les logs ne sont **pas accessibles directement** avec `docker logs`.

👉 Pour pallier ça, Docker a introduit la fonctionnalité **dual logging** :

* Le **driver distant** continue d’envoyer les logs vers sa destination externe.
* En parallèle, Docker utilise **le driver `local` comme cache** afin que tu puisses quand même utiliser `docker logs`.

***

### 🔹 Fonctionnement du cache local

* **Rotation activée par défaut** ✅
* Limite : **5 fichiers de 20 MB** chacun (avant compression) par conteneur.
* Accessible via `docker logs <container_id>` même si ton conteneur envoie les logs vers AWS, GCP, Fluentd, etc.

***

### 🔹 Exemple : lecture avec et sans dual logging

#### Sans dual logging (exemple : `fluentd` configuré **sans cache local**)

```bash
docker logs my_container
```

👉 Renvoie une erreur ou rien du tout, car le driver `fluentd` ne supporte pas `docker logs`.

***

#### Avec dual logging activé (par défaut si non support natif)

```bash
docker run -d \
  --log-driver=fluentd \
  --log-opt fluentd-address=localhost:24224 \
  alpine sh -c "while true; do echo hello $(date); sleep 2; done"
```

Même si les logs partent vers `fluentd`, tu peux encore faire :

```bash
docker logs -f <container_id>
```

✅ Tu vois les logs grâce au **cache local**.

***

⚙️ **Personnaliser la configuration du cache local**\
Dans `/etc/docker/daemon.json` :

```json
{
  "log-driver": "fluentd",
  "log-opts": {
    "fluentd-address": "localhost:24224"
  },
  "dual-logging": true,
  "log-opts-local": {
    "max-size": "10m",
    "max-file": "7"
  }
}
```

***

⚠️ **Attention**

* Dual logging **augmente l’utilisation disque** (cache local + envoi distant).
* Tu peux **désactiver dual logging** si tu veux uniquement du remote logging → mais tu perdras la possibilité d’utiliser `docker logs`.

## 🚫 Utiliser `docker logs` **sans dual logging activé**

Quand tu **désactives le dual logging**, Docker **ne garde plus de copie locale** des logs. Résultat : `docker logs` ne fonctionne pas, et tu dois consulter les logs **directement dans ton backend distant** (Splunk, Fluentd, CloudWatch, etc.).

***

### 🔹 Exemple avec **Splunk**

#### Étape 1 : Configurer `daemon.json`

On désactive explicitement le cache local avec `cache-disabled=true` :

```json
{
  "log-driver": "splunk",
  "log-opts": {
    "splunk-url": "https://splunk.example.com:8088",
    "splunk-token": "your-splunk-token",
    "splunk-insecureskipverify": "true",
    "cache-disabled": "true"
  }
}
```

Puis on redémarre le démon :

```bash
sudo systemctl restart docker
```

***

#### Étape 2 : Lancer un conteneur

```bash
docker run -d --name testlog busybox top
```

***

#### Étape 3 : Tenter de lire les logs

```bash
docker logs testlog
```

👉 Résultat :

```
Error response from daemon: configured logging driver does not support reading
```

***

### 🔹 Résumé

* ✅ Les logs partent bien vers **Splunk**.
* 🚫 Impossible de les lire avec `docker logs`.
* Tu dois **consulter les logs depuis ton backend** (Splunk dans ce cas).

## ✅ Utiliser `docker logs` **avec dual logging activé**

Quand tu **gardes le dual logging activé** (c’est le mode par défaut), Docker conserve une **copie locale** des logs, même si tu utilises un **driver de logging distant** comme Splunk, Fluentd ou CloudWatch.

👉 Résultat : tu peux **continuer à utiliser `docker logs`** en plus de ton système de logs externe.

***

### 🔹 Exemple avec Splunk

#### Étape 1 : Configurer `daemon.json`

Ici on définit le driver par défaut sur **Splunk**, mais **sans désactiver** le cache (`cache-disabled` n’apparaît pas) :

```json
{
  "log-driver": "splunk",
  "log-opts": {
    "splunk-url": "https://splunk.example.com:8088",
    "splunk-token": "your-splunk-token",
    "splunk-insecureskipverify": "true"
  }
}
```

Puis on redémarre le démon Docker :

```bash
sudo systemctl restart docker
```

***

#### Étape 2 : Lancer un conteneur

```bash
docker run -d --name testlog busybox top
```

***

#### Étape 3 : Lire les logs avec `docker logs`

Même si Splunk reçoit déjà les logs, tu peux **aussi** les lire en local :

```bash
docker logs testlog
```

👉 Exemple de sortie :

```
2019-02-04T19:48:15.423Z [INFO]  core: marked as sealed
2019-02-04T19:48:15.423Z [INFO]  core: pre-seal teardown starting
2019-02-04T19:48:15.423Z [INFO]  core: stopping cluster listeners
2019-02-04T19:48:15.423Z [INFO]  core: shutting down forwarding rpc listeners
2019-02-04T19:48:15.599Z [INFO]  core: rpc listeners successfully shut down
2019-02-04T19:48:15.599Z [INFO]  core: cluster listeners successfully shut down
```

***

### 🔹 Résumé

* 🚀 **Avec dual logging** : logs accessibles à la fois dans `docker logs` **et** dans Splunk.
* ❌ **Sans dual logging** : logs envoyés uniquement dans Splunk (erreur si tu fais `docker logs`).
* ℹ️ Pour les drivers qui supportent déjà la lecture (`json-file`, `local`, `journald`), il n’y a pas de différence.

0, or false).

**ChatGPT a dit :**

Parfait 👍 ! Voici un récap clair et détaillé des **options de configuration du dual logging cache** (qui est en fait basé sur le driver `local` mais avec le préfixe `cache-`) :

***

## ⚙️ Options de configuration du dual logging cache

| Option             | Valeur par défaut | Description                                                                                                                                                                                                                                                                                         |
| ------------------ | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **cache-disabled** | `"false"`         | <p>Désactive complètement le cache local. Valeur booléenne sous forme de chaîne (<code>"true"</code>, <code>"false"</code>, <code>"1"</code>, <code>"0"</code>).<br>👉 Si activé (<code>true</code>), tu ne pourras <strong>plus utiliser <code>docker logs</code></strong> pour lire les logs.</p> |
| **cache-max-size** | `"20m"`           | <p>Taille maximale d’un fichier de log avant rotation. Format : entier + unité (<code>k</code>, <code>m</code>, <code>g</code>).<br>👉 Exemple : <code>"50m"</code> = rotation après 50 Mo.</p>                                                                                                     |
| **cache-max-file** | `"5"`             | Nombre maximum de fichiers de logs conservés. Si la rotation crée plus de fichiers, les plus anciens sont supprimés.                                                                                                                                                                                |
| **cache-compress** | `"true"`          | <p>Active ou désactive la <strong>compression</strong> des fichiers de logs après rotation. Valeur booléenne sous forme de chaîne.<br>👉 Compression = gain d’espace disque mais un peu plus de CPU.</p>                                                                                            |

***

## 🔹 Exemple : Configurer globalement dans `daemon.json`

👉 Ici on configure le cache pour garder **10 fichiers de 50 Mo max chacun, sans compression** :

```json
{
  "log-driver": "splunk",
  "log-opts": {
    "splunk-url": "https://splunk.example.com:8088",
    "splunk-token": "your-splunk-token",
    "splunk-insecureskipverify": "true",
    "cache-max-size": "50m",
    "cache-max-file": "10",
    "cache-compress": "false"
  }
}
```

Puis on redémarre Docker :

```bash
sudo systemctl restart docker
```

***

## 🔹 Exemple : Configurer au niveau d’un conteneur

```bash
docker run -d \
  --log-driver=splunk \
  --log-opt splunk-url=https://splunk.example.com:8088 \
  --log-opt splunk-token=your-splunk-token \
  --log-opt cache-max-size=100m \
  --log-opt cache-max-file=3 \
  --log-opt cache-compress=true \
  busybox top
```

***

✅ Résultat :

* Logs envoyés **à Splunk**
* Copie locale avec rotation : max **3 fichiers de 100 Mo** compressés
* Accessible avec `docker logs <container>`

## 🚫 Désactiver le cache de double journalisation (dual logging cache)

Le **dual logging cache** permet de lire les logs avec `docker logs` même si tu utilises un driver de logs **qui ne supporte pas la lecture locale** (comme `splunk`, `fluentd`, `awslogs`, etc.).

👉 **Quand le désactiver ?**

* Si tu veux **économiser de l’espace disque** sur la machine hôte.
* Si tu n’as **aucun besoin** d’utiliser `docker logs` (tu lis uniquement via ton système de logs distant).
* Si tu veux éviter une **double écriture** (locale + remote).

***

### 🔹 Désactiver globalement (via `daemon.json`)

Exemple : activer `splunk` comme driver de logs **par défaut** et désactiver le cache :

```json
{
  "log-driver": "splunk",
  "log-opts": {
    "cache-disabled": "true",
    "splunk-url": "https://splunk.example.com:8088",
    "splunk-token": "your-splunk-token",
    "splunk-insecureskipverify": "true"
  }
}
```

Puis redémarre Docker :

```bash
sudo systemctl restart docker
```

***

### 🔹 Désactiver pour un seul conteneur

```bash
docker run -d \
  --log-driver=splunk \
  --log-opt splunk-url=https://splunk.example.com:8088 \
  --log-opt splunk-token=your-splunk-token \
  --log-opt cache-disabled=true \
  busybox top
```

***

### ⚠️ Notes importantes

*   Avec `cache-disabled=true`, la commande **`docker logs` ne fonctionnera plus** pour ce conteneur.\
    Exemple :

    ```bash
    docker logs <container>
    Error response from daemon: configured logging driver does not support reading
    ```
* Cela n’a **aucun effet** si tu utilises un driver qui supporte déjà la lecture locale (`json-file`, `local`, `journald`).

