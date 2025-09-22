# 🔒 Protection du socket Docker daemon

Le **Docker daemon (`dockerd`)** est le cœur de Docker : il gère les conteneurs, les images, les volumes et le réseau.\
Il expose une API (Docker Remote API) que les clients (ex. `docker CLI`, `Docker Compose`) utilisent pour communiquer avec lui.\
👉 Par défaut, cette communication se fait via un **socket UNIX local**.

***

### ⚙️ Modes de communication possibles

1. **Socket UNIX (par défaut)**
   * Localisation : `/var/run/docker.sock`
   * Accessible uniquement par **l’utilisateur root** et les membres du groupe `docker`.
   * Avantage : pas exposé au réseau → surface d’attaque réduite.
   * ⚠️ Risque : tout utilisateur du groupe `docker` a les **mêmes privilèges que root**, car il peut lancer des conteneurs avec accès complet au système.

***

2. **Socket TCP avec TLS (HTTPS sécurisé)**
   * Permet d’exposer l’API Docker à distance (ex. pour un cluster, un CI/CD).
   * Nécessite **certificats TLS** (CA, clé privée, certificat serveur).
   *   Exemple d’activation dans `/etc/docker/daemon.json` :

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
   *   Client :

       ```bash
       docker --tlsverify \
         --tlscacert=ca.pem \
         --tlscert=cert.pem \
         --tlskey=key.pem \
         -H=tcp://SERVER_IP:2376 ps
       ```

***

3. **Connexion via SSH**
   * Pas besoin d’ouvrir un port TCP dédié.
   * Fonctionne avec un serveur SSH déjà en place.
   *   Exemple :

       ```bash
       docker -H ssh://user@remote-docker-host ps
       ```
   * Avantage : sécurité SSH existante (clé privée/publique, gestion des accès, bastion).

***

### 🚨 Risques principaux

* **Exposition du socket non sécurisé (`tcp://0.0.0.0:2375`)** :
  * ⚠️ **Jamais** activer sans TLS !
  * Sinon, n’importe qui peut exécuter des conteneurs root sur ta machine → compromission totale.
* **Ajout d’utilisateurs au groupe `docker`** :
  * Ces utilisateurs obtiennent indirectement **des privilèges root complets**.
  *   Exemple :

      ```bash
      docker run -v /:/host -it alpine chroot /host
      ```

      → accès total au système hôte.

***

### ✅ Bonnes pratiques de protection

1. **Ne jamais exposer Docker en clair** (`tcp://0.0.0.0:2375`).
2. **Limiter l’accès au socket UNIX** :
   * Restreindre le groupe `docker` aux utilisateurs de confiance.
   *   Vérifier les permissions :

       ```bash
       ls -l /var/run/docker.sock
       ```
3. **Utiliser TLS** pour un daemon accessible en réseau.
4. **Préférer SSH** si l’accès distant est ponctuel ou réservé à quelques utilisateurs.
5. **Surveiller l’accès** : journaliser les commandes Docker (auditd, journald).
6. **En production critique** : utiliser un **proxy/API gateway** devant Docker pour filtrer les appels.

## 🔐 Utiliser SSH pour protéger le socket Docker daemon

L’un des moyens les plus sûrs d’accéder à un **Docker daemon distant** est d’utiliser **SSH** plutôt qu’un socket TCP exposé.\
Ainsi, tu bénéficies de la sécurité de SSH (authentification par clé, chiffrement, contrôle d’accès) sans avoir à gérer de certificats TLS manuellement.

***

### ⚙️ Pré-requis

* Le **daemon Docker** (`dockerd`) est installé et actif sur la machine distante.
* L’utilisateur (`docker-user` dans l’exemple) doit avoir accès au socket Docker local (`/var/run/docker.sock`) → voir gestion de Docker en tant qu’utilisateur non-root.
*   Tu peux déjà te connecter en SSH à la machine distante :

    ```bash
    ssh docker-user@host1.example.com
    ```

***

### 🔹 Créer un contexte Docker via SSH

```bash
docker context create \
  --docker host=ssh://docker-user@host1.example.com \
  --description="Remote engine" \
  my-remote-engine
```

📌 Explications :

* `--docker host=ssh://docker-user@host1.example.com` → définit la connexion via SSH.
* `--description` → ajout d’une description pour identifier le contexte.
* `my-remote-engine` → nom attribué au contexte.

***

### 🔹 Utiliser le contexte

Bascule sur le contexte que tu viens de créer :

```bash
docker context use my-remote-engine
```

Vérifie que tu es bien connecté au daemon distant :

```bash
docker info
```

🔄 Pour revenir au daemon local par défaut :

```bash
docker context use default
```

***

### 🔹 Connexion ad-hoc via la variable d’environnement

Si tu veux tester rapidement sans créer de contexte :

```bash
export DOCKER_HOST=ssh://docker-user@host1.example.com
docker info
```

👉 Dès que tu fermes le terminal ou réinitialises la variable, Docker revient à la configuration locale.

***

### 💡 Astuce SSH : connexion persistante

Pour éviter d’ouvrir une nouvelle session SSH à chaque commande Docker, configure `~/.ssh/config` :

```ssh-config
Host host1.example.com
  User docker-user
  ControlMaster auto
  ControlPath ~/.ssh/control-%C
  ControlPersist yes
```

Ainsi, la connexion SSH est réutilisée, ce qui accélère les commandes Docker successives.

***

✅ Avec cette méthode :

* Pas besoin d’exposer un port réseau (`2375` ou `2376`).
* Pas de risque de laisser un daemon accessible en clair.
* Tu profites de la **sécurité éprouvée de SSH**.

## 🔐 Utiliser TLS (HTTPS) pour protéger le socket Docker daemon

Si tu veux rendre ton **daemon Docker accessible via HTTP(S)** plutôt que SSH, tu dois **activer TLS**. Cela permet de sécuriser les communications et d’exiger une authentification par certificats (clients et serveur).

***

### ⚙️ Principe

* **Côté daemon (`dockerd`)** :
  * On active `--tlsverify`.
  * On fournit un certificat signé par une **Autorité de Certification (CA)** de confiance.
  * Seuls les clients disposant d’un certificat valide signé par cette CA pourront se connecter.
* **Côté client (`docker CLI`)** :
  * On configure `--tlsverify`.
  * On fournit son certificat client signé par la même CA.
  * On valide le certificat du serveur.

👉 Cela met en place une **authentification mutuelle (mTLS)** : serveur ↔ client.

***

### 🔹 Étape 1 : Créer une CA

Sur la machine qui hébergera le daemon :

```bash
openssl genrsa -aes256 -out ca-key.pem 4096
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
```

* `ca-key.pem` → clé privée (à protéger strictement).
* `ca.pem` → certificat public de la CA (servira à signer les certificats serveur/clients).

***

### 🔹 Étape 2 : Créer un certificat serveur

```bash
openssl genrsa -out server-key.pem 4096
openssl req -subj "/CN=$HOST" -sha256 -new -key server-key.pem -out server.csr
```

Ajoute les adresses IP/DNS autorisées :

```bash
echo subjectAltName = DNS:$HOST,IP:10.10.10.20,IP:127.0.0.1 >> extfile.cnf
echo extendedKeyUsage = serverAuth >> extfile.cnf
```

Signe le certificat :

```bash
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out server-cert.pem -extfile extfile.cnf
```

***

### 🔹 Étape 3 : Créer un certificat client

```bash
openssl genrsa -out key.pem 4096
openssl req -subj '/CN=client' -new -key key.pem -out client.csr
echo extendedKeyUsage = clientAuth > extfile-client.cnf
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out cert.pem -extfile extfile-client.cnf
```

***

### 🔹 Étape 4 : Sécuriser les permissions des fichiers

```bash
chmod 0400 ca-key.pem key.pem server-key.pem
chmod 0444 ca.pem server-cert.pem cert.pem
```

***

### 🔹 Étape 5 : Lancer le daemon avec TLS

```bash
dockerd \
  --tlsverify \
  --tlscacert=ca.pem \
  --tlscert=server-cert.pem \
  --tlskey=server-key.pem \
  -H=0.0.0.0:2376
```

👉 `-H=0.0.0.0:2376` → ouvre le daemon sur le port TLS standard de Docker (`2376`).

***

### 🔹 Étape 6 : Se connecter côté client

Copie **ca.pem**, **cert.pem** et **key.pem** sur la machine cliente, puis exécute :

```bash
docker --tlsverify \
  --tlscacert=ca.pem \
  --tlscert=cert.pem \
  --tlskey=key.pem \
  -H=$HOST:2376 version
```

***

### ⚠️ Points importants

* **Sécurité critique** : quiconque possède `key.pem` + `cert.pem` peut agir comme **root** sur ton hôte Docker.
* **Port recommandé** : `2376` (jamais `2375` car c’est **non sécurisé**).
* **Gestion CA** : nécessite une bonne compréhension de TLS, OpenSSL, et des certificats x509.
* **Alternative plus simple** : utiliser SSH (pas de gestion manuelle des certificats).

## 🔐 Sécuriser Docker **par défaut** avec TLS

Normalement, tu dois lancer chaque commande Docker avec `--tlsverify`, `--tlscert`, `--tlskey`, `--tlscacert` et `-H=...`.\
👉 Pour éviter ça et **rendre la connexion sécurisée par défaut**, tu peux configurer ton environnement.

***

### 📂 Étape 1 : Organiser les certificats

Crée un dossier standard pour Docker dans ton home et place-y les fichiers :

```bash
mkdir -pv ~/.docker
cp -v {ca,cert,key}.pem ~/.docker
```

Ainsi Docker saura où chercher les certificats **par défaut**.

***

### 🔹 Étape 2 : Exporter les variables d’environnement

Ajoute (dans `~/.bashrc`, `~/.zshrc` ou équivalent) :

```bash
export DOCKER_HOST=tcp://$HOST:2376
export DOCKER_TLS_VERIFY=1
```

* `DOCKER_HOST` → indique l’endpoint sécurisé (TLS).
* `DOCKER_TLS_VERIFY=1` → force Docker à utiliser TLS avec vérification.

Redémarre ton shell, puis vérifie :

```bash
docker ps
```

👉 Tu es maintenant connecté **sécurisé par défaut**, sans options manuelles.

***

### 🔹 Étape 3 : Modes de fonctionnement TLS possibles

Selon les **flags utilisés**, tu peux activer différents niveaux de sécurité :

#### 🚀 Côté **daemon** (`dockerd`)

* `--tlsverify --tlscacert --tlscert --tlskey` → **authentifie les clients** (mTLS complet ✅ recommandé).
* `--tls --tlscert --tlskey` → chiffre le trafic, **n’authentifie pas les clients** (⚠️ vulnérable aux intrusions).

#### 🚀 Côté **client** (`docker`)

* `--tls` → vérifie uniquement le serveur avec la CA système par défaut.
* `--tlsverify --tlscacert` → vérifie le serveur avec une CA spécifique.
* `--tls --tlscert --tlskey` → le client **s’authentifie** mais ne vérifie pas le serveur (⚠️ MITM possible).
* `--tlsverify --tlscacert --tlscert --tlskey` → **authentification mutuelle** (mTLS complet ✅).

***

### 🔹 Étape 4 : Utiliser un autre chemin pour les certificats

Si tu stockes les certificats ailleurs, configure :

```bash
export DOCKER_CERT_PATH=~/.docker/zone1/
docker --tlsverify ps
```

***

### 🔹 Étape 5 : Tester avec `curl`

Tu peux tester l’API Docker sécurisée avec TLS directement via `curl` :

```bash
curl https://$HOST:2376/images/json \
  --cert ~/.docker/cert.pem \
  --key ~/.docker/key.pem \
  --cacert ~/.docker/ca.pem
```

***

✅ Résultat : ton **client Docker est sécurisé par défaut** et toutes les commandes utilisent TLS.\
⚠️ Attention : **quiconque possède `cert.pem` + `key.pem` a un accès root à ton hôte Docker** → protège-les comme un mot de passe root.
