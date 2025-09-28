# 🍳 Rendu et Commit : la cuisine de React

Imagine que **ton appli React soit un restaurant**.

* Tes **composants** = les cuisiniers.
* Le **DOM (l’écran)** = la table du client.
* **React** = le serveur qui prend la commande et s’assure que les bons plats arrivent.

Le processus se fait en **3 grandes étapes** :

***

### 1️⃣ Déclenchement du rendu (prendre la commande)

Le rendu est déclenché quand :

* Le **composant racine** est affiché pour la première fois (`ReactDOM.createRoot(...).render(<App />)`).
* Ou quand un **état** ou une **prop** change → React doit recalculer ce qu’il doit afficher.

👉 Exemple :

```jsx
setCount(count + 1); // 🛎️ dit à React "recalcule-moi l’UI"
```

***

### 2️⃣ Rendu du composant (cuisine en action 👨‍🍳)

Pendant cette étape, **React exécute ton composant comme une fonction**.

* Il lit les **props** et **états** actuels.
* Il produit un **arbre de JSX** (la "commande préparée").
* Mais ⚠️ **rien n’est encore affiché à l’écran** → ce n’est qu’une description de l’UI.

👉 Exemple :

```jsx
function Button({ count }) {
  return <button>Cliqué {count} fois</button>;
}
```

Si `count = 3`, React produit une description :

```
<button>Cliqué 3 fois</button>
```

***

### 3️⃣ Commit (service en salle 🍽️)

Une fois que React sait **ce qui a changé**, il met à jour le **DOM** :

* Si c’est le 1er rendu → il **crée** tous les nœuds DOM nécessaires.
* Si c’est une mise à jour → il fait un **diff** entre l’ancien arbre et le nouveau, et ne modifie que ce qui a changé.

👉 Exemple :

* Avant : `<button>Cliqué 3 fois</button>`
* Après : `<button>Cliqué 4 fois</button>`\
  ➡️ React ne recrée pas tout, il change juste le texte.

***

## ⚡ Points importants à retenir

* **Rendu ≠ Commit**\
  Le rendu ne modifie pas directement l’écran. Il **prépare** seulement l’UI. Le commit, lui, applique les changements au DOM.
* **Le DOM n’est pas toujours mis à jour**\
  Si le rendu produit exactement le même JSX qu’avant → React n’a rien à changer dans le DOM.
* **Performance**\
  Comme React "cuisine" en mémoire avant de servir, il évite de toucher inutilement au DOM (qui est lent à manipuler).

***

✅ **Résumé (métaphore restaurant)**

1. **Déclenchement du rendu** → Le serveur prend la commande (un changement d’état/props).
2. **Rendu du composant** → La cuisine prépare les plats (React calcule le JSX).
3. **Commit** → Le serveur amène les plats (React applique les changements au DOM).

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

## ⚡ Étape 1 : Déclenchement d’un rendu

Il existe **deux cas** où React décide de faire le rendu d’un composant :

***

### 🟢 1. Rendu initial

C’est le tout premier affichage de ton composant dans le DOM.

👉 Exemple classique (dans `index.js`) :

```jsx
import Image from './Image.js';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(<Image />);
```

***

### 🔄 2. Nouveaux rendus suite à des mises à jour d’état

Une fois ton composant affiché, un rendu peut être redemandé si **l’état local change** via un `setState` (`setXxx`).

👉 Exemple :

```jsx
function Card() {
  const [color, setColor] = useState("black");

  return (
    <div>
      <button onClick={() => setColor("pink")}>Changer en rose</button>
      <div className={`card ${color}`}>Carte {color}</div>
    </div>
  );
}
```

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

1. La cliente (utilisateur) dit : _"Je veux une carte rose, pas noire !"_
2. React retourne dans la **Cuisine des Composants** 🧑‍🍳.
3. Le **Chef des Cards** prépare une nouvelle **Card rose**.
4. React la ramène au client.

## ✨ Étape 2 : Rendu des composants par React

👉 Une fois le rendu déclenché (étape 1), React **appelle vos fonctions composants** pour savoir quoi afficher.

***

### 🔹 Comment ça marche ?

1. **Rendu initial**
   * React appelle le **composant racine**.
   * Exemple : `root.render(<Gallery />)` → React exécute `Gallery()`.
2. **Rendus suivants (mises à jour)**
   * React appelle uniquement le **composant dont l’état a changé** (et ses enfants).
   * Si ce composant renvoie d’autres composants, React les appelle à leur tour → processus **récursif**.

***

### 🔍 Exemple

```jsx
export default function Gallery() {
  return (
    <section>
      <h1>Sculptures inspirantes</h1>
      <Image />
      <Image />
      <Image />
    </section>
  );
}

function Image() {
  return (
    <img
      src="https://i.imgur.com/ZF6s192.jpg"
      alt="Floralis Genérica' par Eduardo Catalano : une sculpture de fleur métallique gigantesque avec des pétales réfléchissants."
    />
  );
}
```

#### 🖼️ Rendu initial

* React crée le DOM pour :
  * `<section>`
  * `<h1>`
  * `<img>` (3 fois)

#### 🔄 Rendus suivants

* Si un **état change** dans `Gallery` ou `Image`, React ré-appelle uniquement ces fonctions, et **prépare les différences** avec le rendu précédent (avant de passer à l’étape 3 : commit).

***

### ⚠️ Piège : le rendu doit être PUR

Un composant React = une **fonction pure** :

* **Mêmes entrées → mêmes sorties**\
  👉 Si `Gallery` reçoit les mêmes props/états, il doit renvoyer exactement le même JSX.
* **Pas de mutation externe**\
  👉 Ne modifie pas des variables ou objets existants en dehors du composant.

💡 En **Mode Strict**, React appelle les composants **2 fois** pour vérifier cette pureté → cela permet de détecter les erreurs.

## 🚀 Étape 3 : Commit dans le DOM par React

👉 Après avoir **appelé vos composants** et produit le JSX (étape 2), React doit maintenant **synchroniser le DOM réel** avec ce JSX.\
C’est la phase de **commit**.

***

### 🔹 Ce qui se passe concrètement

#### ✅ Premier rendu (initial)

* React **crée tous les nœuds DOM** nécessaires.
* Puis il les **ajoute à la page** grâce à l’API DOM `appendChild()`.

👉 C’est l’instant où votre interface apparaît pour la première fois à l’écran.

***

#### 🔄 Rendus suivants (mises à jour)

* React compare le **nouveau rendu** avec le **rendu précédent**.
* Il applique uniquement le **strict minimum de changements** dans le DOM (→ optimisation).
* C’est ce qu’on appelle la **diffing algorithm** (ou reconciliation).

⚡ Exemple :

* Si seul le texte d’un `<h1>` change, React ne recrée pas tout le DOM.
* Il met à jour uniquement le **contenu texte du nœud `<h1>`**.
* Les autres éléments (comme un `<input>`) restent intacts.

***

### 🔍 Exemple pratique

```jsx
export default function Clock({ time }) {
  return (
    <>
      <h1>{time}</h1>
      <input />
    </>
  );
}
```

* Chaque seconde, la prop `time` change → `Clock` refait son rendu.
* React met à jour **uniquement le `<h1>`**.
* L’`<input />` n’a pas changé → React le laisse tel quel.\
  👉 Même si l’utilisateur tape du texte dedans, son contenu n’est pas effacé.

***

### ⚠️ À retenir

* **React ne touche pas au DOM s’il n’y a pas de différence.**
* Cette approche rend React **rapide et efficace** ✨.
* L’input garde sa **value** car React ne le recrée pas → il réutilise le même nœud.

## 🎨 Épilogue : dessin par le navigateur (Browser Paint)

Une fois que **React a terminé son travail de commit dans le DOM**, le navigateur prend le relais.\
👉 Il doit **rafraîchir visuellement l’écran** pour refléter les changements.

***

### 🔹 React vs Navigateur

* **React** → compare l’ancien et le nouveau JSX, met à jour le **DOM virtuel** et applique les **changements minimaux** dans le DOM réel.
* **Navigateur** → se charge ensuite de **redessiner les pixels** sur l’écran.

⚡ C’est cette étape que les développeurs appellent souvent « rendering » dans le contexte du navigateur.\
👉 Pour éviter toute confusion, dans la documentation React on parle plutôt de **painting** (dessin).

***

### 🖼️ Exemple visuel

1. React dit :
   * "Ce `<h1>` est passé de `10:00` à `10:01`."
   * "Voici le DOM mis à jour."
2. Le navigateur reçoit l’info et **peint le nouveau texte** à l’écran.

🎯 Résultat : l’utilisateur voit bien l’UI mise à jour.

***

### ⚠️ À retenir

* Le **rendu (rendering)** chez React = appel des composants → production de JSX → commit dans le DOM.
* Le **dessin (painting)** chez le navigateur = mise à jour **graphique** de l’écran.

En résumé :\
👉 React prépare et met à jour le DOM.\
👉 Le navigateur **dessine** les changements visuellement.

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
