---
description: >-
  Lorsque tu rencontres des problèmes avec le démon Docker (plantages, lenteur,
  comportements étranges…), tu disposes de plusieurs méthodes de diagnostic.
---

# 🐳 Dépannage du démon Docker (dockerd)

### 🔍 Étapes de dépannage principales

#### 1. **Activer le mode debug**

Ajoute `"debug": true` dans ton fichier `daemon.json` :

```json
{
  "debug": true
}
```

Puis recharge la configuration :

```bash
sudo kill -SIGHUP $(pidof dockerd)
```

👉 Le mode debug fournit beaucoup plus de détails dans les logs, très utile pour comprendre ce que fait Docker.

***

#### 2. **Vérifier les logs du démon**

Selon ton système :

* **Linux (systemd)** :

```bash
journalctl -u docker.service
```

* **Linux (ancien système)** :

```bash
tail -f /var/log/docker.log
tail -f /var/log/daemon.log
```

* **macOS (Docker Desktop)** :

```bash
tail -f ~/Library/Containers/com.docker.docker/Data/log/vm/dockerd.log
```

* **Windows (WSL2)** :

```powershell
Get-Content $env:LOCALAPPDATA\Docker\log\vm\dockerd.log -Wait
```

***

#### 3. **Forcer un dump de stack trace**

Si le démon devient **bloqué ou ne répond plus**, envoie un signal :

* **Linux** :

```bash
sudo kill -SIGUSR1 $(pidof dockerd)
```

* **Windows Server** : utiliser `docker-signal` avec l’option `--pid=<PID>`.

👉 Cela force Docker à écrire un **stack trace complet** dans ses logs ou dans un fichier temporaire (`/var/run/docker/goroutine-stacks-*.log`).

***

#### 4. **Analyser les stack traces**

Un stack trace Go montre :

* l’état de chaque **goroutine** (équivalent des threads),
* si Docker est bloqué sur une **opération système**,
* les **deadlocks** éventuels.

***

#### 5. **Cas particuliers**

* Si tu utilises **Swarm** → assure-toi que les nœuds managers sont bien en quorum.
* Si tu utilises **IPv6, proxys ou runtimes alternatifs** → vérifie tes configurations dans `/etc/docker/daemon.json`.
* Pour des problèmes de **réseau**, teste avec :

```bash
docker network inspect bridge
docker run busybox ping 8.8.8.8
```

***

✅ En résumé :

* Active le **debug** pour plus de logs.
* Consulte les **logs système**.
* Si bloqué → force un **stack trace** avec `SIGUSR1`.
* Analyse les traces pour identifier blocages ou erreurs système.

## 🐳 Erreur : **"Unable to connect to the Docker daemon"**

Quand tu vois le message :

```
Cannot connect to the Docker daemon. Is 'docker daemon' running on this host?
```

cela veut dire que **ton client Docker (`docker`) n’arrive pas à communiquer avec le démon (`dockerd`)**.

***

### 🚦 Causes possibles et solutions

#### 1. **Le démon Docker n’est pas lancé**

Vérifie son état :

```bash
docker info
```

ou avec systemd :

```bash
sudo systemctl is-active docker
sudo systemctl status docker
```

S’il est arrêté, lance-le :

```bash
sudo systemctl start docker
```

👉 Pour qu’il démarre automatiquement au boot :

```bash
sudo systemctl enable docker
```

***

#### 2. **Problème de socket Unix**

Docker communique par défaut via le socket `/var/run/docker.sock`.

Vérifie qu’il existe et que tu as les bons droits :

```bash
ls -l /var/run/docker.sock
```

Si tu vois :

```
srw-rw---- 1 root docker ...
```

👉 Assure-toi que ton utilisateur appartient bien au groupe **docker** :

```bash
sudo usermod -aG docker $USER
```

Puis reconnecte-toi.

***

#### 3. **Mauvaise configuration de l’hôte**

Ton client peut être configuré pour se connecter à un **hôte distant** via `DOCKER_HOST`.

Vérifie :

```bash
env | grep DOCKER_HOST
```

* Si la variable est vide → Docker tente de se connecter au démon local.
* Si elle pointe ailleurs → assure-toi que l’hôte est accessible.
* Pour la réinitialiser :

```bash
unset DOCKER_HOST
```

Et enlève la ligne correspondante de `~/.bashrc` ou `~/.profile`.

***

#### 4. **Conflit de configuration (`daemon.json` vs flags)**

Si tu définis des **options dans `/etc/docker/daemon.json`** _et_ en ligne de commande (ou via systemd), Docker ne démarre pas.

Exemple d’erreur :

```
unable to configure the Docker daemon with file /etc/docker/daemon.json:
the following directives are specified both as a flag and in the configuration file: hosts
```

👉 Solution :

* supprime le `-H` des scripts systemd (Debian/Ubuntu par ex.) :

Crée `/etc/systemd/system/docker.service.d/docker.conf` :

```ini
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
```

Puis recharge systemd et redémarre Docker :

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

***

#### 5. **Cas spécifiques**

* **Docker Desktop (Mac/Windows)** : tu ne peux pas modifier `hosts` dans `daemon.json`.
* **Remote API (2375/2376)** : si tu veux te connecter à un démon distant, configure `hosts` proprement et ouvre le port dans ton firewall.

***

### ✅ Résumé rapide (checklist)

1.  Vérifie si Docker tourne :

    ```bash
    sudo systemctl status docker
    ```
2.  Vérifie l’accès au socket :

    ```bash
    ls -l /var/run/docker.sock
    ```
3.  Vérifie si tu utilises `DOCKER_HOST` :

    ```bash
    env | grep DOCKER_HOST
    ```
4. Vérifie les conflits entre `daemon.json` et systemd.

## 🐳 Troubleshooting Docker : **Out of Memory (OOM) et compatibilité noyau**

***

### 🚨 1. Out of Memory (OOM)

Quand un container ou le démon Docker consomme **plus de mémoire que disponible**, le **OOM Killer du kernel** peut tuer :

* soit ton container,
* soit `dockerd` lui-même.

👉 **Solutions :**

*   **Vérifie les ressources dispo** :

    ```bash
    free -h
    top -o %MEM
    ```
* **Alloue plus de mémoire à ton hôte** (VM/serveur).
*   **Ajoute des limites mémoire aux containers** :

    ```bash
    docker run -m 512m --memory-swap 1g nginx
    ```

    Ici :

    * `-m 512m` → limite la RAM à 512 Mo
    * `--memory-swap 1g` → permet 512 Mo de swap max en plus

⚠️ Si tu ne définis pas de limites, un container peut consommer **toute la RAM du host**.

***

### 🧩 2. Compatibilité du noyau

Docker **nécessite un noyau ≥ 3.10** avec certains modules activés.

👉 Vérifie ta version de noyau :

```bash
uname -r
```

👉 Télécharge et lance le script officiel de vérification :

```bash
curl -fsSL https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh -o check-config.sh
bash ./check-config.sh
```

Le script te dit si ton noyau est compatible et quels modules manquent.

***

### 🔄 3. Support des **cgroups swap limit** (Ubuntu/Debian)

Par défaut tu peux voir un warning :

```
WARNING: Your kernel does not support swap limit capabilities. Limitation discarded.
```

Si tu veux activer la **limite mémoire + swap** :

1.  Édite **/etc/default/grub**\
    Ajoute dans `GRUB_CMDLINE_LINUX` :

    ```bash
    GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"
    ```
2.  Mets à jour GRUB :

    ```bash
    sudo update-grub
    ```
3.  Redémarre la machine :

    ```bash
    sudo reboot
    ```

👉 Attention :

* Cela ajoute environ **1 % de perte mémoire** disponible,
* et environ **10 % de baisse de performance globale**, même si Docker ne tourne pas.

***

✅ **Résumé pratique :**

* Mets des **limites mémoire** sur tes containers.
* Vérifie que ton **kernel ≥ 3.10** avec `check-config.sh`.
* Active `swapaccount=1` si tu veux contrôler swap + RAM.

## 🐳 Docker Networking : **Problèmes de transfert IP (IP forwarding)**

***

### 🚨 Le problème

Si tu configures ton réseau **manuellement avec systemd-networkd** (systemd ≥ **219**), les containers Docker peuvent ne pas avoir accès au réseau.

En effet :

* Depuis **systemd 220**, l’option `net.ipv4.conf.<interface>.forwarding` est **désactivée par défaut**.
* Cela empêche le **forwarding IP**, alors que Docker s’appuie dessus pour permettre aux containers de communiquer avec l’extérieur.

👉 Résultat : les containers sont isolés et ne peuvent pas accéder au réseau externe.

***

### 🛠️ La solution : activer l’IP forwarding

1.  Identifie ton interface réseau sur l’hôte (par ex. `eth0`, `ens33`, …) :

    ```bash
    ip addr
    ```
2.  Édite son fichier de configuration systemd-networkd :

    ```bash
    sudo nano /usr/lib/systemd/network/80-container-eth0.network
    ```
3.  Ajoute dans la section `[Network]` :

    ```ini
    [Network]
    ...
    IPForward=kernel
    # OU
    IPForward=true
    ```

    🔹 `IPForward=kernel` → suit la valeur du noyau (`/proc/sys/net/ipv4/ip_forward`)\
    🔹 `IPForward=true` → force l’activation du forwarding
4.  Recharge systemd-networkd :

    ```bash
    sudo systemctl restart systemd-networkd
    ```
5.  Vérifie que l’IP forwarding est bien activé :

    ```bash
    sysctl net.ipv4.ip_forward
    ```

    Doit renvoyer `1`.

***

### ✅ Résumé

* Depuis systemd 220, `IPForward` est désactivé → containers Docker bloqués.
* Ajoute `IPForward=kernel` ou `IPForward=true` dans ton fichier `.network`.
* Redémarre `systemd-networkd` pour appliquer.

## 🐳 Docker Networking : **Problèmes de DNS Resolver**

***

### 🚨 Le problème

Sur Linux, certains environnements de bureau utilisent un **gestionnaire réseau** (comme NetworkManager) qui active **dnsmasq** pour :

* mettre en cache les requêtes DNS,
* et gérer DHCP.

Dans ce cas, `/etc/resolv.conf` pointe souvent vers une **adresse loopback** (`127.0.0.1` ou `127.0.1.1`).

👉 Problème :

* Les containers Docker utilisent **leur propre espace réseau**.
* Ils interprètent `127.0.0.1` comme **eux-mêmes** → or ils n’exécutent pas de serveur DNS local.
* Résultat : les containers **ne peuvent pas résoudre les noms de domaine internes**.

⚠️ Docker détecte ce cas et affiche un avertissement :

```
WARNING: Local (127.0.0.1) DNS resolver found in resolv.conf and containers
can't use it. Using default external servers : [8.8.8.8 8.8.4.4]
```

***

### 🔍 Vérification

Teste si `dnsmasq` tourne sur ta machine :

```bash
ps aux | grep dnsmasq
```

***

### 🛠️ Solutions possibles

#### 🔹 1. Spécifier un DNS pour Docker

Tu peux indiquer explicitement les serveurs DNS que Docker doit utiliser.

👉 Dans `/etc/docker/daemon.json` :

```json
{
  "dns": ["8.8.8.8", "8.8.4.4"]
}
```

Puis redémarre Docker :

```bash
sudo systemctl restart docker
```

👉 Tu peux aussi spécifier un DNS **au lancement d’un container** :

```bash
docker run --rm --dns=192.168.1.1 alpine nslookup google.com
```

***

#### 🔹 2. Désactiver dnsmasq

Si tu n’as pas besoin du cache DNS local, tu peux **désactiver dnsmasq**.\
Cela forcera `/etc/resolv.conf` à utiliser directement les serveurs DNS réels.

* Pour **NetworkManager** :

```bash
sudo nano /etc/NetworkManager/NetworkManager.conf
```

Et désactive dnsmasq :

```ini
[main]
dns=default
```

Puis redémarre :

```bash
sudo systemctl restart NetworkManager
```

***

### ✅ Résumé

* **Cause** : `/etc/resolv.conf` utilise `127.0.0.1` avec dnsmasq → inaccessible depuis les containers.
* **Solution 1** : définir un DNS valide dans `daemon.json` ou via `--dns`.
* **Solution 2** : désactiver dnsmasq pour forcer l’usage de DNS externes.

## 🐳 Docker – Spécifier des serveurs DNS

Par défaut, Docker utilise les DNS de ton système hôte (fichier `/etc/resolv.conf`). Si ce fichier pointe vers `127.0.0.1` (dnsmasq, systemd-resolved, etc.), les containers ne pourront pas résoudre les noms de domaine.

👉 Solution : définir explicitement les serveurs DNS dans la configuration du **daemon Docker**.

***

### 🔹 Étape 1 : Éditer le fichier de configuration du daemon

Ouvre le fichier `/etc/docker/daemon.json` :

```bash
sudo nano /etc/docker/daemon.json
```

Ajoute (ou modifie) la clé `"dns"` :

```json
{
  "dns": ["8.8.8.8", "8.8.4.4"]
}
```

* Ici on utilise les DNS publics de Google.
* Tu peux ajouter aussi ton DNS interne (exemple : `192.168.1.1`) avant Google DNS :

```json
{
  "dns": ["192.168.1.1", "8.8.8.8", "8.8.4.4"]
}
```

***

### 🔹 Étape 2 : Redémarrer Docker

Applique la nouvelle configuration :

```bash
sudo service docker restart
```

ou avec systemd :

```bash
sudo systemctl restart docker
```

***

### 🔹 Étape 3 : Vérifier la résolution DNS

Teste avec une image publique :

```bash
docker pull hello-world
```

Vérifie aussi la résolution **interne** avec un container léger :

```bash
docker run --rm -it alpine ping -c4 <mon_hôte_interne>
```

Exemple :

```
PING google.com (192.168.1.2): 56 data bytes
64 bytes from 192.168.1.2: seq=0 ttl=41 time=7.597 ms
64 bytes from 192.168.1.2: seq=1 ttl=41 time=7.635 ms
64 bytes from 192.168.1.2: seq=2 ttl=41 time=7.660 ms
64 bytes from 192.168.1.2: seq=3 ttl=41 time=7.677 ms
```

***

✅ **Résultat attendu** :

* Les containers résolvent correctement les noms de domaine publics (Docker Hub, Internet).
* Ils peuvent aussi résoudre les noms internes si tu as ajouté ton DNS d’entreprise/local.

## 🐳 Docker – Désactiver **dnsmasq** dans NetworkManager (Ubuntu, RHEL, CentOS, Fedora)

Si tu ne veux pas modifier la configuration DNS de Docker (`daemon.json`), tu peux **désactiver `dnsmasq`** pour éviter qu’il injecte `127.0.0.1` dans `/etc/resolv.conf` (ce qui empêche les containers de résoudre les noms de domaine).

***

### 🔹 Étape 1 : Éditer la configuration de NetworkManager

Ouvre le fichier de configuration :

```bash
sudo nano /etc/NetworkManager/NetworkManager.conf
```

Cherche la ligne suivante :

```
dns=dnsmasq
```

Commente-la en ajoutant un `#` :

```
# dns=dnsmasq
```

***

### 🔹 Étape 2 : Sauvegarder et fermer

Appuie sur **CTRL+O**, puis **Entrée**, et **CTRL+X** pour fermer `nano`.

***

### 🔹 Étape 3 : Redémarrer les services

Redémarre **NetworkManager** puis **Docker** :

```bash
sudo systemctl restart network-manager
sudo systemctl restart docker
```

👉 Tu peux aussi simplement **redémarrer ta machine**.

***

### 🔹 Étape 4 : Vérifier

Lance un container de test et fais un ping :

```bash
docker run --rm alpine ping -c4 google.com
```

Tu dois obtenir une réponse sans erreur de résolution DNS ✅.

***

⚡ **Remarque** :

* Désactiver `dnsmasq` supprime son cache DNS (résolutions plus lentes).
* Si tu veux garder `dnsmasq` _mais_ rendre Docker compatible, il vaut mieux **configurer le daemon avec `"dns": [...]`** dans `/etc/docker/daemon.json`.

## 🐳 Résoudre les problèmes de **réseaux Docker qui disparaissent**

Il peut arriver que le réseau **docker0** ou tes réseaux personnalisés disparaissent ou se comportent de façon instable.\
Souvent, c’est dû à des outils de gestion réseau du système hôte (comme **NetworkManager**, **systemd-networkd**, ou **netscript**) qui modifient ou suppriment les interfaces créées par Docker.

Voici comment corriger ce problème 👇

***

### 🔹 1. Vérifier si `netscript` est installé

Certains systèmes Debian/Ubuntu installent `netscript` qui entre en conflit avec Docker.\
Supprime-le si c’est le cas :

```bash
sudo apt-get remove netscript-2.4
```

***

### 🔹 2. Dire à ton gestionnaire réseau de **ne pas gérer Docker**

#### ✅ Avec **NetworkManager**

Crée un fichier pour indiquer que l’interface `docker0` doit être ignorée :

```bash
sudo nano /etc/network/interfaces.d/20-docker0
```

Ajoute :

```
iface docker0 inet manual
```

Cela empêche NetworkManager de prendre la main sur l’interface **docker0** (mais pas sur tes réseaux personnalisés).

➡️ Redémarre NetworkManager :

```bash
sudo systemctl restart NetworkManager
```

➡️ Vérifie que l’interface est bien en **unmanaged** :

```bash
nmcli device
```

Tu devrais voir quelque chose comme :

```
DEVICE    TYPE      STATE         CONNECTION
docker0   bridge    unmanaged     --
```

***

#### ✅ Avec **systemd-networkd**

Si tu utilises **systemd-networkd**, tu dois créer un fichier pour indiquer que `docker0` est **unmanaged**.\
Par exemple dans `/etc/systemd/network/10-docker0.network` :

```
[Match]
Name=docker0

[Network]
Unmanaged=yes
```

Puis recharge la config :

```bash
sudo systemctl restart systemd-networkd
```

***

### 🔹 3. Cas particulier : **Netplan**

Si ton système utilise **Netplan** (Ubuntu récent), tu peux définir une configuration personnalisée pour que Netplan **ignore Docker**.\
Exemple dans `/etc/netplan/01-netcfg.yaml` :

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
  bridges:
    docker0:
      dhcp4: no
      optional: true
```

Puis applique :

```bash
sudo netplan apply
```

***

✅ Après ces changements, Docker sera le **seul à gérer ses interfaces**, et tu n’auras plus de réseaux qui disparaissent à cause d’outils externes.

## 🐳 Empêcher **Netplan** de réécrire la configuration réseau Docker

Sur certains systèmes (souvent Ubuntu installés via **cloud-init**), **Netplan** peut réécrire ou supprimer la configuration des interfaces Docker (`docker0`, réseaux personnalisés, etc.).\
Pour éviter cela, il faut appliquer une **configuration personnalisée**.

***

### 🔹 Étapes pour empêcher Netplan d’écraser Docker

#### 1. Définir les interfaces Docker comme _unmanaged_

Suis d’abord la procédure pour **déclarer `docker0` comme unmanaged** avec NetworkManager ou systemd-networkd :

➡️ Exemple avec NetworkManager :

```bash
sudo nano /etc/network/interfaces.d/20-docker0
```

```ini
iface docker0 inet manual
```

***

#### 2. Créer un fichier Netplan personnalisé

Crée ou édite un fichier de configuration dans `/etc/netplan/50-cloud-init.yml` :

```bash
sudo nano /etc/netplan/50-cloud-init.yml
```

Ajoute (et ajuste en fonction de ton interface réseau réelle, ici `en*`) :

```yaml
network:
  ethernets:
    all:
      dhcp4: true
      dhcp6: true
      match:
        # adapte ce filtre au nom de tes interfaces physiques (ex: ens33, eth0, eno1…)
        name: en*
  renderer: networkd
  version: 2
```

👉 Ce fichier dit à Netplan de gérer uniquement les interfaces correspondant à `en*` (par ex. `ens33`, `enp0s3`) et **pas les interfaces Docker (`docker0`, `br-xxxx`, etc.)**.

***

#### 3. Appliquer la configuration Netplan

Recharge Netplan avec :

```bash
sudo netplan apply
```

⚠️ Attention : une erreur de syntaxe ou une mauvaise configuration peut casser ta connexion réseau.\
Si ça arrive, tu peux revenir en arrière en restaurant le fichier précédent et réappliquer Netplan.

***

#### 4. Redémarrer Docker

Relance Docker pour qu’il recrée ses interfaces si besoin :

```bash
sudo systemctl restart docker
```

***

#### 5. Vérifier que les interfaces Docker sont ignorées

Utilise `networkctl` pour vérifier l’état :

```bash
networkctl
```

Tu devrais voir les interfaces Docker (`docker0`, `br-xxx`) listées comme **unmanaged** ✅.

## 🐳 Volumes – Erreur _Unable to remove filesystem_

Cette erreur se produit souvent lorsqu’un autre conteneur **monte le répertoire système de Docker** (`/var/lib/docker`) et garde des fichiers ouverts à l’intérieur.\
Un cas typique est l’utilisation de **Google cAdvisor**, qui a besoin de ce montage pour fonctionner.

***

### 🔹 Exemple d’erreur

```bash
Error: Unable to remove filesystem for
74bef250361c7817bee19349c93139621b272bc8f654ae112dd4eb9652af9515:
remove /var/lib/docker/containers/74bef250361c7817bee19349c93139621b272bc8f654ae112dd4eb9652af9515/shm:
Device or resource busy
```

👉 Cela arrive car `/var/lib/docker/` est monté dans un autre conteneur (ex. cAdvisor) et celui-ci garde des **descripteurs de fichiers ouverts** (`statfs` ou `fstatfs`), empêchant Docker de supprimer les fichiers liés.

***

### 🔹 Pourquoi ça arrive ?

* `/var/lib/docker/` contient **tous les fichiers des conteneurs Docker** (systèmes de fichiers, volumes, etc.).
*   Si un conteneur (comme cAdvisor) **bind-mount** ce répertoire :

    ```bash
    --volume=/var/lib/docker/:/var/lib/docker:ro
    ```

    → alors il accède aux systèmes de fichiers des autres conteneurs.
* Quand Docker essaie de supprimer un conteneur, les fichiers restent occupés par cAdvisor → **suppression impossible**.

***

### 🔹 Comment identifier le processus bloquant ?

Utilise `lsof` pour voir quel processus garde le fichier ouvert :

```bash
sudo lsof /var/lib/docker/containers/<ID>/shm
```

***

### 🔹 Solution / contournement

1.  **Arrêter le conteneur qui monte `/var/lib/docker`** (ex. cAdvisor) :

    ```bash
    docker stop cadvisor
    ```
2.  **Supprimer le conteneur bloqué** normalement :

    ```bash
    docker rm <ID>
    ```
3. Relancer cAdvisor ensuite si nécessaire.

***

✅ **Bonne pratique** :\
Il est déconseillé de bind-mount directement `/var/lib/docker/` (sauf si c’est strictement nécessaire, comme avec cAdvisor).\
Si tu utilises d’autres outils de monitoring/logging, privilégie leurs intégrations **sans montage de `/var/lib/docker`**.

