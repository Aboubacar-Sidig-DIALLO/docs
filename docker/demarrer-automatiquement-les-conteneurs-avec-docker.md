# Démarrer automatiquement les conteneurs avec Docker

Docker propose des **politiques de redémarrage** (_restart policies_) pour gérer si un conteneur doit être relancé automatiquement quand :

* il s’arrête,
* il plante (exit code ≠ 0),
* ou quand le **daemon Docker** lui-même redémarre.

C’est une alternative plus simple et plus intégrée que d’utiliser un gestionnaire de processus externe (_systemd_, _supervisor_, etc.).

⚠️ À ne pas confondre avec l’option `--live-restore` :

* `--live-restore` permet de garder les conteneurs **en cours d’exécution** lors d’une mise à jour de Docker, mais ne gère pas les relances automatiques après un arrêt.

***

### 🔧 Utiliser une politique de redémarrage

Lors du lancement d’un conteneur (`docker run`), on peut ajouter `--restart` avec différentes valeurs :

| Option                     | Description                                                                                                                                                                                                                                                |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `no` (par défaut)          | Ne redémarre jamais le conteneur automatiquement.                                                                                                                                                                                                          |
| `on-failure[:max-retries]` | <p>Redémarre <strong>seulement si le conteneur s’arrête avec une erreur</strong> (code ≠ 0).<br>Optionnel : limiter le nombre de tentatives avec <code>:max-retries</code>.</p>                                                                            |
| `always`                   | <p>Redémarre <strong>toujours</strong> le conteneur s’il s’arrête (sauf si on l’arrête manuellement).<br>Si le conteneur est stoppé manuellement, il sera relancé uniquement si le <strong>daemon redémarre</strong> ou si on le relance manuellement.</p> |
| `unless-stopped`           | Similaire à `always`, mais **ne redémarre pas** le conteneur si on l’a arrêté volontairement (même après un reboot du daemon).                                                                                                                             |

***

#### Exemple 1 : Redis qui redémarre automatiquement

```bash
docker run -d --restart unless-stopped redis
```

➡️ Redis se relance toujours, sauf si tu l’arrêtes explicitement (`docker stop`).

***

#### Exemple 2 : Appliquer une politique à un conteneur existant

```bash
docker update --restart unless-stopped redis
```

***

#### Exemple 3 : Appliquer à **tous les conteneurs en cours**

```bash
docker update --restart unless-stopped $(docker ps -q)
```

***

### 📌 Détails importants

1. Une politique de redémarrage **ne s’applique que si le conteneur démarre correctement au moins 10 secondes**.\
   👉 Évite les boucles infinies de conteneurs qui échouent immédiatement.
2. Si tu arrêtes manuellement un conteneur (`docker stop`), la politique est **désactivée** jusqu’à :
   * redémarrage du daemon, ou
   * redémarrage manuel du conteneur.
3. Les politiques `--restart` ne concernent que les **conteneurs classiques**.\
   👉 Pour **Swarm services**, il existe d’autres flags spécifiques (ex. `--restart-condition`).

***

### ▶️ Exemple d’un conteneur qui plante

#### Dockerfile

```dockerfile
FROM busybox:latest
COPY --chmod=755 <<"EOF" /start.sh
echo "Starting..."
for i in $(seq 1 5); do
  echo "$i"
  sleep 1
done
echo "Exiting..."
exit 1
EOF
ENTRYPOINT /start.sh
```

#### Construction

```bash
docker build -t startstop .
```

#### Exécution avec relance automatique

```bash
docker run --restart always startstop
```

Sortie :

```
Starting...
1
2
3
4
5
Exiting...
```

➡️ Le terminal se ferme car le conteneur sort, mais si tu vérifies avec `docker ps` :

```bash
docker ps
```

Tu verras le conteneur en **redémarrage** ou **relancé** grâce à la politique.

***

### 🔄 Attacher de nouveau au conteneur

Tu peux te reconnecter entre deux cycles de redémarrage :

```bash
docker container attach <id_conteneur>
```

➡️ Mais tu seras à nouveau déconnecté quand le conteneur s’arrête.

***

### ⚠️ Gestionnaires de processus (systemd, supervisor)

Si les politiques Docker ne suffisent pas (par exemple, si des **processus externes au conteneur** dépendent de lui), tu peux utiliser un gestionnaire de processus comme :

* **systemd**
* **supervisor**

👉 Mais **ne combine pas** les deux systèmes (Docker restart policy + systemd), sinon conflits.

***

### ⚠️ Process managers à l’intérieur du conteneur

Tu pourrais installer `supervisord` ou autre à l’intérieur du conteneur, mais :

* Ce n’est pas recommandé par Docker,
* Ce n’est pas portable entre distributions,
* Ça rend ton conteneur plus complexe.

***

✅ En résumé :

* Utilise `--restart` (`always`, `unless-stopped`, ou `on-failure`) pour automatiser le redémarrage.
* Évite d’exposer des conteneurs instables à des **boucles infinies**.
* N’utilise un gestionnaire externe que si tu dois orchestrer **en dehors de Docker**.
