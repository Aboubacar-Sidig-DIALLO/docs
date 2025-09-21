# 📦 Volumes dans Docker

### 1. Définition

Un **volume** est un espace de stockage **persistant** géré par Docker.\
Contrairement aux **bind mounts** (qui dépendent du système de fichiers de l’hôte), les volumes sont :

* **isolés de l’OS de l’hôte**,
* **gérés directement par Docker**,
* **plus portables et faciles à sauvegarder**.

👉 Les volumes permettent donc de stocker les données même si un conteneur est supprimé.

***

### 2. Pourquoi utiliser des volumes ?

Les volumes sont **le mécanisme recommandé** pour la persistance de données dans Docker car :

* ✅ Plus faciles à sauvegarder et migrer
* ✅ Gérés par Docker CLI/API (`docker volume create`, `docker volume ls`, etc.)
* ✅ Fonctionnent sur **Linux et Windows**
* ✅ Peuvent être partagés en toute sécurité entre plusieurs conteneurs
* ✅ Possibilité de **pré-peupler** un volume (par copie des fichiers existants ou via une image)
* ✅ Performances supérieures à l’écriture dans la couche "writable" du conteneur

👉 Utilise un **bind mount** plutôt qu’un volume uniquement si tu veux accéder directement aux fichiers sur l’hôte.

***

### 3. Cycle de vie

* Un volume **vit indépendamment des conteneurs**.
* Même si un conteneur est supprimé, le volume reste disponible.
*   On peut supprimer les volumes inutilisés avec :

    ```bash
    docker volume prune
    ```
* Un même volume peut être **monté dans plusieurs conteneurs**.

***

### 4. Montage d’un volume

Deux syntaxes possibles :

#### a) Avec `--mount` (plus explicite, recommandé) :

```bash
docker run --mount type=volume,src=monvolume,dst=/data nginx
```

#### b) Avec `-v` ou `--volume` (plus court) :

```bash
docker run -v monvolume:/data nginx
```

***

### 5. Options principales de `--mount`

| Option             | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| `src` ou `source`  | Nom du volume (si nommé).                                    |
| `dst` ou `target`  | Dossier de destination dans le conteneur.                    |
| `readonly` ou `ro` | Montage en lecture seule.                                    |
| `volume-nocopy`    | Empêche la copie auto des fichiers existants vers le volume. |
| `volume-subpath`   | Permet de monter un sous-dossier spécifique du volume.       |
| `volume-opt`       | Ajout d’options pour le driver de volume.                    |

👉 Exemple avec plusieurs options :

```bash
docker run --mount type=volume,src=myvolume,dst=/data,ro,volume-subpath=/foo
```

***

### 6. Volumes nommés vs anonymes

*   **Només** : créés explicitement → plus facile à partager et réutiliser.

    ```bash
    docker volume create monvolume
    docker run -v monvolume:/data nginx
    ```
* **Anonymes** : créés automatiquement avec un ID aléatoire.\
  Ils restent persistants sauf si on lance le conteneur avec `--rm` (alors ils sont supprimés en même temps).

***

### 7. Cas particuliers

* **Écrasement de fichiers existants** :\
  Si tu montes un volume **non vide** dans un dossier qui contient déjà des fichiers, ces fichiers sont **masqués** par le contenu du volume.
* **Pré-population** :\
  Si le volume est vide, les fichiers du conteneur à ce chemin sont copiés dans le volume par défaut (sauf si `volume-nocopy` est utilisé).
* **Données temporaires** :\
  Utilise un **tmpfs** si tu veux garder les données uniquement en mémoire (plus rapide et non persistant).

👉 Exemple tmpfs :

```bash
docker run --mount type=tmpfs,dst=/tmp tmp-container
```

## 📦 Créer et gérer des volumes dans Docker

### 1. Gérer un volume indépendamment d’un conteneur

Contrairement aux **bind mounts**, les **volumes** peuvent être créés et administrés **hors du cycle de vie d’un conteneur**.

#### 🔹 Créer un volume

```bash
docker volume create my-vol
```

#### 🔹 Lister les volumes

```bash
docker volume ls
```

Exemple de sortie :

```
local               my-vol
```

#### 🔹 Inspecter un volume

```bash
docker volume inspect my-vol
```

Exemple de résultat JSON :

```json
{
  "Driver": "local",
  "Labels": {},
  "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
  "Name": "my-vol",
  "Options": {},
  "Scope": "local"
}
```

#### 🔹 Supprimer un volume

```bash
docker volume rm my-vol
```

***

### 2. Utiliser un volume dans un conteneur

Si tu montes un volume qui n’existe pas encore, **Docker le crée automatiquement**.

#### Exemple avec `--mount`

```bash
docker run -d \
  --name devtest \
  --mount source=myvol2,target=/app \
  nginx:latest
```

#### Exemple équivalent avec `-v`

```bash
docker run -d \
  --name devtest \
  -v myvol2:/app \
  nginx:latest
```

👉 Vérification avec :

```bash
docker inspect devtest
```

Regarde la section **Mounts** pour confirmer que le volume est bien monté.

#### Nettoyage :

```bash
docker container stop devtest
docker container rm devtest
docker volume rm myvol2
```

***

### 3. Utiliser des volumes avec Docker Compose

```yaml
services:
  frontend:
    image: node:lts
    volumes:
      - myapp:/home/node/app

volumes:
  myapp:
```

➡️ Lors du premier `docker compose up`, Docker crée `myapp`.\
➡️ Il est réutilisé automatiquement ensuite.

Si tu veux utiliser un volume créé **à l’avance** :

```yaml
volumes:
  myapp:
    external: true
```

***

### 4. Volumes avec les services (Swarm)

Chaque **réplica** d’un service aura son propre volume local (sauf si tu utilises un driver de stockage partagé).

Exemple :

```bash
docker service create -d \
  --replicas=4 \
  --name devtest-service \
  --mount source=myvol2,target=/app \
  nginx:latest
```

***

### 5. Pré-remplir un volume avec un conteneur

Docker copie les fichiers existants du conteneur dans le volume lors de son premier montage.

Exemple :

```bash
docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html \
  nginx:latest
```

👉 Ici, le volume `nginx-vol` contient par défaut les fichiers HTML de Nginx.

***

### 6. Volume en lecture seule

Utile pour partager un volume entre plusieurs conteneurs (certains en lecture seule, d’autres en lecture/écriture).

```bash
docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html,readonly \
  nginx:latest
```

Vérifie avec :

```bash
docker inspect nginxtest
```

👉 La section **Mounts → RW** doit être à `false`.

***

### 7. Monter un **sous-dossier** d’un volume

Tu peux utiliser `volume-subpath` pour ne monter qu’un sous-dossier d’un volume.

Exemple :

```bash
docker volume create logs
docker run --rm --mount src=logs,dst=/logs alpine mkdir -p /logs/app1 /logs/app2

docker run -d \
  --name=app1 \
  --mount src=logs,dst=/var/log/app1,volume-subpath=app1 \
  app1:latest

docker run -d \
  --name=app2 \
  --mount src=logs,dst=/var/log/app2,volume-subpath=app2 \
  app2:latest
```

👉 Ici, `app1` et `app2` écrivent dans **des sous-dossiers différents** du même volume `logs`.\
Ils ne peuvent pas accéder aux logs de l’autre.

## 📦 Partager des données entre machines avec Docker

### 1. Pourquoi partager des volumes ?

Quand tu déploies une app **tolérante aux pannes** avec plusieurs réplicas (scalabilité), il faut que **toutes les instances accèdent aux mêmes fichiers/données**.\
Exemples :

* Plusieurs containers d’une même API accèdent à un répertoire partagé.
* Une app web partage ses uploads/images entre plusieurs serveurs.
* Une base ou un cache utilise un volume commun (ex: NFS, S3).

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

***

### 2. Solutions possibles

#### 🔹 a) Stockage objet externe (S3, MinIO, etc.)

👉 Tu ajoutes la logique directement dans l’application (upload → S3).\
✔️ Robuste et simple à scaler.\
❌ Nécessite que l’app sache gérer ce stockage.

#### 🔹 b) Volumes avec un driver externe

👉 Tu montes un volume basé sur un système partagé comme **NFS, CIFS/Samba ou S3** via un plugin Docker.\
✔️ Transparent pour l’app.\
✔️ Tu peux changer de driver sans modifier le code.

***

### 3. Utiliser un **volume driver**

#### Exemple avec **rclone** (SFTP distant)

Installer le plugin :

```bash
docker plugin install --grant-all-permissions rclone/docker-volume-rclone --aliases rclone
```

Créer un volume :

```bash
docker volume create \
  -d rclone \
  --name rclonevolume \
  -o type=sftp \
  -o path=remote \
  -o sftp-host=1.2.3.4 \
  -o sftp-user=user \
  -o "sftp-password=$(cat ./password.txt)"
```

Démarrer un conteneur en utilisant ce volume :

```bash
docker run -d \
  --name rclone-container \
  --mount type=volume,volume-driver=rclone,src=rclonevolume,target=/app \
  nginx:latest
```

***

### 4. Exemple avec **NFS**

Deux variantes selon la version :

#### 🔹 NFSv3

```bash
docker service create -d \
  --name nfs-service \
  --mount 'type=volume,source=nfsvolume,target=/app,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/var/docker-nfs,volume-opt=o=addr=10.0.0.10' \
  nginx:latest
```

#### 🔹 NFSv4

```bash
docker service create -d \
  --name nfs-service \
  --mount 'type=volume,source=nfsvolume,target=/app,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/var/docker-nfs,"volume-opt=o=addr=10.0.0.10,rw,nfsvers=4,async"' \
  nginx:latest
```

***

### 5. Exemple avec **CIFS/Samba**

```bash
docker volume create \
  --driver local \
  --opt type=cifs \
  --opt device=//monserveur/backup \
  --opt o=addr=monserveur,username=user,password=motdepasse,file_mode=0777,dir_mode=0777 \
  --name cifs-volume
```

***

### 6. Utiliser un **bloc device**

👉 Tu peux monter directement un disque ou partition (⚠️ avancé/réservé aux experts).

Exemple :

```bash
docker run -it --rm \
  --mount='type=volume,dst=/external-drive,volume-driver=local,volume-opt=device=/dev/loop5,volume-opt=type=ext4' \
  ubuntu bash
```

***

### 7. Sauvegarder et restaurer un volume

#### 🔹 Sauvegarde

```bash
docker run -v /dbdata --name dbstore ubuntu /bin/bash

docker run --rm --volumes-from dbstore -v $(pwd):/backup ubuntu \
  tar cvf /backup/backup.tar /dbdata
```

#### 🔹 Restauration

```bash
docker run -v /dbdata --name dbstore2 ubuntu /bin/bash

docker run --rm --volumes-from dbstore2 -v $(pwd):/backup ubuntu \
  bash -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"
```

***

### 8. Nettoyage des volumes

* Supprimer un volume anonyme avec un conteneur :

```bash
docker run --rm -v /foo busybox top
```

→ Ici `/foo` est supprimé à la fin.

* Supprimer tous les volumes inutilisés :

```bash
docker volume prune
```
