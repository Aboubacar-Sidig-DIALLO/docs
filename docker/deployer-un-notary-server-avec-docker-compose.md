# 🚀 Déployer un Notary Server avec Docker Compose

### 1. Prérequis

* Avoir installé **Docker** et **Docker Compose**.
* Avoir `git` pour cloner le dépôt officiel.

***

### 2. Cloner le dépôt officiel

Le code source du serveur Notary se trouve sur GitHub :

```bash
git clone https://github.com/theupdateframework/notary.git
cd notary
```

***

### 3. Lancer le serveur avec Compose

Le dépôt contient déjà un fichier `docker-compose.yml` prêt à l’emploi (avec des certificats d’exemple).

Démarre le service en arrière-plan :

```bash
docker compose up -d
```

👉 Cela va :

* Construire les images nécessaires.
* Lancer **Notary Server** et **Notary Signer** avec des certificats de test.
* Exposer les endpoints nécessaires pour interagir avec le client Docker ou le client Notary CLI.

***

### 4. Faire confiance au certificat

⚠️ Le client Docker (ou Notary CLI) doit **faire confiance au certificat TLS** du serveur Notary avant toute interaction.

* Si tu utilises Docker :
  * Place le certificat racine dans `/etc/docker/certs.d/<notary-host>:<port>/ca.crt`
* Si tu utilises **Notary CLI** :
  * Ajoute le certificat dans le fichier `~/.notary/config.json`.

Exemple de config minimal Notary CLI (`~/.notary/config.json`) :

```json
{
  "trust_dir" : "~/.docker/trust",
  "remote_server": {
    "url": "https://<notary-server>:4443",
    "root_ca": "/path/to/ca.pem"
  }
}
```

***

### 5. Tester le serveur

Une fois le serveur lancé et configuré :

```bash
notary status https://<notary-server>:4443 <image>
```

ou depuis Docker (si DCT activé) :

```bash
export DOCKER_CONTENT_TRUST=1
docker pull <registry>/<repo>:<tag>
```

***

### 6. Utilisation en production

⚠️ Le `docker-compose.yml` du dépôt Notary utilise des certificats d’exemple → **ne pas utiliser tel quel en prod**.

En production, il faut :

* Fournir ses propres certificats TLS valides.
* Sécuriser le stockage des clés.
* Déployer avec un orchestrateur (Swarm, Kubernetes…).

👉 Documentation détaillée : [Notary Repository](https://github.com/theupdateframework/notary).

***

✅ Résumé :

* `git clone` du repo officiel.
* `docker compose up -d` pour lancer rapidement.
* Ajouter le certificat au client Docker ou Notary.
* Utiliser avec `docker trust` ou `notary`.
* Pour la prod → générer et gérer ses propres certificats.
