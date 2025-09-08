---
description: >-
  Bienvenue dans la documentation React ! 🎉 Ici, tu vas découvrir les 80% des
  notions essentielles que tu utiliseras tous les jours en développant avec
  React.
---

# 🚀 Démarrage rapide

### 🧩 Créer et imbriquer des composants

Une application React est constituée de **composants**.\
👉 Un composant, c’est tout simplement un petit morceau d’interface utilisateur (**UI**) qui a sa propre apparence et son propre comportement.

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
Tu utilises des **accolades `{}`** pour « ressortir » du JSX et injecter des variables ou expressions JavaScript.

Exemple simple :

```jsx
return (
  <h1>{user.name}</h1>
);
```

Dans les attributs aussi :

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

Ici, `style={{}}` n’est pas une syntaxe magique : c’est juste un **objet JavaScript** passé en prop.

***

### 🔀 Affichage conditionnel

En React, pas de syntaxe spéciale pour les conditions. Tu utilises simplement **JavaScript**.

Avec un `if` classique :

```jsx
if (isLoggedIn) {
  return <AdminPanel />;
} else {
  return <LoginForm />;
}
```

Avec un **opérateur ternaire** :

```jsx
<div>
  {isLoggedIn ? <AdminPanel /> : <LoginForm />}
</div>
```

Ou avec `&&` si tu n’as pas besoin du `else` :

```jsx
<div>
  {isLoggedIn && <AdminPanel />}
</div>
```

***

### 📝 Afficher des listes

Pour afficher une liste, on utilise principalement la méthode **`map()`** de JavaScript.

Exemple :

```jsx
const products = [
  { title: 'Chou', id: 1 },
  { title: 'Ail', id: 2 },
  { title: 'Pomme', id: 3 },
];

const listItems = products.map(product =>
  <li key={product.id}>{product.title}</li>
);

return <ul>{listItems}</ul>;
```

👉 N’oublie pas la prop `key` : elle aide React à identifier chaque élément de manière unique.

***

### 🖱️ Réagir à des événements

Tu peux gérer des événements avec des gestionnaires comme en JavaScript classique.

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

⚠️ Attention : on écrit `onClick={handleClick}` (sans parenthèses), sinon la fonction serait exécutée immédiatement au lieu d’attendre le clic.

***

### 🔄 Mettre à jour l’affichage avec l’état

Parfois, ton composant doit **se souvenir de quelque chose** (par exemple, combien de fois un bouton a été cliqué).\
C’est ce qu’on appelle l’**état**.

En React, on utilise le **Hook `useState`** pour gérer ça :

```jsx
import { useState } from 'react';

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

Chaque fois que tu cliques :

1. React appelle `setCount`
2. Le composant est **rendu à nouveau**
3. `count` a la nouvelle valeur

Si tu affiches plusieurs `MyButton`, chacun aura son **état indépendant**.

***

### 🪝 Les Hooks

Un **Hook** est une fonction spéciale de React (leur nom commence toujours par `use`).

* `useState` permet de gérer l’état
* Il existe d’autres Hooks intégrés (comme `useEffect`, `useContext`, etc.)
* Tu peux même créer tes propres Hooks pour factoriser du code réutilisable

⚠️ Règle importante : les Hooks doivent toujours être appelés **au début d’un composant**, jamais dans une condition ou une boucle.

***

### 🤝 Partager des données entre composants

Dans l’exemple précédent, chaque `MyButton` avait son propre compteur. Mais parfois, tu veux que **tous les boutons partagent la même donnée**.

👉 Pour ça, on fait « remonter l’état » vers un composant parent, puis on le redescend sous forme de **props**.

Exemple :

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

function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      Cliqué {count} fois
    </button>
  );
}
```

Maintenant, les deux boutons affichent et mettent à jour le **même état**. 🎉

C’est ce qu’on appelle **lifting state up** (remonter l’état).

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
