# ♻️ Étendre votre fichier Compose

L’attribut **`extends`** de Docker Compose permet de **réutiliser des configurations communes** entre plusieurs fichiers Compose, voire même entre plusieurs projets différents.

👉 L’extension de services est particulièrement utile lorsque vous avez plusieurs services qui partagent un ensemble d’options de configuration.

Avec **`extends`**, vous pouvez :

* définir une **base de configuration commune** pour vos services dans un seul endroit,
* y faire référence depuis n’importe quel autre fichier,
* et **surcharger/adapter certains attributs** selon les besoins spécifiques de votre application.

***

### 🎯 Exemple d’usage concret

* Vous pouvez référencer un **autre fichier Compose**,
* sélectionner un service déjà défini,
* puis l’étendre en **modifiant ou en ajoutant** certaines options propres à votre projet.

***

### ⚠️ Point important

Lorsque vous utilisez **plusieurs fichiers Compose avec `extends`** :

* Tous les **chemins doivent être relatifs au fichier de base** (`base Compose file`).
* Le fichier de base correspond au fichier **situé dans le répertoire principal de votre projet**.
* Pourquoi ? Parce que les fichiers d’extension n’ont pas besoin d’être des fichiers Compose complets.
  * Ils peuvent contenir uniquement de petits **fragments de configuration**.
  * Suivre quel fragment est relatif à quel chemin deviendrait confus.

👉 Pour garder les chemins clairs et éviter les erreurs, **Docker impose que tous les chemins soient définis par rapport au fichier de base**.

## ⚙️ Comment fonctionne l’attribut `extends`

### 🔄 Étendre un service depuis un autre fichier

Prenons l’exemple suivant :

```yaml
services:
  web:
    extends:
      file: common-services.yml
      service: webapp
```

👉 Ici, **Compose réutilise uniquement** les propriétés du service `webapp` défini dans le fichier **`common-services.yml`**.\
➡️ Attention : le service `webapp` **n’est pas inclus** dans le projet final.

***

#### 🗂️ Exemple du fichier commun

**common-services.yml** :

```yaml
services:
  webapp:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - "/data"
```

⚡ Résultat :\
Vous obtenez exactement le même résultat que si vous aviez écrit un `compose.yaml` contenant directement :

```yaml
services:
  web:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - "/data"
```

***

### 🛠️ Inclure le service original dans le projet final

Si vous souhaitez que le service **`webapp`** apparaisse aussi dans le projet final, vous devez **l’inclure explicitement** en plus du service étendu :

```yaml
services:
  web:
    build: alpine
    command: echo
    extends:
      file: common-services.yml
      service: webapp

  webapp:
    extends:
      file: common-services.yml
      service: webapp
```

👉 Dans ce cas :

* `web` réutilise la configuration de `webapp` **mais avec ses propres surcharges** (`build: alpine`, `command: echo`).
* `webapp` est également inclus comme service autonome.

***

### 🔄 Alternative moderne

Au lieu de `extends`, vous pouvez également utiliser **`include`** pour importer directement des fichiers Compose entiers.

## 🧩 Étendre des services dans le **même fichier Compose**

Si vous définissez plusieurs services dans un **même fichier Compose** et qu’un service **étend** un autre, alors :\
➡️ **les deux services (l’original et l’étendu) feront partie de la configuration finale.**

***

### 📄 Exemple

```yaml
services:
  web:
    build: alpine
    extends: webapp

  webapp:
    environment:
      - DEBUG=1
```

***

### 🔍 Explication

* Le service **`web`** hérite de la configuration définie dans **`webapp`**, tout en ajoutant ou surchargeant certaines options (`build: alpine` ici).
* Le service **`webapp`** reste **disponible en tant que service autonome** dans la configuration finale.

👉 Résultat : la configuration fusionnée contient **deux services distincts** :

* `web` (qui utilise `build: alpine` et hérite de `webapp`),
* `webapp` (avec sa propre configuration `DEBUG=1`).

***

⚡ Cela diffère de l’extension **entre plusieurs fichiers** où le service original (`webapp`) n’est **pas automatiquement inclus** dans le projet final, sauf si vous l’ajoutez explicitement.

## 🔗 Étendre des services dans le même fichier **et** depuis un autre fichier

Vous pouvez aller encore plus loin avec **`extends`** en combinant plusieurs niveaux d’extension :

* hériter d’une configuration définie dans un **fichier externe** (`common-services.yml`),
* puis **ajouter ou redéfinir localement** certaines options dans votre `compose.yaml`,
* et même **chaîner l’extension** en créant de nouveaux services qui héritent de ceux déjà étendus.

***

### 📄 Exemple

```yaml
services:
  web:
    extends:
      file: common-services.yml
      service: webapp
    environment:
      - DEBUG=1
    cpu_shares: 5

  important_web:
    extends: web
    cpu_shares: 10
```

***

### 🔍 Explication pas à pas

1. **`web`** hérite du service `webapp` défini dans **`common-services.yml`**.
   * Il surcharge ensuite localement :
     * `environment: DEBUG=1`
     * `cpu_shares: 5`
2. **`important_web`** étend à son tour le service **`web`**.
   * Il conserve toutes ses propriétés mais redéfinit `cpu_shares: 10`.

***

### ✅ Résultat attendu

La configuration finale contient donc :

* **`web`** → basé sur `common-services.yml:webapp`, enrichi avec `DEBUG=1` et `cpu_shares: 5`.
* **`important_web`** → basé sur `web`, avec `cpu_shares: 10`.

***

⚡ Avec cette approche, vous pouvez construire une **hiérarchie de services** réutilisables, où chaque service hérite d’une base commune et ajoute ses propres spécificités.

## 📝 Exemple supplémentaire : factoriser une configuration commune

L’extension d’un service individuel est particulièrement utile lorsque vous avez **plusieurs services** qui partagent une même configuration de base.

👉 Dans cet exemple, nous avons une application Compose avec :

* un service **webapp** (application web),
* un service **queue\_worker** (consommateur de files de messages).

Les deux utilisent le **même code source** et partagent de nombreuses options de configuration.

***

### 📄 Fichier commun – `common.yaml`

Ce fichier définit la configuration partagée par tous les services :

```yaml
services:
  app:
    build: .
    environment:
      CONFIG_FILE_PATH: /code/config
      API_KEY: xxxyyy
    cpu_shares: 5
```

***

### 📄 Fichier principal – `compose.yaml`

Ce fichier définit les services concrets qui **héritent** de la configuration commune :

```yaml
services:
  webapp:
    extends:
      file: common.yaml
      service: app
    command: /code/run_web_app
    ports:
      - 8080:8080
    depends_on:
      - queue
      - db

  queue_worker:
    extends:
      file: common.yaml
      service: app
    command: /code/run_worker
    depends_on:
      - queue
```

***

### 🔍 Explication

* **`webapp`** :
  * hérite de `app` (défini dans `common.yaml`),
  * ajoute un `command` spécifique (`/code/run_web_app`),
  * ouvre un port `8080:8080`,
  * dépend des services `queue` et `db`.
* **`queue_worker`** :
  * hérite également de `app`,
  * définit son propre `command` (`/code/run_worker`),
  * dépend du service `queue`.

***

### ✅ Avantage

Grâce à cette approche :

* La configuration commune (`build`, `environment`, `cpu_shares`) est centralisée dans **`common.yaml`**.
* Chaque service (`webapp`, `queue_worker`) ajoute uniquement ses particularités.
* Cela réduit la duplication et améliore la maintenabilité dans des projets avec plusieurs services similaires.

## 📂 Chemins relatifs avec `extends`

Lorsque vous utilisez **`extends`** avec l’attribut **`file`** qui pointe vers un autre dossier, les **chemins relatifs** définis dans le service étendu sont **automatiquement convertis** afin de continuer à pointer vers les bons fichiers, même lorsqu’ils sont utilisés par le service qui fait l’extension.

***

### 📄 Exemple

#### 🔹 Fichier Compose principal (`compose.yaml`)

```yaml
services:
  webapp:
    image: example
    extends:
      file: ../commons/compose.yaml
      service: base
```

#### 🔹 Fichier commun (`../commons/compose.yaml`)

```yaml
services:
  base:
    env_file: ./container.env
```

***

### 🔍 Résultat

Le service **`webapp`** hérite de `base` et la référence au fichier **`./container.env`** est automatiquement convertie pour pointer vers **`../commons/container.env`** (donc dans le bon dossier).

On peut le vérifier avec la commande :

```bash
docker compose config
```

👉 Sortie obtenue :

```yaml
services:
  webapp:
    image: example
    env_file:
      - ../commons/container.env
```

***

✅ Cela garantit que même si les chemins sont définis dans un fichier d’extension situé ailleurs, Docker Compose ajuste correctement les chemins relatifs afin d’éviter toute erreur de référence.
