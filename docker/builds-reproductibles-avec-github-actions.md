# 🔄 Builds Reproductibles avec GitHub Actions

👉 Les **builds reproductibles** garantissent que **chaque exécution produit exactement le même résultat** 🔒.\
Cela évite que des différences de timestamps ou de métadonnées rendent les images différentes, alors que le code source n’a pas changé.

***

### 📌 La variable `SOURCE_DATE_EPOCH`

* `SOURCE_DATE_EPOCH` est une **variable d’environnement standardisée** 🌍.
* Elle **force les timestamps** dans :
  * l’index de l’image,
  * la configuration,
  * et les métadonnées des fichiers\
    ➝ à refléter **une date précise en Unix time** (secondes depuis 1970-01-01).

💡 Il suffit de la définir dans ton workflow GitHub Actions avec la propriété `env` sur l’étape de build.

***

### 🕐 Exemple 1 : Timestamps fixes (Unix Epoch = 0)

Ici, on définit la date comme **1970-01-01 00:00:00 UTC** :

```yaml
name: ci

on:
  push:

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build
        uses: docker/build-push-action@v6
        with:
          tags: user/app:latest
        env:
          SOURCE_DATE_EPOCH: 0
```

✅ Chaque build générera une image avec exactement les mêmes métadonnées de temps.

***

### 🕒 Exemple 2 : Timestamps du commit Git

Pour plus de précision, tu peux caler la date du build sur **le timestamp du dernier commit Git** :

```yaml
name: ci

on:
  push:

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # Récupère le timestamp du dernier commit
      - name: Get Git commit timestamps
        run: echo "TIMESTAMP=$(git log -1 --pretty=%ct)" >> $GITHUB_ENV

      - name: Build
        uses: docker/build-push-action@v6
        with:
          tags: user/app:latest
        env:
          SOURCE_DATE_EPOCH: ${{ env.TIMESTAMP }}
```

✅ Chaque build sera lié au commit Git correspondant → reproductibilité + traçabilité.

***

### 🎯 Avantages des builds reproductibles

* 🔒 **Sécurité** : on évite des différences binaires invisibles (supply chain).
* 📦 **Cohérence** : l’image ne change que si le code change.
* ⚡ **Optimisation du cache** : Docker peut mieux réutiliser les couches.
* 🕵️ **Traçabilité** : facile de prouver qu’une image correspond à un commit exact.
