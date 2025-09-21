# ⚙️ Configuration du proxy pour le démon Docker

### 1️⃣ Méthodes possibles

Tu as deux façons de configurer un proxy pour `dockerd` :

1. **Via fichier de configuration (`daemon.json`) ou via flags de lancement**\
   → C’est la méthode la plus propre car centralisée.
2. **Via variables d’environnement au niveau système**\
   → Plus simple à mettre en place rapidement.

👉 **Priorité** :\
Configuration explicite dans `daemon.json` > Variables d’environnement système.

***

### 2️⃣ Configuration via fichier `daemon.json`

Chemin du fichier selon ton OS :

* Linux : `/etc/docker/daemon.json`
* Rootless : `~/.config/docker/daemon.json`
* Windows : `C:\ProgramData\docker\config\daemon.json`

Exemple :

```json
{
  "proxies": {
    "default": {
      "httpProxy": "http://proxy.example.com:3128",
      "httpsProxy": "https://proxy.example.com:3129",
      "noProxy": "*.internal.example.com,127.0.0.1,localhost"
    }
  }
}
```

⚠️ Attention : ces paramètres ne fonctionnent **pas sur Docker Desktop**, qui a ses propres réglages de proxy.

***

### 3️⃣ Configuration via variables d’environnement système

Sur Linux (exemple avec `systemd`) :

Crée/modifie `/etc/systemd/system/docker.service.d/proxy.conf` :

```ini
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:3128"
Environment="HTTPS_PROXY=https://proxy.example.com:3129"
Environment="NO_PROXY=*.internal.example.com,127.0.0.1,localhost"
```

Puis recharge systemd et redémarre Docker :

```bash
sudo systemctl daemon-reexec
sudo systemctl restart docker
```

***

### 4️⃣ Vérification

Teste que le démon utilise bien le proxy :

```bash
docker pull alpine
```

Vérifie les variables proxy appliquées :

```bash
systemctl show --property=Environment docker
```

***

### 🔑 À retenir

* **daemon.json** = méthode recommandée.
* **systemd (ou équivalent)** si tu veux gérer via variables d’environnement.
* **Docker Desktop → ne prend pas en compte `daemon.json`** → utilise l’UI.

### 🔧 Configuration du proxy dans `daemon.json`

Chemin du fichier :

* **Linux** : `/etc/docker/daemon.json`
* **Rootless** : `~/.config/docker/daemon.json`
* **Windows** : `C:\ProgramData\docker\config\daemon.json`

Exemple de configuration avec un proxy HTTP/HTTPS et une exception `no-proxy` :

```json
{
  "proxies": {
    "http-proxy": "http://proxy.example.com:3128",
    "https-proxy": "https://proxy.example.com:3129",
    "no-proxy": "*.test.example.com,.example.org,127.0.0.0/8"
  }
}
```

***

### 🚀 Appliquer les changements

Une fois le fichier modifié, redémarre le démon Docker pour appliquer les réglages :

```bash
sudo systemctl restart docker
```

***

### ✅ Points importants

* `http-proxy` : proxy pour les connexions HTTP.
* `https-proxy` : proxy pour les connexions HTTPS.
* `no-proxy` : liste d’adresses exclues (séparées par des virgules).

⚠️ Sur **Docker Desktop** : `daemon.json` est **ignoré** → il faut configurer via l’UI de Docker Desktop.

### 🌍 Variables d’environnement supportées par `dockerd`

* `HTTP_PROXY` ou `http_proxy` → définit le proxy pour les connexions HTTP
* `HTTPS_PROXY` ou `https_proxy` → définit le proxy pour les connexions HTTPS
* `NO_PROXY` ou `no_proxy` → définit les exceptions (adresses qui ne passent pas par le proxy)

***

### 🔧 Exemple sur Linux avec `systemd`

Éditer le fichier d’override pour le service `docker` :

```bash
sudo systemctl edit docker
```

Ajouter :

```ini
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:3128/"
Environment="HTTPS_PROXY=https://proxy.example.com:3129/"
Environment="NO_PROXY=localhost,127.0.0.1,.example.org"
```

***

### 🚀 Appliquer les changements

```bash
sudo systemctl daemon-reexec
sudo systemctl restart docker
```

***

### ⚠️ Points importants

* Ces variables **ne s’appliquent pas automatiquement aux conteneurs** → elles concernent uniquement le **démon Docker**.
* Les conteneurs, eux, héritent du proxy uniquement si tu l’as défini dans `~/.docker/config.json` ou avec `docker run -e HTTP_PROXY=...`.

## ⚙️ systemd unit file — Configurer un proxy pour le démon Docker

Lorsque le démon **Docker** (`dockerd`) est exécuté comme un service **systemd**, il est possible de créer un **fichier drop-in** pour définir des variables d’environnement (HTTP/HTTPS proxy, NO\_PROXY, etc.).

***

### 📌 Cas classiques

#### 1. Créer un dossier de configuration drop-in

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
```

#### 2. Créer le fichier `http-proxy.conf`

Exemple avec un proxy HTTP :

```ini
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:3128"
```

Pour un proxy HTTPS :

```ini
[Service]
Environment="HTTPS_PROXY=https://proxy.example.com:3129"
```

Pour les deux en même temps :

```ini
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:3128"
Environment="HTTPS_PROXY=https://proxy.example.com:3129"
```

***

### 🔐 Gestion des caractères spéciaux

Si l’URL du proxy contient des caractères spéciaux (`#?!()[]{}` etc.), il faut les **doubler avec %%**.\
Exemple :

```ini
[Service]
Environment="HTTP_PROXY=http://domain%%5Cuser:complex%%23pass@proxy.example.com:3128/"
```

***

### 🚫 Exclure certaines adresses du proxy (NO\_PROXY)

La variable `NO_PROXY` permet de **ne pas utiliser le proxy** pour certains hôtes/domains :

```ini
[Service]
Environment="NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp"
```

🔹 Règles utiles :

* `example.com` → correspond à `example.com` et `foo.example.com`
* `.example.com` → correspond uniquement à `foo.example.com`
* `*` → désactive tout proxy
* `1.2.3.4:80` ou `host:80` → inclut les ports spécifiques

***

### 🔄 Appliquer et vérifier

Après modification :

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

Vérifier que la config est bien prise en compte :

```bash
sudo systemctl show --property=Environment docker
```

***

### 📌 Mode rootless

Si Docker est exécuté en **rootless mode**, les fichiers systemd sont stockés dans :

```
~/.config/systemd/user/docker.service.d/
```

👉 Dans ce cas, utilise `systemctl --user` (sans sudo).
