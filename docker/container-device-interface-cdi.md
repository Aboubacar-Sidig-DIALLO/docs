# ⚙️ Container Device Interface (CDI)

### 🌍 Qu’est-ce que CDI ?

La **Container Device Interface (CDI)** est une **spécification** conçue pour **standardiser** la manière dont les périphériques matériels (tels que :

* 🎮 GPU,
* ⚡ FPGA,
* 🔬 accélérateurs matériels divers)

sont exposés et utilisés par des conteneurs.

#### 🎯 Objectifs de CDI :

* Fournir un mécanisme **cohérent et sécurisé** pour utiliser du matériel dans des environnements conteneurisés.
* Éliminer la complexité liée aux **configurations spécifiques aux périphériques**.
* Rendre l’intégration de matériel dans Docker plus **portable** et **prévisible**.

***

### ➕ Que permet CDI en plus ?

Outre l’accès du conteneur au **nœud de périphérique** (_device node_), CDI permet aussi de spécifier :

* 🔧 des **variables d’environnement** associées au périphérique,
* 📂 des **montages de l’hôte** (par ex. des bibliothèques partagées `*.so`),
* ⚙️ des **hooks exécutables** (scripts ou binaires déclenchés automatiquement).

👉 En résumé : CDI ne fait pas que « brancher » le périphérique, il fournit aussi son contexte d’exécution complet.

***

## 🚀 Mise en route

### 📌 Prérequis

Pour commencer à utiliser CDI, il faut :

* **Docker v27+** installé avec CDI configuré,
* **Buildx v0.22+** ou plus récent.

***

### 📁 Création des spécifications de périphériques

Les spécifications CDI doivent être définies via des fichiers **JSON** ou **YAML**, et placées dans l’un des emplacements suivants :

* `/etc/cdi`
* `/var/run/cdi`
* `/etc/buildkit/cdi`

***

### 📑 Remarques importantes

#### 🔧 Modifier l’emplacement des specs

* Si vous utilisez **BuildKit directement**, vous pouvez changer l’emplacement avec l’option **`specDirs`** dans la section **`cdi`** du fichier `buildkitd.toml`.

#### 🐳 Si vous utilisez Docker Daemon avec le driver `docker`

* Consultez la doc **Configure CDI devices** pour configurer correctement les périphériques CDI.

#### 💻 Cas particulier WSL (Windows Subsystem for Linux)

* Si vous créez un **builder de conteneurs sur WSL** :
  * installez **Docker Desktop**,
  * activez la **paravirtualisation GPU de WSL 2**,
  * utilisez **Buildx v0.27+** (nécessaire pour monter les bibliothèques WSL dans le conteneur).

## 🛠️ Construire avec une spécification CDI simple

### 🎯 Exemple de base

On va démarrer avec une spécification CDI très simple, qui injecte une **variable d’environnement** dans l’environnement de build.\
On écrit cette spécification dans le fichier **`/etc/cdi/foo.yaml`** :

```yaml
/etc/cdi/foo.yaml

cdiVersion: "0.6.0"
kind: "vendor1.com/device"
devices:
- name: foo
  containerEdits:
    env:
    - FOO=injected
```

***

### 🔍 Vérifier la détection du périphérique

Ensuite, inspectons le builder par défaut pour vérifier que **`vendor1.com/device`** est bien détecté :

```bash
docker buildx inspect
```

Extrait pertinent de la sortie :

```
Devices:
 Name:                  vendor1.com/device=foo
 Automatically allowed: false
```

👉 Le périphérique est bien détecté (`foo`), mais **il n’est pas autorisé automatiquement**.

***

### 🐳 Créer un Dockerfile qui utilise ce périphérique

```dockerfile
# syntax=docker/dockerfile:1-labs
FROM busybox
RUN --device=vendor1.com/device \
  env | grep ^FOO=
```

* Ici, on utilise la commande **`RUN --device`** pour demander un périphérique CDI.
* Le périphérique demandé est `vendor1.com/device`.
* Dans notre fichier `/etc/cdi/foo.yaml`, le premier périphérique est `foo`, donc c’est lui qui sera utilisé.

⚠️ **Note importante** :\
La commande `RUN --device` n’est disponible que dans le **canal labs** depuis `Dockerfile frontend v1.14.0-labs`.\
Elle n’est **pas encore disponible dans la syntaxe stable**.

***

### 🚧 Premier essai de build

```bash
docker buildx build .
```

Résultat :

```
ERROR: failed to build: failed to solve: failed to load LLB: device vendor1.com/device=foo is requested by the build but not allowed
```

👉 L’erreur se produit car, comme vu plus haut, le périphérique `vendor1.com/device=foo` n’est pas automatiquement autorisé.

***

### ✅ Autoriser explicitement le périphérique

Deux solutions possibles :

#### 1. Avec le flag `--allow`

Autoriser le périphérique manuellement lors du build :

```bash
docker buildx build --allow device .
```

#### 2. Avec une annotation dans la spec CDI

Modifier le fichier `/etc/cdi/foo.yaml` pour autoriser automatiquement le périphérique pour tous les builds :

```yaml
cdiVersion: "0.6.0"
kind: "vendor1.com/device"
devices:
- name: foo
  containerEdits:
    env:
    - FOO=injected
annotations:
  org.mobyproject.buildkit.device.autoallow: true
```

***

### 🚀 Relancer le build avec autorisation

```bash
docker buildx build --progress=plain --allow device .
```

Sortie :

```
7 [2/2] RUN --device=vendor1.com/device   env | grep ^FOO=
7 0.155 FOO=injected
7 DONE 0.2s
```

👉 Cette fois, le build réussit 🎉\
👉 La variable d’environnement **`FOO=injected`** a bien été injectée dans l’environnement de build, comme spécifié dans la spec CDI.

## ⚡ Mettre en place un builder de conteneur avec support GPU (NVIDIA)

### 🎯 Contexte

Depuis **Buildx v0.22**, quand vous créez un **builder de type conteneur**, une requête GPU est **ajoutée automatiquement** si le système hôte dispose des **drivers GPU installés dans le noyau**.

👉 Cela fonctionne un peu comme si vous lanciez un conteneur avec :

```bash
docker run --gpus=all ...
```

***

### 📝 Note importante

L’image officielle de **BuildKit** est basée sur **Alpine**, qui **ne supporte pas les drivers NVIDIA**.

👉 Pour cette raison, une image spéciale a été créée :

* basée sur **Ubuntu**,
* intégrant les bibliothèques clients NVIDIA,
* générant automatiquement une **spécification CDI GPU** si un périphérique est demandé lors d’un build.

📦 Cette image est hébergée temporairement sur Docker Hub :

```
crazymax/buildkit:v0.23.2-ubuntu-nvidia
```

***

### 🚀 Étape 1 : Créer un builder GPU

Créons un builder nommé **gpubuilder** avec l’image spéciale BuildKit :

```bash
docker buildx create \
  --name gpubuilder \
  --driver-opt "image=crazymax/buildkit:v0.23.2-ubuntu-nvidia" \
  --bootstrap
```

Sortie (abrégée) :

```
1 pulling image crazymax/buildkit:v0.23.2-ubuntu-nvidia
1 creating container buildx_buildkit_gpubuilder0
gpubuilder
```

👉 Le builder est maintenant créé et initialisé.

***

### 🔍 Étape 2 : Inspecter le builder

```bash
docker buildx inspect gpubuilder
```

Sortie pertinente :

```
Devices:
 Name:      nvidia.com/gpu
 On-Demand: true
```

👉 Cela signifie que le **vendor `nvidia.com/gpu`** est bien détecté comme périphérique.\
👉 Donc les **drivers NVIDIA** ont été trouvés avec succès sur l’hôte. 🎉

***

### 🖥️ Étape 3 (optionnelle) : Vérifier avec `nvidia-smi`

Vous pouvez vérifier que le GPU est bien disponible dans le conteneur builder :

```bash
docker exec -it buildx_buildkit_gpubuilder0 nvidia-smi -L
```

Exemple de sortie :

```
GPU 0: Tesla T4 (UUID: GPU-6cf00fa7-59ac-16f2-3e83-d24ccdc56f84)
```

👉 Confirmation : la carte GPU est bien visible et utilisable dans le builder.

***

## ✅ Résumé

* Création d’un builder **GPU-aware** grâce à l’image Ubuntu spéciale.
* Détection automatique du périphérique **`nvidia.com/gpu`**.
* Vérification possible avec `nvidia-smi`.

## ⚡ Construire avec support GPU

### 🎯 Objectif

Nous allons créer un build simple qui exploite un **GPU NVIDIA** via CDI (Container Device Interface).

***

### 📝 Dockerfile d’exemple

```dockerfile
# syntax=docker/dockerfile:1-labs
FROM ubuntu
RUN --device=nvidia.com/gpu nvidia-smi -L
```

👉 Ici :

* La directive **`RUN --device=nvidia.com/gpu`** demande à BuildKit d’utiliser le périphérique GPU détecté (`nvidia.com/gpu`).
* La commande **`nvidia-smi -L`** liste les GPU disponibles avec leur UUID.

***

### 🚀 Lancer le build avec le builder GPU

On utilise le builder **gpubuilder** créé précédemment :

```bash
docker buildx --builder gpubuilder build --progress=plain .
```

Sortie (abrégée) :

```
7 preparing device nvidia.com/gpu
...
7 17.80 time="2025-04-15T08:58:16Z" level=info msg="Generated CDI spec with version 0.8.0"
...
8 [2/2] RUN --device=nvidia.com/gpu nvidia-smi -L
8 0.527 GPU 0: Tesla T4 (UUID: GPU-6cf00fa7-59ac-16f2-3e83-d24ccdc56f84)
```

#### ✅ Analyse :

* L’étape **7** prépare le périphérique GPU → installation des bibliothèques clientes NVIDIA + génération automatique de la spécification CDI.
* L’étape **8** exécute `nvidia-smi -L` dans le conteneur → le GPU est bien reconnu (ex. **Tesla T4**).

***

### 🔍 Vérifier la spécification CDI générée

On peut consulter le fichier généré dans le builder :

```bash
docker exec -it buildx_buildkit_gpubuilder0 cat /etc/cdi/nvidia.yaml
```

Exemple de contenu (simplifié, instance EC2 `g4dn.xlarge`) :

```yaml
cdiVersion: 0.6.0
containerEdits:
  deviceNodes:
  - path: /dev/nvidia-modeset
  - path: /dev/nvidia-uvm
  - path: /dev/nvidiactl
  env:
  - NVIDIA_VISIBLE_DEVICES=void
  hooks:
  - args:
    - nvidia-cdi-hook
    - enable-cuda-compat
    - --host-driver-version=570.133.20
    hookName: createContainer
    path: /usr/bin/nvidia-cdi-hook
  mounts:
  - containerPath: /usr/bin/nvidia-smi
    hostPath: /usr/bin/nvidia-smi
    options:
    - ro
    - bind
devices:
- containerEdits:
    deviceNodes:
    - path: /dev/nvidia0
  name: "0"
- containerEdits:
    deviceNodes:
    - path: /dev/nvidia0
  name: GPU-6cf00fa7-59ac-16f2-3e83-d24ccdc56f84
- containerEdits:
    deviceNodes:
    - path: /dev/nvidia0
  name: all
kind: nvidia.com/gpu
```

👉 Cette spécification CDI :

* Monte automatiquement les **bibliothèques NVIDIA** (`libcuda`, `libnvidia-ml`, etc.),
* Ajoute les **sockets et exécutables nécessaires** (`nvidia-smi`, `nvidia-persistenced`…),
* Gère les **hooks** (scripts exécutés au démarrage du conteneur) pour activer certaines fonctionnalités,
* Expose les **devices `/dev/nvidia*`**.

***

## 🎉 Félicitations

Vous venez de réaliser votre **premier build utilisant un GPU NVIDIA avec BuildKit et CDI** !
