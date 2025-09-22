# 🔒 Docker Security Non-Events

Cette page répertorie des vulnérabilités de sécurité que **Docker a automatiquement atténuées** grâce à son modèle de sécurité par défaut — même avant que les failles ne soient corrigées dans le noyau Linux.

👉 Cela s’applique **tant que les conteneurs ne sont pas lancés avec des capacités supplémentaires ou en mode `--privileged`**.

Docker s’appuie par défaut sur :

* **AppArmor**
* **Seccomp**
* **Réduction des capacités Linux (capabilities)**

Ces mécanismes font que de nombreux bugs connus (et inconnus) sont **inexploitables dans un conteneur Docker standard**.

***

### ✅ Vulnérabilités atténuées par Docker

* **CVE-2013-1956, 1957, 1958, 1959, 1979, CVE-2014-4014, 5206, 5207, 7970, 7975, CVE-2015-2925, 8543, CVE-2016-3134, 3135, etc.**\
  → Exploits liés aux **user namespaces** (exposition accrue via `mount()` et autres appels root-only).\
  → Mitigation : **Docker empêche la création de namespaces imbriqués** dans les conteneurs via le profil seccomp.
* **CVE-2014-0181, CVE-2015-3339**\
  → Vulnérabilités exploitant des binaires `setuid`.\
  → Mitigation : **Docker désactive les binaires setuid** avec le flag `NO_NEW_PRIVS`.
* **CVE-2014-4699**\
  → Bug dans `ptrace()` permettant une élévation de privilèges.\
  → Mitigation : `ptrace` est **bloqué par AppArmor, Seccomp et la suppression de `CAP_PTRACE`**.
* **CVE-2014-9529**\
  → Exploit via des appels `keyctl()` causant DoS ou corruption mémoire.\
  → Mitigation : **`keyctl()` bloqué par seccomp**.
* **CVE-2015-3214, 4036**\
  → Bugs dans des pilotes de virtualisation, permettant à un invité VM d’exécuter du code sur l’hôte.\
  → Mitigation : Docker **cache l’accès direct aux périphériques de virtualisation** sauf en mode `--privileged`.\
  ⚡ Intéressant : ici, **un conteneur Docker est plus sûr qu’une VM**.
* **CVE-2016-0728**\
  → Use-after-free avec `keyctl()` pouvant mener à une escalade de privilèges.\
  → Mitigation : **`keyctl()` désactivé par défaut via seccomp**.
* **CVE-2016-2383**\
  → Bug dans **eBPF** permettant des lectures mémoire arbitraires.\
  → Mitigation : **Appel système `bpf()` bloqué par seccomp**.
* **CVE-2016-3134, 4997, 4998**\
  → Bug dans `setsockopt` (IPT\_SO\_SET\_REPLACE, etc.) causant corruption mémoire / escalade.\
  → Mitigation : Bloqué par l’absence de `CAP_NET_ADMIN` dans Docker.

***

### ❌ Vulnérabilités non atténuées

* **CVE-2015-3290, 5157**\
  → Bugs dans la gestion des interruptions non masquables (NMI) du noyau, permettant une escalade.\
  → Exploitable dans Docker car **l’appel système `modify_ldt()` n’est pas bloqué** par seccomp.
* **CVE-2016-5195 — Dirty COW**\
  → Condition de concurrence dans la gestion du Copy-On-Write (COW), permettant d’écrire dans de la mémoire en lecture seule.\
  → Partiellement atténué sur certains systèmes par :
  * Seccomp bloquant `ptrace`
  * `/proc/self/mem` accessible uniquement en lecture.

***

### 📌 Conclusion

Même si toutes les vulnérabilités ne sont pas couvertes (ex. Dirty COW, `modify_ldt()`), **Docker offre par défaut une couche de sécurité importante** grâce à l’isolement et au blocage d’appels sensibles.

Cela rend beaucoup d’exploits **inapplicables dans un conteneur standard**, contrairement à une VM classique ou à un processus direct sur l’hôte.
