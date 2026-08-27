# DotResto

Application SaaS de gestion de restaurant (POS, commandes, cuisine, réservations, inventaire, rapports) — architecture full-stack Node.js/Express + MySQL pour le backend, React/Vite pour le frontend.

Par défaut, l'application est en **français** (anglais disponible), la devise par défaut des nouveaux comptes est le **Franc guinéen (GNF)**, et trois moyens de paiement (**Cash, Mobile Money, Carte bancaire**) sont créés automatiquement pour chaque nouveau restaurant inscrit.

## Prérequis

- [Node.js](https://nodejs.org/) 18 ou supérieur
- [MySQL](https://dev.mysql.com/downloads/mysql/) 8.0 ou supérieur (le schéma utilise la collation `utf8mb4_0900_ai_ci`, disponible à partir de MySQL 8.0)
- npm

## Structure du dépôt

```
dotresto/
├── backend/dotresto-backend/    # API Express (port 3000)
├── frontend/dotresto-frontend/  # App React/Vite (port 5173)
└── dotresto.sql                      # Schéma complet de la base de données
```

## 1. Cloner le dépôt

```bash
git clone https://github.com/Douraaa1/dotresto.git
cd dotresto
```

## 2. Base de données

Créez la base et importez le schéma (vide, sans données) :

```bash
mysql -u root -p -e "CREATE DATABASE dotresto CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;"
mysql -u root -p dotresto < dotresto.sql
```

## 3. Backend

```bash
cd backend/dotresto-backend
npm install
cp .env.example .env
```

Renseignez `.env` :

| Variable | Description |
|---|---|
| `DATABASE_URL` | `mysql://utilisateur:motdepasse@localhost:3306/dotresto` |
| `JWT_SECRET` | Chaîne secrète arbitraire pour signer les tokens |
| `JWT_EXPIRY` / `JWT_EXPIRY_REFRESH` | Ex. `15m` / `30d` |
| `COOKIE_EXPIRY` / `COOKIE_EXPIRY_REFRESH` | En millisecondes, doit correspondre aux durées JWT ci-dessus |
| `FRONTEND_DOMAIN` | `http://localhost:5173` en local |
| `FRONTEND_DOMAIN_COOKIE` | `localhost` en local |
| `SMTP_HOST` / `SMTP_PORT` / `SMTP_EMAIL` / `SMTP_PASSWORD` | Optionnel — nécessaire pour l'envoi d'e-mails (réinitialisation de mot de passe). [Mailtrap](https://mailtrap.io/) convient pour du test en local |
| `STRIPE_SECRET` / `STRIPE_WEBHOOK_SECRET` | Optionnel — uniquement si vous activez les abonnements payants via Stripe |
| `ENCRYPTION_KEY` | Chaîne arbitraire utilisée pour le chiffrement de certaines données |

Démarrez le serveur :

```bash
npm run dev
```

L'API tourne sur `http://localhost:3000`.

## 4. Frontend

```bash
cd frontend/dotresto-frontend
npm install
npm run dev
```

L'app tourne sur `http://localhost:5173`. Le fichier `.env` fourni pointe déjà vers `http://localhost:3000`, aucune configuration supplémentaire n'est nécessaire pour du développement local.

## 5. Créer votre premier restaurant

Rendez-vous sur `http://localhost:5173/register` pour créer un compte.

⚠️ **Important** : un nouveau restaurant est créé **inactif** et sans abonnement — c'est la logique métier normale de l'app (gestion d'abonnements Stripe/Paystack). Sans plan actif, la connexion redirige vers une page « pas d'accès ». Pour du développement local sans passerelle de paiement configurée, activez-le manuellement en base :

```sql
-- Créer un plan "développement local" donnant accès à toutes les fonctionnalités
INSERT INTO plans (title, payment_gateway, payment_gateway_product_id, features, is_trial)
VALUES (
  'Local Dev', 'manual', 'local-dev-plan',
  '["DASHBOARD","POS","ORDERS","KITCHEN","RESERVATIONS","CUSTOMERS","INVOICES","MEMBERSHIP","INVENTORY","SETTINGS","REPORTS","FEEDBACK","USER","QRMENU"]',
  0
);

-- Activer votre tenant avec ce plan (remplacez <TENANT_ID> par l'id du tenant créé)
UPDATE tenants SET is_active = 1, payment_gateway_product_id = 'local-dev-plan' WHERE id = <TENANT_ID>;
```

## 6. Créer un compte superadmin

Il n'existe pas d'inscription libre-service pour le rôle superadmin — il faut l'insérer directement en base avec un mot de passe hashé :

```bash
cd backend/dotresto-backend
node -e "require('bcrypt').hash('VOTRE_MOT_DE_PASSE', 10).then(console.log)"
```

```sql
INSERT INTO superadmins (email, password, name)
VALUES ('admin@example.com', '<hash_bcrypt_généré_ci-dessus>', 'Super Admin');
```

Connexion sur `http://localhost:5173/superadmin/login`.

## Scripts disponibles

**Backend** (`backend/dotresto-backend`)
- `npm run dev` — démarre avec nodemon (rechargement automatique)
- `npm start` — démarre en mode production

**Frontend** (`frontend/dotresto-frontend`)
- `npm run dev` — serveur de développement Vite
- `npm run build` — build de production
- `npm run preview` — prévisualise le build de production
