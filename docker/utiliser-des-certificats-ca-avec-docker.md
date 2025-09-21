# 🔒 Utiliser des certificats CA avec Docker

### ⚠️ Avertissement

Lorsqu’on utilise des **certificats d’autorité de certification (CA)** dans un contexte de **proxy MITM (Man-in-the-Middle)** en production, il faut appliquer des **bonnes pratiques de sécurité**.

➡️ Si le certificat est compromis, un attaquant peut :

* intercepter des données sensibles,
* usurper un service de confiance,
* lancer des attaques de type **MITM**.

👉 Toujours consulter l’équipe sécurité avant d’appliquer ce genre de configuration.

***

### Pourquoi ?

Si ton entreprise utilise un **proxy qui inspecte le trafic HTTPS**, il faut que :

* **l’hôte Docker** fasse confiance au certificat du proxy,
* et que **les conteneurs** aient également ce certificat dans leur magasin de confiance.

Sinon :

* Les commandes comme `docker pull` échoueront,
* Les applis à l’intérieur des conteneurs verront des erreurs SSL/TLS.

***

### 🖥️ Ajouter un certificat CA sur l’hôte

#### macOS

1. Télécharge le certificat CA de ton proxy MITM.
2. Ouvre l’app **Keychain Access** (Trousseaux d’accès).
3. Sélectionne **System**, onglet **Certificates**.
4. Fais glisser ton certificat téléchargé.
5. Clique dessus → section **Trust** → choisis **Always Trust**.
6. Redémarre Docker Desktop.
7.  Vérifie avec :

    ```bash
    docker pull alpine
    ```

***

#### Windows

Tu peux l’installer via **MMC** ou ton **navigateur**. Exemple avec **MMC** :

1. Télécharge le certificat CA.
2. Ouvre **mmc.exe**.
3. Menu **File → Add/Remove Snap-in → Certificates → Add**.
4. Choisis **Computer Account → Local Computer**.
5. Va dans **Trusted Root Certification Authorities → Certificates**.
6. Clic droit → **All Tasks → Import** → importe ton certificat.
7. Redémarre Docker Desktop.
8. Vérifie avec `docker pull`.

💡 Selon ton SDK ou runtime (Java, .NET, Node.js…), il peut falloir ajouter le certificat aussi dans le **trust store** applicatif.

***

### 🐧 Ajouter un certificat CA dans les **images Linux** ou **conteneurs**

#### Pourquoi ?

* Pour que les applis dans les conteneurs puissent accéder aux **APIs internes**, **bases de données**, ou **services sécurisés** sans erreurs TLS.
* On peut l’ajouter :
  * **à l’image (build-time)** → persistant.
  * **au conteneur en cours d’exécution (runtime)** → temporaire.

***

#### Ajouter un certificat CA à une **image**

Exemple pour Ubuntu :

```dockerfile
# Installer le paquet des certificats
RUN apt-get update && apt-get install -y ca-certificates

# Copier le certificat dans l’image
COPY your_certificate.crt /usr/local/share/ca-certificates/

# Mettre à jour le trust store
RUN update-ca-certificates
```

➡️ Tous les conteneurs construits à partir de cette image feront confiance à ce certificat.

***

#### Ajouter un certificat CA à un **conteneur existant**

1. Télécharge le certificat.
   *   Si nécessaire, convertis-le au format `.crt` :

       ```bash
       openssl x509 -in cacert.der -inform DER -out myca.crt
       ```
2.  Copie le certificat dans le conteneur :

    ```bash
    docker cp myca.crt <containerid>:/tmp
    ```
3.  Entre dans le conteneur :

    ```bash
    docker exec -it <containerid> sh
    ```
4.  Vérifie que `ca-certificates` est installé :

    ```bash
    apt-get update && apt-get install -y ca-certificates
    ```
5.  Déplace le certificat au bon emplacement :

    ```bash
    cp /tmp/myca.crt /usr/local/share/ca-certificates/root_cert.crt
    ```
6.  Mets à jour la liste des certificats :

    ```bash
    update-ca-certificates
    ```

    Exemple de sortie :

    ```
    Updating certificates in /etc/ssl/certs...
    1 added, 0 removed; done.
    ```
7.  Vérifie avec `curl` :

    ```bash
    curl https://example.com
    ```

    👉 Si la page HTML s’affiche, le certificat est bien pris en compte.

***

✅ **Résumé :**

* **Hôte Docker** : installer le certificat dans le trust store système.
* **Conteneurs** : soit l’ajouter à l’image (persistance), soit directement à un conteneur (temporaire).
* ⚠️ Toujours prendre en compte les impacts sécurité (MITM).
