# 🔒 Profils de sécurité Seccomp pour Docker

### 📝 Qu’est-ce que **seccomp** ?

**Seccomp** (Secure Computing Mode) est une fonctionnalité du noyau Linux qui permet de **restreindre les appels système (syscalls)** disponibles pour un processus.\
👉 Dans Docker, on utilise seccomp pour limiter ce qu’un conteneur peut demander au noyau Linux, réduisant ainsi la surface d’attaque.

***

### ⚙️ Vérifier si ton noyau supporte seccomp

Exécute :

```bash
grep CONFIG_SECCOMP= /boot/config-$(uname -r)
```

Si tu obtiens :

```
CONFIG_SECCOMP=y
```

➡️ ton noyau supporte seccomp.

***

### 📂 Le profil seccomp par défaut

* Docker active **par défaut** un profil seccomp qui **bloque \~44 appels système** jugés dangereux (sur environ 300).
* C’est une **liste blanche** :
  * par défaut → tout est bloqué (`SCMP_ACT_ERRNO` = erreur Permission Denied).
  * explicitement → certains appels sont autorisés (`SCMP_ACT_ALLOW`).

👉 Le profil par défaut est **modérément restrictif mais compatible avec la majorité des applications**.\
👉 ⚠️ Il n’est **pas recommandé de modifier ce profil par défaut** sauf cas particulier.

***

### ▶️ Utiliser un profil seccomp

Pour exécuter un conteneur avec un profil spécifique :

```bash
docker run --rm -it \
  --security-opt seccomp=/path/to/seccomp/profile.json \
  debian:latest
```

👉 Exemple avec `hello-world` :

```bash
docker run --rm -it \
  --security-opt seccomp=/path/to/seccomp/profile.json \
  hello-world
```

***

### 🚫 Syscalls bloqués par défaut

Voici quelques exemples significatifs :

| **Syscall**                    | **Raison du blocage**                                   |
| ------------------------------ | ------------------------------------------------------- |
| `mount`, `umount`              | Monter/démonter doit rester une opération privilégiée   |
| `ptrace`, `kcmp`               | Inspection de processus, fuite d’infos sensibles        |
| `reboot`, `kexec`              | Empêcher le redémarrage ou le remplacement du noyau     |
| `keyctl`, `add_key`            | Empêcher l’utilisation du **keyring noyau** (non isolé) |
| `clock_settime`                | L’heure système n’est pas _namespaced_                  |
| `perf_event_open`              | Empêcher le profilage / fuite d’infos noyau             |
| `init_module`, `delete_module` | Interdire le chargement de modules noyau                |

👉 En résumé, **tous les appels système dangereux ou obsolètes sont bloqués**.

***

### 🚀 Lancer un conteneur **sans seccomp**

Si tu veux désactiver complètement seccomp :

```bash
docker run --rm -it \
  --security-opt seccomp=unconfined \
  debian:latest \
  unshare --map-root-user --user sh -c whoami
```

⚠️ Cela revient à **donner beaucoup plus de liberté au conteneur**, ce qui augmente fortement les risques de sécurité.

***

### ✅ Bonnes pratiques

1. **Garde le profil seccomp par défaut** → il couvre la majorité des cas.
2. **Utilise `--security-opt seccomp=...`** uniquement si ton application a besoin d’appels système spécifiques.
3. **N’utilise `unconfined` qu’en dernier recours**, si tu comprends les risques.
