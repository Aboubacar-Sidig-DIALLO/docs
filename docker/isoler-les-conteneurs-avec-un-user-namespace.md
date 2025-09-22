# 🔐 Isoler les conteneurs avec un user namespace

Les **namespaces Linux** permettent d’isoler les processus en limitant leur accès aux ressources du système, sans que ces processus aient conscience de ces restrictions.\
👉 Référence : Linux namespaces.

***

### 🚨 Risque : escalade de privilèges dans les conteneurs

La meilleure défense contre une **élévation de privilèges** dans un conteneur est de **ne jamais exécuter les applications avec root** à l’intérieur du conteneur.

* ⚡ Mais certains conteneurs nécessitent que leurs processus tournent en tant que **root dans le conteneur**.

➡️ Dans ce cas, Docker permet de **re-mapper l’utilisateur root du conteneur vers un utilisateur non privilégié de l’hôte** grâce aux **user namespaces**.

***

### 🛡️ Fonctionnement du _user namespace_

* Docker crée un **mapping UID/GID** entre le conteneur et l’hôte.
* Dans le conteneur :
  * Les UIDs vont de `0` à `65536` (où `0` = root dans le conteneur).
* Sur l’hôte :
  * Ces UIDs/GIDs correspondent à des **utilisateurs non privilégiés**.
  * Exemple : le `root` du conteneur (`UID 0`) est mappé à un utilisateur comme `UID 100000` sur l’hôte.

👉 Résultat : même si un processus malveillant prend le contrôle de `root` dans le conteneur, **il reste un utilisateur sans privilèges sur l’hôte**.

***

### ✅ Avantages du user namespace

* Empêche qu’un root dans le conteneur soit root sur l’hôte.
* Réduit la surface d’attaque en cas de faille noyau ou Docker.
* Compatible avec d’autres mécanismes de sécurité (seccomp, AppArmor, capabilities).

## 👤 Remapping et IDs subordonnés (subuid / subgid) dans Docker

Quand on active le mécanisme **user namespace remapping** dans Docker, la gestion des identités se fait à travers deux fichiers critiques :

* `/etc/subuid` → pour les utilisateurs (UIDs)
* `/etc/subgid` → pour les groupes (GIDs)

***

### 🔑 Exemple d’entrée dans `/etc/subuid`

```
testuser:231072:65536
```

➡️ Cela signifie :

* **Utilisateur concerné** : `testuser`
* **Plage UID sur l’hôte** : de `231072` à `296607` (65536 IDs)
* **Mapping dans le conteneur** :
  * UID `0` (root dans le conteneur) = `231072` sur l’hôte
  * UID `1` (dans le conteneur) = `231073` sur l’hôte
  * etc.

👉 Résultat : si un processus malveillant devient `root` dans le conteneur, il reste **un simple utilisateur non privilégié** sur l’hôte.

***

### ⚠️ Points importants

1. **Plages multiples possibles**
   * On peut attribuer plusieurs plages par utilisateur (non chevauchantes).
   * Limite : Linux permet max **5 mappings** par processus (`/proc/self/uid_map` et `/proc/self/gid_map`).
   * Docker n’utilise que les 5 premières.
2. **Utilisateur `dockremap` par défaut**
   * Si on configure `userns-remap=default`, Docker crée automatiquement un utilisateur et groupe `dockremap`.
   * Il reçoit une plage définie dans `/etc/subuid` et `/etc/subgid`.
3. **Attention aux distributions**
   * Certaines distribs n’ajoutent pas automatiquement les entrées nécessaires dans `/etc/subuid` et `/etc/subgid`.
   * Dans ce cas, il faut les éditer **manuellement**, sans chevauchement.
4. **Association obligatoire avec un utilisateur réel**
   * Même si c’est un détail d’implémentation, chaque plage doit être associée à un **compte existant** (dans `/etc/passwd` et `/etc/group`).
   *   Vérifiable avec :

       ```bash
       id testuser
       ```
5. **Conséquences sur `/var/lib/docker/`**
   * Quand on active `userns-remap`, Docker **masque** les couches/images/objets existants, car il doit recréer la hiérarchie avec les nouveaux UID/GID mappés.
   * 👉 À activer de préférence sur une **installation Docker neuve**.
   * Si on le désactive ensuite, on ne peut plus accéder aux ressources créées pendant qu’il était actif.
6. **Accès aux volumes / bind mounts**
   * Le remapping complique l’accès aux fichiers montés depuis l’hôte :
     * L’utilisateur mappé peut ne pas avoir les droits sur les chemins bindés.
     * Solution : adapter les permissions sur l’hôte pour l’utilisateur mappé (`chown` / `chmod`).

***

### ✅ Procédure pratique

1.  Vérifier que l’utilisateur existe (ex. `dockremap` ou `testuser`) :

    ```bash
    id testuser
    ```
2.  Vérifier / créer les entrées dans `/etc/subuid` et `/etc/subgid` :

    ```
    testuser:231072:65536
    ```
3.  Configurer Docker dans `/etc/docker/daemon.json` :

    ```json
    {
      "userns-remap": "testuser"
    }
    ```

    ou bien laisser Docker créer `dockremap` automatiquement :

    ```json
    {
      "userns-remap": "default"
    }
    ```
4.  Redémarrer Docker :

    ```bash
    sudo systemctl restart docker
    ```

## 🔐 Activer **userns-remap** dans Docker Daemon

L’option **userns-remap** permet d’isoler davantage les conteneurs en remappant l’utilisateur `root` d’un conteneur vers un utilisateur non privilégié sur l’hôte.

***

### 🛠️ 1. Activer avec un flag (temporaire)

Tu peux lancer le démon Docker directement avec :

```bash
dockerd --userns-remap="testuser:testuser"
```

👉 Ici, `root` dans le conteneur = `testuser` (UID/GID) sur l’hôte.

***

### 🛠️ 2. Activer via `daemon.json` (recommandé ✅)

1. Ouvrir le fichier de config :

```bash
sudo nano /etc/docker/daemon.json
```

2. Ajouter (si le fichier est vide, sinon compléter) :

```json
{
  "userns-remap": "testuser"
}
```

3. Valeurs possibles :
   * `"testuser"` → nom d’utilisateur
   * `"testuser:testuser"` → utilisateur + groupe
   * `"1001"` → UID
   * `"1001:1001"` → UID:GID
   * `"testuser:1001"` ou `"1001:testuser"` → mixte

👉 Si tu veux que Docker crée automatiquement l’utilisateur `dockremap`, mets :

```json
{
  "userns-remap": "default"
}
```

4. Redémarrer Docker :

```bash
sudo systemctl restart docker
```

***

### 🛠️ 3. Vérifier la création des utilisateurs/groupes

Si tu utilises `dockremap` :

```bash
id dockremap
```

Exemple attendu :

```
uid=112(dockremap) gid=116(dockremap) groups=116(dockremap)
```

Vérifie les fichiers `/etc/subuid` et `/etc/subgid` :

```bash
grep dockremap /etc/subuid
dockremap:231072:65536

grep dockremap /etc/subgid
dockremap:231072:65536
```

👉 Si les entrées n’existent pas, ajoute-les **à la main** (sans chevauchement de plages).\
Exemple :

```
dockremap:231072:65536
```

***

### 🛠️ 4. Vérifier le fonctionnement

1. Vérifie que les anciennes images ne sont plus visibles :

```bash
docker image ls
```

➡️ La liste doit être vide (car un nouvel espace de stockage est utilisé).

2. Lance un conteneur de test :

```bash
docker run hello-world
```

3. Vérifie le répertoire mappé :

```bash
sudo ls -ld /var/lib/docker/231072.231072/
```

Exemple attendu :

```
drwx------ 11 231072 231072 11 Jun 21 21:19 /var/lib/docker/231072.231072/
```

À l’intérieur :

```bash
sudo ls -l /var/lib/docker/231072.231072/
```

Exemple :

```
drwx------ 5 231072 231072 5 Jun 21 21:19 aufs
drwx------ 3 231072 231072 3 Jun 21 21:21 containers
drwx------ 3 root   root   3 Jun 21 21:19 image
drwxr-x--- 3 root   root   3 Jun 21 21:19 network
...
```

👉 Les dossiers appartenant à `231072:231072` sont ceux utilisés par le **userns-remap**.

***

✅ **Conclusion** :\
Avec `userns-remap`, même si un attaquant obtient `root` dans le conteneur, il n’est qu’un **utilisateur non privilégié** (ex. UID `231072`) sur l’hôte.

## 🚫 Désactiver le **namespace remapping** pour un conteneur

Quand **userns-remap** est activé au niveau du démon Docker, **tous les conteneurs** héritent de ce mécanisme par défaut.\
Mais parfois (ex. pour un conteneur **privilégié** ou avec des besoins particuliers), tu peux vouloir le **désactiver uniquement pour un conteneur**.

***

### 🛠️ 1. Utilisation de `--userns=host`

Pour créer ou lancer un conteneur **sans remapping**, ajoute :

```bash
docker run --rm -it --userns=host ubuntu bash
```

Ou encore :

```bash
docker container create --userns=host myimage
docker container start -ai <container_id>
```

👉 Cela force le conteneur à utiliser **les namespaces utilisateurs de l’hôte**, et non ceux définis par `--userns-remap`.

***

### ⚠️ Effet secondaire important

Même si le conteneur n’est **pas remappé**, ses **couches de système de fichiers (read-only layers)** sont **partagées** avec les autres conteneurs qui, eux, utilisent le remapping.\
Résultat :

* Le système de fichiers entier du conteneur appartient **à l’UID/GID configuré dans `--userns-remap`** (ex. `231072` dans ton cas).
* Cela peut causer des comportements inattendus :
  * `sudo` ne fonctionne pas (il attend que ses binaires appartiennent à `root` réel, UID 0).
  * Les binaires **setuid** (comme `passwd`, `ping`, etc.) ne fonctionnent pas correctement.

***

✅ **Conclusion** :\
Tu peux désactiver le remapping avec `--userns=host` pour un conteneur spécifique, mais attention :

* tu perds la protection du remapping pour ce conteneur,
* tu risques des effets secondaires liés aux permissions du FS partagé.

## ⚠️ Limitations connues des **user namespaces** dans Docker

Activer le **remapping d’UID/GID** avec `userns-remap` renforce la sécurité, mais cela introduit aussi plusieurs limitations importantes. Voici un résumé clair et détaillé :

***

### 🚫 Fonctionnalités Docker incompatibles

1. **Partage de namespaces PID ou NET avec l’hôte**
   * `--pid=host` ❌
   * `--network=host` ❌\
     Ces options nécessitent un accès direct à l’hôte, ce qui est incompatible avec l’isolation fournie par les user namespaces.

***

2. **Drivers externes de volumes ou de stockage**
   * Certains **drivers de volume/storage** ne savent pas gérer les remappings d’UID/GID.
   * Résultat : problèmes d’accès aux fichiers ou impossibilité d’utiliser certains volumes.

***

3. **Mode privilégié `--privileged`**
   * Normalement, `docker run --privileged` donne presque tous les privilèges à un conteneur.
   * Avec `userns-remap`, cela ne marche pas **sauf si tu ajoutes `--userns=host`**.

***

### ⚙️ Restrictions techniques liées au kernel

Même si l’utilisateur `root` **dans le conteneur** a (en apparence) les privilèges de superutilisateur, le noyau **sait qu’il est dans un user namespace**.\
👉 Résultat : certaines actions restent interdites, par ex. :

* **`mknod`** : impossible de créer un périphérique dans le conteneur → `Permission denied`.
* D’autres opérations sensibles liées au noyau peuvent aussi être bloquées.

***

### 📦 Volumes et montages

Quand tu montes un **volume depuis l’hôte** :

* Les fichiers du volume appartiennent à des UID/GID réels de l’hôte.
* Pour que le conteneur puisse lire/écrire correctement, tu dois **préparer les permissions en amont** (adapter `chown`/`chmod` sur l’hôte avec les UID/GID remappés).
* Sinon → accès refusé dans le conteneur.

***

### ✅ Bonnes pratiques

* Activer `userns-remap` **au début d’une installation Docker**, pas sur une instance déjà utilisée (sinon tes images/volumes existants deviennent inaccessibles).
* Utiliser des volumes **préconfigurés avec les bons UID/GID**.
* Si un conteneur a besoin de `--privileged`, ajoute **aussi** `--userns=host`.
* Tester les workloads avant migration pour identifier les incompatibilités.
