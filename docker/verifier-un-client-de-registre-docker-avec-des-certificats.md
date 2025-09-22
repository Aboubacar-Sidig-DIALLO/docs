# 🔐 Vérifier un client de registre Docker avec des certificats

Lorsque tu utilises un **registre Docker privé** avec **HTTPS**, il est essentiel de t’assurer que :

1. ✅ Le registre est **authentique** (certificat signé par une autorité de confiance).
2. ✅ La communication entre le **daemon Docker (client)** et le **registre (serveur)** est **chiffrée et authentifiée**.
3. ✅ Seuls les clients disposant d’un **certificat valide** peuvent interagir avec le registre.

Ceci se fait via **TLS mutuel (mTLS)** :

* Le **registre** présente son certificat au client.
* Le **client Docker** présente son certificat au registre.
* Chacun valide le certificat de l’autre grâce à l’**autorité de certification (CA)**.

***

### 📌 Étape 1 : Créer une autorité de certification (CA)

Sur la machine du registre :

```bash
openssl genrsa -out ca.key 4096
openssl req -x509 -new -nodes -key ca.key -days 365 -out ca.crt \
  -subj "/CN=MyDockerRegistry"
```

👉 `ca.crt` sera le certificat racine à distribuer aux clients.

***

### 📌 Étape 2 : Générer le certificat serveur (registre)

Toujours sur le registre :

```bash
openssl genrsa -out registry.key 4096
openssl req -new -key registry.key -out registry.csr -subj "/CN=my-registry.example.com"

# Crée un fichier pour les extensions
echo subjectAltName = DNS:my-registry.example.com,IP:127.0.0.1 > extfile.cnf
echo extendedKeyUsage = serverAuth >> extfile.cnf

# Signe avec le CA
openssl x509 -req -in registry.csr -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out registry.crt -days 365 -sha256 -extfile extfile.cnf
```

👉 Tu obtiens `registry.crt` (serveur) et `registry.key` (privée).

***

### 📌 Étape 3 : Générer un certificat client (Docker daemon)

Sur la machine client (où tourne Docker) :

```bash
openssl genrsa -out client.key 4096
openssl req -new -key client.key -out client.csr -subj "/CN=docker-client"

# Crée un fichier pour extensions client
echo extendedKeyUsage = clientAuth > extfile-client.cnf

# Signe le certificat client avec le CA
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out client.crt -days 365 -sha256 -extfile extfile-client.cnf
```

👉 Tu obtiens `client.crt` et `client.key`.

***

### 📌 Étape 4 : Configurer le registre avec TLS

Si tu déploies un registre Docker (par exemple via `registry:2` en Docker) :

```yaml
version: "3"
services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
    environment:
      REGISTRY_HTTP_TLS_CERTIFICATE: /certs/registry.crt
      REGISTRY_HTTP_TLS_KEY: /certs/registry.key
    volumes:
      - ./certs:/certs
      - ./data:/var/lib/registry
```

👉 Place `registry.crt` et `registry.key` dans `./certs`.

***

### 📌 Étape 5 : Configurer Docker client

Sur le client Docker, place les certificats dans :

```
/etc/docker/certs.d/my-registry.example.com:5000/
```

Avec les fichiers suivants :

* `ca.crt` → certificat CA
* `client.crt` → certificat client
* `client.key` → clé privée du client

Exemple :

```bash
sudo mkdir -p /etc/docker/certs.d/my-registry.example.com:5000/
sudo cp ca.crt client.crt client.key /etc/docker/certs.d/my-registry.example.com:5000/
```

***

### 📌 Étape 6 : Vérifier la connexion

Teste en listant les images du registre :

```bash
docker --tlsverify \
  --tlscacert=/etc/docker/certs.d/my-registry.example.com:5000/ca.crt \
  --tlscert=/etc/docker/certs.d/my-registry.example.com:5000/client.crt \
  --tlskey=/etc/docker/certs.d/my-registry.example.com:5000/client.key \
  -H=my-registry.example.com:2376 info
```

Ou directement :

```bash
docker pull my-registry.example.com:5000/hello-world
```

***

✅ Résultat attendu :

* La connexion échoue si le client **n’a pas de certificat valide**.
* La connexion échoue si le **serveur** n’a pas de certificat signé par le CA.
* La communication est **chiffrée et authentifiée**.

## 🔐 Comprendre la configuration des certificats Docker

### 📂 Où placer les certificats ?

Docker s’attend à trouver les certificats dans un répertoire spécifique :

```
/etc/docker/certs.d/<hostname>:<port>/
```

👉 `<hostname>:<port>` doit correspondre **exactement** à l’adresse de ton registre (ex. `localhost:5000`, `my-registry.example.com:443`).

***

### 📌 Quels fichiers placer dans ce répertoire ?

* **`ca.crt`** → le certificat de l’**autorité de certification (CA)** qui a signé le certificat du registre.\
  🔹 Sert à vérifier que le registre est bien authentique.
* **`client.cert`** → certificat public du **client Docker**.\
  🔹 Utilisé pour l’**authentification mutuelle (mTLS)** côté client.
* **`client.key`** → clé privée du client Docker.\
  🔹 Toujours gardée **secrète** et protégée (chmod `0400`).

***

### 📌 Exemple concret

Arborescence pour un registre en **localhost:5000** :

```
/etc/docker/certs.d/
└── localhost:5000
    ├── client.cert   # certificat du client
    ├── client.key    # clé privée du client
    └── ca.crt        # certificat racine (CA) qui signe le registre
```

***

### 📌 Points importants à savoir

1. **Fusion avec les certificats système (Linux uniquement)**
   * Sur **Linux** : Docker combine ces certificats personnalisés avec ceux déjà présents dans le système (ex. `/etc/ssl/certs`).
   * Sur **Windows Server** ou **Docker Desktop (Windows containers)** : si tu configures un certificat personnalisé, Docker **ignore les CAs système** → seuls ceux du répertoire Docker sont utilisés.

***

2. **Plusieurs certificats possibles**
   * Tu peux avoir plusieurs paires (`client.cert` / `client.key`) dans le dossier.
   * Docker les teste **dans l’ordre alphabétique**.
   * Si un certificat renvoie une erreur **4xx ou 5xx**, Docker essaie le suivant.

***

### 📌 Exemple de fonctionnement

* Tu démarres un registre local sécurisé avec TLS sur `localhost:5000`.
* Tu configures `/etc/docker/certs.d/localhost:5000/ca.crt` → Docker saura faire confiance au registre.
* Tu ajoutes `client.cert` et `client.key` → le registre pourra **authentifier le client**.
* Quand tu feras un `docker pull localhost:5000/monimage`, la communication sera :
  * **chiffrée (TLS)** 🔒
  * **authentifiée (CA + certificats client/serveur)** ✅

## 🛠️ Conseils de dépannage — certificats Docker

#### 1️⃣ Vérifier les extensions de fichiers

* **`.crt`** → réservé aux certificats **CA (autorité de certification)**.
* **`.cert`** → réservé aux certificats **client**.

👉 Si tu mets une CA avec l’extension `.cert`, Docker affichera une erreur comme :

```
Missing key KEY_NAME for client certificate CERT_NAME. 
CA certificates should use the extension .crt.
```

***

#### 2️⃣ Respecter le nommage des répertoires

Le chemin attendu est basé sur le **hostname exact du registre** :

* Si ton registre écoute sur le port **par défaut 443 (HTTPS)** → **ne pas inclure le port** dans le nom du dossier.\
  Exemple :

```
/etc/docker/certs.d/
└── my-https.registry.example.com
    ├── client.cert
    ├── client.key
    └── ca.crt
```

* Si ton registre utilise un port personnalisé (ex. `5000`), il faut l’inclure :

```
/etc/docker/certs.d/
└── my-https.registry.example.com:5000
    ├── client.cert
    ├── client.key
    └── ca.crt
```

***

#### 3️⃣ Vérifier les droits des fichiers 🔒

*   Les **clés privées (`*.key`)** doivent être protégées :

    ```bash
    chmod 0400 client.key
    ```
*   Les **certificats (`*.crt` / `*.cert`)** peuvent être en lecture publique :

    ```bash
    chmod 0444 client.cert ca.crt
    ```

***

#### 4️⃣ Vérifier les logs du démon Docker

Si quelque chose échoue :

```bash
sudo journalctl -u docker.service -f
```

ou

```bash
dockerd --debug
```

Cela montre si le certificat est mal interprété, manquant ou invalide.

***

#### 5️⃣ Vérifier avec `openssl`

Tu peux tester si tes certificats correspondent bien avec :

```bash
openssl x509 -in client.cert -noout -text
openssl rsa -in client.key -check
```

***

🔎 **Bon à savoir :**

* Le certificat client doit être signé par la même CA que celle configurée dans `ca.crt`.
* Docker testera **tous les certificats présents** dans le répertoire, dans l’ordre alphabétique, jusqu’à trouver un qui marche.
