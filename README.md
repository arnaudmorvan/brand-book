# brand-book · pipeline IA pour identités visuelles éditoriales

Skill Claude qui transforme un logo + quelques `.txt` sources en charte de marque complète, brand book Figma 15 slides, et landing page production-ready. Documenté avec le cas d'étude **fokusia** (graphiste freelance, identités visuelles pour cabinets d'investigation, avocats en cybercriminalité, entreprises de cybersécurité).

---

## Structure du repo

```
brand-book/
├── README.md
├── .gitignore
├── skill/                       ← LE SKILL RÉUTILISABLE
│   ├── SKILL.md                  (2700+ lignes, doc + règles)
│   ├── prerequis-legaux.md       (checklist brute LCEN + RGPD)
│   └── templates/                ← squelette projet prêt à copier
│       ├── description-services.txt
│       ├── infos-entreprise.txt
│       ├── temoignages-clients.txt
│       ├── logo/
│       │   └── README.md          (instructions de dépôt)
│       └── images/
│           ├── README.md
│           └── cas-clients/
│               ├── texte-cas-clients.txt
│               └── README.md
└── examples/
    └── fokusia/                  ← CAS D'ÉTUDE COMPLET
        ├── brand.md
        ├── landing-page.md
        ├── landing.html
        ├── landing-content.txt
        ├── description-services.txt    (rempli)
        ├── infos-entreprise.txt
        ├── temoignages-clients.txt
        ├── logo/
        │   ├── logo.svg
        │   └── logo-white.svg
        └── images/
            ├── hero.jpg
            ├── photo-profil.jpg
            └── cas-clients/
                ├── texte-cas-clients.txt
                ├── cas1.png
                ├── cas2.png
                └── cas3.png
```

Deux dossiers indépendants :

- **`/skill/`** — le pipeline réutilisable (skill Claude + templates vides). Tout ce qu'il faut pour démarrer un nouveau projet de marque.
- **`/examples/fokusia/`** — un cas d'étude complet, livrable de bout en bout. Sert de référence pour voir ce que produit le skill avec des inputs réels.

---

## Quick start

### 1. Cloner et copier les templates

```bash
git clone <ce-repo>
cp -r brand-book/skill/templates ~/mon-nouveau-projet
cd ~/mon-nouveau-projet
```

Tu obtiens un squelette de projet avec les 4 `.txt` vides à remplir et les sous-dossiers `logo/` et `images/cas-clients/` documentés par leur README.

### 2. Remplir les `.txt` sources

Éditer :

- `description-services.txt` — identité, promesse, persona, méthode, formule signature, objection #1, différenciateur, CTA
- `infos-entreprise.txt` — SIRET, hébergeur, mentions RGPD, URLs Calendly/LinkedIn, email
- `temoignages-clients.txt` — 3 verbatim clients avec attribution
- `images/cas-clients/texte-cas-clients.txt` — 3 cas projet phares

Tout `.txt` rempli skippe la vague d'interview correspondante du skill (gain de temps important).

### 3. Déposer les visuels

Coller :

- `logo/logo.svg` et `logo/logo-white.svg` (variantes couleur + blanche, exportées depuis Figma)
- `images/hero.jpg` (image principale, optionnel — fallback CSS sinon)
- `images/photo-profil.jpg` (portrait section About, optionnel)
- `images/cas-clients/cas1.png`, `cas2.png`, `cas3.png` (mockups des 3 cas, optionnel)

Le skill consomme tous ces fichiers automatiquement à la détection.

### 4. Lancer le skill

Dans Claude (Cowork, Claude Code, ou Claude.ai avec extension), donner :

- L'URL du fichier Figma cible (avec les deux logos `Logo couleur` et `Logo blanc` collés sur Page 1)
- Le chemin du dossier projet
- (Optionnel) un trigger explicite : « scan ma charte », « brand setup complet », « fais le brand book », « génère la landing »

Le skill scanne le dossier, lit les `.txt`, pose uniquement les questions auxquelles il n'a pas trouvé de réponse (typo, univers visuel), et livre :

- `brand.md` — source de vérité de l'identité
- `landing-page.md` — brief structuré de landing
- `landing.html` — landing single-page production-ready
- `landing-content.txt` — textes IA-générés éditables
- Brand book Figma 15 slides 1920×1280

### 5. Éditer le contenu (mode SYNC)

Pour modifier un texte de la landing sans toucher au HTML :

1. Éditer `landing-content.txt` (textes IA) ou n'importe lequel des 4 `.txt` sources
2. Dire à Claude : « resynchronise la landing »
3. Claude relit tout, regénère `landing.html` en 3 secondes, sans interview

---

## Les 4 modes du skill

| Mode | Quand | Sortie |
|---|---|---|
| **CRÉATION** | `brand.md` absent, logo détecté | Phase A (interview ou pré-consommation `.txt`) + Phase B (Figma) |
| **BOOK** | `brand.md` présent, nouveau fichier Figma | Brand book Figma uniquement |
| **LANDING** | `brand.md` présent, trigger « refais la landing » | Régénération `landing-page.md` + `landing.html` |
| **SYNC** | `landing.html` + `landing-content.txt` présents, trigger « resynchronise » | Régénération rapide de `landing.html` depuis les `.txt` (zéro question) |

---

## Garanties production-ready

Le skill applique 5 règles bloquantes avant de livrer le HTML :

- **LOGO** — toujours le vrai SVG du dossier `./logo/`, jamais un wordmark recréé en CSS
- **IMAGES** — cascade de fallback en 3 niveaux (image locale > Unsplash hot-link > composition CSS riche), `<img>` toujours présent dans le DOM
- **PRODUCTION-READY** — zéro `href="#"` orphelin, toutes les URLs réelles (Calendly, mailto, LinkedIn, mentions), `target="_blank" rel="noopener noreferrer"` sur les externes
- **ANIMATIONS** — easing éditorial `cubic-bezier(0.16, 1, 0.3, 1)`, stagger 0.1-0.15s, parallax léger sur hero, `prefers-reduced-motion` respecté
- **ANTI-AI-SLOP** — pas de gradients violet, pas de blobs colorés, pas de cards rounded-2xl shadow-xl génériques

Mentions légales et politique de confidentialité intégrées en accordions au pied de la landing (single-page, pas de fichiers HTML séparés), remplis depuis `infos-entreprise.txt`.

---

## Architecture des `.txt`

Pour gagner du temps d'interview, l'utilisateur prépare ces fichiers à l'avance. Le skill les détecte automatiquement (étape A.1.f du SKILL.md).

| Fichier | Contenu | Remplit |
|---|---|---|
| `description-services.txt` | Identité, promesse, persona, méthode (3 étapes), formule signature, objection #1, différenciateur, texte du CTA | Vagues 1, 2, 5 de l'interview |
| `infos-entreprise.txt` | SIRET, forme juridique, hébergeur, mentions RGPD, URLs Calendly/LinkedIn, email contact | Étape production-ready A.5.0.quater entièrement |
| `temoignages-clients.txt` | 3 verbatim clients avec attribution (fonction, lieu, année) | Section Témoignages |
| `images/cas-clients/texte-cas-clients.txt` | 3 cas : titre, secteur, année, brief original, ce qui a été livré | Section Portfolio |
| `landing-content.txt` *(écrit par le skill)* | Eyebrows, H1/H2 IA, intros, bullets, réponses FAQ Q2-Q5, sous-titres | Édition légère sans interview · mode SYNC |

Priorité des sources pour chaque variable : `.txt` > réponse interview > inférence brand.md > défaut documenté.

---

## Workflow détaillé

```
┌─────────────────────────────────────────────────────────────────┐
│  1. Préparation utilisateur                                     │
│     · Logo SVG dans Figma + dans /logo/                         │
│     · Remplir les 4 .txt sources (ou laisser vide → interview)  │
│     · Déposer hero.jpg, photo-profil.jpg, cas1-3.png (optionnel)│
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  2. Skill · Phase A                                             │
│     · A.1 scan dossier + détection logos Figma                  │
│     · A.1.f pré-consommation des 4 .txt → variables internes    │
│     · A.2 interview (vagues skippées si .txt rempli)            │
│     · A.3 extraction tokens depuis logo                         │
│     · A.4 écriture brand.md                                     │
│     · A.5.0 → A.5.0.quater (questions structure, hero,          │
│       portfolio, production-ready)                              │
│     · A.5.a → A.5.f rédaction landing-page.md + lint            │
│     · A.5.g génération landing.html + landing-content.txt       │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  3. Skill · Phase B (Figma)                                     │
│     · 7 appels use_figma batchés                                │
│     · 15 slides 1920×1280 dans page « ✦ Brand Book »            │
│     · Variables couleur bindées (collection « Brand Tokens »)   │
│     · Logos clonés depuis Page 1 du Figma cible                 │
│     · QA visuelle automatique                                   │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  4. Édition continue · mode SYNC                                │
│     · User édite n'importe quel .txt                            │
│     · « resynchronise la landing » → regen landing.html         │
│     · 3 secondes, zéro question, idempotent                     │
└─────────────────────────────────────────────────────────────────┘
```

---

## Cas d'étude · fokusia

Le dossier `/examples/fokusia/` contient un livrable complet et cohérent :

- Identité visuelle premium pour cabinets de la discrétion corporate
- Palette teal `#0A2B32` + rouge focus `#E73737` chirurgical
- Typographie Playfair Display + Inter
- Persona dirigeants de cabinet d'investigation qui basculent vers le B2B
- Landing single-page de ~1700 lignes HTML, accessible WCAG AA, accordions mentions/confidentialité intégrés

Ouvre `/examples/fokusia/landing.html` dans ton navigateur pour voir le rendu final.

Tu peux dupliquer ce dossier comme point de départ pour un projet similaire (secteur premium / éditorial), remplacer logo + textes + images, et regénérer.

---

## Licence

Skill et code de pipeline · MIT. Cas d'étude fokusia (textes, identité, logo) · propriété d'Isabelle Marques, utilisable uniquement comme référence éditoriale dans le cadre de ce repo.

---

## Crédits

Skill conçu par [Arnaud Morvan](https://linkedin.com/in/arnaud-morvan-design) avec Claude. Cas d'étude **fokusia** par Isabelle Marques.

Itérations clés documentées dans l'historique des commits.
