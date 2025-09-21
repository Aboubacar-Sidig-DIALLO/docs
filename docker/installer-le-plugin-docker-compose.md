# 🐳 Installer le plugin Docker Compose

Cette page explique **comment installer le plugin Docker Compose** sur **Linux** directement en ligne de commande.

👉 Ces instructions supposent que tu as déjà **Docker Engine** et le **Docker CLI** installés, et que tu souhaites ajouter le **plugin Docker Compose**.

***

### ⚡ Méthode 1 : Installer via le dépôt officiel Docker (recommandée ✅)

1. Configure le dépôt Docker sur ta distribution.
   * Les instructions spécifiques se trouvent ici :\
     👉 Ubuntu | Debian | CentOS | Fedora | RHEL | Raspberry Pi OS.
2. Mets à jour l’index des paquets et installe le plugin :

* Pour **Ubuntu et Debian** :

```bash
sudo apt-get update
sudo apt-get install docker-compose-plugin
```

* Pour les distributions basées sur **RPM** (CentOS, Fedora, RHEL, etc.) :

```bash
sudo yum update
sudo yum install docker-compose-plugin
```

3. Vérifie l’installation :

```bash
docker compose version
```

***

### 🔄 Mettre à jour Docker Compose

Si tu veux mettre à jour :

* Ubuntu/Debian :

```bash
sudo apt-get update
sudo apt-get install docker-compose-plugin
```

* RPM (CentOS, Fedora, RHEL) :

```bash
sudo yum update
sudo yum install docker-compose-plugin
```

***

### ⚡ Méthode 2 : Installation manuelle (non recommandée ⚠️)

👉 Attention : cette méthode **n’active pas les mises à jour automatiques**. Pour un usage long terme, préfère le dépôt officiel.

1. Télécharge le binaire :

```bash
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v2.39.3/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
```

* Installe dans **`$HOME/.docker/cli-plugins`** pour l’utilisateur courant.
* Pour installer **pour tous les utilisateurs**, utilise :\
  `/usr/local/lib/docker/cli-plugins`
* Pour une autre **version**, remplace `v2.39.3` par celle que tu veux.
* Pour une autre **architecture** (ARM, ARM64, etc.), remplace `x86_64`.

2. Donne les droits d’exécution au binaire :

* Pour un seul utilisateur :

```bash
chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
```

* Pour tous les utilisateurs :

```bash
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
```

3. Vérifie l’installation :

```bash
docker compose version
```

***

✅ Résumé :

* **Méthode 1 (dépôt officiel)** → maintenance et mises à jour automatiques.
* **Méthode 2 (manuelle)** → plus flexible (choix de version/archi), mais mise à jour manuelle obligatoire.
