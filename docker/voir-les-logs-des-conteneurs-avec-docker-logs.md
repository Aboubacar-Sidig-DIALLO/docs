# 📜 Voir les logs des conteneurs avec docker logs

La commande **`docker logs`** permet d’afficher les informations enregistrées par un **conteneur en cours d’exécution**.\
Pour un service (Swarm), on utilise **`docker service logs`**, qui affiche les logs de **tous les conteneurs du service**.

***

### 🔹 1. Fonctionnement

* Les logs affichés dépendent **du processus exécuté dans le conteneur**.
* Par défaut, `docker logs` montre :
  * **STDOUT** → sortie normale,
  * **STDERR** → messages d’erreur.

👉 Cela correspond exactement à ce que tu verrais si tu lançais la commande directement dans un terminal.

***

### 🔹 2. Exemple simple

Démarrage d’un conteneur qui affiche du texte :

```bash
docker run --name test-logs alpine echo "Hello Docker Logs"
```

Puis affichage des logs :

```bash
docker logs test-logs
```

Résultat :

```
Hello Docker Logs
```

***

### 🔹 3. Cas particuliers

1. **Si tu utilises un&#x20;**_**logging driver**_**&#x20;externe** (ex. vers un fichier, une base ou un serveur distant) et que le _dual logging_ est désactivé 👉 `docker logs` peut ne rien afficher.
2. **Si ton conteneur exécute une application non interactive** (ex. serveur web ou base de données) 👉 l’appli écrit souvent dans ses propres fichiers de logs au lieu de `STDOUT`/`STDERR`.

***

### 🔹 4. Solutions pour les applis comme Nginx / Apache

*   **Nginx officiel** : redirige ses logs vers `STDOUT` et `STDERR` avec des liens symboliques :

    ```bash
    /var/log/nginx/access.log -> /dev/stdout
    /var/log/nginx/error.log  -> /dev/stderr
    ```
* **Apache httpd officiel** : configure directement l’appli pour écrire vers :
  * `/proc/self/fd/1` → `STDOUT`
  * `/proc/self/fd/2` → `STDERR`

Ainsi, les logs sont accessibles avec `docker logs`.

***

### 🔹 5. Pour aller plus loin

Quelques options utiles de `docker logs` :

* `-f` → suivre les logs en temps réel (équivalent `tail -f`)
* `--tail 100` → n’afficher que les 100 dernières lignes
* `-t` → afficher l’horodatage

Exemple :

```bash
docker logs -f --tail 50 -t my-container
```
