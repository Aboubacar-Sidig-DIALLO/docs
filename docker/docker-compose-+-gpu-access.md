# ⚡️ Docker Compose + GPU Access

### ✅ Prérequis

1. **Pilotes GPU installés sur l’hôte**
   * Pour NVIDIA → NVIDIA drivers + NVIDIA Container Toolkit.
   * Pour d’autres GPU (AMD, Intel), utiliser l’équivalent du runtime compatible.
2. **Docker Daemon configuré**\
   Le runtime NVIDIA doit être disponible (souvent `--gpus` est activé automatiquement après installation du toolkit).
3. **Compose V2** (ou `docker-compose` V1 avec `runtime: nvidia`, mais V1 est obsolète).

***

### 🛠️ Exemple simple avec `device_requests`

Depuis Compose **V2.3+**, on utilise `device_requests` dans la section `deploy.resources` :

```yaml
services:
  gpu-app:
    image: nvidia/cuda:12.2.0-base-ubuntu22.04
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all    # ou un nombre spécifique, ex: 1
              capabilities: [gpu]
```

* `driver: nvidia` → indique d’utiliser le runtime NVIDIA.
* `count: all` → alloue tous les GPU dispo (tu peux mettre `1`, `2`, etc.).
* `capabilities: [gpu]` → indique qu’on veut utiliser les GPU (tu peux ajouter `utility`, `video`, `compute` selon besoins).

***

### 🖥️ Exemple : allouer un seul GPU spécifique

Tu peux demander un GPU particulier via son index :

```yaml
services:
  gpu-app:
    image: nvidia/cuda:12.2.0-runtime-ubuntu22.04
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              device_ids: ["0"]   # GPU n°0 uniquement
              capabilities: [gpu]
```

***

### 🔍 Vérifier dans le conteneur

Une fois le service lancé :

```bash
docker compose up -d
docker compose exec gpu-app nvidia-smi
```

👉 Si tout est bien configuré, tu verras l’état de tes GPU depuis le conteneur.

***

### ⚠️ Attention

* Le champ `deploy` est **normalement utilisé pour Swarm**, mais **`docker compose` (standalone) supporte aussi `device_requests`**.
* Si tu es encore en **Compose V1**, il faut utiliser :

```yaml
services:
  gpu-app:
    image: nvidia/cuda:12.2.0-base-ubuntu22.04
    runtime: nvidia
```

Mais → **Compose V1 est obsolète**, passe à V2.

## ⚡ Utiliser les GPU avec Docker Compose

### 🔑 Règles importantes

* Les GPU s’ajoutent via la section `deploy.resources.reservations.devices`.
* **`capabilities` est obligatoire** → au minimum `[gpu]`.
* Tu dois choisir **soit `count`** (nombre de GPU à réserver), **soit `device_ids`** (cibler un GPU précis). Pas les deux en même temps.
* `driver: nvidia` → requis pour activer le runtime NVIDIA.
* `options` → permet d’ajouter des réglages spécifiques au pilote (cas avancés).

***

### ✅ Exemple 1 : Service avec 1 GPU

```yaml
services:
  test:
    image: nvidia/cuda:12.9.0-base-ubuntu22.04
    command: nvidia-smi
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
```

👉 Lancer avec :

```bash
docker compose up
```

➡️ Le conteneur exécutera `nvidia-smi` et affichera les informations de ton GPU.

***

### ✅ Exemple 2 : Cibler un GPU précis

```yaml
services:
  gpu-service:
    image: nvidia/cuda:12.9.0-runtime-ubuntu22.04
    command: nvidia-smi
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              device_ids: ["2"]   # Utilise uniquement le GPU n°2
              capabilities: [gpu]
```

***

### ✅ Exemple 3 : Utiliser tous les GPU disponibles

```yaml
services:
  gpu-all:
    image: nvidia/cuda:12.9.0-base-ubuntu22.04
    command: nvidia-smi
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

***

### 🔍 Vérifier les GPU disponibles

Sur ta machine hôte (avant Compose) :

```bash
nvidia-smi
```

👉 Tu verras la liste des GPU avec leur **ID** (0,1,2,3, …).\
Ces IDs peuvent être utilisés avec `device_ids`.

***

### ⚠️ Erreurs courantes

* ❌ Oublier `capabilities: [gpu]` → erreur au déploiement.
* ❌ Utiliser `count` **et** `device_ids` en même temps → erreur.
* ❌ Demander plus de GPU que la machine n’en possède → erreur immédiate.

## 🎯 Accès à des GPU spécifiques avec Docker Compose

Si ta machine possède plusieurs GPU (ex. 0, 1, 2, 3), tu peux limiter un service pour qu’il n’utilise **que certains d’entre eux**.

👉 Exemple : donner uniquement accès à **GPU-0 et GPU-3** :

```yaml
services:
  test:
    image: tensorflow/tensorflow:latest-gpu
    command: python -c "import tensorflow as tf; print(tf.test.gpu_device_name())"
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              device_ids: ['0', '3']
              capabilities: [gpu]
```

***

### 🔍 Explication

* `driver: nvidia` → indispensable pour utiliser le runtime NVIDIA.
* `device_ids: ['0', '3']` → seuls les GPU **0 et 3** seront visibles par le conteneur.
* `capabilities: [gpu]` → obligatoire (sinon Docker Compose renvoie une erreur).
* `command: ...` → ici on vérifie dans TensorFlow que le GPU est bien accessible.

***

### ✅ Vérification

Avant de lancer ton service, regarde tes GPU disponibles sur l’hôte avec :

```bash
nvidia-smi
```

Tu verras une liste comme :

```
+-----------------------------------------------------------------------------+
| GPU   Name                 Persistence-M| Bus-Id        Disp.A | Volatile ECC |
|  0   Tesla T4                        On | 00000000:00:1B.0 Off | 0            |
|  1   Tesla T4                        On | 00000000:00:1C.0 Off | 0            |
|  2   Tesla T4                        On | 00000000:00:1D.0 Off | 0            |
|  3   Tesla T4                        On | 00000000:00:1E.0 Off | 0            |
+-----------------------------------------------------------------------------+
```

👉 Ici, si tu choisis `['0','3']`, seul le **premier et dernier GPU** seront visibles dans le conteneur.

***

### ⚠️ Attention

* Tu ne peux **pas mélanger `count` et `device_ids`** dans la même config.
* Si tu mets un ID qui n’existe pas (ex. `5` alors que tu n’as que 4 GPU), Docker retournera une erreur.
* Si tu ne précises pas `device_ids`, **tous les GPU disponibles** seront exposés par défaut.
