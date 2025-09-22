# 🧠⚛️Penser en React

React ne sert pas seulement à coder : il change aussi ta manière de **réfléchir** quand tu construis une application.\
Au lieu de voir ton interface comme un gros bloc, tu vas apprendre à la **découper**, à imaginer ses **différents états visuels**, et à **faire circuler les données** entre les morceaux **(du parent vers les enfants, et parfois inversement).**

Pour illustrer ça, on va créer ensemble une petite appli :\
👉 Un **tableau de produits filtrable** (tu peux rechercher un produit et n’afficher que ceux en stock).

### Étape 1 : Décomposer l’UI en une hiérarchie de composants

Imaginons que ton designer t’a donné une maquette, et que tu as déjà une API JSON qui renvoie les produits :

```json
[
  { "category": "Fruits", "price": "$1", "stocked": true, "name": "Pomme" },
  { "category": "Fruits", "price": "$1", "stocked": true, "name": "Fruit du dragon" },
  { "category": "Fruits", "price": "$2", "stocked": false, "name": "Fruit de la passion" },
  { "category": "Légumes", "price": "$2", "stocked": true, "name": "Épinard" },
  { "category": "Légumes", "price": "$4", "stocked": false, "name": "Citrouille" },
  { "category": "Légumes", "price": "$1", "stocked": true, "name": "Petits pois" }
]
```

La maquette ressemble à ça :

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1).png" alt="" width="257"><figcaption></figcaption></figure>



👉 Côté design, tu vois un tableau avec :

* une **barre de recherche**,
* une **case à cocher** “n’afficher que les produits en stock”,
* un **tableau de produits** classés par catégorie.

***

### Les 5 étapes à suivre avec React

Quand tu transformes une maquette en appli React, tu vas généralement suivre **5 étapes** simples :

#### 1. **Découper l’UI en composants**

Imagine que tu découpes ta maquette en morceaux logiques.\
Chaque morceau devient un **composant React**.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt="" width="334"><figcaption></figcaption></figure>

Exemple dans notre cas :

* `FilterableProductTable` → le conteneur principal.
* `SearchBar` → la zone de recherche + case à cocher.
* `ProductTable` → le tableau filtré.
* `ProductCategoryRow` → l’entête de chaque catégorie.
* `ProductRow` → une ligne pour un produit.

👉 Chaque composant a une **responsabilité claire**.

Maintenant que vous avez identifié les composants de la maquette, déterminez leur hiérarchie. Les composants qui apparaissent au sein d’un autre composant dans la maquette devraient apparaître comme enfants dans cette arborescence :

* `FilterableProductTable`
  * `SearchBar`
  * `ProductTable`
    * `ProductCategoryRow`
    * `ProductRow`

***

#### 2. **Construire une version statique**

Commence par afficher les données **sans interactivité**.

* Pas d’état (`useState`) pour l’instant.
* Tu passes les données avec des **props** du parent vers les enfants.

L’idée ? Mettre en place la structure visuelle avant de s’occuper des interactions.\
Ça évite de se perdre : tu poses les briques d’abord, puis tu leur donnes vie.

Pourquoi ?

* Construire une version statique demande surtout de la saisie (écrire du JSX), mais peu de réflexion.
* Ajouter de l’interactivité, au contraire, demande plus de réflexion (où placer l’état, comment gérer les événements), mais peu de code supplémentaire.\
  💡 Donc autant séparer les deux : **visuel d’abord, logique ensuite**.

***

### Les props comme canal de données 📦➡️

Pour cette version statique, tu vas utiliser les **props** :

* Les props permettent à un **composant parent** de transmettre des données à un **enfant**.
* Elles sont en lecture seule : l’enfant affiche simplement ce qu’il reçoit.

⚠️ Pas d’état (`useState`) à ce stade !\
L’état est réservé aux données qui **changent dans le temps**. Ici, on veut juste **afficher les infos brutes**.

***

### Construire de haut en bas ou de bas en haut ? 🔝🔽 ou 🔽🔝

Deux approches sont possibles :

1. **Top-down (haut → bas)** : tu commences par les composants principaux (`FilterableProductTable`) puis tu descends.
2. **Bottom-up (bas → haut)** : tu commences par les petits composants réutilisables (`ProductRow`, `ProductCategoryRow`) et tu remontes.

👉 Pour des applis simples, la première approche est plus naturelle.\
👉 Pour des projets plus gros, on préfère souvent construire de bas en haut.

Exemple de version statique 📄

```jsx
function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Nom</th>
          <th>Prix</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar() {
  return (
    <form>
      <input type="text" placeholder="Recherche..." />
      <label>
        <input type="checkbox" />
        {' '}
        N’afficher que les produits en stock
      </label>
    </form>
  );
}

function FilterableProductTable({ products }) {
  return (
    <div>
      <SearchBar />
      <ProductTable products={products} />
    </div>
  );
}

const PRODUCTS = [
  { category: "Fruits", price: "1 €", stocked: true, name: "Pomme" },
  { category: "Fruits", price: "1 €", stocked: true, name: "Fruit du dragon" },
  { category: "Fruits", price: "2 €", stocked: false, name: "Fruit de la passion" },
  { category: "Légumes", price: "2 €", stocked: true, name: "Épinard" },
  { category: "Légumes", price: "4 €", stocked: false, name: "Citrouille" },
  { category: "Légumes", price: "1 €", stocked: true, name: "Petits pois" }
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}

```

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt="" width="229"><figcaption></figcaption></figure>

👉 Ici :

* `ProductCategoryRow` affiche un en-tête de catégorie.
* `ProductRow` affiche une ligne de produit (en rouge si le produit n’est pas en stock).
* `ProductTable` parcourt les produits et assemble les lignes dans un tableau.

***

### Flux de données unidirectionnel 🔽

Dans cette version statique :

* Les données **descendent** depuis le composant racine (`FilterableProductTable`).
* Elles circulent uniquement par les **props** vers les composants enfants (`ProductTable`, `ProductRow`, etc.).

On appelle ça le **flux de données unidirectionnel** :\
👉 Les parents transmettent les données, les enfants se contentent de les afficher.

***

⚠️ **Piège à éviter** :\
Ne mets pas d’état (`useState`) pour le moment.\
Si tu sens que tu veux en ajouter, respire 😅 : ce sera l’étape suivante. Ici, tu n’as besoin que de **props et JSX**.

***

#### 3. **Identifier le minimum d’état nécessaire**

Jusqu’ici, ton appli affichait juste des données statiques.\
👉 Mais pour la rendre **interactive**, il faut qu’elle puisse **se souvenir de certaines infos** et les **mettre à jour** quand l’utilisateur agit dessus.

C’est là qu’intervient **l’état** (_state_).

⚠️ Règle d’or : garde ton état au **strict minimum**.

***

### L’état : c’est quoi au juste ? 🧠

* Imagine l’état comme la **mémoire interne** de ton composant.
* C’est **le minimum d’informations qui peuvent changer** au fil du temps, et dont l’appli doit se souvenir pour fonctionner.
* Tout le reste ? Tu peux le **recalculer à la volée**.

👉 Principe d’or : garde ton état **DRY (Don’t Repeat Yourself)**.\
Pas de doublons !\
Si une info peut être dérivée d’une autre, ne la stocke pas.

Exemple concret :

* Si tu as un tableau `panier = [ ... ]` dans ton état,
* et que tu veux afficher le nombre d’articles,\
  ⚠️ inutile de stocker `nbArticles` comme état aussi → tu peux simplement faire `panier.length`.

***

### Appliquons ça à notre tableau de produits 🛒

Les données disponibles sont :

1. **La liste originale des produits**\
   → Elle vient des **props** (elle est fournie par l’API).\
   ❌ Pas besoin de l’avoir en état.
2. **Le texte de recherche** (ce que tape l’utilisateur)\
   → Ça change avec le temps, et ça ne peut pas être calculé ailleurs.\
   ✅ C’est de l’état.
3. **La case à cocher** (“n’afficher que les produits en stock”)\
   → Idem, ça change selon l’utilisateur.\
   ✅ C’est de l’état.
4. **La liste filtrée des produits**\
   → Elle peut être recalculée à partir de la **liste originale + texte de recherche + case cochée**.\
   ❌ Donc pas besoin de la stocker comme état.

👉 Résultat :\
Seuls **le texte de recherche** et **l’état de la case à cocher** sont du véritable **state**.

***

### Props vs État ⚡

Pour bien distinguer les deux, voici une analogie simple :

* **Les props** → ce sont comme les **arguments d’une fonction**.
  * Elles sont données par le parent,
  * et elles servent à personnaliser ou configurer un composant.
  * Exemple : un composant `<Button color="blue" />`.
* **L’état** → c’est la **mémoire interne** d’un composant.
  * Il évolue avec le temps,
  * et permet au composant de réagir aux actions de l’utilisateur.
  * Exemple : un bouton qui garde en mémoire s’il est survolé (`isHovered`).

👉 Dans la pratique :\
Un **composant parent** garde souvent des infos dans son **état**,\
et transmet ces infos à ses **enfants** via leurs **props**.

⚠️ Si la frontière entre props et état te semble encore floue → c’est normal !\
Ça devient naturel avec la pratique 😉.

***

#### 4. **Choisir où l’état doit vivre**

Tu sais maintenant que dans notre exemple, les deux seuls vrais états sont :

* le **texte de recherche** (`filterText`)
* et l’**état de la case à cocher** (`inStockOnly`).

Reste à savoir : **qui doit les posséder** ?

***

### Rappel : flux de données en React 🔽

En React, les données suivent un **flux unidirectionnel** :\
👉 Elles descendent des **parents vers les enfants** via les **props**.

Résultat :

* Un composant qui **possède un état** (via `useState`) peut l’envoyer à ses enfants.
* Mais l’inverse n’est pas vrai : un enfant **ne peut pas créer ou modifier directement l’état d’un parent**.

D’où la question :\
**Dans quel composant placer l’état pour que tout le monde ait accès à ce dont il a besoin ?**

***

### La méthode pour décider 🤔

Pour chaque morceau d’état, fais ces trois étapes :

1. **Identifie quels composants en ont besoin.**
   * Qui lit ou affiche cette donnée ?
   * Qui doit la modifier ?
2. **Trouve leur plus proche ancêtre commun.**
   * Le composant le plus haut dans l’arborescence qui englobe tous ceux qui utilisent cet état.
3. **Place l’état dans cet ancêtre.**
   * Comme ça, l’état est accessible à tous les composants concernés.
   * Et tu peux le transmettre facilement via les props.

👉 Si tu ne trouves aucun composant logique où mettre l’état, crée-en un nouveau juste pour ça.

***

### Application à notre cas 🎯

* Le texte de recherche et la case à cocher sont utilisés par :
  * **`SearchBar`** → pour les afficher et permettre à l’utilisateur de les modifier.
  * **`ProductTable`** → pour filtrer la liste des produits.
* Leur **ancêtre commun** est **`FilterableProductTable`**.

👉 Conclusion : l’état doit vivre dans **`FilterableProductTable`**.

***

### Mise en place avec `useState` ⚡

On ajoute donc deux variables d’état dans `FilterableProductTable` :

```jsx
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly}
      />
      <ProductTable
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly}
      />
    </div>
  );
}
```

👉 Désormais :

* `FilterableProductTable` **possède l’état**,
* et le **passe aux enfants** (`SearchBar` et `ProductTable`) via des **props**.

***

### Petit test 🧪

Essaie de changer la valeur initiale :

```jsx
const [filterText, setFilterText] = useState('fruit');
```

Tu verras que le champ de recherche est pré-rempli avec “fruit” et que le tableau est déjà filtré.\
Preuve que ton état **contrôle bien l’UI** 💪.

***

```jsx
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly} />
      <ProductTable
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Nom</th>
          <th>Prix</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input
        type="text"
        value={filterText}
        placeholder="Recherche..."/>
      <label>
        <input
          type="checkbox"
          checked={inStockOnly} />
        {' '}
        N’afficher que les produits en stock
      </label>
    </form>
  );
}

const PRODUCTS = [
  { category: "Fruits", price: "1 €", stocked: true, name: "Pomme" },
  { category: "Fruits", price: "1 €", stocked: true, name: "Fruit du dragon" },
  { category: "Fruits", price: "2 €", stocked: false, name: "Fruit de la passion" },
  { category: "Légumes", price: "2 €", stocked: true, name: "Épinard" },
  { category: "Légumes", price: "4 €", stocked: false, name: "Citrouille" },
  { category: "Légumes", price: "1 €", stocked: true, name: "Petits pois" }
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}

```

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt="" width="230"><figcaption></figcaption></figure>

***

⚠️ Attention : si tu essaies de taper dans la barre de recherche à ce stade… ça ne marche pas encore.\
Pourquoi ?\
Parce qu’on a bien fixé la valeur avec une prop (`value={filterText}`), mais on n’a pas encore branché d’**événement `onChange`** pour mettre à jour l’état.

👉 Et c’est justement ce qu’on fera dans l’**étape 5 : ajouter un flux de données inverse** 🔄.

#### 5. **Ajouter le flux de données inverse**

Jusqu’ici, ton appli affichait bien les données :

* `FilterableProductTable` contenait l’état,
* et cet état descendait dans `SearchBar` et `ProductTable` via les **props**.

👉 Problème : impossible de taper dans la barre de recherche ou de cocher la case.\
Pourquoi ? Parce que les champs de formulaire sont **contrôlés par React** :

* Leur `value` est fixé par une prop,
* mais aucun événement ne permet de **mettre à jour cet état**.

Résultat → l’UI est bloquée (lecture seule).\
Et c’est volontaire ! React veut que tu rendes **explicite** la façon dont les données remontent.

***

### L’idée clé 🗝️

Quand un utilisateur tape du texte ou clique sur la case :

* le champ doit **demander au parent** (`FilterableProductTable`) de mettre à jour l’état.
* Ensuite, le parent redescend la nouvelle valeur en props → et le champ se met à jour.

👉 C’est ce qu’on appelle **un flux de données inverse**.

***

### Comment faire ?

#### 1. Passer des fonctions de mise à jour en props

Dans `FilterableProductTable`, on passe directement les fonctions `setFilterText` et `setInStockOnly` :

```jsx
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly}
      />
      <ProductTable
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly}
      />
    </div>
  );
}
```

👉 Ici, `SearchBar` reçoit **à la fois** :

* les valeurs (`filterText`, `inStockOnly`),
* et les moyens de les modifier (`onFilterTextChange`, `onInStockOnlyChange`).

***

#### 2. Ajouter les gestionnaires d’événements dans `SearchBar`

Dans `SearchBar`, on connecte chaque champ avec un `onChange` :

```jsx
function SearchBar({
  filterText,
  inStockOnly,
  onFilterTextChange,
  onInStockOnlyChange
}) {
  return (
    <form>
      <input
        type="text"
        value={filterText}
        placeholder="Recherche..."
        onChange={(e) => onFilterTextChange(e.target.value)}
      />
      <label>
        <input
          type="checkbox"
          checked={inStockOnly}
          onChange={(e) => onInStockOnlyChange(e.target.checked)}
        />
        {' '}
        N’afficher que les produits en stock
      </label>
    </form>
  );
}
```

👉 Résultat :

* Quand l’utilisateur tape dans l’input → `onFilterTextChange` est appelé → `setFilterText` met à jour l’état du parent.
* Quand il coche la case → `onInStockOnlyChange` est appelé → `setInStockOnly` met à jour l’état du parent.

***

```jsx
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly} />
      <ProductTable
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Nom</th>
          <th>Prix</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({
  filterText,
  inStockOnly,
  onFilterTextChange,
  onInStockOnlyChange
}) {
  return (
    <form>
      <input
        type="text"
        value={filterText} placeholder="Recherche..."
        onChange={(e) => onFilterTextChange(e.target.value)} />
      <label>
        <input
          type="checkbox"
          checked={inStockOnly}
          onChange={(e) => onInStockOnlyChange(e.target.checked)} />
        {' '}
        N’afficher que les produits en stock
      </label>
    </form>
  );
}

const PRODUCTS = [
  { category: "Fruits", price: "1 €", stocked: true, name: "Pomme" },
  { category: "Fruits", price: "1 €", stocked: true, name: "Fruit du dragon" },
  { category: "Fruits", price: "2 €", stocked: false, name: "Fruit de la passion" },
  { category: "Légumes", price: "2 €", stocked: true, name: "Épinard" },
  { category: "Légumes", price: "4 €", stocked: false, name: "Citrouille" },
  { category: "Légumes", price: "1 €", stocked: true, name: "Petits pois" }
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}

```

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1) (1).png" alt="" width="233"><figcaption></figcaption></figure>

### Résultat final 🎉

Ton application fonctionne maintenant **de bout en bout** :

* L’état vit dans `FilterableProductTable`.
* Les props descendent vers `SearchBar` et `ProductTable`.
* Les événements remontent de `SearchBar` vers `FilterableProductTable`.
* Le tableau se met à jour automatiquement selon la recherche et le filtre.

C’est le **cycle React classique** :

* **État → Props → UI** (descendant)
* **Événements → Mise à jour de l’état → Nouveau rendu** (ascendant + descendant).
