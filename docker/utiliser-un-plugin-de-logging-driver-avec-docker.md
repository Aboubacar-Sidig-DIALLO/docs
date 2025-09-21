# 📝 Utiliser un plugin de logging driver avec Docker

Les **plugins de logging** permettent d’étendre et de personnaliser les capacités de journalisation de Docker, au-delà des drivers intégrés (`json-file`, `local`, `fluentd`, etc.).\
👉 Par exemple, un fournisseur peut publier un plugin sur **Docker Hub** ou dans un **registre privé**.

***

### 🔹 1. Installer un plugin de logging

On installe un plugin avec :

```bash
docker plugin install <organisation/image>
```

Exemple fictif (dépend du plugin fourni par un éditeur tiers) :

```bash
docker plugin install myorg/docker-logs-plugin
```

📌 Vérifier les plugins installés :

```bash
docker plugin ls
```

Inspecter un plugin précis :

```bash
docker inspect myorg/docker-logs-plugin
```

***

### 🔹 2. Configurer le plugin comme **driver par défaut**

Modifier le fichier **daemon.json** (généralement `/etc/docker/daemon.json`) :

```json
{
  "log-driver": "myorg/docker-logs-plugin",
  "log-opts": {
    "option1": "value1",
    "option2": "value2"
  }
}
```

Puis **redémarrer Docker** :

```bash
sudo systemctl restart docker
```

***

### 🔹 3. Utiliser le plugin sur un **conteneur spécifique**

Si tu veux appliquer le plugin uniquement à un conteneur :

```bash
docker run -d \
  --log-driver=myorg/docker-logs-plugin \
  --log-opt option1=value1 \
  --log-opt option2=value2 \
  alpine echo "Hello plugin logging!"
```

***

✅ Avec cette approche :

* Tu peux centraliser tes logs vers une solution externe (Splunk, ELK, Datadog, etc.).
* Tu n’es pas limité aux drivers intégrés à Docker.
* Tu peux définir des options spécifiques par **conteneur** ou au **niveau global** (daemon).
