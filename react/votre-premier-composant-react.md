# 🎯 Votre premier composant React



Avec le **HTML classique**, tu disposes déjà de briques toutes faites comme `<h1>` (titre), `<ul>` (liste), `<button>` (bouton).\
Avec **React**, tu peux inventer tes propres briques personnalisées, par exemple :

* `<Profile />` pour afficher un profil,
* `<Gallery />` pour afficher une galerie,
* `<TableOfContents />` pour une table des matières.

***

### 🏗 Exemple sans React (HTML pur)

Voici du HTML classique :

```html
<article>
  <h1>Mon premier composant</h1>
  <ol>
    <li>Les composants : les blocs de construction de l’UI</li>
    <li>Définir un composant</li>
    <li>Utiliser un composant</li>
  </ol>
</article>
```

👉 Ici on décrit un **article** avec un titre (`<h1>`) et une table des matières (`<ol>`).\
Mais ce bloc est figé : si tu veux le répéter, tu dois **copier-coller**.

***

### ⚛️ Exemple avec React (composant)

Avec React, tu transformes ce bloc en **composant réutilisable** :

```jsx
function TableOfContents() {
  return (
    <article>
      <h1>Mon premier composant</h1>
      <ol>
        <li>Les composants : les blocs de construction de l’UI</li>
        <li>Définir un composant</li>
        <li>Utiliser un composant</li>
      </ol>
    </article>
  );
}
```

Ensuite, tu peux l’utiliser partout avec `<TableOfContents />`, comme si c’était une balise HTML inventée par toi.

***

### 🔄 Composition de composants

Un composant peut contenir d’autres composants. Par exemple, une page peut être composée comme ceci :

```jsx
<PageLayout>
  <NavigationHeader>
    <SearchBar />
    <Link to="/docs">Docs</Link>
  </NavigationHeader>
  <Sidebar />
  <PageContent>
    <TableOfContents />
    <DocumentationText />
  </PageContent>
</PageLayout>
```

👉 Ici :

* `<PageLayout>` est le **parent**,
* à l’intérieur on retrouve plusieurs enfants (`<NavigationHeader>`, `<Sidebar>`, `<PageContent>`),
* eux-mêmes contiennent encore d’autres composants (`<SearchBar />`, `<TableOfContents />`, etc.).

C’est comme des **poupées russes** 🪆 : chaque composant peut contenir d’autres composants, ce qui te permet de construire une application entière.

***

### 🚀 Avantages

* **Réutilisation** : tu écris une fois, tu utilises partout (`<TableOfContents />` peut apparaître sur toutes les pages).
* **Organisation** : tu découpes ton application en petits morceaux clairs.
* **Communauté** : tu peux utiliser des milliers de composants déjà prêts (Material UI, Chakra UI, etc.).

## Définir un composant React

### 1. Le principe

Un **composant React** est tout simplement une **fonction JavaScript** qui retourne du **JSX** (un mélange de JavaScript + HTML).

👉 Avant :\
Les devs écrivaient une page en HTML, puis ajoutaient un peu de JavaScript “par-dessus” pour l’interactivité.\
👉 Aujourd’hui :\
Avec React, **l’interactivité est au cœur du composant**, dès le départ.

***

### 2. Exemple concret : un composant `Profile`

```jsx
export default function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3Am.jpg"
      alt="Katherine Johnson"
    />
  )
}
```

Ce code crée un **composant réutilisable** appelé `Profile`.\
Quand tu écris `<Profile />` dans ton app, ça affiche l’image de Katherine Johnson.

***

### 3. Étapes pour construire un composant

#### ✅ Étape 1 : Exporter le composant

```jsx
export default
```

C’est du **JavaScript pur** (rien de spécial à React).

* Ça veut dire “j’autorise ce composant à être importé ailleurs dans l’application”.
* Si tu ne l’exportes pas → tu ne pourras pas l’utiliser dans d’autres fichiers.

***

#### ✅ Étape 2 : Définir la fonction

```jsx
function Profile() { ... }
```

Tu déclares une fonction normale.\
⚠️ **Très important : son nom doit commencer par une majuscule** (`Profile`, pas `profile`).\
Sinon React pensera que c’est une balise HTML (`<profile>`) et ça ne marchera pas.

***

#### ✅ Étape 3 : Ajouter du balisage (JSX)

Le composant doit **retourner** du JSX :

```jsx
return (
  <img
    src="https://i.imgur.com/MK3eW3Am.jpg"
    alt="Katherine Johnson"
  />
)
```

👉 Ici, ça ressemble à du HTML, mais en réalité c’est **du JavaScript** (grâce au JSX).\
C’est ce qui permet de mélanger la logique JS et le rendu visuel.

***

### 4. Deux façons d’écrire le `return`

#### En une seule ligne :

```jsx
return <img src="..." alt="Katherine Johnson" />;
```

#### Sur plusieurs lignes (recommandé si c’est long) :

```jsx
return (
  <div>
    <img src="..." alt="Katherine Johnson" />
  </div>
);
```

⚠️ **Piège classique :**\
Si tu écris `return` puis ton JSX sur la ligne d’après **sans parenthèses**, le code sera ignoré.\
Exemple qui ne marche pas :

```jsx
return 
  <img src="..." alt="Katherine Johnson" />
```

👉 Le navigateur considère que `return` est **vide** et ne lit pas ton `<img />`.

***

### 🧠 À retenir

1. Un composant React = une fonction JS qui retourne du JSX.
2. Toujours commencer le nom par une **majuscule**.
3. Toujours **exporter** ton composant si tu veux le réutiliser ailleurs.
4. Quand tu retournes du JSX sur plusieurs lignes → utilise des **parenthèses**.

## ⚛️ Utiliser un composant React

### 1. Exemple concret

```jsx
function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Scientifiques de renom</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

👉 Ici :

* `Profile` est ton **composant enfant**
* `Gallery` est ton **composant parent**
* `Gallery` réutilise `Profile` **3 fois**, comme si tu copiais-collais du HTML, mais en plus **propre et réutilisable**.

***

### 2. Ce que voit le navigateur

React transforme tes composants en **HTML classique**.\
Ton code devient :

```html
<section>
  <h1>Scientifiques de renom</h1>
  <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
</section>
```

💡 Remarque importante :

* `<section>` en minuscule → React comprend que c’est une **balise HTML**.
* `<Profile />` en majuscule → React comprend que c’est un **composant React** que TU as créé.

***

### 3. Organisation des composants

* Tu peux mettre plusieurs petits composants dans un même fichier (`Gallery.js`).
* Si ça devient trop lourd → tu peux mettre chaque composant dans un fichier séparé (`Profile.js`, `Gallery.js`).

⚠️ **Erreur à éviter** :\
Ne définis pas un composant _à l’intérieur d’un autre composant_ :

```jsx
export default function Gallery() {
  // ❌ Mauvaise pratique
  function Profile() {
    return <img src="..." alt="..." />;
  }
  return <Profile />;
}
```

Cela cause des bugs et des lenteurs.\
✅ Toujours définir tes composants au **niveau racine** du fichier (tout en haut, pas imbriqués dans une autre fonction).

***

### 4. Parent & enfant

* `Gallery` est le **parent**
* `Profile` est un **enfant**

Le parent peut afficher autant d’enfants qu’il veut.\
👉 C’est ça la magie de React : tu écris un composant **une seule fois** et tu l’utilises **autant de fois que nécessaire**.

***

### 🧠 À retenir

1. Tu peux **réutiliser** un composant partout en écrivant `<MonComposant />`.
2. Les balises en **minuscules** → HTML natif ; en **Majuscules** → composants React.
3. Ne **jamais définir un composant dans un autre** → toujours au niveau racine.
4. Les composants peuvent être organisés dans un même fichier ou séparés en plusieurs fichiers.

## 🔎 L’idée clé

En React, **tout est composant** — pas seulement les petits boutons. Tu vas des **atomes** (bouton, avatar) jusqu’aux **organismes** (sidebar, liste) et **pages entières**. Cette approche te sert à **organiser l’UI et le code** de façon cohérente, même pour des éléments utilisés une seule fois.

***

### 🧷 Le composant racine (Root)

Ton appli démarre toujours par un **composant racine** : c’est **le point d’entrée** de l’UI.

*   **Sans framework (Vite, CRA, etc.)**\
    Tu montes ton composant racine dans un nœud DOM, souvent `#root`.

    ```jsx
    // main.jsx
    import { createRoot } from "react-dom/client";
    import App from "./App.jsx";

    createRoot(document.getElementById("root")).render(<App />);
    ```
* **Avec Next.js (pages router historique)**\
  Le “composant racine” de la page est celui exporté par `pages/index.js` (ou `.tsx`).\
  Next se charge du montage et du routage.
* **Avec Next.js (App Router moderne)**\
  Le point d’entrée visuel d’une route est `app/page.tsx`, enveloppé par `app/layout.tsx`.\
  Ces fichiers définissent la **structure racine** de ta page. _(Même idée : un composant racine par route.)_

> Retenir : il y a toujours un composant qui **démarre l’arbre** et sous lequel **tout le reste est imbriqué**.

***

### 🧱 Des composants “jusqu’au bout”

* Tu **n’utilises pas React uniquement pour les widgets réutilisables** (boutons, cards…) : tu l’utilises **aussi pour les grandes zones** (navigation, sidebar, listes, modales) et **jusqu’aux pages entières**.
* Bénéfices :
  * **Lisibilité** (tu découpes l’UI en morceaux logiques)
  * **Réutilisation** (tu factorises, tu évites le copier-coller)
  * **Testabilité** (chaque pièce est testable isolément)

***

### 🖨️ Génération de HTML côté serveur (SSR/SSG) & Hydratation

Les **frameworks React** (Next.js, Remix, etc.) vont plus loin que “React dans un HTML vide” :

1. **Ils génèrent du HTML** à partir de tes composants **avant** que le JavaScript du navigateur ne soit chargé.
   * Résultat : **premier rendu rapide**, meilleur **SEO**, **meilleure accessibilité**.
2. Une fois le JS chargé, React **hydrate** ce HTML :
   * Il **attache les écouteurs d’événements** et **réactive** l’interactivité sur le markup déjà présent.
   * Tu profites du **meilleur des deux mondes** : rendu initial rapide + UI interactive.

> Dans Next.js :
>
> * **SSG** (Static Site Generation) : HTML pré-construit au build.
> * **SSR** (Server-Side Rendering) : HTML généré à la demande.
> * **ISR** (Incremental Static Regeneration) : HTML statique régénéré périodiquement.
> * **RSC** (React Server Components) : une partie de l’arbre s’exécute côté serveur (moins de JS côté client), et tu marques ce qui doit être interactif avec des _client components_ (`"use client"`).

***

### 🧩 Plusieurs racines sur une même page (progressive enhancement)

Tu n’es **pas obligé** de faire “toute la page en React”.\
Sur un site existant (WordPress, Rails, ou HTML “classique”), tu peux ajouter **plusieurs petites îlots React** :

```html
<div id="cart-root"></div>
<div id="comments-root"></div>
<script type="module">
  import { createRoot } from "react-dom/client";
  import Cart from "/Cart.js";
  import Comments from "/Comments.js";

  createRoot(document.getElementById("cart-root")).render(<Cart />);
  createRoot(document.getElementById("comments-root")).render(<Comments />);
</script>
```

* Avantages :
  * **Migration progressive** : tu modernises par morceaux.
  * **Moins de JS global** si la page est majoritairement statique.
* Inconvénients :
  * Plusieurs racines = **plusieurs arbres séparés** → pense à la **cohérence d’état** et au **partage de styles**.

***

### 🗺️ Quand choisir quoi ?

* **Appli full React (une racine)**
  * SPA ou Next.js complet.
  * Idéal pour des apps riches, navigation client fluide, état global (Redux/Zustand), UI très interactive.
* **Plusieurs racines React (îlots)**
  * Site existant où tu veux **injecter de l’interactivité** uniquement à certains endroits (panier, commentaires, recherche, etc.).
  * **Migration en douceur** sans tout réécrire.
* **Next.js / SSR / RSC**
  * Tu veux **performances initiales**, **SEO**, **streaming**, et **réduire le JS côté client**.
  * Tu composes serveur + client pour n’envoyer au navigateur **que le JS nécessaire**.

***

### ⚠️ Pièges & bonnes pratiques

* **Ne définis jamais un composant à l’intérieur d’un autre composant** (définitions imbriquées) → recréation à chaque render, pertes de perfs et bugs.\
  ✅ Déclare-les **au niveau module** (top-level).
* **Nom en Majuscule** pour les composants (`<Profile />`, pas `<profile />`), sinon React le traite comme une balise HTML.
* **Évite la sur-fragmentation** : trop de micro-composants rend le code verbeux. Découpe avec bon sens (lisibilité > fétichisme de la granularité).
* **Partage d’état** : avec plusieurs racines, l’état global est plus difficile à synchroniser. En full app, utilise un store (Redux, Zustand) ou le contexte.

***

### 🧠 TL;DR

* Il y a toujours un **composant racine** qui démarre l’arbre React.
* **Tout peut être un composant** : du bouton à la page, en passant par les grandes sections.
* Les frameworks **génèrent du HTML** à partir de tes composants (SSR/SSG), puis **hydratent** pour l’interactivité.
* Tu peux utiliser **une seule racine** (appli complète) ou **plusieurs racines** (îlots React) selon ton besoin.
* **Bonnes pratiques** : composants au top-level, noms en Majuscule, découpage raisonnable.
