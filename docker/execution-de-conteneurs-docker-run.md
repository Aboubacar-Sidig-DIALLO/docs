# 🐳 Exécution de conteneurs (docker run)

### 🔹 1. Principe

Un conteneur est **un processus isolé** qui tourne sur une machine hôte (locale ou distante).\
Quand tu exécutes :

```bash
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

👉 Docker démarre un conteneur à partir de l’**image** spécifiée.\
Chaque conteneur a :

* Son **système de fichiers**
* Son **réseau isolé**
* Son **arbre de processus** indépendant

***

### 🔹 2. Références d’images

* **Nom + tag** : `ubuntu:24.04` (si tu omets le tag → `latest` par défaut).
*   **Digest (SHA256)** : identifiant basé sur le contenu.\
    Exemple :

    ```bash
    docker run alpine@sha256:9cacb7... date
    ```

***

### 🔹 3. Options (`[OPTIONS]`)

Quelques options utiles :

* `--name` → donner un nom au conteneur
* `-d` → exécuter en arrière-plan (détaché)
* `-it` → mode interactif avec un terminal
* `--rm` → auto-suppression à l’arrêt
* `--mount` → monter un volume ou répertoire
* `--network` → rattacher à un réseau

***

### 🔹 4. Commandes et arguments

Tu peux remplacer la commande par défaut définie dans l’image :

```bash
docker run -it ubuntu sh
```

👉 Ouvre un shell `sh` dans un conteneur basé sur Ubuntu.

***

### 🔹 5. Modes : premier plan vs arrière-plan

* **Premier plan (par défaut)** : la sortie est visible dans ton terminal.
* **Arrière-plan (`-d`)** : le conteneur tourne sans occuper ton terminal.

Exemple :

```bash
docker run -d nginx
docker logs -n 5 <ID>     # consulter les logs
docker attach <ID>        # revenir en avant-plan
```

***

### 🔹 6. Identification des conteneurs

Trois façons :

1. **ID complet** (long UUID)
2. **ID court** (12 premiers caractères)
3. **Nom** (aléatoire ou défini via `--name`)

Exemple :

```bash
docker ps -q --filter ancestor=nginx:alpine
```

👉 liste les conteneurs créés depuis l’image `nginx:alpine`.

***

### 🔹 7. Réseau des conteneurs

* Par défaut : accès Internet sortant.
* Pour que les conteneurs **se parlent entre eux**, crée un réseau utilisateur :

```bash
docker network create my-net
docker run -d --name web --network my-net nginx:alpine
docker run --rm -it --network my-net busybox
/ # ping web
```

👉 Résolution DNS possible avec le **nom du conteneur**.

***

### 🔹 8. Système de fichiers et montages

Les données dans un conteneur sont **éphémères** (disparaissent à l’arrêt).\
Pour les rendre persistantes ou les partager :

#### 🔸 Volumes

Stockage persistant géré par Docker :

```bash
docker run --rm --mount source=my_volume,target=/data busybox sh -c "echo 'hello' > /data/test.txt"
docker run --rm --mount source=my_volume,target=/data busybox cat /data/test.txt
```

#### 🔸 Bind mounts

Partager un dossier entre hôte et conteneur :

```bash
docker run -it --mount type=bind,source="$(pwd)",target=/app busybox
```

👉 Les fichiers créés/modifiés dans `/app` sont visibles sur l’hôte.

***

## 🚀 Résumé visuel

* `docker run` = crée et démarre un conteneur à partir d’une image.
* Options utiles : `-d`, `-it`, `--name`, `--rm`, `--mount`, `--network`.
* Identifiants : ID long, ID court, nom.
* Réseau : custom network = communication par nom DNS.
* Données : **volumes** (persistants) / **bind mounts** (partage hôte-conteneur).

## 🐳 Codes de sortie des conteneurs Docker

Quand un conteneur s’arrête, Docker renvoie un **code de sortie** qui permet de comprendre pourquoi.\
Tu peux afficher ce code avec `echo $?` juste après un `docker run`.

#### ✅ Codes spéciaux réservés à Docker

*   **125** → erreur venant du **daemon Docker** (ex : option inconnue)

    ```bash
    docker run --foo busybox; echo $?
    # flag provided but not defined: --foo
    # => 125
    ```
*   **126** → la commande existe mais **ne peut pas être exécutée**

    ```bash
    docker run busybox /etc; echo $?
    # => 126
    ```
*   **127** → la commande n’existe pas

    ```bash
    docker run busybox foo; echo $?
    # => 127
    ```

#### ✅ Autres codes

Tout autre code ≠ 125/126/127 → c’est le **code de sortie du programme** dans le conteneur.\
Exemple :

```bash
docker run busybox sh -c "exit 3"
echo $?
# => 3
```

***

## 🐳 Contraintes d’exécution (Runtime constraints)

Par défaut → un conteneur peut consommer **toutes les ressources de l’hôte**.\
Docker permet de limiter mémoire, CPU, I/O, etc. via des **flags**.

***

### 🔹 Mémoire

* `-m, --memory="512m"` → limite mémoire dure (hard limit).
* `--memory-swap="1g"` → mémoire + swap max.
* `--memory-reservation="200m"` → limite mémoire souple (soft limit).
* `--kernel-memory="100m"` → limite mémoire noyau.
* `--oom-kill-disable` → empêche le kernel de tuer le conteneur en cas de manque mémoire (⚠️ risqué).
* `--oom-score-adj` → priorité d’OOM (-1000 à 1000).
* `--memory-swappiness=0` → empêche le swap (0 = jamais, 100 = swap max).
* `--shm-size="128m"` → taille de `/dev/shm` (mémoire partagée).

***

### 🔹 CPU

* `--cpus="1.5"` → limite le conteneur à 1,5 CPU.
* `--cpu-shares=512` → poids relatif (soft limit, défaut = 1024).
* `--cpu-quota=50000 --cpu-period=100000` → contrôle fin du CFS scheduler.
* `--cpuset-cpus="0-2"` → forcer à tourner sur certains cœurs.
* `--cpuset-mems="0,1"` → lier à certaines banques mémoire NUMA.
* `--cpu-rt-period / --cpu-rt-runtime` → limites pour scheduling temps-réel.
* `--ulimit rtprio=99` + `--cap-add=sys_nice` → activer la priorité temps-réel.

***

### 🔹 I/O disque (Block IO)

* `--blkio-weight=500` → poids relatif pour l’accès disque (10–1000).
* `--blkio-weight-device=/dev/sda:300` → pondération par périphérique.
* `--device-read-bps=/dev/sda:10mb` → limite lecture à 10 Mo/s.
* `--device-write-bps=/dev/sda:5mb` → limite écriture à 5 Mo/s.
* `--device-read-iops=/dev/sda:100` → limite à 100 lectures/s.
* `--device-write-iops=/dev/sda:50` → limite à 50 écritures/s.

***

## 🚀 Exemple pratique

Limiter un conteneur Nginx à **512 Mo RAM**, **1 CPU** et **10 Mo/s max en écriture disque** :

```bash
docker run -d --name web \
  -m 512m \
  --cpus=1 \
  --device-write-bps /dev/sda:10mb \
  nginx
```

## 🐳 Contraintes de mémoire utilisateur dans Docker

Par défaut → **un conteneur peut utiliser toute la RAM et le swap de l’hôte**.\
Tu peux ajuster ce comportement avec les options **`--memory`**, **`--memory-swap`**, **`--memory-reservation`**, et les mécanismes liés à l’**OOM killer**.

***

### 🔹 4 scénarios de configuration mémoire

| Option                                    | Résultat                                                                    |
| ----------------------------------------- | --------------------------------------------------------------------------- |
| `--memory=inf --memory-swap=inf` (défaut) | Pas de limite. Le conteneur peut consommer autant de RAM + swap qu’il veut. |
| `--memory=L --memory-swap=-1`             | RAM limitée à `L`, mais swap **illimité** (si activé sur l’hôte).           |
| `--memory=L` (sans `--memory-swap`)       | RAM limitée à `L`. Swap = **L par défaut** (donc RAM + swap = `2*L`).       |
| `--memory=L --memory-swap=S (S ≥ L)`      | RAM limitée à `L`. RAM + swap = `S`. Donc swap dispo = `S - L`.             |

***

#### Exemples pratiques

1.  **Sans limite**

    ```bash
    docker run -it ubuntu:24.04 bash
    ```

    👉 RAM et swap illimités.
2.  **300M RAM, swap illimité**

    ```bash
    docker run -it -m 300M --memory-swap -1 ubuntu:24.04 bash
    ```

    👉 RAM limitée à 300M, mais peut utiliser autant de swap que l’hôte autorise.
3.  **300M RAM, 300M swap (par défaut)**

    ```bash
    docker run -it -m 300M ubuntu:24.04 bash
    ```

    👉 Total virtuel = 600M (300M RAM + 300M swap).
4.  **300M RAM, 700M swap**

    ```bash
    docker run -it -m 300M --memory-swap 1G ubuntu:24.04 bash
    ```

    👉 300M RAM + 700M swap = 1 Go max.

***

### 🔹 `--memory-reservation` (limite souple)

* **Soft limit** : Docker essaie de garder le conteneur sous la valeur fixée **en cas de contention mémoire**.
* Ne garantit pas le respect strict, mais sert de “seuil de confort”.

⚠️ La valeur doit toujours être **< `--memory`** si les deux sont utilisés.

#### Exemple 1 : RAM max 500M, réservation 200M

```bash
docker run -it -m 500M --memory-reservation 200M ubuntu:24.04 bash
```

👉 Le conteneur peut utiliser jusqu’à 500M, mais en cas de pression mémoire, le kernel tente de le ramener sous 200M.

#### Exemple 2 : uniquement réservation 1G

```bash
docker run -it --memory-reservation 1G ubuntu:24.04 bash
```

👉 Pas de limite dure, mais en cas de contention le conteneur est bridé vers 1 Go.

***

### 🔹 OOM Killer et priorités

Quand l’hôte manque de mémoire, le **kernel tue des processus** (OOM = Out Of Memory).

* **Par défaut** → un processus du conteneur peut être tué.
* **`--oom-kill-disable`** → désactive ce comportement (⚠️ dangereux).
  * À utiliser **seulement** si `--memory` est défini.
  * Sinon, l’hôte peut tomber (kill de services critiques du système).

#### Exemple sûr :

```bash
docker run -it -m 100M --oom-kill-disable ubuntu:24.04 bash
```

👉 Le conteneur est limité à 100M, mais ses processus ne sont pas tués automatiquement par OOM.

#### Exemple dangereux :

```bash
docker run -it --oom-kill-disable ubuntu:24.04 bash
```

👉 RAM illimitée, OOM killer désactivé → l’hôte peut crasher si le conteneur consomme toute la mémoire.

***

### 🔹 Priorité avec `--oom-score-adj`

* Valeurs de **-1000 à +1000** :
  * Négatif → moins de chances d’être tué.
  * Positif → plus de chances d’être tué.

Exemple : protéger un conteneur critique (ex: base de données)

```bash
docker run -d -m 512M --oom-score-adj=-500 postgres
```

***

## 🚀 En résumé

* `--memory` = limite dure (hard limit).
* `--memory-swap` = RAM + swap max.
* `--memory-reservation` = limite souple (soft limit).
* `--oom-kill-disable` = désactive le tueur OOM (⚠️ danger si pas de `--memory`).
* `--oom-score-adj` = définit la priorité de survie en cas de manque mémoire.

## 🐳 Contraintes de mémoire noyau dans Docker

### 🔹 Différence fondamentale

* **Mémoire utilisateur (User memory)** → piles, tas, données applicatives. Elle peut être **swappée** (envoyée sur disque quand la RAM manque).
* **Mémoire noyau (Kernel memory)** → utilisée pour :
  * les **pages de pile (stack pages)**,
  * les **slab pages** (structures internes du noyau),
  * la mémoire des **sockets**,
  * la mémoire **TCP** (buffers réseau).\
    👉 **Elle ne peut pas être swappée !**

⚠️ Cela signifie qu’un conteneur qui consomme trop de mémoire noyau peut bloquer le système entier (même si tu as encore de la RAM libre).

***

### 🔹 Modes de configuration

On suppose :

* **U = limite mémoire utilisateur (`--memory`)**
* **K = limite mémoire noyau (`--kernel-memory`)**

| Option                         | Résultat                                                                                                                                                                                                                         |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `U != 0, K = inf` (par défaut) | Cas normal : seule la mémoire utilisateur est limitée. La mémoire noyau est ignorée.                                                                                                                                             |
| `U != 0, K < U`                | La mémoire noyau est un **sous-ensemble** de la mémoire utilisateur. Utile si tu veux éviter la surconsommation de mémoire noyau. ⚠️ Mais un mauvais réglage peut encore faire planter la machine (mémoire noyau non libérable). |
| `U != 0, K > U`                | La mémoire noyau fait partie de la limite utilisateur. Cela donne une **vue unifiée** (tout est comptabilisé ensemble).                                                                                                          |

***

### 🔹 Exemples pratiques

1. **Mémoire totale 500M, noyau max 50M**

```bash
docker run -it -m 500M --kernel-memory 50M ubuntu:24.04 bash
```

👉 Le conteneur peut utiliser au total **500M**, dont **max 50M pour le noyau**.

***

2. **Seulement limite noyau (50M)**

```bash
docker run -it --kernel-memory 50M ubuntu:24.04 bash
```

👉 Pas de limite pour la mémoire utilisateur,\
mais **seulement 50M de mémoire noyau**.\
Si l’application crée beaucoup de sockets ou de threads → elle sera bloquée.

***

### 🔹 ⚠️ Bonnes pratiques

* Toujours définir **`--memory` ET `--kernel-memory`** ensemble → éviter que le conteneur consomme trop de RAM ou bloque l’hôte.
* Ne pas trop **sous-dimensionner** `--kernel-memory`, sinon :
  * impossible de créer de nouveaux threads,
  * erreurs réseau (sockets refusés),
  * conteneur bloqué.
* Utiliser surtout en **production sensible** (ex: multi-tenant, hébergement mutualisé).

***

### 🚀 Résumé

* La **mémoire noyau** est critique car non swappable.
* `--kernel-memory` te permet de limiter combien un conteneur peut utiliser pour ses piles, sockets, TCP buffers.
* Configurations possibles :
  * `K ignoré` (par défaut),
  * `K < U` → limite stricte noyau,
  * `K > U` → tout compte dans la limite utilisateur.
* À utiliser **en complément de `--memory`** pour éviter de bloquer ton système.

## 🐳 Contrainte **Swappiness** dans Docker

### 🔹 Qu’est-ce que le _swappiness_ ?

* Le **swappiness** définit **dans quelle mesure le noyau peut échanger (swap)** des pages de mémoire **anonymes** (non liées à un fichier, ex: heap, stack) vers le disque.
* Ça permet d’éviter un crash quand la RAM est saturée, mais le **swap est beaucoup plus lent** → impact négatif sur les performances.

👉 Valeurs possibles :

* `0` → **interdit le swap** de la mémoire anonyme. (Tout reste en RAM.)
* `100` → **tout est swappable** (le noyau peut déplacer agressivement la mémoire vers le swap).
* (par défaut) → hérite de la configuration du **host**.

***

### 🔹 Utilisation dans Docker

Tu peux contrôler ce comportement avec l’option :

```bash
docker run -it --memory-swappiness=<0-100> ubuntu:24.04 /bin/bash
```

***

### 🔹 Exemple pratique

#### 1. Pas de swap (swappiness = 0)

```bash
docker run -it --memory-swappiness=0 ubuntu:24.04 bash
```

➡️ Les pages mémoire restent en RAM → très performant, mais risque d’**OOM (Out Of Memory)** si la RAM est saturée.

***

#### 2. Swap autorisé à 100%

```bash
docker run -it --memory-swappiness=100 ubuntu:24.04 bash
```

➡️ Le conteneur peut envoyer **toutes ses pages anonymes** vers le swap → ralentit fortement si le swap disque est utilisé.

***

#### 3. Valeur intermédiaire (ex: 60)

```bash
docker run -it --memory-swappiness=60 ubuntu:24.04 bash
```

➡️ Le noyau choisit un compromis : garder une partie en RAM, envoyer une partie dans le swap.

***

### 🔹 Quand l’utiliser ?

✅ Cas où `--memory-swappiness=0` est recommandé :

* Applications sensibles à la latence (ex: bases de données, serveurs temps réel).
* Tu veux éviter les pénalités liées au swap.

✅ Cas où un `--memory-swappiness` élevé est utile :

* Conteneurs de calcul batch, où un ralentissement est tolérable mais un OOM ne l’est pas.
* Environnements de test/CI où la stabilité est prioritaire à la vitesse.

## 🐳 Contrainte **CPU share** dans Docker

### 🔹 Principe

* Par défaut, **tous les conteneurs partagent équitablement** le CPU de l’hôte.
* Docker utilise une **valeur de pondération** (`--cpu-shares`) pour définir la proportion de cycles CPU attribués.
* La valeur par défaut est **1024** (équilibre).
* Plus la valeur est élevée → plus le conteneur a de **poids** dans l’allocation CPU.
* ⚠️ Ce n’est **pas une limite stricte**, mais une **priorité relative**.

***

### 🔹 Syntaxe

```bash
docker run -it -c <valeur> ubuntu:24.04 bash
# ou
docker run -it --cpu-shares=<valeur> ubuntu:24.04 bash
```

***

### 🔹 Exemples

#### 1. Trois conteneurs

* `C1`: `--cpu-shares=1024`
* `C2`: `--cpu-shares=512`
* `C3`: `--cpu-shares=512`

👉 Quand les trois demandent 100% du CPU :

* `C1` reçoit **50%**
* `C2` reçoit **25%**
* `C3` reçoit **25%**

***

#### 2. Ajout d’un quatrième conteneur

* `C1`: `1024`
* `C2`: `512`
* `C3`: `512`
* `C4`: `1024`

👉 Quand les 4 demandent 100% :

* `C1`: **33%**
* `C2`: **16.5%**
* `C3`: **16.5%**
* `C4`: **33%**

***

### 🔹 Sur un système multi-cœurs

* Les parts CPU se répartissent sur **tous les cœurs**.
* Même si un conteneur est limité à **moins de 100% global**, il peut **saturer un cœur complet** si les autres sont libres.

📌 Exemple :

* `C0`: `--cpu-shares=512` avec **1 process**
* `C1`: `--cpu-shares=1024` avec **2 process**

👉 Répartition possible :

* `C0` → **100% de CPU0**
* `C1` → **100% de CPU1 + 100% de CPU2**

***

### 🔹 Points importants

* `--cpu-shares=0` = Docker ignore et prend la valeur **par défaut (1024)**.
* Ce mécanisme agit **uniquement quand il y a contention CPU**.\
  Si un conteneur est seul, il peut utiliser **100% du CPU** quelle que soit sa valeur.
* C’est une **pondération relative**, pas une **limite stricte** (contrairement à `--cpus` ou `--cpu-quota`).

## 🐳 Contrainte **CPU period** et **CPU quota** dans Docker

### 🔹 Contexte

Docker utilise le **CFS (Completely Fair Scheduler)** du noyau Linux pour limiter l’usage CPU.\
Par défaut :

* Une période = **100ms (100000 µs)**
* À l’intérieur de cette période, on définit **combien de temps** le conteneur peut utiliser le CPU.

👉 On joue avec **2 options** :

* `--cpu-period` → définit la **longueur de la période** (par défaut 100000 µs = 100 ms).
* `--cpu-quota` → définit la **quantité de temps CPU autorisée** dans chaque période.

***

### 🔹 Exemple de calcul

#### Cas 1 : Limiter à 50% du CPU

```bash
docker run -it --cpu-period=50000 --cpu-quota=25000 ubuntu:24.04 /bin/bash
```

* `--cpu-period=50000` → période = 50 ms
* `--cpu-quota=25000` → quota = 25 ms

👉 Cela veut dire que le conteneur peut **utiliser le CPU 25 ms sur 50 ms**\
\= **50% du CPU disponible**

***

#### Cas 2 : Équivalent avec `--cpus`

```bash
docker run -it --cpus=0.5 ubuntu:24.04 /bin/bash
```

* Ici, `--cpus=0.5` = **50% d’un CPU**
* C’est un raccourci qui revient à écrire `--cpu-period` et `--cpu-quota`.

***

#### Cas 3 : Un CPU entier

```bash
docker run -it --cpu-period=100000 --cpu-quota=100000 ubuntu:24.04 /bin/bash
```

* Le conteneur peut utiliser **100 ms sur 100 ms** → accès complet à **1 CPU**.

***

#### Cas 4 : 2 CPUs

```bash
docker run -it --cpu-period=100000 --cpu-quota=200000 ubuntu:24.04 /bin/bash
```

* Ici : quota = 200 ms par période de 100 ms\
  👉 donc le conteneur peut utiliser **2 CPUs complets**.

***

### 🔹 Différences entre les options

* `--cpu-shares` → pondération relative (souple, agit seulement en cas de contention).
* `--cpu-quota` + `--cpu-period` → limitation stricte en temps CPU.
* `--cpus` → raccourci pratique pour quota + période.

***

✅ **En résumé** :

* `--cpu-period` définit la fenêtre de temps (par défaut 100 ms).
* `--cpu-quota` définit combien de temps CPU est autorisé dans cette fenêtre.
* `quota ÷ period` = **% du CPU accessible**.

## 🐳 Cpuset constraint dans Docker

### 🔹 Qu’est-ce que c’est ?

Par défaut, un conteneur peut utiliser **tous les cœurs CPU** et **toute la mémoire** disponibles sur la machine hôte.\
Avec **cpuset**, tu peux restreindre un conteneur à utiliser **certains CPU spécifiques** ou **certains nœuds mémoire** (utile sur les systèmes **NUMA**).

👉 Cela permet de mieux contrôler les performances et d’éviter que les conteneurs se marchent dessus.

***

### 🔹 Restreindre les **CPU**

#### Exemple 1 : Utiliser uniquement CPU 1 et CPU 3

```bash
docker run -it --cpuset-cpus="1,3" ubuntu:24.04 /bin/bash
```

* Le conteneur n’exécutera ses processus **que sur CPU 1 et CPU 3**.
* Utile si tu veux **isoler une application** sur certains cœurs.

***

#### Exemple 2 : Utiliser CPU 0, 1 et 2

```bash
docker run -it --cpuset-cpus="0-2" ubuntu:24.04 /bin/bash
```

* Le conteneur pourra s’exécuter sur **CPU 0, 1 et 2**.
* Note que `0-2` est équivalent à `0,1,2`.

***

### 🔹 Restreindre la **mémoire** (NUMA systems)

Sur des serveurs **NUMA (Non-Uniform Memory Access)**, la RAM est divisée en **nœuds mémoire** associés à des CPU spécifiques.\
Avec `--cpuset-mems`, tu peux dire au conteneur de n’utiliser que certains nœuds mémoire.

#### Exemple 1 : Utiliser mémoire des nœuds 1 et 3

```bash
docker run -it --cpuset-mems="1,3" ubuntu:24.04 /bin/bash
```

#### Exemple 2 : Utiliser mémoire des nœuds 0, 1 et 2

```bash
docker run -it --cpuset-mems="0-2" ubuntu:24.04 /bin/bash
```

⚠️ Cela **n’a d’effet que sur NUMA**.\
Sur une machine classique (non-NUMA), cette option est ignorée.

***

### 🔹 Quand utiliser cpuset ?

✅ Cas d’usage typiques :

* **Performance critique** : isoler une app sensible sur certains cœurs CPU.
* **Multi-conteneurs** : éviter que 2 conteneurs se concurrencent sur les mêmes CPU.
* **NUMA** : coller un conteneur à un nœud mémoire + ses CPU pour **réduire la latence**.

❌ À éviter :

* Sur des petites machines (sans NUMA ou avec peu de CPU), car ça risque de sous-utiliser les ressources.
* Si ton application a besoin de scaler dynamiquement → mieux d’utiliser `--cpus` ou `--cpu-shares`.

## 🐳 CPU Quota Constraint dans Docker

### 🔹 Qu’est-ce que c’est ?

Docker s’appuie sur le **CFS (Completely Fair Scheduler)** du noyau Linux pour gérer le temps CPU attribué aux conteneurs.

Avec l’option **`--cpu-quota`**, tu peux fixer la **quantité maximale de temps CPU** qu’un conteneur peut utiliser pendant une période donnée.

👉 Cette période est définie par `--cpu-period` (par défaut **100 000 µs = 100 ms**).\
👉 `--cpu-quota` définit combien de microsecondes de CPU le conteneur peut utiliser pendant cette période.

***

### 🔹 Exemple simple (1 CPU)

```bash
docker run -it --cpu-quota=50000 ubuntu:24.04 /bin/bash
```

* Par défaut, `--cpu-period=100000` (100 ms).
* `--cpu-quota=50000` = le conteneur peut utiliser **50 000 µs sur 100 000 µs**.
* Résultat : **50% d’un CPU** maximum.

***

### 🔹 Exemple avec plusieurs CPUs

Si l’hôte a **2 CPU** et que tu veux limiter un conteneur à **1 CPU complet** :

```bash
docker run -it --cpu-quota=100000 --cpu-period=100000 ubuntu:24.04 /bin/bash
```

* Ici, le conteneur a droit à **100% d’un CPU** (même si la machine en a 2).
* Ça veut dire qu’il ne peut jamais dépasser **1 cœur**.

***

### 🔹 Équivalence avec `--cpus`

Aujourd’hui, Docker recommande plutôt **`--cpus`**, qui est plus simple.

Exemple :

```bash
docker run -it --cpus=0.5 ubuntu:24.04 /bin/bash
```

➡️ Équivaut à :

```bash
docker run -it --cpu-period=100000 --cpu-quota=50000 ubuntu:24.04 /bin/bash
```

***

### 🔹 Résumé des usages

| Option         | Signification                                                |
| -------------- | ------------------------------------------------------------ |
| `--cpu-period` | Durée d’un cycle CPU (par défaut 100 ms)                     |
| `--cpu-quota`  | Temps CPU autorisé dans cette période                        |
| `--cpus`       | Plus simple → fixe directement le nombre de CPUs utilisables |

## 🐳 Contraintes **Block I/O (blkio)** dans Docker

### 🔹 1. Qu’est-ce que le blkio ?

* Le **blkio (Block I/O)** représente les **entrées/sorties disque** (lecture/écriture sur un périphérique de stockage : SSD, HDD, etc.).
* Par défaut, tous les conteneurs partagent équitablement la bande passante disque avec un poids de **500** (sur une échelle 10 → 1000).

⚠️ Attention :\
Ces règles ne s’appliquent qu’aux **I/O directes (oflag=direct)**.\
Le **Buffered I/O (cache mémoire)** n’est pas pris en compte.

***

### 🔹 2. Poids relatif (`--blkio-weight`)

Permet de définir une **priorité** entre conteneurs.

Exemple :

```bash
docker run -it --name c1 --blkio-weight 300 ubuntu:24.04 /bin/bash
docker run -it --name c2 --blkio-weight 600 ubuntu:24.04 /bin/bash
```

➡️ Si les deux font des opérations disque simultanément :

* c1 recevra \~**33%** du débit disque
* c2 recevra \~**67%**

***

### 🔹 3. Poids par périphérique (`--blkio-weight-device`)

Permet de fixer un poids spécifique par périphérique.

Exemple :

```bash
docker run -it \
  --blkio-weight 300 \
  --blkio-weight-device "/dev/sda:200" \
  ubuntu
```

➡️ Ici : poids **300** global, mais sur **/dev/sda** le poids est **200**.

***

### 🔹 4. Limiter la bande passante (BPS : bytes per second)

* **Lecture** limitée à 1 MB/s :

```bash
docker run -it --device-read-bps /dev/sda:1mb ubuntu
```

* **Écriture** limitée à 1 MB/s :

```bash
docker run -it --device-write-bps /dev/sda:1mb ubuntu
```

👉 Format : `<device-path>:<valeur>[kb|mb|gb]`

***

### 🔹 5. Limiter les IOPS (opérations disque / seconde)

* Lecture limitée à 1000 IOPS :

```bash
docker run -it --device-read-iops /dev/sda:1000 ubuntu
```

* Écriture limitée à 1000 IOPS :

```bash
docker run -it --device-write-iops /dev/sda:1000 ubuntu
```

👉 Format : `<device-path>:<valeur>` (valeur = entier positif).

***

### 🔹 6. Résumé des options blkio

| Option                  | Rôle                                              |
| ----------------------- | ------------------------------------------------- |
| `--blkio-weight`        | Poids global du conteneur (10-1000, défaut = 500) |
| `--blkio-weight-device` | Poids spécifique à un périphérique                |
| `--device-read-bps`     | Limite de lecture en **bytes/sec**                |
| `--device-write-bps`    | Limite d’écriture en **bytes/sec**                |
| `--device-read-iops`    | Limite de lecture en **IOPS/sec**                 |
| `--device-write-iops`   | Limite d’écriture en **IOPS/sec**                 |

***

✅ En pratique :

* **`--blkio-weight`** = priorités relatives (qui a plus/moins de bande passante).
* **`--device-read/write-bps`** = limite en débit absolu (ex : 1 MB/s max).
* **`--device-read/write-iops`** = limite en nombre d’opérations (utile pour HDD).

## 👥 **Docker – Groupes supplémentaires (`--group-add`)**

### 🔹 1. Principe

* Quand un conteneur s’exécute, il tourne avec un **utilisateur** et ses **groupes secondaires** (définis dans `/etc/group` de l’image).
* Par défaut : Docker applique uniquement les groupes associés à l’utilisateur (par ex. `root` → `wheel`).
* Avec `--group-add`, tu peux **ajouter des groupes supplémentaires** au processus du conteneur.

👉 C’est utile quand tu veux que ton conteneur ait accès à des ressources ou périphériques protégés par certains **groupes Unix** (ex : `audio`, `video`, `docker`, etc.).

***

### 🔹 2. Exemple de base

```bash
docker run --rm --group-add audio --group-add nogroup --group-add 777 busybox id
```

#### Sortie possible :

```
uid=0(root) gid=0(root) groups=10(wheel),29(audio),99(nogroup),777
```

➡️ Le conteneur tourne en tant que **root** (`uid=0, gid=0`),\
mais appartient aussi aux groupes **audio**, **nogroup** et **777**.

***

### 🔹 3. Cas d’usage courant

1. **Accès aux périphériques audio/vidéo** :

```bash
docker run --rm -it --group-add audio --device /dev/snd ubuntu bash
```

➡️ Permet à un conteneur d’accéder au micro/haut-parleurs.

2. **Accès à Docker depuis un conteneur** :

```bash
docker run -it --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --group-add $(getent group docker | cut -d: -f3) \
  docker-cli
```

➡️ Le conteneur peut communiquer avec le daemon Docker via le socket.

***

### 🔹 4. Valeurs acceptées

* **Nom de groupe** (`--group-add audio`)
* **ID numérique** (`--group-add 777`)

⚠️ Attention :\
Le groupe doit exister dans l’hôte ou être défini dans `/etc/group` de l’image.\
Sinon, tu verras un simple **ID** sans nom.

***

### 🔹 5. Comparaison avec `--user`

* `--user UID:GID` → définit **l’utilisateur principal** et son **groupe principal**.
* `--group-add` → ajoute des **groupes secondaires supplémentaires**.

Exemple combiné :

```bash
docker run --rm --user 1001:1001 --group-add audio ubuntu id
```

Sortie possible :

```
uid=1001 gid=1001 groups=1001,29(audio)
```

***

✅ En résumé :

* `--group-add` te permet **d’ajouter des permissions supplémentaires** via des groupes Unix.
* Très utile pour les périphériques (`audio`, `video`, `tty`), ou pour donner accès à certains fichiers ou sockets (`docker.sock`, `usb`, etc.).

## 🔐 **Docker – Privilèges et capacités Linux**

### 🔹 1. Principe

* Par défaut, un conteneur Docker est **non privilégié** → il a un **ensemble réduit de permissions** par rapport à un processus classique du système hôte.
* Ces permissions sont contrôlées par :
  * **Capabilities Linux** → permissions granulaires (ex : changer l’IP, charger un module kernel…).
  * **AppArmor / SELinux** → sécurité complémentaire.
  * **cgroups devices** → contrôle de l’accès aux périphériques (`/dev/*`).

👉 Donc : un conteneur **n’a pas tous les droits** pour éviter les risques de sécurité.

***

### 🔹 2. Options Docker principales

#### a) `--privileged`

* Donne **tous les droits** au conteneur.
* Accès à **tous les périphériques** (`/dev/*`).
* Désactive une grande partie des restrictions (AppArmor, SELinux).
* ⚠️ **Très dangereux** : c’est presque comme exécuter un root sur la machine hôte.

Exemple :

```bash
docker run --privileged -it ubuntu bash
```

***

#### b) `--device`

* Permet d’exposer **un périphérique spécifique** au conteneur (au lieu de `--privileged` qui les donne tous).
* Par défaut : accès en lecture + écriture + création (`rwm`).
* Tu peux restreindre avec `:r`, `:w`, `:m`.

Exemples :

```bash
# Accès complet au disque
docker run -it --device=/dev/sda:/dev/xvdc ubuntu bash

# Lecture seule
docker run -it --device=/dev/sda:/dev/xvdc:r ubuntu bash

# Accès audio
docker run -it --device=/dev/snd ubuntu bash
```

***

#### c) `--cap-add` et `--cap-drop`

* Linux divise les permissions root en **"capabilities"** (capacités).
* Par défaut, Docker **ajoute un ensemble de base** (comme `NET_BIND_SERVICE`, `KILL`, `CHOWN`).
* Tu peux :
  * **Ajouter** une capacité : `--cap-add`
  * **Retirer** une capacité : `--cap-drop`

Exemples :

```bash
# Ajouter SYS_ADMIN (super-puissant, permet de monter des systèmes de fichiers)
docker run -it --cap-add=SYS_ADMIN ubuntu bash

# Retirer NET_RAW (empêche le conteneur de créer des sockets RAW)
docker run -it --cap-drop=NET_RAW ubuntu bash
```

⚠️ `SYS_ADMIN` est quasiment équivalent à `--privileged` → à éviter sauf besoin précis.

***

### 🔹 3. Exemples pratiques

#### 🔧 Réseau : ajouter une interface

```bash
docker run -it ubuntu ip link add dummy0 type dummy
# Erreur → pas de droits

docker run -it --cap-add=NET_ADMIN ubuntu ip link add dummy0 type dummy
# ✅ fonctionne
```

#### 🔧 FUSE (ex: `sshfs`)

```bash
# Pas de droit
docker run -it sshfs sshfs user@host:/dir /mnt

# Nécessite à la fois une capacité et un device
docker run -it \
  --cap-add SYS_ADMIN \
  --device /dev/fuse \
  sshfs sshfs user@host:/dir /mnt
```

***

### 🔹 4. Liste des capacités (simplifiée)

#### ✅ Par défaut (exemples utiles)

* `CHOWN` → changer les propriétaires de fichiers
* `NET_BIND_SERVICE` → ouvrir un port < 1024
* `KILL` → tuer un processus
* `SYS_CHROOT` → utiliser `chroot`

#### ❌ Non inclus par défaut (à ajouter si besoin)

* `NET_ADMIN` → modifier la configuration réseau
* `SYS_ADMIN` → actions d’admin système (montages, namespace…)
* `SYS_MODULE` → charger des modules kernel
* `SYS_PTRACE` → déboguer un autre processus
* `CAP_BPF` → utiliser eBPF

***

### 🔹 5. Bonnes pratiques

* **Éviter `--privileged`** (trop puissant, trou de sécurité).
* Utiliser **`--device` + `--cap-add`** seulement pour ce qui est nécessaire.
* Vérifier toujours quelles capacités ton appli **doit réellement avoir**.

***

✅ **En résumé :**

* `--privileged` = tous les droits (dangereux ⚠️).
* `--device` = accès à un périphérique précis (`/dev/...`).
* `--cap-add / --cap-drop` = gestion fine des permissions Linux.

## ⚙️ **Surcharger les paramètres par défaut d’une image**

Quand tu construis une image avec un **Dockerfile**, certains paramètres sont définis par défaut :

* **ENTRYPOINT** → programme principal exécuté (ex : `nginx`)
* **CMD** → arguments ou commande par défaut (ex : `["nginx", "-g", "daemon off;"]`)
* **EXPOSE** → ports exposés
* **ENV** → variables d’environnement
* **HEALTHCHECK** → test d’état de santé du conteneur
* **USER** → utilisateur qui exécute le processus
* **WORKDIR** → répertoire de travail

👉 Avec `docker run`, tu peux **remplacer ces paramètres**.

***

### 🔹 1. **ENTRYPOINT** (point d’entrée)

Définit le programme qui sera exécuté au démarrage.

#### Exemple dans un Dockerfile :

```dockerfile
ENTRYPOINT ["echo", "Bonjour"]
```

Si tu lances :

```bash
docker run monimage
```

➡️ Affiche `Bonjour`.

#### Pour le surcharger :

```bash
docker run --entrypoint ls monimage /
```

➡️ Le conteneur exécutera `ls /` au lieu de `echo Bonjour`.

***

### 🔹 2. **CMD** (commande par défaut)

Ajoute une commande ou des arguments par défaut.

#### Exemple :

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

* Si tu lances simplement :

```bash
docker run nginx
```

➡️ Démarre `nginx`.

* Pour le surcharger :

```bash
docker run nginx echo "Hello"
```

➡️ Exécute `echo Hello` à la place.

***

### 🔹 3. **EXPOSE** (ports exposés)

Indique quels ports l’image prévoit d’utiliser.

#### Exemple dans le Dockerfile :

```dockerfile
EXPOSE 80
```

#### Pour le surcharger (et mapper) :

```bash
docker run -p 8080:80 nginx
```

➡️ Le port **80 interne** du conteneur est mappé sur **8080 de l’hôte**.

***

### 🔹 4. **ENV** (variables d’environnement)

Définit des variables par défaut.

#### Exemple dans Dockerfile :

```dockerfile
ENV MODE=production
```

#### Pour le surcharger :

```bash
docker run -e MODE=dev monimage
```

➡️ Dans ce conteneur, `MODE=dev`.

***

### 🔹 5. **HEALTHCHECK**

Définit un test pour vérifier que le conteneur est en bonne santé.

#### Exemple :

```dockerfile
HEALTHCHECK CMD curl -f http://localhost/ || exit 1
```

#### Pour le désactiver :

```bash
docker run --no-healthcheck monimage
```

***

### 🔹 6. **USER** (utilisateur)

Définit l’utilisateur par défaut du conteneur.

#### Exemple :

```dockerfile
USER appuser
```

#### Pour lancer avec un autre utilisateur :

```bash
docker run -u root monimage
```

***

### 🔹 7. **WORKDIR** (répertoire de travail)

Définit le répertoire de travail.

#### Exemple :

```dockerfile
WORKDIR /app
```

#### Pour le surcharger :

```bash
docker run -w /tmp monimage pwd
```

➡️ Affiche `/tmp`.

***

## ✅ **Résumé rapide**

| Dockerfile    | Paramètre par défaut | Surcharge avec `docker run` |
| ------------- | -------------------- | --------------------------- |
| `ENTRYPOINT`  | Commande principale  | `--entrypoint`              |
| `CMD`         | Commande / arguments | Ajouter une commande/args   |
| `EXPOSE`      | Ports exposés        | `-p` ou `--publish`         |
| `ENV`         | Variables d’env.     | `-e`                        |
| `HEALTHCHECK` | Vérification santé   | `--no-healthcheck`          |
| `USER`        | Utilisateur          | `-u`                        |
| `WORKDIR`     | Répertoire           | `-w`                        |

## ⚙️ **Commande et options par défaut (`CMD`)**

Quand tu lances un conteneur avec :

```bash
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

👉 La partie `[COMMAND] [ARG...]` est **optionnelle**.\
En effet, l’image peut déjà définir un **`CMD`** dans son **Dockerfile**.

***

### 🔹 1. **Cas simple : seulement CMD**

Exemple de Dockerfile :

```dockerfile
FROM ubuntu:24.04
CMD ["echo", "Hello depuis CMD"]
```

* Si tu lances :

```bash
docker run monimage
```

➡️ Affiche : `Hello depuis CMD`

* Si tu surcharges avec une commande :

```bash
docker run monimage ls /
```

➡️ Affiche le contenu de `/`, car ton **CMD est remplacé**.

***

### 🔹 2. **Cas avec ENTRYPOINT**

Exemple de Dockerfile :

```dockerfile
FROM ubuntu:24.04
ENTRYPOINT ["echo"]
CMD ["Hello", "par défaut"]
```

* Si tu lances :

```bash
docker run monimage
```

➡️ Exécute : `echo Hello par défaut`

* Si tu surcharges avec une commande :

```bash
docker run monimage Bonjour Docker
```

➡️ Exécute : `echo Bonjour Docker`

⚠️ Ici, la différence importante :\
👉 Avec un `ENTRYPOINT`, le `CMD` devient **des arguments par défaut** de l’`ENTRYPOINT`.\
👉 Si tu passes un `COMMAND` dans `docker run`, il **remplace le CMD**, mais reste un **argument de l’ENTRYPOINT**.

***

### 🔹 3. **Résumé clair**

* **`CMD` seul** = commande par défaut (remplaçable avec `docker run IMAGE autre_commande`).
* **`ENTRYPOINT` seul** = commande fixe (surchargée seulement avec `--entrypoint`).
* **`ENTRYPOINT + CMD`** = `ENTRYPOINT` est fixe, `CMD` fournit des **arguments par défaut**.

***

### 🔹 4. **Exemple concret**

Dockerfile :

```dockerfile
FROM alpine
ENTRYPOINT ["ping"]
CMD ["localhost"]
```

* Lancer sans argument :

```bash
docker run monimage
```

➡️ Fait `ping localhost`

* Lancer avec argument :

```bash
docker run monimage google.com
```

➡️ Fait `ping google.com`

* Changer complètement l’ENTRYPOINT :

```bash
docker run --entrypoint ls monimage /
```

➡️ Fait `ls /` (plus de `ping`).

## ⚙️ **Entrypoint par défaut (`ENTRYPOINT`)**

### 🔹 1. Qu’est-ce que l’ENTRYPOINT ?

* Défini dans le **Dockerfile** via `ENTRYPOINT`.
* Représente **l’exécutable principal** du conteneur (le programme qui démarre automatiquement quand tu fais `docker run`).
* L’idée est de faire en sorte que ton conteneur **se comporte comme une commande**.

👉 Exemple de Dockerfile :

```dockerfile
FROM redis
ENTRYPOINT ["/usr/bin/redis-server"]
```

Si tu lances :

```bash
docker run redis_image
```

➡️ Cela exécute automatiquement `/usr/bin/redis-server`.

***

### 🔹 2. Différence avec `CMD`

* **`CMD`** = commande par défaut, **facile à remplacer** avec des arguments (`docker run image autre_commande`).
* **`ENTRYPOINT`** = commande fixe, **remplaçable uniquement avec `--entrypoint`**.

Exemple avec **ENTRYPOINT + CMD** :

```dockerfile
FROM alpine
ENTRYPOINT ["ping"]
CMD ["localhost"]
```

* `docker run monimage` → fait `ping localhost`
* `docker run monimage google.com` → fait `ping google.com`
* `docker run --entrypoint ls monimage /` → fait `ls /` (plus de `ping`).

***

### 🔹 3. Changer l’ENTRYPOINT avec `--entrypoint`

Tu peux remplacer l’ENTRYPOINT défini dans l’image avec le flag `--entrypoint`.

Exemples :

```bash
docker run -it --entrypoint /bin/bash example/redis
```

➡️ Ouvre un shell Bash **au lieu** de lancer Redis.

Tu peux aussi passer des **arguments supplémentaires** après :

```bash
docker run -it --entrypoint /bin/bash example/redis -c ls -l
```

➡️ Exécute `/bin/bash -c "ls -l"` dans le conteneur.

Autre exemple :

```bash
docker run -it --entrypoint /usr/bin/redis-cli example/redis --help
```

➡️ Lance l’outil client Redis au lieu du serveur.

***

### 🔹 4. Réinitialiser l’ENTRYPOINT

Si tu veux **supprimer l’ENTRYPOINT défini dans l’image**, tu peux passer une chaîne vide :

```bash
docker run -it --entrypoint="" mysql bash
```

➡️ Ici, l’ENTRYPOINT est effacé, et le conteneur exécute seulement `bash`.

⚠️ **Important** : Quand tu mets `--entrypoint=""`, cela **efface aussi le CMD par défaut** de l’image (défini avec `CMD` dans le Dockerfile).

***

✅ **Résumé clair** :

* `ENTRYPOINT` = commande par défaut, fixe, pensée comme **binaire principal** du conteneur.
* `CMD` = arguments par défaut (ou commande si pas d’ENTRYPOINT).
* `--entrypoint` = permet d’écraser ou de vider complètement l’ENTRYPOINT.

## 🌐 **Exposition et publication de ports**

### 🔹 1. Par défaut

Quand tu démarres un conteneur avec `docker run`, **aucun port interne n’est accessible depuis l’hôte**.\
➡️ Même si ton application écoute sur `80` dans le conteneur, tu ne pourras pas y accéder depuis ton navigateur ou ton hôte tant que tu n’as pas publié les ports.

***

### 🔹 2. Rendre les ports accessibles

Il existe deux façons principales :

#### ✅ **Option 1 : `-P` ou `--publish-all`**

* Publie **tous les ports exposés** par le conteneur (via `EXPOSE` dans le Dockerfile ou `--expose`).
* Docker attribue automatiquement des **ports aléatoires** de l’hôte pour chaque port exposé.

Exemple :

```bash
docker run -d -P nginx
```

➡️ Si `nginx` expose `80`, Docker va peut-être le publier sur `32768` côté hôte.\
Tu peux vérifier avec :

```bash
docker port <container_id>
```

***

#### ✅ **Option 2 : `-p` ou `--publish`**

* Permet de **mapper explicitement** un port hôte → port conteneur.
*   Syntaxe :

    ```
    -p [port_hôte]:[port_conteneur]
    ```

Exemples :

```bash
docker run -d -p 8080:80 nginx
```

➡️ Le port `80` dans le conteneur (nginx) est accessible sur `http://localhost:8080`.

Tu peux aussi publier uniquement sur une IP spécifique :

```bash
docker run -d -p 127.0.0.1:8080:80 nginx
```

➡️ Accessible uniquement depuis l’hôte (`localhost`), pas depuis l’extérieur.

Et publier une **plage de ports** :

```bash
docker run -d -p 5000-5010:80 nginx
```

***

### 🔹 3. Exposer vs publier

⚠️ **Attention** :

* `EXPOSE` (dans le Dockerfile) ou `--expose` (dans `docker run`) **ne rend pas le port accessible** depuis l’hôte.\
  C’est juste **une documentation** : ça dit _"ce service utilise tel port"_.
* Pour réellement rendre un port accessible → il faut utiliser `-p` ou `-P`.

***

### 🔹 4. Vérifier les ports publiés

Utilise :

```bash
docker ps
```

ou

```bash
docker port <container_id>
```

Exemple :

```bash
docker ps
CONTAINER ID   IMAGE   PORTS
abc123         nginx   0.0.0.0:8080->80/tcp
```

***

✅ **Résumé rapide :**

* `EXPOSE` = documentation (le port que le conteneur utilise).
* `-P` = publie automatiquement tous les ports exposés sur des ports aléatoires de l’hôte.
* `-p` = publie explicitement un port ou une plage de ports (recommandé en prod).

## 🌍 **Variables d’environnement dans Docker**

### 🔹 1. Variables définies automatiquement

Quand tu crées un **conteneur Linux**, Docker définit certaines variables par défaut :

| Variable     | Valeur                                                                                                   |
| ------------ | -------------------------------------------------------------------------------------------------------- |
| **HOME**     | Défini en fonction de `USER` (souvent `/root`)                                                           |
| **HOSTNAME** | Le nom d’hôte attribué au conteneur                                                                      |
| **PATH**     | Inclut les répertoires système courants (`/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`) |
| **TERM**     | Défini sur `xterm` si le conteneur a un pseudo-TTY                                                       |

⚠️ Sur **Windows containers**, Docker ne définit pas automatiquement de variables spécifiques (elles viennent de Windows).

***

### 🔹 2. Définir des variables avec `-e`

Tu peux définir des variables supplémentaires au moment du `docker run` avec l’option `-e`.

Exemple :

```bash
docker run -e "deep=purple" -e "mode=prod" alpine env
```

Sortie possible :

```
deep=purple
mode=prod
HOME=/root
HOSTNAME=9f12c3d45abc
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

***

### 🔹 3. Propager une variable de l’hôte

Si tu donnes seulement le **nom d’une variable sans valeur**, Docker prend sa valeur depuis ton **hôte**.

Exemple :

```bash
export today=Wednesday
docker run --rm -e today alpine env
```

Sortie :

```
today=Wednesday
```

***

### 🔹 4. Override des valeurs

* Tu peux **écraser** les variables par défaut (ex: `HOME`, `PATH`, etc.).
* Tu peux aussi écraser les variables définies dans le **Dockerfile** via l’instruction `ENV`.

***

### 🔹 5. Fichier d’environnement (`--env-file`)

Pour éviter de mettre toutes les variables à la main avec `-e`, tu peux fournir un fichier `.env` :

📄 `app.env`

```
APP_ENV=production
API_URL=https://api.example.com
SECRET_KEY=supersecret
```

Puis :

```bash
docker run --env-file app.env alpine env
```

***

### 🔹 6. Exemple Windows

Sous **Windows containers**, on peut aussi passer des variables :

```powershell
docker run --rm -e "foo=bar" mcr.microsoft.com/windows/nanoserver cmd /s /c set
```

Sortie (extrait) :

```
foo=bar
USERNAME=ContainerAdministrator
OS=Windows_NT
Path=C:\Windows\system32;C:\Windows
```

***

✅ **Résumé rapide :**

* Docker définit automatiquement certaines variables (HOME, HOSTNAME, PATH, TERM sous Linux).
* `-e VAR=valeur` → ajoute une variable.
* `-e VAR` → propage la valeur depuis l’hôte.
* `--env-file fichier.env` → charger plusieurs variables facilement.
* Tu peux **override** les variables par défaut ou celles définies dans le Dockerfile.

## 👤 **Utilisateur par défaut dans un conteneur**

* Par défaut, **tout conteneur démarre avec l’utilisateur `root`** (`uid=0`).
* Cela signifie que le processus principal (défini par `CMD` ou `ENTRYPOINT`) s’exécute avec **tous les privilèges root** à l’intérieur du conteneur.

⚠️ Même si cet utilisateur est root, il est **isolé du système hôte** par défaut. Cependant, par sécurité, il est recommandé de **ne pas exécuter ses applications en root** dans le conteneur.

***

### 🔹 1. Définir un utilisateur par défaut dans une image

Dans un `Dockerfile`, on peut définir l’utilisateur par défaut avec l’instruction **`USER`** :

```dockerfile
FROM ubuntu:24.04
RUN useradd -ms /bin/bash appuser
USER appuser
CMD ["echo", "Hello depuis appuser !"]
```

➡️ Ici, tout processus du conteneur tournera avec l’utilisateur **`appuser`**.

***

### 🔹 2. Changer l’utilisateur au démarrage (`docker run`)

On peut ignorer le `USER` défini dans le `Dockerfile` avec l’option **`-u`** ou `--user` :

```bash
docker run -u 1000:1000 ubuntu:24.04 whoami
```

👉 Le conteneur s’exécute avec l’UID **1000** et le GID **1000**.

***

### 🔹 3. Syntaxes possibles avec `--user`

L’option accepte plusieurs formats :

* `--user=user` → utilise le nom d’utilisateur existant dans le conteneur
* `--user=user:group` → utilisateur + groupe
* `--user=uid` → identifiant utilisateur numérique
* `--user=uid:gid` → UID + GID numériques
* `--user=user:gid` → utilisateur + GID numérique
* `--user=uid:group` → UID numérique + groupe par nom

Exemple :

```bash
docker run -u appuser:staff ubuntu:24.04 id
```

Résultat :

```
uid=1001(appuser) gid=50(staff) groups=50(staff)
```

***

### 🔹 4. Bonnes pratiques

✅ **Créer un utilisateur non-root** dans l’image avec `RUN useradd`.\
✅ Utiliser `USER` dans le Dockerfile pour définir un utilisateur par défaut.\
✅ Surcharger avec `-u` uniquement pour des cas particuliers (debug, test).

## 📂 **Working Directory dans un conteneur**

### 🔹 1. Répertoire par défaut

* Quand un conteneur démarre, le répertoire courant est **`/`** (la racine du système de fichiers du conteneur).
* Donc si tu lances une commande simple :

```bash
docker run --rm alpine pwd
```

Résultat :

```
/
```

***

### 🔹 2. Définir un répertoire par défaut dans une image

Dans un **Dockerfile**, tu peux définir le répertoire de travail avec l’instruction **`WORKDIR`** :

```dockerfile
FROM alpine:latest

WORKDIR /app

COPY script.sh .
CMD ["sh", "script.sh"]
```

➡️ Ici, quand le conteneur démarre :

* Le répertoire courant est `/app`
* `script.sh` sera exécuté depuis `/app`

***

### 🔹 3. Changer le répertoire au lancement (`docker run`)

Tu peux ignorer le `WORKDIR` défini dans le `Dockerfile` en utilisant **`-w`** ou `--workdir` :

```bash
docker run --rm -w /my/workdir alpine pwd
```

Résultat :

```
/my/workdir
```

👉 Si le dossier **n’existe pas déjà** dans le conteneur, **Docker le crée automatiquement**.

***

### 🔹 4. Exemple pratique

```bash
docker run --rm -it -w /data alpine sh -c "touch test.txt && ls -l"
```

➡️ Cela crée le dossier `/data` dans le conteneur, y place `test.txt`, puis affiche le contenu.

***

✅ **En résumé :**

* Par défaut → `/`
* Dans le Dockerfile → `WORKDIR` fixe un répertoire par défaut
* Au lancement → `-w` peut surcharger ce choix
* Docker crée automatiquement le dossier si nécessaire

