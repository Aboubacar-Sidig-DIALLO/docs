# 📦 Exportateurs d’images et de registres (Buildx)

👉 Les **exportateurs** déterminent **où et comment le résultat d’un build est enregistré ou envoyé**.\
Ils produisent une **image de conteneur** au format **OCI** ou **Docker**.

Il existe deux principaux exportateurs :

* **image exporter** → crée une image localement (dans ton environnement ou ton moteur Docker).
* **registry exporter** → identique, mais envoie directement l’image vers un **registre** (comme Docker Hub, GHCR, etc.) grâce à `push=true`.

***

### 🔧 Syntaxe de base

```bash
# Exporter vers une image locale
docker buildx build --output type=image[,paramètres] .

# Exporter vers un registre
docker buildx build --output type=registry[,paramètres] .
```

***

### 📑 Paramètres disponibles (type=image)

| Paramètre                | Type                              | Défaut                | Description                                                                                                |
| ------------------------ | --------------------------------- | --------------------- | ---------------------------------------------------------------------------------------------------------- |
| **name**                 | Chaîne                            | _(vide)_              | Nom(s) de l’image à créer (ex: `user/app:latest`).                                                         |
| **push**                 | vrai,faux                         | faux                  | Pousser l’image vers un registre après sa création.                                                        |
| **push-by-digest**       | vrai,faux                         | faux                  | Pousser uniquement l’image par digest (`sha256`), sans nom.                                                |
| **registry.insecure**    | vrai,faux                         | faux                  | Autoriser les registres **insecure** (HTTP sans TLS).                                                      |
| **dangling-name-prefix** | texte                             | _(vide)_              | Ajoute un préfixe au nom anonyme `prefix@<digest>`.                                                        |
| **name-canonical**       | vrai,faux                         | _(vide)_              | Ajoute un nom canonique `nom@<digest>` en plus des tags.                                                   |
| **compression**          | uncompressed, gzip, estargz, zstd | gzip                  | Type de compression des couches de l’image.                                                                |
| **compression-level**    | 0..22                             | _(selon compression)_ | Niveau de compression appliqué.                                                                            |
| **force-compression**    | vrai,faux                         | faux                  | Forcer la compression même si inutile.                                                                     |
| **rewrite-timestamp**    | vrai,faux                         | faux                  | Réécrit les horodatages des fichiers selon `SOURCE_DATE_EPOCH` (utile pour les **builds reproductibles**). |
| **oci-mediatypes**       | vrai,faux                         | faux                  | Utilise les formats **OCI** dans le manifeste.                                                             |
| **oci-artifact**         | vrai,faux                         | faux                  | Formate les attestations comme **artefacts OCI**.                                                          |
| **unpack**               | vrai,faux                         | faux                  | Décompresse l’image après sa création (utile avec containerd).                                             |
| **store**                | vrai,faux                         | vrai                  | Stocke l’image dans le moteur de l’environnement (ex: containerd).                                         |
| **annotation.\<clé>**    | Chaîne                            | _(vide)_              | Ajoute une **annotation personnalisée** OCI.                                                               |

***

### 🏷️ Ajouter des annotations (métadonnées OCI)

Les **annotations** permettent d’ajouter des métadonnées personnalisées à ton image.

#### Exemple :

```bash
docker buildx build \
  --output "type=registry,name=docker.io/utilisateur/app:1.0.0,annotation.org.opencontainers.image.title=MonApplication" .
```

➡️ L’image sera poussée avec une annotation `org.opencontainers.image.title=MonApplication`.

***

### 🚀 Exemples pratiques

1. **Construire une image locale compressée en Zstandard**

```bash
docker buildx build --output "type=image,name=monimage:latest,compression=zstd,compression-level=10" .
```

2. **Construire et pousser vers un registre sécurisé**

```bash
docker buildx build --output "type=registry,name=ghcr.io/utilisateur/app:latest,push=true" .
```

3. **Créer une image reproductible avec horodatage fixe**

```bash
docker buildx build --output "type=image,name=monimage:stable,rewrite-timestamp=true" .
```

***

⚡ **En résumé** :

* `type=image` = image locale.
* `type=registry` = image envoyée vers un registre.
* Tu peux gérer **compression**, **sécurité**, **noms automatiques**, **reproductibilité** et **annotations**.
