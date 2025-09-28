# 📌 Transmettre des données en profondeur avec le Contexte (Context)

Habituellement, tu transmets des informations d’un composant parent à un composant enfant via **les props**. Mais cela peut vite devenir verbeux et peu pratique si tu dois passer ces props à travers de nombreux composants intermédiaires, ou si plusieurs composants de ton application ont besoin de la même information.

👉 **Le Contexte (Context)** permet à un composant parent de rendre une information disponible à n’importe quel composant de l’arbre situé en dessous de lui—**peu importe sa profondeur**—sans avoir à la passer explicitement via des props.

***

### ✅ Tu vas apprendre

* Ce qu’est le **prop drilling** (forage de props)
* Comment remplacer un passage répétitif de props par le **Context**
* Les cas d’usage fréquents du Context
* Les alternatives courantes au Context

***

### 🔹 Le problème avec le passage de props

Passer des props est une excellente manière de transmettre **explicitement** des données dans ton arbre de composants, jusqu’aux composants qui en ont besoin.

Mais… cela devient vite **lourd et répétitif** quand :

* tu dois passer une prop **très profondément** dans l’arbre,
* ou si **plusieurs composants** différents ont besoin de cette prop.

Dans ce cas, le plus proche ancêtre commun peut se trouver très loin des composants qui utilisent réellement cette donnée. Souvent, cela oblige à **remonter l’état très haut** dans l’arborescence, ce qui mène à une situation appelée **prop drilling**.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

***

#### 📌 Illustration : Lifting State Up

📍 _Schéma_ :

* Un arbre avec trois composants.
* Le parent contient une bulle représentant une valeur (mise en évidence en violet).
* Cette valeur descend via les props vers deux enfants, qui sont eux aussi surlignés en violet.

***

#### 📌 Illustration : Prop Drilling

📍 _Schéma_ :

* Un arbre plus grand (10 nœuds).
* Le nœud racine contient une bulle représentant une valeur (violet).
* Cette valeur est transmise de parent à enfant, en passant **par plusieurs composants intermédiaires** qui ne l’utilisent pas, mais la transmettent seulement.
* Finalement, seuls certains composants feuilles utilisent vraiment cette valeur (surlignés en violet).

***

👉 Tu vois le problème ? Certains composants intermédiaires **n’ont pas besoin de cette donnée**, mais doivent quand même la recevoir et la retransmettre. Cela alourdit ton code et réduit sa lisibilité.

***

### 💡 La solution : React Context

Ne serait-ce pas génial s’il existait une façon de **“téléporter”** une donnée directement aux composants qui en ont besoin, sans passer par toute la chaîne des props ?

➡️ C’est exactement ce que propose **le Context de React**.

Avec Context :

* Le parent rend une valeur **disponible globalement** pour toute une branche de l’arbre.
* N’importe quel composant enfant peut alors la récupérer **directement**, sans passer par toutes les étapes intermédiaires.

## 🔹 Context : une alternative au passage de props

Le **Context** permet à un composant parent de fournir une donnée à **tout l’arbre de composants en dessous de lui**. C’est très pratique quand :

* plusieurs composants d’une branche ont besoin de la même info,
* ou quand tu veux éviter le **prop drilling** (passage répété de props à travers des composants intermédiaires inutiles).

***

### Exemple : titres avec niveaux (Headings)

#### Cas initial

On a un composant `<Heading>` qui accepte une prop `level` pour définir sa taille, et un composant `<Section>` pour structurer la page.

👉 Exemple :

```jsx
<Section>
  <Heading level={3}>About</Heading>
  <Heading level={3}>Photos</Heading>
  <Heading level={3}>Videos</Heading>
</Section>
```

Problème : tu dois **répéter `level={3}` sur chaque Heading**.

***

#### Objectif

Il serait plus pratique de pouvoir écrire :

```jsx
<Section level={3}>
  <Heading>About</Heading>
  <Heading>Photos</Heading>
  <Heading>Videos</Heading>
</Section>
```

Ainsi, le **`level` est défini une seule fois dans `<Section>`**, et tous les `<Heading>` enfants héritent de ce niveau automatiquement.

***

#### ⚡ Mais… comment un `<Heading>` peut-il savoir le `level` de sa `<Section>` la plus proche ?

👉 Pas possible uniquement avec les **props**.\
➡️ C’est là que **Context** entre en jeu.

***

### 📌 Mise en place en 3 étapes

1.  **Créer un contexte**\
    On crée un contexte pour stocker le niveau (ici `LevelContext`).

    ```jsx
    import { createContext } from 'react';

    export const LevelContext = createContext(1); // valeur par défaut = 1
    ```

***

2.  **Utiliser ce contexte dans le composant qui en a besoin**\
    Dans `<Heading>`, on utilise `useContext(LevelContext)` pour récupérer le niveau courant :

    ```jsx
    import { useContext } from 'react';
    import { LevelContext } from './LevelContext.js';

    export default function Heading({ children }) {
      const level = useContext(LevelContext);

      switch (level) {
        case 1:
          return <h1>{children}</h1>;
        case 2:
          return <h2>{children}</h2>;
        case 3:
          return <h3>{children}</h3>;
        case 4:
          return <h4>{children}</h4>;
        case 5:
          return <h5>{children}</h5>;
        case 6:
          return <h6>{children}</h6>;
        default:
          throw Error('Invalid heading level: ' + level);
      }
    }
    ```

***

3.  **Fournir ce contexte depuis le composant parent (`<Section>`)**\
    Dans `<Section>`, on utilise le composant **`LevelContext.Provider`** pour fournir la valeur aux enfants.

    ```jsx
    import { useContext } from 'react';
    import { LevelContext } from './LevelContext.js';

    export default function Section({ level, children }) {
      return (
        <LevelContext.Provider value={level}>
          <section className="section">{children}</section>
        </LevelContext.Provider>
      );
    }
    ```

***

### 🔥 Résultat final

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>Title</Heading>
      <Section level={2}>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section level={3}>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section level={4}>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

***

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

#### **Contexte avec enfants proches**

Un parent fournit une valeur (bulle orange), transmise directement à ses enfants :

📍 _Diagramme_ :

* Arbre avec 3 composants.
* Le parent contient une bulle orange (la valeur).
* La valeur se propage à ses 2 enfants, tous deux surlignés en orange.

***

#### **Contexte avec enfants éloignés**

Un parent fournit une valeur (bulle orange), transmise à des composants **très profonds dans l’arbre**.

📍 _Diagramme_ :

* Arbre avec 10 nœuds.
* Le parent racine contient une bulle orange.
* La valeur est transmise **directement** à 4 feuilles et 1 composant intermédiaire, sans que les autres composants aient besoin de la passer explicitement.

***

👉 Grâce à **Context**, un composant peut “téléporter” une donnée à ses descendants sans avoir à modifier tous les composants intermédiaires.

## 🟢 Étape 1 : Créer le contexte

Pour utiliser le **Context** dans React, tu dois d’abord le **créer** avec `createContext`.\
On le place dans un fichier séparé (ex: `LevelContext.js`) afin qu’il puisse être importé partout.

***

#### 👉 Exemple de code

**LevelContext.js**

```jsx
import { createContext } from 'react';

// On crée un contexte avec une valeur par défaut = 1 (plus grand titre <h1>)
export const LevelContext = createContext(1);
```

***

#### ⚡ Explications

* `createContext` prend un **argument** : la **valeur par défaut**.
* Ici, on met `1` car on l’utilisera comme **niveau de titre** (`<h1>`, `<h2>`, etc.).
* Tu pourrais très bien y mettre une **chaîne de caractères, un booléen ou même un objet** selon ton besoin.
* Cette valeur par défaut sera utilisée **si aucun Provider ne fournit de valeur** (tu verras ça à l’étape suivante).

***

📌 Exemple d’arborescence de fichiers :

```
src/
 ├── App.js
 ├── Section.js
 ├── Heading.js
 └── LevelContext.js   ✅ Contexte créé ici
```

***

👉 Prochaine étape (Step 2) : **utiliser ce contexte dans un composant consommateur (Heading)**.

## 🟠 Étape 2 : Utiliser le contexte dans `Heading`

Jusqu’ici, ton composant `<Heading>` recevait un prop `level`.\
Maintenant, on va **supprimer ce prop** et demander directement à React la valeur via `useContext`.

***

#### 👉 Exemple de code

**Heading.js**

```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  // On lit la valeur du contexte
  const level = useContext(LevelContext);

  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Niveau de titre inconnu : ' + level);
  }
}
```

***

#### ⚡ Explications

* On **importe** `useContext` de React.
* On lit la valeur partagée avec `useContext(LevelContext)`.
* Désormais, le composant **n’a plus besoin de prop `level`** 🎉.
* Tous les `<Heading>` utilisent le **niveau fourni par le contexte**.

***

#### 👉 Mise à jour du JSX dans **App.js**

Avant :

```jsx
<Section>
  <Heading level={4}>Sub-sub-heading</Heading>
  <Heading level={4}>Sub-sub-heading</Heading>
  <Heading level={4}>Sub-sub-heading</Heading>
</Section>
```

Après :

```jsx
<Section level={4}>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
</Section>
```

***

📌 Résultat intermédiaire :\
Pour le moment, **tous les titres ressortent en `<h1>`**, car on utilise encore la valeur **par défaut (1)** du contexte.\
👉 On corrigera ça à la prochaine étape, quand **chaque `<Section>` fournira son propre contexte**.

## 🟠 Étape 3 : Fournir le contexte depuis `<Section>`

Pour que chaque `<Heading>` connaisse automatiquement son niveau, il faut que `<Section>` fournisse cette valeur à ses enfants via le **Provider** du contexte.

***

#### 👉 Exemple de code

**Section.js**

```jsx
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

***

#### ⚡ Explications

* On utilise `<LevelContext.Provider>` (et non pas directement `<LevelContext>`).
* La prop `value={level}` dit à React :\
  &#xNAN;_« Tous les composants descendants qui appellent `useContext(LevelContext)` recevront cette valeur. »_
* Si plusieurs `Provider` existent dans l’arbre, un composant enfant prendra la **valeur du Provider le plus proche**.

***

#### 👉 Exemple complet

**App.js**

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>Title</Heading>
      <Section level={2}>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section level={3}>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section level={4}>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

***

#### ✅ Résultat

* Tu passes **`level` uniquement aux `<Section>`**.
* Chaque `<Heading>` détermine son niveau automatiquement via le **contexte**.
* Tu n’as plus besoin de dupliquer `level={3}`, `level={4}`, etc. sur chaque `<Heading>`. 🎉

***

📌 Résumé du fonctionnement :

1. Tu passes `level` à `<Section>`.
2. `<Section>` entoure ses enfants avec `<LevelContext.Provider value={level}>`.
3. Chaque `<Heading>` lit la valeur avec `useContext(LevelContext)` et rend le bon `<h1>…<h6>`.

## 🟠 Utiliser et fournir le contexte dans le même composant

Avant, on devait écrire manuellement les niveaux :

```jsx
<Section level={1}>
  <Heading>Title</Heading>
  <Section level={2}>
    <Heading>Heading</Heading>
    <Section level={3}>
      <Heading>Sub-heading</Heading>
    </Section>
  </Section>
</Section>
```

➡️ Avec le contexte, chaque `<Section>` peut lire son **niveau actuel** et fournir automatiquement `level + 1` à ses enfants.

***

#### 👉 Exemple de code

**Section.js**

```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  // Récupère le niveau actuel depuis le contexte
  const level = useContext(LevelContext);

  return (
    <section className="section">
      {/* Fournit level + 1 à ses enfants */}
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

***

#### 👉 Exemple complet

**App.js**

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading>Title</Heading>
      <Section>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

***

#### ⚡ Résultat

* Le **premier `<Section>`** part de la valeur **1** (défaut défini dans `LevelContext`).
* Chaque `<Section>` lit ce niveau et le transmet **incrémenté de 1** à ses enfants.
* Les `<Heading>` affichent automatiquement le bon `<h1>`, `<h2>`, `<h3>`, etc.
* Tu n’as **plus besoin de passer de prop `level` nulle part** 🎉.

***

📌 **Résumé du mécanisme** :

1. `LevelContext` est créé avec une valeur par défaut `1`.
2. `<Section>` lit la valeur avec `useContext(LevelContext)`.
3. `<Section>` fournit à ses enfants `level + 1`.
4. `<Heading>` lit la valeur courante avec `useContext(LevelContext)`.

## 🟠 Le contexte traverse les composants intermédiaires

👉 Tu peux insérer autant de composants que tu veux entre le composant **qui fournit un contexte** (`Provider`) et celui **qui l’utilise** (`useContext`).

* Peu importe que ce soit des composants React que tu crées (`<Post />`, `<RecentPosts />`) ou des composants natifs (`<div>`, `<section>`).
* Chaque composant **descendant** a accès au contexte, sauf si un composant intermédiaire le **remplace** par un nouveau Provider.

***

### 🔹 Exemple complet

#### **App.js**

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function ProfilePage() {
  return (
    <Section>
      <Heading>My Profile</Heading>
      <Post
        title="Hello traveller!"
        body="Read about my adventures."
      />
      <AllPosts />
    </Section>
  );
}

function AllPosts() {
  return (
    <Section>
      <Heading>Posts</Heading>
      <RecentPosts />
    </Section>
  );
}

function RecentPosts() {
  return (
    <Section>
      <Heading>Recent Posts</Heading>
      <Post
        title="Flavors of Lisbon"
        body="...those pastéis de nata!"
      />
      <Post
        title="Buenos Aires in the rhythm of tango"
        body="I loved it!"
      />
    </Section>
  );
}

function Post({ title, body }) {
  return (
    <Section isFancy={true}>
      <Heading>{title}</Heading>
      <p><i>{body}</i></p>
    </Section>
  );
}
```

***

### ⚡ Ce qu’il se passe

* Le **premier `<Section>`** définit un `level = 1`.
* Chaque `<Section>` suivant incrémente automatiquement le niveau grâce au `Provider` (`level + 1`).
* Peu importe que tu insères `<Post />`, `<RecentPosts />` ou `<div>` entre eux, le `<Heading>` retrouve **le bon niveau**.

***

#### 🖼️ Illustration

**Propagation du contexte à travers la hiérarchie :**

* `<Section>` (level 1)
  * `<Heading>` → `<h1>`
  * `<Post>` → contient un `<Heading>` qui devient automatiquement `<h2>`
  * `<AllPosts>` → `<Section>` (level 2)
    * `<Heading>` → `<h2>`
    * `<RecentPosts>` → `<Section>` (level 3)
      * `<Heading>` → `<h3>`
      * `<Post>` → `<Heading>` → `<h4>`

➡️ Tu n’as rien eu à passer manuellement : le **contexte traverse les couches intermédiaires** ✨.

***

### 🔹 Analogie avec le CSS

Le fonctionnement du contexte ressemble beaucoup à l’héritage des propriétés CSS :

*   En CSS :

    ```css
    div { color: blue; }
    ```

    Tous les enfants héritent de `color: blue`, sauf si un enfant redéfinit `color: green`.
* En React avec Context :
  * `LevelContext.Provider value={2}` → tous les descendants utilisent `level = 2`,
  * sauf si un descendant redéfinit un autre `Provider` avec `value={3}`.

## 🔹 Avant d’utiliser le contexte

Le **contexte** est un outil puissant, mais justement parce qu’il est puissant, il est souvent **surutilisé**.\
➡️ Le réflexe doit être : **est-ce que je peux faire plus simple avec des props ?**

***

### ✅ Alternatives au contexte

#### 1. **Commence par passer les props**

* Même si ça paraît “lourd” de passer une prop à travers plusieurs composants, c’est en réalité **clair et explicite**.
* Tu sais immédiatement quel composant utilise quelle donnée.
* Un futur développeur qui lit ton code sera content de voir **le flux de données bien défini**.

💡 Exemple :

```jsx
function App() {
  return <Layout user="Alice" />;
}

function Layout({ user }) {
  return <Sidebar user={user} />;
}

function Sidebar({ user }) {
  return <UserInfo user={user} />;
}
```

***

#### 2. **Extraire des composants et utiliser `children`**

Souvent, si tu te retrouves à **passer des props à des composants intermédiaires qui ne les utilisent pas**, c’est peut-être que tu n’as pas assez découpé ton interface.

➡️ Solution : utiliser la composition avec `children`.

💡 Exemple AVANT (prop drilling inutile) :

```jsx
function Layout({ posts }) {
  return <Sidebar posts={posts} />;
}

function Sidebar({ posts }) {
  return <Posts posts={posts} />;
}
```

💡 Exemple APRÈS (composition plus claire) :

```jsx
function Layout({ children }) {
  return <div className="layout">{children}</div>;
}

function App() {
  return (
    <Layout>
      <Posts posts={posts} />
    </Layout>
  );
}
```

Ici, `Layout` ne se préoccupe plus des `posts`. Il gère seulement la mise en page, et tu mets directement le composant qui en a besoin (`<Posts />`) comme enfant.

***

#### 3. **Et si ça ne marche pas ?**

➡️ C’est seulement **si ni les props ni la composition ne suffisent** que tu peux envisager le **contexte**.

👉 Le contexte est pertinent quand :

* Beaucoup de composants distants ont besoin de la même donnée (ex. : **thème de couleur, utilisateur connecté, langue courante**).
* Les props deviennent vraiment trop lourdes à gérer.

***

⚡ **Règle d’or** : Utilise le contexte avec parcimonie.\
➡️ Passe d’abord par les props et la composition, et seulement si c’est trop compliqué, alors introduis un `Context`.

## 🔹 Cas d’utilisation du contexte

Le **contexte** est utile quand certaines données doivent être accessibles **à de nombreux composants distants dans l’arbre**. Voici les cas les plus fréquents :

***

### 🎨 1. Thématisation (Theming)

* Exemple : **mode clair/sombre**, choix de couleurs, taille de police.
* On place un **provider** de thème tout en haut de l’application.
* Tous les composants peuvent ensuite lire le thème sans avoir besoin qu’il soit passé comme prop.

💡 Exemple :

```jsx
<ThemeContext.Provider value="dark">
  <App />
</ThemeContext.Provider>
```

***

### 👤 2. Compte utilisateur actuel

* Presque toutes les applis ont besoin de savoir **quel utilisateur est connecté**.
* Mettre l’utilisateur courant dans un contexte permet à n’importe quel composant (profil, menu, bouton logout, etc.) d’y accéder.

⚡ Exemple avancé : Certaines applis (comme Slack ou Gmail) permettent de se connecter avec plusieurs comptes. On peut alors **imbriquer plusieurs providers** pour gérer chaque sous-section avec un compte différent.

***

### 🧭 3. Routage

* La majorité des librairies de routing (React Router, Next.js, etc.) utilisent un **contexte en interne**.
* C’est ainsi que chaque `<Link>` sait s’il est actif ou non en fonction de la route courante.

💡 Si tu construis ton propre routeur maison, le contexte sera une solution idéale pour gérer la route active.

***

### 🔄 4. Gestion d’état complexe

* Quand l’état global de ton app devient trop gros pour être géré uniquement avec `useState` local.
* Combiner **`useReducer` + `Context`** permet de :
  * Centraliser la logique dans un reducer.
  * Fournir l’état et `dispatch` à tous les composants, même éloignés.

💡 Exemple : gestion d’un panier e-commerce, avec ajout/retrait/mise à jour d’articles.

***

### ⚡ Important : Contexte et état

* Le contexte **n’est pas limité à des valeurs statiques**.
* Si la valeur fournie change au prochain rendu, **tous les composants qui l’utilisent se mettent à jour automatiquement**.
* C’est pourquoi on combine souvent `Context` + `useState` (ou `useReducer`).

***

## 📝 Récapitulatif

* Le **contexte** permet à un composant de fournir une donnée à tout son sous-arbre.
* Pour l’utiliser :
  1.  Crée-le :

      ```js
      export const MyContext = createContext(defaultValue);
      ```
  2.  Lis-le dans un enfant :

      ```js
      const value = useContext(MyContext);
      ```
  3.  Fournis-le dans un parent :

      ```jsx
      <MyContext.Provider value={...}>
        {children}
      </MyContext.Provider>
      ```
* Le contexte traverse tous les composants intermédiaires (il “téléporte” la donnée).
* Il permet de créer des composants qui **s’adaptent à leur environnement**.
* ⚠️ Avant de l’utiliser, essaie d’abord de **passer des props** ou de **composer avec `children`**.
