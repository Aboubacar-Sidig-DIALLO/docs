# 🐳 Lancer plusieurs processus dans un conteneur Docker

### 📌 Principe de base

* Par défaut, **un conteneur = un processus principal** (défini par `ENTRYPOINT` ou `CMD` dans le Dockerfile).
* Ce processus peut engendrer d’autres sous-processus (ex. Apache qui lance plusieurs workers).
* **Bonne pratique** : séparer les responsabilités → **un service par conteneur**.
  * Exemple : un conteneur pour la base de données, un pour l’application, un pour le cache, etc.
* Si tu as besoin de plusieurs services, il vaut mieux utiliser **plusieurs conteneurs reliés entre eux** via :
  * les **réseaux définis par l’utilisateur** (`docker network create`)
  * les **volumes partagés**.

⚠️ Mais si tu dois absolument faire tourner plusieurs processus dans un seul conteneur, il existe 3 solutions.

***

### 🔹 1. Utiliser un **script wrapper**

Tu regroupes le lancement de tous tes processus dans un script Bash, qui s’occupe aussi de gérer leur fin.

**Exemple du script :**

```bash
#!/bin/bash

# Lancer le premier processus
./my_first_process &

# Lancer le second processus
./my_second_process &

# Attendre qu’un processus s’arrête
wait -n

# Sortir avec le code du processus qui a échoué
exit $?
```

**Dockerfile :**

```dockerfile
FROM ubuntu:latest
COPY my_first_process my_first_process
COPY my_second_process my_second_process
COPY my_wrapper_script.sh my_wrapper_script.sh
CMD ./my_wrapper_script.sh
```

👉 Avantage : simple, facile à mettre en place.\
👉 Inconvénient : si tu ne gères pas bien les erreurs, un processus mort peut passer inaperçu.

***

### 🔹 2. Utiliser les **jobs Bash**

Utile quand un **processus principal doit rester actif**, et qu’un **processus secondaire** doit tourner temporairement.

**Script wrapper :**

```bash
#!/bin/bash

# Activer le contrôle de jobs
set -m

# Lancer le processus principal en arrière-plan
./my_main_process &

# Lancer le processus auxiliaire
./my_helper_process

# Remettre le processus principal au premier plan
fg %1
```

**Dockerfile :**

```dockerfile
FROM ubuntu:latest
COPY my_main_process my_main_process
COPY my_helper_process my_helper_process
COPY my_wrapper_script.sh my_wrapper_script.sh
CMD ./my_wrapper_script.sh
```

👉 Avantage : permet de garder un service principal en avant-plan.\
👉 Inconvénient : fragile si tu dois gérer plusieurs services complexes.

***

### 🔹 3. Utiliser un **gestionnaire de processus** (recommandé pour plusieurs services)

Exemple : **supervisord**.\
Il lance et supervise plusieurs processus dans ton conteneur.

**Dockerfile :**

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y supervisor
RUN mkdir -p /var/log/supervisor

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY my_first_process my_first_process
COPY my_second_process my_second_process

CMD ["/usr/bin/supervisord"]
```

**supervisord.conf :**

```ini
[supervisord]
nodaemon=true
logfile=/dev/null
logfile_maxbytes=0

[program:app]
stdout_logfile=/dev/fd/1
stdout_logfile_maxbytes=0
redirect_stderr=true
```

👉 Avantages :

* Gère automatiquement le redémarrage des processus
* Centralise les logs dans les logs Docker (`docker logs`)
* Plus robuste pour des services longs.

👉 Inconvénient : ajoute une couche supplémentaire (complexité + taille de l’image).

***

### ⚠️ Autres points importants

*   Si ton **processus principal ne gère pas bien ses enfants**, utilise `--init` avec `docker run`.\
    👉 Exemple :

    ```bash
    docker run --init myimage
    ```

    Cela ajoute un petit **init process** qui s’assure de "récolter" les processus enfants quand le conteneur s’arrête.
* **Évite systemd ou sysvinit** dans un conteneur : trop lourds et contraires aux bonnes pratiques Docker.

***

✅ **En résumé :**

* **Meilleure pratique** : 1 service = 1 conteneur.
* Si tu dois quand même lancer plusieurs services :
  * Simple → script wrapper,
  * Processus principal + aide → job control,
  * Plusieurs services robustes → `supervisord`.
* Pour éviter les zombies → utilise `--init`.
