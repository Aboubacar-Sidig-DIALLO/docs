# 📧 Email dans BetterAuth

L’**email** est une pièce maîtresse dans BetterAuth.\
👉 Quel que soit le mode d’authentification utilisé par vos utilisateurs (**mot de passe**, **connexion sociale**, **passkey**, etc.), **chaque compte utilisateur doit être associé à une adresse email unique**.

***

### 🚀 Rôle de l’email dans BetterAuth

1. **Authentification classique**
   * Email + mot de passe → méthode d’auth intégrée par défaut.
2. **Vérification d’email (Email Verification)**
   * BetterAuth gère l’envoi de liens de confirmation pour valider une adresse avant activation du compte.
3. **Réinitialisation de mot de passe (Password Reset)**
   * Utilisation d’un **token envoyé par email** pour sécuriser la récupération d’accès.
4. **Connexion avancée (Magic Link / OTP)**
   * Possibilité d’utiliser un **lien magique** ou un **code à usage unique (OTP)** envoyé par email.
5. **Notifications liées à la sécurité**
   * Exemple : alerte de nouvelle connexion, confirmation d’activation du 2FA, etc.

***

### 📦 Support natif

BetterAuth fournit :

* ✅ Authentification **email + mot de passe** (out of the box).
* ✅ Utilitaires pour :
  * **Vérifier les adresses email**
  * **Réinitialiser les mots de passe**
  * **Gérer les flux avec tokens temporaires**

***

### 🧩 Extensible

Grâce à son système de **plugins**, BetterAuth permet aussi d’aller plus loin avec l’email :

* Magic Link ✨
* Email OTP 🔑
* Double authentification via email 📮

***

### 🎯 En résumé

* 📧 L’**email est obligatoire** pour tous les utilisateurs (quelle que soit la méthode d’authentification).
* 🔐 Il est utilisé pour la **sécurité** (vérification, reset password, OTP).
* 🚀 BetterAuth fournit déjà **les outils nécessaires** pour gérer ces cas sans avoir à tout coder soi-même.

## ✅ Vérification d’Email (Email Verification)

La **vérification d’email** est une fonctionnalité de sécurité indispensable 🔒.\
👉 Elle permet de **confirmer que l’adresse email fournie appartient bien à l’utilisateur**.

#### Pourquoi est-ce important ?

* 🛡️ **Limiter le spam et les faux comptes**
* 👤 **S’assurer de l’identité réelle de l’utilisateur**
* 🔑 **Sécuriser l’accès** avant d’autoriser des actions sensibles (connexion, réinitialisation de mot de passe, etc.)

***

### 📌 Méthodes de vérification

BetterAuth propose deux approches principales :

1. **📩 Vérification par token (lien unique)**
   * Un **lien avec un token** est envoyé à l’email de l’utilisateur.
   * L’utilisateur doit cliquer sur ce lien pour valider son compte.
   * 🎯 C’est la méthode la plus courante, expliquée dans ce guide.
2. **🔢 Vérification par OTP (code à usage unique)**
   * Un **code numérique (OTP)** est envoyé par email.
   * L’utilisateur saisit ce code dans votre application.
   * 👉 Voir la section **OTP Verification** pour ce cas.

***

### 🚀 Ce que nous allons voir ici

👉 Dans ce guide, nous allons mettre en place la **vérification par token** (lien d’activation envoyé par email).

* Génération et envoi du token.
* Création du lien de vérification.
* Validation du token côté serveur.
* Activation du compte utilisateur.

***

### 🎯 En résumé

* La vérification d’email est un **bouclier contre les abus** et garantit que l’utilisateur contrôle bien son adresse.
* BetterAuth intègre nativement la **gestion des tokens et des flux de vérification**.
* Deux méthodes existent :
  * Token (lien d’activation).
  * OTP (code numérique).

## 📧 Ajouter la vérification d’email à votre application

La vérification d’email permet de s’assurer qu’un utilisateur **confirme bien son adresse** avant d’utiliser votre application.\
👉 Pour l’activer dans **BetterAuth**, vous devez fournir une fonction qui envoie un email de vérification contenant un **lien unique**.

***

### ⚙️ Étape 1 – Activer la vérification d’email

Dans votre fichier `auth.ts`, ajoutez la configuration **`emailVerification`** avec la fonction **`sendVerificationEmail`**.

#### Exemple :

```ts
// auth.ts
import { betterAuth } from 'better-auth';
import { sendEmail } from './email'; // votre propre fonction d’envoi d’emails

export const auth = betterAuth({
  emailVerification: {
    sendVerificationEmail: async ({ user, url, token }, request) => {
      await sendEmail({
        to: user.email, // 📧 email de l’utilisateur
        subject: 'Vérifiez votre adresse email',
        text: `Cliquez sur ce lien pour vérifier votre email : ${url}`
      })
    }
  }
})
```

***

### 📌 Détails de la fonction `sendVerificationEmail`

Cette fonction est appelée automatiquement par BetterAuth quand la vérification d’email est déclenchée.\
Elle reçoit :

1.  **`user`** → l’objet utilisateur (contenant au minimum l’email).

    ```ts
    user.email // ex: "test@exemple.com"
    ```
2. **`url`** → l’URL de vérification générée par BetterAuth.\
   👉 L’utilisateur doit cliquer sur ce lien pour confirmer son email.
3. **`token`** → le token de vérification (au cas où vous préférez construire un lien personnalisé ou gérer le flux vous-même).
4. **`request`** → l’objet `Request` de l’appel HTTP (utile si vous voulez ajouter du contexte, comme l’IP de l’utilisateur).

***

### 🚀 Exemple d’implémentation possible

* Utiliser un service comme **SendGrid, Mailgun, Postmark, SES** ou **Nodemailer** pour envoyer vos emails.
* Personnaliser l’email en HTML pour qu’il soit plus esthétique ✨.

Exemple rapide avec **Nodemailer** :

```ts
import nodemailer from 'nodemailer';

const transporter = nodemailer.createTransport({
  host: 'smtp.example.com',
  port: 465,
  secure: true,
  auth: {
    user: process.env.SMTP_USER,
    pass: process.env.SMTP_PASS
  }
});

export async function sendEmail({ to, subject, text }: { to: string; subject: string; text: string }) {
  await transporter.sendMail({
    from: '"MonApp" <no-reply@monapp.com>',
    to,
    subject,
    text
  });
}
```

***

### 🎯 En résumé

* BetterAuth déclenche automatiquement la fonction **`sendVerificationEmail`** lors de la création d’un compte ou d’un flux nécessitant une vérification.
* Vous personnalisez **l’envoi d’email** (contenu + service SMTP/API).
* L’email contient un **lien de vérification** (`url`) généré par BetterAuth.
* Vous pouvez aussi utiliser le **token** si vous souhaitez générer une URL spécifique.

## 📧 Déclencher la vérification d’email (Triggering Email Verification)

BetterAuth permet de **lancer la vérification d’email** de plusieurs façons.\
Selon vos besoins, vous pouvez choisir d’envoyer automatiquement l’email à l’inscription, de l’exiger avant toute connexion, ou encore de le déclencher manuellement.

***

### 🔹 1. Envoyer automatiquement à l’inscription (Sign-up)

Si vous voulez que **chaque nouvel utilisateur** reçoive immédiatement un email de vérification lors de son inscription, activez l’option **`sendOnSignUp`**.

```ts
// auth.ts
import { betterAuth } from 'better-auth';

export const auth = betterAuth({
  emailVerification: {
    sendOnSignUp: true
  }
})
```

👉 Fonctionnement :

* Lorsqu’un utilisateur s’inscrit → BetterAuth envoie automatiquement l’email de vérification.
* Pour les **logins sociaux (SSO)** → BetterAuth vérifie si le provider (Google, GitHub, etc.) affirme que l’email est déjà vérifié.
* Si le provider **ne garantit pas** la vérification → BetterAuth envoie quand même un email de vérification, **mais** :
  * 🔑 l’utilisateur peut quand même se connecter même si son email n’est pas encore confirmé (sauf si vous activez l’option suivante).

***

### 🔹 2. Exiger la vérification avant connexion (Require Email Verification)

Si vous voulez empêcher un utilisateur **non vérifié** de se connecter, utilisez **`requireEmailVerification: true`**.

```ts
// auth.ts
export const auth = betterAuth({
  emailAndPassword: {
    requireEmailVerification: true
  }
})
```

👉 Fonctionnement :

* L’utilisateur doit **vérifier son email avant toute connexion**.
* Chaque tentative de login déclenche l’appel à **`sendVerificationEmail`** (vous devez donc l’avoir implémenté).
* Cela ne s’applique qu’à la connexion **email + mot de passe** (pas aux SSO).

#### Exemple côté client :

```ts
// auth-client.ts
await authClient.signIn.email({
  email: "email@example.com",
  password: "password"
}, {
  onError: (ctx) => {
    // Gérer l’erreur
    if (ctx.error.status === 403) {
      alert("Veuillez vérifier votre adresse email avant de vous connecter 🚨");
    }
    // Ou afficher le message brut renvoyé par le serveur
    alert(ctx.error.message);
  }
})
```

***

### 🔹 3. Déclenchement manuel (Manually)

Vous pouvez aussi déclencher l’envoi de l’email **quand vous le voulez** :

```ts
await authClient.sendVerificationEmail({
  email: "user@email.com",
  callbackURL: "/" // Redirection après la vérification
})
```

👉 Utile si vous proposez dans votre interface :

* Un bouton **"Renvoyer l’email de vérification"**.
* Une option dans le **profil utilisateur** pour vérifier son email plus tard.

***

### 🚀 En résumé

* **`sendOnSignUp: true`** → vérification automatique dès l’inscription.
* **`requireEmailVerification: true`** → empêche la connexion tant que l’email n’est pas confirmé.
* **`sendVerificationEmail()`** → permet de **renvoyer manuellement** un email de vérification (ex: bouton "Renvoyer").

BetterAuth couvre ainsi **tous les cas pratiques** liés à la vérification d’email 🎉.

## ✅ Vérification de l’Email (Verifying the Email)

Une fois que l’utilisateur reçoit son email de vérification, il doit cliquer sur le **lien unique** qui lui a été envoyé.

👉 Deux scénarios possibles :

***

### 🔹 1. Vérification automatique via l’URL

* BetterAuth génère une **URL de vérification** (ex: `/api/auth/verify-email?token=...`).
* Lorsque l’utilisateur clique dessus :
  * 🔑 Son email est automatiquement marqué comme **vérifié** dans la base.
  * 🌍 L’utilisateur est **redirigé** vers l’URL définie dans **`callbackURL`** (par exemple : `/dashboard` ou `/welcome`).

📌 Vous n’avez rien à faire côté client, BetterAuth gère tout.

***

### 🔹 2. Vérification manuelle (custom flow)

Si vous souhaitez créer votre **propre lien personnalisé** (par ex. `/verify?token=xxx`), vous devez utiliser la méthode **`verifyEmail()`** côté client pour valider le token.

#### Exemple :

```ts
await authClient.verifyEmail({
  query: {
    token: "" // Le token récupéré depuis l’URL personnalisée
  }
})
```

👉 Fonctionnement :

1.  Vous envoyez un email avec votre propre lien, par exemple :

    ```
    https://monapp.com/verify?token=abc123
    ```
2. Dans la page `/verify`, vous extrayez le `token` depuis l’URL (`window.location.search`).
3. Vous appelez **`authClient.verifyEmail()`** avec ce token.
4. Si le token est valide → l’email de l’utilisateur est vérifié ✅.

***

### 🚀 Résumé du flow utilisateur

1. 👤 L’utilisateur s’inscrit (ou demande un nouvel email de vérification).
2. 📩 Il reçoit un email avec un **lien contenant un token**.
3. 🔗 Il clique sur ce lien → BetterAuth vérifie automatiquement son email **ou** vous le faites via **`verifyEmail()`** si vous avez un flow personnalisé.
4. 🎉 Son compte est désormais **activé et vérifié**.

## 🔑 Connexion automatique après vérification (Auto Sign In After Verification)

Par défaut, après avoir cliqué sur le lien de vérification, l’utilisateur est **redirigé** vers votre `callbackURL`.\
👉 Mais il n’est **pas connecté automatiquement**.

Si vous voulez fluidifier le parcours et éviter qu’il doive ressaisir son email + mot de passe, vous pouvez activer l’option **`autoSignInAfterVerification`**.

***

### ⚙️ Exemple d’activation

```ts
// auth.ts
import { betterAuth } from "better-auth"

const auth = betterAuth({
  // ... vos autres options
  emailVerification: {
    autoSignInAfterVerification: true
  }
})
```

***

### 🚀 Fonctionnement

* L’utilisateur s’inscrit → reçoit un email de vérification.
* Il clique sur le lien envoyé (avec le token).
* BetterAuth **vérifie l’email** ✅.
* Et immédiatement après → il est **connecté automatiquement** 🎉.

***

### 🎯 Avantages

* ✅ **Expérience fluide** : pas besoin de refaire un `signIn`.
* ✅ **Gain de temps** : surtout utile après l’inscription.
* ✅ **Réduction du churn** : moins de frictions = plus d’utilisateurs actifs.

***

### ⚠️ Points d’attention

* Cette option ne s’applique qu’à la **vérification par email + mot de passe** (pas forcément aux SSO).
* Vérifiez que vous redirigez bien l’utilisateur vers une page pertinente après connexion (ex: `/dashboard`).

***

👉 En résumé :\
Avec **`autoSignInAfterVerification: true`**, BetterAuth permet à vos utilisateurs d’être **directement connectés** dès qu’ils confirment leur email → un parcours beaucoup plus agréable ✨.

## 🔄 Callback après une vérification réussie d’email

BetterAuth vous permet d’exécuter du **code personnalisé immédiatement après la confirmation d’un email**.\
👉 C’est très utile pour appliquer des effets secondaires :

* 🎁 Donner accès à une fonctionnalité spéciale (premium, beta, etc.).
* 📝 Enregistrer un événement dans vos logs ou analytics.
* 🔔 Envoyer une notification à l’équipe ou à l’utilisateur.
* 🔑 Activer des permissions supplémentaires (ex: rôle “membre vérifié”).

***

### ⚙️ Exemple d’utilisation

```ts
// auth.ts
import { betterAuth } from 'better-auth';

export const auth = betterAuth({
  emailVerification: {
    async afterEmailVerification(user, request) {
      // Votre logique personnalisée
      console.log(`${user.email} a bien vérifié son adresse email ✅`);
      
      // Exemple : activer un flag "verified"
      // await db.users.update({ where: { id: user.id }, data: { isVerified: true } });

      // Exemple : envoyer un email de bienvenue
      // await sendEmail({
      //   to: user.email,
      //   subject: "Bienvenue 🎉",
      //   text: "Merci d’avoir vérifié votre adresse email !"
      // });
    }
  }
})
```

***

### 📌 Détails importants

* La fonction reçoit en paramètres :
  * **`user`** → l’objet utilisateur dont l’email vient d’être confirmé.
  * **`request`** → la requête HTTP associée (ex: pour logger l’IP ou l’user-agent).
* Le callback est exécuté **après la mise à jour de l’email en "vérifié"**, donc vous êtes sûr que l’utilisateur est confirmé.

***

### 🎯 Cas d’usage concrets

1. **Onboarding amélioré** → donner accès au dashboard uniquement après vérification.
2. **Système de rôles** → ajouter automatiquement un rôle "verified\_user".
3. **Monitoring** → loguer chaque vérification réussie pour traquer les comportements suspects.
4. **Récompenses / bonus** → créditer des points, envoyer un coupon de bienvenue, etc.

***

👉 En résumé :\
Le callback **`afterEmailVerification`** est parfait pour exécuter du **code personnalisé après la confirmation d’un email**.\
Il vous donne la flexibilité d’ajouter des **actions business ou techniques** directement au cœur du processus 🎉.

## 🔑 Email de réinitialisation du mot de passe (Password Reset Email)

La **réinitialisation de mot de passe** permet aux utilisateurs de retrouver l’accès à leur compte en cas d’oubli.\
👉 BetterAuth simplifie énormément cette mise en place en gérant pour vous la génération du **token sécurisé** et la création du **lien de réinitialisation**.

***

### ⚙️ Étape 1 – Activer le Password Reset

Il suffit d’ajouter une fonction **`sendResetPassword`** dans la configuration `emailAndPassword`.\
Cette fonction est appelée automatiquement lorsque l’utilisateur déclenche une demande de reset.

#### Exemple :

```ts
// auth.ts
import { betterAuth } from 'better-auth';
import { sendEmail } from './email'; // votre fonction d’envoi d’emails

export const auth = betterAuth({
  emailAndPassword: {
    enabled: true, // active login email+password
    sendResetPassword: async ({ user, url, token }, request) => {
      await sendEmail({
        to: user.email, // 📧 adresse de l’utilisateur
        subject: 'Réinitialisation de votre mot de passe',
        text: `Cliquez sur ce lien pour réinitialiser votre mot de passe : ${url}`
      })
    }
  }
})
```

***

### 📌 Paramètres disponibles

La fonction `sendResetPassword` reçoit :

* **`user`** → l’objet utilisateur (contenant au moins `email`).
* **`url`** → l’URL générée par BetterAuth pour réinitialiser le mot de passe.
* **`token`** → le jeton sécurisé (utile si vous préférez construire votre propre URL personnalisée).
* **`request`** → l’objet requête HTTP (utile si vous voulez logger l’origine de la demande).

***

### 🔄 Flow utilisateur classique

1. 🔑 L’utilisateur clique sur **"Mot de passe oublié ?"**.
2. 📩 BetterAuth déclenche `sendResetPassword` → envoi d’un email avec un **lien sécurisé**.
3. 🔗 L’utilisateur clique sur le lien reçu.
4. 📝 Il est redirigé vers une page **"Nouveau mot de passe"** dans votre app.
5. ✅ Son mot de passe est mis à jour et il peut se reconnecter.

***

### 🔐 Alternatives possibles

BetterAuth est flexible :

* Utiliser un **email avec lien sécurisé (token)** → méthode expliquée ci-dessus.
* Utiliser un **OTP envoyé par email** (code à usage unique) → voir le guide **OTP Verification**.

***

### 🎯 En résumé

* BetterAuth fournit une API simple pour gérer la **réinitialisation de mot de passe**.
* Vous n’avez qu’à définir **comment envoyer l’email** (service SMTP, SendGrid, SES, etc.).
* Deux approches possibles :
  * Lien sécurisé avec token (flux standard).
  * OTP (code à usage unique par email).
