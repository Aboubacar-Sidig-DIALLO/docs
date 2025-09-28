# ✨ Passer des props à un composant

### 1. Qu’est-ce qu’une prop ?

👉 Les **props** (abréviation de _properties_) sont comme des **paramètres de fonction**, mais pour les composants React.\
Elles permettent à un **composant parent** de donner des informations à un **composant enfant**.

💡 Tu peux voir les props comme l’équivalent des **attributs HTML**, mais en beaucoup plus puissant :

* Elles peuvent contenir des **chaînes de caractères** (`"texte"`),
* des **nombres** (`100`),
* des **booléens** (`true/false`),
* des **objets** (`{clé: valeur}`),
* des **fonctions** (`()=>{}`),
* ou même du **JSX** (oui, tu peux passer un autre composant en prop).

***

### 2. Exemple avec une balise HTML

Quand tu écris ça :

```jsx
<img
  className="avatar"
  src="https://i.imgur.com/1bX5QH6.jpg"
  alt="Lin Lanying"
  width={100}
  height={100}
/>
```

👉 Ici, les props sont :

* `className="avatar"`
* `src="https://i.imgur.com/1bX5QH6.jpg"`
* `alt="Lin Lanying"`
* `width={100}`
* `height={100}`

Ces props sont **prédéfinies** par le standard HTML.

***

### 3. Exemple avec un composant React

Tu peux créer ton propre composant, et lui donner des props **personnalisées** :

```jsx
function Avatar(props) {
  return (
    <img
      className="avatar"
      src={props.src}
      alt={props.alt}
      width={props.size}
      height={props.size}
    />
  );
}

export default function Profile() {
  return (
    <Avatar
      src="https://i.imgur.com/1bX5QH6.jpg"
      alt="Lin Lanying"
      size={100}
    />
  );
}
```

👉 Ici :

* Le parent `<Profile />` passe **3 props** (`src`, `alt`, `size`) au composant enfant `<Avatar />`.
* Dans `Avatar`, tu les récupères grâce à l’objet `props`.

***

### 4. Props = paramètres de fonction

Un composant est **juste une fonction**.\
Donc au lieu d’écrire `props.src`, `props.alt`, etc., tu peux **déstructurer** directement les props comme des paramètres de fonction :

```jsx
function Avatar({ src, alt, size }) {
  return (
    <img
      className="avatar"
      src={src}
      alt={alt}
      width={size}
      height={size}
    />
  );
}
```

👉 Plus propre, plus lisible ✨.

***

### 5. Points importants

* Les props sont **en lecture seule** → un composant enfant ne peut pas modifier ses props (il peut juste les utiliser).
* Elles permettent de rendre un composant **réutilisable**.
* Tu peux donner des valeurs **par défaut** si une prop n’est pas fournie (on verra ça juste après).

***

✅ **À retenir** :

* Les props, ce sont les **infos passées par le parent** au composant enfant.
* C’est ce qui permet d’avoir des composants **dynamiques et configurables**.
* Tu les récupères dans ton composant via `props` ou avec la **déstructuration**

## ✨ Passer des props à un composant

### 1. Situation de départ

Ici, le composant parent **Profile** appelle le composant enfant **Avatar**, mais ne lui donne encore **aucune information** :

```jsx
export default function Profile() {
  return (
    <Avatar />
  );
}
```

Résultat 👉 **Avatar n’a aucun contexte** : il ne sait pas quoi afficher.

***

### 2. Étape 1 : passer des props au composant enfant

On ajoute deux props à Avatar :

* `person` → un **objet** qui contient `name` et `imageId`.
* `size` → un **nombre** pour définir la taille de l’image.

```jsx
export default function Profile() {
  return (
    <Avatar
      person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      size={100}
    />
  );
}
```

⚡ Remarque importante :

* Les **premières accolades** → indiquent « je mets du JavaScript dans du JSX ».
* Les **secondes accolades** après `person=` → définissent un **objet littéral** en JavaScript.

Donc `person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}` est tout simplement **un objet passé comme prop**.

***

### 3. Étape 2 : lire les props dans le composant enfant

Dans React, un composant est **une fonction**.\
Et comme toute fonction, il prend un argument : **`props`**.

👉 Ici, on utilise la **déstructuration** pour lire directement les propriétés qu’on veut :

```jsx
function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}
```

* `{ person, size }` = déstructuration de `props`.
* Tu pourrais écrire la version longue :

```jsx
function Avatar(props) {
  let person = props.person;
  let size = props.size;
  // ...
}
```

Mais la version déstructurée est **plus concise et plus lisible**.

***

### 4. Exemple complet

Avec plusieurs `<Avatar />` configurés différemment :

```jsx
import { getImageUrl } from './utils.js';

function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

export default function Profile() {
  return (
    <div>
      <Avatar
        size={100}
        person={{ name: 'Katsuko Saruhashi', imageId: 'YfeOqp2' }}
      />
      <Avatar
        size={80}
        person={{ name: 'Aklilu Lemma', imageId: 'OKS67lh' }}
      />
      <Avatar
        size={50}
        person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      />
    </div>
  );
}
```

👉 Ici, **Profile** envoie à chaque **Avatar** des props différentes (nom + imageId + taille).\
👉 Chaque **Avatar** reste indépendant : il lit seulement ses props et affiche ce qu’on lui donne.

***

### 5. Comment penser les props ?

* Les **props** sont comme des **molettes de réglage** :
  * Tu modifies la prop dans le parent, l’enfant s’adapte.
* Elles jouent le **même rôle que les arguments des fonctions**.
* Elles sont **en lecture seule** → l’enfant ne peut pas les modifier.

***

### 6. ⚠️ Piège courant

Quand tu déstructures les props dans la signature :

```jsx
function Avatar({ person, size }) { ... }
```

⚡ Il faut bien mettre les **accolades `{ }` entre parenthèses `( )`**.\
Sinon, React croira que tu définis juste un argument unique appelé `props`.

***

✅ **À retenir**

* Un composant parent → passe des **props** à son enfant.
* Les props = **seul argument d’un composant fonction React**.
* Tu peux soit utiliser `props.nom`, soit déstructurer `{ nom }`.
* Les props rendent un composant **flexible, réutilisable et configurable**.

## ✨ Spécifier une valeur par défaut pour une prop

### 1. Définir une valeur par défaut

Tu peux donner une valeur par défaut à une prop directement dans la **déstructuration** :

```jsx
function Avatar({ person, size = 100 }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}
```

👉 Ici, si le parent utilise :

```jsx
<Avatar person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }} />
```

Alors `size` prendra automatiquement **100**, même si le parent n’a rien fourni.

⚡ Important :

* La valeur par défaut est utilisée si la prop est **absente** ou **undefined**.
* Mais si tu passes `null` ou `0`, la valeur par défaut **n’est pas utilisée** (parce que `null` et `0` sont considérés comme des valeurs valides).

***

## ✨ Transmettre des props avec la syntaxe de spread JSX

### 2. Cas classique : répétition

Souvent, tu écris des composants « relais » qui transmettent leurs props à un composant enfant.

Exemple sans spread :

```jsx
function Profile({ person, size, isSepia, thickBorder }) {
  return (
    <div className="card">
      <Avatar
        person={person}
        size={size}
        isSepia={isSepia}
        thickBorder={thickBorder}
      />
    </div>
  );
}
```

👉 Ça marche, mais c’est répétitif : tu listes toutes les props **une par une**.

***

### 3. Version concise avec spread

Tu peux utiliser la syntaxe **spread (`...`)** pour transmettre **toutes les props d’un coup** :

```jsx
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
```

👉 Ici :

* `...props` décompose l’objet `props` en attributs séparés.
* Si `Profile` reçoit `person`, `size`, `isSepia`, `thickBorder`, alors **Avatar reçoit exactement les mêmes props**.

***

### 4. ⚠️ Attention au spread

* Le spread rend le code **plus concis**, mais **moins explicite** : on ne sait pas au premier coup d’œil quelles props passent au composant enfant.
* Si tu l’utilises partout, c’est souvent le signe qu’il faut **repenser tes composants** (par ex. passer du JSX enfant plutôt que toutes les props).

👉 En résumé :

* Lisibilité > concision.
* Utilise `...props` quand tu as un composant « relais » qui n’ajoute rien par lui-même.

***

✅ **À retenir**

* Tu peux définir des **valeurs par défaut** pour éviter d’écrire des vérifications partout.
* La syntaxe de **spread JSX (`...props`)** permet de transmettre toutes les props d’un coup, mais à utiliser avec discernement.

## ✨ Passer du JSX comme enfant (prop `children`)

### 1. Le parallèle avec le HTML

En HTML natif, tu imbriques naturellement les balises :

```html
<div>
  <img />
</div>
```

👉 Le `<div>` agit comme un **conteneur** autour de `<img>`.

***

### 2. La même idée avec des composants React

Tu peux faire exactement pareil avec tes **propres composants** :

```jsx
<Card>
  <Avatar />
</Card>
```

👉 Ici :

* `<Card>` est un **composant parent**.
* `<Avatar />` est du **contenu enfant**.

***

### 3. La prop `children`

Quand tu écris du contenu **entre les balises ouvrantes et fermantes** d’un composant, React le transmet automatiquement au composant sous forme de **prop spéciale** appelée `children`.

Exemple :

```jsx
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}
```

👉 Ici, `children` contiendra tout ce que tu as mis **à l’intérieur de `<Card>...</Card>`**.

***

### 4. Exemple complet

#### App.js

```jsx
import Avatar from './Avatar.js';

function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </Card>
  );
}
```

👉 Dans cet exemple :

* `<Card>...</Card>` reçoit un `<Avatar />` comme `children`.
* `Card` l’affiche dans une `<div className="card">`.

***

### 5. Flexibilité de `children`

Tu peux remplacer `<Avatar />` par **n’importe quel contenu** :

```jsx
<Card>
  <p>Hello, je suis du texte dans Card !</p>
</Card>
```

Ou même plusieurs éléments :

```jsx
<Card>
  <h2>Titre</h2>
  <p>Un petit paragraphe</p>
</Card>
```

👉 **Card n’a pas besoin de savoir à l’avance** ce qui sera placé dedans.\
C’est ce qui fait la force de la prop `children` : elle transforme ton composant en **boîte réutilisable et polyvalente**.

***

### 6. Quand utiliser `children` ?

Tu utiliseras `children` pour tous les **composants enrobants**, par exemple :

* des **panneaux** (`<Card>`),
* des **layouts** (`<PageLayout>`),
* des **conteneurs visuels** (grilles, sections, modals…).

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

✅ **À retenir**

* Tout contenu placé entre `<Composant>...</Composant>` devient la **prop `children`**.
* Tu peux l’afficher où tu veux dans le composant enfant avec `{children}`.
* C’est une **façon souple et puissante** de construire des interfaces modulaires.

## ✨ Les props changent avec le temps

### 1. Exemple : une horloge

Voici un composant `Clock` qui reçoit deux props :

* `color` → la couleur du texte,
* `time` → l’heure à afficher.

```jsx
export default function Clock({ color, time }) {
  return (
    <h1 style={{ color: color }}>
      {time}
    </h1>
  );
}
```

👉 Ici :

* `time` est mis à jour chaque seconde par le parent,
* `color` change quand on choisit une couleur dans un menu déroulant.

***

### 2. Les props ne sont pas figées

Cet exemple montre que :

* Les **props reflètent les données à un instant donné**.
* Elles peuvent changer **quand le parent se re-render** (rafraîchit l’enfant avec de nouvelles valeurs).

Exemple visuel :

* À 12:00:01 → `time = "12:00:01"`.
* Une seconde plus tard → `time = "12:00:02"`.
* Si l’utilisateur choisit _rouge_, `color = "red"`.

***

### 3. ⚠️ Props = immuables

Même si les props **changent avec le temps**, tu ne dois **jamais les modifier directement** dans l’enfant.

👉 Mauvaise pratique (à ne jamais faire) :

```jsx
function Clock({ color, time }) {
  color = 'blue'; // ❌ on NE change PAS une prop
  return <h1 style={{ color }}>{time}</h1>;
}
```

👉 Bonne pratique :\
C’est **le parent** qui contrôle les props. Si tu veux que l’enfant change, il doit demander au parent de lui passer de **nouvelles props**.

***

### 4. Comment ça marche ?

* Quand une interaction ou des nouvelles données arrivent,
* Le **parent met à jour son état** (state) → on verra ça bientôt 😉,
* Le parent se re-render et passe des **nouvelles props** à l’enfant,
* L’enfant est réaffiché automatiquement avec les nouvelles valeurs.

💡 C’est ce qu’on appelle un **flux de données unidirectionnel** : les données descendent des parents vers les enfants via les props.

***

### 5. Métaphore simple

Pense aux **props comme à un tableau Excel** :

* Chaque cellule affiche une valeur (heure, couleur…).
* Quand les données changent, React met à jour la cellule avec la nouvelle valeur.
* Mais la cellule **ne décide jamais elle-même** de changer son contenu → c’est toujours la donnée source (le parent) qui dicte.

***

✅ **À retenir**

* Les props peuvent changer dans le temps → elles suivent les données.
* Mais elles sont **immutables** → un enfant ne peut pas les modifier.
* Si un enfant veut du changement, il **demande au parent** d’envoyer de nouvelles props.
* C’est le cœur du fonctionnement de React : **les données descendent, les événements remontent**.
