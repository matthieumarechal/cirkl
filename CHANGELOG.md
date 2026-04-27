# Changelog

Toutes les modifications notables de CIRKL sont documentées ici.  
Format basé sur [Keep a Changelog](https://keepachangelog.com/fr/1.0.0/).

---

## [0.1.3] — Sorbet Mimosa — 27 Avril 2026

### Added
- Ajout d'un ystème de points (les Bogomils) avec barème dégressif (25→1 pt par tâche, bonus 50 pts si tout terminé - barème suceptible de changer - récompense en cours d'élaboration)
- Système anti-abus par jour pour les Bogomils
- Animation de particule quand on gagne des Bogomils

## [0.1.2] — Mauve Crépuscule — Avril 2026

### Added
- Ajout de la fonctionnalité "mot de passe oublié"
- Ajout du champ "prénom" lors de l'inscription via compte e-mail
- Affichage de votre prénom ou pseudo Discord dans le menu uniquement en version Desktop
- Ajout de gros curseurs pour la souris (en essai pour le moment)


## [0.1.1] — Mauve Crépuscule — Avril 2026

### Added
- Suppression définitive d'une tâche (icône ✕, son de suppression)
- Menu hamburger sur mobile (Effacer, À propos, Déconnexion)
- Section "Bugs connus / améliorations" dans le modal À propos
- Notice Umami dans le modal À propos
- Amélioration de la persistance de session sur iOS PWA (visibilitychange + TOKEN_REFRESHED)

### Changed
- Modal À propos : largeur augmentée sur desktop, coins droits carrés, scrollbar inset personnalisée

### Fixed
- Erreur 403 à l'ajout de tâche causée par des triggers pgsodium résiduels (`crypto_aead_det_encrypt`)

---

## [0.1.0] — Mauve Crépuscule — Avril 2026

### Added
- Authentification email/mot de passe et connexion Discord (via Supabase)
- Gestion de compte pour synchroniser les tâches sur plusieurs appareils
- Rafraîchissement en temps réel des tâches (Supabase Realtime)
- Ajout, complétion et réorganisation des tâches par drag & drop
- Barre de progression toujours visible en haut du site
- Effetti confettis à la completion de toutes les tâches
- Installable sur mobile en raccourci écran d'accueil (iOS PWA)
- Modal "À propos"
- Captcha Cloudflare Turnstile à l'inscription

---

