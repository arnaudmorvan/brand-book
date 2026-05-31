# /images/

Déposer ici les visuels de la landing :

- `hero.jpg` — image principale du hero plein écran (recommandé : 1920×1280 minimum, format JPG ou PNG). Optionnel : si absente, le skill génère un fallback CSS riche ou hot-link une photo Unsplash temporaire.
- `photo-profil.jpg` — portrait pour la section À propos (recommandé : ratio 4:5, lumière naturelle indirecte, traitement éditorial cohérent avec le brand). Optionnel : si absente, fallback monogramme + initiales.

Le sous-dossier `/cas-clients/` contient les visuels des 3 projets phares (voir son README).

Tous les chemins d'images sont référencés via `<img>` dans la landing, avec fallback `onerror` gracieux si un fichier manque.
