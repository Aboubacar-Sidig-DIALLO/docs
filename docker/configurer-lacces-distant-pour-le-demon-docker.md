# 🌐 Configurer l’accès distant pour le démon Docker

Par défaut, le démon **Docker (`dockerd`)** écoute uniquement sur un **socket Unix** (`/var/run/docker.sock`) et n’accepte que les connexions locales.\
➡️ Tu peux cependant activer l’**accès distant** en le configurant pour écouter aussi sur une **adresse IP + port**.

⚠️ **Attention :** activer l’accès réseau expose ton hôte Docker à des risques importants (élévation de privilèges, accès root distant, attaques réseau). Il est donc **vivement recommandé de sécuriser la connexion avec TLS**.

***

### 🔎 Étape 1 : Modifier la configuration du démon

Édite (ou crée) le fichier `/etc/docker/daemon.json` :

```json
{
  "hosts": [
    "unix:///var/run/docker.sock",
    "tcp://0.0.0.0:2375"
  ]
}
```

* `unix:///var/run/docker.sock` → garde l’accès local.
* `tcp://0.0.0.0:2375` → écoute sur toutes les interfaces réseau, port **2375** (non sécurisé).
* Tu peux remplacer `0.0.0.0` par une IP spécifique pour limiter l’accès (ex. `tcp://192.168.1.100:2375`).

***

### 🔎 Étape 2 : Redémarrer Docker

Recharge la configuration et redémarre le démon :

```bash
sudo systemctl daemon-reexec
sudo systemctl restart docker
```

***

### 🔎 Étape 3 : Tester la connexion distante

Depuis un client distant (ayant Docker installé), exécute :

```bash
docker -H tcp://192.168.1.100:2375 info
```

Cela retournera les informations de ton démon Docker distant.

***

### ⚠️ Sécurité : à ne pas négliger

* **2375 sans TLS = connexion non sécurisée (HTTP simple)** → dangereux 🚨.
* Pour sécuriser, utilise **TLS** avec le port **2376** :
  * Génère un certificat serveur et client.
  *   Configure `dockerd` avec :

      ```json
      {
        "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2376"],
        "tls": true,
        "tlscacert": "/etc/docker/ca.pem",
        "tlscert": "/etc/docker/server-cert.pem",
        "tlskey": "/etc/docker/server-key.pem",
        "tlsverify": true
      }
      ```
*   Sur le client, utilise :

    ```bash
    docker --tlsverify \
      --tlscacert=ca.pem \
      --tlscert=cert.pem \
      --tlskey=key.pem \
      -H tcp://192.168.1.100:2376 info
    ```

## 🔐 Activer l’accès distant au démon Docker (via **systemd**)

Tu peux activer l’accès distant à Docker en configurant son **service systemd** (`docker.service`).\
⚠️ Important : il faut choisir **soit systemd**, **soit `daemon.json`** (mais pas les deux), sinon Docker refusera de démarrer.

***

### 🛠️ Étapes de configuration avec systemd

#### 1. Créer un fichier d’override pour Docker

Exécute :

```bash
sudo systemctl edit docker.service
```

Cela ouvre un fichier d’override dans ton éditeur (ex. `nano` ou `vi`).

***

#### 2. Ajouter ou modifier la directive `ExecStart`

Colle ceci :

```ini
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375
```

📌 Explications :

* `ExecStart=` → vide l’ancienne configuration.
* `-H fd://` → conserve la gestion par socket Unix.
* `-H tcp://127.0.0.1:2375` → active l’écoute réseau sur **localhost:2375** (accès distant uniquement local).

👉 Si tu veux exposer Docker sur toutes les interfaces :

```ini
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375
```

⚠️ **Très dangereux en production** sans TLS → tout le monde peut accéder à ton démon Docker.

***

#### 3. Recharger systemd

Applique la nouvelle config :

```bash
sudo systemctl daemon-reload
```

***

#### 4. Redémarrer Docker

Relance le service :

```bash
sudo systemctl restart docker.service
```

***

#### 5. Vérifier l’écoute du port

Teste si Docker écoute bien sur le port **2375** :

```bash
sudo netstat -lntp | grep dockerd
```

Exemple de sortie attendue :

```
tcp   0   0 127.0.0.1:2375   0.0.0.0:*   LISTEN   1234/dockerd
```

***

✅ Ton démon Docker accepte maintenant les connexions sur le port configuré.

## 🌐 Configurer l’accès distant avec `daemon.json`

En plus de la méthode **systemd**, tu peux configurer l’accès distant directement via le fichier de configuration **`/etc/docker/daemon.json`**.

***

### 🛠️ Étapes de configuration avec `daemon.json`

#### 1. Modifier le fichier `daemon.json`

Édite ou crée le fichier :

```bash
sudo nano /etc/docker/daemon.json
```

Ajoute ou complète la section `hosts` :

```json
{
  "hosts": ["unix:///var/run/docker.sock", "tcp://127.0.0.1:2375"]
}
```

📌 Explications :

* `unix:///var/run/docker.sock` → garde le socket local classique.
* `tcp://127.0.0.1:2375` → active l’écoute réseau en **local** sur le port 2375.
* Tu peux remplacer `127.0.0.1` par `0.0.0.0` si tu veux accepter des connexions depuis d’autres machines (⚠️ dangereux sans TLS).

***

#### 2. Redémarrer Docker

Recharge la configuration :

```bash
sudo systemctl restart docker
```

***

#### 3. Vérifier que Docker écoute bien sur le port

Teste avec :

```bash
sudo netstat -lntp | grep dockerd
```

Résultat attendu :

```
tcp   0   0 127.0.0.1:2375   0.0.0.0:*   LISTEN   3758/dockerd
```

***

### 🔥 Configurer le pare-feu (si accès externe activé)

Si tu ouvres l’accès réseau (`0.0.0.0`), il faut configurer ton pare-feu pour autoriser le port **2375** (non sécurisé) ou **2376** (sécurisé avec TLS).

#### 🔹 Avec **ufw** (Ubuntu)

Modifie la config :

```bash
sudo nano /etc/default/ufw
```

Change la ligne :

```
DEFAULT_FORWARD_POLICY="ACCEPT"
```

Puis autorise le port :

```bash
sudo ufw allow 2375/tcp
```

***

#### 🔹 Avec **firewalld** (CentOS, Fedora, RHEL)

Ajoute des règles :

```xml
<direct>
  [ <rule ipv="ipv6" table="filter" chain="FORWARD_direct" priority="0"> -i zt0 -j ACCEPT </rule> ]
  [ <rule ipv="ipv6" table="filter" chain="FORWARD_direct" priority="0"> -o zt0 -j ACCEPT </rule> ]
</direct>
```

👉 Remplace `zt0` par le nom de ton interface réseau.

***

⚠️ **Attention** :

* **2375 = NON sécurisé** (pas de TLS).
* **2376 = sécurisé avec TLS** (recommandé en production).
