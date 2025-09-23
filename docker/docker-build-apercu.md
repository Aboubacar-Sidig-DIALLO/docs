# 🐳 Docker Build – Aperçu

**Docker Build** repose sur une **architecture client-serveur** :

* **Client → Buildx**
  * C’est l’interface que tu utilises pour lancer et gérer des builds.
  * C’est ce que tu invoques quand tu tapes `docker build` ou `docker buildx`.
  * Il transmet les instructions de build à l’arrière-plan.
* **Serveur → BuildKit**
  * C’est le moteur de build (le **builder**) qui exécute réellement les étapes de construction.
  * Il interprète les instructions du `Dockerfile`, gère le cache, les multi-stages, les secrets, etc.
  * C’est lui qui produit l’image finale (ou d’autres artefacts si tu utilises les exporters).

***

### 🔄 Fonctionnement

1.  Tu lances une commande comme :

    ```bash
    docker build -t monapp:1.0 .
    ```
2. **Buildx (client)** envoie une requête de build.
3. **BuildKit (serveur)** reçoit la requête, exécute les instructions du `Dockerfile` (copie, compilation, installation, etc.).
4. Le résultat est :
   * soit renvoyé au client (image locale),
   * soit **poussé vers un registre** (par ex. Docker Hub, GHCR, Harbor).

***

### 📦 Où trouve-t-on Buildx et BuildKit ?

* Sur **Docker Desktop** et **Docker Engine**, **Buildx** et **BuildKit** sont inclus par défaut.
* Quand tu fais `docker build`, tu utilises en fait **Buildx** qui lance un build via **BuildKit**.

***

### ✅ En résumé

* **Buildx = le client** (interface utilisateur pour gérer les builds).
* **BuildKit = le serveur** (moteur qui exécute réellement les builds).
* Ensemble, ils rendent les builds plus rapides, portables et puissants (cache avancé, multi-platformes, multi-stages, secrets, orchestration avec Bake, etc.).

## 🔧 Buildx

**Buildx** est l’outil **CLI** (en ligne de commande) utilisé pour exécuter et gérer des builds dans Docker.\
En réalité, la commande `docker build` n’est qu’un **raccourci (wrapper)** autour de **Buildx**.

Quand tu fais :

```bash
docker build .
```

➡️ Docker appelle **Buildx**, qui interprète les options et transmet une requête de build au moteur **BuildKit**.

***

### ✨ Ce que Buildx peut faire

Buildx ne se limite pas à lancer des builds, il fournit aussi des fonctionnalités avancées :

* **Créer et gérer des “builders” (backends BuildKit)**\
  → Tu peux choisir où et comment exécuter tes builds (local, remote, cluster, etc.).
* **Exécuter des builds concurrentiels**\
  → Plusieurs builds peuvent être lancés en parallèle.
* **Support des images multi-plateformes**\
  → Par exemple, construire une seule image qui fonctionne sur `amd64`, `arm64`, etc.
* **Gestion des images dans les registres**\
  → Buildx peut pousser des images, gérer les tags et orchestrer plusieurs builds vers différents registres.

***

### ⚙️ Installation

* Sur **Docker Desktop**, Buildx est installé **par défaut**.
* Sinon, tu peux :
  * Compiler le plugin CLI à partir des sources.
  * Télécharger un binaire depuis le [dépôt GitHub officiel de Buildx](https://github.com/docker/buildx).

***

### ⚠️ À noter

Même si `docker build` utilise Buildx en interne, il existe quelques **différences subtilement importantes** entre :

* `docker build` (plus limité, mode “legacy”)
* `docker buildx build` (la version “complète” et canonique)

👉 Exemple : certaines options avancées (multi-plateformes, cache, exporters) ne sont disponibles **qu’avec** `docker buildx build`.

## ⚙️ BuildKit

**BuildKit** est le **moteur (daemon)** qui exécute réellement les builds Docker.\
C’est lui qui fait le "travail lourd" quand tu lances un `docker build`.

***

### 🔄 Comment ça marche ?

1.  Tu lances une commande :

    ```bash
    docker build .
    ```
2. **Buildx** (le client CLI) interprète la commande et envoie une **requête de build** à **BuildKit**.\
   Cette requête contient :
   * Le **Dockerfile**
   * Les **arguments de build** (`--build-arg`)
   * Les **options d’export** (par ex. image vers un registre, tar, etc.)
   * Les **options de cache**
3. **BuildKit** :
   * Résout les instructions du Dockerfile.
   * Exécute les étapes du build.
   * Demande uniquement les ressources nécessaires **au moment où elles sont utiles**.
4. Pendant ce temps, **Buildx** surveille l’état du build et affiche la progression dans ton terminal.

***

### 🚀 Efficacité par rapport à l’ancien moteur

L’ancien moteur Docker (avant BuildKit) était **moins optimisé** :

* Il copiait toujours tout le **filesystem local** comme contexte de build, même si seule une partie était utilisée.

👉 BuildKit, lui, est **plus intelligent** :

* Il ne télécharge ou ne copie que ce qui est nécessaire, quand c’est nécessaire.

***

### 📦 Ressources que BuildKit peut demander via Buildx

* **Contextes locaux** du filesystem (les fichiers nécessaires pour le build).
* **Secrets de build** (ex: clés API, mots de passe).
* **Sockets SSH** (utile pour accéder à des dépôts privés via SSH).
* **Jetons d’authentification aux registres** (Docker Hub, GitHub Registry, etc.).

***

✅ En résumé :\
**BuildKit est le moteur moderne et optimisé de Docker Build**, qui améliore la rapidité, la sécurité et l’efficacité des builds grâce à une meilleure gestion des ressources.
