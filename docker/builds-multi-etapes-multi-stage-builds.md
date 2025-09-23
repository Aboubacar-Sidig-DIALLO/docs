# 📌 Builds multi-étapes (Multi-stage builds)

👉 Les **builds multi-étapes** sont une technique qui permet de :

* **optimiser la taille des images finales** (pas besoin d’embarquer les dépendances de compilation)
* **garder un Dockerfile lisible et maintenable**
* **éviter les scripts externes de build** (tout est dans un seul Dockerfile).

***

### 🔹 Principe

* Tu peux avoir **plusieurs instructions `FROM`** dans un même Dockerfile.
* Chaque `FROM` démarre une **nouvelle étape** (stage).
* Tu peux copier uniquement ce qui t’intéresse d’une étape vers une autre avec `COPY --from`.

Résultat : l’image finale ne contient que le strict nécessaire (ex. ton binaire ou ton application), sans outils de build ni fichiers temporaires.

***

### 🔹 Exemple : Go "Hello World"

Voici un Dockerfile en **multi-étapes** :

```dockerfile
# syntax=docker/dockerfile:1

# Étape 1 : Compilation (builder)
FROM golang:1.24 AS builder
WORKDIR /src

# Copier et compiler le code Go
COPY <<EOF ./main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF

RUN go build -o /bin/hello ./main.go

# Étape 2 : Image finale (production)
FROM scratch
COPY --from=builder /bin/hello /bin/hello
CMD ["/bin/hello"]
```

***

### 🔹 Explications

1. **Étape "builder"**
   * Basée sur `golang:1.24` (qui contient tous les outils nécessaires pour compiler du Go).
   * On compile `main.go` en un binaire `/bin/hello`.
2. **Étape finale**
   * Basée sur `scratch` (une image vide et minimale).
   * On copie uniquement le binaire depuis l’étape `builder`.
   * Résultat : une image **ultra légère** avec juste le binaire compilé.

***

### 🔹 Construction et exécution

```bash
# Construire l’image
docker build -t hello .

# Exécuter le conteneur
docker run --rm hello
```

👉 Sortie attendue :

```
hello, world
```

***

### ✅ Avantages des builds multi-étapes

* 🔹 **Images finales plus petites** (pas de dépendances inutiles).
* 🔹 **Sécurité accrue** (moins de surface d’attaque).
* 🔹 **Dockerfile plus simple** (tout est dans un seul fichier).
* 🔹 **Pas besoin de nettoyer manuellement** (les étapes intermédiaires sont ignorées).

## 📌 Nommer vos étapes de build (Name your build stages)

👉 Par défaut, Docker attribue un **numéro** à chaque étape (stage) :

* `0` pour le premier `FROM`
* `1` pour le deuxième, etc.

Mais tu peux leur donner un **nom explicite** avec `AS <nom>` dans l’instruction `FROM`.

Cela rend le Dockerfile :

* ✅ plus **lisible**
* ✅ plus **robuste** (pas besoin de te soucier si tu ajoutes ou réordonnes les étapes plus tard)
* ✅ plus **maintenable** (tu comprends tout de suite quel stage sert à quoi).

***

### 🔹 Exemple : Hello World en Go avec un stage nommé

```dockerfile
# syntax=docker/dockerfile:1

# Étape 1 : Compilation (nommée "build")
FROM golang:1.24 AS build
WORKDIR /src

# Copier et compiler le code Go
COPY <<EOF /src/main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF

RUN go build -o /bin/hello ./main.go

# Étape 2 : Image finale
FROM scratch
# Ici, on utilise "build" au lieu de --from=0
COPY --from=build /bin/hello /bin/hello
CMD ["/bin/hello"]
```

***

### 🔹 Points importants

1. **`AS build`**
   * On donne un nom explicite au stage de compilation.
   * On peut en nommer plusieurs (`AS test`, `AS deps`, `AS prod`, etc.).
2. **COPY --from=build**
   * On copie depuis l’étape nommée, au lieu de `--from=0`.
   * Même si on ajoute une étape avant `FROM golang:1.24`, ça ne cassera pas le build.

***

### ✅ Bonnes pratiques

* Donne toujours des **noms explicites** à tes stages (ex. `build`, `test`, `prod`).
* Cela rend ton **Dockerfile auto-documenté**.
* Ça évite les surprises si tu modifies l’ordre de tes `FROM`.

## 📌 Stopper un build à une étape donnée (`--target`)

Quand tu as un **multi-stage build**, tu n’es pas obligé d’aller jusqu’au bout du Dockerfile.\
Tu peux demander à Docker de ne construire que jusqu’à une étape (stage) précise grâce à l’option :

```bash
docker build --target <nom_du_stage> -t <nom_image> .
```

***

### 🔹 Exemple avec notre Dockerfile Go

```dockerfile
# syntax=docker/dockerfile:1

# Étape 1 : Compilation
FROM golang:1.24 AS build
WORKDIR /src
COPY main.go .
RUN go build -o /bin/hello ./main.go

# Étape 2 : Image finale
FROM scratch
COPY --from=build /bin/hello /bin/hello
CMD ["/bin/hello"]
```

#### Si on veut s’arrêter à `build` :

```bash
docker build --target build -t hello-build .
```

👉 Résultat : l’image `hello-build` contient l’environnement Go + le binaire compilé, mais **pas la partie finale `FROM scratch`**.

***

### 🔹 Cas d’usage typiques

1. **Debug d’un stage intermédiaire**
   * Tu veux vérifier que tes dépendances ou ta compilation marchent avant de passer à l’étape suivante.
2. **Stage de développement / debug**
   * Tu peux avoir une étape `dev` avec plus d’outils (debuggers, logs, etc.) et une étape `prod` minimale.
   *   Exemple :

       ```bash
       docker build --target dev -t myapp:dev .
       docker build --target prod -t myapp:prod .
       ```
3. **Tests automatisés**
   * Tu peux avoir une étape `test` qui lance des tests unitaires avec des données factices.
   * Puis une étape `prod` qui prend uniquement le code final validé.

***

### ✅ Avantage

* Tu réutilises le **même Dockerfile** pour plusieurs usages (dev, test, prod).
* Plus besoin de maintenir plusieurs fichiers séparés.

## 📌 Utiliser une image externe comme étape (`COPY --from`)

Avec les **multi-stage builds**, tu peux copier des fichiers non seulement depuis des étapes précédentes de ton Dockerfile, mais aussi directement depuis une **autre image** (locale ou distante, sur un registre comme Docker Hub).

👉 Syntaxe :

```dockerfile
COPY --from=<image> <chemin_source> <chemin_destination>
```

***

### 🔹 Exemple simple avec Nginx

```dockerfile
# syntax=docker/dockerfile:1

FROM alpine:latest

# Copier le fichier de config nginx directement depuis l’image officielle
COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf

CMD ["cat", "/nginx.conf"]
```

👉 Ici :

* Pas besoin d’installer Nginx dans ton conteneur.
* Tu récupères **juste le fichier `nginx.conf`** depuis l’image `nginx:latest`.

***

### 🔹 Cas pratiques

1. **Récupérer des binaires compilés**
   *   Exemple avec `node:alpine` pour récupérer `node` sans installer tout NodeJS :

       ```dockerfile
       FROM alpine:latest
       COPY --from=node:20-alpine /usr/local/bin/node /usr/local/bin/
       ```
2. **Extraire des configurations par défaut**
   *   Exemple avec `postgres` :

       ```dockerfile
       FROM alpine
       COPY --from=postgres:16 /usr/share/postgresql/postgresql.conf.sample /postgres.conf
       ```
3. **Réutiliser des assets pré-packagés**
   *   Exemple avec `nginx` pour copier les pages HTML par défaut :

       ```dockerfile
       COPY --from=nginx:latest /usr/share/nginx/html /app/html
       ```

***

### ✅ Avantages

* Pas besoin de reconstruire ou réinstaller quelque chose que tu veux **juste récupérer**.
* Permet de **réduire la taille des images** finales.
* Tu profites des artefacts fournis dans les images officielles (binaire, config, assets).

## 📌 Réutiliser une étape précédente (`FROM <stage>`)

Dans un **multi-stage build**, chaque `FROM` démarre normalement un nouvel environnement.\
Mais tu peux aussi redémarrer **à partir d’une étape précédente**, au lieu de repartir d’une image de base.

👉 Syntaxe :

```dockerfile
FROM <nom_d’étape_précédente> AS <nouvelle_étape>
```

***

### 🔹 Exemple fourni

```dockerfile
# syntax=docker/dockerfile:1

FROM alpine:latest AS builder
RUN apk --no-cache add build-base

FROM builder AS build1
COPY source1.cpp source.cpp
RUN g++ -o /binary source.cpp

FROM builder AS build2
COPY source2.cpp source.cpp
RUN g++ -o /binary source.cpp
```

#### 📝 Explication :

1. **Étape `builder`**
   * Démarre sur `alpine:latest`.
   * Installe `build-base` (compilateur C++ et outils nécessaires).
2. **Étape `build1`**
   * Redémarre **à partir de `builder`** (qui a déjà les outils installés).
   * Copie `source1.cpp`, compile en `/binary`.
3. **Étape `build2`**
   * Redémarre aussi **à partir de `builder`**.
   * Copie `source2.cpp`, compile un autre binaire.

👉 Résultat :

* Tu as deux builds différents (`/binary` compilé à partir de deux sources), mais tu n’as pas besoin de répéter l’installation des dépendances dans chaque étape.

***

### ✅ Avantages

* **Réduction des duplications** → tu factorises l’installation des dépendances.
* **Plus de flexibilité** → tu peux générer plusieurs variantes à partir d’un même environnement de base.
* **Lisibilité améliorée** → un seul bloc pour la configuration commune, puis plusieurs étapes spécifiques.

***

### 🔹 Cas pratique réel

Imaginons un projet avec plusieurs microservices en Go :

```dockerfile
FROM golang:1.22 AS builder
WORKDIR /src
RUN go install github.com/cespare/reflex@latest # outil utile pour dev

# Service 1
FROM builder AS service1
COPY service1/ .
RUN go build -o /bin/service1 .

# Service 2
FROM builder AS service2
COPY service2/ .
RUN go build -o /bin/service2 .
```

👉 Ici :

* On partage la même base (`builder`) avec Go et un outil installé.
* On compile séparément `service1` et `service2`.

## 🔹 Différences entre le builder classique et BuildKit

### 1️⃣ Comportement du **legacy builder**

* Le builder **historique** (sans BuildKit) **exécute toutes les étapes** du Dockerfile, **jusqu’à la cible (--target)**.
* Même si une étape **n’est pas nécessaire** pour atteindre la cible, elle sera quand même construite.

👉 Exemple :\
Dockerfile :

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu AS base
RUN echo "base"

FROM base AS stage1
RUN echo "stage1"

FROM base AS stage2
RUN echo "stage2"
```

Commande :

```bash
DOCKER_BUILDKIT=0 docker build --no-cache -f Dockerfile --target stage2 .
```

➡️ Résultat :

* `base` est construit ✅
* `stage1` est construit ❌ (inutile mais quand même exécuté)
* `stage2` est construit ✅

***

### 2️⃣ Comportement avec **BuildKit**

* BuildKit est **plus intelligent** ⚡
* Il **ne construit que les étapes nécessaires** pour atteindre la cible (--target).
* Si une étape n’est pas une dépendance directe ou indirecte, elle est **ignorée**.

Commande :

```bash
DOCKER_BUILDKIT=1 docker build --no-cache -f Dockerfile --target stage2 .
```

➡️ Résultat :

* `base` est construit ✅ (car `stage2` en dépend)
* `stage1` est **ignoré** 🚫 (inutile pour `stage2`)
* `stage2` est construit ✅

***

### 3️⃣ Avantages de BuildKit

* ⚡ **Performance** → évite les étapes inutiles, builds plus rapides.
* 🧹 **Efficacité** → ne copie que les ressources nécessaires, économise espace disque et temps réseau.
* 🔒 **Fonctionnalités avancées** → gestion des secrets (`--secret`), montages (`--mount`), parallélisation des étapes, cache plus performant, etc.

***

### ✅ Résumé comparatif

| Fonctionnalité                 | Legacy builder 🐢 | BuildKit 🚀                               |
| ------------------------------ | ----------------- | ----------------------------------------- |
| Construit toutes les étapes    | Oui               | Non (uniquement les dépendances utiles)   |
| Gestion avancée du cache       | Basique           | Optimisée (plus granulaire, incrémentale) |
| Support des secrets / SSH      | Non               | Oui                                       |
| Exécution parallèle des étapes | Non               | Oui                                       |
| Utilisation par défaut         | Ancien Docker     | Activé par défaut sur Docker récents      |

***

👉 En clair : **BuildKit = moderne, rapide, efficace, modulaire.**\
C’est pour ça qu’il est activé par défaut dans Docker Desktop et les versions récentes de Docker Engine.
