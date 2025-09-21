# 📂 Bind Mounts dans Docker

### 🔹 Qu’est-ce qu’un Bind Mount ?

* Un **bind mount** relie directement un dossier ou un fichier de ta machine hôte dans un conteneur.
* Contrairement aux **volumes** (stockés et gérés par Docker dans `/var/lib/docker/volumes/`), ici Docker **ne gère rien** : il pointe vers ton vrai dossier/fichier sur le disque hôte.

👉 Exemple : tu veux partager ton **code source** entre ton PC et ton conteneur, pour que les modifications locales soient visibles dans le conteneur en temps réel.

***

### 🔹 Quand utiliser les Bind Mounts ?

✔️ Cas typiques :

* **Développement** → partager le code source entre ta machine et le conteneur.
* **Configuration** → monter un fichier de conf depuis l’hôte (`/etc/nginx/nginx.conf`) dans le conteneur.
* **Build/CI** → partager des artefacts ou dépendances (ex: un dossier `target/` avec un projet Java/Maven).
* **Logs** → écrire les logs du conteneur directement dans un répertoire lisible sur l’hôte.

❌ Cas où il vaut mieux **éviter** :

* En production (manque de portabilité, dépendance forte au chemin hôte).
* Si tu veux que Docker gère la persistance (préférer les volumes).

***

### 🔹 Considérations importantes

⚠️ Points à garder en tête :

* Accès **en écriture** par défaut → une app dans le conteneur peut modifier/supprimer tes fichiers hôte.
*   Tu peux limiter à **lecture seule** avec `ro` :

    ```bash
    docker run -v $(pwd):/app:ro nginx
    ```
* **Obscurcissement des fichiers existants** : si tu montes un bind sur un dossier déjà rempli du conteneur, son contenu est caché (comme un montage de clé USB sur `/mnt`).
* Les conteneurs deviennent **fortement liés** à la structure de ton hôte → pas portable entre machines.

***

### 🔹 Syntaxes disponibles

Deux façons de déclarer un bind mount :

#### 1. Avec `--mount` (recommandé)

Plus explicite, supporte toutes les options :

```bash
docker run --mount type=bind,src="$(pwd)"/target,dst=/app nginx:latest
```

#### 2. Avec `-v` ou `--volume`

Plus court, mais moins clair :

```bash
docker run -v $(pwd)/target:/app nginx:latest
```

👉 Ces deux commandes montent ton dossier `target/` dans `/app` du conteneur.

***

### 🔹 Options utiles

Avec `--mount` :

* `type=bind` → indique un bind mount.
* `src=<chemin hôte>` → chemin sur ton PC (relatif ou absolu).
* `dst=<chemin conteneur>` → où monter dans le conteneur.
* `readonly` ou `ro` → monte en lecture seule.
* `bind-propagation` → contrôle la propagation (avancé : `rprivate`, `shared`, etc.).

👉 Exemple complet :

```bash
docker run --mount type=bind,src=.,dst=/project,ro,bind-propagation=rshared nginx:latest
```

Avec `-v` :

```bash
docker run -v .:/project:ro,rshared nginx:latest
```

***

### 🔹 Exemple pratique

Tu développes une app Node.js, tu veux que les fichiers locaux soient accessibles dans ton conteneur pour éviter de reconstruire l’image à chaque fois :

```bash
docker run -it --name dev-node \
  -v $(pwd):/usr/src/app \
  -w /usr/src/app \
  node:lts bash
```

👉 Ici :

* `-v $(pwd):/usr/src/app` → ton dossier courant est monté dans `/usr/src/app`.
* Tu peux lancer `npm install` et `npm start` dans le conteneur, mais voir les changements en direct depuis ton éditeur sur l’hôte.

***

### 🔹 Exemple de piège ⚠️

Si tu montes `/tmp` de ton hôte sur `/usr` dans le conteneur :

```bash
docker run -d --name broken \
  --mount type=bind,src=/tmp,dst=/usr nginx:latest
```

👉 Résultat : les fichiers `/usr` du conteneur sont cachés → `nginx` ne démarre pas.

## 📂 Bind Mounts avancés

### 🔹 1. Bind mount en lecture seule (`ro`)

Par défaut, un bind mount est **en lecture/écriture** (RW).\
Tu peux le rendre **lecture seule** (RO) pour éviter que ton conteneur ne modifie les fichiers de l’hôte.

👉 Exemple avec `--mount` :

```bash
docker run -d -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app,readonly \
  nginx:latest
```

👉 Exemple avec `-v` :

```bash
docker run -d -it \
  --name devtest \
  -v "$(pwd)"/target:/app:ro \
  nginx:latest
```

Vérification avec `docker inspect devtest` :

```json
"Mounts": [
  {
    "Type": "bind",
    "Source": "/tmp/source/target",
    "Destination": "/app",
    "Mode": "ro",
    "RW": false,
    "Propagation": "rprivate"
  }
]
```

✅ Résultat : ton conteneur **voit les fichiers**, mais ne peut pas les modifier.

***

### 🔹 2. Recursive mounts (`bind-recursive`)

Si le chemin monté contient déjà d’autres **montages (submounts)**, ils sont aussi montés dans le conteneur.

⚙️ Options possibles (`--mount`) :

* `enabled` (défaut) → submounts inclus, si kernel ≥ 5.12 alors ils deviennent aussi RO si le parent est RO.
* `disabled` → submounts ignorés.
* `writable` → submounts RW même si parent RO.
* `readonly` → submounts RO (nécessite kernel ≥ 5.12).

👉 Exemple :

```bash
docker run -d -it \
  --name devtest \
  --mount type=bind,src=/data,dst=/mnt,bind-recursive=readonly \
  alpine
```

***

### 🔹 3. Bind propagation

La **propagation** détermine si les montages faits **à l’intérieur** du bind mount apparaissent aussi ailleurs.

| Option     | Description                                                      |
| ---------- | ---------------------------------------------------------------- |
| `private`  | Rien n’est partagé (isolé).                                      |
| `rprivate` | (défaut) Comme `private`, mais appliqué récursivement.           |
| `shared`   | Les submounts sont visibles dans toutes les répliques.           |
| `rshared`  | Idem `shared`, mais récursif.                                    |
| `slave`    | Submounts visibles dans un sens seulement (original → réplique). |
| `rslave`   | Idem `slave`, mais récursif.                                     |

👉 Exemple : deux montages avec propagation différente

```bash
docker run -d -it \
  --name devtest \
  --mount type=bind,src="$(pwd)"/target,dst=/app \
  --mount type=bind,src="$(pwd)"/target,dst=/app2,readonly,bind-propagation=rslave \
  nginx:latest
```

Si tu crées `/app/foo/`, alors `/app2/foo/` existe aussi.

⚠️ Attention : ça **ne marche pas sur Docker Desktop** (car ça repose sur des mécanismes du kernel Linux).

***

### 🔹 4. SELinux et bind mounts

Si tu es sur une machine avec **SELinux** activé (ex: Fedora, RHEL, CentOS), tu dois configurer les **labels de sécurité**.\
Options :

* `:z` → le contenu est **partageable** entre plusieurs conteneurs.
* `:Z` → le contenu est **privé** à ce conteneur uniquement.

👉 Exemple :

```bash
docker run -d -it \
  --name devtest \
  -v "$(pwd)"/target:/app:z \
  nginx:latest
```

⚠️ Danger : si tu montes `/home` ou `/usr` avec `:Z`, tu risques de rendre ton host inutilisable (permissions corrompues).

***

### 🔹 5. Bind mounts avec Docker Compose

Avec **Compose**, tu peux définir un bind mount comme ceci :

```yaml
services:
  frontend:
    image: node:lts
    volumes:
      - type: bind
        source: ./static
        target: /opt/app/static
```

👉 Ici :

* `./static` sur ton hôte est monté dans `/opt/app/static` dans le conteneur.
