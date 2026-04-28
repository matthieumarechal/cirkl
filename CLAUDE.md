# CIRKL — Claude Context

Application web PWA de gestion de tâches routinières personnelle. Projet solo de Matthieu.

## Architecture

**Tout le projet tient dans un seul fichier : `index.html`**
CSS, HTML, et JavaScript sont tous dans ce fichier. Il n'y a pas de build step, pas de framework, pas de bundler.

- `img/` — assets (icônes PWA, splash screens iOS, curseurs custom, `img/icon/miriel.svg`)
- `manifest.json` — manifest PWA Android (à la racine, même niveau qu'index.html)
- `CHANGELOG.md`, `README.md` — documentation
- Repo GitHub : `git@github.com:matthieumarechal/cirkl.git`

## Supabase

Deux projets Supabase séparés (données isolées, schéma identique) :

**Prod** — `cirkl.craftbench.fr`
```
URL  : https://orcfwzmejmbeluvpyspc.supabase.co
ANON : sb_publishable_Dt9kU8AybQhJNtwIbPG59g_Qd7O9LmF
```

**Dev** — `cirkl-prod.craftbench.fr` (nom trompeur, c'est bien le site de DEV)
```
URL  : https://aeftmtflthbkawiezucr.supabase.co
ANON : sb_publishable_Cuk6o3dc2z4abZDVTRncMA_NyIXSUu1
```

Pour basculer entre les deux : commenter/décommenter les deux blocs `const SUPABASE_URL / SUPABASE_ANON` vers la ligne 1147 de `index.html`.

La clé `anon` en clair dans le frontend est **sûre** grâce au Row Level Security (RLS) Supabase. Ne jamais mettre la clé `service_role` dans le code.

Auth : email/password + Discord OAuth (Discord pas configuré sur dev, email seulement). Cloudflare Turnstile anti-bot sur l'écran de connexion (même widget, plusieurs domaines configurés).

## Schéma base de données

```sql
-- Tâches
tasks (
  id uuid PK,
  user_id uuid FK auth.users,
  text text,
  done boolean,
  position integer,
  created_at timestamptz
)

-- Système de points
user_points (
  user_id uuid PK FK auth.users,
  balance integer,
  updated_at timestamptz
)

points_log (
  id uuid PK,
  user_id uuid FK auth.users,
  task_id uuid FK tasks (nullable),
  points integer,
  log_date date,         -- clé anti-abus : 1 récompense par task par jour
  reason text,           -- 'task_complete' | 'all_tasks_bonus'
  created_at timestamptz
)
```

Toutes les tables ont RLS activé (politique `auth.uid() = user_id`).

## Fonctionnalités implémentées

- Auth email/password + Discord OAuth
- CRUD tâches avec drag & drop pour réordonner
- Reset quotidien des tâches (bouton "Tout effacer")
- Confettis + son au cochage d'une tâche
- Pluie de confettis + son de victoire quand toutes les tâches sont faites
- **Suppression de tâche avec undo** :
  - Suppression optimiste (UI d'abord, DB après 10s)
  - Toast en bas à droite (desktop) ou bas centré (mobile) avec bouton "Annuler"
  - Barre de progression animée dans le toast (rétrécit sur 10s)
  - Undo = annulation du timer + restauration dans le tableau local (aucun appel DB car la suppression n'a pas encore eu lieu)
  - Si une nouvelle suppression arrive avant la fin du timer, l'ancienne est commitée immédiatement
- **Système de points (Bogomils)** :
  - La monnaie s'appelle **Bogomils** (symbole : `miriel.svg`)
  - Barème dégressif : 25, 20, 15, 10, 5, puis 1 pt par tâche cochée dans la journée
  - Bonus 50 pts quand toutes les tâches sont complétées
  - Anti-abus : `log_date` empêche de re-gagner des points en cochant/décochant
  - Animation particules depuis le point de clic vers le compteur de points
  - Sons (Web Audio API) : tick toutes les 5 particules, pitch proportionnel aux points gagnés
  - **Desktop** (≥769px) : pastille en haut à droite du header (`#points-display`), à droite du prénom (`#greeting`). Affiche icône + solde + "Bogomils"
  - **Mobile** (<769px) : affiché dans `#counter-row` à droite de "X/Y terminées", même format icône + solde + "Bogomils". Particules remontent vers ce compteur (`animatePointsGain` utilise `window.innerWidth >= 769` pour choisir la cible)
- **Curseurs custom** :
  - Curseur par défaut : `img/cursor/pointer.svg`
  - Curseur liens/boutons : `img/cursor/pointer_links.svg`
  - Curseur drag : `img/cursor/cursor_grab.svg`
  - Curseur champ texte (`#new-task-input`) : `img/cursor/cursor_text.svg`
  - **Important** : tous les curseurs utilisent des URLs absolues `https://cirkl.craftbench.fr/img/cursor/...` — ne pas utiliser de chemins relatifs sinon ça ne fonctionne pas
- **Favicon** : `img/icon/favicon.ico` (32×32)
- **Modal "À propos"** avec accordéons :
  - 6 sections : C'est quoi ?, Comment ça marche ?, Gestion des points, Bugs connus / améliorations, Disclaimer, Notes de version
  - Comportement séquentiel : ferme d'abord la section ouverte (650ms CSS transition), puis ouvre la nouvelle
  - `touch-action: manipulation` sur `.accordion-trigger` pour éliminer le délai 300ms mobile
  - Flèches chevron agrandies via `transform: scale(1.5)` (sans affecter la hauteur des lignes)
  - **Desktop** : scroll limité à `.accordion-list` (pas le modal entier), fond `#f0e9de` dans la zone vide sous les items, items avec fond `var(--surface)`, bordure bottom sur le dernier item
  - **Mobile** : bottom sheet plein largeur (85svh), animation slide-up, scroll dans `.accordion-list`, flèche décalée à gauche via `margin-right`, padding spécifique sur la section "Notes de version"

## PWA

- **iOS** : meta tags `apple-mobile-web-app-capable`, `apple-touch-icon` (76, 120, 152, 167, 180px), splash screens. Pas de bannière d'installation automatique sur iOS — l'utilisateur doit passer par Partager → "Sur l'écran d'accueil" dans Safari
- **Android** : `manifest.json` à la racine avec icônes 192×192 et 512×512 (`purpose: "any maskable"`), `<link rel="manifest">` et `<meta name="theme-color" content="#F5F0E8">` dans le head. Chrome propose automatiquement l'installation après quelques visites

## Points à faire

- **Shop / utilisation des Bogomils** : à définir (idées : thèmes de couleur, effets particules alternatifs, badges/titres...)
- **Merger `feature/points` dans `main`** et déployer en prod
- **Pousser CLAUDE.md sur GitHub**
- **Mettre à jour CHANGELOG.md** avec les nouveautés (Bogomils, undo suppression, affichage mobile, modal accordéon, PWA Android, favicon, curseurs)
- **E2E encryption** : Web Crypto API, AES-GCM 256 bits, avant lancement public
- **Support Android PWA** : manifest.json en place ✓, à tester sur un vrai appareil Android

## Branches git notables

- `main` — branche prod déployée
- `feature/points` — système de points (à merger dans main)
- `feature/realtime` — sync temps réel entre onglets (en attente)

## Déploiement

Déploiement manuel : modifier `index.html`, pousser sur GitHub. Le serveur (`craftbench.fr`) récupère depuis GitHub (mécanisme exact non documenté ici).

Quand une feature est validée sur dev → faire tourner le même SQL sur la BDD prod via Supabase SQL Editor avant de déployer.
