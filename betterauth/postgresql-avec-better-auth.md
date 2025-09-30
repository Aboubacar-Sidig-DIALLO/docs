# 📌 PostgreSQL avec Better Auth

Better Auth peut utiliser **PostgreSQL** comme base de données principale pour stocker tes utilisateurs, sessions, tokens et toutes les tables générées par tes plugins.\
C’est une solution robuste et scalable 🔥.

***

### ⚡ Exemple d’utilisation

Assure-toi d’avoir PostgreSQL installé et configuré, puis connecte-le directement dans ton `auth.ts` :

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { Pool } from "pg";

export const auth = betterAuth({
  database: new Pool({
    connectionString: "postgres://user:password@localhost:5432/database",
  }),
});
```

👉 Ici on utilise `pg.Pool` pour gérer les connexions.

***

### ⚙️ Génération et migration du schéma

Better Auth fournit une **CLI** pour générer et migrer automatiquement le schéma SQL nécessaire.

#### Générer le schéma

```bash
npx @better-auth/cli@latest generate
```

#### Appliquer la migration

```bash
npx @better-auth/cli@latest migrate
```

Cela crée (ou met à jour) les tables dans PostgreSQL, basées sur ta config et tes plugins (sessions, utilisateurs, rate limit, etc.).

***

### ✅ Support

* **Schema Generation** : Oui
* **Schema Migration** : Oui
* **Adapter** : Kysely (Better Auth s’appuie dessus).
* **Compatibilité** : Tout moteur supporté par Kysely fonctionne (MySQL, SQLite, PlanetScale, etc.), mais **PostgreSQL est recommandé** pour la production.

***

### 🚀 Bonus : Performance optimisée PostgreSQL

Quelques astuces si tu veux pousser plus loin :

1. **Indexation** : ajoute des index sur `userId`, `email`, `sessionId` pour accélérer les requêtes.
2. **Connection Pooling** : utilise pgBouncer ou le Pool de `pg` en production.
3. **Monitoring** : surveille les requêtes longues (`pg_stat_statements`).
4. **Partitions** : utile si tu gères beaucoup de logs ou sessions expirées.
