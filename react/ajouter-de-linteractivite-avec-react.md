# 🎛️ Ajouter de l’interactivité avec React

**En React, les données qui changent au fil du temps sont appelées&#x20;**_**état (state)**_**.**\
👉 C’est l’état qui permet aux composants de « se souvenir » d’une information et de **réagir aux interactions utilisateur** (clics, saisies, sélections, etc.).

***

### 📌 Dans ce chapitre, tu vas apprendre :

1. **Gérer les événements** ⚡
   * Exemple : un clic (`onClick`), une saisie clavier (`onChange`), un survol (`onMouseEnter`), etc.
   *   Les événements en React utilisent une syntaxe camelCase :

       ```jsx
       <button onClick={handleClick}>Clique-moi</button>
       ```
2. **Faire en sorte que les composants se souviennent grâce aux états** 🧠
   * Utilisation du **hook `useState`** pour stocker des données qui changent.
   *   Exemple :

       ```jsx
       const [count, setCount] = useState(0);
       ```
3. **Comprendre comment React met à jour l’UI en deux phases** 🔄
   * Phase de **rendu** → React calcule à quoi devrait ressembler l’UI.
   * Phase de **commit** → React applique les changements dans le DOM.
4. **Pourquoi l’état ne se met pas à jour immédiatement** ⏳
   * Les mises à jour sont **asynchrones** → React les regroupe pour optimiser les performances.
5. **Cumuler plusieurs mises à jour d’un même état** 🔁
   * React fusionne les mises à jour pour éviter les recalculs inutiles.
   *   Exemple :

       ```jsx
       setCount(c => c + 1);
       setCount(c => c + 1);
       // Résultat : +2 et non écrasement !
       ```
6. **Mettre à jour un objet dans l’état** 🗂️
   *   Toujours créer une **nouvelle copie immuable** de l’objet :

       ```jsx
       setUser({ ...user, name: "Alice" });
       ```
7. **Mettre à jour un tableau dans l’état** 📋
   *   Même principe → utiliser la **copie immuable** :

       ```jsx
       setItems([...items, newItem]);
       setItems(items.filter(item => item.id !== idToRemove));
       ```

***

✅ **Idée clé** : Un composant React est **pur lors du rendu** → il calcule l’UI **en fonction de ses props et de son état**.\
👉 L’interactivité vient donc des **événements** (qui déclenchent des changements d’état), et React se charge de **recalculer l’UI automatiquement**.

## 🎯 Ajouter des gestionnaires d’événements en React

### 🔹 Exemple de base : un bouton sans action

```jsx
export default function Button() {
  return (
    <button>
      Je ne fais rien
    </button>
  );
}
```

👉 Ici, ton bouton n’a **aucun comportement**. On va lui ajouter un gestionnaire d’événement.

***

### 🔹 Étape 1 : Définir une fonction dans ton composant

```jsx
function handleClick() {
  alert('Vous avez cliqué !');
}
```

***

### 🔹 Étape 2 : Passer la fonction comme prop `onClick`

```jsx
export default function Button() {
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

✅ Désormais, ton bouton **réagit au clic**.

***

### 🔹 Nommage conventionnel

* Les gestionnaires portent souvent des noms comme `handleClick`, `handleChange`, `handleSubmit`…
* La prop côté JSX garde la convention camelCase (`onClick`, `onMouseEnter`, `onChange`…).

***

### 🔹 Trois manières équivalentes d’écrire un gestionnaire

#### 1. Définir une fonction classique (lisible, réutilisable)

```jsx
function handleClick() {
  alert('Vous avez cliqué !');
}
<button onClick={handleClick}>Cliquez ici</button>
```

#### 2. Définir la fonction **directement dans le JSX**

```jsx
<button onClick={function handleClick() {
  alert('Vous avez cliqué !');
}}>
  Cliquez ici
</button>
```

#### 3. Utiliser une **fonction fléchée**

```jsx
<button onClick={() => {
  alert('Vous avez cliqué !');
}}>
  Cliquez ici
</button>
```

👉 Tu verras beaucoup le style **fonction fléchée** quand la logique est très courte.

***

### ⚠️ Piège à éviter : ne pas appeler la fonction directement

❌ Mauvais :

```jsx
<button onClick={handleClick()}>
  Cliquez ici
</button>
```

Ici, la fonction s’exécute **au moment du rendu**, pas lors du clic.

✅ Bon :

```jsx
<button onClick={handleClick}>
  Cliquez ici
</button>
```

***

### 🔑 Idée clé

En React, tu ne passes pas un résultat, tu passes une **référence à une fonction**.\
👉 React l’exécutera plus tard **quand l’événement survient**.

## 🎯 Lire les props dans les gestionnaires d’événements

Puisque les **gestionnaires d’événements** sont définis **dans un composant**, ils ont accès aux **props de ce composant**.

#### Exemple :

```jsx
function AlertButton({ message, children }) {
  return (
    <button onClick={() => alert(message)}>
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <AlertButton message="Lecture en cours !">
        Voir le film
      </AlertButton>
      <AlertButton message="Téléversement en cours !">
        Téléverser une image
      </AlertButton>
    </div>
  );
}
```

👉 Chaque bouton a une **prop différente (`message`)** et peut donc afficher un **message unique**.

***

## 🎯 Passer des gestionnaires d’événements en props

Souvent, tu ne veux pas « enfermer » la logique dans le composant enfant, mais plutôt la laisser **au parent**.\
Solution : tu passes la fonction comme **prop** 👇

#### Exemple :

```jsx
function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}

function PlayButton({ movieName }) {
  function handlePlayClick() {
    alert(`${movieName} en cours de lecture !`);
  }

  return (
    <Button onClick={handlePlayClick}>
      Voir « {movieName} »
    </Button>
  );
}

function UploadButton() {
  return (
    <Button onClick={() => alert('Téléversement en cours !')}>
      Téléverser une image
    </Button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <PlayButton movieName="Kiki la petite sorcière" />
      <UploadButton />
    </div>
  );
}
```

👉 Ici :

* `PlayButton` définit sa propre logique et la passe au composant `Button`.
* `UploadButton` fait pareil mais avec une fonction fléchée.

***

## 🎯 Nommer les props de gestionnaires d’événements

Tu peux **choisir tes propres noms** pour les props de gestionnaires dans tes composants.\
Convention : `onQuelqueChose`.

#### Exemple avec une prop renommée

```jsx
function Button({ onSmash, children }) {
  return (
    <button onClick={onSmash}>
      {children}
    </button>
  );
}

export default function App() {
  return (
    <div>
      <Button onSmash={() => alert('Lecture en cours !')}>
        Voir le film
      </Button>
      <Button onSmash={() => alert('Téléversement en cours !')}>
        Téléverser une image
      </Button>
    </div>
  );
}
```

***

## 🎯 Gestionnaires spécifiques à l’application

Tu peux créer des props **très explicites** selon ton besoin.\
Exemple : une `Toolbar` qui reçoit `onPlayMovie` et `onUploadImage` :

```jsx
export default function App() {
  return (
    <Toolbar
      onPlayMovie={() => alert('Lecture en cours !')}
      onUploadImage={() => alert('Téléversement en cours !')}
    />
  );
}

function Toolbar({ onPlayMovie, onUploadImage }) {
  return (
    <div>
      <Button onClick={onPlayMovie}>
        Voir le film
      </Button>
      <Button onClick={onUploadImage}>
        Téléverser une image
      </Button>
    </div>
  );
}

function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}
```

👉 Ici, `App` ne sait pas comment `Toolbar` va utiliser `onPlayMovie`.\
Ça donne beaucoup de **flexibilité**.

***

## ⚠️ Rappel important

Toujours utiliser les **bons éléments natifs** (`<button>` pour un bouton, pas `<div>`).\
Sinon tu perds :

* l’accessibilité clavier
* le comportement natif du navigateur

Tu peux toujours changer le **style CSS** d’un `<button>` si tu veux qu’il ressemble à un lien ou autre.

## 🔄 Propagation d’événements en React

👉 Lorsqu’un événement se produit (clic, touche, etc.), il **démarre sur l’élément ciblé** (ex. : un bouton) puis **remonte dans l’arbre des composants**, en déclenchant au passage tous les gestionnaires qui écoutent ce même événement.

***

#### Exemple :

```jsx
export default function Toolbar() {
  return (
    <div
      className="Toolbar"
      onClick={() => {
        alert('Vous avez cliqué sur la barre d’outils !');
      }}
    >
      <button onClick={() => alert('Lecture en cours !')}>
        Voir le film
      </button>
      <button onClick={() => alert('Téléversement en cours !')}>
        Téléverser une image
      </button>
    </div>
  );
}
```

***

#### 🖱️ Cas possibles :

* **Clique sur "Voir le film"**\
  👉 `alert('Lecture en cours !')` → puis **propagation** → `alert('Vous avez cliqué sur la barre d’outils !')`.
* **Clique sur "Téléverser une image"**\
  👉 `alert('Téléversement en cours !')` → puis **propagation** → `alert('Vous avez cliqué sur la barre d’outils !')`.
* **Clique directement sur la `div.Toolbar` (hors des boutons)**\
  👉 Seul `alert('Vous avez cliqué sur la barre d’outils !')`.

***

## ⚠️ Piège

* **Tous les événements se propagent en React**, sauf **`onScroll`** :\
  → il ne déclenche que sur l’élément où il est défini.

***

## ✋ Stopper la propagation

Parfois tu veux empêcher l’événement de remonter (ex. : éviter que le clic d’un bouton déclenche aussi un événement parent).\
Tu utilises **`event.stopPropagation()`** :

```jsx
export default function Toolbar() {
  return (
    <div
      className="Toolbar"
      onClick={() => alert('Vous avez cliqué sur la barre d’outils !')}
    >
      <button
        onClick={(e) => {
          e.stopPropagation(); // ✋ Stoppe la remontée
          alert('Lecture en cours !');
        }}
      >
        Voir le film
      </button>
    </div>
  );
}
```

👉 Ici, cliquer sur le bouton **affiche seulement** "Lecture en cours !" (le parent n’est pas alerté).

## 🛑 Stopper la propagation (`e.stopPropagation()`)

Quand un événement se déclenche :

1. React appelle d’abord le gestionnaire **directement attaché à l’élément cliqué**.
2. Ensuite, l’événement **remonte dans l’arbre** (phase de **bubbling**) → les parents reçoivent aussi l’événement.
3. Si tu ne veux **pas que l’événement remonte**, tu appelles `e.stopPropagation()`.

#### Exemple (corrigé avec stopPropagation) :

```jsx
function Button({ onClick, children }) {
  return (
    <button
      onClick={(e) => {
        e.stopPropagation(); // 🔴 stoppe la propagation vers les parents
        onClick(); // ✅ exécute la logique du bouton
      }}
    >
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div
      className="Toolbar"
      onClick={() => {
        alert("Vous avez cliqué sur la barre d’outils !");
      }}
    >
      <Button onClick={() => alert("Lecture en cours !")}>
        Voir le film
      </Button>
      <Button onClick={() => alert("Téléversement en cours !")}>
        Téléverser une image
      </Button>
    </div>
  );
}
```

👉 Résultat :

* Clic sur un **bouton** → **1 seule alert** (celle du bouton).
* Clic sur la **barre d’outils** (div) → **alert du parent**.

***

## 🔎 Les phases d’un événement en React

Chaque événement passe par **3 phases** :

1. **Capture (descente)** → l’événement part de la racine et **descend** vers l’élément cible.
   * React déclenche les gestionnaires `onClickCapture`.
2. **Cible** → l’événement exécute le gestionnaire sur **l’élément cliqué**.
   * React déclenche `onClick`.
3. **Bubbling (remontée)** → l’événement **remonte** dans l’arbre.
   * React déclenche les `onClick` des parents.

***

## 🪣 Exemple avec capture

```jsx
export default function Toolbar() {
  return (
    <div
      className="Toolbar"
      onClick={() => alert("Bubbling → div")}
      onClickCapture={() => alert("Capture → div")}
    >
      <button
        onClick={(e) => {
          e.stopPropagation();
          alert("Bouton cliqué !");
        }}
      >
        Cliquez-moi
      </button>
    </div>
  );
}
```

👉 Si tu cliques sur le bouton :

* **Phase de capture** → `Capture → div`
* **Phase de cible** → `Bouton cliqué !`
* (La propagation vers le parent est stoppée, donc pas de `Bubbling → div`).

***

✅ **À retenir** :

* `onClick` = phase de **bubbling** (par défaut, la plus utilisée).
* `onClickCapture` = phase de **capture** (rare, utile pour les logs, analytics, etc.).
* `e.stopPropagation()` = bloque la remontée après l’élément cible.

## ⚡ Passer des gestionnaires d’événements au lieu de propager

Vous remarquerez ci-dessous que le gestionnaire du clic exécute une ligne de code **puis appelle la prop `onClick` passée par le parent** :

```jsx
function Button({ onClick, children }) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      onClick();
    }}>
      {children}
    </button>
  );
}
```

👉 Vous pourriez ajouter davantage de code à ce gestionnaire **avant** d’appeler le gestionnaire d’événement `onClick` du parent.

Cette approche offre une alternative à la **propagation** :

* ✅ Le composant enfant gère d’abord son événement.
* ✅ Le composant parent peut quand même spécifier un comportement supplémentaire.

💡 Contrairement à la propagation (automatique), ici c’est **explicite et traçable** → vous savez clairement quel code s’exécute.

***

## 🛑 Empêcher le comportement par défaut

Certains événements natifs du navigateur ont un **comportement par défaut**.

Exemple : dans un `<form>`, cliquer sur un bouton **soumet le formulaire** et provoque par défaut un **rechargement complet** de la page :

```jsx
export default function Signup() {
  return (
    <form onSubmit={() => alert('Envoi en cours !')}>
      <input />
      <button>Envoyer</button>
    </form>
  );
}
```

👉 Pour bloquer ce comportement, utilisez **`e.preventDefault()`** :

```jsx
export default function Signup() {
  return (
    <form onSubmit={e => {
      e.preventDefault(); // 🔴 stoppe le rechargement par défaut
      alert('Envoi en cours !');
    }}>
      <input />
      <button>Envoyer</button>
    </form>
  );
}
```

***

## 🔎 Différence entre `e.stopPropagation()` et `e.preventDefault()`

* **`e.stopPropagation()`** → empêche la **remontée** de l’événement vers les parents (bubbling).
* **`e.preventDefault()`** → empêche le **comportement par défaut** du navigateur (ex. soumission de formulaire, navigation de lien).

👉 Les deux sont utiles mais **indépendants** : on peut utiliser l’un, l’autre, ou les deux selon le besoin.

## ⚡ Les gestionnaires d’événements peuvent-ils avoir des effets de bord ?

👉 **Oui, absolument !**\
Les **gestionnaires d’événements** sont l’endroit **idéal** pour introduire des **effets de bord**.

***

## 🔎 Différence avec les fonctions de rendu

* Les **fonctions de rendu** doivent rester **pures** : elles décrivent seulement le JSX à afficher, **sans rien modifier autour**.
* Les **gestionnaires d’événements**, en revanche, **n’ont pas besoin d’être purs**.

Cela signifie que dans un gestionnaire d’événement, vous pouvez :\
✅ Modifier une valeur d’entrée (`input`) en réponse à une frappe clavier.\
✅ Mettre à jour une liste quand l’utilisateur clique sur un bouton.\
✅ Déclencher une requête API, lancer une animation, etc.

***

## 📌 Exemple simple

```jsx
export default function Counter() {
  let count = 0;

  function handleClick() {
    count = count + 1; // effet de bord : on modifie une variable
    alert("Compteur : " + count);
  }

  return <button onClick={handleClick}>+1</button>;
}
```

⚠️ Ici on modifie une variable, mais en React ce n’est pas suffisant : le rendu ne se mettra pas à jour.

***

## 💡 La solution : l’état (state)

En React, pour que les modifications soient **mémorisées et visibles à l’écran**, il faut les stocker dans **l’état d’un composant** grâce au hook `useState`.

👉 C’est ce qu’on va voir dans la **prochaine étape** : **l’état = la mémoire d’un composant**.

