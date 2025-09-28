# Base image

## 🔹 Qu’est-ce qu’une _base image_ ?

* C’est **l’image de départ** de ton Dockerfile.
* Elle est définie dans l’instruction **`FROM`**.
* Elle fournit déjà un **système de fichiers initial** (OS minimal, runtime, librairies, etc.) sur lequel tu viens ajouter **tes propres couches**.

👉 Exemple simple :

```dockerfile
FROM debian
```

Ici, tu commences ton image sur **Debian**, ce qui te donne accès à `apt-get`, la libc, etc.

***

## 🔹 Pourquoi utiliser une base image ?

✅ **Gain de temps** → tu n’as pas besoin de reconstruire tout un OS.\
✅ **Confiance & sécurité** → les images officielles sont maintenues et mises à jour régulièrement.\
✅ **Interopérabilité** → tu bénéficies de standards connus (Debian, Alpine, Ubuntu, etc.).\
✅ **Best practices intégrées** → par ex. l’image `python` officielle inclut déjà `pip`.

***

## 🔹 Types de base images

1. **Official Images** (ex. `debian`, `alpine`, `ubuntu`, `node`, `python`)
   * Maintenues par Docker ou par la communauté avec des guidelines stricts.
   * Régulièrement mises à jour (patchs de sécurité).
   * Bonne documentation.
2. **Verified Publisher Images**
   * Publiées par des entreprises partenaires de Docker.
   * Ex. `mcr.microsoft.com/dotnet`, `bitnami/*`.
   * Docker garantit l’authenticité (logo ✅ "Verified").
3. **Docker-Sponsored Open Source Images**
   * Maintenues par des projets open-source avec le support de Docker.
   * Exemple : `nginx`, `postgres`.

***

## 🔹 Alternative : scratch

Tu peux aussi partir de **`FROM scratch`** :

```dockerfile
FROM scratch
COPY hello /
CMD ["/hello"]
```

* `scratch` est une **image vide**, utilisée pour créer des images **ultra-minimales**.
* Très utile si tu compiles ton binaire statiquement (Go, Rust, C).
* Résultat → images de quelques Mo seulement.

***

## 🔹 Illustration (conceptuelle)

📌 Exemple avec une stack Node.js :

```
[ Base Image ] → debian:bookworm-slim
       ↓
[ Runtime Layer ] → node:20
       ↓
[ Dependencies ] → npm install
       ↓
[ Application Code ] → COPY . .
       ↓
[ Final Image ] → prête à tourner
```

## 🔹 2 façons de créer une _base image_

### 1. Utiliser **`FROM scratch`**

C’est une image **vide**, le point zéro de toute construction.\
Elle est idéale si tu veux :

* Créer une image **ultra-minimale** (ex. binaire Go ou Rust compilé statiquement).
* N’ajouter **que ton exécutable et ses dépendances**.

👉 Exemple :

```dockerfile
FROM scratch
COPY hello /
CMD ["/hello"]
```

Ici, ton image ne contient que le binaire `hello`. Parfait pour réduire la taille et la surface d’attaque.

➡️ Voir : **Create a minimal base image using scratch**.

***

### 2. Créer une base image à partir d’une distribution Linux

Tu peux construire une base à partir d’un **root filesystem** (`rootfs`) de ta distro.

* Le rootfs = système de fichiers de base (binaires, librairies, etc.).
* On le prépare comme une archive `.tar` (souvent téléchargée depuis la distro elle-même).
* Puis on l’importe dans Docker.

👉 Exemple avec **Ubuntu** :

```bash
curl -o ubuntu-base.tar.gz http://cdimage.ubuntu.com/ubuntu-base/releases/24.04/release/ubuntu-base-24.04-base-amd64.tar.gz

cat ubuntu-base.tar.gz | docker import - ubuntu:24.04
```

Ensuite, tu peux utiliser ton image :

```dockerfile
FROM ubuntu:24.04
RUN apt-get update && apt-get install -y curl
```

➡️ Voir : **Create a full image using tar**.

***

## 🔹 Quand créer ton propre base image ?

✅ Tu veux un **contrôle total** sur ce qui entre dans l’image (sécurité, dépendances).\
✅ Tu veux un **rootfs customisé** (par ex. une distro maison ou un Linux spécialisé embarqué).\
✅ Tu veux **réduire la taille** et éviter tout ce qui est inutile.

## 🪶 Créer une image minimale avec **`scratch`**

#### 🔹 Qu’est-ce que `scratch` ?

* `scratch` n’est **pas une image classique** :
  * tu ne peux **pas la pull** (`docker pull scratch` échoue ❌).
  * tu ne peux pas **l’exécuter directement**.
* C’est un **mot réservé** qui indique à Docker :\
  👉 "Je veux commencer **à zéro**, sans système de fichiers pré-installé".

En clair, c’est **une base vide** dans laquelle tu dois toi-même **ajouter tout ce dont ton programme a besoin**.

***

#### 🔹 Exemple : programme ultra-minimal

👉 Supposons que tu aies un exécutable **`hello`** déjà compilé (par exemple en Go ou Rust, statiquement lié — donc sans dépendances externes).

**Dockerfile :**

```dockerfile
# syntax=docker/dockerfile:1
FROM scratch
ADD hello /
CMD ["/hello"]
```

* `FROM scratch` → point de départ vide.
* `ADD hello /` → copie le binaire `hello` dans la racine `/`.
* `CMD ["/hello"]` → exécute ton programme quand le conteneur démarre.

***

#### 🔹 Construire l’image

Assure-toi d’avoir ton binaire **`hello`** dans le dossier courant (ton _build context_).\
Puis construis l’image :

```bash
docker build --tag hello .
```

***

#### 🔹 Exécuter l’image

```bash
docker run --rm hello
```

👉 Le conteneur démarre et exécute ton binaire minimal.

***

## ⚠️ Attention aux dépendances

Ton binaire doit être **statiquement compilé** (sans dépendances dynamiques comme libc, SSL, etc.), sinon il ne fonctionnera pas dans un conteneur `scratch`.

Exemples de dépendances possibles :

* **Runtimes** (ex. Python, Java) → ils n’existent pas dans `scratch`.
* **Librairies dynamiques C** (glibc, musl, etc.).
* **Certificats CA** (si ton programme fait du HTTPS).

***

## 🎯 Conclusion

✅ `scratch` est parfait pour des **programmes simples et statiquement liés** (Go, Rust, C statique).\
✅ C’est le **meilleur moyen de créer une image minuscule** et avec une **surface d’attaque minimale**.\
⚠️ Mais si ton programme dépend de bibliothèques ou runtimes → tu dois les **ajouter toi-même**, ce qui peut vite devenir complexe.

## 🐧 Créer une image complète avec `tar`

#### 🔹 Principe

1. Tu prépares un **root filesystem** de ta distribution (Ubuntu, Debian, CentOS, Alpine…).\
   👉 soit en partant d’une machine existante,\
   👉 soit avec un outil comme **`debootstrap`** (pour Debian/Ubuntu).
2. Tu le compresses en **tarball** (`tar -c`).
3. Tu l’**importes** dans Docker via `docker import`.

***

#### 🔹 Exemple concret : Ubuntu 24.04 (codename _noble_)

```bash
# 1. Générer un rootfs Ubuntu minimal avec debootstrap
sudo debootstrap noble noble > /dev/null

# noble = dossier cible qui contient tout le rootfs
# "noble" = codename d'Ubuntu 24.04
```

👉 Après cette étape, tu as un répertoire `noble/` qui ressemble à `/` d’une vraie machine Ubuntu (bin, etc, usr, var, …).

***

```bash
# 2. Créer une archive tar et l'importer dans Docker
sudo tar -C noble -c . | docker import - noble
```

* `tar -C noble -c .` → archive tout le contenu du rootfs.
* `docker import - noble` → crée une image nommée `noble` à partir de cette archive.

***

#### 🔹 Vérification

```bash
docker run noble cat /etc/lsb-release
```

👉 Résultat attendu :

```
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=24.04
DISTRIB_CODENAME=noble
DISTRIB_DESCRIPTION="Ubuntu 24.04.2 LTS"
```

***

## 🛠️ Points importants

* `docker import` **ne garde pas d’historique de couches** → contrairement à `docker build`.
* Tu obtiens une **image de base monolithique** → pratique pour avoir un OS complet, mais moins optimisé pour la maintenance.
* Pour Debian/Ubuntu, `debootstrap` est l’outil classique.
* Pour CentOS/AlmaLinux/RHEL → `yum --installroot` ou `dnf --installroot`.
*   Tu peux aussi capturer un système existant avec :

    ```bash
    sudo tar -c / | docker import - mybase
    ```

    (mais ⚠️ il faut nettoyer `/proc`, `/sys`, `/dev` avant).

***

## 🎯 Quand utiliser `tar` vs `scratch`

* **`scratch`** → images **ultra-minimalistes** (binaire statique Go/Rust, microservices légers).
* **`tar`** → créer une **vraie base OS personnalisée**, ex. une Ubuntu avec ton propre hardening ou une distribution interne.
