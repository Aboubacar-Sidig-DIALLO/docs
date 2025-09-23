# ✨ Écrire du balisage avec JSX

### 1. Qu’est-ce que JSX ?

* JSX = **JavaScript XML**
* C’est une **extension de syntaxe JavaScript** qui te permet d’écrire du **balisage qui ressemble à du HTML**, directement **dans ton fichier JavaScript**.
* Exemple :

```jsx
function App() {
  return (
    <h1>Hello, React !</h1>
  );
}
```

👉 Ici, tu écris `<h1>` **comme en HTML**, mais en réalité ce n’est **pas du HTML** → c’est du **JSX**, qui sera transformé en appels JavaScript par React (`React.createElement(...)`).

***

### 2. Pourquoi React mélange balisage et logique ?

Dans le web traditionnel :

* Tu avais **HTML** dans un fichier `.html`
* **CSS** dans un fichier `.css`
* **JavaScript** dans un fichier `.js`

👉 Résultat : la logique et le rendu étaient **séparés artificiellement**, alors qu’en réalité ils sont liés.

React casse ce cloisonnement et dit :

* **Un composant = la logique + le rendu** dans une seule “boîte” (fonction).
* Avantage : chaque composant est **autonome, réutilisable et facile à raisonner**.

***

### 3. Différences entre JSX et HTML

À première vue, JSX **ressemble à du HTML**, mais il y a des différences importantes :

#### 🔹 Noms des attributs

* En HTML → `class`, `for`
* En JSX → `className`, `htmlFor`\
  Car `class` et `for` sont des **mots réservés en JavaScript**.

Exemple :

```jsx
<h1 className="title">Bonjour</h1>
<label htmlFor="email">Email :</label>
```

***

#### 🔹 Tout doit être dans une seule balise parent

En HTML, tu peux écrire plusieurs balises côte à côte.\
En JSX, tout doit être **enveloppé dans un parent unique**.

❌ Erreur :

```jsx
return (
  <h1>Hello</h1>
  <p>World</p>
);
```

✅ Correct :

```jsx
return (
  <div>
    <h1>Hello</h1>
    <p>World</p>
  </div>
);
```

Ou avec un fragment (`<> ... </>`) :

```jsx
return (
  <>
    <h1>Hello</h1>
    <p>World</p>
  </>
);
```

***

#### 🔹 Expressions JS entre `{ }`

Tu peux insérer du JavaScript **dans ton balisage JSX** grâce aux accolades `{ }`.

Exemple :

```jsx
const name = "Katherine Johnson";

function App() {
  return <h1>Scientifique : {name}</h1>;
}
```

👉 Les `{ }` permettent d’évaluer du JS (variables, fonctions, calculs…).

***

### 4. Afficher des informations avec JSX

Puisque JSX est du JavaScript, tu peux :

* Insérer des **variables**
* Faire des **calculs**
* Appeler des **fonctions**

Exemple :

```jsx
const user = {
  firstName: "Marie",
  lastName: "Curie"
};

function App() {
  return (
    <h1>
      Bienvenue {user.firstName} {user.lastName} !
    </h1>
  );
}
```

👉 Résultat affiché :

```
Bienvenue Marie Curie !
```

***

### 🧠 TL;DR

1. JSX = mélange **JavaScript + balisage HTML-like**.
2. C’est **pas du HTML**, mais ça y ressemble → transformé en JS par React.
3. Différences clés :
   * `className` au lieu de `class`, `htmlFor` au lieu de `for`.
   * Un seul parent par `return`.
   * Expressions JS entre `{ }`.
4. Avantage : **composants autonomes et lisibles**, où le rendu et la logique vivent ensemble.

## ✨ JSX : mettre du balisage dans JavaScript

### 1. Le Web traditionnel : séparation stricte

Pendant longtemps, le développement web se basait sur une **séparation stricte** :

* **HTML** → pour le contenu
* **CSS** → pour la présentation
* **JavaScript** → pour l’interactivité

Et bien souvent dans **des fichiers séparés**.

***

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Un exemple de balisage HTML simple :

```html
<div>
  <p>Bonjour</p>
  <form>
    ...
  </form>
</div>
```

👉 Ici, tu vois le contenu (un paragraphe et un formulaire), mais **aucune logique**.

***

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Le JavaScript était **ailleurs** :

```js
function onSubmit() { ... }
function onLogin() { ... }
function onClick() { ... }
```

👉 Ici, tu vois la **logique**, mais elle est **coupée** de son balisage HTML.

***

### 2. Quand le Web devient interactif

Avec le temps, le JavaScript a commencé à **contrôler le contenu** :

* Il créait ou modifiait des balises HTML,
* Il réagissait aux événements,
* Et il devenait en pratique responsable de ce que l’utilisateur voyait.

👉 Ce cloisonnement (HTML d’un côté, JS de l’autre) devenait lourd et source d’erreurs.

***

### 3. La solution React : regrouper logique + balisage

React change la donne en disant :\
➡️ **La logique de rendu et le balisage doivent vivre ensemble, dans les composants.**

Chaque composant devient **une boîte autonome** qui contient :

* Sa logique (conditions, fonctions, événements),
* Son balisage (JSX).

***

```jsx
export default function Sidebar() {
  const isLoggedIn = true;

  return (
    <div>
      <p>Bonjour</p>
      <Form />
    </div>
  );
}
```

👉 Ici, on mélange :

* la **logique** (`isLoggedIn`),
* le **balisage** (`<p>` + `<Form />`).

***

```jsx
export default function Form() {
  function onSubmit(e) { ... }
  function onClick() { ... }

  return (
    <form onSubmit={onSubmit}>
      <input type="text" onClick={onClick} />
      <input type="password" onClick={onClick} />
    </form>
  );
}
```

👉 Ici, la logique (`onSubmit`, `onClick`) est **directement liée** au balisage (`<form>`, `<input>`).

***

### 4. Pourquoi c’est mieux ?

* ✅ **Synchronisation garantie** : la logique et le rendu évoluent ensemble.
* ✅ **Isolation** : la logique d’un bouton ne risque pas d’être mélangée avec celle d’une sidebar.
* ✅ **Réutilisabilité** : chaque composant est une brique autonome.

***

### 5. JSX en deux mots

* JSX est une **extension de syntaxe JavaScript**.
* Il ressemble à du HTML, mais **plus strict** (ex. `className` au lieu de `class`).
* Il permet d’**afficher du contenu dynamique** avec `{ ... }`.

👉 Exemple simple :

```jsx
const name = "Katherine Johnson";

function App() {
  return <h1>Bienvenue {name} !</h1>;
}
```

👉 Affiche :

```
Bienvenue Katherine Johnson !
```

***

### ⚠️ Remarque

* **React** et **JSX** sont **deux choses différentes**.
  * JSX = juste une syntaxe (transformée en `React.createElement`).
  * React = la bibliothèque qui utilise ces éléments pour construire l’UI.
* On les utilise presque toujours ensemble, mais techniquement, tu peux utiliser React **sans JSX** (en écrivant uniquement du JS brut).

***

### 🧠 TL;DR

1. Avant → HTML, CSS, JS séparés → logique et balisage découplés.
2. Aujourd’hui → le JS pilote le contenu → React regroupe logique + balisage.
3. JSX = syntaxe qui permet d’écrire du balisage **dans du JS**.
4. Résultat → composants autonomes, lisibles, réutilisables.

## 📝 Convertir du HTML en JSX

### 1. Le HTML de départ

Tu pars d’un code HTML valide :

```html
<h1>Liste de tâches de Hedy Lamarr</h1>
<img
  src="https://i.imgur.com/yXOvdOSs.jpg"
  alt="Hedy Lamarr"
  class="photo"
/>
<ul>
  <li>Inventer de nouveaux feux de circulation</li>
  <li>Répéter une scène de film</li>
  <li>Améliorer les techniques de spectrographie</li>
</ul>
```

Et tu veux l’utiliser dans un composant React :

```jsx
export default function TodoList() {
  return (
    // ???
  )
}
```

***

### 2. Pourquoi ça casse ?

Si tu copies-colles ton HTML tel quel, React te renvoie une erreur comme :

```
Adjacent JSX elements must be wrapped in an enclosing tag.
```

👉 Traduction :\
En JSX, **tout doit être dans un seul élément parent** (pas plusieurs balises au même niveau).\
De plus :

* `class` n’existe pas → il faut écrire `className`
* Les balises doivent être bien **fermées** (`<img />`, `<li></li>`)

***

### 3. La version corrigée en JSX

Voici ton HTML converti correctement en JSX :

```jsx
export default function TodoList() {
  return (
    <div>
      <h1>Liste de tâches de Hedy Lamarr</h1>
      <img
        src="https://i.imgur.com/yXOvdOSs.jpg"
        alt="Hedy Lamarr"
        className="photo"
      />
      <ul>
        <li>Inventer de nouveaux feux de circulation</li>
        <li>Répéter une scène de film</li>
        <li>Améliorer les techniques de spectrographie</li>
      </ul>
    </div>
  );
}
```

***

### 4. Règles importantes à retenir 🔑

Quand tu transformes du HTML → JSX :

1. **Un seul parent**
   * Tous les éléments doivent être englobés (`<div>` ou `<>...</>`).
   * ❌ Erreur : `<h1>...</h1><p>...</p>`
   * ✅ Correct : `<div><h1>...</h1><p>...</p></div>`
2. **`class` → `className`**
   * Car `class` est un mot réservé en JavaScript.
3. **Bien fermer toutes les balises**
   * En HTML : `<img>` ou `<li>` peuvent être laissés ouverts.
   * En JSX : il faut **les fermer** : `<img />`, `<li>…</li>`.
4. **Respecter la casse des props**
   * En HTML : `onclick="..."`
   * En JSX : `onClick={...}` (camelCase).

***

### 5. Petite astuce pratique 💡

* Quand tu copies du HTML en JSX, les erreurs affichées par React dans la console ou l’écran t’aident souvent à corriger rapidement.
* Exemple : il te dit directement “`class` n’existe pas → veux-tu utiliser `className` ?”

***

### 🧠 TL;DR

1. JSX ressemble à HTML mais il est **plus strict**.
2. **Tout doit être englobé** dans un seul parent.
3. `class` → `className`.
4. Balises toujours **fermées**.
5. Les erreurs React sont **tes amies** : elles indiquent quoi corriger.

## 📜 Les règles de JSX

### 🔹 Règle 1 : Un seul élément racine

Dans React, **chaque composant doit retourner un seul élément racine**.\
👉 Ça veut dire que si tu veux afficher plusieurs éléments, tu dois les **enrober** dans une balise parent.

***

#### 1️⃣ Exemple avec une `<div>`

```jsx
export default function TodoList() {
  return (
    <div>
      <h1>Liste de tâches de Hedy Lamarr</h1>
      <img
        src="https://i.imgur.com/yXOvdOSs.jpg"
        alt="Hedy Lamarr"
        className="photo"
      />
      <ul>
        <li>Inventer de nouveaux feux de circulation</li>
        <li>Répéter une scène de film</li>
        <li>Améliorer les techniques de spectrographie</li>
      </ul>
    </div>
  );
}
```

👉 Ici, tous les éléments (`<h1>`, `<img>`, `<ul>`) sont contenus **dans une seule `<div>`**.\
C’est correct ✅.

***

#### 2️⃣ Exemple avec un Fragment `<> </>`

Si tu ne veux pas ajouter de balise inutile (par ex. une `<div>` qui ne sert à rien), tu peux utiliser un **Fragment** :

```jsx
export default function TodoList() {
  return (
    <>
      <h1>Liste de tâches de Hedy Lamarr</h1>
      <img
        src="https://i.imgur.com/yXOvdOSs.jpg"
        alt="Hedy Lamarr"
        className="photo"
      />
      <ul>
        <li>Inventer de nouveaux feux de circulation</li>
        <li>Répéter une scène de film</li>
        <li>Améliorer les techniques de spectrographie</li>
      </ul>
    </>
  );
}
```

👉 Résultat dans le navigateur : il n’y aura pas de `<div>` en trop, seulement le `<h1>`, `<img>` et `<ul>`.

***

### 🧐 En détail : pourquoi cette règle ?

JSX **n’est pas du HTML** → c’est du **JavaScript déguisé**.

Quand tu écris ça :

```jsx
return (
  <h1>Hello</h1>
  <p>World</p>
);
```

React essaie de le transformer en appels JavaScript (`React.createElement(...)`).\
Mais une fonction **ne peut pas retourner deux valeurs en même temps**, comme elle ne peut pas retourner deux objets distincts.

👉 Solution : les regrouper dans :

* **un seul conteneur** (`<div>`), ou
* **un Fragment** (`<>...</>`).

***

### 🧠 TL;DR

1. Chaque composant React doit retourner **un seul élément racine**.
2. Pour regrouper plusieurs éléments :
   * Utilise une `<div>` (si structure utile dans le DOM).
   * Utilise un **Fragment** `<>...</>` (si tu veux grouper sans polluer le DOM).
3. Raison : sous le capot, JSX → objets JavaScript. Et une fonction ne peut pas renvoyer deux objets sans les envelopper.

### 🔹 2. Fermez toutes les balises

En HTML classique :

* Certaines balises peuvent rester **ouvertes** → `<img>`, `<li>`, `<br>`…
* D’autres peuvent être mal fermées et quand même acceptées par le navigateur.

👉 Mais en **JSX**, pas de tolérance : **toutes les balises doivent être fermées** ✅.

#### Exemple ❌ incorrect en JSX :

```jsx
<img src="..." alt="..." class="photo">
<li>Inventer de nouveaux feux de circulation
```

#### Exemple ✅ correct :

```jsx
<>
  <img
    src="https://i.imgur.com/yXOvdOSs.jpg"
    alt="Hedy Lamarr"
    className="photo"
  />
  <ul>
    <li>Inventer de nouveaux feux de circulation</li>
    <li>Répéter une scène de film</li>
    <li>Améliorer les techniques de spectrographie</li>
  </ul>
</>
```

⚡ Retenir :

* `<img />` et `<input />` doivent être **auto-fermées**.
* `<li>…</li>` et `<div>…</div>` doivent avoir une balise ouvrante **et** fermante.

***

### 🔹 3. Utilisez le **camelCase** pour les attributs

JSX = JavaScript → et en JavaScript, les noms de variables **ne peuvent pas contenir de tirets** et certains mots sont réservés (`class`, `for`…).

👉 Donc :

* `class` → `className`
* `for` → `htmlFor`
* `stroke-width` → `strokeWidth`

#### Exemple en JSX :

```jsx
<img
  src="https://i.imgur.com/yXOvdOSs.jpg"
  alt="Hedy Lamarr"
  className="photo"
/>
```

⚠️ **Piège historique** :

* Les attributs `aria-*` et `data-*` restent **écrits comme en HTML**, avec tirets.\
  Exemple :

```jsx
<button aria-label="Fermer" data-id="42">X</button>
```

***

### 🔹 4. Astuce pratique : utiliser un convertisseur JSX

Quand tu veux convertir rapidement un gros bloc HTML ou SVG → JSX, utilise un **convertisseur automatique** (ex. celui de Babel, ou des outils en ligne).\
👉 Mais garde en tête ces règles, car tu auras toujours besoin de **corriger à la main** certaines choses.

***

### ✅ Résultat final : TodoList en JSX

```jsx
export default function TodoList() {
  return (
    <>
      <h1>Liste de tâches de Hedy Lamarr</h1>
      <img
        src="https://i.imgur.com/yXOvdOSs.jpg"
        alt="Hedy Lamarr"
        className="photo"
      />
      <ul>
        <li>Inventer de nouveaux feux de circulation</li>
        <li>Répéter une scène de film</li>
        <li>Améliorer les techniques de spectrographie</li>
      </ul>
    </>
  );
}
```

👉 Ici, on respecte :

1. **Un seul parent** → fragment `<>...</>`.
2. **Toutes les balises fermées** → `<img />`, `<li>...</li>`.
3. **Attributs en camelCase** → `className`.

***

### 🧠 TL;DR

* JSX est plus strict que HTML : **pas d’oubli de fermeture de balises**.
* Attributs → **camelCase**, avec exceptions pour `aria-*` et `data-*`.
* Utilise des outils pour convertir, mais comprends les règles pour corriger toi-même.
