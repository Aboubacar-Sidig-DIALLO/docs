---
description: '📋 Options de la page  Nécessite : Docker Compose 2.30.0 ou version ultérieure'
---

# ⚓ Utiliser les hooks de cycle de vie avec Compose

### 🔄 Hooks de cycle de vie des services

Lorsque **Docker Compose** exécute un conteneur, il s’appuie sur deux éléments :

* **ENTRYPOINT**
* **COMMAND**

Ces deux composants définissent **ce qui se passe au démarrage et à l’arrêt du conteneur**.

👉 Cependant, il peut parfois être plus simple et plus flexible de gérer ces actions séparément grâce aux **hooks de cycle de vie** (_lifecycle hooks_) :

* Ce sont des commandes exécutées **juste après le démarrage** d’un conteneur,
* ou **juste avant son arrêt**.

***

### 🌟 Pourquoi les hooks sont utiles ?

Les hooks de cycle de vie sont particulièrement intéressants car ils peuvent bénéficier de **privilèges spéciaux** (par exemple, être exécutés avec l’utilisateur `root` 👑),\
➡️ même si le conteneur lui-même fonctionne avec des privilèges restreints pour des raisons de sécurité 🔒.

Cela signifie que :

* ✅ Vous pouvez réaliser certaines tâches nécessitant des autorisations élevées,
* ❌ Sans compromettre la sécurité globale du conteneur.

***

⚡ Exemple d’utilisation typique :

* Nettoyage avant arrêt 🧹
* Initialisation de fichiers ou de services au démarrage 🚀
* Configuration avancée nécessitant des droits élevés 🔧

## 🟢 Hooks **Post-start**

Les **hooks Post-start** sont des commandes qui s’exécutent **après que le conteneur ait démarré**.\
⚠️ Toutefois, il n’existe **aucun moment garanti** pour leur exécution précise.\
👉 En d’autres termes, leur déclenchement n’est pas assuré **pendant l’exécution de l’ENTRYPOINT** du conteneur.

***

### 📖 Exemple expliqué pas à pas

Dans l’exemple ci-dessous :

1. 🔒 Par défaut, les volumes sont créés avec un **propriétaire `root`**.
2. 👤 Le hook est utilisé pour **changer le propriétaire du volume** vers un utilisateur non-root.
3. 📂 Après le démarrage du conteneur, la commande `chown` modifie la propriété du répertoire `/data` vers l’utilisateur **1001**.

***

#### 📝 Exemple YAML

```yaml
services:
  app:
    image: backend
    user: 1001
    volumes:
      - data:/data    
    post_start:
      - command: chown -R /data 1001:1001
        user: root

volumes:
  data: {} # un volume Docker est créé avec la propriété root par défaut
```

***

### ✨ À retenir

* ✅ Les hooks **Post-start** permettent d’exécuter des commandes **immédiatement après le démarrage** du conteneur.
* 🔑 Ils sont utiles pour des opérations d’initialisation comme :
  * changer les permissions,
  * ajuster des répertoires,
  * lancer des tâches de configuration post-démarrage.

## 🔴 Hooks **Pre-stop**

Les **hooks Pre-stop** sont des commandes qui s’exécutent **juste avant que le conteneur ne soit arrêté** par une commande explicite, comme :

* `docker compose down` 🛑
* ou un arrêt manuel avec `Ctrl + C` ⌨️

⚠️ Attention :\
Ces hooks **ne s’exécutent pas** si le conteneur s’arrête de lui-même ou s’il est tué brutalement (`kill -9` par exemple).

***

### 📖 Exemple expliqué

Dans l’exemple ci-dessous :

1. Avant que le conteneur ne s’arrête,
2. Le script `./data_flush.sh` est exécuté,
3. Ce script permet d’effectuer les **nettoyages nécessaires** 🧹 (flush des données, sauvegardes, libération de ressources, etc.).

***

#### 📝 Exemple YAML

```yaml
services:
  app:
    image: backend
    pre_stop:
      - command: ./data_flush.sh
```

***

### 📚 Informations de référence

* **post\_start** 👉 commandes exécutées **après** le démarrage du conteneur.
* **pre\_stop** 👉 commandes exécutées **juste avant** l’arrêt du conteneur.

***

✅ En résumé, les hooks de cycle de vie (**post\_start** & **pre\_stop**) permettent de mieux contrôler les étapes critiques : **initialisation** et **arrêt**, sans devoir tout gérer dans l’`ENTRYPOINT`.
