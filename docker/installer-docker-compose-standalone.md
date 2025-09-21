# 🐳 Installer Docker Compose standalone

👉 Cette méthode permet d’installer **Docker Compose en binaire autonome**.\
⚠️ Attention :

* Le mode **standalone** utilise l’ancienne syntaxe `docker-compose` avec un tiret (`-`), et **non pas** la syntaxe moderne `docker compose`.
* Exemple :
  * **Avec plugin (recommandé)** : `docker compose up`
  * **Avec standalone** : `docker-compose up`
* Ce mode n’est conseillé **que pour la rétrocompatibilité** (anciens scripts, CI/CD, etc.).

***

### 🐧 Installation sur Linux

1. Téléchargez et installez Docker Compose standalone :

```bash
curl -SL https://github.com/docker/compose/releases/download/v2.39.3/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```

2. Donnez les permissions d’exécution au binaire :

```bash
chmod +x /usr/local/bin/docker-compose
```

3. Vérifiez l’installation :

```bash
docker-compose version
```

⚡ Astuce :\
Si la commande `docker-compose` ne fonctionne pas (problème de **$PATH**), créez un **lien symbolique** :

```bash
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

***

### 🪟 Installation sur Windows Server

👉 Cette méthode est utile si vous utilisez directement le **Docker Daemon** sur **Windows Server** (sans Docker Desktop).

1. Ouvrez **PowerShell en administrateur**.\
   → Acceptez quand Windows vous demande si vous autorisez la modification.
2. (Optionnel) Activez **TLS 1.2** (obligatoire avec GitHub sur Windows Server 2016 ou versions anciennes) :

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
```

3. Téléchargez Docker Compose standalone (exemple avec v2.39.3) :

```powershell
Start-BitsTransfer -Source "https://github.com/docker/compose/releases/download/v2.39.3/docker-compose-windows-x86_64.exe" -Destination $Env:ProgramFiles\Docker\docker-compose.exe
```

➡️ Pour installer une autre version, remplacez `v2.39.3` par la version souhaitée.

4. Sur **Windows Server 2019**, placez simplement l’exécutable dans :

```
$Env:ProgramFiles\Docker
```

📌 Ce répertoire est déjà dans le **PATH système**, donc vous pourrez utiliser directement la commande suivante.

5. Vérifiez l’installation :

```powershell
docker-compose.exe version
```

***

✅ Résumé :

* **Linux** → télécharger le binaire + chmod + (optionnel symlink).
* **Windows Server** → PowerShell admin + téléchargement + test avec `docker-compose.exe`.
* ⚠️ À n’utiliser que si vous avez besoin de l’ancienne syntaxe `docker-compose`.
