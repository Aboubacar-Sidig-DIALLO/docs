# 🐳 Désinstaller Docker Compose

La méthode de désinstallation dépend de la façon dont Docker Compose a été installé.

Il existe trois cas principaux :

1. **Via Docker Desktop**
2. **En tant que plugin CLI**
3. **Manuellement (binaire standalone téléchargé avec `curl`)**

***

### 🖥️ 1. Désinstaller Docker Compose installé avec Docker Desktop

⚠️ Attention : si vous désinstallez **Docker Desktop**, cela supprime **tous les composants Docker**, y compris :

* Docker Engine
* Docker CLI
* Docker Compose

👉 Pour désinstaller, suivez les étapes de la désinstallation de **Docker Desktop** (selon votre OS).

***

### 🔌 2. Désinstaller Docker Compose installé comme **plugin CLI**

Si vous avez installé Docker Compose via un **gestionnaire de paquets** :

#### Sur **Ubuntu / Debian** :

```bash
sudo apt-get remove docker-compose-plugin
```

#### Sur **RPM-based distributions (Fedora, CentOS, RHEL, etc.)** :

```bash
sudo yum remove docker-compose-plugin
```

***

### 📦 3. Désinstaller Docker Compose installé **manuellement (binaire)**

#### Pour un utilisateur spécifique (installation dans `~/.docker/cli-plugins/`) :

```bash
rm $DOCKER_CONFIG/cli-plugins/docker-compose
```

#### Pour tous les utilisateurs (installation globale dans `/usr/local/lib/`) :

```bash
rm /usr/local/lib/docker/cli-plugins/docker-compose
```

⚠️ Si vous obtenez une erreur **Permission denied**, relancez la commande avec `sudo` :

```bash
sudo rm /usr/local/lib/docker/cli-plugins/docker-compose
```

***

### 🔍 Vérifier où Docker Compose est installé

Si vous ne savez pas où Docker Compose est installé, utilisez :

```bash
docker info --format '{{range .ClientInfo.Plugins}}{{if eq .Name "compose"}}{{.Path}}{{end}}{{end}}'
```

Cela affichera le **chemin exact** du binaire `docker-compose`.

***

✅ Résumé rapide :

* **Docker Desktop** → désinstallation supprime tout Docker.
* **Plugin CLI via apt/yum** → `sudo apt-get remove docker-compose-plugin` ou `sudo yum remove docker-compose-plugin`.
* **Binaire manuel (`curl`)** → supprimer directement le fichier avec `rm`.
