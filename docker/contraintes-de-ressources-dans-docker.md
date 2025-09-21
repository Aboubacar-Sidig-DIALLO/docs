# 🐳 Contraintes de ressources dans Docker

### 🔹 Principe

* Par défaut, un conteneur **n’a aucune limite de ressources** :\
  → il peut utiliser **autant de mémoire et de CPU que l’ordonnanceur du noyau Linux lui autorise**.
* Cela peut poser problème : un conteneur mal configuré peut **affamer tout l’hôte**, faire tomber Docker ou d’autres applications.

👉 Docker permet donc de **fixer des limites de mémoire et CPU** via des options au lancement (`docker run`).

***

### 🔹 Mémoire

#### ⚠️ Risques

* Si un conteneur consomme trop de RAM → le noyau Linux déclenche un **OOME (Out Of Memory Exception)**.
* Le noyau tue alors des processus au hasard (par priorité), ce qui peut :
  * tuer le conteneur,
  * tuer Docker,
  * ou pire, tuer un processus critique de ton système.

⚠️ Docker ajuste déjà la priorité OOM pour protéger son démon, mais **les conteneurs eux-mêmes ne sont pas protégés**.

👉 Solution : définir des **limites mémoire**.

***

#### 🚧 Options mémoire

Toutes ces options s’utilisent avec `docker run`.

| Option                 | Description                                                                                               |
| ---------------------- | --------------------------------------------------------------------------------------------------------- |
| `-m, --memory`         | Limite dure → mémoire max que le conteneur peut utiliser (min = 6m).                                      |
| `--memory-reservation` | Limite souple → priorité à réduire la mémoire quand le système est sous pression. Doit être < `--memory`. |
| `--memory-swap`        | Quantité totale mémoire + swap.                                                                           |
| `--memory-swappiness`  | Pourcentage (0–100) → à quel point le noyau swap les pages anonymes du conteneur.                         |
| `--kernel-memory`      | Mémoire max pour le noyau (min = 6m).                                                                     |
| `--oom-kill-disable`   | Désactive le _OOM killer_ pour ce conteneur. ⚠️ À utiliser uniquement si `--memory` est défini.           |

***

#### 📌 Détails importants

**`--memory-swap`**

* Représente **mémoire + swap**.
* Cas typiques :
  * `--memory=300m --memory-swap=1g` → 300 Mo RAM + 700 Mo swap.
  * `--memory=300m --memory-swap=300m` → **pas de swap autorisé**.
  * `--memory=300m` seul → peut utiliser swap = mémoire (soit 600 Mo total).
  * `--memory-swap=-1` → swap illimité (jusqu’à ce que l’hôte soit saturé).

**`--memory-swappiness`**

* `0` → pas de swap.
* `100` → swap libre.
* Par défaut, hérite de la config de l’hôte.

**`--kernel-memory`**

* **Mémoire noyau** ≠ mémoire utilisateur.
* 4 scénarios :
  1. Illimité RAM + illimité kernel → par défaut.
  2. Illimité RAM + limité kernel → utile si tu veux protéger ton hôte.
  3. Limité RAM + illimité kernel → rare.
  4. Limité RAM + limité kernel → utile en debug mémoire (tu vois vite si un conteneur consomme trop côté user ou kernel).

***

#### ⚠️ Bonnes pratiques mémoire

* Toujours **tester** la conso mémoire avant mise en prod.
* Toujours définir **`--memory`** (sinon un conteneur peut tout manger).
* N’utilise pas `--oom-kill-disable` sans limite mémoire, sinon tu risques de geler l’hôte.
* Utilise `--memory-reservation` pour prévenir la saturation de l’hôte.
* Le swap peut aider, mais il est **lent** → utile comme **tampon**, pas comme extension de RAM.

***

### 🔹 CPU (aperçu rapide)

Bien que ton extrait se concentre sur la mémoire, Docker permet aussi de limiter le CPU avec :

* `--cpus` → nombre de CPUs max.
* `--cpu-shares` → priorité CPU relative.
* `--cpuset-cpus` → liste des cœurs CPU autorisés.

***

### ✅ Exemple concret

Limiter un conteneur Redis à **300 Mo de RAM max**, **sans swap**, et avec une priorité basse de swap :

```bash
docker run -d \
  --name redis_safe \
  --memory="300m" \
  --memory-swap="300m" \
  --memory-swappiness=0 \
  redis
```

👉 Ici :

* Redis ne peut pas dépasser 300 Mo.
* Pas de swap → meilleur perfs.
* Pas de risque qu’il tue ton hôte.

## ⚙️ Contraintes CPU et GPU dans Docker

### 🔹 Par défaut

* Un conteneur a **un accès illimité aux cycles CPU** de l’hôte.
* Sans limite, il peut saturer le CPU et ralentir les autres services.

Docker permet de fixer des **limites CPU** grâce au planificateur du noyau Linux (**CFS**, Completely Fair Scheduler) ou via le planificateur **temps réel**.

***

### 🔹 Planificateur CFS (par défaut)

#### 📌 Options disponibles

| Option                  | Description                                                                                                                           |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `--cpus=<valeur>`       | Plus simple : fraction ou multiple de CPU max autorisés. Exemple : `--cpus="1.5"` = 1,5 cœurs max.                                    |
| `--cpu-period=<valeur>` | Définit la période de temps (en µs) du scheduler (par défaut `100000` µs = 100 ms).                                                   |
| `--cpu-quota=<valeur>`  | Définit le quota CPU max par période. Exemple : `--cpu-period=100000 --cpu-quota=50000` = 50% d’un CPU.                               |
| `--cpuset-cpus`         | Restreint à certains cœurs spécifiques. Ex : `--cpuset-cpus="0-2"` = CPU 0,1,2.                                                       |
| `--cpu-shares`          | Pondération relative (par défaut 1024). Sert uniquement si plusieurs conteneurs se disputent le CPU. Plus haut = priorité plus forte. |

***

#### ✅ Exemples

👉 Limiter un conteneur Ubuntu à **50% d’un CPU** :

```bash
docker run -it --cpus=".5" ubuntu /bin/bash
```

Équivalent avec période + quota :

```bash
docker run -it --cpu-period=100000 --cpu-quota=50000 ubuntu /bin/bash
```

👉 Restreindre un conteneur à **CPU 0 et 2 uniquement** :

```bash
docker run -it --cpuset-cpus="0,2" ubuntu /bin/bash
```

👉 Donner une priorité plus forte à un conteneur web :

```bash
docker run -d --cpu-shares=2048 nginx
```

_(si un autre conteneur est à 1024, celui-ci recevra environ 2× plus de CPU en cas de contention)_.

***

### 🔹 Planificateur Temps Réel (RT)

⚠️ **Avancé et risqué** : réservé aux cas critiques (audio, télécoms, robotique, etc.).\
Mauvaise config = système instable voire inutilisable.

#### 1️⃣ Vérifier le noyau

```bash
zcat /proc/config.gz | grep CONFIG_RT_GROUP_SCHED
```

ou vérifier que `/sys/fs/cgroup/cpu.rt_runtime_us` existe.

#### 2️⃣ Configurer Docker daemon

Lancer `dockerd` avec par ex. :

```bash
--cpu-rt-runtime=950000
```

👉 signifie : sur une période de 1s (`1000000 µs`), les tâches temps réel ont droit à 950ms CPU.

#### 3️⃣ Configurer un conteneur

Besoin de `--cap-add=sys_nice` pour autoriser les priorités temps réel.

Exemple :

```bash
docker run -it \
  --cpu-rt-runtime=950000 \
  --ulimit rtprio=99 \
  --cap-add=sys_nice \
  debian:jessie
```

***

### 🔹 GPU avec Docker

Docker peut exposer une **carte NVIDIA GPU** aux conteneurs.\
👉 nécessite le **NVIDIA Container Toolkit**.

#### Étapes :

1. Installer les **drivers NVIDIA** sur l’hôte.
2. Installer `nvidia-container-toolkit`.
3. Lancer le conteneur avec `--gpus`.

***

#### ✅ Exemples GPU

👉 Exposer **tous les GPUs** et vérifier avec `nvidia-smi` :

```bash
docker run -it --rm --gpus all ubuntu nvidia-smi
```

👉 Exposer **un GPU spécifique** :

```bash
docker run -it --rm --gpus device=0 ubuntu nvidia-smi
```

👉 Exposer plusieurs GPUs :

```bash
docker run -it --rm --gpus '"device=0,2"' ubuntu nvidia-smi
```

👉 Restreindre aux capacités "utilitaires" (pour `nvidia-smi`) :

```bash
docker run --gpus 'all,capabilities=utility' --rm ubuntu nvidia-smi
```

***

### 🚀 Bonnes pratiques CPU & GPU

* **Toujours utiliser `--cpus`** pour une limite simple et claire.
* `--cpu-shares` = pondération, pas une vraie limite.
* **Utiliser `--cpuset-cpus`** pour réserver certains cœurs à un service critique.
* Temps réel : **uniquement si indispensable**.
* GPU : bien vérifier compatibilité drivers / CUDA / Docker.
