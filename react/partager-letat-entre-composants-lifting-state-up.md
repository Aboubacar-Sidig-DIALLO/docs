# 🔄 Partager l’état entre composants (Lifting State Up)

Il arrive souvent que **deux composants doivent rester synchronisés**.\
👉 Exemple : deux panneaux dont un seul doit être actif à la fois.

➡️ La solution : **supprimer l’état local des deux composants enfants**, le **remonter au parent commun**, et le transmettre via des **props**.\
C’est ce qu’on appelle **lifting state up** (_remonter l’état_).

***

### 🎯 Ce que tu vas apprendre

* Comment partager l’état entre composants en le remontant au parent.
* La différence entre **composants contrôlés** et **non contrôlés**.

***

## 🔼 Lifting State Up par l’exemple

Ici, un parent `Accordion` rend deux composants enfants `Panel` :

```
Accordion
 ├─ Panel
 └─ Panel
```

Chaque `Panel` a un état local `isActive` qui détermine si son contenu est visible.

***

### Exemple de départ : chaque `Panel` gère son propre état

```jsx
import { useState } from 'react';

function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          Show
        </button>
      )}
    </section>
  );
}

export default function Accordion() {
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="About">
        With a population of about 2 million, Almaty is Kazakhstan's largest city. From 1929 to 1997, it was its capital city.
      </Panel>
      <Panel title="Etymology">
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for "apple" and is often translated as "full of apples". In fact, the region surrounding Almaty is thought to be the ancestral home of the apple, and the wild <i lang="la">Malus sieversii</i> is considered a likely candidate for the ancestor of the modern domestic apple.
      </Panel>
    </>
  );
}
```

***

<figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

1️⃣ État initial\
Diagramme montrant un arbre de trois composants : un parent `Accordion` et deux enfants `Panel`.\
Chaque `Panel` contient `isActive = false`.\
➡️ Résultat : les deux panneaux sont **repliés**.

***

2️⃣ Après un clic sur le premier bouton\
Même diagramme, mais le `Panel` de gauche a `isActive = true` (contenu visible), et celui de droite a encore `isActive = false`.\
➡️ Résultat : seul le premier panneau s’ouvre.

***

⚠️ Problème : les deux `Panel` sont **indépendants**.\
Donc si tu cliques sur les deux boutons, **les deux panneaux peuvent rester ouverts en même temps**.

***

## 🎯 Objectif

On veut que **seul un panneau soit ouvert à la fois**.\
👉 Si tu ouvres le deuxième, le premier doit se fermer automatiquement.

## 🛠️ Solution : Remonter l’état

Pour coordonner les deux panneaux :

#### Étape 1 : Supprimer l’état des enfants

Les `Panel` ne doivent plus avoir de `useState`.

#### Étape 2 : Passer des données codées en dur depuis le parent

Exemple : `<Panel isActive={true} />`.

#### Étape 3 : Ajouter un état commun dans `Accordion`

Le parent `Accordion` gère l’index du panneau actif (`activeIndex`), et passe les props `isActive` + `onShow` à ses enfants.

***

💡 Résultat final :

* `Accordion` possède **un seul état partagé**.
* Les `Panel` deviennent des composants **contrôlés**.
* Un seul panneau peut être ouvert en même temps.

### 🔹 Étape 1 : Retirer l’état local du composant enfant

Avant, ton `Panel` gérait son état avec :

```jsx
const [isActive, setIsActive] = useState(false);
```

➡️ Cela rendait chaque panneau **indépendant** et incontrôlable depuis le parent.

***

#### 🔄 Après modification

On supprime `useState` dans `Panel`, et on fait en sorte que le parent décide de la valeur :

```jsx
function Panel({ title, children, isActive }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button>
          Show
        </button>
      )}
    </section>
  );
}
```

***

* Avant :\
  Chaque `Panel` avait son propre `isActive = false`.\
  👉 Chaque bouton affectait uniquement son panneau.
* Après :\
  `isActive` n’existe plus dans `Panel`, il est **contrôlé depuis le parent `Accordion`**.\
  👉 Les `Panel` deviennent **dépendants de props**.

## ⚡ Étape 2 : Passer des données codées en dur depuis le parent commun

On a identifié que le parent **`Accordion`** est le plus proche parent commun de `Panel`.\
➡️ Il va donc devenir la **source de vérité** pour décider quel `Panel` est actif.

Voici la version avec **valeurs codées en dur** :

```jsx
import { useState } from 'react';

export default function Accordion() {
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="About" isActive={true}>
        With a population of about 2 million, Almaty is Kazakhstan's largest city. 
        From 1929 to 1997, it was its capital city.
      </Panel>
      <Panel title="Etymology" isActive={false}>
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for "apple" 
        and is often translated as "full of apples". In fact, the region surrounding Almaty 
        is thought to be the ancestral home of the apple, and the wild 
        <i lang="la">Malus sieversii</i> is considered a likely candidate for the ancestor 
        of the modern domestic apple.
      </Panel>
    </>
  );
}

function Panel({ title, children, isActive }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button>Show</button>
      )}
    </section>
  );
}
```

***

* `Accordion` décide :
  * `isActive={true}` pour le premier panel
  * `isActive={false}` pour le second

```
Accordion
 ├─ Panel 1 → isActive = true
 └─ Panel 2 → isActive = false
```

➡️ En changeant la valeur codée (`true/false`), on contrôle directement quel panel s’affiche.

***

## ⚡ Étape 3 : Ajouter l’état dans le parent commun

Plutôt que de coder en dur, on va maintenant stocker dans `Accordion` **quel panel est actif**.\
On utilise un `state` :

```jsx
const [activeIndex, setActiveIndex] = useState(0);
```

👉 Quand `activeIndex === 0`, le premier panel est actif.\
👉 Quand `activeIndex === 1`, le deuxième panel est actif.

On passe aussi une fonction `onShow` à chaque `Panel` pour que le clic du bouton **change l’état du parent**.

***

#### Code complet

```jsx
import { useState } from 'react';

export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);

  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel
        title="About"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        With a population of about 2 million, Almaty is Kazakhstan's largest city. 
        From 1929 to 1997, it was its capital city.
      </Panel>
      <Panel
        title="Etymology"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for "apple" 
        and is often translated as "full of apples". In fact, the region surrounding Almaty 
        is thought to be the ancestral home of the apple, and the wild 
        <i lang="la">Malus sieversii</i> is considered a likely candidate for the ancestor 
        of the modern domestic apple.
      </Panel>
    </>
  );
}

function Panel({ title, children, isActive, onShow }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>Show</button>
      )}
    </section>
  );
}
```

***

<figure><img src="../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

#### 1. État initial

* `activeIndex = 0` dans `Accordion`.
* Le premier `Panel` reçoit `isActive = true`.
* Le deuxième reçoit `isActive = false`.

```
Accordion (activeIndex = 0)
 ├─ Panel 1 → isActive = true
 └─ Panel 2 → isActive = false
```

***

#### 2. Après clic sur le bouton du 2ᵉ panel

* `setActiveIndex(1)` met à jour l’état.
* `Accordion` re-render.
* Maintenant :
  * `Panel 1 → isActive = false`
  * `Panel 2 → isActive = true`

```
Accordion (activeIndex = 1)
 ├─ Panel 1 → isActive = false
 └─ Panel 2 → isActive = true
```

***

👉 Résultat : **un seul panel est ouvert à la fois**, et tout est synchronisé grâce à l’état dans le parent.

## 🎯 Une seule source de vérité pour chaque état

Dans une application React, **plusieurs composants** vont avoir leur propre état (`state`).

* Certains états vont **vivre près des feuilles** (les composants les plus bas dans l’arbre), par exemple la valeur d’un champ de formulaire.
* D’autres états vont **vivre plus haut dans l’arborescence**, par exemple l’état de la route courante (chemin de navigation), souvent géré par une librairie de routing côté client.

👉 Le principe important est que pour **chaque morceau unique d’état**, il doit exister **un seul endroit où il est défini et stocké**.\
C’est ce qu’on appelle une **single source of truth** (_une seule source de vérité_).

***

### 📌 Que signifie “une seule source de vérité” ?

* Cela **ne veut pas dire** que tout l’état doit être centralisé dans un seul composant.
* Cela signifie que pour **chaque donnée particulière**, il faut déterminer **quel composant en est le propriétaire** (c’est-à-dire celui qui détient la version “officielle” de cette donnée).

➡️ Ensuite, si plusieurs composants ont besoin de cette information, tu **l’élèves** (_lift state up_) jusqu’à leur parent commun.\
➡️ Ce parent devient alors la **source de vérité** et transmet cette donnée via des **props** aux enfants qui en ont besoin.

***

### ⚡ Exemple simple

Imaginons deux composants `Panel` qui doivent partager le même état d’expansion.

❌ Mauvaise approche :\
Chaque `Panel` a son propre `isActive`. Résultat → incohérences possibles, car deux panels peuvent s’ouvrir indépendamment.

✅ Bonne approche :\
L’état `activeIndex` vit dans le parent `Accordion`.

* `Accordion` est la **single source of truth**.
* Il transmet à chaque `Panel` via `props` si celui-ci est actif ou non.
* Les enfants notifient le parent avec un callback (ex: `onShow`) pour modifier cet état.

***

### 🔄 Déplacement de l’état pendant le développement

Quand tu construis ton application, il est **normal** de déplacer un état :

* D’abord tu peux le mettre dans un composant localement.
* Puis, si tu réalises que plusieurs composants en ont besoin, tu l’**élèves** plus haut.
* Parfois même tu peux redescendre un état si tu t’aperçois qu’il n’était pas nécessaire de le partager.

👉 C’est un processus naturel dans le développement React.

***

### ✅ Récapitulatif

1. **Un seul propriétaire par état** → une _single source of truth_.
2. Si deux composants doivent être coordonnés → élève l’état vers leur parent commun.
3. Passe les données **du parent vers les enfants** via `props`.
4. Passe aussi des **event handlers** aux enfants pour qu’ils puissent informer le parent de changements.
5. Considère si un composant doit être :
   * **Contrôlé** → piloté par les `props` (plus flexible).
   * **Non contrôlé** → gère son propre `state` local (plus simple à utiliser).

***

⚡ Exemple concret à lire ensuite : 👉 **Thinking in React**, qui montre comment appliquer ces principes pour construire une application étape par étape.
