# 🍪 Cookies dans BetterAuth

BetterAuth utilise des **cookies** pour stocker certaines données essentielles liées à l’authentification, comme :

* 🔑 **les jetons de session** (_session tokens_)
* 🔄 **l’état OAuth** (utilisé lors de connexions via Google, GitHub, etc.)
* 🗝️ et d’autres métadonnées nécessaires au bon fonctionnement de l’auth

***

### 🔒 Sécurité des cookies

Tous les cookies générés par BetterAuth sont **signés** ✍️ à l’aide de la **clé secrète (`BETTER_AUTH_SECRET`)** que vous avez fournie dans vos options d’authentification (`auth.ts`).

👉 Cela garantit que :

* ✅ un cookie ne peut pas être **altéré par un utilisateur malveillant**
* ✅ le serveur peut **vérifier l’intégrité** des cookies reçus
* ✅ vos sessions restent **fiables et sécurisées**

***

### 🚀 Points forts

* 🔐 **Cookies sécurisés par signature** → protection contre la falsification.
* 🌍 **Standard web** → fonctionne avec tous les navigateurs modernes.
* ⚡ **Transparence pour le développeur** → BetterAuth gère automatiquement la création, mise à jour et suppression des cookies.

***

👉 En résumé :\
Les **cookies** dans BetterAuth jouent un rôle central dans la gestion de session et l’authentification sociale (OAuth).\
Grâce à la **signature avec votre clé secrète**, ils offrent une **sécurité renforcée** contre toute manipulation 🎉.

## 🏷️ Préfixe des Cookies (Cookie Prefix)

Par défaut, BetterAuth génère ses cookies avec un **préfixe** standardisé.\
👉 La convention utilisée est la suivante :

```
${prefix}.${cookie_name}
```

* **`prefix`** → par défaut = `"better-auth"`
* **`cookie_name`** → dépend du type de cookie (ex: `session`, `oauth_state`, etc.)

***

### 📌 Exemple par défaut

Sans configuration spécifique, un cookie de session sera nommé ainsi :

```
better-auth.session
```

***

### ⚙️ Personnaliser le préfixe

Vous pouvez changer le **préfixe par défaut** en configurant la propriété **`cookiePrefix`** dans l’option **`advanced`** de votre instance BetterAuth.

#### Exemple :

```ts
// auth.ts
import { betterAuth } from "better-auth"

export const auth = betterAuth({
  advanced: {
    cookiePrefix: "my-app" // 🔄 nouveau préfixe
  }
})
```

Résultat → le cookie de session sera désormais nommé :

```
my-app.session
```

***

### 🚀 Pourquoi changer le préfixe ?

* 🏗️ **Multi-apps** → éviter les conflits si plusieurs applications partagent le même domaine.
* 🔒 **Sécurité** → masquer le nom du framework/lib pour réduire les informations exposées.
* 🎯 **Clarté** → personnaliser le nommage pour qu’il reflète votre application.

***

👉 En résumé :\
Le **préfixe des cookies** dans BetterAuth est **personnalisable** via `cookiePrefix`.\
Cela permet d’adapter les noms des cookies à votre projet tout en évitant les conflits 🎉.

## 🍪 Cookies Personnalisés (Custom Cookies)

BetterAuth gère automatiquement les cookies nécessaires à l’authentification.\
👉 Par défaut :

* ✅ Tous les cookies sont **`httpOnly`** et **`secure`** lorsque le serveur tourne en **mode production**.
* ✅ Cela empêche leur accès côté JavaScript (`document.cookie`) et les protège sur HTTPS.

***

### 📌 Cookies par défaut utilisés par BetterAuth

* **`session_token`** → stocke le **jeton de session** de l’utilisateur.
* **`session_data`** → stocke les données de session si le **cache par cookies** est activé.
* **`dont_remember`** → stocke le flag quand l’option **`rememberMe`** est désactivée.

⚡ En plus, certains **plugins** utilisent leurs propres cookies.\
Exemple :

* Le plugin **Two Factor Authentication (2FA)** utilise un cookie **`two_factor`** pour stocker l’état de la double authentification.

***

### ⚙️ Personnaliser les cookies

Vous pouvez redéfinir les **noms** et les **attributs** des cookies via l’option **`advanced.cookies`** dans votre configuration BetterAuth.

#### Exemple :

```ts
// auth.ts
import { betterAuth } from "better-auth"

export const auth = betterAuth({
  advanced: {
    cookies: {
      session_token: {
        name: "custom_session_token", // 🔄 nouveau nom du cookie
        attributes: {
          // Définir vos propres attributs
          path: "/",          // disponible sur tout le site
          sameSite: "Strict", // protège contre CSRF
          secure: true,       // seulement via HTTPS
          httpOnly: true,     // inaccessible depuis JS
          maxAge: 60 * 60 * 24 * 7 // durée de vie : 7 jours
        }
      },
    }
  }
})
```

***

### 🧐 Pourquoi personnaliser les cookies ?

* 🏗️ **Éviter les conflits** → si plusieurs apps partagent le même domaine.
* 🔒 **Renforcer la sécurité** → en ajoutant des règles strictes (SameSite, durée limitée, etc.).
* 🎯 **Adapter à vos besoins** → par ex. réduire la durée de vie du cookie ou le limiter à un chemin spécifique (`/auth`).

***

### 🚀 Points forts

* 🔐 **Sécurisés par défaut** (`httpOnly`, `secure`).
* 🧩 **Extensibles** → plugins ajoutent leurs propres cookies automatiquement.
* ⚡ **Personnalisables** → noms + attributs ajustables selon votre projet.

***

👉 En résumé :\
BetterAuth configure par défaut des cookies **sécurisés et adaptés à l’auth**, mais vous pouvez **personnaliser leur nom et leurs attributs** pour mieux répondre à vos besoins 🎉.

## 🌐 Cookies inter-sous-domaines (Cross Subdomain Cookies)

Dans certains cas, vous pouvez avoir besoin de **partager les cookies entre plusieurs sous-domaines**.

👉 Exemple concret :

* Votre service d’auth est sur **`auth.example.com`**
* Votre application principale est sur **`app.example.com`**\
  ➡️ Vous souhaitez que la session soit reconnue sur les deux.

***

### 📌 Rôle de l’attribut `domain`

L’attribut **`domain`** du cookie définit **quels domaines** peuvent y accéder.

* Si vous le définissez sur la **racine du domaine** (`example.com`) → le cookie sera disponible sur **tous les sous-domaines** (`auth.example.com`, `app.example.com`, `admin.example.com`, etc.).
* Si vous le définissez sur un sous-domaine précis (`app.example.com`) → seul **ce sous-domaine** y aura accès.

***

### ⚠️ Bonnes pratiques de sécurité

* ✅ **Activez le partage de cookies uniquement si nécessaire**
* ✅ Utilisez le **scope le plus spécifique possible**
  * Préférez : `app.example.com`
  * Évitez : `.example.com` (trop large, tous les sous-domaines y accèdent)
* ⚠️ Attention aux **sous-domaines non fiables** qui pourraient accéder aux cookies sensibles
* 🛡️ Pour des services non critiques (ex: `status.company.com`) → utilisez un **domaine séparé** du domaine applicatif (`app.company.com`)

***

### ⚙️ Configuration dans BetterAuth

Exemple d’activation du partage de cookies :

```ts
// auth.ts
import { betterAuth } from "better-auth"

export const auth = betterAuth({
  advanced: {
    crossSubDomainCookies: {
      enabled: true,           // 🔄 active les cookies inter-sous-domaines
      domain: "app.example.com" // 🌍 domaine autorisé
    },
  },
  trustedOrigins: [
    'https://example.com',
    'https://app1.example.com',
    'https://app2.example.com',
  ],
})
```

***

### 🧐 Explications

* **`crossSubDomainCookies.enabled`** → active le partage de cookies entre sous-domaines.
* **`crossSubDomainCookies.domain`** → définit le **scope** du cookie (sous-domaine ou racine).
* **`trustedOrigins`** → liste des **origines autorisées** pour éviter qu’un domaine non approuvé accède à l’auth.

***

### 🚀 Points forts

* 🌍 **Support natif multi-sous-domaines** → pratique pour des apps SaaS organisées par sous-domaines.
* 🔒 **Contrôle de la sécurité** → via `trustedOrigins` + scope précis du `domain`.
* ⚡ **Flexible** → permet à la fois des cookies par app, ou globaux à tout le domaine.

***

👉 En résumé :\
BetterAuth vous permet de partager facilement les **sessions via cookies entre sous-domaines**, tout en vous donnant les outils nécessaires pour **contrôler et sécuriser** cet accès 🎉.

## 🔒 Secure Cookies

Dans BetterAuth, les **cookies sécurisés** (`Secure`) signifient qu’ils **ne sont envoyés que via HTTPS** 🌍.\
👉 Cela empêche leur transmission en clair sur HTTP, ce qui protège vos utilisateurs contre les attaques de type **man-in-the-middle (MITM)**.

***

### 📌 Comportement par défaut

* En **mode développement** → les cookies ne sont **pas sécurisés** (plus pratique pour tester en local).
* En **mode production** → les cookies sont automatiquement marqués comme **`Secure`** ✅.

***

### ⚙️ Forcer les cookies sécurisés

Si vous voulez que les cookies soient **toujours sécurisés**, même en dehors de la production, vous pouvez activer l’option **`useSecureCookies`**.

#### Exemple :

```ts
// auth.ts
import { betterAuth } from "better-auth"

export const auth = betterAuth({
  advanced: {
    useSecureCookies: true // 🔒 force tous les cookies à être sécurisés
  }
})
```

***

### 🚀 Quand utiliser cette option ?

* ✅ **En production** → toujours recommandé (défaut).
* ⚠️ **En développement HTTPS local** (ex: via `mkcert`, `localhost.https`) → utile si vous testez dans un environnement avec SSL.
* ❌ **En développement HTTP classique** (`http://localhost`) → à éviter car les cookies ne fonctionneront pas (ils ne seront pas envoyés sans HTTPS).

***

### 🧩 Points forts

* 🔒 Renforce la **sécurité** en empêchant la transmission de cookies via HTTP.
* 🌍 Compatible avec toutes les apps web modernes qui utilisent HTTPS.
* ⚡ Flexible → choix de conserver le comportement auto ou de forcer le `Secure`.

***

👉 En résumé :\
Par défaut, BetterAuth sécurise automatiquement les cookies **en production**.\
Avec **`useSecureCookies: true`**, vous pouvez **forcer cette sécurité dans tous les environnements**, idéal si vous travaillez toujours en HTTPS 🔐.
