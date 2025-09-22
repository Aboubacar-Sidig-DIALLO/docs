# 🏖️ Sandbox Docker Content Trust

### 🎯 Objectif

Le **sandbox** te permet de :

* expérimenter **localement** avec DCT (signatures, révocations, délégations),
* tester la signature et la vérification d’images,
* apprendre à manipuler les clés (root, repository, delegation)\
  ➡️ **sans risque pour tes images de production**.

***

### ✅ Prérequis

1. Système : **Linux ou macOS** (VM ou machine locale).
2. Accès avec privilèges `docker` (pas besoin de root si Docker est configuré en rootless).
3. Versions minimales :
   * **Docker Engine ≥ 1.10.0**
   * **Docker Compose ≥ 1.6.0**
4. Installation :
   * Installer Docker Engine (choisir ta plateforme)
   * Installer Docker Compose

***

### ⚙️ Étapes de mise en place

#### 1. Activer Content Trust

Active le mode signature Docker en exportant la variable d’environnement :

```bash
export DOCKER_CONTENT_TRUST=1
```

👉 Avec cette option, toutes les commandes `docker push`, `docker pull`, `docker build` exigent des images signées.\
Sans signature → la commande échoue.

***

#### 2. Préparer un registre sandbox

Tu as deux options :

* **Option 1 : utiliser Docker Hub (facile)**
* **Option 2 : déployer un registre local + serveur Notary**

👉 Pour le mode sandbox, l’option **registre local avec Notary** est la plus utilisée, car elle te permet de jouer avec toutes les fonctions (signatures, délégations, révocations).

**Exemple avec Docker Compose (registre + notary) :**

```yaml
version: "3"

services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
    environment:
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /var/lib/registry
    volumes:
      - ./data:/var/lib/registry

  notary:
    image: theupdateframework/notary_server
    ports:
      - "4443:4443"
    environment:
      - NOTARY_SERVER_STORAGE=memory
      - NOTARY_SERVER_TRUST_SERVICE=remote
    depends_on:
      - registry
```

Lance ton sandbox avec :

```bash
docker compose up -d
```

Ton **registre local** est dispo sur `localhost:5000`\
et ton **notary server** sur `localhost:4443`.

***

#### 3. Générer et gérer des clés

Quand tu signes une image avec DCT :

* Docker génère automatiquement :
  * **root key** (stockée localement, à sauvegarder),
  * **repository key** (pour signer les tags),
  * et Notary gère les **snapshot** et **timestamp keys** côté serveur.

Clés stockées localement :

```
~/.docker/trust/private
```

***

#### 4. Signer une image

1.  Crée une image simple :

    ```bash
    docker pull alpine:latest
    docker tag alpine:latest localhost:5000/demo:1
    ```
2.  Signe et pousse-la :

    ```bash
    docker trust sign localhost:5000/demo:1
    ```

👉 Tu devras entrer une **passphrase** pour la root key et la repository key.

***

#### 5. Vérifier les signatures

Inspecte les signatures d’une image :

```bash
docker trust inspect --pretty localhost:5000/demo:1
```

Exemple de sortie :

```
Signatures for localhost:5000/demo:1

SIGNED TAG          DIGEST                                                             SIGNERS
1                   3d2e482b82608d153a374df3357c0291589a61cc194ec4a9ca2381073a17f58e   jeff
```

***

#### 6. Tester la protection

* Essaie de `pull` une image non signée avec `DOCKER_CONTENT_TRUST=1`\
  👉 tu auras une **erreur**.
* Essaie avec un digest explicite (`@sha256:...`) → fonctionne toujours.
* Ajoute un signataire (`docker trust signer add`) et observe comment les délégations sont gérées.
* Révoque un signataire (`docker trust signer remove`) et vois comment la confiance est cassée.

***

### 📌 Résumé

Le **sandbox DCT** te permet :

* d’**apprendre à signer et vérifier des images**,
* de **tester la rotation et révocation des clés**,
* d’expérimenter avec un **registre + Notary local**,
* **sans toucher à la prod**.

### 🏖️ Qu’y a-t-il dans le **sandbox Content Trust** ?

Le **sandbox** est un environnement isolé qui reproduit les conditions d’un système de confiance en production, mais sans aucun impact sur vos vraies images Docker ou votre Docker Hub.

Il met en place tous les composants nécessaires pour **signer, vérifier et gérer des images Docker de manière sécurisée**, tout en restant dans un cadre d’expérimentation.

***

#### 🧱 Les composants du sandbox

1. **Conteneur `trustsandbox`**
   * C’est un **Docker-in-Docker (dind)**, c’est-à-dire un moteur Docker fonctionnant dans un conteneur.
   * Il contient déjà des **certificats et configurations de confiance préinstallés**.
   * Il sert de **client Docker expérimental** :
     * vous exécutez vos commandes `docker trust`,
     * vous poussez et tirez des images signées,
     * vous testez la vérification des signatures.
   * Les **clés de confiance** (racine, repository, délégation) sont stockées _uniquement dans ce conteneur_.\
     👉 Dès que vous détruisez le conteneur, toutes les clés disparaissent.

***

2. **Serveur de registre (Registry server)**
   * Un registre Docker local, équivalent à un mini **Docker Hub privé**.
   * C’est là que vous poussez vos images de test (`localhost:5000/...`).
   * Complètement séparé de votre vrai Docker Hub ou de vos registres de production.
   * Permet de tester des scénarios d’images **signées et non signées**.

***

3. **Serveur Notary**
   * Service responsable de la gestion des **métadonnées de confiance** :
     * vérification des signatures,
     * gestion des clés _snapshot_ et _timestamp_,
     * ajout/suppression de clés de délégation.
   * Dans un vrai environnement, Notary est intégré directement à **Docker Hub** ou **Docker Trusted Registry (DTR)**.
   * Ici, vous exécutez votre propre serveur Notary pour expérimenter sans risque.

***

#### 🔒 Pourquoi ce montage est utile ?

* **Clés éphémères** → elles existent uniquement dans le conteneur `trustsandbox`.
* **Images isolées** → elles ne vont pas sur Docker Hub, seulement dans votre registre local.
* **Aucune pollution** → grâce au Docker-in-Docker, votre démon Docker réel et son cache ne sont jamais touchés.

➡️ Quand vous arrêtez et supprimez le sandbox, **tout disparaît** (images, clés, métadonnées).

***

#### 📌 Résumé

Le **sandbox = mini-environnement de production simulé** avec :

| Composant      | Rôle                                            |
| -------------- | ----------------------------------------------- |
| `trustsandbox` | Client Docker isolé avec clés & certificats     |
| Registry local | Stocke vos images signées/non signées           |
| Serveur Notary | Gère les signatures et métadonnées de confiance |

👉 Si vous travaillez uniquement avec **Docker Hub**, vous n’avez pas besoin de monter ces services. Mais pour tester et apprendre, le sandbox est idéal.

#### 🧪 Construire le **Sandbox Docker Content Trust**

Voici les étapes pour mettre en place un **sandbox** complet qui relie :

* un **conteneur de test** (`trustsandbox`),
* un **serveur Notary** (pour gérer la confiance et les signatures),
* un **registre local** (pour héberger les images).

***

### Étape 1 – Créer le répertoire du sandbox

```bash
mkdir trustsandbox
cd trustsandbox
```

***

### Étape 2 – Créer le fichier `compose.yaml`

Avec ton éditeur préféré (ex. `vim`, `nano`) :

```bash
touch compose.yaml
vim compose.yaml
```

Ajoute le contenu suivant :

```yaml
version: "2"
services:
  notaryserver:
    image: dockersecurity/notary_autobuilds:server-v0.5.1
    volumes:
      - notarycerts:/var/lib/notary/fixtures
    networks:
      - sandbox
    environment:
      - NOTARY_SERVER_STORAGE_TYPE=memory
      - NOTARY_SERVER_TRUST_SERVICE_TYPE=local

  sandboxregistry:
    image: registry:2.4.1
    networks:
      - sandbox
    container_name: sandboxregistry

  trustsandbox:
    image: docker:dind
    networks:
      - sandbox
    volumes:
      - notarycerts:/notarycerts
    privileged: true
    container_name: trustsandbox
    entrypoint: ""
    command: |-
        sh -c '
            cp /notarycerts/root-ca.crt /usr/local/share/ca-certificates/root-ca.crt &&
            update-ca-certificates &&
            dockerd-entrypoint.sh --insecure-registry sandboxregistry:5000'

volumes:
  notarycerts:
    external: false

networks:
  sandbox:
    external: false
```

***

### Étape 3 – Lancer les services

Exécute la commande suivante :

```bash
docker compose up -d
```

👉 La première fois, Docker téléchargera les images suivantes :

* `docker:dind` (Docker-in-Docker),
* `dockersecurity/notary_autobuilds:server-v0.5.1` (serveur Notary),
* `registry:2.4.1` (registre local).

***

### Résultat attendu

Une fois lancé :

* Le **registre local** tourne sur `sandboxregistry:5000`.
* Le **serveur Notary** gère la signature et la vérification des images.
* Le **conteneur `trustsandbox`** agit comme un Docker client isolé (avec ses propres clés et cache d’images).

#### 🧪 Jouer dans le **sandbox Docker Content Trust**

Maintenant que ton **environnement sandbox** est installé (Notary + registre + trustsandbox), tu peux tester les opérations de **signature et de vérification d’images Docker**.

***

### Étape 1 – Entrer dans le conteneur `trustsandbox`

Depuis ton hôte :

```bash
docker container exec -it trustsandbox sh
```

Tu arrives dans un shell :

```bash
/ #
```

***

### Étape 2 – Télécharger et tagger une image de test

```bash
/ # docker pull docker/trusttest
/ # docker tag docker/trusttest sandboxregistry:5000/test/trusttest:latest
```

***

### Étape 3 – Activer la Content Trust

```bash
/ # export DOCKER_CONTENT_TRUST=1
/ # export DOCKER_CONTENT_TRUST_SERVER=https://notaryserver:4443
```

> ⚠️ Cette deuxième variable est nécessaire uniquement dans le sandbox (car tu utilises ton propre Notary Server). Avec Docker Hub, ce n’est pas requis.

***

### Étape 4 – Pousser et signer l’image

```bash
/ # docker push sandboxregistry:5000/test/trusttest:latest
```

👉 Lors du premier push, Docker va :

* Générer une **clé root (racine)** → tu définis un passphrase.
* Générer une **clé de repository** → tu définis un autre passphrase.
* Signer et pousser l’image dans le registre local **avec métadonnées de confiance**.

***

### Étape 5 – Vérifier le pull de l’image signée

```bash
/ # docker pull sandboxregistry:5000/test/trusttest
```

➡️ Tu verras que Docker télécharge l’image **en vérifiant la signature** (digest SHA256 inclus).

***

### Étape 6 – Simuler une attaque (image corrompue)

Depuis ton hôte, ouvre un autre terminal et entre dans le conteneur **registry** :

```bash
docker container exec -it sandboxregistry bash
```

Modifie une couche de l’image :

```bash
cd /var/lib/registry/docker/registry/v2/blobs/sha256/aa/aac0c133338db2b18ff054943cee3267fe50c75cdee969aed88b1992539ed042
echo "Malicious data" > data
```

***

### Étape 7 – Forcer un nouveau téléchargement de l’image

Retourne dans le conteneur `trustsandbox`, supprime l’image locale :

```bash
/ # docker image rm -f cc7629d1331a
```

Puis essaie de re-télécharger :

```bash
/ # docker pull sandboxregistry:5000/test/trusttest
```

👉 Résultat attendu : **Docker rejette l’image** car la signature ne correspond plus (erreur `unexpected EOF`).

***

### Étape 8 – Nettoyer le sandbox

Quand tu as terminé tes tests, supprime tout avec :

```bash
docker compose down -v
```

***

⚡ Tu viens de tester un scénario complet :

* **Pull → Push → Signature → Vérification → Attaque → Détection**.

