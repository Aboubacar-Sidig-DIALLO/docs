# 📑 Utiliser plusieurs fichiers Compose

### 🧐 Pourquoi utiliser plusieurs fichiers Compose ?

Travailler avec plusieurs fichiers Compose permet de **personnaliser une application** selon différents **environnements** (développement, test, production) ou **workflows**.

👉 C’est particulièrement utile pour les **grosses applications** qui :

* utilisent des dizaines de conteneurs 🐳,
* avec une gestion répartie entre **plusieurs équipes** 👥.

#### Exemple concret

* Dans une organisation qui utilise un **monorepo**, chaque équipe peut avoir son propre fichier Compose “local” 📂 pour exécuter uniquement une partie de l’application.
* Chaque équipe doit aussi se baser sur un **fichier Compose de référence** maintenu par une autre équipe, qui décrit la manière attendue d’exécuter le sous-ensemble d’application dont elle dépend.

👉 Résultat : la **complexité** n’est plus seulement dans le **code**, mais aussi dans l’**infrastructure** et la **configuration**.

***

### ⚡ La méthode la plus rapide

Le moyen le plus simple pour utiliser plusieurs fichiers Compose est de **fusionner** plusieurs fichiers avec l’option **`-f`** dans la ligne de commande :

```bash
docker compose -f compose.yaml -f compose.override.yaml up
```

⚠️ Cependant, les **règles de fusion** peuvent vite rendre cette approche **complexe et difficile à maintenir**.

***

### 🛠️ Les solutions avancées de Compose

Docker Compose propose deux mécanismes supplémentaires pour mieux gérer cette complexité :

1. **Étendre un fichier Compose**
   * Vous pouvez faire référence à un autre fichier Compose,
   * sélectionner uniquement les parties qui vous intéressent,
   * et même **surcharger certains attributs** selon vos besoins.
2. **Inclure directement d’autres fichiers Compose**
   * Vous pouvez intégrer d’autres fichiers Compose **dans votre fichier principal**,
   * ce qui centralise la configuration tout en gardant une organisation modulaire.

***

✅ En résumé :

* **Multi-fichiers Compose = flexibilité** selon l’environnement (dev, test, prod).
* Possibilités :
  * `-f` → fusion simple mais complexe à gérer à grande échelle.
  * **Extend** → hériter d’un fichier Compose et le modifier partiellement.
  * **Include** → inclure directement d’autres fichiers pour composer un tout.
