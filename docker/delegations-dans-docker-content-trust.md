# 🔑 Délégations dans Docker Content Trust

Les **délégations** permettent de définir **qui est autorisé ou non à signer un tag d’image**.\
Chaque délégation est associée à une **paire de clés privée/publique**.

👉 Une délégation peut contenir :

* plusieurs paires de clés (pour plusieurs contributeurs),
* une gestion de **rotation des clés** (remplacement en cas de compromission ou expiration).

***

### 🌟 La délégation principale : `targets/releases`

* C’est **la source canonique de confiance** pour un tag d’image.
* **Sans être ajouté à `targets/releases`, un utilisateur ne peut pas signer de tag.**
* Les signatures validées via cette délégation sont celles que les consommateurs de DCT reconnaissent comme **officielles**.

***

### 🔧 Gestion via `docker trust`

Heureusement, les commandes `docker trust` simplifient cette gestion.\
Exemple :

```bash
docker trust signer add --key jeff.pub jeff registry.example.com/admin/demo
```

Ce qui se passe en arrière-plan :

1. Le dépôt est initialisé si nécessaire.
2. Les **clés du dépôt** sont gérées automatiquement.
3. La clé publique de `jeff` est ajoutée à la délégation **`targets/releases`**.

👉 Résultat : **Jeff est autorisé à signer les tags de l’image `registry.example.com/admin/demo`**.

***

✅ **Résumé :**

* Une **délégation** = un groupe d’autorisations de signature.
* La plus importante = `targets/releases`.
* Seuls les contributeurs ajoutés à cette délégation peuvent signer officiellement des tags.
* `docker trust signer add` gère l’ajout d’un collaborateur.

## ⚙️ Configuration du client Docker avec DCT

### 🔹 1. Comportement par défaut

* Les commandes `docker trust` supposent que **l’URL du serveur Notary = l’URL du registre** (ex. Docker Hub ou Docker Trusted Registry).
* Donc si tu utilises Docker Hub → rien à configurer.

👉 Mais pour un **registre auto-hébergé** ou **tiers**, il faut indiquer une URL différente pour Notary.

***

### 🔹 2. Définir un Notary server personnalisé

Ajoute cette variable d’environnement :

```bash
export DOCKER_CONTENT_TRUST_SERVER=https://URL:PORT
```

Exemple :

```bash
export DOCKER_CONTENT_TRUST_SERVER=https://notary.mycompany.com:4443
```

Sans ça, tu peux rencontrer des erreurs comme :

```bash
Error: trust data missing for remote repository ... timestamp key trust data unavailable.
```

***

### 🔹 3. Authentification

Si ton serveur Notary (ou DTR) requiert une authentification → tu dois être connecté avant d’ajouter ou de signer :

```bash
docker login registry.example.com/user/repo
Username: admin
Password:

Login Succeeded
```

Sinon → erreur `401 unauthorized`.

***

### 🔹 4. Exemple complet

```bash
# connexion
docker login registry.example.com/user/repo

# ajout d’un signataire
docker trust signer add --key cert.pem jeff registry.example.com/user/repo
```

Résultat attendu :

```
Successfully initialized "registry.example.com/user/repo"
Successfully added signer: jeff to registry.example.com/user/repo
```

***

## ⚙️ Configuration du client Notary (avancé)

Pour des besoins plus avancés (délégations complexes, gestion fine des clés, automatisation), on peut utiliser le **client Notary CLI**.

#### 🔧 Étapes :

1. Télécharger le binaire `notary` et l’ajouter au **PATH**.
2. Créer un fichier de config `~/.notary/config.json` :

```json
{
  "trust_dir" : "~/.docker/trust",
  "remote_server": {
    "url": "https://registry.example.com",
    "root_ca": "../.docker/ca.pem"
  }
}
```

* `trust_dir` → où sont stockées localement les clés et métadonnées de confiance.
* `remote_server.url` → l’URL du serveur Notary.
* `root_ca` → optionnel, pour spécifier ton certificat racine si le serveur utilise une CA privée.

***

✅ **Résumé :**

* Docker Hub/DTR → rien à configurer.
* Registre privé/tiers → définir `DOCKER_CONTENT_TRUST_SERVER`.
* Toujours se connecter (`docker login`) avant d’ajouter/signature.
* Pour des cas avancés → configurer le client Notary avec `~/.notary/config.json`.

## 🔑 Création et gestion des clés de délégation dans DCT

### 1. Qu’est-ce qu’une clé de délégation ?

* Les **clés de délégation** permettent de donner à un ou plusieurs contributeurs l’autorisation de **signer des tags d’images**.
* Une délégation peut contenir plusieurs clés (pour plusieurs contributeurs ou pour gérer la rotation de clés).
* La délégation la plus importante est **`targets/releases`**, car c’est la source canonique d’une image signée de confiance.

***

### 2. Générer une paire de clés de délégation

#### 🔹 a) Génération via Docker Trust (simple)

Docker peut générer directement une paire clé privée/publique et l’ajouter dans le **trust store local** :

```bash
docker trust key generate jeff
```

👉 Cela va :

* Créer une clé privée (protégée par mot de passe).
* Placer la clé privée dans `~/.docker/trust/private`.
* Générer une clé publique (`jeff.pub`) à partager.

***

#### 🔹 b) Génération manuelle (OpenSSL)

Si tu veux gérer les clés toi-même :

1.  Générer une clé RSA 2048 bits :

    ```bash
    openssl genrsa -out delegation.key 2048
    ```
2.  Créer une CSR (Certificate Signing Request) :

    ```bash
    openssl req -new -sha256 -key delegation.key -out delegation.csr
    ```
3.  Soit tu la fais signer par une **CA** (interne/externe), soit tu auto-signes :

    ```bash
    openssl x509 -req -sha256 -days 365 -in delegation.csr \
    -signkey delegation.key -out delegation.crt
    ```
4.  Importer la clé privée dans Docker Trust :

    ```bash
    docker trust key load delegation.key --name jeff
    ```

***

### 3. Ajouter une délégation dans un dépôt

Avec la clé publique (par ex. `cert.pem`) :

```bash
docker trust signer add --key cert.pem jeff registry.example.com/admin/demo
```

👉 Cela :

* **Initialise le dépôt** (si c’est la 1ère fois).
* Crée les clés administratives (root, repository).
* Ajoute `jeff` comme signataire.

Tu auras besoin du **passphrase de la root key** et du **repository key**.

***

### 4. Vérifier les délégations

Tu peux inspecter :

```bash
docker trust inspect --pretty registry.example.com/admin/demo
```

Exemple de sortie :

```
SIGNER   KEYS
jeff     1091060d7bfd
```

Ou plus en détail avec Notary CLI :

```bash
notary delegation list registry.example.com/admin/demo
```

***

### 5. Ajouter plusieurs signataires

Tu peux ajouter d’autres contributeurs (par ex. `ben`) :

```bash
docker trust signer add --key ben.pub ben registry.example.com/admin/demo
```

***

### 6. Rotation ou ajout de clés pour un signataire

Si tu veux ajouter une **nouvelle clé pour un signataire existant** (ex. `jeff`) :

```bash
docker trust signer add --key cert2.pem jeff registry.example.com/admin/demo
```

👉 Résultat : `jeff` aura plusieurs clés valides pour signer.

***

## ✅ Bonnes pratiques

* **Sauvegarde de la root key** (c’est la plus critique, si elle est perdue → tout est bloqué).
* **Utiliser des passphrases fortes** pour toutes les clés privées.
* **Rotation régulière** des clés de délégation en cas de compromission ou changement d’équipe.

## 🗑️ Suppression d’une délégation ou d’une clé dans Docker Content Trust

### 1. Supprimer une délégation entière (signataire complet)

Si tu veux **retirer un signataire** (et toutes ses clés associées dans `targets/releases`) :

```bash
docker trust signer remove ben registry.example.com/admin/demo
```

👉 Ce qui se passe :

* Le signataire `ben` est supprimé de la délégation.
* Les **tags signés par ben** devront être **re-signés** par une délégation encore active.

⚠️ Si tu vois une erreur du type :

```
WARN[0000] role targets/releases has fewer keys than its threshold of 1...
```

→ Cela signifie qu’il n’y a plus assez de clés actives. Il faut **ajouter une délégation avec `docker trust signer add`** puis **resigner les images**.

***

### 2. Supprimer une clé précise d’un contributeur

Cas utile pour **rotation de clés** : garder la délégation (ex. `jeff`) mais supprimer une clé en particulier.

1️⃣ Lister les clés sur le serveur Notary :

```bash
notary delegation list registry.example.com/admin/demo
```

Exemple :

```
ROLE                PATHS             KEY IDS
targets/jeff        "" <all paths>    8fb597c...
                                      1091060d...
targets/releases    "" <all paths>    8fb597c...
                                      1091060d...
```

2️⃣ Supprimer la clé de `targets/releases` :

```bash
notary delegation remove registry.example.com/admin/demo targets/releases 1091060d7bfd... --publish
```

3️⃣ Supprimer aussi la clé de `targets/jeff` :

```bash
notary delegation remove registry.example.com/admin/demo targets/jeff 1091060d7bfd... --publish
```

4️⃣ Vérifier la liste mise à jour :

```bash
notary delegation list registry.example.com/admin/demo
```

***

### 3. Supprimer une clé privée locale

Si tu veux retirer une clé de ton **trust store local** (`~/.docker/trust/private`) :

1️⃣ Lister les clés locales :

```bash
notary key list
```

2️⃣ Supprimer une clé par son ID :

```bash
notary key remove 1091060d7bfd938dfa5be703fa057974f9322a4faef6f580334f3d6df44c02d1
```

👉 Tu devras confirmer avec `yes`.

***

### 4. Supprimer toutes les métadonnées de confiance d’un dépôt

Si tu veux **réinitialiser complètement un repo signé** :

```bash
notary delete registry.example.com/admin/demo --remote
```

👉 Cela supprime **repository key, snapshot, delegations, trust data**.\
⚠️ C’est souvent requis avant de supprimer le dépôt du registre.

Vérification :

```bash
docker trust inspect --pretty registry.example.com/admin/demo
```

→ devrait renvoyer :

```
No signatures or cannot access registry.example.com/admin/demo
```

***

✅ En résumé :

* `docker trust signer remove` → supprime un signataire complet.
* `notary delegation remove` → supprime une clé spécifique d’une délégation.
* `notary key remove` → supprime une clé locale.
* `notary delete` → supprime toutes les données de confiance d’un repo.
