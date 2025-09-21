---
description: >-
  Le daemon Docker (dockerd) est le processus principal qui fait tourner Docker.
  Il est généralement lancé automatiquement par le système d’exploitation.
---

# 🚀 Démarrer le daemon Docker

### 1️⃣ Démarrage via les utilitaires du système d’exploitation (recommandé)

Sur une installation classique, **le système gère automatiquement le démarrage du daemon**.\
Cela permet à Docker de redémarrer automatiquement après un reboot de la machine.

✅ Exemple avec **systemd** (Linux moderne, ex: Ubuntu, Debian, CentOS, Fedora) :

```bash
# Démarrer le service Docker
sudo systemctl start docker

# Activer Docker au démarrage du système
sudo systemctl enable docker

# Vérifier l’état du service
sudo systemctl status docker
```

***

### 2️⃣ Démarrage manuel (pour debug ou test)

Tu peux aussi démarrer le daemon **directement depuis le terminal** :

```bash
sudo dockerd
```

👉 Utile pour **tester une configuration** ou **lancer Docker en mode debug**.\
Exemple avec options supplémentaires :

```bash
sudo dockerd --debug --host tcp://192.168.1.10:2376
```

⚠️ Attention : en mode manuel, Docker **ne redémarrera pas automatiquement** après un reboot.

***

### 3️⃣ Selon le système

* **Linux** → `systemctl` (systemd) ou `service docker start` (si SysVinit)
* **macOS & Windows** → le daemon est lancé automatiquement par **Docker Desktop**
* **Rootless mode (Linux)** → démarrage via `systemctl --user start docker`

***

✅ En pratique : sur un serveur Linux, la méthode **systemctl** est la plus fiable.\
La commande directe (`dockerd`) est surtout utile pour **debugger**.

## 🚀 Démarrer Docker avec **systemd**

Sur la plupart des distributions modernes (Ubuntu, Debian, Fedora, CentOS, etc.), le daemon Docker est géré par **systemd**.

### ▶️ Démarrer Docker manuellement

```bash
sudo systemctl start docker
```

👉 Cette commande démarre le service **docker.service** immédiatement.

***

### 🔄 Vérifier si Docker tourne

```bash
sudo systemctl status docker
```

Exemple de sortie attendue :

```
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2025-09-20 10:15:32 UTC; 2min ago
```

***

### ⚡ Redémarrer ou arrêter Docker

```bash
# Redémarrer Docker
sudo systemctl restart docker

# Arrêter Docker
sudo systemctl stop docker
```

***

### 🖥️ Activer Docker au démarrage du système

Pour que Docker démarre **automatiquement après un reboot** :

```bash
sudo systemctl enable docker
```

👉 Pour désactiver l’auto-démarrage :

```bash
sudo systemctl disable docker
```

***

✅ En résumé :

* `start` → lancer Docker maintenant
* `status` → voir l’état
* `restart` → relancer
* `stop` → arrêter
* `enable` → activer au boot
* `disable` → désactiver au boot

## ▶️ Démarrer le démon **manuellement**

Si tu ne veux pas utiliser un utilitaire système (comme `systemd`) ou que tu veux **tester rapidement**, tu peux lancer le démon Docker directement avec la commande :

```bash
dockerd
```

👉 Selon ta configuration système, tu pourrais avoir besoin de **sudo** :

```bash
sudo dockerd
```

***

### 🔎 Exemple de sortie lors du lancement

```
INFO[0000] +job init_networkdriver()
INFO[0000] +job serveapi(unix:///var/run/docker.sock)
INFO[0000] Listening for HTTP on unix (/var/run/docker.sock)
```

* Le démon démarre en **premier plan**
* Les logs sont affichés **directement dans ton terminal**
* Docker écoute par défaut sur le socket **`/var/run/docker.sock`**

***

### ⏹️ Arrêter le démon

Puisqu’il tourne dans ton terminal au **premier plan**, il suffit de faire :

👉 `Ctrl + C`

Cela **arrête immédiatement le démon Docker**.

***

⚠️ Attention : lancer `dockerd` manuellement est utile pour **tester ou déboguer**, mais pour un usage en production ou au quotidien, il vaut mieux utiliser un **service système** (`systemctl start docker`) afin d’assurer :

* un redémarrage automatique après crash ou reboot,
* une gestion centralisée des logs,
* une sécurité renforcée.
