# 🔑 Gestion des clés pour Docker Content Trust (DCT)

La **confiance** dans une image Docker est assurée par l’utilisation de **clés cryptographiques**. Chaque type de clé a un rôle précis dans la chaîne de confiance.

***

### 1. Types de clés

| Clé                                 | Description                                                                                                                                                                                                              |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Root key** (clé racine)           | C’est la racine de confiance. Générée une seule fois quand le **Content Trust** est activé pour la première fois. 🔒 Elle doit être conservée **hors ligne** (clé la plus sensible). On l’appelle aussi **offline key**. |
| **Targets key** (ou Repository key) | Sert à **signer les tags des images** et gérer les **délégations** (ajout/suppression de signataires). C’est elle qui détermine **quels tags** peuvent être signés dans un dépôt.                                        |
| **Snapshot key**                    | Signe la **collection actuelle des tags d’images** → protège contre les attaques de type _mix and match_ (où un attaquant pourrait combiner un ancien tag avec un nouveau).                                              |
| **Timestamp key**                   | Fournit une **garantie de fraîcheur** (anti-rejeu). Permet aux dépôts d’indiquer qu’une signature est récente, sans que les clients aient à re-télécharger tout le contenu régulièrement.                                |
| **Delegation keys**                 | Clés optionnelles pour déléguer la signature de tags à d’autres personnes/équipes **sans partager la clé `targets` principale**. Exemple : une équipe CI/CD peut signer automatiquement des builds.                      |

***

### 2. Génération et stockage des clés

Lors du premier `docker push` avec DCT activé (`DOCKER_CONTENT_TRUST=1`) :

* ✅ **Root key** → générée **localement** et stockée sur ton poste.
* ✅ **Targets key** → générée **localement** et stockée sur ton poste.
* ✅ **Snapshot key** → générée et stockée côté serveur (Notary), **chiffrée au repos**.
* ✅ **Timestamp key** → générée et stockée côté serveur (Notary).
* ❌ **Delegation keys** → **non générées automatiquement**, elles doivent être créées et ajoutées manuellement.

👉 Par défaut :

* Les clés locales sont dans `~/.docker/trust/private/`.
* Les clés gérées côté serveur sont protégées et ne sont pas directement accessibles.

***

### 3. Gestion pratique des clés

#### 📍 Générer une clé de délégation

```bash
docker trust key generate devteam
```

👉 Génère une clé privée et publique pour la délégation **devteam**.\
La clé privée est stockée localement, la clé publique peut être partagée.

***

#### 📍 Importer une clé existante

```bash
docker trust key load delegation.key --name devteam
```

***

#### 📍 Ajouter un signataire (délégation)

```bash
docker trust signer add --key devteam.crt devteam registry.example.com/project/repo
```

***

#### 📍 Lister les clés locales

```bash
notary key list
```

***

#### 📍 Lister les délégations d’un repo

```bash
notary delegation list registry.example.com/project/repo
```

***

### 4. 🔒 Bonnes pratiques de sécurité

1. **Conserver la Root Key hors ligne**
   * Stocker dans un **HSM**, une clé USB chiffrée, ou un coffre-fort de secrets.
   * Elle n’est nécessaire que lors de la création d’un nouveau repository.
2. **Sauvegarder les clés locales (`targets`)**
   * Copie sécurisée.
   * Ne pas les laisser sur une machine CI/CD non sécurisée.
3. **Utiliser les clés de délégation pour les builds CI/CD**
   * La clé principale `targets` reste protégée.
   * Les délégations peuvent être renouvelées facilement.
4. **Rotation des clés**
   * Prévoir un mécanisme pour remplacer une clé compromise.
   * Utiliser `docker trust signer add` pour ajouter une nouvelle clé, puis retirer l’ancienne.

***

✅ En résumé :

* **Root Key** = identité maîtresse (offline).
* **Targets Key** = signature des tags et gestion des délégations.
* **Snapshot + Timestamp Keys** = gérées côté serveur pour intégrité et fraîcheur.
* **Delegation Keys** = pour déléguer la signature de manière granulaire et sécurisée.

## 🔐 Passphrases et gestion des clés dans Docker Content Trust

### 1. 🔑 Passphrases

* Chaque **clé privée** (root key, repository key, délégations) est **chiffrée** avec une **passphrase**.
* **But** : si ton PC est volé ou si une sauvegarde fuite, les clés ne sont pas exploitables sans le mot de passe.
* **Bonnes pratiques :**
  * Génère des passphrases **fortes et aléatoires** (via un gestionnaire de mots de passe).
  * Ne réutilise jamais la même passphrase.
  * Stocke-les dans un **password manager** (Bitwarden, 1Password, KeePass, etc.).

⚠️ **Important** :

* La passphrase **ne protège que le stockage local**. Si tu oublies la passphrase de la clé **root**, tu ne peux plus l’utiliser → perte définitive.

***

### 2. 💾 Sauvegarde des clés

*   Les clés de confiance Docker sont stockées dans :

    ```
    ~/.docker/trust/private
    ```
* Elles sont déjà chiffrées par passphrase, mais il faut les sauvegarder correctement.
* Bonnes pratiques :
  * Crée **au moins deux sauvegardes chiffrées** sur **clés USB sécurisées**.
  *   Utilise des commandes avec permissions strictes pour éviter les fuites :

      ```bash
      umask 077; tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private; umask 022
      ```

      🔒 `umask 077` garantit que seul ton utilisateur peut lire le fichier.

***

### 3. 📦 Stockage matériel (Yubikey)

* Docker Content Trust peut stocker et signer les **root keys** directement dans un **Yubikey 4**.
* Priorité : si un Yubikey est détecté, Docker l’utilise au lieu du système de fichiers.
* Avantages :
  * La clé privée **ne quitte jamais le hardware**.
  * Protection contre l’exfiltration (même si ton PC est compromis).
* Cas pratique : quand tu initialises un nouveau repo avec DCT, Docker vérifie d’abord si un root key existe en local.
  * Sinon, et si un Yubikey est présent → il génère la root key dans le Yubikey.

***

### 4. ⚠️ Perte de clés

* **Perte de la Repository Key (`targets`)** :
  * Récupérable ✅.
  * Tu peux contacter **Docker Hub Support** pour réattribuer une nouvelle clé.
  * Mais : les **images déjà signées** devront être re-signées avec la nouvelle clé.
  * Les utilisateurs devront tirer les nouvelles images.
* **Perte de la Root Key** :
  * 🚨 **Non récupérable !**
  * Tu perds **définitivement** la capacité de signer ce repo.
  * Tous les utilisateurs devront basculer sur un **nouveau repo signé avec une nouvelle root key**.
* Conséquences pour les consommateurs d’images :
  *   Ils verront une erreur comme :

      ```
      Warning: potential malicious behavior - trust data has insufficient signatures
      ```
  * Solution : tirer une nouvelle image signée avec la clé valide.

***

### 5. ✅ Résumé des bonnes pratiques

1. **Choisir des passphrases fortes** et les stocker dans un password manager.
2. **Sauvegarder les clés privées chiffrées** (archive `tar.gz` protégée, USB sécurisée).
3. **Utiliser un Yubikey** pour la root key si possible.
4. **Ne jamais perdre la root key** → sinon ton repo devient inutilisable.
5. Prévoir un **plan de rotation** des repository keys et delegation keys.
