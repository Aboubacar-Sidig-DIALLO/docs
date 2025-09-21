# 🧠 Tmpfs mounts (montages en mémoire)

### 🔹 Qu’est-ce que c’est ?

* Contrairement aux **volumes** (persistants) et aux **bind mounts** (liés à l’host),\
  un **tmpfs mount** existe uniquement **dans la mémoire RAM du host Linux**.
* Quand le conteneur s’arrête → **tout est perdu**.
* C’est directement basé sur le mécanisme **`tmpfs` du kernel Linux**.

⚠️ **Limitations :**

* Pas partageable entre plusieurs conteneurs (contrairement aux volumes).
* Fonctionne uniquement sur **Linux** (pas Windows/Mac directement).
* Si swap activé, il se peut que le contenu soit écrit temporairement sur disque → pas 100% “volatile”.

***

### 🔹 Quand utiliser un tmpfs ?

✅ Idéal quand :

* Tu veux stocker des **données sensibles** (ex: clés, tokens) → ne pas laisser de trace sur disque.
* Tu as des **données temporaires** ou du **cache**.
* Tu veux améliorer les **performances I/O** (écriture RAM ≫ disque).

❌ Pas adapté quand tu veux **persister** des données ou les partager.

***

### 🔹 Syntaxe

#### Avec `--mount` (préféré, plus explicite) :

```bash
docker run --mount type=tmpfs,dst=/app,tmpfs-size=64m,tmpfs-mode=1770 nginx:latest
```

#### Avec `--tmpfs` (plus court) :

```bash
docker run --tmpfs /app:rw,noexec,size=64m,mode=1777 nginx:latest
```

***

### 🔹 Options disponibles

#### Avec `--tmpfs` :

* `ro` / `rw` → lecture seule ou lecture/écriture (par défaut).
* `noexec` → interdit d’exécuter des binaires dans le tmpfs.
* `size=64m` → limite de taille.
* `mode=1777` → permissions d’accès.
* `uid=1000` / `gid=1000` → propriétaire utilisateur/groupe.
* `nosuid`, `nodev` → restrictions de sécurité.

👉 Exemple :

```bash
docker run --tmpfs /data:noexec,size=128m,mode=700,uid=1000,gid=1000 alpine
```

***

#### Avec `--mount` :

```bash
docker run --mount type=tmpfs,dst=/cache,tmpfs-size=104857600,tmpfs-mode=1777 alpine
```

* `tmpfs-size` → taille max en bytes (par défaut : 50% RAM host).
* `tmpfs-mode` → mode octal (par défaut : `1777`, donc world-writable).

***

### 🔹 Vérification

Après avoir lancé un conteneur avec un tmpfs, tu peux vérifier avec :

```bash
docker inspect tmptest --format '{{ json .Mounts }}'
```

Résultat :

```json
[{
  "Type": "tmpfs",
  "Source": "",
  "Destination": "/app",
  "Mode": "",
  "RW": true,
  "Propagation": ""
}]
```

***

### 🔹 Nettoyage

Un tmpfs est supprimé automatiquement à l’arrêt du conteneur :

```bash
docker stop tmptest
docker rm tmptest
```

***

## ✅ Résumé visuel

| Type de mount  | Persistance | Partageable entre conteneurs | Performance    | Usage principal        |
| -------------- | ----------- | ---------------------------- | -------------- | ---------------------- |
| **Volume**     | Oui         | Oui                          | Moyen          | Bases de données, logs |
| **Bind mount** | Oui (host)  | Oui                          | Moyen          | Dev local, config host |
| **Tmpfs**      | Non (RAM)   | Non                          | 🚀 Très rapide | Données sensibles/temp |
