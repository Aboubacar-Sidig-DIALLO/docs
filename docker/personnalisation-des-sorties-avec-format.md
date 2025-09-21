# 🎨 Personnalisation des sorties avec --format

Docker permet d’utiliser les **templates Go** pour formater la sortie de certaines commandes (`docker ps`, `docker images`, `docker inspect`, etc.).

Cela sert à afficher **uniquement les champs qui t’intéressent**, au lieu du tableau par défaut.

***

### ⚙️ Syntaxe générale

```bash
docker COMMAND --format '{{TEMPLATE}}'
```

* `TEMPLATE` est une expression Go qui référence un champ de la structure de sortie.
* Les champs accessibles dépendent de la commande (ex: `.ID`, `.Names`, `.Image`, `.Status`, …).

***

### 🔹 Exemple simple

Afficher seulement les **IDs des conteneurs** actifs :

```bash
docker ps --format '{{.ID}}'
```

Résultat :

```
c3f279d17e0a
aed84ee21bde
```

***

### 🔹 Exemple avec plusieurs champs

Afficher **nom et image** des conteneurs :

```bash
docker ps --format '{{.Names}} -> {{.Image}}'
```

Résultat :

```
web -> nginx:alpine
db -> postgres:15
```

***

### 🔹 Exemple avec `docker inspect`

Afficher les arguments du process dans le conteneur :

```bash
docker inspect --format '{{join .Args " , "}}' mycontainer
```

👉 Attention :

* Sous Linux/macOS : `'...'` fonctionne.
*   Sous PowerShell : il faut échapper `"` :

    ```powershell
    docker inspect --format '{{join .Args \" , \"}}' mycontainer
    ```

***

### 🔹 Fonctions disponibles

Docker inclut quelques fonctions utiles dans les templates Go :

* `join` → concatène une liste (ex: `{{join .Args ", "}}`)
* `json` → renvoie un objet JSON (ex: `{{json .Config}}`)
* `lower` / `upper` → met en minuscule ou majuscule
* `truncate` → tronque une chaîne
* `println` → ajoute un retour à la ligne
* `split` → coupe une chaîne en tableau

***

### 🔹 Exemple pratique avec logs

Format JSON lisible :

```bash
docker inspect --format '{{json .NetworkSettings.Networks}}' mycontainer
```

Exemple d’affichage :

```json
{
  "bridge": {
    "IPAMConfig": null,
    "Links": null,
    "Aliases": ["mycontainer"],
    "NetworkID": "c8b2f...",
    "IPAddress": "172.17.0.2",
    "Gateway": "172.17.0.1"
  }
}
```

***

### ✅ Astuce

Si tu veux **sortir en tableau propre**, combine avec `printf` :

```bash
docker ps --format 'table {{.ID}}\t{{.Names}}\t{{.Status}}'
```

Résultat :

```
CONTAINER ID   NAMES   STATUS
c3f279d17e0a   web     Up 3 minutes
aed84ee21bde   db      Up 3 minutes
```

## 🔗 Fonction `join` dans Docker `--format`

* **But** : concaténer une **liste de chaînes** (slice en Go) en une seule chaîne.
*   **Syntaxe** :

    ```go
    {{join LIST "séparateur"}}
    ```
* `LIST` → un tableau ou une liste (exemple : `.Args`, qui contient les arguments passés au conteneur).
* `"séparateur"` → ce que tu veux mettre entre les éléments (virgule, espace, tiret, etc.).

***

### Exemple concret

Afficher tous les arguments du conteneur séparés par une virgule :

```bash
docker inspect --format '{{join .Args " , "}}' container_id
```

👉 Si ton conteneur a été lancé avec :

```bash
docker run alpine echo hello world
```

Résultat attendu :

```
echo , hello , world
```

***

### Autres exemples pratiques

#### Séparateur espace simple

```bash
docker inspect --format '{{join .Args " "}}' container_id
```

➡️ `echo hello world`

#### Séparateur `|`

```bash
docker inspect --format '{{join .Args " | "}}' container_id
```

➡️ `echo | hello | world`

***

⚠️ **Astuce shell** :

* Sous Linux/macOS → `'{{...}}'` fonctionne directement.
*   Sous PowerShell → il faut échapper `"` :

    ```powershell
    docker inspect --format '{{join .Args \" , \"}}' container_id
    ```

## 📊 Fonction `table` avec `--format`

* **But** : afficher la sortie sous forme de **tableau**, avec un en-tête généré automatiquement.
*   **Syntaxe** :

    ```bash
    docker <commande> --format "table {{.Champ1}}\t{{.Champ2}}\t..."
    ```
* Chaque **`{{.Champ}}`** correspond à un champ disponible dans l’objet de sortie (exemple : `.ID`, `.Repository`, `.Tag` pour les images).
* `\t` = tabulation → permet d’aligner les colonnes.

***

### Exemple pratique

Lister les images Docker avec un tableau personnalisé :

```bash
docker image list --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
```

➡️ Résultat (exemple) :

```
IMAGE ID       REPOSITORY   TAG
33a5cc25d22c   ubuntu       24.04
152dc042452c   ubuntu       22.04
a8cbb8c69ee7   alpine       3.21
7144f7bab3d4   alpine       latest
```

👉 Ici :

* **`{{.ID}}`** → ID de l’image
* **`{{.Repository}}`** → nom du repo
* **`{{.Tag}}`** → tag de l’image

***

### Autre exemple avec containers

Lister les conteneurs avec ID, nom et statut :

```bash
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"
```

➡️ Résultat (exemple) :

```
CONTAINER ID   NAMES         STATUS
f1c3d1b2a123   web_server    Up 3 minutes
a4e5d6c7b890   db_service    Exited (0) 5 minutes ago
```

***

👉 Si tu n’écris pas `table` au début, tu n’auras **pas les en-têtes** (`IMAGE ID`, `REPOSITORY`, etc.), juste les valeurs.

## 📦 Fonction `json`

* **But** : encoder un élément (objet, tableau, champ complexe) en **chaîne JSON**.
*   **Syntaxe** :

    ```bash
    docker inspect --format '{{json .Champ}}' <objet>
    ```
* Très utile pour :
  * afficher des structures imbriquées (listes, dictionnaires) de manière lisible,
  * exporter des données pour être traitées par un autre outil (jq, scripts, etc.).

***

### Exemple : afficher les montages d’un conteneur

```bash
docker inspect --format '{{json .Mounts}}' mon_conteneur
```

➡️ Résultat typique (tout en JSON) :

```json
[
  {
    "Type": "volume",
    "Name": "my_data",
    "Source": "/var/lib/docker/volumes/my_data/_data",
    "Destination": "/data",
    "Driver": "local",
    "Mode": "",
    "RW": true,
    "Propagation": ""
  }
]
```

***

### Autres exemples pratiques

#### 🔹 Afficher les ports exposés

```bash
docker inspect --format '{{json .NetworkSettings.Ports}}' mon_conteneur
```

Exemple de sortie :

```json
{
  "80/tcp": [
    {
      "HostIp": "0.0.0.0",
      "HostPort": "8080"
    }
  ]
}
```

***

#### 🔹 Récupérer les labels

```bash
docker inspect --format '{{json .Config.Labels}}' mon_conteneur
```

Exemple de sortie :

```json
{
  "maintainer": "admin@example.com",
  "version": "1.0"
}
```

***

### Bonus 🚀

En combinant avec **jq**, tu peux filtrer directement :

```bash
docker inspect --format '{{json .Mounts}}' mon_conteneur | jq '.[].Source'
```

➡️ Affichera uniquement les **sources** des volumes montés.

## 🔡 Fonction `lower`

* **But** : transformer une **chaîne de caractères** en sa version **minuscules**.
*   **Syntaxe** :

    ```bash
    docker inspect --format "{{lower .Champ}}" <container>
    ```

***

### Exemple pratique

#### 🔹 Nom du conteneur en minuscules

```bash
docker inspect --format "{{lower .Name}}" mon_conteneur
```

➡️ Si le conteneur s’appelle `/MyApp`, la sortie sera :

```
/myapp
```

***

### Autres usages utiles

#### 🔹 Forcer l’uniformité des labels

```bash
docker inspect --format '{{lower .Config.Labels.maintainer}}' mon_conteneur
```

➡️ Si le label est `Admin@Example.COM` → la sortie sera :

```
admin@example.com
```

***

#### 🔹 Baser des scripts sur des noms normalisés

Exemple : comparer le nom d’un conteneur sans se soucier de la casse :

```bash
if [ "$(docker inspect --format '{{lower .Name}}' mon_conteneur)" = "/backend" ]; then
  echo "Conteneur backend détecté"
fi
```

## ✂️ Fonction `split`

* **But** : découper une **chaîne de caractères** en une **liste** selon un **séparateur**.
*   **Syntaxe** :

    ```bash
    docker inspect --format '{{split .Champ "séparateur"}}' <container>
    ```

***

### Exemple concret

#### 🔹 Découper l’image en **nom** et **tag**

```bash
docker inspect --format '{{split .Image ":"}}' mon_conteneur
```

➡️ Si l’image est :

```
alpine:3.21
```

La sortie sera une **liste** :

```
[alpine 3.21]
```

***

### Utilisation pratique

#### 🔹 Récupérer uniquement le nom de l’image

```bash
docker inspect --format '{{index (split .Image ":") 0}}' mon_conteneur
```

➡️ Résultat :

```
alpine
```

#### 🔹 Récupérer uniquement le tag

```bash
docker inspect --format '{{index (split .Image ":") 1}}' mon_conteneur
```

➡️ Résultat :

```
3.21
```

## 🔠 Fonction `title`

* **But** : Met en **majuscule la première lettre** d’une chaîne de caractères.
*   **Syntaxe** :

    ```bash
    docker inspect --format "{{title .Champ}}" <container>
    ```

***

### Exemple concret

#### 🔹 Capitaliser le nom du conteneur

```bash
docker inspect --format "{{title .Name}}" mon_conteneur
```

➡️ Si `.Name` vaut :

```
/mycontainer
```

Le résultat sera :

```
/Mycontainer
```

***

### 💡 Utilisation pratique

* Rendre des noms ou labels plus lisibles dans les sorties.
* Exemple avec une liste de conteneurs :

```bash
docker ps --format "table {{.ID}}\t{{title .Names}}\t{{.Image}}"
```

➡️ Produit une table où la première lettre du nom des conteneurs est en majuscule.

## 🔠 Fonction `upper`

* **But** : Transformer **toute une chaîne de caractères en majuscules**.
*   **Syntaxe** :

    ```bash
    docker inspect --format "{{upper .Champ}}" <container>
    ```

***

### Exemple concret

#### 🔹 Mettre le nom du conteneur en majuscules

```bash
docker inspect --format "{{upper .Name}}" mon_conteneur
```

➡️ Si `.Name` vaut :

```
/mycontainer
```

Le résultat sera :

```
/MYCONTAINER
```

***

### 💡 Utilisation pratique

* Normaliser des valeurs de sortie (labels, noms, IDs, chemins).
* Exemple avec `docker ps` :

```bash
docker ps --format "table {{.ID}}\t{{upper .Names}}\t{{.Image}}"
```

➡️ Affiche une table où tous les noms de conteneurs sont en **MAJUSCULES**.

## 📐 Fonction `pad`

* **But** : Ajouter des espaces avant et/ou après une chaîne pour obtenir un affichage **aligné** ou plus lisible.
*   **Syntaxe** :

    ```bash
    {{pad CHAINE avant après}}
    ```

    * `avant` → nombre d’espaces ajoutés **avant** la chaîne.
    * `après` → nombre d’espaces ajoutés **après** la chaîne.

***

### Exemple concret

#### 🔹 Ajouter du padding à un dépôt d’image

```bash
docker image list --format '{{pad .Repository 5 10}}'
```

➡️ Si le dépôt est `ubuntu`, le résultat sera :

```
     ubuntu          
```

* 5 espaces **avant** `ubuntu`
* 10 espaces **après**

***

### 💡 Cas pratiques

*   Alignement manuel des colonnes lors d’un affichage personnalisé :

    ```bash
    docker image list --format "table {{pad .Repository 0 15}}{{pad .Tag 0 10}}{{.ID}}"
    ```

    ➝ Crée une table bien alignée même si les noms de dépôts et tags ont des longueurs différentes.

## ✂️ Fonction `truncate`

* **But** : Raccourcir une chaîne de caractères à une longueur précise.
*   **Syntaxe** :

    ```bash
    {{truncate CHAINE longueur}}
    ```

    * `CHAINE` → la valeur (ex. `.Repository`)
    * `longueur` → le nombre maximum de caractères à garder.

👉 Si la chaîne est **plus courte**, elle reste inchangée.\
👉 Si elle est **plus longue**, elle est coupée après la longueur indiquée.

***

### Exemple concret

#### 🔹 Tronquer les noms de dépôts à 15 caractères

```bash
docker image list --format '{{truncate .Repository 15}}'
```

➡️ Résultat (exemple) :

```
verylongrepository → verylongreposito
alpine             → alpine
ubuntu             → ubuntu
```

***

### 💡 Cas pratiques

* Éviter que les colonnes débordent quand les noms d’images ou conteneurs sont trop longs.
* Produire des rapports lisibles en coupant les chaînes trop verbeuses.

## 🖨️ Fonction `println`

* **But** : Afficher plusieurs valeurs en les séparant automatiquement par un **saut de ligne**.
*   **Syntaxe** :

    ```bash
    {{println valeur1 valeur2 ...}}
    ```

👉 Contrairement à `print`, qui met tout sur une seule ligne, `println` ajoute un retour à la ligne après chaque appel.

***

### Exemple simple

```bash
docker inspect --format='{{println .Name .Id}}' container
```

➡️ Résultat :

```
/my_container
3f5b6c2e4a9d...
```

***

### Exemple avec un `range`

Lister tous les réseaux attachés à un conteneur :

```bash
docker inspect --format='{{range .NetworkSettings.Networks}}{{println .IPAddress}}{{end}}' container
```

➡️ Résultat (exemple) :

```
172.17.0.2
192.168.1.10
```

***

### 💡 Cas pratiques

* Lister les IPs d’un conteneur, une par ligne.
* Afficher des labels ou variables sans devoir gérer les tabulations manuellement.
* Générer une sortie lisible pour des scripts shell qui consomment ligne par ligne.



## 📌 Champs disponibles avec `--format` dans Docker

### 🔹 `docker container ls`

Exemple pour voir tous les champs :

```bash
docker container ls --format '{{json .}}'
```

#### Champs principaux :

* **.ID** → ID du conteneur
* **.Image** → Image utilisée
* **.Command** → Commande de démarrage
* **.CreatedAt** → Date/heure de création
* **.RunningFor** → Depuis combien de temps il tourne
* **.Ports** → Les ports exposés
* **.Status** → Statut (Up, Exited…)
* **.State** → État (running, exited…)
* **.Names** → Nom du conteneur
* **.Networks** → Réseau(s) associés
* **.Mounts** → Volumes montés

📌 Exemple :

```bash
docker container ls --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
```

***

### 🔹 `docker image ls`

Exemple :

```bash
docker image ls --format '{{json .}}'
```

#### Champs principaux :

* **.ID** → ID de l’image
* **.Repository** → Nom du dépôt
* **.Tag** → Tag de l’image
* **.Digest** → Digest de l’image
* **.CreatedSince** → Depuis quand l’image a été créée
* **.CreatedAt** → Date de création précise
* **.Size** → Taille de l’image

📌 Exemple :

```bash
docker image ls --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

***

### 🔹 `docker volume ls`

Exemple :

```bash
docker volume ls --format '{{json .}}'
```

#### Champs principaux :

* **.Driver** → Pilote du volume
* **.Name** → Nom du volume
* **.Mountpoint** → Point de montage (via inspect)

📌 Exemple :

```bash
docker volume ls --format "table {{.Name}}\t{{.Driver}}"
```

***

### 🔹 `docker network ls`

Exemple :

```bash
docker network ls --format '{{json .}}'
```

#### Champs principaux :

* **.ID** → ID du réseau
* **.Name** → Nom du réseau
* **.Driver** → Pilote (bridge, overlay, host…)
* **.Scope** → Portée (local, swarm)

📌 Exemple :

```bash
docker network ls --format "table {{.Name}}\t{{.Driver}}\t{{.Scope}}"
```

***

⚡ Grâce à ça, tu peux composer tes propres affichages personnalisés avec `table`, `pad`, `truncate`, etc.
