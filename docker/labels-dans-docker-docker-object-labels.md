# 🐳 Labels dans Docker (Docker object labels)

Les **labels** dans Docker sont un mécanisme pour attacher des **métadonnées** aux objets Docker.\
Ils sont composés de paires **clé=valeur** et permettent de documenter, organiser ou automatiser la gestion des ressources Docker.

***

### 🔹 Objets Docker qui supportent les labels

Tu peux appliquer des labels à plusieurs types d’objets :

* **Images**
* **Containers**
* **Volumes**
* **Networks**
* **Daemons locaux**
* **Nœuds Swarm**
* **Services Swarm**

***

### 🔹 À quoi servent les labels ?

Les labels sont **libres et personnalisables**, tu peux les utiliser par exemple pour :

* Organiser tes images (ex. `maintainer=dev-team`)
* Documenter des informations de licence ou d’usage (`license=GPL`)
* Décrire des relations entre conteneurs, volumes et réseaux (`app=frontend`)
* Catégoriser des environnements (`env=production`, `env=staging`)
* Gérer les ressources dans Swarm (ex. placement des services avec `node.labels`)

***

### 🔹 Exemple d’utilisation

#### Sur une image

```bash
docker build -t myapp:1.0 \
  --label maintainer="admin@example.com" \
  --label version="1.0" .
```

#### Sur un conteneur

```bash
docker run -d \
  --label app=frontend \
  --label env=production \
  nginx
```

#### Sur un volume

```bash
docker volume create \
  --label project=myapp \
  myvolume
```

#### Sur un réseau

```bash
docker network create \
  --label team=backend \
  mynetwork
```

***

### 🔹 Lister les labels

Tu peux inspecter les labels avec :

```bash
docker inspect <objet>
```

Exemple pour un conteneur :

```bash
docker inspect nginx | grep Labels -A 5
```

***

👉 Les labels sont aussi très utiles pour **filtrer des commandes** avec `--filter "label=clé=valeur"`.

## 🐳 Clés et valeurs des labels Docker

Un **label** est toujours une **paire clé=valeur**, stockée sous forme de chaîne de caractères.\
👉 Tu peux associer **plusieurs labels** à un même objet Docker (image, conteneur, volume, etc.), mais **chaque clé doit être unique** par objet.\
⚠️ Si la même clé est utilisée plusieurs fois, **la dernière valeur écrasera les précédentes**.

***

### 🔹 Règles pour les clés de labels (label keys)

La **clé** est la partie gauche de la paire clé=valeur.

* Doit être **alphanumérique**.
* Peut contenir :
  * des **points** (`.`)
  * des **underscores** (`_`)
  * des **slashes** (`/`)
  * des **tirets** (`-`)

#### ✅ Bonnes pratiques :

1.  **Utiliser un préfixe basé sur ton domaine** (notation DNS inversée) pour éviter les collisions.\
    Exemple :

    * `com.example.env`
    * `org.mycompany.project`

    ⚠️ **Ne pas utiliser un domaine qui ne t’appartient pas**.
2. Les préfixes **réservés à Docker** :
   * `com.docker.*`
   * `io.docker.*`
   * `org.dockerproject.*`\
     → 🚫 ne pas utiliser.
3. Format recommandé :
   * Commencer et finir par une **lettre minuscule**.
   * Contenir uniquement : lettres minuscules, chiffres, points (`.`), tirets (`-`).
   * ❌ Pas de points ou tirets consécutifs (`..` ou `--`).
4. Les clés **sans namespace** (ex. `env`, `version`) sont réservées pour un usage rapide en CLI.

***

### 🔹 Règles pour les valeurs de labels (label values)

* Une **valeur** peut contenir **n’importe quel type de donnée converti en string** :
  * Texte simple (`production`, `frontend`)
  * JSON (`{"role":"backend"}`)
  * XML, CSV, YAML, etc.
* Docker **ne désérialise pas** la valeur.
  * Donc un label avec du JSON est juste une chaîne, pas une structure exploitable directement.
  * Si tu veux filtrer ou manipuler ce genre de labels complexes, il faut passer par des outils tiers.

***

### 🔹 Exemple pratique

```bash
docker run -d \
  --label env=production \
  --label version=1.0 \
  --label com.example.team=backend \
  --label com.example.metadata='{"owner":"alice","priority":"high"}' \
  nginx
```

👉 Ici :

* `env=production` et `version=1.0` sont de petits labels CLI-friendly.
* `com.example.team=backend` suit la bonne pratique avec DNS inversé.
* `com.example.metadata='{"owner":"alice","priority":"high"}'` montre qu’on peut stocker du JSON en valeur.

## 🐳 Gestion des labels sur les objets Docker

Chaque type d’objet Docker (images, conteneurs, volumes, réseaux, démons, nœuds Swarm, services) prend en charge les **labels**, mais la façon de les gérer dépend de l’objet.

⚠️ Important :

* Les labels des **images, conteneurs, démons locaux, volumes et réseaux** sont **statiques** → tu dois **recréer l’objet** si tu veux changer ses labels.
* Les labels des **nœuds et services Swarm** sont **dynamiques** → tu peux les mettre à jour sans recréer.

***

### 🔹 Images et conteneurs

#### ➕ Ajouter des labels à une image

Dans un `Dockerfile` :

```dockerfile
FROM nginx
LABEL maintainer="admin@example.com" \
      version="1.0" \
      env="production"
```

Puis construire l’image :

```bash
docker build -t my-nginx .
```

#### 🔄 Redéfinir les labels d’un conteneur au runtime

```bash
docker run -d --label env=dev --label version=2.0 nginx
```

#### 🔍 Inspecter les labels

```bash
docker inspect my-nginx | jq '.[0].Config.Labels'
```

#### 🔎 Filtrer par labels

```bash
docker images --filter "label=version=1.0"
docker ps --filter "label=env=dev"
```

***

### 🔹 Démon Docker local

#### ➕ Ajouter des labels

Dans `/etc/docker/daemon.json` :

```json
{
  "labels": ["env=production", "region=eu"]
}
```

Puis redémarrer Docker :

```bash
sudo systemctl restart docker
```

#### 🔍 Inspecter les labels du démon

```bash
docker info --format '{{json .Labels}}'
```

***

### 🔹 Volumes

#### ➕ Ajouter des labels

```bash
docker volume create --label env=prod --label team=backend myvolume
```

#### 🔍 Inspecter les labels

```bash
docker volume inspect myvolume
```

#### 🔎 Filtrer

```bash
docker volume ls --filter "label=env=prod"
```

***

### 🔹 Réseaux

#### ➕ Ajouter des labels

```bash
docker network create --label env=prod --label app=web mynet
```

#### 🔍 Inspecter

```bash
docker network inspect mynet
```

#### 🔎 Filtrer

```bash
docker network ls --filter "label=app=web"
```

***

### 🔹 Nœuds Swarm

#### ➕ Ajouter ou mettre à jour des labels

```bash
docker node update --label-add region=us-east worker1
```

#### 🔍 Inspecter

```bash
docker node inspect worker1 | jq '.[0].Spec.Labels'
```

#### 🔎 Filtrer

```bash
docker node ls --filter "label=region=us-east"
```

***

### 🔹 Services Swarm

#### ➕ Ajouter des labels à la création

```bash
docker service create --name web --label env=prod nginx
```

#### 🔄 Mettre à jour

```bash
docker service update --label-add version=2.0 web
```

#### 🔍 Inspecter

```bash
docker service inspect web | jq '.[0].Spec.Labels'
```

#### 🔎 Filtrer

```bash
docker service ls --filter "label=env=prod"
```
