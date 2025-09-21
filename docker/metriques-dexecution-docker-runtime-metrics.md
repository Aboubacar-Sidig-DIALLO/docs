# 📊 Métriques d’exécution Docker (Runtime Metrics)

### 🔹 1. `docker stats`

La commande `docker stats` permet de **surveiller en temps réel** la consommation des conteneurs.\
Elle affiche :

* **CPU %** → pourcentage d’utilisation CPU par conteneur
* **MEM USAGE / LIMIT** → mémoire utilisée / limite mémoire
* **MEM %** → pourcentage de mémoire utilisée par rapport à la limite
* **NET I/O** → trafic réseau entrant/sortant
* **BLOCK I/O** → lecture/écriture disque (I/O bloc)

👉 Exemple :

```bash
docker stats redis1 redis2
```

Résultat :

```
CONTAINER   CPU %   MEM USAGE / LIMIT   MEM %   NET I/O        BLOCK I/O
redis1      0.07%   796 KB / 64 MB      1.21%   788 B / 648 B  3.568 MB / 512 KB
redis2      0.07%   2.7 MB / 64 MB      4.29%   1.2 KB / 648 B 12.4 MB / 0 B
```

👉 Très pratique pour un **suivi en direct**, comme `top` pour les processus.

***

### 🔹 2. Cgroups (Control Groups)

Sous Linux, les **cgroups** sont utilisés pour :

* **Limiter** les ressources (CPU, mémoire, disque, réseau)
* **Surveiller** la consommation
* **Isoler** les processus dans des hiérarchies

📌 Les cgroups exposent leurs métriques via un **pseudo-système de fichiers** :

```
/sys/fs/cgroup/
```

On y trouve plusieurs sous-dossiers : `cpu`, `memory`, `blkio`, etc.

👉 Pour voir où les cgroups sont montés :

```bash
grep cgroup /proc/mounts
```

***

### 🔹 3. cgroup v1 vs v2

* **cgroup v1** : hiérarchies séparées (CPU, mémoire, I/O…).
* **cgroup v2** : hiérarchie unifiée, introduite par systemd.

📌 Vérification :

```bash
ls /sys/fs/cgroup/cgroup.controllers
```

* Si présent → v2.
* Sinon → v1.

📌 Activation cgroup v2 (avec systemd) :

```bash
sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=1"
```

(sinon modifier `/etc/default/grub` puis `sudo update-grub`).

> ⚠️ Changer de version nécessite un **redémarrage complet**.

***

### 🔹 4. Associer un conteneur à ses cgroups

Chaque conteneur a un répertoire de cgroup :

*   v1 (driver `cgroupfs`) :

    ```
    /sys/fs/cgroup/memory/docker/<longid>/
    ```
*   v1 (driver `systemd`) :

    ```
    /sys/fs/cgroup/memory/system.slice/docker-<longid>.scope/
    ```
*   v2 (driver `cgroupfs`) :

    ```
    /sys/fs/cgroup/docker/<longid>/
    ```
*   v2 (driver `systemd`) :

    ```
    /sys/fs/cgroup/system.slice/docker-<longid>.scope/
    ```

👉 `<longid>` = l’ID long du conteneur (`docker inspect`).

***

### 🔹 5. Exemple : Métriques mémoire (`memory.stat`)

Dans un cgroup mémoire, le fichier `memory.stat` donne des infos détaillées :

```txt
cache 11492564992
rss 1930993664
mapped_file 306728960
pgpgin 406632648
pgpgout 403355412
swap 0
pgfault 728281223
pgmajfault 1724
...
```

#### Les champs principaux :

* **cache** → mémoire utilisée comme cache disque (lecture/écriture fichiers, tmpfs)
* **rss** → mémoire "pure" non liée au disque (heap, stack, mmap anonyme)
* **mapped\_file** → mémoire issue de fichiers mappés
* **pgfault** → nombre de _page faults_ (accès mémoire nécessitant allocation)
* **pgmajfault** → _page faults majeurs_ (nécessitant accès disque)
* **swap** → espace swap utilisé
* **active\_anon / inactive\_anon** → mémoire anonyme active/inactive
* **active\_file / inactive\_file** → cache fichiers actif/inactif
* **unevictable** → mémoire non expulsable (ex: clés cryptographiques via `mlock`)
* **hierarchical\_memory\_limit** → limite mémoire physique du cgroup
* **hierarchical\_memsw\_limit** → limite RAM + swap

📌 Les versions avec préfixe `total_` incluent aussi les sous-cgroups.

***

### 🔹 6. Différence "gauge" vs "counter"

* **Gauge** → valeur instantanée (peut monter/descendre).
  * Ex : `rss`, `swap`
* **Counter** → compteur d’événements (seulement croissant).
  * Ex : `pgfault`, `pgmajfault`

***

### 🚀 En résumé

* `docker stats` = vision rapide en temps réel (CPU, RAM, I/O).
* **cgroups** = mécanisme Linux de suivi + limitation des ressources.
* **v1 vs v2** : différence d’organisation, mais même finalité.
* Les fichiers comme `memory.stat` donnent un suivi **granulaire** (page faults, swap, cache).

## 📊 Métriques CPU, I/O et Réseau dans Docker

### 🔹 1. CPU Metrics (`cpuacct.stat`)

Les métriques CPU d’un conteneur se trouvent dans le contrôleur **cpuacct** :

```
/sys/fs/cgroup/cpuacct/docker/<container_id>/cpuacct.stat
```

Contenu typique :

```txt
user 120
system 45
```

* **user** → temps CPU passé en mode utilisateur (exécution directe du code de l’app).
* **system** → temps CPU passé en mode noyau (exécution des appels système pour le processus).

📌 Mesuré en **jiffies** (ticks de 1/100 s sur x86).\
👉 100 jiffies = 1 seconde.

Exemple :

* `user 120` = 1,2 s de CPU en mode user
* `system 45` = 0,45 s de CPU en mode noyau

***

### 🔹 2. Block I/O Metrics (`blkio`)

Les métriques disque se trouvent dans le contrôleur **blkio**.\
Fichiers principaux :

* **blkio.sectors**\
  → nombre total de secteurs (512 octets) lus/écrits par conteneur.
* **blkio.io\_service\_bytes**\
  → nombre total de **bytes lus/écrits**, différenciés par :
  * lecture synchrone
  * écriture synchrone
  * lecture asynchrone
  * écriture asynchrone
* **blkio.io\_serviced**\
  → nombre total d’**opérations I/O** (peu importe la taille).
* **blkio.io\_queued**\
  → nombre d’opérations **en file d’attente** pour ce cgroup.\
  (⚠️ une file vide ≠ conteneur inactif, car les I/O synchrones ne passent pas par la queue).

👉 Ces infos permettent d’identifier **quel conteneur stresse le disque**.

***

### 🔹 3. Network Metrics

Contrairement au CPU et à la mémoire, les **cgroups ne fournissent pas directement de métriques réseau**.\
Raison : les interfaces réseau dépendent des **network namespaces**. Un conteneur peut avoir plusieurs interfaces (`eth0`, `lo`, …), donc le suivi doit se faire au niveau **namespace**.

#### 📌 Solutions possibles

1.  **iptables**\
    Ajouter une règle qui compte les paquets :

    ```bash
    iptables -I OUTPUT -p tcp --sport 80
    iptables -nxvL OUTPUT
    ```

    Cela donne nombre de paquets et d’octets envoyés.
2.  **Counters d’interfaces réseau**\
    Chaque conteneur a une interface `vethXXXX`.\
    Vérifier les compteurs TX/RX avec :

    ```bash
    ip netns exec <container_ns> netstat -i
    ```

    Exemple de préparation :

    ```bash
    CID=<container_id>
    TASKS=/sys/fs/cgroup/devices/docker/$CID*/tasks
    PID=$(head -n 1 $TASKS)

    mkdir -p /var/run/netns
    ln -sf /proc/$PID/ns/net /var/run/netns/$CID

    ip netns exec $CID netstat -i
    ```

    Cela affiche les stats réseau depuis **l’espace réseau du conteneur**.
3. **Collectd / Exporters**\
   Des outils comme **collectd** ou **cAdvisor** automatisent la collecte réseau pour Docker.

***

### 🔹 4. Collecte haute performance

* Démarrer un processus **persistant** (plutôt que relancer une commande à chaque fois).
* Utiliser l’appel système **`setns()`** pour basculer un processus dans un namespace réseau et collecter les métriques.
* ⚠️ Attention : ne pas garder le descripteur de namespace ouvert en permanence (risque de fuite réseau).

***

### 🔹 5. Collecte à la fin de vie du conteneur

Si on veut connaître la **consommation totale** d’un conteneur après son arrêt :

1. Lancer un **processus de monitoring** et le placer dans le même cgroup (`tasks`).
2. Suivre les métriques régulièrement.
3. Quand le conteneur s’arrête → ce processus devient le dernier du cgroup.
4. Il collecte les dernières métriques (CPU, mémoire, I/O, réseau).
5. Il se replace dans le cgroup root et supprime le cgroup du conteneur (`rmdir`).

***

## 🚀 En résumé

* **CPU** : `cpuacct.stat` → temps user/system en jiffies.
* **I/O Bloc** : `blkio.*` → secteurs, octets, opérations, files d’attente.
* **Réseau** : pas via cgroups → utiliser `iptables`, `netstat` via `ip netns`, ou collectd.
* On peut suivre en **temps réel** (`docker stats`, cAdvisor) ou en **fin de vie** du conteneur (via un processus dédié dans son cgroup).
