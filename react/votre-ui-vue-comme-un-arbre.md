---
description: >-
  Votre appli React prend forme, et de nombreux composants sont imbriqués les
  uns dans les autres. Comment React garde-t-il trace de la structure de
  composants de votre appli ?
---

# 🌳 Votre UI vue comme un arbre

React, comme de nombreuses autres bibliothèques d’UI, modélise l’UI comme un **arbre**. Penser à votre appli comme à un arbre s’avère très utile pour comprendre les relations entre les composants. Cela vous aidera également à déboguer plus facilement des problématiques liées à l’**optimisation des performances** ou à la **gestion d’état**.

***

### 📚 Vous allez apprendre

* 🌐 Comment React « voit » les structures de composants
* 🌲 Ce qu’est un **arbre de rendu**, et en quoi il est utile
* 🔗 Ce qu’est un **arbre de dépendances de modules**, et à quoi il sert

## 🌳 Votre UI vue comme un arbre

**Les arbres sont un modèle relationnel entre des éléments, et l’UI est souvent représentée au moyen de structures arborescentes.**\
👉 Les navigateurs utilisent par exemple des arbres pour modéliser **HTML (le DOM)** et **CSS (le CSSOM)**.\
👉 Les plateformes mobiles utilisent aussi des arbres pour représenter leurs **hiérarchies de vues**.

***

### 🔹 De vos composants au DOM

<figure><img src="../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

💡 **React crée un arbre d’UI à partir de vos composants.** Dans cet exemple, cet arbre sert à produire le **DOM**.

Comme les navigateurs et plateformes mobiles, React utilise des structures arborescentes pour :

* gérer les relations entre composants,
* comprendre la circulation des données,
* optimiser le **rendu** et la **taille du code**.

***

### 🔹 L’arbre de rendu

Un **composant React** peut contenir d’autres composants : on parle de **composition**.

* Chaque **parent** peut avoir des **enfants**,
* et chaque parent est lui-même l’enfant d’un autre.

👉 Lorsque React fait le rendu de votre appli, il construit un **arbre de rendu**.

#### Exemple : une appli de citations

📌 **Code :**

```jsx
import FancyText from './FancyText';
import InspirationGenerator from './InspirationGenerator';
import Copyright from './Copyright';

export default function App() {
  return (
    <>
      <FancyText title text="Une appli pour être inspiré·e" />
      <InspirationGenerator>
        <Copyright year={2004} />
      </InspirationGenerator>
    </>
  );
}
```

<figure><img src="../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

⚡ Chaque nœud représente un **composant React**. Le nœud racine est le **composant racine de l’appli** (ici `App`).

***

### 🔹 Où sont passées les balises HTML ?

📌 **Important :**\
L’arbre de rendu **ne contient pas les balises HTML** (`<div>`, `<h1>`, etc.).

Pourquoi ?

* Parce que React est **indépendant de la plateforme**.
* Sur le web → React traduit en HTML.
* Sur mobile → React traduit en UIView (iOS) ou FrameworkElement (Windows).

👉 L’arbre de rendu montre **uniquement les composants React**, quelle que soit la plateforme cible.

***

### 🔹 Rendu conditionnel et variation des arbres

Un arbre de rendu correspond à **une seule passe de rendu**.\
Avec du **JSX conditionnel**, l’arbre peut changer d’un rendu à l’autre.

📌 **Code conditionnel simplifié :**

```jsx
<InspirationGenerator>
  {inspiration.type === "text" ? <FancyText /> : <Color />}
  <Copyright year={2004} />
</InspirationGenerator>
```

<figure><img src="../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

⚡ Selon `inspiration.type`, l’arbre varie !

***

### 🔹 Pourquoi identifier l’arbre est utile ?

* Les **composants de haut niveau** (près de la racine) impactent les performances de toute la sous-arborescence.
* Les **composants feuilles** (sans enfants) se rerendent fréquemment.

👉 Bien comprendre l’arbre aide à :

* analyser la **circulation des données**,
* identifier les **points critiques de performance**.

## 🌐 L’arbre de dépendances de modules

**Les arbres ne servent pas qu’à représenter les composants : ils peuvent aussi modéliser les dépendances entre modules JavaScript.**

👉 Lorsque nous découpons notre code en plusieurs fichiers (composants, fonctions utilitaires, constantes, etc.), nous créons un réseau de **modules interconnectés**.\
👉 Chaque **nœud** représente un module, et chaque **branche** représente une instruction `import`.

***

### 🔹 Exemple : l’appli _Inspirations_

<figure><img src="../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

* Racine : `App.js`
  * importe `InspirationGenerator.js`
  * importe `FancyText.js`
  * importe `Copyright.js`
* `InspirationGenerator.js` importe aussi :
  * `FancyText.js`
  * `Color.js`
  * `inspirations.js`

👉 **Image/Diagramme :** 7 nœuds reliés, avec les flèches libellées « importe ».

***

### 🔹 Différences avec l’arbre de rendu

Bien que similaires dans leur structure, l’arbre de dépendances et l’arbre de rendu ne modélisent pas la même chose :

1. **Nature des nœuds :**
   * Arbre de rendu → **composants React**
   * Arbre de dépendances → **modules JavaScript**
2. **Modules sans composants :**
   * Ex : `inspirations.js` (contient seulement des données ou des fonctions).
   * → Visible dans l’arbre de dépendances ✅
   * → Invisible dans l’arbre de rendu ❌
3. **Relation import vs. rendu :**
   * Exemple : `Copyright.js` est importé par `App.js` → donc il est directement sous `App.js` dans l’arbre de dépendances.
   * Mais dans l’arbre de rendu, `Copyright` est affiché **à l’intérieur de `InspirationGenerator`** (car passé via `children`).

***

### 🔹 Utilité de l’arbre de dépendances

* Les **bundlers** (ex. Webpack, Vite, Parcel, Rollup) analysent cet arbre pour déterminer **quels modules inclure** dans le **bundle final**.
* Plus votre app grandit → plus le bundle grossit.
* ⚠️ **Un bundle trop massif =**
  * plus long à télécharger,
  * plus lent à exécuter,
  * UI qui s’affiche en retard.

👉 **Avoir une bonne visibilité de l’arbre de dépendances permet de :**

* comprendre **de quoi dépend votre app**,
* déboguer les problèmes liés à la **taille du bundle**,
* appliquer des optimisations (code splitting, lazy loading, etc.).
