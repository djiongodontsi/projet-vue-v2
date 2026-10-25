# 📅 Application de réservation

Mini-projet **FJSE328 – Frameworks JavaScript / TypeScript** (EPSI Paris, séance 3).
Application de réservation construite avec **Vue 3**, **Vue Router** et **Pinia**.

---

## Sommaire

1. [Fonctionnalités](#fonctionnalités)
2. [Technologies](#technologies)
3. [Prérequis](#prérequis)
4. [Installation et lancement](#installation-et-lancement)
5. [Structure du projet](#structure-du-projet)
6. [Fonctionnement détaillé](#fonctionnement-détaillé)
7. [Design et apparence](#design-et-apparence)
8. [Tests](#tests)
9. [Problèmes fréquents](#problèmes-fréquents)
10. [Pistes d'amélioration](#pistes-damélioration)

---

## Fonctionnalités

### Demandées par le sujet

| # | Consigne | Où c'est fait |
|---|----------|---------------|
| 1 | Deux pages : `/` (Réserver) et `/reservations` | `src/router.js` |
| 2 | Un store Pinia : liste, ajouter, annuler | `src/stores/reservations.js` |
| 3 | Après une réservation, aller sur la liste | `ReserverView.vue` (`router.push`) |
| 4 | L'en-tête affiche le nombre de réservations | `App.vue` (badge) |
| 5 | La date en français avec `useDate` | `src/composables/useDate.js` |
| 6 | `npm run build` puis `npm run preview` | scripts de `package.json` |

### Fonctionnalités ajoutées

- **Nombre de personnes** : chaque réservation indique le nombre de personnes (1 à 20), avec accord automatique (« 1 personne », « 2 personnes »).
- **Recherche** : un champ permet de filtrer la liste des réservations par nom, en direct.
- **Sauvegarde automatique** : les réservations sont conservées dans le `localStorage` du navigateur. Elles survivent à un rechargement de la page.
- **Validation du formulaire** : nom obligatoire, et impossible de réserver à une date passée. Un message d'erreur s'affiche en cas de problème.

---

## Technologies

| Outil | Rôle |
|-------|------|
| [Vue 3](https://vuejs.org) | Interface (composants, `<script setup>`) |
| [Vue Router](https://router.vuejs.org) | Navigation entre les pages |
| [Pinia](https://pinia.vuejs.org) | Stockage partagé des réservations |
| [Vite](https://vitejs.dev) | Serveur de développement et build |

---

## Prérequis

- [Node.js](https://nodejs.org) version 18 ou plus récente
- npm (installé avec Node.js)

Vérification :

```bash
node -v
npm -v
```

---

## Installation et lancement

Toutes les commandes se lancent **à la racine du projet** (là où se trouve `package.json`).

```bash
# 1. Installer les dépendances
npm install

# 2. Lancer en mode développement
npm run dev
```

Ouvrir ensuite l'adresse affichée dans le terminal (en général <http://localhost:5173>).
Arrêter le serveur avec `Ctrl + C`.

### Version finale (build + aperçu)

```bash
npm run build     # crée le dossier dist/
npm run preview   # sert le contenu de dist/
```

> ⚠️ `npm run preview` ne fonctionne qu'**après** `npm run build`.

### Scripts disponibles

| Commande | Description |
|----------|-------------|
| `npm run dev` | Serveur de développement avec rechargement automatique |
| `npm run build` | Compile l'application pour la production dans `dist/` |
| `npm run preview` | Lance un serveur local pour tester le build |

---

## Structure du projet

```
NOM1_NOM2/
├── index.html                  Page HTML de départ
├── package.json                Dépendances et scripts
├── vite.config.js              Configuration de Vite
├── tests.md                    Liste des tests manuels
├── README.md                   Ce fichier
└── src/
    ├── main.js                 Démarrage de l'app + sauvegarde localStorage
    ├── router.js               Définition des deux pages
    ├── style.css               Styles globaux (couleurs, boutons, cartes…)
    ├── App.vue                 En-tête commun + zone d'affichage des pages
    ├── stores/
    │   └── reservations.js     Store Pinia (liste, ajouter, annuler)
    ├── composables/
    │   └── useDate.js          Formatage des dates en français
    └── views/
        ├── ReserverView.vue    Page « Réserver » (formulaire)
        └── ReservationsView.vue  Page « Mes réservations » (liste)
```

---

## Fonctionnement détaillé

### 1. Démarrage (`main.js`)

Crée l'application Vue, branche Pinia (les données) et le routeur (les pages), importe `style.css`, puis l'affiche dans `<div id="app">` de `index.html`.

C'est aussi ici que la **sauvegarde automatique** est branchée : `store.$subscribe` s'exécute à chaque modification du store et écrit son contenu dans le `localStorage`.

### 2. Routeur (`router.js`)

Associe une adresse à un composant :

| Adresse | Page |
|---------|------|
| `/` | `ReserverView` |
| `/reservations` | `ReservationsView` |

Le composant `<RouterView />` de `App.vue` affiche la page correspondant à l'adresse.

### 3. Store Pinia (`stores/reservations.js`)

Le store est la **mémoire commune** de l'application, partagée entre toutes les pages.

- **State** : `liste` (tableau des réservations) et `prochainId` (compteur pour donner un identifiant unique à chaque réservation).
- **Getter** : `nombre`, la taille de la liste (utilisé par l'en-tête).
- **Actions** :
  - `ajouter(nom, date, personnes)` : ajoute une réservation à la liste.
  - `annuler(id)` : retire la réservation correspondante.

Au démarrage, le state est lu depuis le `localStorage` (fonction `charger`). Si rien n'est sauvegardé ou si les données sont illisibles, le store démarre avec une liste vide.

Une réservation ressemble à ceci :

```js
{ id: 1, nom: 'Dupont', date: '2026-10-25', personnes: 2 }
```

### 4. En-tête (`App.vue`)

Affiche le logo et deux liens. Le lien « Mes réservations » contient un badge qui lit `store.nombre` : il se met à jour **tout seul** à chaque ajout ou annulation. Le lien de la page courante est mis en évidence (classe `router-link-exact-active`).

### 5. Page Réserver (`ReserverView.vue`)

1. L'utilisateur saisit un nom, une date et un nombre de personnes (champs liés avec `v-model`).
2. Au clic sur « Réserver », la fonction `reserver()` vérifie que :
   - le nom n'est pas vide ;
   - la date est renseignée et n'est pas dans le passé.
3. Si tout est correct, elle appelle `store.ajouter(...)` puis `router.push('/reservations')` pour afficher la liste.
4. Sinon, un message d'erreur rouge s'affiche.

L'attribut `:min="aujourdhui"` du champ date empêche aussi de choisir une date passée dans le sélecteur.

### 6. Page Mes réservations (`ReservationsView.vue`)

- S'il n'y a aucune réservation, un message l'indique.
- Sinon, un champ de recherche apparaît. La liste affichée (`resultats`) est une propriété `computed` qui garde uniquement les réservations dont le nom contient le texte saisi (sans tenir compte des majuscules).
- Chaque réservation affiche le nom, la date en français, le nombre de personnes et un bouton « Annuler » (`store.annuler(r.id)`).

### 7. Dates en français (`composables/useDate.js`)

Le composable `useDate()` renvoie une fonction `formater(date)` qui transforme `2026-10-25` en **« dimanche 25 octobre 2026 »** grâce à `toLocaleDateString('fr-FR', …)`.

---

## Design et apparence

Le style est centralisé dans `src/style.css` et repose sur des **variables CSS** (`--vert`, `--rouge`, `--bord`…) définies en haut du fichier : changer une couleur à cet endroit la change dans toute l'application.

- **Palette** : vert (identité du projet), gris doux pour le fond, rouge pour les actions de suppression.
- **En-tête** : bandeau vert avec navigation en « pilules » ; la page active est surlignée en blanc.
- **Cartes** : formulaire et réservations présentés dans des blocs blancs arrondis avec ombre légère.
- **Boutons** : effet au survol et léger enfoncement au clic ; le bouton « Annuler » est en contour rouge pour éviter les clics par erreur.
- **Formulaires** : contour vert au focus.
- **Responsive** : le contenu est centré (720 px max) et les champs passent sur une seule colonne sur petit écran.

---

## Tests

La liste des tests manuels est dans [`tests.md`](./tests.md). Elle couvre :

- l'affichage des deux pages ;
- la validation du formulaire ;
- l'ajout, l'annulation et la redirection ;
- le compteur de l'en-tête ;
- la date en français ;
- la recherche ;
- la persistance après rechargement ;
- le build et le preview.

---

## Problèmes fréquents

| Problème | Cause | Solution |
|----------|-------|----------|
| `'vite' is not recognized` | Dépendances non installées, ou mauvais dossier | Aller à la racine du projet, puis `npm install` |
| `The directory "dist" does not exist` | `preview` lancé sans build | Lancer `npm run build` d'abord |
| `npm ERR! enoent package.json` | Terminal dans le mauvais dossier | `cd` vers le dossier contenant `package.json` |
| Page blanche après `preview`, ou erreur 404 en rechargeant `/reservations` | Comportement possible selon l'hébergement | Revenir à `/` et naviguer avec les liens |
| Les réservations ont disparu | `localStorage` vidé ou autre navigateur | Normal : les données sont propres à chaque navigateur |

---

## Pistes d'amélioration

- Page `/salles` avec choix d'une salle (bonus du sujet).
- Mise en ligne sur Netlify (bonus du sujet).
- Modification d'une réservation existante.
- Tri des réservations par date.
- Migration vers TypeScript.

---

## Auteurs

NOM1 et NOM2 — Bachelor EPSI Paris.
