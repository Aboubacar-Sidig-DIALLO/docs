# 📦 Exporter des binaires

### 💡 Le saviez-vous ?

Vous pouvez utiliser **Docker** pour construire votre application en **binaires autonomes** (_standalone binaries_) 🎉.

👉 En effet, il arrive que vous ne vouliez pas forcément empaqueter et distribuer votre application sous forme d’image Docker.\
Dans ce cas, vous pouvez :

* utiliser Docker pour compiler votre application,
* puis employer les **exporteurs** (_exporters_) afin de sauvegarder le résultat directement sur le disque.

***

### 📌 Format de sortie par défaut

Par défaut, la commande `docker build` produit une **image de conteneur** :

* Cette image est automatiquement chargée dans votre **image store local**.
* Vous pouvez ensuite :
  * exécuter un conteneur basé sur cette image,
  * ou bien pousser cette image vers un **registre** (Docker Hub, registre privé, etc.).

👉 En coulisse, cela utilise l’exporteur par défaut appelé **docker exporter**.

***

### 🛠️ Exporter les résultats sous forme de fichiers

Si vous voulez exporter vos résultats de build **comme fichiers** plutôt que comme image, utilisez l’option **`--output`** (ou son alias **`-o`**).

#### Exemple simple :

```bash
docker build --output ./bin .
```

👉 Ici, au lieu de produire une image Docker, les fichiers générés par le build seront **exportés sur disque**, dans le dossier `./bin`.

## 📤 Exporter des binaires depuis un build

### 📌 Principe

Quand vous spécifiez un chemin de fichier avec l’option **`docker build --output`**, Docker exporte le contenu du conteneur de build, à la fin du processus, vers l’emplacement indiqué sur le système de fichiers de votre hôte.

👉 Ce mécanisme utilise l’**exporteur local** (_local exporter_).

***

### 🎯 Intérêt

Cela permet de tirer parti :

* de la puissance d’isolation de Docker,
* et de ses fonctionnalités de build,

➡️ pour générer des **binaires autonomes** (_standalone binaries_).

👉 C’est particulièrement pratique avec des langages comme **Go**, **Rust**, ou d’autres capables de compiler en un seul binaire.

***

### 🧪 Exemple pratique : programme Rust minimal

#### 1️⃣ Créer un répertoire de travail

```bash
mkdir hello-world-bin
cd hello-world-bin
```

#### 2️⃣ Créer un Dockerfile

```dockerfile
# syntax=docker/dockerfile:1
FROM rust:alpine AS build
WORKDIR /src
COPY <<EOT hello.rs
fn main() {
    println!("Hello World!");
}
EOT
RUN rustc -o /bin/hello hello.rs

FROM scratch
COPY --from=build /bin/hello /
ENTRYPOINT ["/hello"]
```

***

### 💡 Astuce : `COPY <<EOT`

La syntaxe `COPY <<EOT` est un **here-document**.\
👉 Cela permet d’écrire directement des chaînes multi-lignes dans un `Dockerfile`.\
Ici, on l’utilise pour créer un petit programme Rust inline.

***

### 🏗️ Analyse du Dockerfile

* Étape **1 (build)** :
  * On compile le programme Rust grâce à l’image `rust:alpine`.
  * Le binaire est généré dans `/bin/hello`.
* Étape **2 (finale)** :
  * On copie le binaire dans une image **scratch** (_vide et minimale_).
  * L’image finale ne contient **que le binaire**.

👉 L’utilisation de `scratch` est un pattern courant pour générer des artefacts minimaux pour des programmes ne nécessitant pas un OS complet.

***

### 🚀 Construire et exporter le binaire

```bash
docker build --output=. .
```

* Cette commande construit le Dockerfile,
* puis exporte le binaire dans le répertoire courant (`.`).

👉 Résultat : un fichier **`hello`** est créé dans votre répertoire de travail.

## 🌍 Exporter des builds multi-plateformes

### 📌 Principe

Avec l’**exporteur local** (_local exporter_), vous pouvez exporter des **binaires multi-plateformes** compilés via Docker.

👉 Cela permet de générer **plusieurs binaires en un seul build**, utilisables sur différentes machines et architectures (à condition que votre compilateur supporte ces plateformes cibles).

***

### 🧪 Exemple Rust multi-plateforme

On reprend le même **Dockerfile** que dans la section précédente (_Export binaries from a build_) :

```dockerfile
# syntax=docker/dockerfile:1
FROM rust:alpine AS build
WORKDIR /src
COPY <<EOT hello.rs
fn main() {
    println!("Hello World!");
}
EOT
RUN rustc -o /bin/hello hello.rs

FROM scratch
COPY --from=build /bin/hello /
ENTRYPOINT ["/hello"]
```

***

### 🚀 Construire pour plusieurs plateformes

Avec le flag **`--platform`**, vous pouvez compiler pour plusieurs architectures d’un seul coup.\
Combiné avec **`--output`**, cela exporte les binaires dans des dossiers séparés selon la plateforme.

Exemple :

```bash
docker build --platform=linux/amd64,linux/arm64 --output=out .
```

👉 Résultat attendu dans le dossier `out/` :

```bash
tree out/
```

```
out/
├── linux_amd64
│   └── hello
└── linux_arm64
    └── hello

3 directories, 2 files
```

✅ Vous obtenez deux binaires compilés :

* `out/linux_amd64/hello`
* `out/linux_arm64/hello`

Prêts à être exécutés sur leurs architectures respectives 🚀

***

### ℹ️ Informations supplémentaires

En plus de l’**exporteur local**, Docker Buildx supporte d’autres **exporteurs** (par ex. `docker`, `tar`, `oci` …).\
👉 Pour en savoir plus sur ces différents exporteurs et leurs usages, consultez la documentation officielle sur les **exporters**.
