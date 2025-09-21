# 📦 containerd image store avec Docker Engine

### 🔹 Disponibilité

* Fonctionnalité **expérimentale**.
* Docker Engine, par défaut, utilise encore **overlay2** comme storage driver.
* `containerd`, le **runtime standard de l’industrie**, utilise à la place des **snapshotters** pour gérer les images et données des conteneurs.

***

### 🔹 Avantage des snapshotters

* Ils remplacent les storage drivers traditionnels.
* Offrent plus de **performance** et une **meilleure intégration** avec l’écosystème containerd.
* Permettent une évolution future de Docker vers un modèle plus unifié avec containerd.

Pour en savoir plus, Docker recommande de consulter la section dédiée **containerd image store sur Docker Desktop**.

***

### 🔹 Activer le containerd image store sur Docker Engine

⚠️ **Attention** :

* Si tu passes aux snapshotters containerd, tu **perds temporairement l’accès aux images et conteneurs créés avec les storage drivers classiques**.
* Ces ressources existent toujours sur ton disque (dans `/var/lib/docker`), mais elles ne sont plus visibles tant que le mode snapshotters est activé.
* Tu peux les récupérer en désactivant cette fonctionnalité.

#### Étapes :

1.  **Modifier la configuration Docker**\
    Ouvre le fichier `/etc/docker/daemon.json` et ajoute :

    ```json
    {
      "features": {
        "containerd-snapshotter": true
      }
    }
    ```
2. **Sauvegarder le fichier**
3.  **Redémarrer le daemon Docker**

    ```bash
    sudo systemctl restart docker
    ```
4.  **Vérifier le driver en cours**

    ```bash
    docker info -f '{{ .DriverStatus }}'
    ```

    Résultat attendu :

    ```
    [[driver-type io.containerd.snapshotter.v1]]
    ```

***

### 🔹 Snapshotter par défaut

* Quand Docker Engine bascule sur containerd, il utilise **overlayfs containerd snapshotter** par défaut.

***

✅ En résumé :

* **Storage drivers classiques** = overlay2, aufs, etc. (gérés par Docker Engine).
* **containerd snapshotters** = nouvelle approche expérimentale pour stocker images et conteneurs, plus proche du runtime containerd.
* Activation via `daemon.json` avec `containerd-snapshotter: true`.
