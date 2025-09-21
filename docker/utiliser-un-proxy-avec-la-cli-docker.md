# 🌐 Utiliser un proxy avec la CLI Docker

Quand tu exécutes des conteneurs, il arrive que ceux-ci aient besoin d’accéder à Internet via un **proxy** (HTTP, HTTPS ou FTP).\
Docker permet de configurer cela grâce aux **variables d’environnement**.

⚠️ Attention :

* Cette configuration ne concerne que **le client Docker (CLI)** et les **conteneurs**.
* Si tu veux configurer le **daemon Docker** (`dockerd`), il faut suivre une autre documentation : _Configurer le daemon Docker pour utiliser un proxy_.
* Si tu es sous **Docker Desktop**, tu dois utiliser les paramètres intégrés à l’application.

***

### 🔹 1. Configurer le client Docker (CLI)

Tu peux définir les variables d’environnement suivantes :

* `HTTP_PROXY` → pour les connexions HTTP
* `HTTPS_PROXY` → pour les connexions HTTPS
* `FTP_PROXY` → pour les connexions FTP
* `NO_PROXY` → pour définir des exceptions (hôtes ou domaines à ne pas passer par le proxy)

Exemple :

```bash
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=https://proxy.example.com:8443
export FTP_PROXY=ftp://proxy.example.com:2121
export NO_PROXY="localhost,127.0.0.1,.mon-domaine.local"
```

👉 Ajoute ces lignes dans `~/.bashrc`, `~/.zshrc` ou ton fichier de configuration de shell préféré pour les rendre permanentes.

***

### 🔹 2. Définir un proxy directement avec Docker CLI

Tu peux aussi passer ces variables directement quand tu exécutes un conteneur :

```bash
docker run -e HTTP_PROXY=http://proxy.example.com:8080 \
           -e HTTPS_PROXY=https://proxy.example.com:8443 \
           -e NO_PROXY=localhost,127.0.0.1 \
           ubuntu:24.04 curl http://example.com
```

Ainsi, le conteneur utilise le proxy défini **seulement pendant cette exécution**.

***

### 🔹 3. Notes importantes

* Il n’existe **aucun standard universel** pour ces variables (`http_proxy`, `HTTP_PROXY`, `Https_proxy` … certaines applis attendent des minuscules, d’autres des majuscules).\
  👉 C’est pour ça qu’on définit souvent les **deux versions** :

```bash
-e HTTP_PROXY=http://proxy.example.com:8080 \
-e http_proxy=http://proxy.example.com:8080
```

* Tu peux consulter un excellent article de l’équipe GitLab :\
  &#xNAN;**"We need to talk: Can we standardize NO\_PROXY?"**

## ⚙️ Configurer le client Docker avec un proxy (`~/.docker/config.json`)

Docker permet de définir des paramètres de **proxy** dans un fichier de configuration JSON situé dans :

```
~/.docker/config.json
```

Cette configuration est appliquée automatiquement pour :

* **les builds (`docker build`)**
* **les conteneurs (`docker run`)**

👉 Elle ne s’applique pas au **CLI Docker lui-même** ni au **daemon Docker**.

***

### 🔹 Exemple de configuration

```json
{
  "proxies": {
    "default": {
      "httpProxy": "http://proxy.example.com:3128",
      "httpsProxy": "https://proxy.example.com:3129",
      "ftpProxy": "ftp://proxy.example.com:2121",
      "noProxy": "*.test.example.com,.example.org,127.0.0.0/8"
    }
  }
}
```

***

### 🔹 Explications des paramètres

| Propriété      | Description                                                                           |
| -------------- | ------------------------------------------------------------------------------------- |
| **httpProxy**  | Définit les variables `HTTP_PROXY` et `http_proxy` + les arguments de build associés  |
| **httpsProxy** | Définit les variables `HTTPS_PROXY` et `https_proxy`                                  |
| **ftpProxy**   | Définit les variables `FTP_PROXY` et `ftp_proxy`                                      |
| **noProxy**    | Définit les variables `NO_PROXY` et `no_proxy` (domaines/IP qui contournent le proxy) |
| **allProxy**   | Définit `ALL_PROXY` et `all_proxy` (utilisé par certains clients réseau, ex. SOCKS)   |

***

### 🔹 Points importants ⚠️

* Les paramètres prennent effet **après avoir sauvegardé le fichier**, inutile de redémarrer Docker.
* Ils ne concernent que **les nouveaux conteneurs** et **les nouvelles builds** (pas les conteneurs déjà lancés).
* ⚠️ Attention : les **URLs de proxy peuvent contenir des identifiants et mots de passe** → ils seront stockés en **clair** dans ce fichier.
* Les variables d’environnement définies sont visibles via l’API distante ou dans un `docker commit`.

***

✅ Donc, si tu veux un proxy **automatiquement appliqué à tous tes conteneurs et builds**, c’est cette méthode qu’il faut utiliser.\
Mais pour configurer le **daemon Docker** lui-même (pour `pull`, `push`, etc.), il faut modifier la configuration du service `dockerd`.

## 🚀 Exécution de conteneurs avec un proxy

Quand tu définis un proxy dans ton fichier `~/.docker/config.json`, **Docker injecte automatiquement les variables d’environnement liées au proxy** dans chaque nouveau conteneur que tu démarres.

***

### 🔹 Exemple

Supposons que tu aies configuré ton fichier ainsi :

```json
{
  "proxies": {
    "default": {
      "httpProxy": "http://proxy.example.com:3128",
      "httpsProxy": "http://proxy.example.com:3129",
      "noProxy": "*.test.example.com,.example.org,127.0.0.0/8"
    }
  }
}
```

Si tu lances un conteneur :

```bash
docker run --rm alpine sh -c 'env | grep -i _PROXY'
```

Tu obtiendras :

```
https_proxy=http://proxy.example.com:3129
HTTPS_PROXY=http://proxy.example.com:3129
http_proxy=http://proxy.example.com:3128
HTTP_PROXY=http://proxy.example.com:3128
no_proxy=*.test.example.com,.example.org,127.0.0.0/8
NO_PROXY=*.test.example.com,.example.org,127.0.0.0/8
```

***

### 🔹 Points à retenir

* Docker définit **les deux versions** (majuscules et minuscules) → car certains programmes ne lisent que l’une ou l’autre (`HTTP_PROXY` vs `http_proxy`).
* Les variables sont présentes **par défaut dans tous les nouveaux conteneurs**, pas besoin de les passer à chaque `docker run`.
* Tu peux **ajouter ou écraser** ces variables manuellement avec `-e` lors du `docker run`.

Exemple :

```bash
docker run --rm -e HTTP_PROXY="http://autre-proxy:8080" alpine env | grep -i proxy
```

## 🚀 Build avec une configuration de proxy

Lorsque tu exécutes un `docker build`, Docker pré-remplit automatiquement des **arguments de build liés au proxy** grâce au fichier `~/.docker/config.json`.

***

### 🔹 Exemple pratique

Si tu as défini un proxy comme ceci dans `~/.docker/config.json` :

```json
{
  "proxies": {
    "default": {
      "httpProxy": "http://proxy.example.com:3128",
      "httpsProxy": "https://proxy.example.com:3129",
      "noProxy": "*.test.example.com,.example.org,127.0.0.0/8"
    }
  }
}
```

Et tu lances un build :

```bash
docker build \
  --no-cache \
  --progress=plain \
  - <<EOF
FROM alpine
RUN env | grep -i _PROXY
EOF
```

Tu verras automatiquement injectées dans l’étape de build :

```
HTTPS_PROXY=https://proxy.example.com:3129
no_proxy=*.test.example.com,.example.org,127.0.0.0/8
NO_PROXY=*.test.example.com,.example.org,127.0.0.0/8
https_proxy=https://proxy.example.com:3129
http_proxy=http://proxy.example.com:3128
HTTP_PROXY=http://proxy.example.com:3128
```

***

### 🔹 Pourquoi c’est utile ?

* Les étapes du `Dockerfile` qui nécessitent un accès externe (ex : `RUN apt-get update`, `RUN wget …`) passent automatiquement par le proxy.
* Pas besoin de définir manuellement les variables `ARG http_proxy`, `ARG https_proxy` à chaque fois.
* Compatible avec **BuildKit**, donc fonctionne aussi avec `docker buildx build`.

***

### 🔹 Points à retenir

1. Les proxies définis dans `~/.docker/config.json` sont **propagés automatiquement aux builds** via des `--build-arg` implicites.
2.  Tu peux toujours **surcharger** les valeurs avec `--build-arg` :

    ```bash
    docker build --build-arg http_proxy=http://autre-proxy:8080 .
    ```
3.  Si tu ne veux **aucun proxy** pendant un build, tu peux **désactiver** en lançant avec :

    ```bash
    docker build --build-arg http_proxy= --build-arg https_proxy= .
    ```

## ⚙️ Configurer des proxys par démon Docker

👉 Par défaut, la clé `default` dans `~/.docker/config.json` définit la configuration de proxy **pour tous les démons Docker auxquels ton client se connecte**.\
Mais si tu as plusieurs démons (par exemple un cluster, ou différents environnements : dev, staging, prod), tu peux définir une configuration spécifique **par démon**.

***

### 🔹 Exemple

Voici un fichier `~/.docker/config.json` :

```json
{
  "proxies": {
    "default": {
      "httpProxy": "http://proxy.example.com:3128",
      "httpsProxy": "https://proxy.example.com:3129",
      "noProxy": "*.test.example.com,.example.org,127.0.0.0/8"
    },
    "tcp://docker-daemon1.example.com": {
      "noProxy": "*.internal.example.net"
    }
  }
}
```

***

#### 📌 Explications

* **`default`** → configuration utilisée pour **tous les démons**, sauf si une règle spécifique existe.
  * Ici : `httpProxy`, `httpsProxy`, `noProxy` par défaut.
* **`tcp://docker-daemon1.example.com`** → configuration spécifique pour le démon situé à cette adresse.
  * Dans cet exemple, on remplace uniquement `noProxy` pour exclure le domaine `*.internal.example.net` du proxy.
  * Si tu ne redéfinis pas `httpProxy` ou `httpsProxy`, Docker garde ceux du `default`.

***

### 🔹 Points importants

1. Cela ne configure **que le client Docker** (`docker CLI`) pour ses connexions au démon (`dockerd`).\
   👉 Si tu veux configurer le proxy **du démon lui-même**, il faut modifier la configuration de `dockerd` (souvent via `systemd` ou `/etc/systemd/system/docker.service.d/proxy.conf`).
2. Les valeurs définies dans `~/.docker/config.json` sont :
   * Appliquées aux **nouvelles connexions**,
   * Utilisées aussi lors des `build` et `run` pour définir les variables d’environnement `HTTP_PROXY`, `HTTPS_PROXY`, `NO_PROXY` dans les conteneurs.
3. Tu peux combiner plusieurs démons avec des proxys différents dans le même fichier.

## 🚀 Définir un proxy via la CLI Docker

Il existe **2 cas principaux** :

* Lors d’un **build** → `docker build`
* Lors d’un **run** → `docker run`

***

### 🔹 1. Proxy lors du build

👉 Utilisation de `--build-arg` pour passer des variables au conteneur **pendant la phase de build**.

Exemple :

```bash
docker build \
  --build-arg HTTP_PROXY="http://proxy.example.com:3128" \
  --build-arg HTTPS_PROXY="https://proxy.example.com:3129" \
  --build-arg NO_PROXY="localhost,127.0.0.1,.example.org" \
  .
```

📌 Points importants :

* Les `--build-arg` définissent des **ARGs prédéfinis** (`HTTP_PROXY`, `HTTPS_PROXY`, `NO_PROXY`, etc.).
* Ces variables existent **pendant le build uniquement**, mais **ne sont pas copiées dans l’image finale** (sécurité ✅).
* Tu peux les utiliser dans ton `Dockerfile` avec `ARG` :

```dockerfile
FROM alpine
ARG HTTP_PROXY
ARG HTTPS_PROXY
RUN apk add curl
```

***

### 🔹 2. Proxy lors du run

👉 Utilisation de `--env` (ou `-e`) pour passer des variables d’environnement **au conteneur au moment de l’exécution**.

Exemple :

```bash
docker run -it --rm \
  --env HTTP_PROXY="http://proxy.example.com:3128" \
  --env HTTPS_PROXY="https://proxy.example.com:3129" \
  --env NO_PROXY="localhost,127.0.0.1,.example.org" \
  alpine sh
```

Dans le conteneur, si tu fais :

```bash
env | grep -i proxy
```

Tu verras :

```
HTTP_PROXY=http://proxy.example.com:3128
HTTPS_PROXY=https://proxy.example.com:3129
NO_PROXY=localhost,127.0.0.1,.example.org
```

***

### 🔹 Différences clés

| Commande       | Utilisation du proxy                                 |
| -------------- | ---------------------------------------------------- |
| `docker build` | Seulement pendant le build (pas dans l’image finale) |
| `docker run`   | Proxy appliqué au conteneur en cours d’exécution     |

## ⚠️ Proxy et variables d’environnement dans les builds

👉 **Mauvaise pratique** :

Utiliser dans un `Dockerfile` :

```dockerfile
ENV HTTP_PROXY=http://proxy.example.com:3128
ENV HTTPS_PROXY=https://proxy.example.com:3129
ENV NO_PROXY=localhost,127.0.0.1,.example.org
```

📌 Problèmes :

1. 🔒 **Sécurité** → Les identifiants ou adresses internes du proxy se retrouvent **stockés dans l’image finale**.
   * Toute personne qui télécharge l’image peut voir ces infos (`docker history`, inspection des layers, etc.).
2. 🌐 **Accessibilité** → Si l’image est partagée sur un autre environnement (CI/CD, cloud, autre machine), le proxy interne peut ne pas exister → l’image devient inutilisable.
3. 🧱 **Couplage rigide** → Tu verrouilles ton image à un environnement spécifique. Une image devrait rester générique et portable.

***

👉 **Bonne pratique** :

✅ Utiliser des **ARGs prédéfinis** dans ton `Dockerfile`, qui seront injectés **au moment du build** mais **non stockés dans l’image**.

```dockerfile
FROM alpine

ARG HTTP_PROXY
ARG HTTPS_PROXY
ARG NO_PROXY

RUN apk add curl
```

Puis au build :

```bash
docker build \
  --build-arg HTTP_PROXY=http://proxy.example.com:3128 \
  --build-arg HTTPS_PROXY=https://proxy.example.com:3129 \
  --build-arg NO_PROXY=localhost,127.0.0.1,.example.org \
  .
```

📌 Résultat :

* Le build utilise bien ton proxy.
* L’image finale **ne contient pas** ces infos sensibles.

***

👉 En résumé :

* `ENV` = variable **stockée** dans l’image → ❌ à éviter pour les proxies.
* `ARG` = variable temporaire pour le **build seulement** → ✅ recommandé.
