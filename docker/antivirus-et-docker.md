# 🛡️ Antivirus et Docker

Lorsque ton **antivirus** analyse les fichiers utilisés par Docker, il peut **verrouiller** certains fichiers, ce qui entraîne :

* des **blocages** ou des **ralentissements** lors de l’exécution de commandes Docker,
* voire des **erreurs** lors de la création ou du démarrage de conteneurs.

***

### 📂 Répertoires à risque

Les répertoires principaux de données Docker sont :

* **Linux** : `/var/lib/docker`
* **Windows Server** : `%ProgramData%\docker`
* **macOS** : `$HOME/Library/Containers/com.docker.docker/`

Ces dossiers contiennent :

* les **images**,
* les **couches écrites des conteneurs**,
* les **volumes**.

***

### ✅ Solution 1 : Exclure Docker de l’analyse en temps réel

Tu peux ajouter le répertoire de données Docker dans la **liste d’exclusions** de ton antivirus.\
➡️ Cela évite les blocages et améliore les performances.

⚠️ **Inconvénient** : les malwares éventuels dans les images Docker ou les volumes **ne seront pas détectés** par ton antivirus.

***

### ✅ Solution 2 : Planifier une analyse sécurisée

Si tu exclues Docker du scan **en temps réel**, tu peux programmer une tâche récurrente :

1.  **Arrêter Docker**

    ```bash
    sudo systemctl stop docker
    ```

    ou sur Windows : arrêter le service Docker Desktop.
2. **Lancer une analyse antivirus manuelle** du répertoire Docker (`/var/lib/docker`, `%ProgramData%\docker`, etc.)
3.  **Redémarrer Docker**

    ```bash
    sudo systemctl start docker
    ```

***

👉 Ce compromis permet :

* d’éviter les blocages en temps réel,
* tout en maintenant une sécurité périodique sur les données Docker.
