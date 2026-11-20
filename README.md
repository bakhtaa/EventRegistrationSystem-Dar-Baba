# Event Registration System

Application web d'inscription à des événements construite avec une
architecture **MERN-lite**. Un participant choisit un événement,
renseigne ses coordonnées et soumet le formulaire. Le serveur enregistre
l'inscription dans MongoDB, génère un QR code et un PDF de confirmation,
puis envoie ce PDF par email.

> **État du projet : MVP fonctionnel.** Le dépôt implémente actuellement
> un parcours public d'inscription avec confirmation automatisée. Il ne
> constitue pas encore une plateforme complète de gestion d'événements :
> il n'inclut notamment ni espace administrateur, ni comptes
> utilisateurs, ni paiement, ni gestion du cycle de vie des
> inscriptions.

------------------------------------------------------------------------

## Sommaire

-   [Fonctionnalités disponibles](#fonctionnalités-disponibles)
-   [Événements proposés](#événements-proposés)
-   [Déroulement d'une inscription](#déroulement-dune-inscription)
-   [Architecture](#architecture)
-   [Technologies](#technologies)
-   [API](#api)
-   [Configuration](#configuration)
-   [Installation et lancement](#installation-et-lancement)
-   [Déploiement actuel](#déploiement-actuel)
-   [Limites connues et points de
    vigilance](#limites-connues-et-points-de-vigilance)
-   [Évolutions envisagées](#évolutions-envisagées)

## Fonctionnalités disponibles

-   Affichage et sélection parmi trois événements prédéfinis.
-   Formulaire public recueillant le nom, l'adresse email et le numéro
    de téléphone du participant.
-   Association automatique du lieu et de l'horaire à l'événement
    sélectionné.
-   Validation des champs dans l'interface React :
    -   champs obligatoires ;
    -   format de l'adresse email ;
    -   numéro de téléphone composé exactement de 10 chiffres.
-   Création d'un identifiant d'inscription côté serveur.
-   Enregistrement des informations de l'inscription dans MongoDB avec
    Mongoose.
-   Génération d'un QR code contenant les informations de l'inscription.
-   Génération d'un PDF A4 côté serveur avec Puppeteer.
-   Envoi d'un email de confirmation contenant le PDF `registration.pdf`
    en pièce jointe.
-   Affichage d'un message de confirmation après une réponse positive du
    serveur.

## Événements proposés

Les événements sont actuellement prédéfinis dans `EventForm.jsx`.

  -----------------------------------------------------------------------
  Événement               Lieu                    Horaire
  ----------------------- ----------------------- -----------------------
  AI Effect On Job        Hall A, Hyderabad       10:00 AM -- 1:00 PM
                          Convention Center       

  Startup Expo            Mudra Banquet,          2:00 PM -- 6:00 PM
                          Hyderabad               

  Google Summit           Hitech-City, Hyderabad  8:00 PM -- 11:00 PM
  -----------------------------------------------------------------------

Le lieu et l'horaire sont associés automatiquement à l'événement choisi.
Ils ne sont pas encore gérés depuis une interface d'administration.

## Déroulement d'une inscription

1.  Le participant sélectionne un événement.
2.  Il renseigne son nom, son adresse email et son numéro de téléphone.
3.  Le frontend effectue une première validation des champs.
4.  React envoie les données au backend via `POST /submit`.
5.  Le backend génère un identifiant d'inscription aléatoire de six
    chiffres.
6.  Le serveur génère un QR code à partir des informations de
    l'inscription.
7.  L'inscription est enregistrée dans MongoDB.
8.  Le backend construit une page HTML et la convertit en PDF A4 avec
    Puppeteer.
9.  Nodemailer envoie un email au participant avec le PDF en pièce
    jointe.
10. Le frontend affiche le message de confirmation après la réponse
    positive du serveur.

### Contenu du QR code et du PDF

Le QR code contient actuellement des données JSON relatives à
l'inscription :

-   nom du participant ;
-   adresse email ;
-   numéro de téléphone ;
-   événement ;
-   lieu ;
-   horaire ;
-   identifiant d'inscription.

Le PDF de confirmation reprend les informations du participant et de
l'événement, l'identifiant d'inscription et le QR code.

**Attention :** le QR code contient des données personnelles. Il ne
possède actuellement ni signature cryptographique ni mécanisme
centralisé permettant de vérifier son authenticité.

## Architecture

Le projet est organisé en deux parties principales :

``` text
project/
├── client/
│   ├── Interface React avec Vite
│   ├── Styles Tailwind CSS
│   ├── Appels HTTP avec Axios
│   ├── Validation du formulaire
│   └── Sélection des événements
│
└── server/
    ├── API Express
    ├── Connexion MongoDB avec Mongoose
    ├── Modèle Submission.js
    ├── Génération du QR code
    ├── Génération du PDF avec Puppeteer
    └── Envoi des emails avec Nodemailer
```

### Responsabilités

**Frontend (`client/`)** - Afficher le formulaire d'inscription. -
Proposer les événements prédéfinis. - Valider les champs avant
l'envoi. - Envoyer les données au backend avec Axios. - Afficher le
résultat de la soumission.

Le frontend contient également une implémentation de génération PDF avec
`html2canvas` et `jsPDF`. Cette génération n'est toutefois pas utilisée
dans le workflow actif : le PDF envoyé au participant est produit par le
backend.

**Backend (`server/`)** - Exposer l'API Express. - Se connecter à
MongoDB au démarrage. - Générer l'identifiant d'inscription utilisé par
le workflow actif. - Générer le QR code. - Enregistrer l'inscription. -
Générer le PDF avec Puppeteer. - Envoyer le PDF par email avec
Nodemailer.

## Technologies

### Frontend

-   React 19
-   Vite
-   Tailwind CSS
-   Axios
-   html2canvas
-   jsPDF
-   UUID

### Backend

-   Node.js
-   Express 5
-   MongoDB
-   Mongoose
-   qrcode
-   Puppeteer
-   Nodemailer
-   dotenv
-   CORS

## API

### `POST /submit`

Endpoint utilisé pour soumettre une inscription. Il est exposé
directement par le serveur principal dans `index.js`.

**Corps de la requête :**

``` json
{
  "name": "Nom",
  "email": "email@exemple.com",
  "phone": "1234567890",
  "eventName": "AI Effect On Job",
  "eventVenue": "Hall A, Hyderabad Convention Center",
  "eventTime": "10:00 AM - 1:00 PM"
}
```

**Réponse en cas de succès :**

``` json
{
  "message": "Form submitted successfully!"
}
```

L'identifiant généré par le serveur est un nombre aléatoire de six
chiffres, compris entre `100000` et `999999`.

> La validation côté serveur des champs obligatoires et de leur format
> doit encore être renforcée. La validation effectuée dans le navigateur
> ne suffit pas à protéger l'API contre les requêtes HTTP directes.

## Configuration

Le backend utilise les variables d'environnement suivantes :

  -----------------------------------------------------------------------
  Variable                            Description
  ----------------------------------- -----------------------------------
  `MONGODB_URI`                       URI de connexion à MongoDB

  `PORT`                              Port d'écoute du serveur

  `EMAIL_USER`                        Adresse du compte email utilisé
                                      pour l'envoi

  `EMAIL_PASS`                        Mot de passe ou secret
                                      d'application associé au compte
                                      email
  -----------------------------------------------------------------------

Le fichier `server/cloudinary.js` fait également référence aux variables
suivantes :

  Variable                  Description
  ------------------------- --------------------------
  `CLOUDINARY_NAME`         Nom du compte Cloudinary
  `CLOUDINARY_API_KEY`      Clé API Cloudinary
  `CLOUDINARY_API_SECRET`   Secret API Cloudinary

La configuration Cloudinary n'est pas utilisée dans le workflow
d'inscription actuellement actif. Ces variables ne sont donc pas
nécessaires à ce parcours tel qu'il est décrit.

**Ne commitez jamais votre fichier `.env` ni vos secrets dans le dépôt
Git.**

## Installation et lancement

Les commandes exactes peuvent dépendre des scripts définis dans les
fichiers `package.json` du dépôt.

### 1. Récupérer le projet

``` bash
git clone <URL_DU_DEPOT>
cd <DOSSIER_DU_PROJET>
```

### 2. Installer les dépendances du frontend

``` bash
cd client
npm install
```

### 3. Installer les dépendances du backend

Dans un autre terminal :

``` bash
cd server
npm install
```

### 4. Configurer l'environnement backend

Créer un fichier `.env` dans le dossier `server/` et y renseigner au
minimum les variables utilisées par le workflow actif :

``` env
MONGODB_URI=<URI_MONGODB>
PORT=<PORT>
EMAIL_USER=<ADRESSE_EMAIL>
EMAIL_PASS=<SECRET_EMAIL>
```

Remplacer les valeurs indicatives par vos propres paramètres. Ne
partagez pas vos identifiants.

### 5. Démarrer les applications

Lancer les commandes correspondant aux scripts disponibles dans chaque
`package.json`. Dans la configuration habituelle d'un projet Vite, le
frontend peut être démarré avec :

``` bash
npm run dev
```

Pour le backend, utiliser le script de démarrage défini dans
`server/package.json` (par exemple `npm start` si ce script y est
déclaré).

> Les scripts, ports et éventuelles étapes supplémentaires doivent être
> vérifiés dans les `package.json` et fichiers de configuration du dépôt
> avant utilisation.

## Déploiement actuel

D'après les URL présentes dans le code :

-   **Frontend :** déployé sur Vercel.
-   **Backend :** déployé sur Render.
-   **Bannière d'événement :** hébergée sur Cloudinary.

La présence de code de configuration Cloudinary ne signifie pas que le
service est intégré au parcours actif d'inscription.

## Limites connues et points de vigilance

Le projet doit être considéré comme un MVP. Les éléments suivants ne
sont pas implémentés ou nécessitent une amélioration.

### Fonctionnalités non disponibles

-   Création, modification, suppression et publication d'événements
    depuis l'application.
-   Tableau de bord administrateur et gestion des participants.
-   Comptes utilisateurs, authentification et gestion des rôles.
-   Consultation de l'historique des inscriptions par un participant.
-   Modification ou annulation d'une inscription.
-   Gestion de capacité, de places restantes et de liste d'attente.
-   Vérification du QR code à l'entrée et pointage des participants.
-   Paiement en ligne, remboursement et facturation.
-   Notifications de rappel ou emails autres que la confirmation
    actuelle.
-   Prévention des inscriptions en double.
-   Export CSV/Excel, statistiques et rapports.
-   Tests automatisés, pipeline CI/CD, sauvegardes et supervision
    dédiées.

### Points techniques à traiter

-   **Validation serveur :** le backend ne vérifie pas encore
    suffisamment les champs obligatoires ni leurs formats.
-   **Identifiants :** le frontend génère un UUID tronqué à huit
    caractères, tandis que le backend actif génère un identifiant
    aléatoire de six chiffres. C'est l'identifiant serveur qui est
    actuellement enregistré et utilisé dans le workflow. Une seule
    stratégie d'identification doit être retenue.
-   **Endpoint dupliqué :** `submit.js` contient une autre
    implémentation de l'endpoint, mais ce routeur n'est pas enregistré
    dans `index.js`. L'implémentation active est celle de `index.js`.
-   **Génération PDF en double :** le frontend contient du code
    `html2canvas`/`jsPDF`, mais le PDF envoyé par email est généré côté
    serveur avec Puppeteer. Il faudra clarifier et supprimer le code non
    utilisé si nécessaire.
-   **Données envoyées par le frontend :** le serveur actif génère son
    propre identifiant et ignore le champ `pdfBase64` envoyé par le
    frontend.
-   **Échec d'envoi email :** une erreur d'envoi peut survenir après
    l'enregistrement MongoDB. L'inscription peut donc exister sans que
    le participant ait reçu son email.
-   **QR code :** il contient des données personnelles et ne bénéficie
    pas encore d'une signature ou d'un mécanisme de vérification.
-   **Configuration frontend :** l'URL du backend et l'origine CORS sont
    codées en dur et devraient être configurables selon l'environnement.
-   **Capacité et doublons :** aucune limite du nombre d'inscriptions
    n'est définie et aucune règle n'empêche plusieurs inscriptions avec
    la même adresse email.
-   **Cloudinary :** `cloudinary.js` n'est pas utilisé dans le workflow
    actif et fait référence à des dépendances qui ne figurent pas dans
    le `package.json` selon l'analyse actuelle.
-   **Dépendances frontend :** `react-qr-code` et `react-qrcode-logo`
    sont signalées comme inutilisées dans l'implémentation actuelle ;
    leur conservation dépendra du choix final concernant la génération
    du QR code.

## Évolutions envisagées

La feuille de route ci-dessous présente les grands chantiers possibles
pour faire évoluer le MVP vers une plateforme de gestion d'événements
plus complète. Ces éléments sont des pistes d'évolution et ne doivent
pas être considérés comme déjà disponibles.

1.  **Fiabilisation du socle**
    -   Centraliser et renforcer la validation côté serveur.
    -   Uniformiser la génération des identifiants.
    -   Supprimer les implémentations et dépendances inutilisées.
    -   Structurer les erreurs et fiabiliser le traitement PDF/email.
    -   Ajouter des tests unitaires, d'intégration et API.
2.  **Gestion des événements**
    -   Créer une interface d'administration des événements.
    -   Gérer les dates, descriptions, images, lieux et horaires.
    -   Définir une capacité maximale et gérer les événements complets.
    -   Ajouter les statuts de publication et d'archivage.
3.  **Comptes et administration**
    -   Ajouter l'authentification et les rôles.
    -   Protéger les API d'administration.
    -   Fournir un tableau de bord pour consulter, rechercher, filtrer
        et exporter les inscriptions.
    -   Ajouter des statistiques et des rapports.
4.  **Cycle de vie des inscriptions**
    -   Permettre la modification et l'annulation.
    -   Gérer les statuts d'inscription.
    -   Prévenir les doublons.
    -   Ajouter une liste d'attente et des notifications associées si
        nécessaire.
5.  **Billetterie et contrôle d'accès**
    -   Mettre en place la lecture et la vérification des QR codes.
    -   Empêcher la réutilisation d'un billet déjà validé.
    -   Enregistrer l'heure d'arrivée.
    -   Étudier l'intégration d'un fournisseur de paiement si les
        événements deviennent payants.
6.  **Sécurité et protection des données**
    -   Ajouter authentification, autorisation et limitation des
        requêtes.
    -   Protéger les données personnelles et réduire les informations
        intégrées aux QR codes.
    -   Gérer les secrets et les configurations par environnement.
    -   Définir les règles de conservation et de suppression des
        données.
7.  **Industrialisation et exploitation**
    -   Ajouter Docker et Docker Compose.
    -   Mettre en place une chaîne CI/CD.
    -   Séparer les environnements de développement, test et production.
    -   Prévoir sauvegardes, restauration, logs, surveillance et
        alertes.
8.  **Expérience utilisateur et documentation**
    -   Ajouter états de chargement et messages d'erreur détaillés.
    -   Améliorer l'accessibilité et l'affichage responsive.
    -   Ajouter une page de suivi d'inscription et des options pratiques
        pour le participant.
    -   Documenter l'architecture finale, les modèles de données, les
        rôles, le déploiement et les procédures d'exploitation.

------------------------------------------------------------------------

## Author

Bakhta Nairouz Ben Hadj Slema
