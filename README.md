# AGR — Déploiement Firebase + GitHub

Ce package remplace le backend Node.js/PostgreSQL par **Firebase Realtime
Database**, dans le même esprit que vos autres applications (AGP/AGM) :
tout se passe côté navigateur, sans serveur à faire tourner vous-même.

**Le projet Firebase `agr-restau` est déjà configuré** dans `index.html`,
`.firebaserc` et le workflow GitHub — vous pouvez passer directement à
l'étape 3.

```
agr-firebase-deploy/
├── index.html                 # L'application complète (1 seul fichier)
├── firebase.json              # Config Firebase Hosting + Database
├── .firebaserc                 # Référence à votre projet Firebase (agr-restau)
├── database.rules.json         # Règles de la Realtime Database
└── .github/workflows/
    └── firebase-hosting-merge.yml   # Déploiement auto à chaque push
```

## Étape 1 — Activer la Realtime Database (si pas déjà fait)

Dans https://console.firebase.google.com → projet **agr-restau** →
**Realtime Database** → **Créer une base de données** → région Europe →
démarrez **en mode test**. L'URL doit correspondre à celle déjà utilisée
dans `index.html` :
`https://agr-restau-default-rtdb.europe-west1.firebasedatabase.app/`

## Étape 2 — Configuration déjà intégrée

Le bloc `firebaseConfig` en haut de `index.html` (`<script type="module">`)
contient déjà les clés de votre projet `agr-restau`. Rien à modifier ici.

## Étape 3 — Créer votre compte Super Admin (concepteur)

Dans la Console Firebase → **Realtime Database** → cliquez sur le **+** à
côté de la racine et créez à la main :

```
agr_platform_admins
  └── admin1
        nom: "Souleymane Diallo"
        email: "souleymane0403@gmail.com"
        password: "choisissez_un_mot_de_passe"
```

⚠️ Ce mot de passe est stocké **en clair** dans la base (comme dans vos
autres applications de ce type) — voir la note de sécurité en bas de ce
fichier.

## Étape 4 — Pousser le code sur GitHub

```bash
cd agr-firebase-deploy
git init
git add .
git commit -m "Première version AGR"
git branch -M main
git remote add origin https://github.com/VOTRE-COMPTE/agr-app.git
git push -u origin main
```

## Étape 5 — Déployer sur Firebase Hosting

Installer l'outil Firebase (une seule fois) :
```bash
npm install -g firebase-tools
firebase login
```

Dans le dossier `agr-firebase-deploy`, remplacez le project ID dans
`.firebaserc`, puis :
```bash
firebase deploy
```

Votre application sera en ligne à une adresse du type
`https://VOTRE-PROJECT-ID.web.app`.

## Étape 6 — Déploiement automatique depuis GitHub (optionnel)

Pour que chaque `git push` redéploie automatiquement, la façon la plus
fiable est de laisser Firebase générer lui-même le workflow et le secret
GitHub nécessaires :

```bash
firebase init hosting:github
```

Répondez aux questions (dépôt GitHub, branche `main`) — cela crée
automatiquement `.github/workflows/` et le secret
`FIREBASE_SERVICE_ACCOUNT_...` dans votre dépôt GitHub. Le fichier
`.github/workflows/firebase-hosting-merge.yml` fourni ici est un modèle de
référence si vous préférez le configurer à la main.

## Structure des données (Realtime Database)

```
agr_tenants/
  <CODE_RESTAURANT>/
    info/       { nomRestaurant, ville, dgEmail, dgNom, statut, dateCreation, dateExpiration }
    users/
      dg/       { role: "dg", nom, email, password }

agr_platform_admins/
  admin1/       { nom, email, password }
```

Le `CODE_RESTAURANT` (ex. `LEGOURMET482`) est généré automatiquement à
l'inscription à partir du nom du restaurant — c'est lui qui sert
d'identifiant pour se connecter, exactement comme le "code prestation" de
vos autres applications.

## ⚠️ Note de sécurité importante

Ce modèle stocke les mots de passe **en clair** dans la Realtime Database,
et les règles fournies (`database.rules.json`) sont **ouvertes en lecture
et écriture à tout le monde** :

```json
{ "rules": { ".read": true, ".write": true } }
```

C'est le choix le plus simple pour démarrer rapidement sans serveur, et
c'est cohérent avec le fonctionnement de vos autres applications
(AGP/AGM). Mais concrètement, cela veut dire que n'importe qui connaissant
l'URL de votre base peut lire ou modifier toutes les données de tous les
restaurants. Si vous voulez une vraie sécurité plus tard, la prochaine
étape serait d'activer **Firebase Authentication** et d'écrire des règles
qui vérifient l'identité de la personne connectée avant d'autoriser une
lecture/écriture — je peux vous accompagner sur cette évolution quand vous
serez prêt.
