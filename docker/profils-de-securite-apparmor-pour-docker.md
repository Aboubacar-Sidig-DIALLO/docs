# 🔒 Profils de sécurité AppArmor pour Docker

### 📝 Qu’est-ce qu’AppArmor ?

* **AppArmor (Application Armor)** est un **module de sécurité Linux (LSM)**.
* Il permet de **restreindre les capacités** des applications et processus via des **profils de sécurité**.
* Chaque programme peut avoir un **profil** qui définit ce qu’il peut ou ne peut pas faire (accès fichiers, réseaux, appels système, etc.).

***

### 🐳 Docker et AppArmor

* Docker utilise **AppArmor** pour renforcer la sécurité de ses conteneurs.
* Lorsqu’un conteneur démarre, **Docker applique automatiquement un profil AppArmor**.
*   Si aucun profil spécifique n’est défini, Docker utilise le **profil par défaut** :

    ```
    docker-default
    ```
* Ce profil est généré automatiquement par Docker (dans **tmpfs**) et chargé dans le noyau.

⚠️ **Important** :

* Ce profil s’applique aux **conteneurs** uniquement.
* Il ne protège pas directement le **daemon Docker**.
* Un profil AppArmor pour le daemon existe dans le dépôt source Docker (`contrib/apparmor`) mais n’est pas installé par défaut via les paquets `.deb`.

***

### 📌 Points essentiels

* Les conteneurs Docker tournent donc par défaut avec une **politique de sécurité AppArmor**.
* Tu peux créer et charger tes **propres profils AppArmor personnalisés** pour restreindre davantage les permissions des conteneurs sensibles.
* Par exemple, limiter l’accès au réseau, empêcher l’écriture dans certains répertoires, etc.

## 🔒 Comprendre les politiques AppArmor avec Docker

### 🛡️ Le profil par défaut : `docker-default`

* Chaque conteneur Docker qui démarre est **attaché automatiquement** au profil AppArmor `docker-default`, sauf si tu précises autre chose.
* Ce profil est **généré à partir d’un modèle fourni par Docker**.
* Il vise un **équilibre entre sécurité et compatibilité** :
  * **Sécurité modérée** → il bloque certaines actions dangereuses (ex. modification de certains fichiers système).
  * **Compatibilité large** → il permet quand même aux applications de tourner normalement dans la plupart des cas.

👉 Exemple d’exécution explicite avec ce profil :

```bash
docker run --rm -it --security-opt apparmor=docker-default hello-world
```

Ici :

* `--security-opt apparmor=docker-default` → force Docker à utiliser la politique par défaut.
* Si tu ne le précises pas, **Docker applique déjà ce profil automatiquement**.

***

### 🛠️ Remplacement du profil

* Tu peux remplacer `docker-default` par un **profil personnalisé** que tu as créé et chargé sur ton hôte Linux.
* Exemple :

```bash
docker run --rm -it --security-opt apparmor=mon-profil-nginx nginx
```

Cela fait tourner le conteneur **nginx** avec les restrictions définies dans ton profil `mon-profil-nginx`.

***

### 📌 À retenir

* **Sans configuration spécifique → profil `docker-default` appliqué automatiquement**.
* **Avec `--security-opt` → tu choisis le profil AppArmor à appliquer**.
* Si aucun profil valide n’est trouvé → Docker refuse de démarrer le conteneur.

## 🔒 Charger et décharger des profils AppArmor avec Docker

### ✅ Charger un profil personnalisé

Si tu as écrit un profil (par ex. `/path/to/your_profile`), tu dois d’abord le **charger dans AppArmor** :

```bash
apparmor_parser -r -W /path/to/your_profile
```

* `-r` → recharge le profil s’il existe déjà.
* `-W` → ignore certains avertissements non critiques.

👉 Une fois chargé, tu peux l’utiliser avec un conteneur Docker :

```bash
docker run --rm -it --security-opt apparmor=your_profile hello-world
```

Ici, le conteneur **hello-world** tournera avec les restrictions définies dans ton profil `your_profile`.

***

### ❌ Décharger un profil

Si tu veux **retirer** un profil d’AppArmor :

```bash
apparmor_parser -R /path/to/profile
```

* `-R` → supprime le profil de la mémoire du noyau.
* Utile si tu veux tester plusieurs variantes d’un même profil ou nettoyer après un essai.

***

### 📘 Ressources utiles pour écrire tes propres profils

Quand tu veux créer un profil AppArmor spécifique pour tes conteneurs, tu dois comprendre sa syntaxe :

1. [**Quick Profile Language**](https://gitlab.com/apparmor/apparmor/-/wikis/QuickProfileLanguage) → guide rapide sur la syntaxe des profils.
2. [**Globbing Syntax**](https://gitlab.com/apparmor/apparmor/-/wikis/AppArmor_Globbing_Syntax) → règles spécifiques pour les wildcards (`*`, `**`, `?`) utilisées dans les profils.

## 🔒 Exemple : Profil AppArmor pour sécuriser un conteneur **Nginx**

### 1. Créer le profil

On commence par créer un fichier profil `docker-nginx` dans :

📂 `/etc/apparmor.d/containers/docker-nginx`

Contenu du fichier :

```
#include <tunables/global>

profile docker-nginx flags=(attach_disconnected,mediate_deleted) {
  #include <abstractions/base>

  # Autorisations réseaux limitées
  network inet tcp,
  network inet udp,
  network inet icmp,

  deny network raw,
  deny network packet,

  # Gestion des fichiers
  file,
  umount,

  # Interdire l’accès en écriture/lecture à divers répertoires critiques
  deny /bin/** wl,
  deny /boot/** wl,
  deny /dev/** wl,
  deny /etc/** wl,
  deny /home/** wl,
  deny /lib/** wl,
  deny /lib64/** wl,
  deny /media/** wl,
  deny /mnt/** wl,
  deny /opt/** wl,
  deny /proc/** wl,
  deny /root/** wl,
  deny /sbin/** wl,
  deny /srv/** wl,
  deny /tmp/** wl,
  deny /sys/** wl,
  deny /usr/** wl,

  # Journaliser les tentatives d’écriture
  audit /** w,

  # Fichiers nécessaires à Nginx
  /var/run/nginx.pid w,
  /usr/sbin/nginx ix,

  # Interdiction d’exécuter certains binaires
  deny /bin/dash mrwklx,
  deny /bin/sh mrwklx,
  deny /usr/bin/top mrwklx,

  # Capacités minimales nécessaires
  capability chown,
  capability dac_override,
  capability setuid,
  capability setgid,
  capability net_bind_service,

  # Restrictions sur /proc
  deny @{PROC}/* w,
  deny @{PROC}/{[^1-9],[^1-9][^0-9],[^1-9s][^0-9y][^0-9s],[^1-9][^0-9][^0-9][^0-9]*}/** w,
  deny @{PROC}/sys/[^k]** w,
  deny @{PROC}/sys/kernel/{?,??,[^s][^h][^m]**} w,
  deny @{PROC}/sysrq-trigger rwklx,
  deny @{PROC}/mem rwklx,
  deny @{PROC}/kmem rwklx,
  deny @{PROC}/kcore rwklx,

  deny mount,

  # Restrictions sur /sys
  deny /sys/[^f]*/** wklx,
  deny /sys/f[^s]*/** wklx,
  deny /sys/fs/[^c]*/** wklx,
  deny /sys/fs/c[^g]*/** wklx,
  deny /sys/fs/cg[^r]*/** wklx,
  deny /sys/firmware/** rwklx,
  deny /sys/kernel/security/** rwklx,
}
```

***

### 2. Charger le profil

On charge le profil dans AppArmor :

```bash
sudo apparmor_parser -r -W /etc/apparmor.d/containers/docker-nginx
```

***

### 3. Lancer Nginx avec le profil

On démarre le conteneur Nginx en mode détaché avec le profil appliqué :

```bash
docker run --security-opt "apparmor=docker-nginx" \
    -p 80:80 -d --name apparmor-nginx nginx
```

***

### 4. Tester la sécurité

Entrer dans le conteneur :

```bash
docker exec -it apparmor-nginx bash
```

Puis tester quelques commandes interdites :

```bash
ping 8.8.8.8
# Résultat : "Lacking privilege for raw socket"

top
# Résultat : "Permission denied"

touch ~/testfile
# Résultat : "Permission denied"

sh
# Résultat : "Permission denied"

dash
# Résultat : "Permission denied"
```

👉 Cela prouve que ton conteneur **Nginx tourne avec des restrictions de sécurité renforcées** grâce à AppArmor.

## 🐳 Debugger AppArmor avec Docker

### 🔍 1. Utiliser `dmesg` pour voir les logs AppArmor

AppArmor écrit des messages **très détaillés** dans le journal du noyau.\
Chaque événement est enregistré et indique si une opération a été **autorisée (`ALLOWED`)** ou **bloquée (`DENIED`)**.

Exemple d’autorisation :

```
[ 5442.864673] audit: type=1400 audit(1453830992.845:37): \
apparmor="ALLOWED" operation="open" profile="/usr/bin/docker" \
name="/home/jessie/docker/man/man1/docker-attach.1" pid=10923 comm="docker" \
requested_mask="r" denied_mask="r" fsuid=1000 ouid=0
```

➡️ Ici, on voit que le **profil `/usr/bin/docker`** (le démon Docker) a autorisé une opération d’ouverture de fichier.

Exemple de refus :

```
[ 3256.689120] type=1400 audit(1405454041.341:73): \
apparmor="DENIED" operation="ptrace" profile="docker-default" \
pid=17651 comm="docker" requested_mask="receive" denied_mask="receive"
```

➡️ Ici, c’est le **profil `docker-default`** (appliqué aux conteneurs) qui a **bloqué** une tentative de `ptrace`.\
👉 C’est attendu, car AppArmor empêche les conteneurs de déboguer d’autres processus.

***

### 🛠 2. Vérifier les profils chargés avec `aa-status`

Pour voir quels profils AppArmor sont actuellement actifs :

```bash
sudo aa-status
```

Exemple de sortie :

```
apparmor module is loaded.
14 profiles are loaded.
1 profiles are in enforce mode.
   docker-default
13 profiles are in complain mode.
   /usr/bin/docker
   /usr/bin/docker///bin/cat
   /usr/bin/docker///bin/ps
   ...
38 processes have profiles defined.
37 processes are in enforce mode.
   docker-default (6044)
   docker-default (31899)
1 processes are in complain mode.
   /usr/bin/docker (29756)
```

#### À retenir :

* **Enforce mode** → AppArmor applique strictement le profil et bloque les actions interdites.
* **Complain mode** → AppArmor ne bloque pas, mais enregistre dans `dmesg` les violations (utile pour tester un profil avant de le durcir).
* `docker-default` est appliqué aux conteneurs par défaut.
* `/usr/bin/docker` est le profil du démon Docker (souvent en _complain mode_ pour éviter les blocages).

***

### 🔧 3. Cas pratiques

* Si ton conteneur ne peut pas exécuter une commande → vérifie dans `dmesg` si AppArmor l’a bloquée.
* Si tu développes un profil personnalisé → commence par le mettre en **complain mode**, observe les logs (`dmesg`), puis ajuste ton profil avant de le passer en **enforce mode**.

***

### 📌 4. Ressources avancées

* Les profils Docker par défaut vivent dans `profiles/apparmor` dans les sources Docker.
* Le profil du démon Docker (`/usr/bin/docker`) est dans `contrib/apparmor`.
* Pour écrire des profils → consulte la doc **AppArmor Profile Language** et **Globbing Syntax**.
