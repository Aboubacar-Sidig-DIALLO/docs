# 🛡️ Docker Rootless Mode

### 🔹 Qu’est-ce que le _rootless mode_ ?

Le **rootless mode** permet d’exécuter le **démon Docker** (`dockerd`) et les **conteneurs** sans utiliser l’utilisateur _root_.\
👉 L’objectif est de **réduire les risques de sécurité**, car même si un attaquant réussit à compromettre un conteneur ou le démon Docker, il n’aura pas d’accès root sur la machine hôte.

***

### 🔹 Avantages

* ✅ **Sécurité renforcée** : pas besoin de privilèges root.
* ✅ **Isolation améliorée** : limite l’impact d’une éventuelle faille dans Docker ou dans un conteneur.
* ✅ **Installation utilisateur** : peut être installé et exécuté par un utilisateur standard, sans sudo.

***

### 🔹 Fonctionnement

En mode rootless :

* Le démon Docker s’exécute **dans l’espace utilisateur**, avec `user namespaces`.
* Les conteneurs utilisent aussi cet espace utilisateur, donc **les processus à l’intérieur du conteneur tournent comme utilisateurs normaux** (pas root).
* Certaines fonctionnalités nécessitant des capacités kernel root (comme le montage direct de systèmes de fichiers, l’IPv6 avancé, ou certains pilotes de stockage) ne sont pas disponibles.

***

### 🔹 Prérequis

Pour utiliser Docker rootless :

1. **Kernel Linux ≥ 5.11** (pour une meilleure compatibilité, bien que ça marche aussi avec ≥ 5.8 avec quelques limitations).
2. **User namespaces activés** (`/proc/sys/user/max_user_namespaces`).
3. Avoir un utilisateur non-root dédié.

***

### 🔹 Installation simplifiée

Depuis un utilisateur normal :

```bash
curl -fsSL https://get.docker.com/rootless | sh
```

Puis, ajoute Docker rootless à ton PATH et démarre le démon :

```bash
export PATH=$HOME/bin:$PATH
systemctl --user start docker
```

Vérifie :

```bash
docker info
```

Tu verras la mention `Rootless: true`.

***

### 🔹 Limitations connues

* 🚫 Pas de **cgroup v1** (il faut cgroup v2 activé).
* 🚫 Pas de **modes privilégiés complets** (`--privileged`).
* 🚫 Certaines fonctionnalités réseau limitées (ex. pas de `overlay` sans configuration supplémentaire).
* 🚫 Pas de support direct pour **NFS, CIFS** et certains pilotes de stockage.

***

👉 En résumé :\
Le **rootless mode** est recommandé si tu veux exécuter Docker en **environnement partagé, sensible ou sécurisé**, mais il peut manquer certaines fonctionnalités avancées.

## ⚙️ Fonctionnement du Rootless Mode

* Le **démon Docker (`dockerd`)** et les **conteneurs** s’exécutent **dans un espace utilisateur (`user namespace`)**.
* Cela ressemble au mode **`userns-remap`**, mais avec une différence clé :
  * **`userns-remap`** : le démon tourne en **root**, seuls les conteneurs tournent dans un namespace utilisateur.
  * **`rootless mode`** : **le démon ET les conteneurs** tournent sans privilèges root.

👉 Résultat : pas de processus root lié à Docker → surface d’attaque réduite ✅.

***

## 🔑 Prérequis

1.  **Paquets nécessaires** :

    * Installer `newuidmap` et `newgidmap` (disponibles dans le paquet `uidmap` sur la plupart des distributions).

    ```bash
    sudo apt-get install -y uidmap
    ```
2.  **UID/GID subordonnés** :

    * Dans `/etc/subuid` et `/etc/subgid`, il faut au moins **65 536 UID/GID** attribués à l’utilisateur.\
      Exemple :

    ```bash
    id -u
    1001

    whoami
    testuser

    grep ^$(whoami): /etc/subuid
    testuser:231072:65536

    grep ^$(whoami): /etc/subgid
    testuser:231072:65536
    ```

    Ici, `testuser` dispose bien de la plage **231072–296607** pour les UID/GID rootless.

***

## 🛠️ Installation du mode rootless

#### 1. Arrêter le Docker rootful (optionnel mais recommandé)

Si Docker est déjà actif en mode root :

```bash
sudo systemctl disable --now docker.service docker.socket
sudo rm /var/run/docker.sock
```

👉 Sinon, tu devras forcer l’installation avec `--force`.

***

#### 2. Installer avec le script fourni

Si tu as Docker ≥ 20.10 via `apt` ou `rpm`, tu as déjà le script :

```bash
dockerd-rootless-setuptool.sh install
```

📌 Ce que ça fait :

* Crée un service `systemd` user : `~/.config/systemd/user/docker.service`
* Active un contexte CLI `rootless`
* Configure la socket utilisateur `unix:///run/user/<UID>/docker.sock`

Sortie typique :

```
[INFO] Creating /home/testuser/.config/systemd/user/docker.service
[INFO] Installed docker.service successfully.
[INFO] To control docker.service, run: systemctl --user (start|stop|restart) docker.service
[INFO] To run docker.service on system startup, run: sudo loginctl enable-linger testuser
[INFO] Creating CLI context "rootless"
Successfully created context "rootless"
[INFO] Using CLI context "rootless"
```

***

#### 3. Ajouter les variables d’environnement

Dans ton `~/.bashrc` :

```bash
export PATH=/usr/bin:$PATH
export DOCKER_HOST=unix:///run/user/1000/docker.sock
```

⚠️ Adapter `1000` à ton UID.

***

## ✅ Vérification

Teste ton installation :

```bash
docker info
```

Exemple de sortie :

```
Client: Docker Engine - Community
 Context: rootless
...
Server:
 Security Options:
  seccomp
   Profile: builtin
  rootless
  cgroupns
...
```

👉 La ligne `rootless` confirme que Docker fonctionne bien en **Rootless mode**.

***

## 📌 Résumé

* **Rootless** = Docker sans root → plus sûr 🔒
* Nécessite `uidmap` + configuration de `/etc/subuid` et `/etc/subgid`.
* Installation via `dockerd-rootless-setuptool.sh install`.
* Vérification avec `docker info`.
