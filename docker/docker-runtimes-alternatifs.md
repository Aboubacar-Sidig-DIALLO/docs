# 🐳 Docker – Runtimes alternatifs

Par défaut, **Docker Engine** utilise :

* **containerd** → pour gérer le cycle de vie des conteneurs (création, démarrage, arrêt).
* **runc** → comme runtime bas niveau (exécute réellement les conteneurs via containerd).

Mais tu peux changer ce runtime par d’autres selon tes besoins (sécurité, performance, sandboxing, etc.).

***

### 🔹 Quels runtimes peut-on utiliser ?

Il existe deux grandes catégories :

#### 1. **Runtimes avec leur propre&#x20;**_**containerd shim**_

👉 Ces runtimes s’intègrent directement sans config particulière, car ils fournissent leur propre shim.\
Exemples :

* **Wasmtime** → exécution de **WebAssembly** (au lieu de binaires Linux).
* **gVisor** → sandbox sécurisé développé par Google, qui intercepte les appels systèmes.
* **Kata Containers** → isole chaque conteneur dans une **machine virtuelle légère** pour plus de sécurité.

⚡ Utilisation : tu peux les activer directement via containerd.

***

#### 2. **Runtimes remplaçant runc** (_drop-in replacement_)

👉 Ces runtimes utilisent le shim de runc mais remplacent le binaire exécuté.\
Exemple :

* **youki** → runtime écrit en Rust, plus rapide et plus sûr que runc.

⚠️ Pour ces runtimes, il faut **les enregistrer manuellement** dans la configuration du démon Docker (`daemon.json`).

***

### 🔹 Exemple d’utilisation avec `youki`

1. Installer **youki**.
2. Modifier `/etc/docker/daemon.json` :

```json
{
  "runtimes": {
    "youki": {
      "path": "/usr/bin/youki"
    }
  }
}
```

3. Redémarrer Docker :

```bash
sudo systemctl restart docker
```

4. Lancer un conteneur avec ce runtime :

```bash
docker run --rm --runtime=youki alpine uname -a
```

***

### 🔹 Quand utiliser un runtime alternatif ?

* **Performance / sécurité renforcée** → `youki` (Rust, rapide, sécurisé).
* **Isolation forte (VM + conteneurs)** → `Kata Containers`.
* **Sandbox utilisateurs non fiables** → `gVisor`.
* **Exécution WASM dans Docker** → `Wasmtime`.

## 🐳 Docker – Utiliser les **containerd shims**

Un **containerd shim** est une couche intermédiaire entre `containerd` et le runtime (ex. `runc`, `Kata`, `gVisor`, etc.).\
👉 L’avantage : tu peux utiliser un runtime alternatif **sans modifier la configuration du démon Docker (`daemon.json`)**.

***

### 🔹 Étapes pour utiliser un shim

1. Installer le **shim** du runtime que tu veux utiliser (par ex. `kata` ou `gVisor`).
   * Le binaire doit être présent dans le **PATH** du système où tourne Docker.
2. Lors de l’exécution d’un conteneur, indiquer explicitement le runtime via `--runtime` :

```bash
docker run --runtime io.containerd.kata.v2 hello-world
```

***

### 🔹 Explications

* `--runtime` → permet de choisir quel runtime utiliser **au moment du lancement du conteneur**.
* `io.containerd.kata.v2` → est le **nom du shim Kata Containers**.
  * Kata isole les conteneurs dans des VM légères pour une sécurité accrue.
* Tu peux remplacer `io.containerd.kata.v2` par d’autres shims installés (ex. `runsc` pour **gVisor**, `wasmtime` pour **WebAssembly**, etc.).

***

### 🔹 Exemple avec différents shims

```bash
# Utiliser gVisor (runsc)
docker run --runtime=runsc alpine uname -a

# Utiliser Kata Containers
docker run --runtime=io.containerd.kata.v2 alpine uname -a

# Utiliser Wasmtime pour exécuter du WebAssembly
docker run --runtime=io.containerd.wasmtime.v1 hello-wasm
```

## 🐳 Docker – Utiliser un **containerd shim** sans l’installer dans le `PATH`

Par défaut, Docker (via `containerd`) recherche les shims installés dans le **PATH système**.\
👉 Mais si ton shim n’est pas installé globalement, tu peux quand même l’utiliser en l’**enregistrant dans la configuration du démon Docker**.

***

### 🔹 Étapes

#### 1. Modifier la configuration du démon

Édite ton fichier `/etc/docker/daemon.json` (ou `~/.config/docker/daemon.json` en mode rootless) et ajoute une section `runtimes` :

```json
{
  "runtimes": {
    "foo": {
      "runtimeType": "/path/to/containerd-shim-foobar-v1"
    }
  }
}
```

* `foo` → est le nom que tu donnes à ce runtime (libre, mais doit être unique).
* `runtimeType` → chemin complet vers le binaire du shim (non présent dans `$PATH`).

***

#### 2. Redémarrer Docker

Pour que la modification prenne effet :

```bash
sudo systemctl restart docker
```

***

#### 3. Lancer un conteneur avec ce runtime

Ensuite, tu peux utiliser ton shim enregistré en appelant son nom (`foo`) :

```bash
docker run --runtime foo hello-world
```

***

✅ Avec ça, tu peux utiliser **des shims locaux** (pas installés dans `/usr/bin` ou `/usr/local/bin`) simplement en les enregistrant.

## 🐳 Configurer des **shims containerd** avec Docker

Docker utilise **containerd** pour gérer le cycle de vie des conteneurs.\
Si tu veux utiliser un **runtime alternatif** (comme gVisor, Kata Containers, Wasmtime, youki…), tu dois le déclarer dans la configuration du démon Docker.

***

### 🔹 Étape 1 : Modifier le fichier `daemon.json`

Édite ton fichier `/etc/docker/daemon.json` (Linux classique) ou `~/.config/docker/daemon.json` (rootless).\
Ajoute une section `runtimes` pour ton shim. Exemple avec **gVisor (`runsc`)** :

```json
{
  "runtimes": {
    "gvisor": {
      "runtimeType": "io.containerd.runsc.v1",
      "options": {
        "TypeUrl": "io.containerd.runsc.v1.options",
        "ConfigPath": "/etc/containerd/runsc.toml"
      }
    }
  }
}
```

* `runtimeType` → le nom complet du runtime reconnu par `containerd`.
* `options` → permet de passer des paramètres supplémentaires :
  * `TypeUrl` : type d’options attendu par le shim.
  * `ConfigPath` : chemin vers un fichier de configuration externe.

***

### 🔹 Étape 2 : Recharger la configuration

Une fois la modification faite :

```bash
sudo systemctl reload docker
```

***

### 🔹 Étape 3 : Utiliser ton runtime personnalisé

Ensuite, lance un conteneur en précisant le runtime configuré :

```bash
docker run --runtime gvisor hello-world
```

***

### 🔹 Autres exemples de runtimes

#### ✅ **youki** (remplacement de `runc`)

```json
{
  "runtimes": {
    "youki": {
      "path": "/usr/local/bin/youki"
    }
  }
}
```

Usage :

```bash
docker run --runtime youki alpine uname -a
```

***

#### ✅ **Wasmtime** (WebAssembly containers)

```json
{
  "runtimes": {
    "wasmtime": {
      "runtimeType": "io.containerd.wasmtime.v1"
    }
  }
}
```

Usage :

```bash
docker run --runtime wasmtime hello-wasm
```

***

📌 Résumé :

* `runtimes` → permet de **déclarer plusieurs runtimes** dans Docker.
* `runtimeType` → obligatoire pour les shims containerd (`io.containerd.…`).
* `options` → pour ajouter une config avancée (ex. gVisor).
* Utilisation via `docker run --runtime <nom>`.

## 🚀 Utiliser **youki** comme runtime alternatif dans Docker

👉 **youki** est un runtime de conteneurs écrit en **Rust**.\
Il se veut plus **rapide** et moins **gourmand en mémoire** que `runc`, ce qui en fait un bon choix pour des environnements à ressources limitées (IoT, serveurs légers, edge computing…).

⚡ Comme youki est un **remplacement direct de `runc`**, il s’appuie sur le **shim runc** existant pour fonctionner avec Docker.

***

### 🔹 Étape 1 : Installer **youki**

Tu dois d’abord installer youki et ses dépendances.\
👉 Reporte-toi au guide officiel d’installation : [youki setup guide](https://github.com/containers/youki).\
Généralement, on place le binaire dans `/usr/local/bin/youki`.

***

### 🔹 Étape 2 : Enregistrer youki dans Docker

Édite le fichier de configuration du démon Docker `/etc/docker/daemon.json` :

```bash
sudo tee /etc/docker/daemon.json > /dev/null <<EOF
{
  "runtimes": {
    "youki": {
      "path": "/usr/local/bin/youki"
    }
  }
}
EOF
```

Ici :

* `path` → indique le chemin du binaire `youki`.
* Tu peux aussi ajouter des `runtimeArgs` si besoin, par exemple :

```json
{
  "runtimes": {
    "youki": {
      "path": "/usr/local/bin/youki",
      "runtimeArgs": ["--log", "/var/log/youki.log"]
    }
  }
}
```

***

### 🔹 Étape 3 : Recharger la configuration du démon

Recharge la config sans redémarrer complètement le service :

```bash
sudo systemctl reload docker
```

***

### 🔹 Étape 4 : Lancer un conteneur avec youki

Tu peux maintenant lancer un conteneur en précisant le runtime `youki` :

```bash
docker run --rm --runtime youki hello-world
```

***

✅ **Résumé** :

* youki est un **drop-in replacement** de `runc`.
* Installation → placer le binaire.
* Configuration → ajouter `youki` dans `daemon.json`.
* Utilisation → `docker run --runtime youki …`.

## 🧩 Utiliser **Wasmtime** comme runtime Docker (expérimental)

👉 **Wasmtime** est un runtime **WebAssembly (Wasm)** développé par la **Bytecode Alliance**.\
Avec Docker, il permet d’exécuter des **conteneurs Wasm** en bénéficiant :

* de l’**isolation des conteneurs Docker**,
*
  * du **sandboxing de l’environnement Wasm** → une double couche de sécurité 🔒.

***

### 🔹 Étape 1 : Activer l’image store containerd

Dans la configuration du démon Docker (`/etc/docker/daemon.json`), ajoute :

```json
{
  "features": {
    "containerd-snapshotter": true
  }
}
```

Puis redémarre le démon :

```bash
sudo systemctl restart docker
```

***

### 🔹 Étape 2 : Installer le shim Wasmtime

Docker utilise un **shim containerd** pour exécuter Wasmtime.\
On doit donc compiler et installer le binaire `containerd-shim-wasmtime-v1`.

#### Compiler avec Docker :

```bash
docker build --output . - <<EOF
FROM rust:latest as build
RUN cargo install \
    --git https://github.com/containerd/runwasi.git \
    --bin containerd-shim-wasmtime-v1 \
    --root /out \
    containerd-shim-wasmtime
FROM scratch
COPY --from=build /out/bin /
EOF
```

Cela génère le binaire **`./containerd-shim-wasmtime-v1`**.

#### Placer le binaire sur le PATH :

```bash
sudo mv ./containerd-shim-wasmtime-v1 /usr/local/bin
```

***

### 🔹 Étape 3 : Lancer un conteneur Wasm avec Wasmtime

Maintenant, tu peux exécuter un conteneur basé sur **WebAssembly** :

```bash
docker run --rm \
  --runtime io.containerd.wasmtime.v1 \
  --platform wasi/wasm32 \
  michaelirwin244/wasm-example
```

***

### 🔹 Points importants

* **Disponibilité** : ce support est encore **expérimental** 🧪.
* **Isolation** : sécurité renforcée (Docker + Wasm sandbox).
*   **Cas d’usage** :

    * microservices ultra-légers,
    * exécutables portables **Wasm**,
    *   environnements contraints (IoT, edge computing).

        ## 🧩 Utiliser **Wasmtime** comme runtime Docker (expérimental)

        👉 **Wasmtime** est un runtime **WebAssembly (Wasm)** développé par la **Bytecode Alliance**.\
        Avec Docker, il permet d’exécuter des **conteneurs Wasm** en bénéficiant :

        * de l’**isolation des conteneurs Docker**,
        *
          * du **sandboxing de l’environnement Wasm** → une double couche de sécurité 🔒.

        ### 🔹 Étape 1 : Activer l’image store containerd

        Dans la configuration du démon Docker (`/etc/docker/daemon.json`), ajoute :

        ```json
        {
          "features": {
            "containerd-snapshotter": true
          }
        }
        ```

        Puis redémarre le démon :

        ```bash
        sudo systemctl restart docker
        ```

        ### 🔹 Étape 2 : Installer le shim Wasmtime

        Docker utilise un **shim containerd** pour exécuter Wasmtime.\
        On doit donc compiler et installer le binaire `containerd-shim-wasmtime-v1`.

        #### Compiler avec Docker :

        ```bash
        docker build --output . - <<EOF
        FROM rust:latest as build
        RUN cargo install \
            --git https://github.com/containerd/runwasi.git \
            --bin containerd-shim-wasmtime-v1 \
            --root /out \
            containerd-shim-wasmtime
        FROM scratch
        COPY --from=build /out/bin /
        EOF
        ```

        Cela génère le binaire **`./containerd-shim-wasmtime-v1`**.



    #### Placer le binaire sur le PATH :

    ```bash
    sudo mv ./containerd-shim-wasmtime-v1 /usr/local/bin
    ```

    ### 🔹 Étape 3 : Lancer un conteneur Wasm avec Wasmtime

    Maintenant, tu peux exécuter un conteneur basé sur **WebAssembly** :

    ```bash
    docker run --rm \
      --runtime io.containerd.wasmtime.v1 \
      --platform wasi/wasm32 \
      michaelirwin244/wasm-example
    ```

    ### 🔹 Points importants

    * **Disponibilité** : ce support est encore **expérimental** 🧪.
    * **Isolation** : sécurité renforcée (Docker + Wasm sandbox).
    * **Cas d’usage** :
      * microservices ultra-légers,
      * exécutables portables **Wasm**,
      * environnements contraints (IoT, edge computing).





