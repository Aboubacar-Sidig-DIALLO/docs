# ✨ Affichage conditionnel dans React

### 1. Principe

Un composant React retourne du **JSX**.\
Mais tu n’es pas obligé de toujours retourner la même chose : tu peux utiliser **les conditions de JavaScript** pour contrôler ce qui est affiché.

C’est comme dire :

* _“Si telle condition est vraie, affiche ça. Sinon, affiche autre chose.”_

***

### 2. Avec `if`

Tu peux utiliser un **if classique** pour retourner différents JSX :

```jsx
function Greeting({ isLoggedIn }) {
  if (isLoggedIn) {
    return <h1>Bienvenue !</h1>;
  }
  return <h1>Veuillez vous connecter.</h1>;
}
```

👉 Ici :

* Si `isLoggedIn = true` → affiche _Bienvenue !_
* Sinon → affiche _Veuillez vous connecter._

***

### 3. Avec l’opérateur ternaire `? :`

Le ternaire est plus compact :

```jsx
function Greeting({ isLoggedIn }) {
  return (
    <h1>
      {isLoggedIn ? 'Bienvenue !' : 'Veuillez vous connecter.'}
    </h1>
  );
}
```

👉 Même logique que le `if`, mais en une seule ligne.

***

### 4. Avec l’opérateur logique `&&`

Pour afficher un élément **seulement si une condition est vraie**, tu peux utiliser `&&` :

```jsx
function Notification({ hasMessages }) {
  return (
    <div>
      <h1>Tableau de bord</h1>
      {hasMessages && <p>Vous avez de nouveaux messages.</p>}
    </div>
  );
}
```

👉 Ici :

* Si `hasMessages = true` → affiche le `<p>`.
* Si `hasMessages = false` → React ignore le `<p>`.

***

### 5. Exemple concret

```jsx
export default function UserProfile({ user }) {
  return (
    <div>
      <h1>Profil utilisateur</h1>

      {user ? (
        <p>Bienvenue, {user.name} !</p>
      ) : (
        <p>Veuillez vous connecter.</p>
      )}
    </div>
  );
}
```

👉 Ici :

* Si `user` existe → affiche son nom.
* Sinon → demande de se connecter.

***

### 6. Résumé des options

* **`if` classique** → lisible pour des blocs de JSX complexes.
* **`? :` ternaire** → concis pour alterner entre deux options.
* **`&&`** → pratique pour afficher un élément seulement si la condition est vraie.

***

✅ **À retenir**

* L’affichage conditionnel en React utilise les outils **classiques de JavaScript**.
* Tu choisis entre `if`, `? :` ou `&&` selon le cas.
* Ça rend tes composants **souples et intelligents**, capables de s’adapter aux données ou aux actions utilisateur.

## ✨ Renvoi conditionnel de JSX

### 1. Exemple de base

Ton composant `Item` reçoit deux props :

* `name` → le nom de l’objet,
* `isPacked` → un booléen (true ou false).

👉 On veut afficher une ✅ si `isPacked` vaut `true`.

#### Version avec `if/else`

```jsx
function Item({ name, isPacked }) {
  if (isPacked) {
    return <li className="item">{name} ✅</li>;
  }
  return <li className="item">{name}</li>;
}
```

✅ Si `isPacked = true` → affiche `Nom de l’objet ✅`.\
❌ Si `isPacked = false` → affiche juste `Nom de l’objet`.

***

### 2. Utiliser l’opérateur ternaire `? :`

Une alternative plus compacte que `if/else` :

```jsx
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {isPacked ? name + ' ✅' : name}
    </li>
  );
}
```

👉 Avantage : une seule instruction `return`.

***

### 3. Utiliser `&&` pour ne rien afficher

Si tu veux **ajouter seulement la coche quand c’est nécessaire**, tu peux utiliser `&&` :

```jsx
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked && '✅'}
    </li>
  );
}
```

* Si `isPacked = true` → React affiche `Nom ✅`.
* Si `isPacked = false` → React affiche seulement `Nom`.

⚠️ Attention : `&&` affiche la valeur de droite uniquement si la gauche est **true**.

***

### 4. Exemple complet

```jsx
export default function PackingList() {
  return (
    <section>
      <h1>Liste d’affaires de Sally Ride</h1>
      <ul>
        <Item isPacked={true} name="Combinaison spatiale" />
        <Item isPacked={true} name="Casque à feuille d’or" />
        <Item isPacked={false} name="Photo de Tam" />
      </ul>
    </section>
  );
}
```

Résultat :

* ✅ sur les deux premiers éléments,
* Pas de coche sur la photo.

***

### 5. À retenir

* Tu peux conditionner ton JSX avec **if/else**, **? :**, ou **&&**.
* Chaque approche a son usage :
  * `if/else` → pour du JSX complexe.
  * `? :` → pour alterner entre deux valeurs.
  * `&&` → pour afficher **uniquement si une condition est vraie**.

## ✨ 1. Renvoyer `null` (ne rien afficher)

Un composant React **doit toujours renvoyer quelque chose**.\
Mais si tu veux “ne rien afficher du tout”, tu peux renvoyer `null` :

```jsx
function Item({ name, isPacked }) {
  if (isPacked) {
    return null; // rien ne sera affiché
  }
  return <li className="item">{name}</li>;
}
```

👉 Ici :

* Si `isPacked = true` → React n’affiche rien pour cet item.
* Sinon → affiche l’élément `<li>`.

⚠️ Astuce : on n’utilise pas `undefined` ou `false` → **seul `null`** signifie explicitement “ne rien afficher”.

***

## ✨ 2. Inclure du JSX conditionnellement

Parfois, tu veux **garder la même structure** mais changer seulement une partie (par ex. le texte ou une balise).\
👉 Pour éviter la duplication, on inclut **juste un morceau conditionnel**.

#### Exemple avec opérateur ternaire `? :`

```jsx
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {isPacked ? name + ' ✅' : name}
    </li>
  );
}
```

👉 Si `isPacked = true` → ajoute un ✅.\
👉 Sinon → affiche seulement `name`.

***

## ✨ 3. Exemple avec du balisage imbriqué

On peut aller plus loin : entourer le texte par une balise `<del>` si l’objet est déjà empaqueté :

```jsx
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {isPacked ? (
        <del>{name + ' ✅'}</del>
      ) : (
        name
      )}
    </li>
  );
}
```

👉 Résultat :

* Objet empaqueté → barré avec un ✅.
* Objet pas encore empaqueté → texte normal.

***

## ✨ 4. Quand utiliser quoi ?

* **`null`** → si tu veux **ne rien afficher du tout** (l’élément disparaît).
* **Condition interne (ternaire ou `&&`)** → si tu veux **changer seulement une partie du JSX**.

***

✅ **À retenir**

* Renvoyer `null` = “ne pas afficher ce composant”.
* Pour éviter la duplication de JSX, tu conditionnes **à l’intérieur du retour**, plutôt que de répéter deux fois le même `<li>`.
* Utilise `? :` pour deux cas clairs, `&&` pour ajouter un bout seulement.

## 1. L’opérateur logique **ET (`&&`)**

C’est le plus concis pour afficher **quelque chose seulement si une condition est vraie** :

```jsx
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked && '✅'}
    </li>
  );
}
```

👉 Lecture naturelle :

* si `isPacked = true` → affiche la coche ✅
* si `isPacked = false` → affiche juste `name`

⚠️ **Piège classique** :\
Ne mets pas un nombre à gauche de `&&`.

```jsx
messageCount && <p>Nouveaux messages</p>
```

Si `messageCount = 0` → React affichera… **0** (car `0 && ...` renvoie `0`).\
✅ Solution : force la condition en booléen →

```jsx
messageCount > 0 && <p>Nouveaux messages</p>
```

***

## 🔹 2. Affecter conditionnellement à une variable

Style plus **verbeux mais lisible et flexible**.\
On commence avec une valeur par défaut, puis on la modifie selon la condition :

```jsx
function Item({ name, isPacked }) {
  let itemContent = name;
  if (isPacked) {
    itemContent = name + " ✅";
  }
  return (
    <li className="item">
      {itemContent}
    </li>
  );
}
```

👉 Avantage :

* Plus facile à lire quand les conditions deviennent complexes.
* Permet d’utiliser **n’importe quel JSX**, pas seulement du texte :

```jsx
if (isPacked) {
  itemContent = <del>{name + " ✅"}</del>;
}
```

Résultat → les items empaquetés sont **barrés** avec un ✅.

***

## 🔹 3. Comparaison rapide des styles

| Style                     | Exemple                           | Avantages                   | Inconvénients                                |
| ------------------------- | --------------------------------- | --------------------------- | -------------------------------------------- |
| **if/else avec `return`** | `if (isPacked) return ...`        | Très clair, pas d’ambiguïté | Peut dupliquer du JSX                        |
| **Ternaire `? :`**        | `{isPacked ? name + " ✅" : name}` | Concis, parfait pour 2 cas  | Devient lourd avec des conditions imbriquées |
| **`&&`**                  | `{isPacked && "✅"}`               | Ultra court, lisible        | ⚠️ Affiche 0 si valeur numérique             |
| **Variable `let`**        | `let content = ...`               | Flexible, maintenable       | Plus verbeux                                 |

***

## ✨ À retenir

* Utilise `&&` pour afficher **ou rien**.
* Utilise `? :` si tu as **2 branches alternatives**.
* Utilise `let + if` si ça devient **plus complexe**.

👉 Prochaine étape logique : après avoir appris à conditionner **un seul élément**, tu vas adorer voir **comment générer une liste complète avec `.map()`** (ultra utilisé en React : todo-lists, galeries, menus, etc.).\
Veux-tu que je t’explique la **rendu de listes avec `.map()`** ?
