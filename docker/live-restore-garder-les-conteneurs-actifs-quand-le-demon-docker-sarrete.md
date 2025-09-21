# ⚡ Live restore — Garder les conteneurs actifs quand le démon Docker s’arrête

Par défaut, lorsque le **démon Docker** (`dockerd`) s’arrête (crash, redémarrage, mise à jour…), tous les conteneurs en cours d’exécution **sont arrêtés**.\
La fonctionnalité **live restore** permet de maintenir les conteneurs actifs même si le démon devient indisponible.

👉 Utile pour réduire les **temps d’arrêt** lors :

* d’un **redémarrage planifié** du démon,
* d’une **mise à jour**,
* ou d’un **crash du démon**.

***

### 🚫 Limitation

* ❌ Non supporté pour les **conteneurs Windows**.
* ✅ Fonctionne pour les **conteneurs Linux**, y compris sous **Docker Desktop pour Windows**.

## ⚡ Activer **Live Restore** dans Docker

Il existe **2 méthodes** pour activer la fonctionnalité **live restore**, qui permet de garder les conteneurs actifs même si le démon Docker (`dockerd`) s’arrête.\
👉 N’en utilisez **qu’une seule** (fichier de configuration **ou** flag au lancement).

***

### 1️⃣ Via le fichier de configuration `daemon.json` (recommandé ✅)

Sous Linux, le fichier est situé par défaut dans :

```
/etc/docker/daemon.json
```

Ajoutez (ou modifiez) la configuration suivante :

```json
{
  "live-restore": true
}
```

#### 🔄 Recharger la configuration sans arrêter les conteneurs

* Avec **systemd** :

```bash
sudo systemctl reload docker
```

* Sinon, envoyez un signal **SIGHUP** au processus `dockerd` :

```bash
sudo kill -SIGHUP $(pidof dockerd)
```

***

### 2️⃣ Via un flag au démarrage manuel de `dockerd` (⚠️ pas recommandé)

Vous pouvez démarrer le démon avec :

```bash
dockerd --live-restore
```

⚠️ Cette méthode **bypasse systemd** (ou tout autre gestionnaire de service), ce qui peut causer des comportements inattendus.

***

✅ **Conclusion** : La méthode la plus sûre est de configurer **`daemon.json`** puis de recharger le service avec `systemctl reload docker`.

## 🚀 Live Restore pendant les mises à jour de Docker

La fonctionnalité **Live Restore** permet de maintenir les conteneurs en cours d’exécution même lorsqu’on redémarre ou met à jour le démon **Docker (`dockerd`)**.\
👉 Mais il y a des **limitations importantes** à connaître lors des mises à jour.

***

### ✅ Cas supportés

* **Mises à jour mineures (patch releases)** :\
  Exemple :
  * Mise à jour de **25.0.1 → 25.0.2**
  * Mise à jour de **24.0.5 → 24.0.6**

➡️ Dans ce cas, les conteneurs restent actifs et le démon reprend correctement leur gestion après redémarrage.

***

### ⚠️ Cas non supportés

* **Mises à jour majeures (version de base change)** :\
  Exemple :
  * Passage de **24.0.x → 25.0.0**
  * Passage de **23.x → 25.x**

➡️ Ici, le démon peut **ne pas réussir à rétablir la connexion** avec les conteneurs.\
Dans ce cas :

* Docker **ne peut plus gérer** les conteneurs déjà en cours d’exécution.
* Vous devez **arrêter manuellement** ces conteneurs et les relancer après la mise à jour.

***

### 🔑 À retenir

* Utilisez **live-restore** pour réduire les interruptions lors des **redémarrages planifiés** ou des **mises à jour mineures**.
* ⚠️ Pour une **montée de version majeure**, il est préférable de :
  1. Arrêter proprement vos conteneurs.
  2. Mettre à jour Docker.
  3. Redémarrer vos conteneurs manuellement.

## 🔄 Live Restore au redémarrage du démon Docker

La fonctionnalité **live-restore** permet de garder les conteneurs actifs même si le démon Docker (`dockerd`) est redémarré.\
👉 Mais elle ne fonctionne **que sous certaines conditions**.

***

### ✅ Cas où Live Restore fonctionne

* Si le redémarrage du démon **ne modifie pas** ses **options de configuration critiques**.\
  Exemples :
* Redémarrage simple avec `systemctl restart docker`.
* Rechargement du démon avec `systemctl reload docker`.
* Mise à jour mineure du binaire Docker (sans changement de configuration réseau ou de stockage).

➡️ Les conteneurs continuent à tourner normalement et sont **repris en charge automatiquement** par Docker.

***

### ⚠️ Cas où Live Restore ne fonctionne pas

Si des **paramètres au niveau du démon** changent, la reprise automatique échoue.\
Exemples :

* Changement de l’adresse IP du **bridge par défaut** (`--bip` ou configuration `bridge`).
* Modification du **driver de stockage** (`overlay2`, `aufs`, etc.).
* Modification du répertoire de données (`--data-root`).
* Toute autre modification structurelle qui change la manière dont Docker gère les conteneurs.

➡️ Dans ce cas :

* Le démon ne peut **pas réassocier** les conteneurs en cours d’exécution.
* Vous devez **arrêter manuellement** ces conteneurs puis les relancer après redémarrage.

***

### 🔑 À retenir

* ✅ **Live Restore** est fiable pour les **redémarrages simples** ou les mises à jour mineures.
* ⚠️ Si vous modifiez la configuration réseau, stockage ou données, **Live Restore ne fonctionnera pas**.

## 📝 Impact du **Live Restore** sur les conteneurs en cours d’exécution

La fonctionnalité **live-restore** permet de garder les conteneurs actifs même si le démon Docker (`dockerd`) est arrêté.\
👉 Mais elle a aussi quelques effets secondaires importants à comprendre.

***

### ⚠️ Problème avec les logs

* Normalement, le démon Docker lit les logs des conteneurs via des **FIFO (pipes)**.
* Si le démon est **arrêté trop longtemps**, les **buffers de logs** peuvent se remplir.
* **Taille par défaut** : `64K`.

➡️ Conséquence :

* Quand le buffer est plein, le conteneur ne peut plus écrire de logs.
* Les processus à l’intérieur du conteneur continuent de tourner, mais **leurs sorties (stdout/stderr) sont bloquées**.

***

### ✅ Solution

* **Redémarrer le démon Docker** : cela permet de vider les buffers et de débloquer l’écriture des logs.

***

### 🔧 Ajuster la taille du buffer (Linux uniquement)

Tu peux modifier la taille maximale des buffers FIFO avec :

```bash
cat /proc/sys/fs/pipe-max-size   # Vérifier la taille actuelle (par défaut 65536 = 64K)
echo 1048576 | sudo tee /proc/sys/fs/pipe-max-size   # Passer à 1 Mo
```

👉 Cela réduit le risque de saturation si `dockerd` reste indisponible longtemps.

***

### 🚫 Limite sur Docker Desktop (Mac / Windows)

* Impossible de modifier cette valeur.
* Les utilisateurs de Docker Desktop doivent redémarrer Docker si les logs se bloquent.

***

💡 **Résumé** :

* Live Restore garde les conteneurs actifs sans démon.
* Mais si `dockerd` est arrêté trop longtemps, les **logs se bloquent après 64K**.
* Solution : **redémarrer le démon** ou **augmenter la taille du buffer (Linux seulement)**.

## 🐳 Live Restore et **Swarm mode**

La fonctionnalité **live-restore** n’a pas le même comportement en mode **Swarm** que pour des conteneurs classiques.

***

### 🔹 Live Restore = uniquement pour conteneurs **standalone**

* En mode normal (hors Swarm), `live-restore` garde les conteneurs en exécution même si le démon `dockerd` tombe.
* Cela permet de réduire les interruptions lors d’un crash ou d’une mise à jour du démon.

***

### 🔹 En **Swarm mode**

* Le mécanisme de **live-restore ne s’applique pas** aux **services Swarm**.
* Les services Swarm sont gérés par les **managers** du cluster.

👉 Fonctionnement :

* Si les **managers Swarm** deviennent indisponibles :
  * Les services **continuent à tourner** sur les nœuds workers (tant que les conteneurs déjà lancés sont en vie).
  * Mais tu **ne peux plus gérer le cluster** (ex. : mise à jour de services, scaling, déploiements, etc.).
* Dès qu’un quorum de managers est rétabli, la gestion reprend.

***

### ⚠️ Conséquence

* **Standalone containers** → protégés par live-restore.
* **Swarm services** → dépendent **exclusivement** de la disponibilité des managers.
* Donc, pour une haute disponibilité en Swarm, il est crucial d’avoir un **nombre impair de managers** (≥ 3) pour garantir un quorum.

***

💡 **Résumé rapide :**

* Live restore = ✅ standalone containers.
* Live restore = ❌ Swarm services (c’est le quorum des managers qui fait foi).
