---
description: >-
  C’est eux qui expliquent comment Docker stocke les images, les couches et le
  writable layer des conteneurs.
---

# 👉 les storage drivers (pilotes de stockage).

## 🚀 1. Rôle des Storage Drivers

* Docker stocke les **images** et les **conteneurs** sous forme de **couches (layers)**.
* Chaque **image** = plusieurs **layers en lecture seule (read-only)**.
* Chaque **conteneur** = image + **un layer en écriture (writable layer)** au-dessus.

⚡ **Important :**

* Le writable layer **disparaît** quand tu supprimes le conteneur.
* Si tu veux **persister les données** : → utilise **Docker Volumes** (beaucoup plus adaptés).

***

## 🧱 2. Images et couches

👉 Exemple Dockerfile :

```dockerfile
FROM ubuntu:22.04
LABEL org.opencontainers.image.authors="org@example.com"
COPY . /app
RUN make /app
RUN rm -r $HOME/.cache
CMD python /app/app.py
```

#### Découpage en couches :

1. `FROM ubuntu:22.04` → crée une couche de base.
2. `LABEL` → modifie uniquement les métadonnées (pas de nouvelle couche).
3. `COPY . /app` → ajoute une couche avec les fichiers copiés.
4. `RUN make /app` → ajoute une couche avec le résultat de la compilation.
5. `RUN rm -r $HOME/.cache` → ajoute une nouvelle couche (⚠️ même si ça supprime, la donnée existe toujours dans une couche précédente).
6. `CMD` → métadonnée uniquement, pas de couche.

***

## 🏗️ 3. Comment ça marche en pratique

### 🔹 Structure d’une image et conteneur

```
           ┌─────────────────────────┐
           │ Layer 3 : RUN rm cache  │ (read-only)
           ├─────────────────────────┤
           │ Layer 2 : RUN make app  │ (read-only)
           ├─────────────────────────┤
           │ Layer 1 : COPY sources  │ (read-only)
           ├─────────────────────────┤
           │ Base : Ubuntu 22.04     │ (read-only)
           └─────────────────────────┘
                       ▲
                       │
           ┌─────────────────────────┐
           │ Writable container layer │ (modifiable)
           └─────────────────────────┘
```

➡️ Quand tu lances un conteneur, Docker ajoute un **fine writable layer** au-dessus.\
➡️ Toutes les modifications sont stockées uniquement dans ce dernier.

***

## 🧩 4. Partage des couches

* Plusieurs conteneurs peuvent **partager les mêmes couches read-only**.
* Chaque conteneur n’a que **son layer writable unique**.

#### Illustration :

```
Image Ubuntu:15.04
    ├── Layer base
    ├── Layer apt install ...
    └── Layer app install

Conteneur A → + Writable layer A  
Conteneur B → + Writable layer B  
Conteneur C → + Writable layer C
```

👉 Les 3 conteneurs partagent **les mêmes couches de l’image** (gain de place).\
👉 Seul leur **layer writable** est unique.

***

## ⚙️ 5. Copy-on-Write (CoW)

Le principe :

* Si un fichier existe déjà dans une couche read-only → le conteneur le lit directement.
* Si le conteneur veut le modifier → Docker le **copie** dans le writable layer, puis le modifie.

Exemple :

1. L’image a `/etc/config.conf` en read-only.
2. Le conteneur le modifie → **copie dans layer writable**.
3. Seule cette copie est modifiée, le fichier original reste intact.

#### Illustration :

```
Lecture :
  Container → utilise fichier /etc/config.conf depuis Layer 2

Écriture :
  Container → copie le fichier dans Writable layer
            → modifie la copie locale
```

⚠️ Effet secondaire :

* Beaucoup de modifications = beaucoup de **copies coûteuses**.
* → c’est pour ça que Docker recommande d’utiliser **Volumes** pour bases de données et fichiers lourds.

***

## 📊 6. Taille des conteneurs

Commande :

```bash
docker ps -s
```

Affiche :

* **Size** → taille de la couche writable du conteneur.
* **Virtual size** → taille de l’image + writable layer.

👉 Si tu crées 5 conteneurs depuis la même image :

* Ils partagent les **mêmes layers read-only**.
* Leur **layer writable** fait 0B tant qu’ils n’écrivent rien.

***

## 🧰 7. Exemple concret

#### Créer plusieurs conteneurs :

```bash
docker run -dit --name c1 acme/my-final-image:1.0 bash
docker run -dit --name c2 acme/my-final-image:1.0 bash
```

#### Vérifier :

```bash
docker ps -s
```

```
CONTAINER ID   IMAGE   SIZE
c1             myimg   0B (virtual 7.7MB)
c2             myimg   0B (virtual 7.7MB)
```

👉 Les 2 conteneurs ne consomment presque rien (juste leur config), car ils partagent l’image.

#### Écrire dans un conteneur :

```bash
docker exec c1 sh -c 'echo hello > /test.txt'
```

Puis :

```bash
docker ps -s
```

```
c1   myimg   5B (virtual 7.7MB)
c2   myimg   0B (virtual 7.7MB)
```

👉 Seul **c1** a grossi car il a copié et modifié un fichier.

***

## ✅ Résumé

* **Storage drivers** = gestion des layers et du writable layer.
* **Copy-on-Write** = optimisation : copier un fichier seulement au premier write.
* **Images** = couches read-only partagées.
* **Conteneurs** = ajout d’un writable layer unique.
* **Volumes** = préférables pour les données persistantes et write-heavy.
