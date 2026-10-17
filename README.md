# MyFood — commande de table pour restaurants (PWA)

Application complète : espace client (menu, panier, suivi de commande, avis) + tableau de bord
restaurateur (commandes en direct, gestion de carte, statistiques, profil, QR codes), avec un
vrai backend et des mises à jour en temps réel.

## Stack technique

- **Backend** : Node.js + Express + Socket.io + SQLite (module natif `node:sqlite`, aucune
  compilation native requise)
- **Frontend** : HTML/CSS/JS modulaire, servi comme PWA installable (manifest + service worker)
- **Temps réel** : quand un client envoie une commande, elle apparaît instantanément dans le
  dashboard (et inversement : le client voit son statut évoluer en direct)

## Structure du projet

```
myfood-pwa/
├── backend/
│   ├── server.js              # serveur Express + Socket.io
│   ├── package.json
│   ├── db/
│   │   ├── database.js        # schéma SQLite
│   │   └── seed.js            # données de démonstration au premier lancement
│   └── routes/
│       ├── menu.js            # CRUD carte (menu / boissons / autre)
│       ├── orders.js          # création + suivi des commandes
│       ├── reviews.js         # avis clients
│       ├── restaurant.js      # profil de l'établissement
│       ├── events.js          # annonces (promos, recrutement...)
│       ├── alerts.js          # alertes dashboard
│       ├── tables.js          # tables + génération de QR codes
│       └── stats.js           # statistiques jour / semaine / mois
└── public/                    # tout ce qui est servi au navigateur (= le PWA)
    ├── index.html              # espace client
    ├── dashboard.html          # tableau de bord restaurateur
    ├── manifest.json
    ├── sw.js                   # service worker (hors-ligne)
    ├── css/ (tokens, client, dashboard)
    ├── js/  (api.js partagé, client.js, dashboard.js)
    └── icons/                  # icônes PWA générées depuis votre logo
```

## Installation et lancement

```bash
cd myfood-pwa/backend
npm install
npm start
```

- Espace client : http://localhost:3000
- Tableau de bord : http://localhost:3000/dashboard.html

Au premier lancement, une base SQLite est créée dans `backend/data/myfood.db` avec un profil
restaurant, une carte et 12 tables de démonstration (`db/seed.js`). Vous pouvez tout modifier
depuis le dashboard ensuite — rien n'est codé en dur côté frontend.

## Fonctionnalités livrées

**Espace client**
- Saisie du numéro de table (ou pré-remplissage automatique via QR code scanné)
- Menu / Boissons / Autre avec photos, prix, ajout au panier
- Panier avec note pour la cuisine, envoi de commande
- Suivi de commande en direct (Reçue → En préparation → Servie)
- Avis avec notation en étoiles, visible en fil d'actualité

**Tableau de bord**
- Commandes en direct façon ticket de cuisine, changement de statut en un clic
- Gestion de la carte : ajout/retrait d'articles avec photo, nom, prix, description, disponibilité
- Alertes en temps réel (nouvelles commandes, avis, etc.)
- Statistiques jour/semaine/mois calculées à partir des vraies commandes enregistrées
- Profil du restaurant (nom, logo, contacts, réseaux, localisation)
- Événements spéciaux (bonus, réductions, recrutement)
- Génération de QR codes par table, prêts à imprimer (SVG), qui ouvrent le menu avec la table
  pré-remplie

## Installer l'app (PWA)

Depuis Chrome/Edge (Android ou desktop) ou Safari (iOS), ouvrez l'URL du site puis
« Ajouter à l'écran d'accueil » / « Installer l'application ». Le service worker met en cache
l'interface pour un chargement instantané et une utilisation partielle hors-ligne (les données
en direct — commandes, menu à jour — nécessitent une connexion).

## À faire avant une mise en production réelle

Ce projet est fonctionnel de bout en bout, mais certains points méritent votre attention avant
d'ouvrir l'accès au public :

1. **Authentification du dashboard.** Il n'y a aujourd'hui aucune protection sur
   `/dashboard.html` ni sur les routes `/api/*` d'administration (ajout de plats, changement de
   statut, etc.) — quiconque connaît l'URL peut y accéder. Pour un vrai déploiement, ajoutez une
   authentification (ex. compte restaurateur + session ou JWT) avant les routes sensibles dans
   `server.js`.
2. **HTTPS.** Le service worker et l'installation PWA exigent HTTPS en production (localhost est
   exempté pour le développement). Déployez derrière un reverse proxy (Nginx, Caddy) ou une
   plateforme qui fournit un certificat TLS automatiquement.
3. **Hébergement des images.** Les photos de plats sont actuellement des URLs externes. Pour la
   production, prévoyez un vrai upload de fichiers (ex. stockage local `/public/uploads` ou un
   service cloud) plutôt que de coller des liens.
4. **Sauvegardes.** La base SQLite vit dans `backend/data/myfood.db` — pensez à l'inclure dans
   vos sauvegardes automatiques.
5. **Multi-restaurants.** La structure actuelle gère un seul restaurant (table `restaurant` à une
   seule ligne). Pour proposer MyFood à plusieurs établissements, il faudra introduire une notion
   de compte/tenant et filtrer chaque requête par restaurant.

## Régénérer les icônes si le logo change

```bash
cd myfood-pwa
python3 -c "
from PIL import Image
src = Image.open('public/img/logo-source.png').convert('RGBA')
for s in [72,96,128,144,152,192,384,512]:
    src.resize((s,s), Image.LANCZOS).save(f'public/icons/icon-{s}.png')
"
```
(voir aussi les variantes *maskable* et *apple-touch-icon* générées de la même façon dans
l'historique de ce projet)
[README.md](https://github.com/user-attachments/files/33590787/README.md)
# myfood-pwa
