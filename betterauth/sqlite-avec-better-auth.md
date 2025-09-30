# 📌 SQLite avec Better Auth

SQLite est une **base de données légère et embarquée**, parfaite pour le développement, les tests, ou les petites applications.\
Better Auth la supporte directement grâce à **Kysely** ✅.

***

### ⚡ Exemple d’utilisation

Better Auth supporte plusieurs drivers SQLite. Voici les options :

#### 1️⃣ **Better-SQLite3 (Recommandé)**

C’est le driver SQLite le plus populaire et stable pour Node.js.

```ts
// auth.ts
import { betterAuth } from "better-auth";
import Database from "better-sqlite3";

export const auth = betterAuth({
  database: new Database("database.sqlite"),
});
```

***

#### 2️⃣ **Node.js Built-in SQLite (Expérimental)**

Disponible à partir de **Node.js 22.5.0+** via le module `node:sqlite`.

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { DatabaseSync } from "node:sqlite";

export const auth = betterAuth({
  database: new DatabaseSync("database.sqlite"),
});
```

👉 Pour lancer :

```bash
node your-app.js
```

***

#### 3️⃣ **Bun Built-in SQLite**

Si tu utilises **Bun**, tu peux employer son module natif :

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { Database } from "bun:sqlite";

export const auth = betterAuth({
  database: new Database("database.sqlite"),
});
```

***

### ⚙️ Génération & Migration du schéma

Better Auth inclut une **CLI** pour générer ou migrer ton schéma SQLite.

#### Générer le schéma

```bash
npx @better-auth/cli@latest generate
```

#### Appliquer la migration

```bash
npx @better-auth/cli@latest migrate
```

✅ **Schema Generation** : supporté\
✅ **Schema Migration** : supporté

***

### ℹ️ Infos supplémentaires

* Better Auth utilise **Kysely** sous le capot → tout ce que Kysely supporte fonctionne aussi.
* SQLite est idéal pour **dev & tests**, mais en **production**, préfère PostgreSQL ou MySQL pour plus de robustesse.
* Tu peux améliorer les performances avec :
  * **WAL mode** (`PRAGMA journal_mode = WAL;`)
  * Indexation manuelle sur `userId`, `email`, `sessionId`
  * Nettoyage périodique des sessions expirées
