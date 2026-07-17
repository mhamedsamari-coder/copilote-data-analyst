# Copilote Data Analyst

Agent IA qui t'aide à préparer un premier poste de Data Analyst : analyse de CV, feuille de route de compétences, entraînement à l'entretien, génération de lettre de motivation. Propulsé par l'API **Mistral**, avec tes données sauvegardées dans une **vraie base de données** (pas seulement dans ton navigateur).

La clé API Mistral et la base de données restent côté serveur — jamais exposées dans le navigateur.

---

## 1. Créer ta clé API Mistral

1. Va sur https://console.mistral.ai/api-keys
2. Crée un compte si besoin, puis génère une clé API. Garde-la de côté.

## 2. Créer une base de données Postgres gratuite

Deux options :

**Option recommandée : Neon (gratuit, sans expiration)**
1. Va sur https://neon.tech et crée un compte gratuit.
2. Crée un nouveau projet. Neon te donne directement une "Connection string" du type :
   `postgresql://user:password@ep-xxxx.neon.tech/dbname?sslmode=require`
3. Copie cette URL, tu en auras besoin à l'étape suivante.

**Option alternative : Postgres directement sur Render**
- Plus simple si tu restes dans un seul écosystème, mais la base gratuite de Render **expire 30 jours après sa création** (puis 14 jours de grâce avant suppression). Il faudra la recréer ou passer sur un plan payant. Neon n'a pas cette limite.

## 3. Lancer en local

Prérequis : [Node.js](https://nodejs.org) version 18 ou plus (voir la note en bas de fichier si tu es sur un vieux macOS).

```bash
cd copilote-data-analyst
npm install
cp .env.example .env
```

Ouvre `.env` et remplis :
```
MISTRAL_API_KEY=ta_vraie_cle_mistral
DATABASE_URL=ta_connection_string_postgres
```

Puis lance le serveur :
```bash
npm start
```

Ouvre http://localhost:3000. Au premier chargement, un identifiant anonyme est généré et affiché en bas de la barre latérale — note-le si tu comptes retrouver tes données depuis un autre appareil (bouton "Restaurer depuis un identifiant" pour le renseigner ailleurs).

## 4. Mettre le projet sur GitHub

1. Installe [Git](https://git-scm.com/downloads) si besoin, crée un compte sur [github.com](https://github.com).
2. Crée un nouveau dépôt vide sur GitHub (bouton "New repository"), par exemple `copilote-data-analyst`. Ne coche aucune case (pas de README/gitignore) — le dépôt doit être vide.
3. Dans le terminal :

```bash
cd copilote-data-analyst
git init
git add .
git commit -m "Premier commit"
git branch -M main
git remote add origin https://github.com/TON_PSEUDO/copilote-data-analyst.git
git push -u origin main
```

(remplace `TON_PSEUDO` par ton nom d'utilisateur GitHub). Le `.gitignore` fourni exclut déjà `node_modules/` et `.env` : ta clé API et ta base de données ne sont jamais envoyées sur GitHub.

## 5. Déployer sur Render

1. Crée un compte sur [render.com](https://render.com) (connexion possible avec GitHub).
2. Clique sur **New → Blueprint**, sélectionne ton dépôt. Render détecte le fichier `render.yaml` fourni et pré-remplit la configuration.
   - À la main sinon : **New → Web Service**, build command `npm install`, start command `npm start`.
3. Render te demande les valeurs de `MISTRAL_API_KEY` et `DATABASE_URL` (marquées `sync: false`, donc jamais stockées dans le repo) — colle ta clé Mistral et ta connection string Neon.
4. Clique sur **Create Web Service** (ou **Apply**). Le déploiement prend 1 à 2 minutes.
5. Render affiche une URL du type `https://copilote-data-analyst.onrender.com` — accessible depuis n'importe quel appareil.

**Note sur le plan gratuit de Render** : le service se met en veille après 15 minutes d'inactivité, ~30-50 secondes pour se relancer au prochain appel. Normal, pas une erreur.

**Alternatives équivalentes à Render** : Railway.app, Fly.io.

---

## Comment fonctionne la sauvegarde des données

- Au premier lancement, le navigateur génère un identifiant anonyme aléatoire, stocké localement (c'est la seule chose gardée dans le navigateur).
- Tout le reste (CV, feuille de route, lettre, profil) est envoyé au serveur et stocké dans la table Postgres `user_data`, indexée par cet identifiant.
- Pour retrouver tes données sur un autre appareil ou navigateur : note ton identifiant (affiché dans la barre latérale) et utilise le bouton "Restaurer depuis un identifiant" sur le nouvel appareil.
- Il n'y a pas de compte/mot de passe — n'importe qui connaissant ton identifiant pourrait accéder à tes données. Si tu veux un vrai système de connexion (email + mot de passe), dis-le-moi, c'est une évolution possible.

## Structure du projet

```
copilote-data-analyst/
├── server.js          # backend Express : proxy Mistral + lecture/écriture Postgres
├── package.json
├── render.yaml         # config de déploiement automatique sur Render
├── .env.example        # à copier en .env avec tes clés
├── .gitignore
└── public/
    └── index.html       # frontend React (aucune étape de build nécessaire)
```

## Personnaliser

- Modèle Mistral : variable `MISTRAL_MODEL` dans `.env` (par défaut `mistral-large-latest`; `mistral-small-latest` pour réduire les coûts).
- Les prompts (analyse de CV, feuille de route, entretien, lettre) sont dans `public/index.html`, modifiables directement.

## Note pour macOS High Sierra (ou plus ancien)

Node 18 nécessite macOS 10.15+. Si ton Mac est bloqué sur une version plus ancienne, tu peux sauter complètement l'étape "lancer en local" (section 3) et déployer directement sur Render (sections 4 et 5) : c'est leur serveur qui fait tourner Node, pas ton Mac.
