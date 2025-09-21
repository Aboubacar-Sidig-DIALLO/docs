# ⚙️ Aperçu de la configuration du démon Docker (dockerd)

Le **démon Docker** (`dockerd`) est le processus principal qui gère :

* Les **conteneurs**
* Les **images**
* Les **volumes**
* Les **réseaux**
* Les **API Docker**

Tu peux personnaliser son comportement avec des **options**, soit :

1. **En ligne de commande** lors du lancement (`dockerd --option valeur`)
2. **Via un fichier de configuration JSON** (`/etc/docker/daemon.json` sur Linux)

***

### 📌 1. Fichier de configuration (`daemon.json`)

👉 C’est la méthode recommandée car elle est **persistante** (contrairement aux options en CLI qui disparaissent après un redémarrage).\
Exemple minimal de `/etc/docker/daemon.json` :

```json
{
  "debug": true,
  "experimental": true,
  "insecure-registries": ["myregistry.local:5000"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "data-root": "/mnt/docker-data",
  "bip": "192.168.1.5/24"
}
```

***

### 📌 2. Options courantes de configuration

| Option                       | Description                                                                             |
| ---------------------------- | --------------------------------------------------------------------------------------- |
| `--debug` ou `"debug": true` | Active le mode debug (logs détaillés).                                                  |
| `--data-root`                | Change l’emplacement du stockage Docker (images, volumes, conteneurs).                  |
| `--exec-opts`                | Configure le pilote de cgroups (`systemd` ou `cgroupfs`).                               |
| `--bip`                      | Définit le sous-réseau par défaut du bridge Docker (`docker0`).                         |
| `--insecure-registries`      | Liste des registres non sécurisés (HTTP).                                               |
| `--registry-mirror`          | Définit un miroir pour accélérer les pulls d’images.                                    |
| `--log-driver`               | Choisit le driver de logs (`json-file`, `syslog`, `journald`, `gelf`, `fluentd`, etc.). |
| `--log-opts`                 | Paramètres du driver de logs (ex: rotation, taille max).                                |
| `--iptables`                 | Active/désactive la gestion des règles iptables par Docker.                             |
| `--ip-forward`               | Active/désactive l’IP forwarding.                                                       |
| `--cgroup-parent`            | Définit un groupe parent cgroups pour organiser les ressources.                         |
| `--experimental`             | Active les fonctionnalités expérimentales.                                              |

***

### 📌 3. Emplacement du fichier `daemon.json`

* **Linux** : `/etc/docker/daemon.json`
* **Windows Server** : `C:\ProgramData\docker\config\daemon.json`
* **macOS / Docker Desktop** : via l’interface graphique (non directement ce fichier).

***

### 📌 4. Recharger la configuration

Après modification de `daemon.json` :

```bash
sudo systemctl restart docker
```

⚠️ Attention : Tous les conteneurs redémarreront si tu relances `dockerd`.

***

### 📌 5. Vérifier la configuration active

👉 Pour voir la configuration utilisée par `dockerd` :

```bash
docker info
```

ou

```bash
ps aux | grep dockerd
```

## ⚙️ Configurer le démon Docker (`dockerd`)

Il existe **deux méthodes principales** pour configurer `dockerd` :

***

### 📝 1. Fichier de configuration JSON (méthode recommandée ✅)

* Permet de **centraliser toutes les options** dans un seul fichier.
* Sur **Linux**, le fichier se trouve par défaut dans :

```
/etc/docker/daemon.json
```

Exemple de configuration :

```json
{
  "debug": true,
  "experimental": true,
  "exec-opts": ["native.cgroupdriver=systemd"],
  "insecure-registries": ["myregistry.local:5000"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "data-root": "/mnt/docker-data"
}
```

Ensuite, redémarrer Docker pour appliquer les changements :

```bash
sudo systemctl restart docker
```

***

### 🚀 2. Utiliser les flags en ligne de commande

* Tu peux lancer `dockerd` avec des **options directes** :

```bash
dockerd --debug --experimental \
  --data-root /mnt/docker-data \
  --log-driver json-file \
  --insecure-registry myregistry.local:5000
```

***

### ⚠️ Attention : ne pas mélanger doublons

Tu peux **combiner fichier JSON + flags CLI**.\
👉 Mais si une option est définie **dans les deux**, `dockerd` ne démarrera pas et affichera une erreur.

Exemple **à éviter** :

* `"debug": true` dans `daemon.json`
* et `--debug` en flag

***

✅ En résumé :

* Utilise **daemon.json** pour une configuration propre et persistante.
* Complète avec des **flags temporaires** si besoin ponctuel.
* ⚠️ Ne duplique pas les mêmes options dans les deux.

## 📂 Localisation du fichier `daemon.json`

Le démon Docker s’attend à trouver le fichier **par défaut** aux emplacements suivants :

| **OS / Mode**             | **Chemin attendu**                         |
| ------------------------- | ------------------------------------------ |
| **Linux (classique)**     | `/etc/docker/daemon.json`                  |
| **Linux (rootless mode)** | `~/.config/docker/daemon.json`             |
| **Windows**               | `C:\ProgramData\docker\config\daemon.json` |

***

### 🔑 Particularité du mode rootless

* Le démon respecte la variable **`XDG_CONFIG_HOME`**.
* Si elle est définie, alors le fichier attendu est :

```
$XDG_CONFIG_HOME/docker/daemon.json
```

Exemple : si `XDG_CONFIG_HOME=/home/alex/configs`, alors le fichier sera :

```
/home/alex/configs/docker/daemon.json
```

***

### 🎯 Spécifier un chemin personnalisé

Tu peux aussi indiquer **explicitement** l’emplacement du fichier lors du démarrage du démon avec :

```bash
dockerd --config-file /chemin/personnalisé/mon-daemon.json
```

***

👉 Ensuite, pour la **liste complète des options disponibles** (logs, stockage, réseau, sécurité…), il faut se référer à la **documentation de référence de `dockerd`**.

## ⚙️ Configuration du Docker Daemon avec des **flags**

Tu peux lancer le démon **manuellement** avec des options en ligne de commande.\
C’est pratique pour le **dépannage** ou les tests rapides, mais en production on préfère `daemon.json` pour centraliser la config.

***

#### ✅ Exemple avec plusieurs flags

```bash
dockerd --debug \
  --tls=true \
  --tlscert=/var/docker/server.pem \
  --tlskey=/var/docker/serverkey.pem \
  --host tcp://192.168.59.3:2376
```

#### 📌 Explications

* `--debug` → Active le mode débogage (logs détaillés).
* `--tls=true` → Active la sécurisation TLS pour les connexions.
* `--tlscert=/var/docker/server.pem` → Spécifie le certificat serveur TLS.
* `--tlskey=/var/docker/serverkey.pem` → Spécifie la clé privée TLS.
* `--host tcp://192.168.59.3:2376` → Définit l’adresse/port sur laquelle le démon écoute (ici, sur le réseau en TCP).\
  Par défaut, Docker écoute sur l’Unix socket `/var/run/docker.sock`.

***

#### 🔎 Obtenir la liste complète des options disponibles

Deux solutions :

1. Consulter la doc officielle : **dockerd reference**
2. Ou directement en CLI :

```bash
dockerd --help
```

***

⚠️ **Attention** :

* Si tu utilises **`daemon.json` ET des flags** pour **la même option**, Docker ne démarre pas (conflit).
* Les flags **écrasent** les configs du fichier uniquement s’ils ne se chevauchent pas.

## 📂 Répertoire de données du Docker Daemon

Le **daemon Docker (`dockerd`)** stocke **toutes les données persistantes** dans un seul répertoire.\
C’est **là où Docker conserve l’état complet** de ton environnement.

***

#### 📌 Contenu du répertoire

* **Images** téléchargées (`docker pull`)
* **Conteneurs** créés (`docker run`)
* **Volumes** (`docker volume`)
* **Secrets** (dans Swarm)
* **Configurations de services** (Swarm/Kubernetes)

En clair : c’est le **cœur de Docker**.\
Si tu perds ce répertoire → tu perds tout ton état local Docker.

***

#### 📍 Emplacement par défaut

* **Linux** → `/var/lib/docker`
* **Windows** → `C:\ProgramData\docker`

***

#### 🔧 Modifier l’emplacement

Tu peux changer l’emplacement via la configuration du daemon :

👉 Dans `/etc/docker/daemon.json` (Linux) :

```json
{
  "data-root": "/mnt/docker-data"
}
```

👉 Puis redémarrer Docker :

```bash
sudo systemctl restart docker
```

***

#### ⚠️ Points importants

1. **Un daemon = un répertoire dédié**\
   Si tu fais pointer deux daemons sur le même dossier (ex. un partage NFS), tu auras des **corruptions difficiles à diagnostiquer**.
2. **Sauvegarde possible**\
   Tu peux sauvegarder le répertoire pour restaurer ton environnement Docker (images, conteneurs, volumes, etc.).
3. **Bonnes pratiques**
   * Utiliser un disque séparé ou rapide (SSD/NVMe) si tu gères beaucoup de conteneurs.
   * Sur les serveurs, rediriger vers une partition avec assez d’espace disque.
