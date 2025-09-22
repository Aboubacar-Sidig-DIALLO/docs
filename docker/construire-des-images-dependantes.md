---
description: '📋 Options de la page  Nécessite : Docker Compose 2.22.0 ou version ultérieure'
---

# 🏗️ Construire des images dépendantes

### 🎯 Pourquoi utiliser des images dépendantes ?

Pour **réduire le temps de push/pull** ⬆️⬇️ et le **poids des images** 🪶, une pratique courante dans les applications Compose est de faire en sorte que les **services partagent un maximum de couches d’image**.

👉 En général, cela se fait en choisissant la **même image de base** du système d’exploitation pour tous vos services.

Mais vous pouvez aller encore plus loin :

* en partageant des couches lorsque vos images installent les **mêmes paquets système** 📦.
* L’enjeu est alors d’**éviter de répéter** les mêmes instructions Dockerfile dans tous vos services.

***

### 📝 Exemple d’illustration

Imaginons que vous vouliez que **tous vos services** :

1. soient construits à partir de l’image de base **`alpine`**,
2. et installent le paquet système **`openssl`**.

***

✅ Cette approche permet d’avoir des builds plus rapides ⚡, des images plus légères 🪶, et un projet **plus facile à maintenir** puisqu’on évite la duplication de code entre services.

## 🏗️ Dockerfile multi-étapes

### 🎯 Approche recommandée

La méthode conseillée consiste à **regrouper les déclarations communes dans un seul `Dockerfile`**, puis à utiliser la fonctionnalité des **multi-stages**.\
👉 Ainsi, chaque image de service sera construite à partir de cette **déclaration partagée**, sans duplication de code.

***

### 📂 Exemple de `Dockerfile`

```dockerfile
FROM alpine as base
RUN /bin/sh -c apk add --update --no-cache openssl

FROM base as service_a
# construction spécifique au service A
...

FROM base as service_b
# construction spécifique au service B
...
```

* Le stage `base` installe une fois pour toutes le paquet **openssl** 🛡️.
* Les stages `service_a` et `service_b` héritent de ce stage commun pour ajouter leur propre logique.

***

### 📂 Exemple de fichier `compose.yaml`

```yaml
services:
  a:
    build:
      target: service_a

  b:
    build:
      target: service_b
```

👉 Ici,

* le service **`a`** utilise le stage **`service_a`**,
* le service **`b`** utilise le stage **`service_b`**,\
  tous deux basés sur le même stage de base **`base`**.

***

✅ Avantages :

* Pas de **répétition d’instructions** entre services ✨
* Des **couches partagées** ➝ builds plus rapides ⚡
* Des images plus **légères** 🪶 et une maintenance simplifiée

## 🔄 Utiliser l’image d’un autre service comme image de base

### 🎯 Principe

Un schéma courant consiste à **réutiliser l’image d’un service comme image de base** pour un autre service.

👉 Problème : Comme Compose **ne lit pas le contenu des Dockerfiles**, il **ne peut pas détecter automatiquement cette dépendance** entre services pour exécuter les builds dans le bon ordre.

***

### 📝 Exemple initial

📂 `a.Dockerfile`

```dockerfile
FROM alpine
RUN /bin/sh -c apk add --update --no-cache openssl
```

📂 `b.Dockerfile`

```dockerfile
FROM service_a
# construction du service b
```

📂 `compose.yaml`

```yaml
services:
  a:
    image: service_a
    build:
      dockerfile: a.Dockerfile

  b:
    image: service_b
    build:
      dockerfile: b.Dockerfile
```

***

### ⚠️ Limite entre Compose v1 et v2

* **Compose v1 (ancien)** : construisait les images de manière **séquentielle** 🕐 → ce schéma fonctionnait directement.
* **Compose v2 (actuel)** : utilise **BuildKit** 🛠️ → les images sont construites **en parallèle** ⚡ → il faut donc **déclarer explicitement** la dépendance.

***

### ✅ Approche recommandée : `additional_contexts`

La solution est de **déclarer l’image de base comme un contexte de build additionnel** :

📂 `compose.yaml`

```yaml
services:
  a:
    image: service_a
    build:
      dockerfile: a.Dockerfile

  b:
    image: service_b
    build:
      dockerfile: b.Dockerfile
      additional_contexts:
        # `FROM service_a` sera résolu comme une dépendance sur le service "a",
        # qui doit donc être construit en premier
        service_a: "service:a"
```

👉 Ici, `service_a` est **reconnu comme dépendance** de `b`. Compose construira donc `a` avant `b`.

***

### 🔄 Variante avec un alias (`base_image`)

Vous pouvez aussi référencer une image de service avec un **nom symbolique** (alias) au lieu de l’appeler directement.

📂 `b.Dockerfile`

```dockerfile
FROM base_image
# `base_image` ne correspond pas à une image réelle,
# mais sera résolu comme un contexte additionnel nommé
# construction du service b
```

📂 `compose.yaml`

```yaml
services:
  a:
    build:
      dockerfile: a.Dockerfile
      # l'image sera taguée automatiquement <project_name>_a

  b:
    build:
      dockerfile: b.Dockerfile
      additional_contexts:
        # `FROM base_image` sera résolu comme dépendance sur "a"
        base_image: "service:a"
```

***

### ✅ Avantages de `additional_contexts`

* Permet de **réutiliser directement** les images construites par d’autres services.
* Assure que les **dépendances sont respectées** (ex. `a` construit avant `b`).
* Évite les erreurs liées au build parallèle de BuildKit.

## 🏗️ Construire avec **Bake**

### 🎯 Pourquoi utiliser Bake ?

L’utilisation de **Bake** vous permet de :

* transmettre une **définition complète de build** pour tous vos services,
* et d’**orchestrer l’exécution des builds** de manière la plus efficace possible ⚡.

***

### ▶️ Activer Bake

Pour activer cette fonctionnalité, lancez Compose avec la variable d’environnement :

```bash
COMPOSE_BAKE=true docker compose build
```

***

#### Exemple de sortie :

```
[+] Building 0.0s (0/1)                                                         
 => [internal] load local bake definitions                                 0.0s
...
[+] Building 2/2 manifest list sha256:4bd2e88a262a02ddef525c381a5bdb08c83  0.0s
 ✔ service_b  Built                                                        0.7s 
 ✔ service_a  Built
```

👉 Ici, `service_a` et `service_b` sont construits via Bake.

***

### ⚙️ Définir Bake comme constructeur par défaut

Vous pouvez aussi définir **Bake** comme **builder par défaut** en modifiant le fichier de configuration suivant :

📂 `$HOME/.docker/config.json`

```json
{
  ...
  "plugins": {
    "compose": {
      "build": "bake"
    }
  }
  ...
}
```

***

### 📚 Ressources supplémentaires

* 🔗 Docker Compose build reference
* 🔗 Apprendre les Dockerfiles multi-étapes

***

✅ En résumé :

* `COMPOSE_BAKE=true` → permet d’utiliser Bake lors du build,
* `config.json` → permet de l’activer **par défaut**,
* bénéfices : **orchestration optimisée, builds plus rapides et efficaces** ⚡.
