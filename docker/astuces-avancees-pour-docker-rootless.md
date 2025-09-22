# ⚙️ Astuces avancées pour Docker Rootless

### 🚀 Gestion du démon avec `systemd` (recommandé)

Quand tu installes Docker en mode rootless avec le script `dockerd-rootless-setuptool.sh install`, un **fichier `systemd` user** est créé automatiquement :

```
~/.config/systemd/user/docker.service
```

👉 Cela veut dire que tu peux gérer Docker **par utilisateur**, indépendamment du démon système.

#### 📌 Commandes utiles

*   Démarrer Docker rootless :

    ```bash
    systemctl --user start docker
    ```
*   Activer au démarrage (session utilisateur persistante) :

    ```bash
    systemctl --user enable docker
    sudo loginctl enable-linger $(whoami)
    ```

⚠️ **Important** :\
Tu **ne peux pas** lancer Rootless Docker comme un service `systemd` global (`/etc/systemd/system/docker.service`) → même avec `User=`.\
👉 Il faut rester dans le contexte **par utilisateur**.

***

### 📂 Emplacements par défaut

Quand Docker tourne en mode rootless :

*   **Socket Docker** (communication client/daemon) :

    ```
    $XDG_RUNTIME_DIR/docker.sock
    ```

    En général : `/run/user/$UID/docker.sock`
*   **Répertoire des données Docker** (images, volumes, etc.) :

    ```
    ~/.local/share/docker
    ```

    ⚠️ **Ne pas mettre sur un NFS** (problèmes de performance et de permissions).
*   **Répertoire de configuration du démon** :

    ```
    ~/.config/docker
    ```

    ⚠️ Différent de `~/.docker` → qui est utilisé par le **client Docker**.

***

### 🔑 Résumé des bonnes pratiques

1. **Toujours utiliser `systemd --user`** pour gérer le démon rootless.
2. **Activer lingering** pour que Docker rootless démarre même après une déconnexion.
3. **Bien connaître les chemins spécifiques rootless** :
   * Socket : `/run/user/$UID/docker.sock`
   * Données : `~/.local/share/docker`
   * Config : `~/.config/docker`
4. **Ne pas utiliser NFS** pour les données (`~/.local/share/docker`).

## 🐳 Docker Rootless – Côté client

### 📌 Depuis Docker Engine v23.0

*   Quand tu exécutes :

    ```bash
    dockerd-rootless-setuptool.sh install
    ```

    👉 Le script configure **automatiquement** le client `docker CLI` pour qu’il utilise le **contexte rootless**.\
    Donc tu peux directement lancer :

    ```bash
    docker ps
    docker run ...
    ```

    sans rien ajouter.

***

### 📌 Avant Docker Engine v23.0

Avant cette version, il fallait **indiquer manuellement** au client Docker où se trouve le démon rootless.

#### 🔹 1. Via la variable d’environnement `$DOCKER_HOST`

Le démon rootless écoute sur un socket utilisateur :

```
$XDG_RUNTIME_DIR/docker.sock
```

Donc il fallait exporter la variable :

```bash
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock
docker run -d -p 8080:80 nginx
```

👉 Ici, `docker run` sait qu’il doit parler au démon rootless au lieu du démon système.

***

#### 🔹 2. Via un **contexte Docker**

Une autre solution était d’utiliser les **contexts** de Docker CLI.

```bash
docker context use rootless
```

Sortie :

```
rootless
Current context is now "rootless"
```

Ensuite, tout est exécuté dans ce contexte :

```bash
docker run -d -p 8080:80 nginx
```

***

### 🚀 Résumé

* **Docker ≥ v23.0** → rien à configurer, le script `setuptool` fait tout.
* **Docker < v23.0** → deux solutions pour connecter le client au démon rootless :
  1. `export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock`
  2. `docker context use rootless`

## 🐳 Bonnes pratiques avec Rootless Docker

### 1. Rootless Docker dans Docker (DinD)

Si tu veux exécuter **Rootless Docker à l’intérieur d’un Docker “rootful”**, utilise l’image spéciale `docker:<version>-dind-rootless` au lieu de `docker:<version>-dind`.

```bash
docker run -d --name dind-rootless --privileged docker:25.0-dind-rootless
```

* Cette image tourne avec l’utilisateur non-root `UID=1000`.
* ⚠️ L’option `--privileged` reste nécessaire pour désactiver **seccomp**, **AppArmor** et les masques de montage.

***

### 2. Exposer le socket Docker via **TCP**

Pour exposer l’API Docker via TCP :\
Lancer `dockerd-rootless.sh` avec la variable `DOCKERD_ROOTLESS_ROOTLESSKIT_FLAGS`.

```bash
DOCKERD_ROOTLESS_ROOTLESSKIT_FLAGS="-p 0.0.0.0:2376:2376/tcp" \
  dockerd-rootless.sh \
  -H tcp://0.0.0.0:2376 \
  --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem
```

👉 Cela rend le démon Docker rootless accessible en TCP sécurisé (TLS obligatoire).

***

### 3. Exposer le socket Docker via **SSH**

Tu peux aussi accéder à l’API via **SSH** si `$DOCKER_HOST` est défini sur la machine distante :

```bash
ssh -l REMOTEUSER REMOTEHOST 'echo $DOCKER_HOST'
# Exemple de sortie : unix:///run/user/1001/docker.sock

docker -H ssh://REMOTEUSER@REMOTEHOST run ...
```

***

### 4. Autoriser les paquets **ping**

Sur certaines distributions, `ping` ne marche pas par défaut.\
Corrige ça en ajoutant à `/etc/sysctl.conf` ou `/etc/sysctl.d/` :

```ini
net.ipv4.ping_group_range = 0 2147483647
```

Puis recharge :

```bash
sudo sysctl --system
```

***

### 5. Exposer des **ports privilégiés (<1024)**

Par défaut, un utilisateur non-root ne peut pas ouvrir des ports <1024.\
Deux solutions :

#### Option 1 : donner la capacité à `rootlesskit`

```bash
sudo setcap cap_net_bind_service=ep $(which rootlesskit)
systemctl --user restart docker
```

#### Option 2 : autoriser tous les ports non-root

Ajouter dans `/etc/sysctl.conf` ou `/etc/sysctl.d/` :

```ini
net.ipv4.ip_unprivileged_port_start=0
```

Puis recharger :

```bash
sudo sysctl --system
```

***

### 6. Limiter les ressources (CPU, mémoire, PIDs)

Rootless Docker supporte les limitations **cgroups** uniquement avec :

* **cgroup v2** activé
* **systemd** comme gestionnaire de cgroup

Vérifie avec :

```bash
docker info | grep "Cgroup Driver"
```

* Si `systemd` → OK
* Si `none` → pas de support natif (fallback ulimit/cpulimit)

Exemple : vérifier les contrôleurs délégués :

```bash
cat /sys/fs/cgroup/user.slice/user-$(id -u).slice/user@$(id -u).service/cgroup.controllers
# Exemple : memory pids
```

Pour déléguer tous les contrôleurs (cpu, io, etc.) :

```bash
sudo mkdir -p /etc/systemd/system/user@.service.d
cat <<EOF | sudo tee /etc/systemd/system/user@.service.d/delegate.conf
[Service]
Delegate=cpu cpuset io memory pids
EOF
sudo systemctl daemon-reload
```

⚠️ Déléguer `cpuset` nécessite **systemd ≥ 244**.

***

### 7. Limiter les ressources **sans cgroup**

Si cgroups ne sont pas disponibles, tu peux utiliser :

* `ulimit`
* `cpulimit`

Mais ⚠️ ces limites sont **au niveau du processus**, pas au niveau du conteneur, et peuvent être contournées par les applis.

#### Exemples :

*   Limiter à **50% d’un CPU** (équivalent `--cpus 0.5`) :

    ```bash
    docker run <IMAGE> cpulimit --limit=50 --include-children <COMMAND>
    ```
*   Limiter mémoire à **64 MiB** (équivalent `--memory 64m`) :

    ```bash
    docker run <IMAGE> sh -c "ulimit -v 65536; <COMMAND>"
    ```
*   Limiter le nombre de processus à **100** :

    ```bash
    docker run --user 2000 --ulimit nproc=100 <IMAGE> <COMMAND>
    ```

***

✅ En résumé :

* Utilise `dind-rootless` si tu veux Docker-in-Docker en rootless.
* Expose le socket via TCP/SSH **avec TLS ou SSH** (jamais brut).
* Configure `ping`, `ports <1024`, et **cgroups v2 + systemd** pour un vrai contrôle des ressources.
* En fallback, `ulimit` et `cpulimit` peuvent servir.
