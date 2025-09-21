# 📖 Lire les logs du démon Docker (dockerd)

Les logs du démon Docker sont essentiels pour **diagnostiquer des problèmes** (plantages, erreurs réseau, images corrompues, etc.). Leur emplacement dépend du système d’exploitation et de la configuration du logging.

***

### 📍 Emplacements des logs selon l’OS

| **Système**                             | **Emplacement / Commande**                                                                                                                                              |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Linux**                               | <p><code>journalctl -xu docker.service</code><br>ou <code>tail -f /var/log/syslog</code> (Debian/Ubuntu)<br>ou <code>tail -f /var/log/messages</code> (CentOS/RHEL)</p> |
| **macOS (dockerd)**                     | `~/Library/Containers/com.docker.docker/Data/log/vm/dockerd.log`                                                                                                        |
| **macOS (containerd)**                  | `~/Library/Containers/com.docker.docker/Data/log/vm/containerd.log`                                                                                                     |
| **Windows (WSL2 - dockerd)**            | `%LOCALAPPDATA%\Docker\log\vm\dockerd.log`                                                                                                                              |
| **Windows (WSL2 - containerd)**         | `%LOCALAPPDATA%\Docker\log\vm\containerd.log`                                                                                                                           |
| **Windows (Windows containers natifs)** | Journaux accessibles via **Windows Event Log** (`eventvwr.msc`)                                                                                                         |

***

### 🖥️ Exemple pratique (Linux avec systemd)

Pour suivre les logs en direct :

```bash
sudo journalctl -fu docker.service
```

***

### 🍏 Exemple pratique (macOS)

Pour suivre les logs Docker Desktop :

```bash
tail -f ~/Library/Containers/com.docker.docker/Data/log/vm/dockerd.log
```

📌 Exemple de sortie :

```
2021-07-28T10:21:21Z dockerd time="2021-07-28T10:21:21.497642089Z" level=debug msg="attach: stdout: begin"
2021-07-28T10:21:21Z dockerd time="2021-07-28T10:21:21.527074928Z" level=debug msg="Calling POST /v1.41/containers/.../start"
```

***

### 🔍 Points clés

* Utilise **`level=debug`** pour obtenir plus de détails si ton `dockerd` est lancé avec `--debug`.
* Sur **Linux avec systemd**, `journalctl` est la source principale.
* Sur **macOS et Windows**, Docker Desktop redirige les logs vers des fichiers dans le répertoire utilisateur.

🐳 Enable debugging in Docker Daemon

Activer le mode **debug** te permet d’avoir des logs beaucoup plus détaillés pour diagnostiquer les problèmes liés à Docker.

***

### ✅ Méthode recommandée : via `daemon.json`

1. Édite (ou crée) le fichier de configuration :

* **Linux** → `/etc/docker/daemon.json`
* **Windows/macOS (Docker Desktop)** → via **Settings > Docker Engine**

2. Ajoute la clé suivante :

```json
{
  "debug": true
}
```

⚠️ Si ton fichier contient déjà d’autres clés, veille à :

* Ajouter une **virgule** avant `"debug": true` si ce n’est pas la dernière entrée.
* Vérifier que la clé `"log-level"` (si présente) est bien à `"info"` ou `"debug"`.

Exemple complet :

```json
{
  "log-level": "debug",
  "debug": true
}
```

***

3. Recharge la configuration sans arrêter tes conteneurs (Linux uniquement) :

```bash
sudo kill -SIGHUP $(pidof dockerd)
```

Sur **Windows/macOS**, il faut **redémarrer Docker Desktop**.

***

### 🔄 Méthode alternative : démarrage manuel avec `-D`

Tu peux aussi démarrer le démon manuellement avec l’option **`-D`** :

```bash
dockerd -D
```

⚠️ Attention :

* Cette méthode **remplace le lancement systemd** ou Docker Desktop → l’environnement peut différer (répertoires, variables).
* À utiliser surtout pour du **debug ponctuel**.

## 🐳 Forcer un stack trace dans Docker Daemon

Si ton **démon Docker (`dockerd`) devient non réactif**, tu peux forcer l’écriture d’un **stack trace complet** (état des threads et goroutines internes).\
Cela permet de **diagnostiquer un blocage** ou un problème interne.

***

### 🔧 Sur Linux

1. Récupère le PID du démon `dockerd` :

```bash
pidof dockerd
```

2. Envoie le signal **SIGUSR1** au processus :

```bash
sudo kill -SIGUSR1 $(pidof dockerd)
```

3. Résultat :

* Le démon **ne s’arrête pas**.
* Le stack trace est écrit dans les **logs Docker** :
  * via `journalctl -u docker.service`
  * ou `/var/log/syslog` (selon ta distro).
* Si les logs sont redirigés vers un fichier, tu verras le **chemin du fichier contenant le stack trace**.

***

### 🔧 Sur Windows Server

1. Télécharge l’outil [**`docker-signal`**](https://github.com/moby/docker-signal).
2. Récupère le PID du démon :

```powershell
Get-Process dockerd
```

3. Lance la commande :

```powershell
docker-signal.exe --pid=<PID>
```

***

### 📌 Remarques importantes

* Le stack trace **n’interrompt pas** le fonctionnement du démon.
* Cela permet d’analyser l’état de tous les **threads/goroutines Go** au moment du problème.
* Très utile pour :
  * **diagnostiquer des deadlocks**
  * **analyser une surcharge CPU**
  * comprendre pourquoi le démon est **lent ou bloqué**.

## 🐳 Visualiser les stack traces du démon Docker (`dockerd`)

Quand tu forces un **dump de stack trace** (avec `SIGUSR1` par exemple), Docker écrit les traces dans ses **logs système**.\
Tu peux ensuite les consulter pour diagnostiquer les blocages.

***

### 📂 Où consulter les logs du démon Docker ?

#### 🔹 Sur Linux

* Avec **systemd** :

```bash
journalctl -u docker.service
```

* Sur les systèmes plus anciens (ou si systemd n’est pas utilisé) :

```bash
cat /var/log/messages
cat /var/log/daemon.log
cat /var/log/docker.log
```

#### 🔹 Exemple de message dans les logs

Après un `kill -SIGUSR1 $(pidof dockerd)`, tu pourrais voir un message comme :

```
...goroutine stacks written to /var/run/docker/goroutine-stacks-2017-06-02T193336z.log
```

👉 Ici, Docker a écrit le **stack trace complet** dans un fichier temporaire (`/var/run/docker/goroutine-stacks-<date>.log`).

***

### 🔹 Sur Docker Desktop (Mac / Windows)

⚠️ Il n’est pas possible de générer manuellement un stack trace via un signal.\
À la place :

* Clique sur l’icône Docker dans la barre des tâches.
* Choisis **Troubleshoot** → cela enverra les infos nécessaires à Docker.

***

### 📌 Points importants

* Les **stack traces Go** listent l’état des **goroutines** (équivalent des threads en Go).
* Elles sont utiles pour :
  * voir si le démon est bloqué sur une **opération I/O**,
  * identifier un **deadlock**,
  * comprendre une **consommation CPU anormale**.
* Tu peux transmettre ces fichiers de stack trace à Docker pour obtenir de l’aide en cas de bug critique.

