# 🔒 Content Trust dans Docker

### 🌍 Problème

Quand on télécharge (pull) ou publie (push) une image Docker via Internet, on doit se poser deux questions :

1. **Intégrité** → l’image a-t-elle été modifiée (corrompue ou compromise) en chemin ?
2. **Authenticité** → est-ce que l’image vient vraiment du **bon éditeur** (et pas d’un attaquant qui l’a remplacée) ?

👉 Sans garanties, une image pourrait contenir du code malveillant ou avoir été manipulée.

***

### 🛡️ Solution : Docker Content Trust (DCT)

Docker **Content Trust (DCT)** permet de :

* Vérifier **l’éditeur** d’une image grâce à des signatures cryptographiques.
* Vérifier **l’intégrité** (aucune modification non autorisée).
* Refuser l’utilisation d’images non signées si on active le mode strict.

#### ⚙️ Comment ça marche ?

* Chaque image peut être **signée** par son créateur avec une **clé privée**.
* La signature est stockée dans le registre via **Notary (TUF – The Update Framework)**.
* Quand un utilisateur fait un `docker pull`, Docker récupère et vérifie la signature avec la **clé publique** associée.

***

### 🚀 Activer Content Trust

Il suffit de définir la variable d’environnement :

```bash
export DOCKER_CONTENT_TRUST=1
```

Ensuite :

* `docker pull <image>` → échoue si l’image n’a **pas de signature valide**.
* `docker push <image>` → pousse aussi la **signature** de l’image.

Désactiver (à éviter en prod) :

```bash
export DOCKER_CONTENT_TRUST=0
```

***

### 📌 Exemple

```bash
# Activation du content trust
export DOCKER_CONTENT_TRUST=1

# Pull d’une image officielle (signée)
docker pull alpine:latest
# ✅ fonctionne, car Alpine est signé

# Pull d’une image sans signature
docker pull monrepo/test:unsigned
# ❌ échoue car aucune signature trouvée
```

***

### ✅ Avantages

* Protège contre l’usage d’images falsifiées.
* Force la vérification en CI/CD et production.
* Permet de contrôler **qui publie** les images (chaîne de confiance).

## 🔒 Docker Content Trust (DCT)

### 📌 Qu’est-ce que c’est ?

Docker Content Trust (DCT) permet d’utiliser des **signatures numériques** pour les données envoyées ou reçues depuis un registre Docker distant.\
Ces signatures permettent une **vérification côté client ou à l’exécution** de :

* l’**intégrité** d’une image (non modifiée),
* l’**éditeur** (publisher) de l’image.

👉 Avec DCT :

* Les **éditeurs** (individus, organisations, pipelines CI/CD) peuvent **signer leurs images**.
* Les **consommateurs** peuvent **s’assurer que les images pullées sont signées**.

***

### ⚠️ Dépréciation de DCT

Docker a annoncé le **retrait progressif de DCT** pour les **Docker Official Images (DOI)**.\
➡️ Il faut **préparer une migration** vers des solutions alternatives comme :

* **Sigstore**
* **Notation**

👉 Les dates précises de dépréciation complète seront publiées bientôt.

🔗 \[Voir : Retiring Docker Content Trust] (illustration / lien de référence ici)

***

### 🏷️ Images, tags et DCT

Un enregistrement d’image Docker est identifié par :

```
[REGISTRY_HOST[:REGISTRY_PORT]/]REPOSITORY[:TAG]
```

* Un **repository** peut avoir plusieurs **tags** (exemple : `mongo:latest` et `mongo:3.1.2`).
* DCT est **associé au TAG** → chaque tag peut être signé ou non.

#### Exemple

* `mongo:latest` → **non signé**
* `mongo:3.1.6` → **signé**

👉 C’est l’éditeur qui décide quels tags sont signés.\
👉 Il peut exister **deux versions différentes** d’un même tag (`latest`) : une signée et une non signée.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

***

### ✅ Tags signés

Un éditeur peut :

* publier une image `someimage:latest` **signée**
* puis republier une version non signée de `someimage:latest`

Résultat :

* le **tag signé `latest` reste valide** et immuable,
* mais le **tag non signé `latest` devient visible** pour ceux qui n’utilisent pas DCT.

***

### 👀 Vue côté consommateur

* **Sans DCT activé** :\
  → toutes les images (signées et non signées) sont visibles.
* **Avec DCT activé** :\
  → seules les images **signées** sont accessibles.\
  → les images non signées deviennent **invisibles**.

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

## 🔑 Docker Content Trust – Les clés

### 📌 Gestion de la confiance avec des clés

La **confiance pour un tag d’image** est gérée grâce à un **ensemble de clés de signature**.

👉 Lorsqu’une opération utilisant DCT est invoquée pour la première fois, un **jeu de clés** est créé.

Un jeu de clés contient trois types principaux :

1. **Clé hors ligne (Offline key)**
   * C’est la **racine de confiance (root key)** pour un tag d’image.
   * Elle sert de clé maîtresse et **doit rester ultra sécurisée**.
2. **Clés de dépôt ou de tag (Repository/Tagging keys)**
   * Elles sont utilisées pour **signer les tags d’images**.
   * Elles fonctionnent sous la racine de confiance.
3. **Clés gérées par le serveur (Server-managed keys)**
   * Exemple : la **clé de timestamp**.
   * Elle fournit des **garanties de fraîcheur** pour le repository (évite qu’un ancien tag signé soit réutilisé frauduleusement).

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

***

### ⚠️ Attention – perte de clés

* Si la **clé racine (root key)** est perdue → elle est **irréversible et non récupérable** ❌
* Si une **autre clé** est perdue → il faut contacter **le support Docker Hub**.\
  👉 Mais dans ce cas, chaque consommateur ayant déjà utilisé un tag signé de ce repo devra intervenir manuellement pour rétablir la confiance.

***

### 🛡️ Bonnes pratiques

* Sauvegarder la **clé racine** dans un endroit **sécurisé et hors ligne** (idéalement matériel : HSM, clé USB sécurisée, coffre-fort).
* Comme elle n’est requise **que pour créer de nouveaux repositories**, il est fortement recommandé de la **garder hors ligne**.
* Lire attentivement la documentation sur la **gestion et la sauvegarde des clés DCT** pour sécuriser correctement votre workflow.

## 📝 Signature d’images avec Docker Content Trust (DCT)

### 🔧 Pré-requis

* Un **registre Docker** avec un **serveur Notary** associé (par exemple Docker Hub).
* Une **paire de clés de délégation** pour signer les images.

👉 Tu peux générer ces clés **localement** avec :

```bash
docker trust key generate NOM
```

Ou bien les obtenir auprès d’une **autorité de certification**.

Les clés sont stockées par défaut dans `~/.docker/trust/`.

***

### 1. Générer ou importer une clé de délégation

#### Générer une nouvelle clé :

```bash
docker trust key generate jeff
```

Exemple de sortie :

```
Generating key for jeff...
Enter passphrase for new jeff key with ID 9deed25:
Successfully generated and loaded private key.
Corresponding public key available: /home/ubuntu/Documents/mytrustdir/jeff.pub
```

#### Importer une clé existante :

```bash
docker trust key load key.pem --name jeff
```

👉 Cela charge une clé privée déjà générée (`key.pem`).

***

### 2. Ajouter la clé publique au registre (Notary)

Chaque clé de délégation doit être associée à un **repository particulier**.\
La première fois, cela **initialise le repository** avec une **root key locale**.

```bash
docker trust signer add --key cert.pem jeff registry.example.com/admin/demo
```

***

### 3. Signer une image

Une fois les clés configurées, tu peux **signer et pousser** une image :

```bash
docker trust sign registry.example.com/admin/demo:1
```

Exemple :

```
Signing and pushing trust data for local image registry.example.com/admin/demo:1
Successfully signed registry.example.com/admin/demo:1
```

***

### 4. Pousser une image signée (avec DCT activé)

Active DCT avec la variable d’environnement :

```bash
export DOCKER_CONTENT_TRUST=1
docker push registry.example.com/admin/demo:1
```

👉 Avec `DOCKER_CONTENT_TRUST=1`, toutes les opérations (`push`, `pull`, `build`, `run`) nécessitent une **signature valide**.

***

### 5. Vérifier la signature d’une image

```bash
docker trust inspect --pretty registry.example.com/admin/demo:1
```

Exemple de sortie :

```
Signatures for registry.example.com/admin/demo:1

SIGNED TAG          DIGEST                                                             SIGNERS
1                   sha256:3d2e482b8260...                                             jeff

List of signers and their keys:
SIGNER              KEYS
jeff                8ae710e3ba82
```

***

### 6. Révoquer une signature

```bash
docker trust revoke registry.example.com/admin/demo:1
```

👉 Supprime la signature associée à un tag.

***

### 7. Forcer l’utilisation des images signées

Quand `DOCKER_CONTENT_TRUST=1` est activé :

* `docker pull image:latest` échoue si `latest` n’est pas signé.
* Par contre, `docker pull image@sha256:...` fonctionne toujours (car basé sur un hash explicite).

***

✅ **Résumé** :

1. Génère ou importe une clé (`docker trust key generate/load`).
2. Ajoute la clé publique au registre (`docker trust signer add`).
3. Signe et pousse ton image (`docker trust sign` ou `docker push` avec DCT activé).
4. Vérifie (`docker trust inspect`) ou révoque (`docker trust revoke`) les signatures.
