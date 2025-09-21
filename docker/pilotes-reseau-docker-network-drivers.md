---
description: >-
  Le sous-système réseau de Docker est modulaire et extensible grâce à des
  pilotes réseau. Plusieurs pilotes existent par défaut et assurent les
  fonctionnalités de base.
---

# 🌐 Pilotes réseau Docker (Network drivers)

### 🔹 1. **bridge** (par défaut)

* C’est le **pilote par défaut** quand tu crées un réseau sans préciser de driver.
* Utilisé lorsque des conteneurs doivent communiquer **entre eux sur la même machine hôte**.
* Exemple : tu lances une application multi-conteneurs (API + base de données) sur un seul serveur.

👉 Voir : \[Bridge network driver]

***

### 🔹 2. **host**

* Supprime l’isolation réseau entre le conteneur et l’hôte.
* Le conteneur utilise **directement la pile réseau de l’hôte**.
* Pas d’adresse IP distincte pour le conteneur → il partage celles de l’hôte.

👉 Utile pour les applications réseau nécessitant des performances maximales.\
👉 Voir : \[Host network driver]

***

### 🔹 3. **overlay**

* Connecte plusieurs démons Docker (sur différents hôtes) via un **réseau distribué**.
* Permet aux services Swarm et aux conteneurs de communiquer **à travers plusieurs machines**.
* Évite de configurer du routage OS complexe.

👉 Parfait pour un cluster **Docker Swarm**.\
👉 Voir : \[Overlay network driver]

***

### 🔹 4. **ipvlan**

* Donne un **contrôle total des adresses IPv4 et IPv6** attribuées aux conteneurs.
* Peut fonctionner avec des **VLANs** (niveau 2) et même faire du **routage L3**.
* Idéal pour une intégration directe avec le **réseau physique sous-jacent (underlay)**.

👉 Voir : \[IPvlan network driver]

***

### 🔹 5. **macvlan**

* Attribue une **adresse MAC unique** à chaque conteneur → il apparaît comme un vrai appareil physique sur le réseau.
* Le trafic est routé directement via son adresse MAC.
* Recommandé pour les **applications legacy** qui s’attendent à être connectées directement au réseau, sans NAT ni translation.

👉 Voir : \[Macvlan network driver]

***

### 🔹 6. **none**

* Retire complètement l’accès réseau du conteneur.
* Pas d’interface réseau, pas de communication avec l’hôte ou les autres conteneurs.
* Utile pour des traitements totalement **isolés**.

👉 ⚠️ Non disponible pour les services Swarm.\
👉 Voir : \[None network driver]

***

### 🔹 7. **Plugins réseau tiers**

* Tu peux installer des **plugins externes** pour intégrer Docker avec des stacks réseau spécialisées (ex. Calico, Weave, Cilium…).
* Permet d’étendre Docker avec des fonctionnalités avancées (sécurité, overlay multi-datacenter, etc.).

***

## 📋 Résumé des choix de pilotes

* **bridge (par défaut)** → bon pour les conteneurs simples sur la même machine.
* **bridge défini par l’utilisateur** → crée un réseau isolé entre plusieurs conteneurs d’un projet.
* **host** → conteneur partage la pile réseau de l’hôte (pas d’isolation).
* **overlay** → communication entre conteneurs sur plusieurs hôtes (Swarm).
* **macvlan** → chaque conteneur a sa propre **MAC unique**, comme un vrai serveur physique.
* **ipvlan** → similaire à macvlan mais sans attribuer de MAC unique (utile si limitation d’adresses MAC).
* **none** → pas de réseau du tout.
* **plugins tiers** → pour intégrer des solutions réseau externes avancées.
