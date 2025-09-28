# 🛠️ Building best practices – Multi-stage builds

### 🎯 Pourquoi utiliser les multi-stage builds ?

* **Réduction de taille** :\
  Les dépendances, outils de compilation ou fichiers temporaires ne polluent pas l’image finale.
* **Séparation claire** :\
  On distingue la phase de **construction** (build, compilation, tests) de la phase **exécution** (runtime minimal).
* **Performances** :\
  Certaines étapes peuvent être parallélisées et réutilisées grâce au cache Docker.
* **Sécurité** :\
  On ne garde que le strict nécessaire → moins de surface d’attaque.

***

### 🔎 Exemple sans multi-stage (classique mais lourd)

```dockerfile
FROM golang:1.22

WORKDIR /app
COPY . .
RUN go build -o myapp .

CMD ["./myapp"]
```

👉 Problème :

* L’image finale contient **Go + outils de build + code source complet**, inutile en production.

***

### ✅ Exemple avec multi-stage build

```dockerfile
# Étape 1 : Build
FROM golang:1.22 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp .

# Étape 2 : Runtime minimal
FROM debian:bookworm-slim
WORKDIR /app
COPY --from=builder /app/myapp .
CMD ["./myapp"]
```

👉 Avantages :

* L’image finale ne contient que :
  * le binaire `myapp`
  * le runtime Debian minimal
* La taille passe de **plusieurs centaines de Mo** → **quelques Mo seulement**.

***

### ⚡ Exemple avancé avec plusieurs étapes

On peut chaîner plusieurs étapes :

1. Compilation →
2. Tests →
3. Image finale allégée.

```dockerfile
# Étape 1 : Build
FROM node:20 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Étape 2 : Tests
FROM build AS test
RUN npm run test

# Étape 3 : Production
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
```

👉 Ici :

* Étape `build` → compile l’application.
* Étape `test` → vérifie la qualité (mais n’est pas incluse dans l’image finale).
* Étape `production` → ne garde que le contenu `dist/` servi par **Nginx**.

***

## 🎉 Conclusion

Les **multi-stage builds** sont une **meilleure pratique incontournable** pour :

* optimiser la taille des images,
* réduire les dépendances inutiles,
* sécuriser vos déploiements.

## 🛠️ Building best practices – **Créer des stages réutilisables**

### 🎯 Pourquoi ?

* **Optimisation** :\
  Les parties communes (dépendances, outils, configurations) sont construites **une seule fois** et mises en cache.
* **DRY (Don’t Repeat Yourself)** :\
  Au lieu de recopier les mêmes instructions dans plusieurs Dockerfiles/stages, tu centralises.
* **Efficacité mémoire** :\
  Les images dérivées partagent les couches déjà construites.
* **Maintenance** :\
  Une modification du stage commun se propage automatiquement à toutes les variantes.

***

### 🔎 Exemple simple : applications Node.js

Supposons que tu construises plusieurs variantes (API, worker, frontend) qui partagent :

* la même base Node.js,
* les mêmes dépendances `package.json`.

👉 **Stage réutilisable :**

```dockerfile
# syntax=docker/dockerfile:1

# Étape commune = dépendances Node.js
FROM node:20 AS base
WORKDIR /app
COPY package*.json ./
RUN npm install
```

👉 **API image :**

```dockerfile
FROM base AS api
COPY api/ .
CMD ["node", "server.js"]
```

👉 **Worker image :**

```dockerfile
FROM base AS worker
COPY worker/ .
CMD ["node", "worker.js"]
```

👉 **Frontend image :**

```dockerfile
FROM base AS frontend
COPY frontend/ .
RUN npm run build
CMD ["npm", "start"]
```

***

### ✅ Avantages

* `npm install` n’est exécuté qu’une seule fois (caché dans `base`).
* Les 3 images (`api`, `worker`, `frontend`) partagent la même base → **gains de temps + mémoire**.
* Si tu mets à jour une dépendance dans `package.json`, toutes les images héritent du changement.

***

### ⚡ Exemple avancé : pipeline CI/CD

Tu peux même chaîner les stages pour centraliser **build + tests + prod** :

```dockerfile
FROM node:20 AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM deps AS build
COPY . .
RUN npm run build

FROM deps AS test
COPY . .
RUN npm test

FROM nginx:alpine AS prod
COPY --from=build /app/dist /usr/share/nginx/html
```

👉 Ici, `deps` sert de **stage commun réutilisable** pour `build`, `test` et potentiellement d’autres.

***

## 🎉 Conclusion

* Les **stages réutilisables** permettent de **factoriser**, **gagner en vitesse** et **éviter la duplication**.
* C’est particulièrement puissant si tu as **plusieurs images similaires** dans un même projet (monorepos, microservices).

## 🛡️ Bonnes pratiques de construction – **Choisir la bonne image de base**

***

### 🎯 Pourquoi c’est important ?

La **première étape** pour obtenir une image **sécurisée et fiable** est de **choisir correctement l’image de base** (`FROM …` dans ton Dockerfile).\
👉 Le choix du point de départ impacte :

* la **sécurité** (moins de failles connues),
* la **taille de l’image** (rapidité de téléchargement et déploiement),
* la **portabilité** (exécution fluide sur différentes plateformes).

***

### ✅ Sources fiables pour les images de base

Docker met en avant plusieurs types d’images **validées et vérifiées** 🔒 :

#### 1. **Docker Official Images**

📦 Collection **officielle et certifiée** par Docker.

* Documentation claire,
* Suivi des bonnes pratiques,
* Mises à jour régulières.\
  👉 Un excellent **point de départ** pour la majorité des projets.

#### 2. **Verified Publisher Images**

🏢 Images publiées par des **éditeurs partenaires** de Docker (Microsoft, Red Hat, etc.).

* Docker vérifie l’authenticité du contenu,
* Gage de **qualité et fiabilité**.

#### 3. **Docker-Sponsored Open Source**

🌍 Images publiées par des projets **open source sponsorisés par Docker**.

* Maintenus par les communautés,
* Bénéficient du programme officiel.

👉 Lorsque tu explores Docker Hub, repère les **badges** 🏅 (Official / Verified Publisher) pour savoir si l’image est **sécurisée et reconnue**.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

***

### ⚡ Bonnes pratiques de choix d’image de base

* **Minimalisme = sécurité + performance** 🚀\
  Plus l’image est **petite**, plus elle est rapide à télécharger, portable, et moins elle embarque de dépendances vulnérables.
* **Séparer build et production** 🏗️➡️🏭\
  Utilise **deux types d’images** :
  * Une **image riche** (avec compilateurs, outils de debug, etc.) pour le **build et les tests unitaires**.
  * Une **image slim** (très légère, sans outils inutiles) pour la **production**.\
    👉 Cela **réduit la surface d’attaque** et accélère le déploiement.

Exemple :

```dockerfile
# Étape de build (image riche)
FROM golang:1.22 AS build
WORKDIR /app
COPY . .
RUN go build -o myapp

# Étape de production (image minimaliste)
FROM alpine:3.19
COPY --from=build /app/myapp /usr/local/bin/myapp
ENTRYPOINT ["myapp"]
```

***

### 🎉 Résumé

* Toujours choisir une **image officielle, vérifiée ou sponsorisée** 📦.
* **Moins c’est gros, mieux c’est** → opte pour des images minimalistes 🪶.
* Sépare **environnement de build** et **environnement de prod** pour sécurité et performance.

## 🔄 Bonnes pratiques de construction – **Reconstruisez vos images régulièrement**

***

### 📦 Pourquoi reconstruire ses images souvent ?

Les **images Docker sont immuables** : une fois construites, elles figent l’état exact de :

* ton **image de base** (ex. `ubuntu:24.04`),
* tes **librairies installées**,
* tes **dépendances**.

👉 Cela veut dire que si tu ne reconstruis pas ton image régulièrement, tu risques de :

* rester avec des **versions obsolètes** de bibliothèques,
* manquer des **mises à jour de sécurité critiques**,
* embarquer des **failles connues** dans tes déploiements.

***

### 🛠️ Comment garder ses images à jour ?

#### 1. **Reconstruire régulièrement**

Planifie des reconstructions fréquentes (CI/CD, jobs planifiés) pour t’assurer que :

* tes images incluent les **derniers correctifs**,
* les **dépendances critiques** sont à jour.

***

#### 2. **Forcer un build sans cache**

Par défaut, Docker optimise en réutilisant le cache des couches précédentes.\
Mais pour s’assurer d’un **rafraîchissement complet** des dépendances, utilise `--no-cache` :

```bash
docker build --no-cache -t my-image:my-tag .
```

👉 Cela évite d’utiliser des versions en cache et garantit un **téléchargement frais** des images de base et dépendances.

***

#### 3. **Exemple avec Ubuntu**

Voici un Dockerfile qui utilise l’image Ubuntu `24.04`.

⚠️ Attention : ce tag évolue avec le temps. L’éditeur (Canonical) peut republier la même balise avec un **nouveau snapshot corrigé**.

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:24.04
RUN apt-get -y update && apt-get install -y --no-install-recommends python3
```

En reconstruisant ton image **avec `--no-cache`**, tu forces :

* le téléchargement de la **dernière version d’Ubuntu 24.04**,
* l’installation des **derniers paquets apt sécurisés**.

***

#### 4. **Pense au “pinning” (fixer les versions)**

⚓ Pour éviter les surprises (changements majeurs involontaires), tu peux :

* **épingle une version précise** (`ubuntu:24.04.1` plutôt que `ubuntu:24.04`).
* garantir que ton build sera **reproductible** et **stable**, tout en prévoyant des rebuilds planifiés pour intégrer les patchs.

***

### 📸 Illustration mentale

Imagine ton image comme une **photo instantanée** 📷 :

* reconstruire sans cache = reprendre une **nouvelle photo à la lumière du jour** 🌞,
* laisser traîner ton image = garder une **vieille photo jaunie** 📜 qui ne reflète plus la réalité.

***

✅ **Résumé :**

* Reconstruis régulièrement pour **sécurité + fraîcheur**.
* Utilise `--no-cache` pour forcer un build propre.
* Épingle tes versions pour garder des builds **prévisibles et stables**.

## 🚫 Bonnes pratiques de construction – **Exclure avec `.dockerignore`**

***

### 📦 Pourquoi utiliser `.dockerignore` ?

Lorsque tu construis une image avec `docker build`, **tout le contexte de build** (les fichiers de ton projet) est envoyé au démon Docker.

👉 Problème : si tu n’exclus pas les fichiers inutiles, tu risques :

* d’augmenter la **taille du contexte de build**,
* de **ralentir les builds** (transfert inutile de fichiers),
* d’embarquer accidentellement des **fichiers sensibles** (mots de passe, clés SSH, etc.),
* de compliquer la maintenance et la sécurité.

La solution ✅ → **`.dockerignore`**, qui fonctionne comme un `.gitignore` : tu indiques quels fichiers/dossiers **ignorer** lors du build.

***

### 🛠️ Exemple simple

Tu veux ignorer tous les fichiers Markdown (`.md`) non pertinents pour ton build :

`.dockerignore`

```dockerignore
*.md
```

👉 Résultat : tous les fichiers `README.md`, `docs/*.md`, etc. **ne seront pas envoyés dans le contexte de build**.

***

### 📖 Cas d’usage courant

Voici quelques exemples pratiques à mettre dans `.dockerignore` :

```dockerignore
# Ignorer fichiers temporaires
*.log
*.tmp

# Ignorer répertoires de dépendances
node_modules
target
dist

# Ignorer fichiers de contrôle de version
.git
.gitignore

# Ignorer fichiers de configuration locaux
.env
docker-compose.override.yml
```

***

### 📸 Illustration (vue conceptuelle)

* Sans `.dockerignore` : 📦 ton **contexte de build gonfle** avec des fichiers inutiles.
* Avec `.dockerignore` : ✂️ tu ne transfères que **ce qui est nécessaire** → build plus rapide, plus propre, plus sécurisé.

***

### ✅ Résumé

* `.dockerignore` = ton **filet de sécurité** contre les fichiers inutiles et sensibles 🚫.
* Réduit la **taille**, accélère les **builds**, améliore la **sécurité**.
* Fonctionne **exactement comme un `.gitignore`** → exclusion par motifs (patterns glob).

## ⚡ Bonnes pratiques de construction – **Créer des conteneurs éphémères**

***

### 🧩 Qu’est-ce qu’un conteneur éphémère ?

Un conteneur **éphémère** est un conteneur qui peut être :

* **arrêté** 🛑,
* **détruit** 🗑️,
* **reconstruit** 🏗️,
* **remplacé** 🔄

👉 Le tout avec **un minimum absolu de configuration manuelle**.

En d’autres termes : **rien dans ton conteneur ne doit être unique ou irremplaçable**.

***

### 🎯 Pourquoi viser l’éphémère ?

Cela rejoint les principes de **The Twelve-Factor App** (méthodologie de conception d’applications modernes et cloud-native).

#### Les avantages :

* 🔄 **Scalabilité** : tu peux créer/détruire des conteneurs à la volée (auto-scaling, load balancing).
* 🛠️ **Maintenance simplifiée** : si un conteneur est corrompu, tu le remplaces, pas besoin de le réparer.
* 🚀 **Déploiements rapides** : un conteneur stateless démarre sans dépendre de configurations locales compliquées.
* 🔒 **Sécurité accrue** : pas de données sensibles stockées dans le conteneur → moins de risques de fuite.

***

### 📦 Comment créer des conteneurs éphémères ?

#### 1. **Aucun état persistant à l’intérieur du conteneur**

* Ne stocke pas de fichiers de logs, uploads ou bases de données dans le conteneur.
* Utilise des volumes ou des services externes pour la persistance (base de données, stockage objet type S3, etc.).

***

#### 2. **Configuration via variables d’environnement**

Au lieu de mettre ta config en dur dans le conteneur :\
👉 Passe-la via des **variables d’environnement** (`-e` ou `.env`).

Exemple :

```bash
docker run -e DATABASE_URL=mysql://user:pass@db/app my-app
```

***

#### 3. **Facile à remplacer**

Ton conteneur doit être **stateless** :

* Tu arrêtes → 🚫 pas de perte critique.
* Tu relances → ✅ il reprend son rôle automatiquement.

***

### 📸 Illustration conceptuelle

Imagine ton conteneur comme une **canette jetable 🥤** :

* Tu l’utilises → tu le jettes → tu en prends une nouvelle.\
  Contrairement à une **bouteille rechargeable en verre 🍾** où tu dois nettoyer, garder et entretenir.

Docker encourage le modèle **canette jetable** → pratique, rapide, sans état persistant.

***

### ✅ Résumé

* Un conteneur doit être **éphémère, stateless, jetable**.
* Stockage et configuration doivent venir **de l’extérieur** (volumes, env vars, services).
* Cette approche facilite la **scalabilité, la résilience et la sécurité**.

## 🧹 Bonnes pratiques de construction – **N’installez pas de paquets inutiles**

***

### 🎯 Le principe

Lors de la création d’images, on est parfois tenté d’installer des outils "au cas où" (par exemple un éditeur de texte comme `vim` dans une image de base de données).\
👉 Mauvaise idée !

Chaque paquet supplémentaire =

* 📦 **plus de dépendances**,
* 🕰️ **plus de temps de build**,
* 🐘 **une image plus lourde**,
* 🔓 **une surface d’attaque plus grande** (plus de failles potentielles).

***

### 🚫 Exemple de mauvaise pratique

Un Dockerfile qui installe des outils non nécessaires :

```dockerfile
FROM postgres:16
RUN apt-get update && apt-get install -y vim curl
```

👉 Ici, `vim` n’apporte rien pour une image PostgreSQL, mais alourdit et complexifie inutilement l’image.

***

### ✅ Bonne pratique

N’installe que ce qui est **strictement nécessaire** à ton application.\
Si tu as besoin ponctuellement d’outils pour du **debug**, utilise un conteneur séparé (par exemple basé sur `alpine` ou `ubuntu`) pour diagnostiquer ton système.

Exemple minimaliste :

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "main.py"]
```

👉 Ici, on ne garde **que Python et les dépendances utiles** à l’application.

***

### 📸 Illustration conceptuelle

Imagine ton image comme un **sac à dos de randonnée 🎒** :

* Si tu le charges d’objets inutiles (textes, gadgets, outils que tu n’utiliseras jamais), il devient **lourd et encombrant**.
* Si tu ne prends que l’essentiel, ton sac reste **léger, rapide et efficace**.

***

### ✅ Résumé

* **Moins de paquets = mieux** 🎯.
* N’installe que ce dont ton application a **réellement besoin**.
* Pas de "juste au cas où" → chaque outil doit avoir une vraie raison d’être dans ton image.

## 🧩 Bonnes pratiques de construction – **Dissocier les applications (Decouple applications)**

***

### 🎯 Le principe

👉 **Chaque conteneur doit avoir un seul rôle clair**.\
L’idée est de découper une application en **plusieurs services autonomes**, chacun tournant dans son propre conteneur.

Exemple classique d’une stack web :

* 🌐 **Web serveur** (ex. Nginx ou Apache),
* 🗄️ **Base de données** (ex. PostgreSQL, MySQL),
* ⚡ **Cache en mémoire** (ex. Redis, Memcached).

Chaque service → **1 conteneur indépendant**, relié aux autres via un **réseau Docker**.

***

### 🚀 Pourquoi découpler ?

* **Scalabilité horizontale** : tu peux scaler indépendamment (par ex. 5 instances web + 1 DB + 2 Redis).
* **Réutilisabilité** : le même conteneur Redis peut être utilisé dans plusieurs projets.
* **Maintenance simplifiée** : si un service tombe, tu remplaces _uniquement_ le conteneur fautif.
* **Modularité & clarté** : chaque conteneur fait UNE chose → plus facile à comprendre et à déboguer.

***

### ⚖️ La règle du “1 conteneur = 1 process”

Une bonne **règle de base** est de limiter chaque conteneur à **un seul processus principal**.

Mais attention ⚠️ → ce n’est **pas une règle absolue** :

* Certains programmes gèrent eux-mêmes plusieurs processus (ex. **Celery** avec plusieurs workers, **Apache** qui spawn un process par requête).
* Tu peux aussi démarrer un conteneur avec un **init process** pour gérer proprement les sous-processus.

👉 Utilise ton **jugement** pour garder tes conteneurs **modulaires et propres**, sans tomber dans l’extrême.

***

### 🔗 Communication entre conteneurs

Si tes conteneurs dépendent les uns des autres :

* Utilise les **réseaux Docker** (`docker network create mynet`) pour les connecter.
* Exemple : ton application web peut communiquer avec la base via `db:5432` au lieu de `localhost`.

***

### 📸 Illustration conceptuelle

Imagine ton application comme un **ensemble de Lego 🧱** :

* Chaque pièce = un conteneur avec un rôle précis.
* Tu peux les **assembler**, **remplacer**, ou **multiplier** sans casser toute la construction.

***

### ✅ Résumé

* **Un conteneur = une responsabilité claire**.
* Favorise la **modularité** et la **scalabilité horizontale**.
* La règle du “1 process par conteneur” est une bonne base, mais garde de la souplesse.
* Connecte tes conteneurs via des **réseaux Docker**.

## 📑 Bonnes pratiques de construction – **Trier les arguments multi-lignes**

***

### 🎯 Le principe

Quand tu as une commande avec **plusieurs éléments listés sur plusieurs lignes** (souvent lors d’un `apt-get install` ou `pip install`), trie-les **par ordre alphabétique**.

👉 Pourquoi ?

* ✅ Plus facile à **maintenir** : tu retrouves rapidement un paquet.
* ✅ Tu évites les **doublons accidentels**.
* ✅ Les **revues de code (PRs)** sont plus claires → on voit immédiatement ce qui a changé.
* ✅ Plus simple à mettre à jour (ajout/suppression rapide).

💡 Astuce : ajoute toujours un **espace avant le `\`** pour rendre la continuité encore plus lisible.

***

### 🚫 Exemple non trié (confus)

```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
  git \
  subversion \
  bzr \
  mercurial \
  cvs \
  && rm -rf /var/lib/apt/lists/*
```

Ici, impossible de voir en un coup d’œil si `git` est déjà listé, ou si l’ordre a du sens.

***

### ✅ Exemple trié (clair et propre)

```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
  bzr \
  cvs \
  git \
  mercurial \
  subversion \
  && rm -rf /var/lib/apt/lists/*
```

👉 Lecture fluide, aucun doublon, maintenance facilitée.

***

### 📸 Illustration conceptuelle

Imagine que tu ranges ta bibliothèque 📚 :

* Sans tri → tu cherches ton livre partout, c’est le chaos.
* Avec tri alphabétique → tu trouves tout en quelques secondes.

Même logique pour les listes multi-lignes dans un Dockerfile.

***

### ✅ Résumé

* Toujours **trier alphabétiquement** les listes d’installations ou de dépendances.
* Ajoute un **espace avant le `\`** pour la lisibilité.
* Gain énorme en **clarté, maintenance et qualité des PRs**.

## ⚡ Bonnes pratiques de construction – **Tirer parti du cache de build**

***

### 📦 Le principe

Lorsque Docker construit une image à partir d’un **Dockerfile**, il suit les instructions **ligne par ligne** (de haut en bas).\
👉 À chaque étape, Docker regarde :

* si le résultat de l’instruction existe déjà dans le **cache de build**,
* et s’il peut **réutiliser** cette version au lieu de tout reconstruire.

C’est ce mécanisme qui fait que parfois ton build est **ultra-rapide** (car il réutilise le cache), et parfois **plus lent** (car une étape a invalidé le cache).

***

### 🔄 Comprendre l’invalidation du cache

⚠️ Si une instruction change, toutes les **instructions suivantes** sont également **reconstruites**.\
Exemple :

```dockerfile
# Étape 1
FROM python:3.12

# Étape 2
COPY requirements.txt .

# Étape 3
RUN pip install -r requirements.txt

# Étape 4
COPY . .
```

* Si tu modifies **`requirements.txt`**, l’étape 2 change → le cache de l’étape 3 est invalidé → `pip install` doit être relancé.
* Mais si tu modifies uniquement un fichier de ton code (pas `requirements.txt`), l’étape 3 est **réutilisée depuis le cache**, donc pas besoin de réinstaller les dépendances 🎉.

***

### 🛠️ Bonnes pratiques pour optimiser le cache

#### 1. **Ordre des instructions = crucial**

Place les instructions **les moins susceptibles de changer** en haut de ton Dockerfile.\
👉 Exemple :

* `apt-get install` (souvent stable) en haut,
* `COPY . .` (souvent modifié) en bas.

***

#### 2. **Séparer la copie des dépendances**

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

✅ Ainsi, si tu modifies ton code mais pas `requirements.txt`, le cache garde l’installation des dépendances → build plus rapide.

***

#### 3. **Nettoyer intelligemment**

Toujours nettoyer (`apt-get clean`, `rm -rf /var/lib/apt/lists/*`) **dans la même couche** que l’installation → sinon tu gardes des fichiers inutiles dans le cache.

***

#### 4. **Utiliser `--target` pour reconstruire partiellement**

Avec les builds multi-étapes (`multi-stage builds`), tu peux cibler un stade précis sans tout reconstruire :

```bash
docker build --target build-env -t myapp:build .
```

***

#### 5. **Forcer un build propre si nécessaire**

Si tu veux **ignorer le cache** (par ex. pour récupérer les dernières mises à jour de sécurité) :

```bash
docker build --no-cache -t myapp:latest .
```

***

### 📸 Illustration conceptuelle

Imagine que tu cuisines 🍳 :

* Tu notes chaque étape (recette).
* Si tu refais la même recette demain, tu peux **réutiliser les étapes déjà préparées** (sauce, légumes coupés).
* Mais si tu changes un ingrédient (ex. `requirements.txt`), tu dois refaire toutes les étapes suivantes.

***

### ✅ Résumé

* Docker construit image **instruction par instruction**, avec un cache pour accélérer.
* L’ordre du Dockerfile est **stratégique** pour optimiser la réutilisation du cache.
* Sépare les dépendances et le code applicatif pour éviter de tout reconstruire.
* Utilise `--no-cache` si tu veux forcer un build frais.

## 📌 Bonnes pratiques de construction – **Épingler les versions d’images de base (Pin base image versions)**

***

### 🧩 Le problème des tags "flottants"

Quand tu écris un Dockerfile avec une image de base, tu utilises généralement une **balise (tag)**, par ex. :

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine:3.21
```

👉 Or, les **tags sont mutables** :

* Aujourd’hui, `alpine:3.21` peut pointer vers **3.21.1**
* Dans 3 mois, le même tag peut pointer vers **3.21.4**

Cela a deux conséquences :

* ✅ Tu bénéficies automatiquement des **patchs de sécurité**
* ❌ Mais tu n’as **aucune garantie de reproductibilité** (ton image n’est pas exactement la même selon la date du build)
* ❌ Pas d’**audit trail clair** → difficile de savoir quelle version exacte a été utilisée en prod

***

### 📌 La solution : épingler par digest

Un digest (`sha256:...`) est un **identifiant unique et immuable** de l’image.\
En l’utilisant, tu garantis que ton build utilisera **toujours exactement la même image**, peu importe les mises à jour côté éditeur.

Exemple :

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine:3.21@sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c
```

👉 Ici :

* Tu indiques toujours `alpine:3.21` pour la lisibilité
* Mais tu précises le digest exact (`sha256:...`) qui ne changera jamais

***

### ⚖️ Avantages / Inconvénients

#### ✅ Avantages

* **Reproductibilité totale** : même image à chaque build
* **Chaîne d’approvisionnement sécurisée** : pas de remplacement surprise par l’éditeur
* **Audit clair** : tu sais exactement quelle version tourne

#### ❌ Inconvénients

* **Maintenance plus lourde** : tu dois rechercher le digest à chaque mise à jour
* **Pas de mise à jour automatique** → tu peux rater des correctifs de sécurité si tu n’updates pas manuellement

***

### 🚀 Automatiser avec Docker Scout

Docker propose **Docker Scout** qui aide à :

* Vérifier si ton image de base est **à jour**
* Détecter si ton digest est devenu obsolète
* Automatiser les **mises à jour via pull requests** → Scout peut proposer un nouveau digest dans tes Dockerfiles

👉 Résultat : tu combines le meilleur des deux mondes :

* Contrôle total via digest
* Automatisation des updates avec audit trail (PRs claires et traçables)

***

### 📸 Illustration conceptuelle

* Utiliser uniquement `alpine:3.21` = 🚪 **porte qui change de clé à chaque fois** → parfois tu entres, parfois non
* Utiliser `alpine:3.21@sha256:xxxx` = 🔒 **clé unique et fixe** → tu es sûr que c’est la bonne porte
* Utiliser `alpine:3.21@sha256:xxxx` + Docker Scout = 🔑 **clé fixe**, mais avec un **majordome** qui t’avertit quand une nouvelle serrure est disponible

***

### ✅ Résumé

* Les tags (`alpine:3.21`) sont **mutables** → bon pour les patchs auto, mauvais pour la reproductibilité
* Les digests (`sha256:...`) sont **immuables** → parfaits pour la sécurité et l’audit
* Docker Scout permet d’automatiser la mise à jour des digests en gardant une **traçabilité claire**

## 📌 Build and test your images in CI

***

### 🧩 Pourquoi ?

* 🛡️ **Sécurité & stabilité** : chaque changement est validé automatiquement → moins de risques d’images corrompues ou vulnérables.
* 🔄 **Reproductibilité** : les builds ne dépendent plus de ta machine locale ("it works on my machine").
* ⚡ **Rapidité** : bugs et failles détectés tôt dans le cycle.
* 📦 **Automatisation** : tes images sont construites, testées, et taguées dès chaque commit/PR.

***

### ⚙️ Exemple avec GitHub Actions

Un workflow simple pour construire et tester une image :

```yaml
name: CI Docker Build

on:
  pull_request:
  push:
    branches: [ "main" ]

jobs:
  docker:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build image
        run: docker build -t myapp:test .

      - name: Run tests
        run: docker run --rm myapp:test pytest
```

#### 🔎 Décomposition :

1. **Checkout** → récupère ton code
2. **Setup Buildx** → prépare l’environnement multi-platform
3. **Login DockerHub** → permet de push si besoin
4. **Build image** → construit ton image avec un tag local `myapp:test`
5. **Run tests** → lance tes tests unitaires/fonctionnels à l’intérieur du conteneur

***

### 🏷️ Tagging automatique

Pour éviter d’écraser les versions :

* Utilise le **SHA du commit** ou la **version du package** comme tag :

```yaml
- name: Build and push image
  run: |
    IMAGE=myorg/myapp:${{ github.sha }}
    docker build -t $IMAGE .
    docker push $IMAGE
```

👉 Ça garantit que chaque commit correspond à une image traçable.

***

### 📸 Illustration conceptuelle

Imagine ta CI/CD comme une **chaîne de production automatisée** :

* 📦 Tu poses ton code (commit/PR) sur le tapis
* 🏗️ La chaîne construit l’image
* 🧪 Elle la teste
* ✅ Elle ne laisse passer que les images validées et traçables

***

### ✅ Résumé

* 🔄 Intègre **build + test** de tes images dans ton pipeline CI/CD
* 🏷️ Utilise des **tags uniques (SHA, version)** pour traçabilité
* 🛡️ Les images sont validées avant de passer en **prod**

## 📌 Dockerfile instructions — bonnes pratiques

Chaque instruction (`FROM`, `RUN`, `COPY`, etc.) a ses **règles de bonne utilisation** pour éviter :

* des images trop lourdes ⚖️,
* des builds trop longs 🐌,
* des failles de sécurité 🔓,
* ou des Dockerfiles difficiles à maintenir 🛠️.

***

### ✅ Recommandations générales

1. **Sois explicite dans tes instructions**\
   → utilise des tags précis pour les images de base (`FROM ubuntu:24.04` plutôt que `FROM ubuntu:latest`).
2. **Réduis le nombre de couches**\
   → combine les commandes `RUN` avec `&&` au lieu de multiplier les lignes.
3. **Trie & nettoie**\
   → trie les paquets par ordre alphabétique et supprime les caches (`apt-get clean`, `rm -rf /var/lib/apt/lists/*`).
4. **Sépare build et runtime**\
   → multi-stage builds : un premier stage pour compiler, un second minimal pour exécuter.
5. **Minimise le contexte de build**\
   → `.dockerignore` doit exclure logs, docs, node\_modules inutiles, etc.
6. **Sécurité avant tout**\
   → évite `ADD` pour des fichiers locaux (utilise `COPY`) et n’exécute pas en `root`.

***

### 📒 Exemple illustratif

Un mauvais Dockerfile 👎 :

```dockerfile
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install -y curl git python3
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
CMD ["python3", "main.py"]
```

Un bon Dockerfile 👍 (multi-stage, optimisé, sécurisé) :

```dockerfile
# Étape build
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Étape runtime
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /usr/local /usr/local
COPY . .
USER 1001
CMD ["python", "main.py"]
```

#### ✅ Points forts :

* `python:3.11-slim` → base image minimale
* multi-stage → runtime allégé
* `--no-cache-dir` → pas de dépendances inutiles
* `USER 1001` → pas root

***

### 💡 Outils utiles

* **Docker VS Code Extension** →\
  🔎 linting + autocomplétion + scan de vulnérabilités directement dans ton IDE.
* **Hadolint** → linter Dockerfile en CLI.

## 🏗️ Instruction `FROM`

La directive **`FROM`** définit l’**image de base** sur laquelle ton image Docker sera construite.\
Chaque image Docker (sauf `FROM scratch`) est construite à partir d’une autre image.

***

### ✅ Bonnes pratiques pour `FROM`

#### 1. Utiliser des images **officielles et maintenues**

* Favoriser les images officielles de Docker Hub.
* Exemple : `FROM ubuntu:24.04`, `FROM node:20-alpine`, `FROM python:3.11-slim`.

#### 2. Préférer les variantes **légères**

* `alpine` : très petite (≈ 6 MB), distribution Linux complète → rapide et peu de surface d’attaque.
* `slim` : un compromis entre taille et compatibilité.

Exemple :

```dockerfile
FROM node:20-alpine
```

#### 3. Éviter `latest`

* `FROM ubuntu:latest` est dangereux car mutable → ton build peut casser du jour au lendemain.
* Toujours **pinner une version** (`FROM ubuntu:24.04` ou mieux avec un **digest SHA256**).

Exemple avec digest :

```dockerfile
FROM alpine:3.21@sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c
```

#### 4. Utiliser `scratch` pour les images minimales

* `scratch` est une image vide.
* Très utile pour les binaires statiques (Go, Rust, C).

Exemple :

```dockerfile
FROM scratch
COPY mybinary /
CMD ["/mybinary"]
```

#### 5. Multi-stage builds

* On peut utiliser **plusieurs `FROM`** dans un même Dockerfile.
* Utile pour séparer la compilation et le runtime.

Exemple :

```dockerfile
# Étape build
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp .

# Étape runtime minimal
FROM alpine:3.21
COPY --from=builder /app/myapp /usr/local/bin/
CMD ["myapp"]
```

***

### 📒 Illustration conceptuelle

Imaginons ton image comme un **sandwich** 🥪 :

* `FROM` = **le pain** (la base : Ubuntu, Alpine, etc.)
* `RUN`, `COPY` = **les ingrédients** que tu ajoutes
* `CMD`/`ENTRYPOINT` = **comment tu le manges** (le processus de démarrage).

## 🏷️ Instruction `LABEL`

Les **labels** sont des **métadonnées** que tu ajoutes à ton image Docker.\
Ils ne changent pas le fonctionnement de ton conteneur, mais servent à **documenter**, **organiser** et **automatiser** la gestion de tes images.

***

### ✅ Bonnes pratiques pour `LABEL`

#### 1. Définir des métadonnées utiles

Quelques cas d’usage :

*   **Versioning** :

    ```dockerfile
    LABEL version="1.0.0"
    ```
*   **Auteur / Mainteneur** :

    ```dockerfile
    LABEL maintainer="john.doe@example.com"
    ```
*   **Projet & Vendor** :

    ```dockerfile
    LABEL org.opencontainers.image.title="MyApp" \
          org.opencontainers.image.vendor="ACME Corp"
    ```
*   **Licence & dépôt source** :

    ```dockerfile
    LABEL org.opencontainers.image.licenses="MIT" \
          org.opencontainers.image.source="https://github.com/acme/myapp"
    ```

👉 **Astuce** : privilégie les labels du **standard OCI (Open Containers Initiative)** pour assurer une meilleure compatibilité avec les outils :\
🔗 [Spécification OCI Annotations](https://github.com/opencontainers/image-spec/blob/main/annotations.md)

Exemples clés standards :

* `org.opencontainers.image.title`
* `org.opencontainers.image.description`
* `org.opencontainers.image.url`
* `org.opencontainers.image.source`
* `org.opencontainers.image.version`
* `org.opencontainers.image.licenses`

***

#### 2. Syntaxes acceptées

*   **Un label par ligne** :

    ```dockerfile
    LABEL com.example.version="0.0.1-beta"
    LABEL vendor="ACME Incorporated"
    ```
*   **Plusieurs labels sur une ligne** :

    ```dockerfile
    LABEL com.example.version="0.0.1-beta" com.example.release-date="2015-02-12"
    ```
*   **Labels multi-lignes (lisibilité ++)** :

    ```dockerfile
    LABEL vendor="ACME Incorporated" \
          com.example.is-beta="" \
          com.example.is-production="" \
          com.example.version="0.0.1-beta" \
          com.example.release-date="2015-02-12"
    ```

⚠️ Attention :

*   Si une valeur contient des **espaces**, il faut la **citer** ou **échapper** :

    ```dockerfile
    LABEL vendor="ZENITH Incorporated"
    # ou
    LABEL vendor=ZENITH\ Incorporated
    ```
* Les guillemets intérieurs doivent être échappés.

***

#### 3. Visualiser les labels

Après un build :

```bash
docker inspect <image_id> --format '{{json .Config.Labels}}' | jq
```

Exemple de sortie :

```json
{
  "version": "1.0.0",
  "maintainer": "john.doe@example.com",
  "org.opencontainers.image.source": "https://github.com/acme/myapp"
}
```

***

### 📒 Illustration

Imagine que ton image Docker soit une **boîte de conserve** 🥫 :

* Le **contenu** = ton app + dépendances
* Les **labels** = l’**étiquette** sur la boîte → infos sur la recette, date de production, version, fabricant.\
  Sans étiquette, tu dois ouvrir la boîte pour savoir ce qu’il y a dedans 😅.

## ⚙️ Instruction `RUN`

L’instruction **`RUN`** exécute une commande dans une nouvelle **couche intermédiaire** de l’image Docker.\
👉 Elle est utilisée pour **installer des paquets**, **configurer des dépendances**, ou **préparer l’environnement** de ton image.

***

### ✨ Bonnes pratiques

#### 🔹 1. Rendre les commandes lisibles

Quand les commandes sont longues, découpe-les sur **plusieurs lignes** avec `\` :

```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
    package-bar \
    package-baz \
    package-foo
```

💡 Astuce : tu peux aussi utiliser les **here documents** (`<<EOF`) pour enchaîner plusieurs commandes sans utiliser `&&` :

```dockerfile
RUN <<EOF
apt-get update
apt-get install -y --no-install-recommends \
    package-bar \
    package-baz \
    package-foo
EOF
```

***

#### 🔹 2. Bien utiliser `apt-get`

Sur les images Debian/Ubuntu, la commande **`apt-get`** est courante, mais il faut respecter certaines règles :

✅ Toujours combiner `apt-get update` et `apt-get install` dans la **même commande** :

```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends curl
```

❌ Mauvais exemple (risque de cache obsolète et paquets dépassés) :

```dockerfile
RUN apt-get update
RUN apt-get install -y curl
```

👉 Pourquoi ? Parce que Docker met en cache chaque `RUN`. Si tu ajoutes un paquet plus tard, l’étape `apt-get update` peut être réutilisée depuis le cache, ce qui fait que tes paquets ne seront **jamais mis à jour**.

***

#### 🔹 3. Utiliser le **cache busting** et le **version pinning**

* **Cache busting** : permet de forcer Docker à ré-exécuter `apt-get update` → garantit des paquets frais.
*   **Version pinning** : installe une **version précise** d’un paquet pour éviter les surprises :

    ```dockerfile
    RUN apt-get update && apt-get install -y --no-install-recommends \
        s3cmd=1.1.* \
        && rm -rf /var/lib/apt/lists/*
    ```

***

#### 🔹 4. Nettoyer après installation

⚡ Pour réduire la taille de l’image, supprime le cache `apt` après installation :

```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*
```

💡 Les images officielles Debian/Ubuntu exécutent déjà `apt-get clean`, mais supprimer `/var/lib/apt/lists` reste une bonne pratique.

***

#### 🔹 5. Gérer les pipes (`|`)

Par défaut, Docker utilise `/bin/sh -c`.\
👉 Problème : dans une commande avec un **pipe (`|`)**, seul le **dernier programme** détermine le succès.

Exemple dangereux ❌ :

```dockerfile
RUN wget -O - https://some.site | wc -l > /number
```

👉 Ici, même si `wget` échoue, l’étape réussit si `wc -l` s’exécute.

✅ Solution : activer `pipefail` :

```dockerfile
RUN set -o pipefail && wget -O - https://some.site | wc -l > /number
```

Si ton shell ne supporte pas `pipefail` (ex. `dash` sur Debian), utilise la forme **exec** :

```dockerfile
RUN ["/bin/bash", "-c", "set -o pipefail && wget -O - https://some.site | wc -l > /number"]
```

***

### 📒 Exemple complet et propre

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    git \
    make \
    && rm -rf /var/lib/apt/lists/*
```

➡️ Ici :

* `apt-get update` et `install` sont regroupés ✅
* les paquets sont listés ligne par ligne (lisibilité + pas de doublons) ✅
* `--no-install-recommends` limite les dépendances inutiles ✅
* suppression du cache apt (image plus légère) ✅

***

⚡ **Résumé** :

* Découpe tes `RUN` longs → lisibilité 📖
* Toujours combiner `update` + `install` → éviter les caches obsolètes 🛑
* Version pinning si tu veux de la stabilité 🔒
* Nettoyer après install → images plus petites 🪶
* Attention aux pipes → utilise `pipefail` ⚡

## ⚙️ Instruction `CMD`

👉 L’instruction **`CMD`** définit la **commande par défaut** exécutée lorsqu’un conteneur est lancé à partir d’une image.

* Elle peut inclure un **programme** + **arguments**,
* Ou simplement une **commande interactive** (ex. un shell).
* Elle est **surchargée** si l’utilisateur passe une commande lors du `docker run`.

***

### 🔹 Bonnes pratiques avec `CMD`

#### 1. Utiliser la forme **exec** (JSON array) ✅

La meilleure pratique est d’écrire :

```dockerfile
CMD ["executable", "param1", "param2"]
```

Pourquoi ?

* Pas d’appel implicite à `/bin/sh -c` (donc plus sûr).
* Gestion plus fiable des signaux (SIGTERM, SIGINT).

Exemple (service Apache qui reste au premier plan) :

```dockerfile
CMD ["apache2", "-DFOREGROUND"]
```

***

#### 2. Usage pour les services 🖥️

Si ton image est dédiée à un service (web, API, worker, etc.), `CMD` doit lancer **ce service**.\
➡️ Cela assure que le conteneur fait “ce pourquoi il a été conçu”.

***

#### 3. Usage pour les environnements interactifs 🐚

Pour des images servant de **base interactive** (langages, shells…), on utilise `CMD` pour fournir une **console prête à l’emploi**.

Exemples :

```dockerfile
CMD ["perl", "-de0"]
CMD ["python"]
CMD ["php", "-a"]
```

Ainsi :

```bash
docker run -it python
```

➡️ Te donne directement une **REPL Python** utilisable.

***

#### 4. Cas à éviter 🚫

Il est **rarement recommandé** d’utiliser `CMD` uniquement comme une **liste de paramètres** en complément d’`ENTRYPOINT` :

```dockerfile
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```

👉 Ce pattern peut être utile dans des cas avancés, mais il devient **confus** si tes utilisateurs ne savent pas comment `ENTRYPOINT` et `CMD` interagissent.

***

### 🔹 Interaction avec `docker run` ⚡

* Si l’utilisateur ne passe **aucune commande**, Docker utilise **`CMD`**.
* Si une commande est passée, **elle remplace `CMD`**.

Exemple :

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu
CMD ["echo", "Hello from CMD"]
```

```bash
docker run myimage
# ➝ affiche "Hello from CMD"

docker run myimage echo "Overridden"
# ➝ affiche "Overridden"
```

***

### 📒 Exemple complet

Image Python prête à l’emploi :

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.11-slim

WORKDIR /app
COPY . .

# Lance Python en mode interactif par défaut
CMD ["python"]
```

Utilisation :

```bash
docker run -it mypython
# ➝ Ouvre un shell Python interactif
```

***

✅ **Résumé :**

* Toujours préférer la forme **exec** (`["..."]`).
* `CMD` = **commande par défaut**, mais **remplaçable** avec `docker run`.
* Idéal pour :
  * Services (ex: `CMD ["apache2", "-DFOREGROUND"]`)
  * Environnements interactifs (`CMD ["python"]`)
* Évite d’utiliser `CMD` uniquement comme arguments d’un `ENTRYPOINT`, sauf si nécessaire.

## ⚓ Instruction `EXPOSE`

👉 **`EXPOSE`** indique les **ports** sur lesquels le conteneur **écoute** pour accepter des connexions.\
C’est une **métadonnée déclarative** dans ton `Dockerfile`, et non une règle de pare-feu ou un mappage automatique de ports.

***

### 🔹 Comment ça marche ?

Exemple classique avec Apache :

```dockerfile
EXPOSE 80
```

➡️ Cela documente que ton conteneur écoute sur le port **80** (HTTP).

Autres exemples :

* MongoDB → `EXPOSE 27017`
* PostgreSQL → `EXPOSE 5432`
* Application custom sur port 8080 → `EXPOSE 8080`

***

### 🔹 Attention : `EXPOSE` ≠ accès externe ❌

* `EXPOSE` **ne publie pas** le port sur ta machine hôte.
* Pour rendre le service accessible depuis l’extérieur, il faut mapper le port avec `-p` ou `--publish` :

```bash
docker run -p 8080:80 my-apache
```

➡️ Ici, le **port 8080** de ta machine hôte est relié au **port 80** du conteneur.

***

### 🔹 Utilisation dans les réseaux Docker 🌐

Quand tu relies des conteneurs entre eux (via des **networks**), l’instruction `EXPOSE` sert surtout de **documentation** et de convention.\
Docker peut utiliser ces métadonnées pour :

* Créer des **variables d’environnement** (ex: `MYSQL_PORT_3306_TCP`).
* Aider à l’**auto-configuration** entre conteneurs liés.

***

### 📒 Exemple complet

Un serveur Node.js :

```dockerfile
# syntax=docker/dockerfile:1
FROM node:20

WORKDIR /app
COPY . .

RUN npm install
EXPOSE 3000

CMD ["npm", "start"]
```

Exécution :

```bash
docker build -t my-node-app .
docker run -p 8080:3000 my-node-app
```

➡️ Ton appli écoute sur **3000** dans le conteneur, mais est exposée sur **8080** côté hôte.

***

✅ **Résumé :**

* `EXPOSE` documente les **ports écoutés** par l’application.
* Il **ne publie pas automatiquement** le port → `-p` ou `--publish` sont nécessaires.
* Sert surtout de convention + interopérabilité entre conteneurs.

## ⚙️ Instruction `ENV`

👉 L’instruction **`ENV`** définit des **variables d’environnement** dans ton image.\
Ces variables sont ensuite disponibles :

* au moment de la construction de l’image 🏗️,
* dans le conteneur au moment de l’exécution ▶️.

***

### 🔹 Cas d’usage courants

#### 1. Mettre à jour le `PATH` 🔍

Tu peux rendre un logiciel accessible sans préciser son chemin complet :

```dockerfile
ENV PATH=/usr/local/nginx/bin:$PATH
```

➡️ Ainsi, `CMD ["nginx"]` fonctionne directement ✅.

***

#### 2. Définir des variables spécifiques aux services 🗄️

Exemple avec **Postgres** :

```dockerfile
ENV PGDATA=/var/lib/postgresql/data
```

Cela permet au service de connaître son répertoire de données.

***

#### 3. Centraliser des **versions de logiciels** 📦

Au lieu de “hardcoder” une version à plusieurs endroits, tu la définis une fois :

```dockerfile
ENV PG_MAJOR=9.3
ENV PG_VERSION=9.3.4

RUN curl -SL https://example.com/postgres-$PG_VERSION.tar.xz \
    | tar -xJC /usr/src/postgres

ENV PATH=/usr/local/postgres-$PG_MAJOR/bin:$PATH
```

💡 C’est comme utiliser des **constantes dans un programme** : un seul changement met à jour toute l’image.

***

### 🔹 ⚠️ Attention aux couches Docker

Chaque `ENV` crée une **nouvelle couche intermédiaire**, tout comme `RUN`.\
👉 Même si tu `unset` une variable plus tard, **elle reste stockée dans l’historique de l’image**.

Exemple :

```dockerfile
FROM alpine
ENV ADMIN_USER="mark"
RUN echo $ADMIN_USER > ./mark
RUN unset ADMIN_USER
```

Test :

```bash
docker run --rm test sh -c 'echo $ADMIN_USER'
```

➡️ Résultat : `mark` (la variable persiste malgré le unset ⚠️).

***

### 🔹 Solution : tout gérer dans une seule couche 🛡️

Pour éviter les fuites de variables sensibles (comme des mots de passe), il faut **définir, utiliser et unset** dans la **même commande `RUN`** :

```dockerfile
FROM alpine
RUN export ADMIN_USER="mark" \
    && echo $ADMIN_USER > ./mark \
    && unset ADMIN_USER
CMD sh
```

Cette fois :

```bash
docker run --rm test sh -c 'echo $ADMIN_USER'
```

➡️ Résultat : **rien**, la variable est bien nettoyée ✅.

***

### 📒 Exemple complet

Un `Dockerfile` qui installe une app avec version configurable :

```dockerfile
# syntax=docker/dockerfile:1
FROM node:20

ENV APP_VERSION=1.2.0
ENV APP_PATH=/usr/local/app

WORKDIR $APP_PATH

RUN curl -sL https://example.com/myapp-$APP_VERSION.tar.gz | tar -xz -C $APP_PATH

ENV PATH=$APP_PATH/bin:$PATH

CMD ["myapp"]
```

💡 Ici :

* `APP_VERSION` = facile à changer quand une nouvelle version sort
* `APP_PATH` = centralisé → moins d’erreurs
* `PATH` = mis à jour → la commande `myapp` fonctionne directement

***

### ✅ Résumé

* `ENV` = définit des variables d’environnement pour la **construction** et l’**exécution**.
* Sert à : mettre à jour le `PATH`, configurer des services, centraliser les versions.
* ⚠️ Attention : les variables persistent dans l’historique → ne pas stocker de secrets.
* Pour protéger des données sensibles → définir, utiliser et unset dans **une seule couche**.

## ⚖️ `ADD` vs `COPY`

### 🔹 **COPY**

* **Usage principal** : copier **des fichiers locaux** du _build context_ (ton projet) → vers le conteneur.
* Supporte aussi la copie entre **multi-stage builds** (`COPY --from=builder ...`).
* C’est la commande **par défaut** pour presque tous les cas.

👉 Exemple simple :

```dockerfile
COPY package.json /app/
```

👉 Exemple multi-stage :

```dockerfile
FROM node:20 AS builder
WORKDIR /app
COPY . .
RUN npm install && npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

***

### 🔹 **ADD**

* Fait tout ce que `COPY` fait, **plus deux fonctionnalités supplémentaires** :
  1. **Télécharger** des fichiers depuis une **URL distante (HTTP/HTTPS/Git)**
  2. **Décompresser automatiquement** les fichiers `.tar` quand ils proviennent du build context

👉 Exemple avec téléchargement direct :

```dockerfile
ARG DOTNET_VERSION=8.0.0-preview.6.23329.7
ADD --checksum=sha256:270d731bd08040c6a3228115de1f74b91cf441c584139ff8f8f6503447cebdbb \
    https://dotnetcli.azureedge.net/dotnet/Runtime/$DOTNET_VERSION/dotnet-runtime-$DOTNET_VERSION-linux-arm64.tar.gz /dotnet.tar.gz
```

👉 Exemple avec extraction auto d’un `.tar` :

```dockerfile
ADD app.tar.gz /app/
```

Ici, le contenu est directement **décompressé** dans `/app`.

***

### 🔹 ⚡ Comparaison rapide

| Instruction | Fonctionnalités principales                                   | Cas recommandé          |
| ----------- | ------------------------------------------------------------- | ----------------------- |
| **COPY**    | Copier fichiers/dossiers depuis le contexte ou un autre stage | **Toujours par défaut** |
| **ADD**     | + Téléchargement URL                                          | + Extraction `.tar`     |

***

### 🔹 Optimisation : `RUN --mount` au lieu de `COPY`

Si tu veux juste utiliser un fichier temporairement (ex. `requirements.txt` pour un `pip install`), pas besoin de le copier dans l’image → tu peux **le monter** juste pour une commande :

```dockerfile
RUN --mount=type=bind,source=requirements.txt,target=/tmp/requirements.txt \
    pip install --requirement /tmp/requirements.txt
```

👉 Avantages :

* Pas stocké dans l’image finale
* Pas de pollution de couches
* Plus rapide ⚡

***

### ✅ Règle de base

* **Toujours utiliser `COPY`** sauf si tu as un vrai besoin de `ADD`.
* Si tu télécharges depuis le net → `ADD` est mieux que `wget/curl` (cache + checksum).
* Si tu as besoin d’extraire un `.tar` → `ADD`.
* Pour les fichiers temporaires → préfère `RUN --mount`.

## 🔹 ENTRYPOINT

* Définit **le binaire ou script principal** que ton conteneur doit exécuter.
* Fait en sorte que ton image puisse être exécutée **comme si c’était ce binaire lui-même**.
* Ne peut pas être facilement remplacé à l’exécution (sauf avec `--entrypoint`).

👉 Exemple minimal :

```dockerfile
ENTRYPOINT ["s3cmd"]
CMD ["--help"]
```

* Ici, `ENTRYPOINT` fige `s3cmd` comme programme principal.
* `CMD ["--help"]` fournit les **arguments par défaut**.
* Donc :
  * `docker run s3cmd` → exécute `s3cmd --help`
  * `docker run s3cmd ls s3://bucket` → exécute `s3cmd ls s3://bucket`

***

## 🔹 ENTRYPOINT avec script d’initialisation

Très utile quand :

* Tu dois préparer l’environnement avant de lancer ton programme (changer les droits, init DB, etc.).
* Tu veux que le conteneur soit **polyvalent** (par défaut il lance ton service, mais il peut aussi exécuter d’autres commandes).

👉 Exemple : **Postgres officiel**

```dockerfile
COPY ./docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["postgres"]
```

#### `docker-entrypoint.sh`

```bash
#!/bin/bash
set -e

if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"

    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi

    exec gosu postgres "$@"
fi

exec "$@"
```

#### Ce que ça donne :

* `docker run postgres` → lance Postgres normalement
* `docker run postgres postgres --help` → lance Postgres avec `--help`
* `docker run -it postgres bash` → ouvre Bash à la place

👉 Pourquoi ça marche ?\
Le script utilise `exec "$@"` → ce qui fait que **le processus final devient le PID 1 du conteneur**.\
Cela permet au programme (Postgres ici) de recevoir directement les **signaux Unix** (`SIGTERM`, `SIGINT`…), indispensables pour un arrêt propre.

***

## 🔹 ENTRYPOINT vs CMD : la règle d’or

* **ENTRYPOINT = ce que fait le conteneur (binaire/script principal)**
* **CMD = arguments par défaut (ou fallback)**

👉 Exemple d’usage typique :

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

* `docker run myapp` → `python app.py`
* `docker run myapp -m http.server` → `python -m http.server`

***

## ✅ Bonnes pratiques

1. Utilise **ENTRYPOINT en mode exec** (`["prog", "arg1", ...]`) → évite `/bin/sh -c` qui complique la gestion des signaux.
2. Combine avec `CMD` pour des **arguments par défaut** mais flexibles.
3. Si tu dois faire un pré-traitement (init DB, migration, config), passe par un **script wrapper** en `ENTRYPOINT`.
4. Utilise `exec` dans le script pour que ton vrai programme devienne **PID 1**.

## 🔹 Qu’est-ce que `VOLUME` ?

La directive `VOLUME` dans un `Dockerfile` déclare un ou plusieurs chemins dans le conteneur comme **points de montage persistants**.

👉 Cela signifie :

* Les fichiers écrits dans ce chemin **ne font plus partie de l’image**.
* Ils sont stockés dans un **volume Docker** (ou un bind mount si tu le rediriges manuellement).
* Même si le conteneur est supprimé, le volume **persiste**.

***

## 🔹 Exemple basique

```dockerfile
FROM mysql:8.0
VOLUME ["/var/lib/mysql"]
```

* Ici, MySQL stocke ses données dans `/var/lib/mysql`.
* Grâce au `VOLUME`, les données sont **séparées de l’image**.
* Si tu reconstruis ou supprimes le conteneur → tes données restent.

***

## 🔹 Pourquoi utiliser `VOLUME` ?

✅ Pour tout ce qui est **mutable** (peut changer après le build) :

* Bases de données (`/var/lib/mysql`, `/var/lib/postgresql/data`)
* Configurations modifiables par l’utilisateur
* Logs générés par l’application

✅ Pour tout ce qui est **user-serviceable** (que l’utilisateur doit gérer/persister).

❌ À éviter pour les fichiers **statiques** (code source, binaires), qui doivent rester dans l’image.

***

## 🔹 Utilisation en pratique

#### 1. Déclarer dans le `Dockerfile`

```dockerfile
VOLUME ["/data"]
```

#### 2. Vérifier avec `docker inspect`

```bash
docker inspect mycontainer
```

→ tu verras que `/data` pointe vers un volume géré par Docker (`/var/lib/docker/volumes/...`).

#### 3. Monter un volume explicite

```bash
docker run -d -v mydata:/data myimage
```

#### 4. Ou utiliser un bind mount

```bash
docker run -d -v $(pwd)/data:/data myimage
```

***

## 🔹 Exemple concret : PostgreSQL

```dockerfile
FROM postgres:16
VOLUME ["/var/lib/postgresql/data"]
```

* Les données sont persistées dans `/var/lib/docker/volumes/...`.
*   Tu peux aussi binder ton dossier local si tu veux manipuler les fichiers directement :

    ```bash
    docker run -v $(pwd)/pgdata:/var/lib/postgresql/data postgres:16
    ```

***

## 🔹 Attention aux pièges

1. **Création implicite de volumes**
   * Si tu utilises `VOLUME` dans le Dockerfile, Docker crée automatiquement un volume anonyme si tu n’en fournis pas un.
   * Résultat : tu te retrouves parfois avec des volumes orphelins qui remplissent ton disque.
   * 👉 Bonne pratique : utiliser des **volumes nommés** (`-v mydata:/data`) ou un orchestrateur (Compose, Kubernetes).
2. **Immutabilité de l’image**
   * Tu ne peux pas modifier un `VOLUME` défini dans l’image sans reconstruire l’image.
3. **Backup / restauration**
   * Comme les données sont hors de l’image, il faut penser à gérer les sauvegardes des volumes séparément.

***

## ✅ Règle d’or

👉 **`VOLUME` = pour la persistance des données dynamiques**\
👉 **`COPY`/`ADD` = pour les fichiers statiques**

## 🔹 Pourquoi `USER` ?

Par défaut, un conteneur démarre sous l’utilisateur **root**.\
👉 Ce n’est pas idéal, car si un attaquant exploite ton conteneur, il a potentiellement les droits **root** dans le conteneur (et parfois sur l’hôte en cas de faille).

➡️ **Bonne pratique :** exécuter ton application avec un utilisateur **non-root**, sauf si elle a absolument besoin de privilèges.

***

## 🔹 Exemple basique

```dockerfile
FROM postgres:16

# Créer un groupe et un utilisateur non-root
RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres

# Définir l'utilisateur à utiliser
USER postgres

# Démarrer le service
CMD ["postgres"]
```

👉 Ici, le service **Postgres** tourne sous l’utilisateur `postgres` et non `root`.

***

## 🔹 Détails importants

#### 1. UID/GID explicite

* Par défaut, `useradd` attribue le prochain UID/GID disponible.
* Mais ça peut varier entre builds → **non déterministe**.
* Donc, si tu veux de la stabilité (CI/CD, sécurité, Kubernetes), définis-les :

```dockerfile
RUN groupadd -g 999 postgres && useradd -u 999 -g postgres postgres
```

***

#### 2. Bug avec `useradd` et UID élevé

⚠️ Avec certains UID/GID très grands, un **bug dans `tar` de Go** remplit `/var/log/faillog` avec des `\0` → disque saturé.\
👉 Solution : ajouter `--no-log-init` comme dans l’exemple :

```dockerfile
RUN useradd --no-log-init -u 999 -g postgres postgres
```

***

#### 3. Pas de `sudo` ❌

* Installer `sudo` est inutile dans un conteneur (pas d’interactivité ni gestion d’accès complexe).
* Ça complique et introduit des problèmes avec les TTY et signaux.

👉 Utilise plutôt **`gosu`** ou **`su-exec`** si tu dois démarrer un process root, puis basculer vers un user non-root :

```dockerfile
RUN apt-get update && apt-get install -y gosu
COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]
```

Dans `docker-entrypoint.sh` :

```bash
#!/bin/bash
if [ "$1" = 'postgres' ]; then
    exec gosu postgres "$@"
fi
exec "$@"
```

***

#### 4. Éviter les allers-retours de `USER`

Changer plusieurs fois entre root/non-root = complexité et couches inutiles.\
👉 Regroupe la configuration root **avant** le `USER` final.

Exemple **mauvais** ❌ :

```dockerfile
USER appuser
RUN echo "test" > /tmp/test
USER root
RUN apt-get install -y curl
USER appuser
```

Exemple **propre** ✅ :

```dockerfile
RUN apt-get install -y curl && \
    groupadd -g 1001 app && useradd -u 1001 -g app app
USER app
```

***

## 🔹 Résumé des bonnes pratiques `USER`

✔️ Toujours utiliser un **utilisateur non-root** si possible.\
✔️ Définir **UID/GID explicites** pour reproductibilité.\
✔️ Utiliser `--no-log-init` pour éviter le bug Go/tar.\
✔️ Pas de `sudo`, préférer `gosu` ou `su-exec`.\
✔️ Ne pas multiplier les changements de `USER`.

## 🔹 Pourquoi utiliser `WORKDIR` ?

* Définit le **répertoire de travail par défaut** pour toutes les instructions suivantes (`RUN`, `CMD`, `ENTRYPOINT`, `COPY`, `ADD`).
* Évite les enchaînements de `cd … && …` qui sont **illisibles et fragiles**.
* Améliore la lisibilité et la maintenance du Dockerfile.
* Rend le conteneur **plus prévisible**.

***

## 🔹 Exemple simple

❌ Mauvaise pratique sans `WORKDIR` :

```dockerfile
FROM python:3.12

RUN cd /app && pip install -r requirements.txt
COPY . /app
CMD ["python", "/app/main.py"]
```

👉 Problèmes :

* `cd` n’est valide que dans cette **instruction RUN** (pas persistant).
* Peu lisible.
* Risque d’erreurs si on oublie le `cd`.

***

✅ Bonne pratique avec `WORKDIR` :

```dockerfile
FROM python:3.12

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

CMD ["python", "main.py"]
```

👉 Avantages :

* Plus clair, chaque étape sait qu’elle s’exécute **dans `/app`**.
* Pas besoin de `cd`.
* Plus facile à maintenir.

***

## 🔹 Points clés à retenir

1.  Toujours utiliser des **chemins absolus** :

    ```dockerfile
    WORKDIR /usr/src/app
    ```

    et pas `WORKDIR src/app` (risque de confusion).
2.  On peut définir **plusieurs `WORKDIR`** dans un Dockerfile.\
    Exemple :

    ```dockerfile
    WORKDIR /app
    RUN echo "fichier1" > f1.txt

    WORKDIR /data
    RUN echo "fichier2" > f2.txt
    ```

    → Chaque `WORKDIR` définit le **contexte pour les instructions suivantes**.
3. Si le répertoire n’existe pas, Docker le **crée automatiquement**.\
   Pas besoin de `RUN mkdir -p`.

***

## 🔹 Exemple combiné (WORKDIR + USER)

Voici un exemple propre et sécurisé pour une appli Node.js :

```dockerfile
FROM node:20-alpine

# Créer un utilisateur non-root
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Définir le répertoire de travail
WORKDIR /usr/src/app

# Copier uniquement les fichiers nécessaires pour installer les deps
COPY package*.json ./

RUN npm install --production

# Copier le reste du code
COPY . .

# Définir l’utilisateur non-root
USER appuser

# Lancer l’app
CMD ["node", "server.js"]
```

👉 Ici :

* `WORKDIR /usr/src/app` simplifie toutes les instructions suivantes.
* Pas besoin de `cd`.
* L’app tourne sous `appuser`.

## 🔹 Qu’est-ce que `ONBUILD` ?

* C’est une **instruction différée**.
* Elle **ne s’exécute pas** quand tu construis l’image qui contient le `ONBUILD`.
* Elle **s’exécute automatiquement** quand une autre image fait un `FROM cette_image`.

👉 On peut voir ça comme un **hook (crochet)** que tu poses dans l’image parent, et qui sera déclenché dans l’image enfant.

***

## 🔹 Exemple basique

**Image parent :**

```dockerfile
# syntax=docker/dockerfile:1
FROM node:20-alpine

# Préparer une instruction différée
ONBUILD COPY . /usr/src/app
ONBUILD WORKDIR /usr/src/app
ONBUILD RUN npm install

CMD ["node", "server.js"]
```

Quand tu construis cette image, les `ONBUILD` **ne sont pas exécutés**.\
Tu peux la tagger par exemple en `my-node-base:onbuild`.

***

**Image enfant :**

```dockerfile
FROM my-node-base:onbuild
```

👉 Lors de la construction de cette image enfant :

* `COPY . /usr/src/app` est exécuté
* `WORKDIR /usr/src/app` est exécuté
* `RUN npm install` est exécuté

Donc tu obtiens automatiquement une image avec ton code Node.js installé, **sans répéter les étapes**.

***

## 🔹 Cas d’usage

* **Stacks de langage/framework** : Ruby, Node.js, Go, etc.\
  → Ex : `ruby:onbuild` est une variante qui déclenche automatiquement un `bundle install` dans l’image enfant.
* **Images génériques pour bootstrapping** : tu fournis une base et imposes aux utilisateurs certaines étapes lors de l’héritage.

***

## 🔹 Bonnes pratiques

✅ Utiliser un **tag séparé** (`myimage:onbuild`) pour ne pas surprendre les utilisateurs qui héritent sans le vouloir.\
✅ Documenter clairement les comportements déclenchés.\
⚠️ Attention avec `COPY` ou `ADD` :

* Si l’image enfant n’a pas les fichiers attendus, la construction **échoue brutalement**.\
  ⚠️ Pas idéal pour des images finales en prod, c’est plus adapté à des **images de dev/gabarits**.

***

## 🔹 Résumé visuel

📌 **Image parent** (prépare des instructions différées)

```
Dockerfile Parent  ----> construit ----> Image Parent (avec hooks ONBUILD)
```

📌 **Image enfant** (déclenche les hooks lors du build)

```
Dockerfile Enfant (FROM Parent) ----> ONBUILD s’exécute ----> Image Enfant prête
```
