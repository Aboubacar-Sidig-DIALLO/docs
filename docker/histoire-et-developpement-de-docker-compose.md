# 📜 Histoire et développement de Docker Compose

### 🎯 Introduction

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Aujourd’hui, la version officiellement supportée est **Compose v2**, basée sur la **Compose Specification**.\
Elle unifie et simplifie ce qui existait auparavant dans Compose v1.

👉 Les différences portent sur :

* le **format des fichiers** (`compose.yaml`)
* la **syntaxe CLI** (`docker-compose` → `docker compose`)
* et certains **éléments de configuration** au niveau supérieur.

***

### 🕰️ Historique du CLI Docker Compose

* **2014** → Première version de **Compose v1** (écrite en **Python**)
  * Invocation : `docker-compose`
  * Fichiers `compose.yaml` avec un élément de version (`version: "2.0" → "3.8"`)
* **2020** → Sortie de **Compose v2** (réécrit en **Go**)
  * Invocation : `docker compose` (sans tiret, intégré directement dans Docker CLI)
  * Ignore l’élément `version:` dans `compose.yaml` (car remplacé par la **Compose Specification**).

***

### 📄 Versions des formats de fichiers Compose

Trois grands formats de fichiers ont marqué Compose v1 :

1. **Format v1 (2014, Compose 1.0.0)**
   * Très différent des suivants.
   * ❌ Pas de clé `services:` au niveau supérieur.
   * ⚠️ Obsolète : les fichiers en format v1 ne fonctionnent pas avec Compose v2.
2. **Format v2.x (2016, Compose 1.6.0)**
   * Introduction d’une structure plus claire avec `services`, `networks`, `volumes`.
   * Utilisation massive en développement.
3. **Format v3.x (2017, Compose 1.10.0)**
   * Très proche du v2.x, mais avec de **nouvelles options** pour les déploiements avec **Swarm**.
   * Exemple : `deploy:` pour configurer réplicas, stratégies de placement, etc.

***

### 🌀 La confusion et la solution

Avant, il y avait un mélange :

* **Version du CLI Compose** (v1 ou v2)
* **Version du format de fichier** (1.x, 2.x, 3.x)
* **Support différent selon Swarm ou pas**

👉 Résultat : beaucoup de confusion.

💡 Pour clarifier, Docker a introduit la **Compose Specification** :

* Fusion des formats 2.x et 3.x
* **Spécification unique, évolutive et “rolling”** (plus de version fixe comme `3.7`, etc.)
* `version:` en haut du fichier devient **optionnel**.
* Support de spécifications optionnelles :
  * **Deploy** → orchestration et scaling
  * **Develop** → outils de développement
  * **Build** → étapes de build avancées

***

### 🔄 Compatibilité avec Compose v2

Pour faciliter la migration :

* Compose v2 est **compatible en rétro** avec certains éléments 2.x/3.x.
* Mais il encourage l’adoption de la **Compose Specification** moderne.

***

✅ En résumé :

* **Compose v1** (2014, Python, `docker-compose`) → historique, versions de fichiers 1, 2.x, 3.x
* **Compose v2** (2020, Go, `docker compose`) → unification, simplification, basée sur la Compose Specification
