# 🐳 Rootless Docker – Dépannage (Troubleshooting)

### 🔹 Cas particulier : **Ubuntu ≥ 24.04**

Depuis Ubuntu 24.04, les **espaces de noms utilisateurs non privilégiés** (unprivileged user namespaces) sont restreints par défaut.\
👉 Cela empêche certains processus non-root (comme Rootless Docker) de créer des user namespaces, **sauf si un profil AppArmor l’autorise**.

#### Cas 1 : installation via **paquet officiel DEB**

```bash
sudo apt-get install docker-ce-rootless-extras
```

✅ L’AppArmor profile pour `rootlesskit` est **déjà fourni** avec le paquet → aucune config manuelle nécessaire.

***

#### Cas 2 : installation via **script d’installation**

Dans ce cas, il faut **ajouter un profil AppArmor manuellement** pour `rootlesskit`.

1.  Créer un profil AppArmor basé sur ton chemin `$HOME/bin/rootlesskit` :

    ```bash
    filename=$(echo $HOME/bin/rootlesskit | sed -e s@^/@@ -e s@/@.@g)
    cat <<EOF > ~/${filename}
    ```

abi \<abi/4.0>,\
include \<tunables/global>

"$HOME/bin/rootlesskit" flags=(unconfined) {\
userns,

include if exists \<local/${filename}>\
}\
EOF

````

2. Déplacer le profil dans `/etc/apparmor.d/` :
```bash
sudo mv ~/${filename} /etc/apparmor.d/${filename}
````

3.  Redémarrer AppArmor :

    ```bash
    sudo systemctl restart apparmor.service
    ```

***

### 🔹 Autres distributions (exemples rapides)

* **Arch Linux** → utilise souvent `systemd` + `cgroup v2`, vérifier que `apparmor` ou `selinux` n’empêchent pas les userns.
* **openSUSE / SLES** → nécessite souvent `sysctl` pour activer userns (`kernel.unprivileged_userns_clone=1`).
* **CentOS / RHEL / Fedora** → vérifier la configuration SELinux et éventuellement ajouter des règles `setsebool` ou profils équivalents.

***

👉 En résumé :

* Sur **Ubuntu ≥ 24.04** : soit tu installes via `apt` et tout est prêt ✅, soit tu installes via script → il faut **ajouter un profil AppArmor** pour autoriser `rootlesskit`.
* Sur d’autres distributions : le problème viendra plutôt de **cgroup v2** ou de **SELinux/AppArmor** à configurer.

## 🚫 Limitations connues de Docker Rootless Mode

### 🔹 Pilotes de stockage supportés

Seuls certains **storage drivers** fonctionnent en mode rootless :

* **overlay2** → uniquement avec **kernel ≥ 5.11**
* **fuse-overlayfs** → uniquement avec **kernel ≥ 4.18** et `fuse-overlayfs` installé
* **btrfs** → uniquement avec **kernel ≥ 4.18** ou si `~/.local/share/docker` est monté avec l’option `user_subvol_rm_allowed`
* **vfs** → supporté mais **lent** (usage plutôt pour tests/dépannage)

***

### 🔹 Gestion des cgroups

* Supporté uniquement avec **cgroup v2** et **systemd**.\
  👉 Voir la section sur limitation des ressources.

***

### 🔹 Fonctionnalités **non supportées**

* **AppArmor** (pas géré en rootless)
* **Checkpoint / Restore** (CRIU)
* **Overlay network**
* **Exposition de ports SCTP**

***

### 🔹 Particularités réseau

* `ping` : ne marche pas par défaut → il faut appliquer la configuration décrite dans **Routing ping packets**.
* Ports privilégiés **< 1024** (ex. 80, 443) → nécessitent des réglages particuliers (voir **Exposing privileged ports**).
* L’adresse IP affichée par `docker inspect` est **namespacée** dans l’espace réseau de **RootlessKit** → elle n’est **pas directement joignable depuis l’hôte** (sauf si tu fais un `nsenter` dans le namespace réseau).
* Le mode `--net=host` est aussi isolé dans le namespace réseau de **RootlessKit**, donc ce n’est pas le vrai réseau de l’hôte.

***

### 🔹 Autres limitations

* **NFS** comme `data-root` (par défaut `~/.local/share/docker`) **n’est pas supporté** → problème général Docker, pas spécifique au rootless.

***

👉 En résumé :\
Le mode rootless est **plus sécurisé** (pas de root), mais il sacrifie certaines fonctionnalités réseau (overlay, ports < 1024, IP non directement accessible), la compatibilité AppArmor, et impose des contraintes sur les pilotes de stockage.

## 🐳 Dépannage Docker Rootless Mode

### 1. ⚡ Installation avec systemd

Erreur :

```bash
dockerd-rootless-setuptool.sh install
[INFO] systemd not detected...
```

👉 Cela arrive si tu utilises `sudo su` pour basculer d’utilisateur.\
➡️ Solution : utilise **machinectl** (fourni par `systemd-container`) :

```bash
sudo machinectl shell myuser@
```

***

### 2. ⚡ Erreurs au démarrage du démon

#### ❌ `operation not permitted`

```bash
[rootlesskit:parent] error: failed to start the child: fork/exec /proc/self/exe: operation not permitted
```

👉 Vérifie `unprivileged_userns_clone` :

```bash
cat /proc/sys/kernel/unprivileged_userns_clone
```

Si `0` → activer :

```bash
echo "kernel.unprivileged_userns_clone=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl --system
```

***

#### ❌ `no space left on device`

```bash
[rootlesskit:parent] error: failed to start the child: fork/exec /proc/self/exe: no space left on device
```

👉 Vérifie `max_user_namespaces` :

```bash
cat /proc/sys/user/max_user_namespaces
```

Si `0` → activer :

```bash
echo "user.max_user_namespaces=28633" | sudo tee -a /etc/sysctl.conf
sudo sysctl --system
```

***

#### ❌ `No subuid ranges found`

👉 Vérifie `/etc/subuid` et `/etc/subgid`.\
Exemple attendu :

```
testuser:231072:65536
```

***

#### ❌ `could not get XDG_RUNTIME_DIR`

* **Sans systemd** → définir manuellement :

```bash
export XDG_RUNTIME_DIR=$HOME/.docker/xrd
rm -rf $XDG_RUNTIME_DIR && mkdir -p $XDG_RUNTIME_DIR
dockerd-rootless.sh
```

⚠️ Il faut supprimer ce dossier à chaque logout.

* **Avec systemd** → connexion via PAM systemd (`ssh`, console graphique, `machinectl shell`).

***

### 3. ⚡ Problème systemctl user

Erreur :

```bash
systemctl --user start docker
Failed to connect to bus: No such file or directory
```

👉 Solution : se connecter avec PAM systemd :

* Connexion graphique
* `ssh <user>@localhost`
* `machinectl shell <user>@`

***

### 4. ⚡ Démarrage automatique

Si le démon ne démarre pas au boot :

```bash
sudo loginctl enable-linger $(whoami)
```

***

### 5. ⚡ Erreurs docker pull

#### ❌ `lchown <FILE>: invalid argument`

→ pas assez d’UID/GID dispo dans `/etc/subuid` ou `/etc/subgid`.\
👉 Ajouter **65 536 entrées** minimum.

#### ❌ `operation not permitted`

→ `~/.local/share/docker` sur **NFS**.\
👉 Déplacer `data-root` :

```json
{
  "data-root": "/path/local"
}
```

***

### 6. ⚡ Erreurs docker run

#### ❌ `OCI runtime create failed ... connection reset by peer`

Cause → `dbus` non actif en cgroup v2.\
👉 Installer dbus utilisateur :

```bash
sudo apt-get install -y dbus-user-session
# ou Fedora
sudo dnf install -y dbus-daemon
systemctl --user enable --now dbus
```

#### ❌ `--cpus`, `--memory`, `--pids-limit` ignorés

→ attendu si cgroup v1.\
👉 Activer **cgroup v2** + systemd.

***

### 7. ⚡ Réseau en rootless mode

RootlessKit gère le réseau → selon driver choisi :

| Driver réseau | Driver port  | Débit net | Débit ports | IP propagée | No SUID | Notes                    |
| ------------- | ------------ | --------- | ----------- | ----------- | ------- | ------------------------ |
| slirp4netns   | builtin      | Lent      | ✅ Rapide    | ❌           | ✅       | défaut                   |
| vpnkit        | builtin      | Lent      | ✅ Rapide    | ❌           | ✅       | si slirp absent          |
| slirp4netns   | slirp4netns  | Lent      | Lent        | ✅           | ✅       | complet                  |
| pasta         | implicit     | Lent      | ✅ Rapide    | ✅           | ✅       | expérimental             |
| lxc-user-nic  | builtin      | ✅ Rapide  | ✅ Rapide    | ❌           | ❌       | expérimental             |
| bypass4netns  | bypass4netns | ✅ Rapide  | ✅ Rapide    | ✅           | ✅       | nécessite seccomp custom |

***

#### 🔹 Cas fréquents

* `docker run -p 80:80 ...` ❌ → port < 1024 interdit → utiliser 8080 ou activer `net.ipv4.ip_unprivileged_port_start=0`.
* `ping` ne marche pas → configurer `ping_group_range`.
* `docker inspect` donne une IP inaccessible → normal, utiliser `docker run -p`.
* `--net=host` ne marche pas → attendu, car isolé dans RootlessKit.
* Réseau lent → installer `slirp4netns` ou utiliser `pasta` (expérimental).

***

### 8. ⚡ Debug avancé

Entrer dans les namespaces dockerd :

```bash
nsenter -U --preserve-credentials -n -m -t $(cat $XDG_RUNTIME_DIR/docker.pid)
```

## 🐳 Désinstallation de Docker Rootless

#### 1. Supprimer le service systemd utilisateur

Exécute :

```bash
dockerd-rootless-setuptool.sh uninstall
```

Ce script :

* **stoppe** le service `docker.service`
* **désactive** le démarrage auto
* supprime le lien dans `~/.config/systemd/user/`

👉 Exemple de sortie :

```
+ systemctl --user stop docker.service
+ systemctl --user disable docker.service
Removed /home/testuser/.config/systemd/user/default.target.wants/docker.service.
[INFO] Uninstalled docker.service
```

⚠️ Cela **ne supprime pas** les binaires ni les données Docker.

***

#### 2. Supprimer les données (conteneurs, images, volumes)

Par défaut stockées dans `~/.local/share/docker`\
Exécute :

```bash
/usr/bin/rootlesskit rm -rf ~/.local/share/docker
```

***

#### 3. Nettoyer les variables d’environnement

Si tu avais ajouté dans ton `~/.bashrc` ou `~/.zshrc` :

```bash
export PATH=/usr/bin:$PATH
export DOCKER_HOST=unix:///run/user/1000/docker.sock
```

👉 Supprime ces lignes et recharge ton shell :

```bash
source ~/.bashrc
```

***

#### 4. Supprimer les binaires

* **Si installé via package manager (apt, dnf, yum, etc.)**\
  Supprime :

```bash
sudo apt remove -y docker-ce-rootless-extras
```

ou

```bash
sudo dnf remove -y docker-ce-rootless-extras
```

* **Si installé via script (**[**https://get.docker.com/rootless**](https://get.docker.com/rootless)**)**\
  Supprime les binaires placés dans `~/bin` :

```bash
cd ~/bin
rm -f containerd containerd-shim containerd-shim-runc-v2 ctr docker dockerd rootlesskit vpnkit slirp4netns
```

***

✅ Après ça, ton environnement est propre :

* Plus de service rootless systemd
* Plus de données locales
* Plus de binaires rootless
