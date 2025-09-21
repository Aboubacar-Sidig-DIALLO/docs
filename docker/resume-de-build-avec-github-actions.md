# 📊 Résumé de build avec GitHub Actions

Les **GitHub Actions Docker** (pour construire et pousser des images) génèrent automatiquement un **résumé de job** (_build summary_) qui détaille :

* 📄 le **Dockerfile utilisé**,
* ⏱️ la **durée du build**,
* ♻️ l’**utilisation du cache**,
* 🔑 les **inputs** (arguments de build, tags, labels, contextes de build),
* 🧩 et, si tu utilises **Bake**, la **définition complète du build**.

👉 Ce résumé te permet d’avoir une vision claire de ce qui s’est passé pendant le build.

***

### 🖥️ Où voir le résumé ?

* Les résumés apparaissent **automatiquement** si tu utilises :
  * `docker/build-push-action@v6`
  * `docker/bake-action@v6`
* Pour le consulter :
  * ouvre la page **Details** du job GitHub Actions,
  * après la fin du job (réussi ✅ ou échoué ❌).
  * En cas d’échec, le résumé affiche aussi le **message d’erreur**.

***

### 📥 Importer les enregistrements de build dans Docker Desktop

⚠️ Disponibilité :

* **Beta**
* Nécessite **Docker Desktop 4.31 ou plus récent**.

👉 Le résumé inclut un lien permettant de **télécharger une archive ZIP** contenant les détails du build (_build record archive_).

Cette archive peut être **importée dans Docker Desktop** pour analyser plus en profondeur ton build via l’interface graphique.

#### Étapes :

1. 📥 Installe **Docker Desktop** (≥ 4.31).
2. Télécharge l’archive ZIP depuis le résumé GitHub Actions.
3. Ouvre l’onglet **Builds** dans Docker Desktop.
4. Clique sur **Import build** et sélectionne l’archive ZIP, ou bien fais un **glisser-déposer**.
5. Clique sur **Import**.

⏳ Après quelques secondes, ton build apparaît sous **Completed builds**.

* Clique dessus pour voir en détail :
  * les **inputs**,
  * les **résultats**,
  * les **étapes de build**,
  * et l’**utilisation du cache**.

***

### ⚙️ Désactiver le job summary

Si tu ne veux **pas générer de résumé**, définis la variable d’environnement **`DOCKER_BUILD_SUMMARY`** sur `false` dans ton workflow :

```yaml
- name: Build
  uses: docker/build-push-action@v6
  env:
    DOCKER_BUILD_SUMMARY: false
  with:
    tags: ${{ steps.meta.outputs.tags }}
    labels: ${{ steps.meta.outputs.labels }}
```

***

### ⚙️ Désactiver l’upload des archives de build

Si tu veux garder le **résumé visible** mais **empêcher l’upload de l’archive ZIP** dans GitHub :

```yaml
- name: Build
  uses: docker/build-push-action@v6
  env:
    DOCKER_BUILD_RECORD_UPLOAD: false
  with:
    tags: ${{ steps.meta.outputs.tags }}
    labels: ${{ steps.meta.outputs.labels }}
```

👉 Résultat : le résumé est affiché, mais **sans lien de téléchargement**.

***

### ⚠️ Limitations

* Les **résumés de build** ne sont **pas disponibles** sur **GitHub Enterprise Server**.
* Fonctionne uniquement pour les dépôts hébergés sur **GitHub.com**.

***

✅ Avec ça, tu sais comment :

* consulter un résumé clair de tes builds Docker dans GitHub Actions,
* importer les enregistrements dans Docker Desktop pour analyse avancée,
* désactiver ou restreindre les résumés si besoin.
