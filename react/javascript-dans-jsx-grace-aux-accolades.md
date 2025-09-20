# ✨ JavaScript dans JSX grâce aux accolades

### 1. Chaînes de caractères avec guillemets

Si tu veux mettre une valeur **fixe**, tu l’écris comme en HTML classique → avec des guillemets (`" "` ou `' '`).

Exemple :

```jsx
export default function Avatar() {
  return (
    <img
      className="avatar"
      src="https://i.imgur.com/7vQD0fPs.jpg"
      alt="Gregorio Y. Zara"
    />
  );
}
```

👉 Ici :

* `className="avatar"` → fixe, valeur textuelle
* `src="https://..."` et `alt="Gregorio Y. Zara"` → fixes aussi

Résultat : ça marche, mais c’est **statique**.

***

### 2. Variables dynamiques avec les `{ }`

Si tu veux rendre ça **dynamique**, tu utilises des **accolades `{ }`**.

Exemple :

```jsx
export default function Avatar() {
  const avatar = "https://i.imgur.com/7vQD0fPs.jpg";
  const description = "Gregorio Y. Zara";

  return (
    <img
      className="avatar"
      src={avatar}
      alt={description}
    />
  );
}
```

👉 Ici :

* `src={avatar}` → lit la **variable JS** `avatar`
* `alt={description}` → lit la **variable JS** `description`

⚡ Différence :

* `"..."` → valeur **littérale** (fixe)
* `{ ... }` → valeur **calculée** ou **variable JS**

***

### 3. Expressions JavaScript dans JSX

Entre `{ }`, tu peux mettre **toute expression JavaScript** :

* Variables
* Fonctions
* Calculs
* Objets

Exemples :

#### Variables :

```jsx
const name = "Hedy Lamarr";
<h1>{name}</h1>
```

👉 Affiche : **Hedy Lamarr**

***

#### Fonctions :

```jsx
function formatUser(user) {
  return user.firstName + " " + user.lastName;
}

const user = { firstName: "Katherine", lastName: "Johnson" };

<h1>{formatUser(user)}</h1>
```

👉 Affiche : **Katherine Johnson**

***

#### Calculs :

```jsx
const tasks = 3;
<p>Nombre de tâches : {tasks + 1}</p>
```

👉 Affiche : **Nombre de tâches : 4**

***

#### Objets :

```jsx
const user = { name: "Marie Curie", age: 66 };

<p>{user.name} a {user.age} ans</p>
```

👉 Affiche : **Marie Curie a 66 ans**

***

### 4. Règles importantes 🔑

* Les accolades **ouvrent une fenêtre vers JavaScript** dans ton JSX.
* Elles ne servent **pas pour tout** :
  * `className="avatar"` → guillemets → valeur fixe
  * `src={variable}` → accolades → valeur dynamique
* Tu peux mettre **n’importe quelle expression JS**, mais pas des instructions complètes (`if`, `for` …) → uniquement ce qui renvoie une valeur.

***

### 🧠 TL;DR

1. `"..."` → valeur fixe (chaîne de caractères).
2. `{ ... }` → insère une variable, une fonction ou un calcul JS.
3. Les accolades permettent de **rendre ton JSX dynamique**.
4. Utilise guillemets pour le statique, accolades pour le dynamique.

## ✨ Les accolades : une fenêtre vers le monde JavaScript

### 1. Insérer une variable

Tu peux insérer directement une **variable JavaScript** dans ton JSX grâce aux accolades `{ }`.

```jsx
export default function TodoList() {
  const name = 'Gregorio Y. Zara';
  return (
    <h1>Liste des tâches de {name}</h1>
  );
}
```

👉 Ici, `{name}` est remplacé par la valeur de la variable `name`.\
Si tu changes `name = 'Hedy Lamarr'`, ton UI s’actualise automatiquement ✅.

***

### 2. Utiliser des fonctions

Tu peux aussi appeler une fonction **directement dans ton JSX** :

```jsx
const today = new Date();

function formatDate(date) {
  return new Intl.DateTimeFormat(
    'fr-FR',
    { weekday: 'long' }
  ).format(date);
}

export default function TodoList() {
  return (
    <h1>Liste de tâches pour {formatDate(today)}</h1>
  );
}
```

👉 Ici, `{formatDate(today)}` affiche le jour de la semaine (ex. _lundi_).

***

### 3. Où peut-on mettre des accolades ?

Il y a **deux endroits possibles** :

#### 🔹 Dans le texte (contenu d’une balise)

```jsx
<h1>Liste de tâches de {name}</h1>
```

⚠️ Attention : tu ne peux pas faire `<{tag}> ... </{tag}>`. Les balises doivent être écrites en dur ou être un composant.

***

#### 🔹 Dans les attributs (après un `=`)

```jsx
<img src={avatar} alt={description} />
```

👉 Ici, `src={avatar}` lit la valeur d’une variable JS, tandis que `src="{avatar}"` passerait littéralement la chaîne de texte `"{avatar}"`.

***

### 4. Les « doubles accolades » : objets en JSX

En JavaScript, un **objet littéral** est écrit entre accolades `{ }`.\
Comme JSX utilise déjà `{ }` pour insérer du JS, ça fait une **double accolade** `{{ ... }}` quand tu passes un objet directement.

Exemple avec des styles en ligne :

```jsx
export default function TodoList() {
  return (
    <ul style={{
      backgroundColor: 'black',
      color: 'pink'
    }}>
      <li>Améliorer le visiophone</li>
      <li>Préparer les cours d’aéronautique</li>
      <li>Travailler sur un moteur à alcool</li>
    </ul>
  );
}
```

👉 Ici :

* Les **premières accolades** → disent « je mets du JavaScript ».
*   Les **secondes accolades** → définissent l’objet JS :

    ```js
    { backgroundColor: 'black', color: 'pink' }
    ```

C’est exactement comme écrire :

```jsx
<ul style={
  {
    backgroundColor: 'black',
    color: 'pink'
  }
}>
```

***

### 5. Piège : propriétés `style` en camelCase

En HTML classique :

```html
<ul style="background-color: black">
```

En JSX, ça devient :

```jsx
<ul style={{ backgroundColor: 'black' }}>
```

👉 Toutes les propriétés CSS doivent être en **camelCase** (`backgroundColor`, `fontSize`, etc.).

***

### 🧠 TL;DR

* `{ ... }` → ouvre une fenêtre vers JavaScript dans ton JSX.
* Tu peux y mettre : variables, fonctions, calculs, objets.
* Deux usages principaux :
  * **Dans le texte** : `<h1>{name}</h1>`
  * **Dans les attributs** : `<img src={avatar} />`
* Les doubles accolades `{{ ... }}` = juste un **objet JS passé dans du JSX**.
* Les styles en ligne → propriétés CSS en **camelCase**.

## ✨ Les accolades : une fenêtre vers le monde JavaScript

### 1. Insérer une variable

Tu peux insérer une variable JavaScript directement dans ton JSX :

```jsx
export default function TodoList() {
  const name = 'Gregorio Y. Zara';
  return (
    <h1>Liste des tâches de {name}</h1>
  );
}
```

👉 Ici, `{name}` sera remplacé par la valeur de la variable `name`.

* Si `name = 'Gregorio Y. Zara'` → affichage : _Liste des tâches de Gregorio Y. Zara_.
* Si `name = 'Hedy Lamarr'` → affichage : _Liste des tâches de Hedy Lamarr_.

💡 C’est dynamique, pas besoin de toucher au HTML !

***

### 2. Utiliser une fonction

Les accolades acceptent aussi des **appels de fonctions** :

```jsx
const today = new Date();

function formatDate(date) {
  return new Intl.DateTimeFormat(
    'fr-FR',
    { weekday: 'long' }
  ).format(date);
}

export default function TodoList() {
  return (
    <h1>Liste de tâches pour {formatDate(today)}</h1>
  );
}
```

👉 Ici, `formatDate(today)` inséré dans `{ }` affiche le **jour actuel** (ex. _lundi_).

***

### 3. Où peut-on utiliser des accolades ?

Tu ne peux utiliser `{ }` qu’à deux endroits précis :

#### 🔹 Comme texte dans une balise

```jsx
<h1>Liste de tâches de {name}</h1>
```

⚠️ Pas possible :

```jsx
<{tag}>Liste de tâches</{tag}>
```

→ Les noms de balises doivent être explicites (`h1`, `div`, ou un composant React).

***

#### 🔹 Comme valeur d’attribut

```jsx
<img src={avatar} alt={description} />
```

👉 Ici, `src={avatar}` lit la variable `avatar`.\
Mais `src="{avatar}"` passerait littéralement la chaîne de caractères `"{avatar}"`.

***

### 4. Les « doubles accolades » : objets en JSX

Un objet JS est écrit entre `{ }`.\
Comme JSX utilise déjà des `{ }` pour exécuter du JS → tu auras deux paires d’accolades `{{ ... }}` quand tu passes un objet.

Exemple avec des **styles en ligne** :

```jsx
export default function TodoList() {
  return (
    <ul style={{
      backgroundColor: 'black',
      color: 'pink'
    }}>
      <li>Améliorer le visiophone</li>
      <li>Préparer les cours d’aéronautique</li>
      <li>Travailler sur un moteur à alcool</li>
    </ul>
  );
}
```

👉 Ici :

* Les **premières accolades** → “je mets du JavaScript dans JSX”.
*   Les **secondes accolades** → “voici un objet JS” :

    ```js
    { backgroundColor: 'black', color: 'pink' }
    ```

On peut aussi l’écrire plus lisiblement :

```jsx
<ul style={
  {
    backgroundColor: 'black',
    color: 'pink'
  }
}>
```

***

### 5. ⚠️ Piège : propriétés `style` en camelCase

En HTML classique :

```html
<ul style="background-color: black">
```

En JSX, ça devient :

```jsx
<ul style={{ backgroundColor: 'black' }}>
```

👉 Toutes les propriétés CSS doivent être en **camelCase** :

* `background-color` → `backgroundColor`
* `font-size` → `fontSize`

***

### 🧠 TL;DR

* `{ ... }` = insérer du JavaScript dans JSX.
* Tu peux y mettre : **variables, fonctions, calculs, objets**.
* Deux usages :
  * Comme texte dans une balise : `<h1>{name}</h1>`.
  * Comme valeur d’attribut : `<img src={avatar} />`.
* Les `{{ ... }}` = **objet JS passé dans JSX** (ex. styles en ligne).
* Les propriétés `style` → **camelCase**.

## ✨ Amusons-nous avec les objets JavaScript et les accolades

### 1. Définir un objet en JavaScript

Ici on crée un objet `person` qui contient :

* une propriété `name` (chaîne de caractères),
* une propriété `theme` (un autre objet pour les styles).

```jsx
const person = {
  name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};
```

👉 `person` est donc une **structure de données réutilisable** :

* `person.name` → donne `"Gregorio Y. Zara"`.
* `person.theme` → donne `{ backgroundColor: 'black', color: 'pink' }`.

***

### 2. Utiliser l’objet dans un composant

On va utiliser cet objet dans le JSX grâce aux accolades `{ }` :

```jsx
export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>Liste des tâches de {person.name}</h1>
      <img
        className="avatar"
        src="https://i.imgur.com/7vQD0fPs.jpg"
        alt="Gregorio Y. Zara"
      />
      <ul>
        <li>Améliorer le visiophone</li>
        <li>Préparer les cours d’aéronautique</li>
        <li>Travailler sur un moteur à alcool</li>
      </ul>
    </div>
  );
}
```

👉 Ici :

* `style={person.theme}` applique directement l’objet `theme` aux styles.
* `{person.name}` insère la valeur de `name` dans le `<h1>`.

***

### 3. Résultat

Ton composant affiche :

* Un `<div>` stylisé avec un fond noir et du texte rose (grâce à `person.theme`).
* Un titre `Liste des tâches de Gregorio Y. Zara` (grâce à `person.name`).
* Une image d’avatar.
* Une liste de tâches.

💡 **Avantage** : si tu changes ton objet `person`, ton UI s’adapte immédiatement sans changer le JSX.

***

### 4. Pourquoi c’est puissant ?

JSX est volontairement **minimal en templating** :

* Pas besoin d’inventer une nouvelle syntaxe compliquée pour injecter des données.
* Tu restes **100 % dans du JavaScript** → tu organises tes données dans des objets, tableaux, fonctions… et tu les insères avec `{ }`.

C’est ce qui rend React **expressif et flexible** :

* Les données (ici l’objet `person`) vivent dans du JS.
* Le rendu (ici la todo list) est lié à ces données, mais sans complexité supplémentaire.

***

✅ **À retenir** :

* Tu peux stocker toutes tes infos dans des objets JavaScript.
* Avec `{ }`, tu peux les réutiliser dans ton JSX (texte, attributs, styles, etc.).
* JSX ne cherche pas à être un langage de template complexe → il te laisse utiliser la **puissance de JavaScript** directement.
