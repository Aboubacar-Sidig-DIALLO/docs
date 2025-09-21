---
description: >-
  Bienvenue dans la documentation React ! 🎉 Ici, tu vas découvrir les 80% des
  notions essentielles que tu utiliseras tous les jours en développant avec
  React.
---

# 🚀 Démarrage rapide

### ✅ Ce que tu vas apprendre

* Comment créer et imbriquer des **composants**
* Comment ajouter du **balisage (markup)** et des **styles**
* Comment **afficher des données**
* Comment gérer des **conditions** et afficher des **listes**
* Comment **réagir aux événements** et mettre à jour l’interface
* Comment **partager des données entre composants**

### 🧩 Créer et imbriquer des composants

Une application React est constituée de **composants**.\
👉 Un composant, c’est tout simplement un petit morceau d’interface utilisateur (**UI**) qui a sa propre apparence et son propre comportement(logique).

Un composant peut être :

* aussi petit qu’un **bouton**,
* ou aussi grand qu’une **page entière**.

En React, un composant est une **fonction JavaScript** qui **retourne du balisage** (souvent du JSX, on en parlera juste après).

Exemple :

```jsx
function MyButton() {
  return (
    <button>Je suis un bouton</button>
  );
}
```

Super ! Maintenant que tu as déclaré `MyButton`, tu peux l’utiliser à l’intérieur d’un autre composant :

```jsx
export default function MyApp() {
  return (
    <div>
      <h1>Bienvenue dans mon appli</h1>
      <MyButton />
    </div>
  );
}
```

✨ Petit détail important : remarque que `<MyButton />` commence par une **majuscule**.

* Les **composants React** doivent toujours commencer par une **majuscule**.
* Les **balises HTML** (comme `<div>`, `<button>`) commencent, elles, par une **minuscule**.

C’est ainsi que React fait la différence entre une **balise HTML classique** et un **composant personnalisé**.

***

### 📝 Écrire du balisage avec JSX

La syntaxe que tu viens de voir s’appelle **JSX**.

👉 JSX ressemble beaucoup à du HTML, mais avec quelques différences :

* Tu dois **toujours fermer tes balises**, même celles qui sont vides (`<br />`, `<img />`).
* Un composant ne peut **retourner qu’un seul élément parent**. Si tu veux plusieurs balises côte à côte, il faut les envelopper dans un parent (souvent un `<div>` ou un `<>...</>` qu’on appelle **Fragment**).

Exemple :

```jsx
function AboutPage() {
  return (
    <>
      <h1>À propos</h1>
      <p>Bien le bonjour.<br />Comment ça va ?</p>
    </>
  );
}
```

💡 Astuce : si tu veux convertir rapidement du HTML en JSX, il existe plein de convertisseurs en ligne qui te faciliteront la tâche.

***

### 🎨 Ajouter des styles

En React, pour appliquer une classe CSS, on n’utilise pas `class` comme en HTML, mais **`className`** :

```jsx
<img className="avatar" />
```

Ensuite, tu écris tes règles CSS dans un fichier séparé :

```css
/* styles.css */
.avatar {
  border-radius: 50%;
}
```

👉 Tu peux importer ce fichier dans ton projet selon ton setup (par exemple avec une balise `<link>` classique ou via ton framework/bundler).

***

### 📊 Afficher des données

Avec JSX, tu peux **mélanger du JavaScript et du balisage**.\
Tu utilises des **accolades `{}`** pour « ressortir » du JSX et injecter des variables ou expressions JavaScript et de l’afficher à l’utilisateur.

Exemple simple :

```jsx
return (
  <h1>{user.name}</h1>
);
```

Tu peux aussi utiliser les accolades dans les **attributs** JSX.\
⚠️ Attention : contrairement aux chaînes de caractères (qui utilisent `"..."`), il faut mettre des accolades pour insérer une **variable JavaScript**.

```jsx
return (
  <img
    className="avatar"
    src={user.imageUrl}
  />
);
```

Tu peux même écrire des expressions plus complexes :

```jsx
<img
  src={user.imageUrl}
  alt={'Photo de ' + user.name}
  style={{
    width: user.imageSize,
    height: user.imageSize
  }}
/>
```

Ici, `style={{}}` n’est pas une syntaxe magique : c’est simplement un **objet JavaScript** utilisé à l’intérieur de `style={ }`.\
Cela permet de générer des styles dynamiques en fonction de variables.

***

### 🔀 Affichage conditionnel

En React, pas de syntaxe spéciale pour les conditions. Tu utilises simplement **JavaScript** habituel.

Avec un `if` classique :

```jsx
if (isLoggedIn) {
  return <AdminPanel />;
} else {
  return <LoginForm />;
}
```

Version plus compacte avec un **opérateur ternaire** :

```jsx
<div>
  {isLoggedIn ? <AdminPanel /> : <LoginForm />}
</div>
```

Et si tu n’as pas besoin du `else`, tu peux utiliser l’opérateur logique `&&`&#x20;

```jsx
<div>
  {isLoggedIn && <AdminPanel />}
</div>
```

Ces techniques fonctionnent aussi dans les **attributs JSX**.\
👉 Si certaines syntaxes JS te paraissent complexes, commence simplement avec `if...else`.

***

### 📝 Afficher des listes

Pour afficher une liste, on utilise principalement la méthode **`map()`** de JavaScript, et parfois **`for.`**&#x20;

Exemple :

<pre class="language-jsx"><code class="lang-jsx">const products = [
  { title: 'Chou', id: 1 },
  { title: 'Ail', id: 2 },
  { title: 'Pomme', id: 3 },
];

<strong>Ici utilise la méthode map() pour transformer un tableau de produits en tableau d’éléments &#x3C;li> :
</strong>
const listItems = products.map(product =>
  &#x3C;li key={product.id}>{product.title}&#x3C;/li>
);

return &#x3C;ul>{listItems}&#x3C;/ul>;
</code></pre>

👉 N’oublie pas la prop `key` : elle aide React à identifier chaque élément de manière unique.

Autrement ,  chaque élément de la liste doit avoir une **clé unique** (`key`).\
Généralement, on utilise un **id provenant de la base de données**.\
React s’en sert pour comprendre quels éléments ont été ajoutés, supprimés ou réorganisés.

Exemple avec du style conditionnel :

```jsx
const products = [
  { title: 'Chou', isFruit: false, id: 1 },
  { title: 'Ail', isFruit: false, id: 2 },
  { title: 'Pomme', isFruit: true, id: 3 },
];

export default function ShoppingList() {
  const listItems = products.map(product =>
    <li
      key={product.id}
      style={{
        color: product.isFruit ? 'magenta' : 'darkgreen'
      }}
    >
      {product.title}
    </li>
  );

  return (
    <ul>{listItems}</ul>
  );
}
```

***

### 🖱️ Réagir à des événements

Tu peux gérer des événements en déclarant des fonctions **gestionnaires d’événements** (_event handlers_) dans tes composants.&#x20;

```jsx
function MyButton() {
  function handleClick() {
    alert('Vous avez cliqué !');
  }

  return (
    <button onClick={handleClick}>
      Cliquez ici
    </button>
  );
}
```

Remarquez que `onClick={handleClick}` n’a pas de parenthèses à la fin !\
⚠️ Attention : on écrit `onClick={handleClick}` (sans parenthèses), sinon la fonction serait exécutée immédiatement au lieu d’attendre le clic.

Exemple :

```jsx
function MyButton() {
  function handleClick() {
    alert('Tu as cliqué !');
  }

  return (
    <button onClick={handleClick}>
      Clique-moi
    </button>
  );
}
```

👉 Remarque que `onClick={handleClick}` **n’appelle pas la fonction** (pas de `()`).\
Tu la passes simplement à React, qui l’exécutera lorsque l’utilisateur cliquera.

***

### 🔄 Mettre à jour l’affichage avec l’état

Parfois, ton composant doit **se souvenir de quelque chose** (par exemple, combien de fois un bouton a été cliqué).\
C’est ce qu’on appelle l’**état**.

Pour cela, React propose un **mécanisme d’état** grâce au Hook `useState`.

***

#### 1. Importer `useState`

Avant d’utiliser l’état, on doit l’importer depuis React :

```jsx
import { useState } from 'react';
```

***

#### 2. Déclarer une variable d’état

À l’intérieur de ton composant, tu déclares une **variable d’état** et sa fonction de mise à jour :

```jsx
function MyButton() {
  const [count, setCount] = useState(0);
  // ...
}
```

* `count` → la valeur actuelle de l’état
* `setCount` → la fonction qui permet de **mettre à jour** cet état
* `useState(0)` → initialise `count` à **0** (la valeur de départ)

👉 Tu peux donner n’importe quel nom à ces deux variables, mais la convention est :\
`[quelqueChose, setQuelqueChose]`.

***

#### 3. Mettre à jour l’état

Lorsqu’on clique sur le bouton, on veut **incrémenter** le compteur.\
On écrit donc une fonction `handleClick` :

```jsx
function MyButton() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      Cliqué {count} fois
    </button>
  );
}
```

***

#### 4. Comment ça fonctionne ?

* La première fois que le composant est affiché, `count` vaut **0**.
* Quand tu cliques, `setCount(count + 1)` met à jour l’état.
* React **réexécute la fonction du composant** avec la nouvelle valeur de `count`.
* Le bouton affiche alors la valeur mise à jour (`1`, puis `2`, puis `3`…).

C’est ce qu’on appelle **le re-rendering** : React relance la fonction du composant pour mettre à jour l’UI en fonction du nouvel état.

***

#### 5. Chaque composant a son propre état

Si tu utilises ton composant plusieurs fois, **chacun garde son état indépendamment des autres**.

Exemple :

```jsx
import { useState } from 'react';

export default function MyApp() {
  return (
    <div>
      <h1>Des compteurs indépendants</h1>
      <MyButton />
      <MyButton />
    </div>
  );
}

function MyButton() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      Cliqué {count} fois
    </button>
  );
}
```

👉 Résultat : chaque bouton **se souvient de son propre compteur**.

* Cliquer sur le premier n’a aucun effet sur le second.
* Chaque instance du composant possède **sa propre mémoire interne**.

***

✨ **En résumé** :

* `useState` permet à un composant de **mémoriser une valeur dans le temps**.
* Chaque composant (ou chaque copie d’un composant) possède **son propre état isolé**.
* Quand tu appelles la fonction de mise à jour (`setCount`), React **réexécute le composant** avec la nouvelle valeur et **met à jour l’UI** automatiquement.

***

### 🪝 Les Hooks

Un **Hook** est une fonction spéciale de React (leur nom commence toujours par `use`).

* `useState` permet de gérer l’état
* Il existe d’autres Hooks intégrés (comme `useEffect`, `useContext`, etc.)
* Tu peux même créer tes propres Hooks pour factoriser du code réutilisable en combinant ceux existants.

⚠️ Règle importante : les Hooks doivent toujours être appelés **au début d’un composant ou d'autres hooks**, jamais dans une condition ou une boucle.

Si vous voulez utiliser `useState` dans une condition ou une boucle, extrayez un composant dédié au besoin et mettez le Hook à l’intérieur.

***

### 🤝 Partager des données entre composants

Dans l’exemple précédent, chaque `MyButton` avait son propre `count` indépendant, et lorsqu’un bouton était cliqué, seul le `count` de ce bouton changeait :

<figure><img src="../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

Toutefois, vous aurez régulièrement besoin que vos composants _partagent des données et se mettent à jour de façon synchronisée_.

Afin que les deux composants `MyButton` affichent le même `count` et se mettent à jour ensemble, vous allez devoir déplacer l’état depuis les boutons individuels « vers le haut », vers le plus proche composant qui les contienne tous (le parent communt), puis on le redescend sous forme de **props**.

Dans cet exemple, il s’agit de `MyApp` :&#x20;

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

À présent quand vous cliquez l’un ou l’autre bouton, le `count` de `MyApp` change, ce qui altère les deux compteurs dans `MyButton`. Voici comment exprimer la même chose sous forme de code.

#### 1. Remonter l’état

La règle générale :

* Si plusieurs composants doivent partager la même information, il faut **déplacer (remonter) cet état dans leur parent commun**.

Ici, le parent est `MyApp`.\
On déplace donc l’état de `MyButton` vers `MyApp` :

```jsx
export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>Des compteurs synchronisés</h1>
      <MyButton />
      <MyButton />
    </div>
  );
}

function MyButton() {
  // ... on a déplacé le state ici vers MyApp ...
}
```

***

#### 2. Transmettre l’état et les événements via des props

Maintenant que `MyApp` possède l’état (`count`) et la fonction qui l’augmente (`handleClick`),\
on peut les **transmettre aux enfants** (`MyButton`) sous forme de **props** :

```jsx
export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>Des compteurs synchronisés</h1>
      <MyButton count={count} onClick={handleClick} />
      <MyButton count={count} onClick={handleClick} />
    </div>
  );
}
```

👉 Les informations qu’un parent transmet à un enfant dans React s’appellent des **props** (_properties_).\
Ici, chaque bouton reçoit :

* `count` → la valeur de l’état,
* `onClick` → la fonction à appeler quand on clique.

***

#### 3. Lire les props dans l’enfant

Maintenant, on adapte `MyButton` pour qu’il **utilise les props passées par le parent** :

```jsx
function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      Cliqué {count} fois
    </button>
  );
}
```

* `{ count, onClick }` → c’est une **déstructuration** des props.
* Chaque bouton **n’a plus son propre état** : il utilise celui fourni par `MyApp`.

***

#### 4. Résultat

* Quand tu cliques sur **n’importe quel bouton**, c’est la fonction `handleClick` de `MyApp` qui s’exécute.
* Elle appelle `setCount(count + 1)` → ce qui met à jour **l’état dans le parent**.
* React redessine `MyApp`, et transmet la **nouvelle valeur de `count`** aux deux boutons.
* Les deux boutons affichent donc le même compteur synchronisé.

C’est ce mécanisme qu’on appelle **remonter l’état** (_lifting state up_).\
👉 En résumé : en déplaçant l’état vers un composant parent, on permet à plusieurs enfants de le **partager**.

***

✨ **Idée clé à retenir** :

* Chaque composant a normalement son propre état.
* Mais si plusieurs composants doivent “voir” ou “modifier” la même donnée, il faut **placer cet état au niveau du parent commun** et le **partager via les props**.

***

### 🎉 Et maintenant ?

Bravo 👏 Tu connais les **bases de React** !\
Tu sais :

* Créer des composants
* Ajouter du JSX et des styles
* Afficher des données
* Gérer conditions et listes
* Réagir aux événements
* Utiliser l’état et les Hooks
* Partager des données entre composants
