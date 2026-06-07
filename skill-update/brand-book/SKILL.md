---
name: brand-book
description: >
  Pipeline charte de marque. Interview 18 questions (5 vagues), écrit
  `brand.md` + `landing-page.md` (copy rédigée, 3 templates), génère
  brochure Figma 15 slides 1920×1280 éditoriale : slots images MoodBoard
  sur chaque slide, 5 presets visuels adaptatifs (Luxe éditorial, Minimal
  tech, Crème organique, Pop contrasté, Corporate clair) selon les
  réponses. Détecte logos (nodes "Logo couleur"/"Logo blanc") et images
  (nodes "IMG · slot") sur Page 1 du Figma cible, pas d'upload. Slots
  vides = placeholder dégradé + images-brief.md (prompts IA). CRÉATION :
  "scan ma charte", "crée le brand system", "j'ai un logo", "génère
  brand.md", "fais le brand book", "brand setup", "structure ma landing",
  "page de vente de la marque". BOOK (brand.md existe) : "brand
  guidelines propre", "brand book Figma", "transforme mes tokens en brand
  book". LANDING : "refais la landing", "structure ma page de vente",
  "génère landing-page.md". Contenu (LinkedIn, PPTX) sans brand.md :
  lancer CRÉATION d'abord.
---

# Skill : brand-book

Pipeline unique qui prend un logo + couleurs en entrée et produit toute la
chaîne de charte : `brand.md` (source de vérité) + `landing-page.md`
(prompt-brief de landing) + brochure Figma 15 slides. Toute la grammaire
de mise en page (positions x/y, sizing, fonts, fills) est **documentée
directement dans ce SKILL.md** — le skill est autonome et n'a pas de
dépendance externe.

**Conventions de dossier projet** (obligatoire — le skill scanne ces
emplacements au démarrage) :

```
projet/
├── brand.md                       ← écrit par le skill (source de vérité)
├── landing-page.md                ← écrit par le skill (prompt-brief de landing)
├── landing.html                   ← écrit par le skill (landing finale production-ready)
├── landing-content.txt            ← écrit par le skill (textes IA-générés éditables pour resync rapide)
├── description-services.txt       ← input utilisateur : promesse, persona, méthode, formule signature
├── infos-entreprise.txt           ← input utilisateur : SIRET, hébergeur, RGPD, URLs Calendly/LinkedIn
├── temoignages-clients.txt        ← input utilisateur : 3 verbatim clients + attributions
├── logo/                          ← exports SVG du logo, source de vérité pour la landing HTML
│   ├── logo.svg                    (variante couleur, fonds clairs, obligatoire)
│   └── logo-white.svg              (variante blanche, fonds sombres, obligatoire)
└── images/
    ├── hero.jpg                    (visuel principal du hero landing)
    ├── photo-profil.jpg            (photo About, portrait du fondateur ou main au travail)
    └── cas-clients/                ← optionnel : selected work / 3 projets phares
        ├── texte-cas-clients.txt    ← input utilisateur : titres, briefs, livraisons des 3 cas
        ├── cas1.png                 (cas client 1 — cover ou mockup)
        ├── cas2.png
        └── cas3.png
    ├── cas-clients/
    │   ├── cas1.png          (cover ou mockup du cas 1, obligatoire)
    │   ├── cas2.png
    │   ├── cas3.png
    │   ├── cas1.md           (optionnel : titre, secteur, année, descriptif)
    │   ├── cas2.md           (optionnel)
    │   └── cas3.md           (optionnel)
    └── ...
```

**Note** : le sous-dossier `./images/cas-clients/` n'est utilisé que si l'option
« Portfolio · 3 projets phares (selected work) » est cochée à l'étape
A.5.0 L2. Convention recommandée pour les graphistes, agences créa,
studios d'identité.

**Logos pour la landing HTML** : le dossier `./logo/` à la racine du projet
contient les SVG exportés depuis Figma. Ces fichiers sont **la seule source
de vérité** pour la génération HTML downstream (cf. règle « LOGO · règle
absolue » dans le bloc Prompt de `landing-page.md`). Si le dossier ou les
SVG manquent, la génération HTML doit être bloquée jusqu'à ce que les
exports soient déposés.

**Fichiers `.txt` sources — pré-consommation au démarrage** : 4 fichiers texte
peuvent être déposés par l'utilisateur à la racine du projet (et dans
`cas-clients/`). Ils contiennent les inputs structurés que le skill aurait
sinon collectés via les vagues d'interview. **Le skill scanne ces fichiers en
priorité (étape A.1.f)** et n'invoque `AskUserQuestion` que pour les variables
réellement manquantes. C'est ce qui permet à l'utilisateur préparé de générer
brand.md + landing.html **en un seul tour**, sans 5 vagues d'interview.

| Fichier | Rôle | Variables remplies |
|---|---|---|
| `description-services.txt` | Identité, promesse, persona, méthode, formule signature, objection #1, différenciateur, texte du CTA | Vagues 1, 2, 5 de l'interview · `LANDING_CTA_PRIMARY` |
| `infos-entreprise.txt` | Mentions légales, RGPD, hébergeur, URLs Calendly/LinkedIn, email contact | Étape A.5.0.quater intégralement (production-ready) |
| `temoignages-clients.txt` | 3 verbatim clients avec attribution (fonction, lieu, année) | Section Témoignages de la landing (plus de `[À COMPLÉTER]`) |
| `images/cas-clients/texte-cas-clients.txt` | 3 cas : titre, secteur, année, brief original, livraison | Section Portfolio · `PORTFOLIO_SOURCE = "files-ready"` |
| `landing-content.txt` *(écrit par le skill, éditable)* | Textes IA-générés non sourcés : eyebrows, H1/H2 de section, paragraphes intro, bullets, réponses FAQ Q2-Q5, sous-titres CTA | Édition légère sans relancer l'interview · mode SYNC (cf. ci-dessous) |

**Logos — source de vérité = le fichier Figma cible** (et non plus le
dossier `/logos/`). Les deux variantes du logo doivent être placées
**directement dans le fichier Figma cible**, sur Page 1 (ou une page
« Assets »), nommées exactement :

| Nom de node Figma | Rôle |
|---|---|
| `Logo couleur` | Variante couleur originale (fonds clairs) — **obligatoire** |
| `Logo blanc` | Variante blanche / cream (fonds sombres) — optionnelle |

**Why** : l'upload programmatique d'assets vers Figma via le MCP est
bloqué côté sandbox (HTTP 403 du proxy interne sur `mcp.figma.com`),
et l'encodage base64 de PNG dans un appel `use_figma` se corrompt en
transit. Le SVG inline via `figma.createNodeFromSvg()` fonctionne mais
ajoute ~7KB de payload par batch et impose une recoloration via
string-replace. Avoir les logos déjà placés dans Figma supprime toute
cette gymnastique : le skill se contente de faire `node.clone()` à chaque
placement (slides 01, 06, 15).

Le skill n'utilise **jamais** d'upload ponctuel : si un node logo manque
dans le Figma cible (ou si une image manque dans `./images/`), il
demande à l'utilisateur de le déposer avant de continuer.

**Images du brand book — même logique que les logos.** L'upload
programmatique étant bloqué (cf. Why ci-dessus), les photos qui habillent
les slides suivent la même convention : des nodes posés sur Page 1 du
Figma cible, nommés `IMG · <slot>` (ex : `IMG · cover`,
`IMG · chapitre-logo`, `IMG · univers-3`). Le skill les détecte en
A.1.a.bis et les clone dans les slides via le helper `placeImage()`
(Étape 5). **Aucun node image n'est obligatoire** : chaque slot absent
est rendu en placeholder dégradé aux couleurs de la marque (nommé
`MoodBoard · <slot>` dans Figma), prêt pour un drag-drop ultérieur, et
listé dans `images-brief.md` avec un prompt IA prêt à coller
(Midjourney / Imagen / Firefly). Le registre complet des slots est à
l'Étape 5 (« Registre des slots images »).

Le dossier local `./images/` reste réservé à la **landing HTML**
(hero.jpg, photo-profil.jpg, cas clients) — il n'alimente pas le brand
book Figma, faute d'upload possible.

---

## Détection du mode — 4 modes

Le skill détecte automatiquement quel mode lancer en fonction du contexte :

| Contexte | Mode | Action |
|---|---|---|
| `brand.md` absent + nodes logo détectés dans le Figma cible | **CRÉATION** | Phase A (interview 18 Q) → écrit `brand.md` → écrit `landing-page.md` (Étape A.5) → Phase B (brochure Figma) |
| `brand.md` présent + nouvelle URL Figma vierge | **BOOK** | Phase B uniquement (brochure depuis brand.md existant) |
| `brand.md` présent + trigger « refais la landing », « structure ma page de vente », « génère landing-page.md » | **LANDING** | Étape A.5 uniquement (regénère le `landing-page.md` depuis le `brand.md` existant — pas d'interview ni de Figma) |
| `landing.html` présent + `landing-content.txt` présent + trigger « resynchronise la landing », « mets à jour la landing depuis les txt », « regen depuis les txt », « sync content » | **SYNC** | Étape A.5.g uniquement (relit les 5 `.txt` et regénère `landing.html` sans aucune question ni interview — édition rapide du contenu) |

**En cas d'ambiguïté** (`brand.md` présent + URL Figma fournie sans trigger
explicite), poser une seule question :
> J'ai détecté : `brand.md` à la racine + URL Figma fournie. Tu veux :
> 1. Régénérer le brand book Figma à partir de brand.md
> 2. Regénérer la landing-page.md à partir de brand.md
> 3. Resynchroniser la landing.html depuis les .txt
> 4. Démarrer une nouvelle charte (écrase brand.md)

Ne **jamais** générer un brand book "à vide" — si `brand.md` est absent
ET qu'aucun node `Logo couleur` n'est trouvé dans le Figma cible, demander
d'abord à l'utilisateur de coller son logo dans le Figma + de fournir les
couleurs.

---

## ✦ Règles d'écriture transversales (applicables à TOUT contenu généré)

> Ces règles s'appliquent à **tous les textes produits par le skill** :
> `brand.md`, `landing-page.md`, libellés Figma (titres slides, eyebrows,
> labels cards, descriptions, citations, footers), et même les messages
> de récap envoyés à l'utilisateur dans la conversation. Aucune
> exception.

### Règle #1 — Zéro tiret cadratin `—` dans la copy

**Le tiret cadratin (U+2014, « — ») est INTERDIT** dans tous les
contenus que le skill génère. Raison : c'est l'un des **signaux IA les
plus reconnaissables** — un brand book ou une landing parsemée de tirets
cadratins se lit immédiatement comme « rédigé par ChatGPT » et perd sa
crédibilité haut de gamme.

Le tiret demi-cadratin `–` (U+2013) est **aussi banni** (même problème).

Seul le **trait d'union court `-`** (U+002D) est autorisé, et
uniquement dans son usage légitime : mots composés (« sans-serif »,
« B2B »), liaisons typographiques propres.

**Remplacements par contexte** :

| Cas d'usage du « — » | Remplacement recommandé |
|---|---|
| Définition / précision (« Word — definition ») | `:` deux-points : *« Word : definition »* |
| Apposition courte (« sobre — calme ») | virgule : *« sobre, calme »* |
| Liaison de deux idées (« A — et B ») | point + nouvelle phrase : *« A. Et B. »* |
| Insertion / incise (« A — pour autant — B ») | parenthèses : *« A (pour autant) B »* ou virgules : *« A, pour autant, B »* |
| Séparateur de chips / attributs (eyebrows Figma) | point médian ` · ` : *« TON · CONFIDENTIEL »* |
| Énumération inline | point-virgule : *« A ; B ; C »* |
| Citation manifeste | guillemets seuls, sans tiret introducteur |

**Exemples avant / après** :

| ❌ Avec tiret cadratin | ✅ Réécriture propre |
|---|---|
| *« Une méthode verticale — pas un studio généraliste. »* | *« Une méthode verticale. Pas un studio généraliste. »* |
| *« Espacement — White space généreux. »* | *« Espacement : white space généreux. »* |
| *« Confidentiel & premium — codes de la discrétion »* | *« Confidentiel & premium : codes de la discrétion »* |
| *« Architecture & passages — verticalité institutionnelle »* | *« Architecture & passages, verticalité institutionnelle »* |
| *« Brand book sur mesure — pas template générique »* | *« Brand book sur mesure. Pas un template générique. »* |

**Cas particulier — séparateurs structurels markdown** : le triple
tiret horizontal `---` (séparateur de section markdown) reste autorisé
**car c'est de la syntaxe markdown**, pas un caractère de copy.
Idem pour les `<!-- commentaires -->` techniques.

**Enforcement** :
- À chaque génération de copy (brand.md, landing-page.md, libellés
  Figma), faire un check `text.includes('—')` avant l'écriture. Si
  trouvé, remplacer immédiatement selon le mapping ci-dessus.
- Le lint sémantique A.5.f inclut une **vérification #6** explicite
  zéro tiret cadratin dans `landing-page.md`.
- Pour les libellés Figma, c'est l'agent lui-même qui doit s'auto-corriger
  avant d'envoyer le code `use_figma` (il n'y a pas de lint automatique
  côté Figma).

### Règle #2 — Pas d'emoji sauf demande explicite

Pas d'emoji dans le contenu généré (brand.md, landing-page.md, libellés
Figma). Le seul caractère décoratif autorisé est le **`✦`** utilisé
dans les blocs de récap conversationnel (pas dans les livrables).

### Règle #3 — Pas de « voici / voilà » introducteurs

Bannir les formules introductives plates : « Voici la slide », « Voilà
le résultat », « Comme vous pouvez le voir ». Aller directement au
contenu. Si une introduction est nécessaire dans un récap, utiliser
une affirmation : *« Brand book installé. »*, *« Slide 04 régénérée. »*.

---

# Phase A — Mode CRÉATION (interview + brand.md)

> Cette phase ne s'exécute qu'en mode CRÉATION. En mode BOOK, sauter
> directement à la Phase B.

## A.1 — Scanner le contexte projet (logos Figma + images + couleurs)

**Avant tout** : scanner les sources pour récupérer les assets. Le skill
ne demande **jamais** un upload ponctuel — les logos viennent du fichier
Figma cible, les images du dossier `./images/`.

### A.1.a — Détection des logos dans le Figma cible

Appeler `get_metadata` sur le fichier Figma cible (Page 1) et chercher
deux nodes par leur **nom exact** :

```js
// Via use_figma au démarrage
const page = figma.root.children[0]; // Page 1
const logoColorNode = page.findOne(n => n.name === "Logo couleur");
const logoWhiteNode = page.findOne(n => n.name === "Logo blanc");
```

**Tolérance de matching** (insensible à la casse, espaces ignorés) :

| Nom attendu | Rôle | Variantes acceptées |
|---|---|---|
| `Logo couleur` | Variante couleur (fonds clairs) — **obligatoire** | `logo couleur`, `Logo Couleur`, `logo-couleur`, `Logo color`, `Logo` (fallback) |
| `Logo blanc` | Variante blanche/cream (fonds sombres) — optionnelle | `logo blanc`, `Logo Blanc`, `logo-blanc`, `Logo white`, `Logo cream` |

Cas possibles :

| Cas | Action |
|---|---|
| Les 2 nodes présents | OK — passer à A.1.b |
| Seul `Logo couleur` présent | OK — utiliser le node couleur pour les fonds clairs, **signaler** que la variante blanche est manquante. Sur les fonds sombres (slides 01, 15, chapter openers), afficher le logo couleur avec un encart `[Variante blanche à produire — voir brand.md > Logo]` en bord de slide. **Ne jamais re-colorer automatiquement** un node Figma. |
| Seul `Logo blanc` présent | Inverse — utiliser le blanc partout, signaler que la version couleur manque pour les fonds clairs. |
| Aucun node logo | **Bloquer la génération** et demander à l'utilisateur de coller son logo SVG dans le Figma cible (Page 1) et de le nommer `Logo couleur`. |
| Fichier Figma totalement vide | Stopper et expliquer : « Le fichier Figma cible est vide. Colle ton logo (au moins la variante couleur) sur Page 1 avant de relancer le skill. »  |

### A.1.a.bis — Détection des images `IMG · <slot>` dans le Figma cible

Dans le même appel que la détection des logos, scanner Page 1 (et une
éventuelle page « Assets ») pour les nodes images déposés par
l'utilisateur :

```js
// Convention : "IMG · cover", "IMG · univers-1", "img-chapitre-logo"…
const IMG_RE = /^img[\s·:\-]+(.+)$/i;
const IMG_TPLS = {};
for (const pg of [homePage, figma.root.children.find(p => /assets/i.test(p.name))]) {
  if (!pg) continue;
  pg.children.forEach(n => {
    const m = n.name.trim().match(IMG_RE);
    if (m) IMG_TPLS[m[1].trim().toLowerCase()] = n;
  });
}
```

Chaque clé de `IMG_TPLS` correspond à un slot du « Registre des slots
images » (Étape 5). Les slots non couverts ne bloquent **jamais** la
génération : ils sont rendus en placeholder dégradé et documentés dans
`images-brief.md` (Étape 9ter). Annoncer le résultat du scan au
démarrage, en une ligne :

```
✦ Images détectées dans le Figma : 4/16 slots (cover, identite, univers-1, univers-2)
  Les 12 autres seront générés en placeholders + images-brief.md
```

### A.1.b — Scan des images

Lister le contenu de `./images/` :

```bash
ls -la ./images/
```

Stocker la liste des fichiers en mémoire dans la variable `AVAILABLE_IMAGES`
sous forme `[{ filename, ext, size, path }]`. Si le dossier est vide ou
inexistant, **ne pas bloquer** — la landing-page.md utilisera des
placeholders explicites `[image manquante : /images/hero.jpg à déposer]`
et le brand book Figma sera généré sans visuels secondaires (la slide 04
Universe & Mood reste générique).

### A.1.c — Collecter les inputs conversationnels restants

Une fois les dossiers scannés, vérifier dans la conversation :

| Input | Obligatoire | Source par défaut |
|---|---|---|
| **Nom de la marque** | **Oui — bloquant** | Brief / conversation / `description-services.txt` / titre de `brand.md`. Si absent : **demander en intro, avant la Vague 1** |
| Couleur(s) principale(s) | Oui (min. 1 hex) | Demander si absent, ou extraire visuellement depuis le node `Logo couleur` via `get_screenshot` (rendu Figma natif) |
| Positionnement / cible court | Recommandé | Brief, profil LinkedIn, conversation |

Si nom ou couleurs manquent, poser **une seule question groupée** pour
tout récupérer d'un coup, **avant de lancer l'interview**.

**Le nom de la marque est une variable, jamais un exemple.** Stocker la
réponse dans `BRAND_NAME` : c'est elle qui alimente le titre cover, le
footer des 15 slides, les crédits de la slide 15, le titre de `brand.md`
et de `images-brief.md`. Les noms qui apparaissent dans ce SKILL.md
(fokusia, Max Piccinini, PICCININI) sont des **exemples de documentation**
: si l'un d'eux se retrouve dans un livrable, c'est un bug (cf.
guardrail ⑰). En mode BOOK, `BRAND_NAME` se lit dans le titre
`# Brand Identity — [Nom]` de `brand.md` ; s'il est introuvable, le
demander avant de générer la moindre slide.

### A.1.d — Préparer les templates logo pour clonage

Au démarrage de la Phase B (Batch 1), localiser les nodes logo sur la
page courante du Figma cible et les stocker comme références templates.
**Aucun upload, aucune création d'image, aucun base64** — on clonera
directement ces nodes dans chaque slide qui en a besoin :

```js
// Au début du Batch 1 (use_figma), après setCurrentPageAsync :
const homePage = figma.root.children[0]; // ou la page où l'utilisateur a placé les logos
const tplColor = homePage.findOne(n => /^logo[\s-]?couleur$/i.test(n.name))
                 || homePage.findOne(n => /^logo$/i.test(n.name));
const tplWhite = homePage.findOne(n => /^logo[\s-]?(blanc|white|cream)$/i.test(n.name));

if (!tplColor) {
  throw new Error(
    "Aucun node 'Logo couleur' trouvé dans le Figma. Colle ton logo SVG " +
    "sur Page 1 et nomme-le 'Logo couleur' avant de relancer le skill."
  );
}

// Variables d'état à stocker au scope du script avant l'Étape 5 :
const LOGO_SOURCE = "figma-node"; // toujours, depuis la convention Figma
const LOGOS = {
  color: tplColor,
  white: tplWhite, // peut être null
};
```

À l'Étape 5 (helper `placeLogo()`), choisir la variante selon le fond :

| Emplacement | Variante utilisée |
|---|---|
| Slide 01 Cover (fond sombre primary) | `LOGOS.white` si dispo, sinon `LOGOS.color` |
| Slide 06 Content Logo (fond clair) | `LOGOS.color` |
| Slide 15 4e couverture (fond sombre) | `LOGOS.white` si dispo, sinon `LOGOS.color` |
| Chapter openers (fond sombre) | `LOGOS.white` si dispo, sinon `LOGOS.color` |

**Note** : les nodes logo doivent rester sur la page (ne pas les supprimer
au cleanup). Le helper `placeLogo()` fait `tpl.clone()` à chaque appel —
les clones sont indépendants, donc on peut les resize, repositionner et
les insérer dans chaque slide sans casser l'original.

**Règle stricte** : on **n'utilise jamais le texte "Logo" 96px** comme
placeholder. Le brand book affiche **toujours le vrai logo de la marque**
— sinon le livrable n'a aucune valeur de présentation.

### A.1.e — Préparer les images pour la landing (pas pour Figma)

Les images de `AVAILABLE_IMAGES` servent **uniquement à la landing**.
Ne pas tenter `upload_assets` vers Figma : l'upload est bloqué côté
sandbox (HTTP 403, cf. « Why » en tête de skill) et l'échec silencieux
coûte un appel + un timeout. **Usage dans le brand book Figma** : aucun.
Les images du book passent exclusivement par les nodes `IMG · <slot>`
détectés en A.1.a.bis, sinon placeholders dégradés.

**Usage dans `landing-page.md`** : voir l'Étape A.5.c.bis ci-dessous —
chaque image dispo est référencée **par son chemin** (`./images/hero.jpg`)
dans le brief visuel de la section correspondante. Pas de placeholder
abstrait si l'image existe — le brief dit explicitement *« Image fournie :
`./images/hero.jpg` — utiliser telle quelle »*.

### A.1.f — Pré-consommation des fichiers `.txt` sources

**Étape critique** : avant de lancer la moindre vague d'interview, le skill
scanne les 4 fichiers `.txt` que l'utilisateur peut avoir préparés. Si
présents et remplis, leur contenu **alimente directement les variables
internes** et **les questions correspondantes sont skippées** dans A.2 et
A.5.0.quater.

**Détection** :

```bash
ls -la \
  ./description-services.txt \
  ./infos-entreprise.txt \
  ./temoignages-clients.txt \
  ./images/cas-clients/texte-cas-clients.txt \
  2>/dev/null
```

Pour chaque fichier détecté, le skill le lit, ignore les lignes commençant
par `#` (commentaires), parse les sections `## Nom` et les paires
`Champ : valeur`, puis remplit les variables correspondantes :

#### `description-services.txt` → vagues 1, 2, 5 + CTA

Mapping des sections vers les réponses d'interview :

| Section `.txt` | Variable interne | Vague interview skippée |
|---|---|---|
| `## Qui je suis` | `IDENTITY_BLURB` | Pour bloc Positionnement de brand.md |
| `## Pour qui` | `PERSONA_PRINCIPAL` | Q15 (Vague 5) |
| `## Promesse principale` | `PROMESSE` | Q4 (Vague 1) |
| `## Formule signature` | `SIGNATURE_PHRASE` | Bloc Identité verbale de brand.md |
| `## Différenciateur` | `DIFFERENCIATEUR` | Q17 (Vague 5) |
| `## Objection #1` | `OBJECTION_1` | Q16 (Vague 5) |
| `## Méthode / process (3 étapes)` | `METHODE_STEPS[3]` | Section Méthode de la landing |
| `## CTA principal (texte du bouton)` | `LANDING_CTA_TEXT` | Texte exact des CTAs |

Si toutes ces sections sont remplies, **Vague 5 entièrement skippée** et
Vague 1 réduite à Q1, Q2, Q3 (ton, formalité, posture — pas dans le .txt).

#### `infos-entreprise.txt` → étape A.5.0.quater entière

Mapping des champs vers les variables production-ready :

| Champ `.txt` | Variable interne |
|---|---|
| `URL Calendly` | `CTA_PRIMARY_URL` (ou fallback mailto avec subject pré-rempli) |
| `Email contact (footer)` | `CONTACT_EMAIL` |
| `URL LinkedIn perso` | `LINKEDIN_URL_PERSO` |
| `URL LinkedIn entreprise` | `LINKEDIN_URL_BRAND` |
| Bloc `## Éditeur` complet | Contenu de l'accordion `#mentions-legales` |
| Bloc `## Hébergeur` complet | Section hébergement de l'accordion mentions |
| Bloc `## RGPD` complet | Contenu de l'accordion `#confidentialite` |

Si tous les champs obligatoires sont remplis (cf. règle LCEN+RGPD) :
**A.5.0.quater entièrement skippée**, accordions générés sans aucun
placeholder rouge dashed `<span class="to-fill">`.

Si certains champs sont vides : ces champs spécifiques restent en
placeholder rouge dashed dans la landing, le reste est rempli. L'utilisateur
peut compléter le `.txt` plus tard et regénérer.

#### `temoignages-clients.txt` → section Témoignages

Mapping :

| Section `.txt` | Variable interne |
|---|---|
| `## Témoignage 1` (Citation, Fonction, Lieu, Année) | `TESTIMONIAL_1 = {quote, role, place, year}` |
| `## Témoignage 2` | `TESTIMONIAL_2` |
| `## Témoignage 3` | `TESTIMONIAL_3` |

Si les 3 témoignages sont remplis, la section Témoignages de la landing
contient directement les verbatim, sans `[À COMPLÉTER]`. Si moins de 3
sont fournis, les manquants restent en placeholder visible.

#### `images/cas-clients/texte-cas-clients.txt` → section Portfolio

Mapping :

| Section `.txt` | Variable interne |
|---|---|
| `## Cas 1` (Titre, Secteur, Année, Brief original, Ce qui a été livré) | `CASE_1 = {title, sector, year, brief, delivered}` |
| `## Cas 2` | `CASE_2` |
| `## Cas 3` | `CASE_3` |

Si présent et `cas1.png/cas2.png/cas3.png` aussi présents, alors
**`PORTFOLIO_SOURCE = "files-ready"`** (cf. A.5.0.ter) et la section
Portfolio est générée avec données réelles, sans marqueurs `(INVENTÉ)`.

#### Priorité des sources

Pour chaque variable interne, le skill applique cette priorité décroissante :

1. **Fichier `.txt`** (si présent et champ rempli) — source primaire.
2. **Réponse `AskUserQuestion`** (si vague d'interview lancée et utilisateur a répondu).
3. **Inférence depuis `brand.md` ou contexte conversationnel** (fallback intelligent).
4. **Valeur par défaut documentée** (dernier recours).

#### Annonce de pré-consommation

Si au moins un `.txt` est trouvé et exploitable, le skill annonce d'emblée
au démarrage :

```
✦ Pré-consommation détectée
  · description-services.txt : Vagues 1, 2, 5 préchargées
  · infos-entreprise.txt : A.5.0.quater entièrement préchargée (15/22 champs remplis)
  · temoignages-clients.txt : 3/3 témoignages
  · texte-cas-clients.txt : 3/3 cas

Interview réduite à : Vague 3 (typo), Vague 4 (univers).
Estimation : 7 questions au lieu de 22.
```

Cette annonce **rassure l'utilisateur** sur le fait que son effort de
préparation n'a pas été ignoré, et donne une visibilité concrète sur les
vagues qui restent à exécuter.

---

## A.2 — Interview d'identité (5 vagues, 18 questions)

> **Note importante** : si l'étape A.1.f a détecté un ou plusieurs `.txt`
> sources remplis, **certaines vagues ou questions de cette interview sont
> skippées**. Voir le tableau de mapping en A.1.f. Le skill ne pose **jamais**
> deux fois la même question : si la réponse est déjà dans un `.txt`, il
> passe à la suivante.

Une fois le logo + couleurs reçus, **ne génère rien tout de suite**. Lance
5 vagues `AskUserQuestion` (3-4 questions par vague, choix multiples avec
option « Autre » pour réponse libre). Les réponses servent à remplir toutes
les sections riches de `brand.md` (Identité verbale, Univers, Style
graphique, Marché & persona) — qui seront ensuite **visibles** dans le
brand book Figma (slides 03 Verbal Identity et 04 Universe & Mood) et
qui nourrissent **directement la copy de la landing** (Vague 5 surtout).

> Annonce avant la vague 1 : « Avant de générer la charte, j'ai 18
> questions pour calibrer l'identité verbale, l'univers visuel et le
> persona. Réparties en 5 vagues. ~4 minutes. »

### A.2.bis — Vocabulaire vivant (input optionnel — recommandé)

**Avant ou juste après la Vague 5**, proposer à l'utilisateur un input libre
qui change radicalement la qualité de la copy de la landing :

> Optionnel mais très recommandé : colle ici **3 à 5 messages que tes
> prospects t'envoient** (DM LinkedIn, mails entrants, retours en RDV).
> Ou 3-5 phrases que tu as lues sur leurs profils. Je vais en extraire
> le **vocabulaire vivant** — les vrais mots qu'ils utilisent pour
> décrire leur problème et leur désir. Ça nourrit directement la liste
> « Mots à utiliser » de ton brand.md.

Si l'utilisateur fournit ces messages :
1. Extraire 8-15 mots/expressions vivants (verbes d'action, noms
   concrets, formulations récurrentes — pas les mots vides).
2. Les insérer dans **« Mots à utiliser »** de `brand.md` avec un
   préfixe `(vivant)` pour les distinguer des mots dérivés du
   positionnement.
3. Repérer les mots **anxiogènes** ou **buzzword** qui reviennent
   dans les messages et les ajouter à **« Mots à éviter »** s'ils
   contredisent Q7.

Si l'utilisateur passe → continuer sans, mais signaler dans le brand.md :
`> Mots à utiliser : dérivés du positionnement (pas de vocabulaire vivant
fourni — à enrichir plus tard depuis des vrais messages prospects).`

### Vague 1 — Positionnement & Ton (4 questions)

| # | Question | Options |
|---|---|---|
| 1 | Quel **ton dominant** doit porter la marque ? | Confidentiel & premium · Direct & tranchant · Chaleureux & humain · Technique & expert |
| 2 | Quel **niveau de formalité** ? | Tutoiement décontracté · Vous professionnel · Hybride contextuel · Très formel (institutionnel) |
| 3 | Quelle **posture face au client** ? | Partenaire stratégique · Expert technique · Coach inspirant · Exécutant fiable |
| 4 | Quelle est la **promesse principale** ? | Gagner du temps · Gagner en clarté · Gagner en performance · Gagner en confiance |

### Vague 2 — Identité verbale (3 questions)

| # | Question | Options |
|---|---|---|
| 5 | Style préféré des **accroches / titres** ? | Affirmation tranchée ("Clarté. Méthode. Résultats.") · Question ouverte ("Et si vous arrêtiez de…?") · Métaphore évocatrice ("Du bruit à l'essentiel") · Chiffre choc ("3× plus de clarté en 90 jours") |
| 6 | Style des **CTAs** ? | Verbe court ("Commencer") · Phrase complète ("Réservez votre diagnostic") · Question engageante ("Prêt à passer à l'action ?") · Bénéfice direct ("Gagnez en clarté") |
| 7 | **Vocabulaire à éviter** (multiSelect) ? | Buzzwords corp · Jargon technique inutile · Familiarités forcées · Mots négatifs / anxiogènes |

### Vague 3 — Style typographique & couleurs (4 questions)

| # | Question | Options |
|---|---|---|
| 8 | **Personnalité typographique** ? | Géométrique moderne (Inter, Helvetica) · Serif éditorial (Playfair, Cormorant) · Sans-serif premium (Manrope, General Sans) · Display dégaussé (Editorial New, PP Mori) |
| 9 | **Échelle** ? | Contraste fort (H1 56px / Body 16px) · Mesurée (H1 40px / Body 18px) · Dramatique (H1 96px+ / Body 18px) · Subtile (uniforme) |
| 10 | **Usage de l'accent couleur** ? | Chirurgical (< 10%, CTA uniquement) · Mesuré (10-20%) · Présent (20-30%) · Pleine couleur (full bleed) |
| 11 | **Ambiance lumineuse** dominante ? | Sombre & dramatique · Clair & aéré · Crème & chaleureux · Contrasté noir/blanc |

### Vague 4 — Univers visuel (4 questions)

| # | Question | Options |
|---|---|---|
| 12 | **Référence d'univers** ? | Architecture & passages · Nature & matières brutes · Tech minimale & grilles · Éditorial luxe & mode |
| 13 | **Style d'imagerie** ? | Photos documentaire · Illustrations vectorielles · Duotone / monochrome · Pas d'imagerie — typographie pure |
| 14 | **Personnalité graphique** en mots-clés (multiSelect) ? | Sobre · Audacieux · Premium · Chaleureux · Technique · Émotionnel |
| 14bis | **Style d'icônes & d'illustrations** ? | Outline fin (Lucide / Phosphor light, stroke 1.5) · Duotone (Phosphor duotone, Tabler filled) · Solid plein (Heroicons solid) · 3D / isométrique (Blender, illustration custom) · Aucune icône — typographie + photo seulement |

### Vague 5 — Marché & persona (3 questions)

> Cette vague nourrit le différenciateur du brand.md, la FAQ de la landing
> et la calibration de la copy au persona réel. Sans elle, la landing reste
> générique. Si l'utilisateur ne sait pas répondre, autoriser « Je verrai
> plus tard » — mais le signaler explicitement comme dette dans le brand.md.

| # | Question | Options |
|---|---|---|
| 15 | **Persona principal** (profil dominant du prospect type) ? | Dirigeant solo (CEO / founder) · Lead opérationnel (Head of, Product, Design, Marketing) · Indépendant / freelance · Équipe technique (Dev, Data, IT) |
| 16 | **Objection #1** que tu entends le plus avant un achat ? | « C'est cher » (prix) · « J'ai pas le temps » (effort) · « Je sais pas si ça marche pour mon cas » (preuve) · « Je peux le faire en interne » (DIY) |
| 17 | **Différenciateur** vs les alternatives existantes (concurrents directs ou DIY) ? | Méthode unique / framework propriétaire · Vitesse d'exécution · Niveau d'expertise / spécialisation · Posture / relation (proximité, accompagnement) · Tarif (positionnement prix) |

> **Important** : ne fais **pas** d'hypothèses sur les options choisies.
> Toutes les réponses doivent être consignées **mot pour mot** dans les
> sections de `brand.md`.

---

## A.3 — Extraire les tokens depuis le logo

### Couleurs
- **Primaire** : couleur dominante / fond / structure
- **Accent** : couleur d'appel à l'action (souvent plus chaude / saturée)
- **Neutres** : blanc + neutre chaud + texte foncé — dériver depuis les
  couleurs extraites si non fournies
- Règle : l'accent suit le ratio choisi en Q10

### Typographie
En l'absence de specs explicites, choisir la paire selon Q8 :
- Géométrique moderne → Inter Bold (titres) + Inter (corps)
- Serif éditorial → Playfair Display (titres) + Inter (corps)
- Sans-serif premium → Manrope ou General Sans
- Display dégaussé → Editorial New ou PP Mori

### Échelle, spacing, radius
Suivre le standard 8pt sauf indication contraire :
- spacing : `xs:4 / sm:8 / md:16 / lg:24 / xl:40 / 2xl:64`
- radius : `sm:4 / md:8 / lg:16 / full:999`
- échelle typo : selon Q9 (Contraste fort / Mesurée / Dramatique / Subtile)

---

## A.4 — Écrire `brand.md` (fichier unique)

Écrire le fichier `brand.md` dans le dossier projet (ou dans un sous-dossier
si l'utilisateur le précise). Template complet :

```markdown
# Brand Identity — [Nom de la marque]

> Source de vérité unique. Toute modification (couleur, typo, ton) se fait ici.

## Positionnement
**Qui** : ...
**Pour qui** : [Q15 — persona principal, étoffé en 1-2 lignes : situation actuelle, déclic d'achat]
**Promesse** : [Q4]
**Différenciateur** : [Q17 — formulé en 1 phrase actionnable, pas un adjectif vague]

**Alternatives existantes** : [liste des concurrents directs cités + le DIY]
**Objection #1** : [Q16 — verbatim brut tel que le prospect le dit]

---

## Identité verbale

**Ton dominant** : [Q1]
**Niveau de formalité** : [Q2]
**Posture client** : [Q3]
**Style d'accroches** : [Q5]
**Style de CTAs** : [Q6]

**Formule signature** : "..." (proposer une formulation cohérente avec Q1+Q5)
**Mots à utiliser** : [issus du brief + du positionnement + vocabulaire vivant (A.2.bis)]
**Mots à éviter** : [issus de Q7 + termes anxiogènes/buzzword détectés en A.2.bis]

**CTAs types** (3 exemples cohérents avec Q6) :
- "..."
- "..."
- "..."

### Tone of voice — « On dirait / On ne dirait pas »

> 3 paires de phrases sur le **même message**, pour rendre le ton
> reproductible. Générer ces exemples à partir de Q1 + Q2 + Q5.

| Message | On dirait ✅ | On ne dirait pas ❌ |
|---|---|---|
| Inviter à un premier RDV | "..." | "..." |
| Annoncer un résultat client | "..." | "..." |
| Présenter la méthode | "..." | "..." |

---

## Logo
**Type** : Logotype / Symbole / Monogramme
**Variantes** : Couleur / Noir / Blanc
**Zone de protection** : 1× hauteur du logo de chaque côté
**Fonds recommandés** : ...

---

## Couleurs

| Rôle | Nom | Hex | Usage |
|------|-----|-----|-------|
| Primaire | [nom] | `#XXXXXX` | Logo, titres, fond sombre |
| Primaire dark | [nom] | `#XXXXXX` | Fond full-bleed, darkmode |
| Accent | [nom] | `#XXXXXX` | CTAs, highlights, chiffres clés |
| Atmosphérique | [nom] | `#XXXXXX` | Sections moodboard, ambiances |
| Neutre | Blanc | `#FFFFFF` | Fond principal |
| Neutre chaud | [nom] | `#XXXXXX` | Fond alternatif |
| Texte | [nom] | `#XXXXXX` | Corps de texte sur clair |

---

## Typographie

### Familles

| Rôle | Famille | Graisses | Usage |
|------|---------|----------|-------|
| Display | [Q8] | 600, 700 | Titres, hero |
| Body | [paire de Q8] | 400, 500 | Corps de texte |

### Échelle (selon Q9)

| Style | Taille | Graisse | Famille | Line-height |
|-------|--------|---------|---------|-------------|
| H1 | [Q9] px | 700 | display | 1.1 |
| H2 | ... | 600 | display | 1.2 |
| H3 | 24px | 500 | body | 1.3 |
| Body | [Q9] px | 400 | body | 1.6 |
| Small | 13px | 500 | body | 1.4 |

---

## Espacements

| Token | Valeur | Usage |
|-------|--------|-------|
| xs | 4px | Détails fins |
| sm | 8px | Inline entre éléments |
| md | 16px | Padding standard de card |
| lg | 24px | Padding minimum section |
| xl | 40px | Marges section standard |
| 2xl | 64px | Sections hero / full-bleed |

---

## Border radius

| Token | Valeur | Usage |
|-------|--------|-------|
| sm | 4px | Tags, inputs |
| md | 8px | Boutons, cards |
| lg | 16px | Cards premium, modales |
| full | 999px | Pills, avatars |

---

## Composition rules

(6 règles courtes — utilisées telles quelles par la slide 14 Key Elements.)

- **Spacing** — ...
- **Contrast** — ...
- **Accentuation** — Accent [Q10], réservé aux CTA
- **Typeface** — Titres en [Q8], corps en [paire Q8]
- **Padding** — Minimum lg (24px), sections hero xl (64px)
- **Atmosphere** — ...

---

## Univers visuel

**Ambiance lumineuse** : [Q11]
**Référence d'univers** : [Q12]
**Style d'imagerie** : [Q13]
**Personnalité graphique en mots-clés** : [Q14]
**Preset visuel** : [sélectionné en Phase B Étape 4bis : Luxe éditorial / Minimal tech / Crème organique / Pop contrasté / Corporate clair]

**Moodboard** :
[3-5 lignes décrivant l'univers visuel construit depuis Q11→Q14,
utilisable comme brief pour Gemini/Midjourney/Imagen.]

---

## Icônes & illustrations

**Style d'icônes** : [Q14bis]
**Librairie recommandée** : [déduite de Q14bis — ex : Lucide, Phosphor, Tabler, Heroicons, custom]
**Stroke / weight** : [ex : 1.5px outline · duotone 2 layers · solid filled]
**Corner radius des icônes** : [aligné avec radius.sm du DS — ex : 2px]

**Style d'illustration** : [déduit de Q13 + Q14bis — ex : 3D isométrique, line art éditorial, photo documentaire duotone]
**Usage** : [où elles apparaissent — hero, section process, section features, etc.]
**Palette d'illustration** : [restreinte aux tokens du DS — pas de couleur hors palette]

**Règles d'or** :
- Toutes les icônes proviennent de **la même librairie** (pas de mix Lucide + Material).
- Les icônes ne portent **pas de couleur d'accent** sauf indication explicite (cohérence visuelle).
- Pas d'icône décorative dans le hero — l'icône doit toujours porter une **information** (statut, action, catégorie).

---

## Personnalité graphique

1. ...
2. ...
3. ...
```

Une fois `brand.md` écrit, **enchaîner sur l'Étape A.5**. La toute
première sous-étape A.5.0 pose 4 questions sur la **structure
commerciale** de la landing (longueur, sections, CTA, réassurance) —
parce que ces choix sont distincts du calibrage d'identité, mais
indissociables de l'écriture de la landing.

---

## A.5 — Écrire `landing-page.md` (structure + copy de la landing)

> Étape obligatoire en mode CRÉATION. Aussi point d'entrée du mode LANDING
> (où on saute A.1→A.4 car `brand.md` existe déjà).

**Objectif** : à partir du `brand.md` qu'on vient d'écrire (ou qui existe
déjà en mode LANDING) **et des réponses A.5.0**, produire un fichier
`landing-page.md` qui contient :
- La **structure** ajustée aux réponses L1-L4 (longueur, sections
  obligatoires, CTA primaire, éléments de réassurance)
- La **copy déjà rédigée** dans le ton de la marque (H1, sous-titres, CTAs,
  bullets, FAQ, etc.) — pas des placeholders vides, du texte vraiment écrit
- Des **briefs visuels** par section (type de visuel, fond, disposition)
- Des **placeholders explicites** uniquement là où l'utilisateur doit fournir
  une donnée externe (témoignages, logos clients, chiffres de résultats)

Le fichier sert ensuite de brief structuré pour générer la landing en HTML,
Framer, Webflow ou autre — dans un 2ème temps (hors scope de cette skill).

### A.5.0 — Interview landing-page (4 questions sur la structure commerciale)

> **Première sous-étape obligatoire de A.5**. Pose 4 questions sur la
> structure commerciale de la landing AVANT toute écriture. Sans cette
> intro, la landing tombe dans le générique « template A par défaut » et
> ne reflète pas les contraintes commerciales réelles de l'utilisateur
> (lead magnet existant ? cycle de vente long ? besoin de réassurance
> forte ?).

**Annonce avant la vague** : « Avant d'écrire la landing, j'ai 4 questions
pour calibrer sa structure (longueur, sections, CTA, réassurance). ~1
minute. »

Lancer une vague unique `AskUserQuestion` (4 questions) :

| # | Question | Options |
|---|---|---|
| L1 | **Longueur de la landing** ? | Courte — hero + 3-4 sections + CTA (vente directe, persona décidé) · Moyenne — hero + 5-7 sections + FAQ + CTA *(défaut)* · Longue — hero + 8-10 sections + cas client + objections + FAQ + CTA · Sur mesure (préciser) |
| L2 | **Sections obligatoires** (multiSelect) ? | Témoignages clients · Logos clients · Cas client détaillé · **Portfolio · 3 projets phares (selected work)** *(défaut pour graphistes / agences créa)* · FAQ *(défaut)* · À propos / About · Process / Méthode · Garantie / Promesse · Vidéo de présentation · Calendly intégré · Comparaison vs alternatives |
| L3 | **CTA primaire** ? | Diagnostic / audit gratuit *(défaut B2B premium)* · Démo produit · Téléchargement lead magnet · Inscription newsletter · Contact / devis · Achat direct |
| L4 | **Éléments de réassurance** (multiSelect) ? | Chiffres clés *(X clients / Y résultats)* · Logos clients · Certifications · Témoignages texte · Témoignages vidéo · Awards / mentions presse · Garantie satisfait/remboursé · Aucun — la copy suffit |

**Optionnel — Q L5** : si l'utilisateur a sélectionné « Téléchargement lead
magnet » en L3, poser une seule question additionnelle (verbatim) :

> Quel lead magnet ? (Guide PDF, audit gratuit, checklist, webinar,
> template, démo recording, autre — préciser le titre et l'angle)

**Important** : ne pas chaîner sur une autre vague. Une seule passe de
4 questions, puis on écrit. Si l'utilisateur ne sait pas répondre,
prendre les défauts marqués « *(défaut)* » et signaler les choix par
défaut dans le **header de `landing-page.md`** (cf. A.5.d).

**Mémoriser les réponses** dans le contexte courant :
- `LANDING_LENGTH` : "courte" / "moyenne" / "longue" / "sur-mesure"
- `LANDING_SECTIONS_INCLUDE` : tableau des sections cochées en L2
- `LANDING_CTA_PRIMARY` : valeur L3
- `LANDING_TRUST_ELEMENTS` : tableau des éléments cochés en L4
- `LANDING_LEAD_MAGNET` : valeur L5 ou null

Ces 5 variables **pilotent l'Étape A.5.a (mapping → structure)** et
**doivent apparaître dans le header documentaire de `landing-page.md`**
(cf. A.5.d, bloc « Décisions structure ») pour que le fichier soit
auto-documenté — n'importe qui qui le rouvre 6 mois plus tard sait
pourquoi telle section a été retenue ou écartée.

### A.5.0.ter — Portfolio · 3 projets phares (conditionnel)

> **Sous-étape conditionnelle** — déclenche **uniquement si** L2 inclut
> « Portfolio · 3 projets phares (selected work) ». Sinon, sauter cette
> étape entièrement et passer à A.5.0.bis.

**Pourquoi cette étape existe** : pour les graphistes, les agences créa,
les studios d'identité, **la section selected work est le cœur du
commerce** — c'est ce qui transforme un visiteur en prospect. Une
landing de graphiste sans portfolio crédible ne convertit pas. On ne
peut pas laisser cette section au générique « cas client à compléter ».

**Pré-scan** : avant de poser P1, scanner le dossier projet pour
détecter si des projets ont déjà été déposés :

```bash
ls -la ./images/cas-clients/ 2>/dev/null
```

**Convention de dossier attendue** (à documenter à l'utilisateur si
l'option « Oui — fichiers prêts » est choisie) :

```
projet/
└── images/
    └── cas-clients/
        ├── cas1.png           (image cover du cas 1 — obligatoire si présent)
        ├── cas2.png
        ├── cas3.png
        └── casN.md            (optionnel : titre, secteur, année, descriptif)
```

Détecter `HAS_PROJECTS = true` si au moins 1 fichier `casN.png`
`./images/cas-clients/casN.png` existe au moins pour cas1. Mémoriser
`PROJECTS_FOUND = [{ slug, cover_path, details_md_path? }]`.

**Annonce avant la vague** : « Tu veux mettre en avant 3 projets phares.
3 questions courtes sur cette section. »

Lancer une vague unique `AskUserQuestion` (3 questions) :

| # | Question | Options |
|---|---|---|
| P1 | **Tu as déjà 3 projets prêts à montrer ?** | Oui — photos et infos déposées dans `./images/cas-clients/` *(si HAS_PROJECTS, devient défaut visible)* · Oui — j'ai les photos mais pas les infos (on génère le copy autour) · Non — propose 3 placeholders cohérents avec brand.md *(défaut si pas de dossier cas-clients/)* · Je verrai plus tard — placeholder visible « [Cas N à fournir] » |
| P2 | **Style de présentation du portfolio** ? | Grille 3 colonnes (vue d'ensemble équivalente, lecture rapide) · Stack vertical (1 projet par scroll, immersif, défaut éditorial) · Carousel horizontal (swipe latéral, mobile-first) · Tabs par catégorie (DA · packaging · digital · etc.) |
| P3 | **Profondeur d'information par projet** ? | Mockup + titre + 1 ligne descriptive *(défaut sobre)* · Mockup + titre + secteur client + année · Mockup + titre + brief original + ce qui a été livré · Mockup + titre + brief + livraison + résultat chiffré (si dispo) |

**Important** :
- Si P1 = « Oui — fichiers prêts » mais `HAS_PROJECTS = false` →
  signaler immédiatement « Je ne trouve pas de dossier `./images/cas-clients/`.
  Dépose tes fichiers selon la convention décrite, puis relance.
  Sinon, je passe en mode placeholder. »
- Si P1 = « Non — propose 3 placeholders » → l'Étape A.5.c **invente
  3 projets-types** cohérents avec `brand.md` (secteur, persona, style)
  et écrit du contenu placeholder réaliste. Exemple pour fokusia :
  « Projet 1 — Cabinet d'investigation Dupont & Associés · Refonte
  d'identité Corporate · 2025 ». L'utilisateur les remplacera plus
  tard.

**Mémoriser les réponses** :
- `PORTFOLIO_SOURCE` : "files-ready" / "photos-only" / "placeholders" / "later"
- `PORTFOLIO_LAYOUT` : "grid-3-cols" / "stack-vertical" / "carousel" / "tabs"
- `PORTFOLIO_DEPTH` : niveau choisi en P3 (1, 2, 3 ou 4)

Ces 3 variables **pilotent la section Portfolio** du landing-page.md
(position dans la structure, layout, profondeur de copy par projet) et
**doivent apparaître dans le header documentaire** (cf. A.5.d, bloc
« Décisions portfolio »).

**Position de la section Portfolio dans la landing** :
- Si Hero porte déjà un visuel fort (`HERO_LAYOUT = full-bleed`) →
  Portfolio en **section 2-3** (juste après le hero, avant le constat —
  le visiteur arrive et voit immédiatement le travail).
- Si Hero est sobre (`HERO_LAYOUT = text-only` ou `text-illustration`) →
  Portfolio peut être déplacé plus bas (sections 4-5, après méthode +
  pour qui — on construit la confiance d'abord, on prouve ensuite).

### A.5.0.bis — Hero & visuels (3 questions sur le hero + style d'imagerie)

> **Deuxième sous-étape de A.5**, juste après A.5.0. Calibre le **layout
> du hero** (la partie qui porte 80% du résultat) + le **style d'imagerie
> global** de la landing. Sans cette intro, le hero tombe sur le default
> « image full-bleed + texte en surimpression » et toutes les images
> suivent aveuglément `brand.md` Q13 — ce qui ne correspond pas toujours
> aux contraintes du projet landing (image déjà prête, mood différent du
> brand book, etc.).

**Annonce avant la vague** : « 3 dernières questions sur le hero et le
style des images. ~30 secondes. »

**Pré-scan** : avant de poser H2, vérifier si `./images/hero.jpg` (ou
`hero.png`, `hero.webp`) est déjà présent dans le dossier projet :

```bash
ls ./images/hero.* 2>/dev/null
```

Si une image hero existe → mémoriser `HAS_HERO_IMAGE = true` et son
chemin → l'option « J'ai déjà une image » devient le défaut recommandé
en H2.

Lancer une vague unique `AskUserQuestion` (3 questions) :

| # | Question | Options |
|---|---|---|
| H1 | **Layout du hero** ? | Image plein écran + texte en surimpression *(visuel fort, défaut)* · Split 50/50 (texte à gauche, image à droite) · Texte centré + image en dessous *(mobile-first, lecture verticale)* · Texte seul sur fond couleur unie (sobre maximal, pas d'image) · Texte + illustration SVG/vectorielle décorative (pas de photo) |
| H2 | **Image du hero** ? | J'ai déjà une image — `./images/hero.jpg` *(option visible UNIQUEMENT si HAS_HERO_IMAGE = true, devient défaut)* · À générer via brief créatif (Midjourney / Imagen / Gemini / banque image) — j'ai besoin d'un brief calibré · À déposer plus tard — placeholder visible « [Hero image à fournir] » |
| H3 | **Style d'imagerie pour TOUTE la landing** ? | Suivre `brand.md` Q13 *(recommandé pour cohérence brand book ↔ landing)* · Photos documentaire (réel, lumière naturelle) · Illustrations vectorielles (style éditorial) · 3D / isométrique · Photos duotone / monochrome (traitement éditorial unifié) · Mix photos + illustrations (différent selon section) |

**Important** :
- Une seule passe de 3 questions, pas de wave suivante.
- Si l'utilisateur choisit H3 = « Suivre brand.md Q13 », charger Q13
  depuis `brand.md` et l'utiliser comme image style. Sinon, le choix H3
  **override** Q13 pour la landing (mais pas pour le brand book Figma —
  brand.md reste source de vérité).

**Mémoriser les réponses** :
- `HERO_LAYOUT` : "full-bleed" / "split-50-50" / "centered-image-below" / "text-only" / "text-illustration"
- `HERO_IMAGE_SOURCE` : "provided" / "to-generate" / "placeholder"
- `HERO_IMAGE_PATH` : chemin si "provided", null sinon
- `LANDING_IMAGERY_STYLE` : valeur H3 (ou "brand-md-q13" pour fallback)

Ces 4 variables **pilotent** :
- La **section 1 Hero** du landing-page.md (layout + brief image)
- Tous les **briefs visuels** des autres sections (qui reprennent
  `LANDING_IMAGERY_STYLE`)

Et **doivent apparaître dans le header documentaire** de `landing-page.md`
(cf. A.5.d, bloc « Décisions hero & visuels »).

**Si HERO_IMAGE_SOURCE = "to-generate"** → l'Étape A.5.c rédige un brief
créatif 5 champs (sujet incarné, composition, lumière, palette,
références culturelles) directement utilisable comme prompt
Midjourney / Imagen. Croiser **HERO_LAYOUT** (ex : full-bleed = cadrage
vertical contre-plongée vs split = cadrage carré ou portrait) +
**LANDING_IMAGERY_STYLE** (calibre la palette et le traitement) +
**brand.md Univers Q12** (référence d'univers) pour produire un brief
calibré, pas générique.

### A.5.0.quater — Production-ready · destinations & liens (vague BLOQUANTE avant génération HTML)

> **Quatrième sous-étape obligatoire de A.5**, à exécuter juste après A.5.0.bis (et après A.5.0.ter si Portfolio coché). **C'est une étape BLOQUANTE** : aucune génération HTML downstream n'est autorisée tant que toutes les variables de cette section n'ont pas été collectées **avec un fallback documenté** pour chaque cas où l'utilisateur n'a pas l'info. **Aucun `href="#"` orphelin n'est jamais acceptable**, même temporairement.
>
> **Pré-consommation** : si `infos-entreprise.txt` est présent et que les
> champs `URL Calendly`, `Email contact (footer)`, `URL LinkedIn perso`,
> et tous les blocs `Éditeur / Hébergeur / RGPD` sont remplis, cette vague
> est **entièrement skippée**. Le skill confirme uniquement en une ligne :
> *« A.5.0.quater préchargée depuis infos-entreprise.txt — passage direct à
> la génération. »* Si certains champs sont vides, seuls ces champs-là sont
> demandés en mini-question complémentaire.

**Pourquoi cette étape existe (et pourquoi elle bloque)** : la première version de la landing fokusia (mai 2026) a livré des `href="#"` partout (CTAs, LinkedIn, mentions légales, projets) parce que le skill générait sans avoir collecté ces inputs. Résultat : trois passes correctives manuelles ont été nécessaires après génération. Cette vague rend la landing **directement exploitable dès la première génération HTML**, sans aller-retour.

**Règle de production single-page** : par défaut, la landing est une **single-page HTML self-contained**. Les mentions légales et la politique de confidentialité sont intégrées en **accordions au pied de landing** (ancres `#mentions-legales` et `#confidentialite`), **pas dans des fichiers `.html` séparés**. Le seul cas où on génère des pages dédiées : si l'utilisateur le demande **explicitement** ou si le site est multi-page hosté.

**Annonce avant la vague** : « 4 dernières questions pour rendre la landing production-ready : CTA principal, LinkedIn, mentions légales, comportement des projets portfolio. ~1 minute. Sans ces infos, je ne peux pas générer le HTML. »

Lancer une vague unique `AskUserQuestion` (4 questions structurées) :

| # | Question | Options |
|---|---|---|
| D1 | **CTA principal** : que se passe-t-il au clic sur « Réservez votre [...] » ? | Lien Calendly direct *(défaut B2B premium)* · Mailto avec sujet pré-rempli · Lien vers formulaire externe (Tally, Typeform, Notion form) · Pas encore configuré → mailto temporaire avec l'email fourni en D-suite |
| D2 | **URL LinkedIn** pro ? | `linkedin.com/in/[handle-perso]` · `linkedin.com/company/[brand]` · Les deux liens (perso + entreprise) · Pas encore de LinkedIn → masquer entièrement du footer (jamais d'href vide) |
| D3 | **Mentions légales et politique de confidentialité** ? | **Accordions intégrés en pied de landing** *(défaut single-page recommandé)* · Page externe hébergée ailleurs (Notion, LegalPlace, Captain Contrat) · Pages dédiées `.html` *(seulement si demande explicite multi-page)* · Pas encore rédigées → accordions avec contenu placeholder rouge dashed visible |
| D4 | **Liens projets portfolio** au clic ? | Pas de lien : lecture visuelle uniquement *(défaut si pas de page projet, retire le `<a>` wrapper)* · Lien vers Behance / Dribbble · Pages dédiées (multi-page uniquement) · Vers section CTA contact |

**Vague complémentaire — précisions URL** (à poser systématiquement selon les réponses D1-D4) :

| Si réponse | Question complémentaire |
|---|---|
| D1 = Calendly | « URL Calendly exacte ? » → format `https://calendly.com/[handle]/[event]` |
| D1 = Formulaire externe | « URL exacte du formulaire ? » |
| D1 = Mailto OU Mailto temporaire | « Email de contact ? » → confirme (`contact@brand.fr`, `prenom@brand.fr`) |
| D2 = perso ou les deux | « Handle LinkedIn ? » → `prenom-nom` |
| D2 = entreprise ou les deux | « Slug LinkedIn entreprise ? » |
| D3 = Page externe | « URL Notion / LegalPlace ? » |
| D4 = Behance/Dribbble | « URLs des 3 fiches projet ? » |
| Toujours | « Email de contact pour le footer et mailto secondaires ? » (si pas déjà capté en D1) |

**Variables collectées (toutes obligatoires)** :
- `CTA_PRIMARY_URL` : URL absolue (`https://calendly.com/...` ou `mailto:contact@brand.fr?subject=...`). **Jamais null.**
- `CONTACT_EMAIL` : email pour footer + mailto secondaires (format `contact@brand.fr`). **Jamais null.**
- `LINKEDIN_URL_PERSO` : URL LinkedIn perso (ou `null` → mécanisme de masquage activé)
- `LINKEDIN_URL_BRAND` : URL LinkedIn entreprise (ou `null`)
- `LEGAL_MODE` : `"accordion"` *(défaut single-page)* / `"external"` / `"dedicated-pages"` / `"placeholder"`
- `LEGAL_EXTERNAL_URL` : URL externe Notion/LegalPlace (si `LEGAL_MODE = "external"`)
- `PORTFOLIO_LINKS_MODE` : `"none"` *(défaut)* / `"external-portfolio"` / `"dedicated-pages"` / `"to-contact"`
- `PORTFOLIO_LINKS_URLS` : tableau de 3 URLs (si `PORTFOLIO_LINKS_MODE = "external-portfolio"`)

**Règle de blocage explicite** : si après cette vague une variable de gauche est `null` ou non collectée, le skill **doit relancer une mini-question complémentaire** pour la combler par un fallback documenté (mailto temporaire, masquage, accordion placeholder, retrait du wrapper `<a>`). Jamais de génération HTML avec une variable manquante.

Ces variables **pilotent toutes les destinations de `<a>` dans le HTML downstream** et **doivent apparaître dans le header documentaire de `landing-page.md`** (cf. A.5.d, bloc « ✦ Décisions production-ready »).

**Mapping A.5.0.quater → patches HTML** :

| Variable | Patch HTML |
|---|---|
| `CTA_PRIMARY_URL` = URL Calendly | Bouton hero, header nav « Diagnostic », bouton CTA final, lien footer « Réserver un diagnostic » → tous pointent vers cette URL avec `target="_blank" rel="noopener noreferrer"` |
| `CTA_PRIMARY_URL` = `mailto:...` | Idem mais sans `target="_blank"` (mailto reste dans onglet courant) |
| `CONTACT_EMAIL` | Footer : `<a href="mailto:[email]">[email]</a>`. CTA secondaire final : `mailto:[email]?subject=Diagnostic%20d%27image%20[Brand]` (sujet URL-encodé) |
| `LINKEDIN_URL_PERSO` non null | Footer : `<a href="[URL]" target="_blank" rel="noopener noreferrer">LinkedIn</a>` |
| `LINKEDIN_URL_PERSO` = null | Le lien LinkedIn est **entièrement retiré** du footer (pas de tooltip "À venir", pas de href vide) |
| `LEGAL_MODE = "accordion"` *(défaut)* | **Section `<section id="legal">` ajoutée en pied de landing, juste avant le footer**, contenant deux `<details>` avec id `mentions-legales` et `confidentialite`. Liens footer pointent vers ces ancres internes. Pattern HTML/CSS détaillé ci-dessous. |
| `LEGAL_MODE = "external"` | Liens footer pointent vers `LEGAL_EXTERNAL_URL` avec `target="_blank" rel="noopener noreferrer"` |
| `LEGAL_MODE = "dedicated-pages"` | Cas exceptionnel multi-page : génération en parallèle des fichiers `./mentions-legales.html` et `./confidentialite.html` avec layout brand-cohérent. **À éviter en single-page**. |
| `LEGAL_MODE = "placeholder"` | Accordions intégrés mais contenu en `<span class="to-fill">[à compléter]</span>` rouge dashed visible. Forces l'utilisateur à compléter avant publication. |
| `PORTFOLIO_LINKS_MODE = "none"` | **Retirer les `<a>` wrappers** autour des `.mockup-card`, remplacer par `<div class="portfolio-card">`. **Jamais `href="#"`**. |
| `PORTFOLIO_LINKS_MODE = "external-portfolio"` | `<a href="[URL projet N]" target="_blank" rel="noopener noreferrer">` autour de chaque card |

**Pattern HTML/CSS pour mentions/confidentialité en accordion single-page (LEGAL_MODE = "accordion")** :

```html
<section id="legal" style="background: var(--cream-light); padding: 64px 0 32px;">
  <div class="max-w-4xl mx-auto px-8 md:px-12">
    <details id="mentions-legales" class="legal-item">
      <summary>
        <span class="legal-q">Mentions légales</span>
        <span class="faq-icon">+</span>
      </summary>
      <div class="legal-content">
        <p><strong>Éditeur du site</strong> — [Nom légal] · SIRET <span class="to-fill">[numéro]</span> · ...</p>
        <p><strong>Hébergement</strong> — ...</p>
        <p><strong>Propriété intellectuelle</strong> — ...</p>
        <p><strong>Cookies</strong> — ...</p>
      </div>
    </details>

    <details id="confidentialite" class="legal-item">
      <summary>
        <span class="legal-q">Politique de confidentialité <span class="legal-meta">· RGPD</span></span>
        <span class="faq-icon">+</span>
      </summary>
      <div class="legal-content">
        <p class="legal-callout">[Phrase manifeste sur la non-revente des données, le secret pro, etc.]</p>
        <p><strong>Responsable du traitement</strong> — ...</p>
        <p><strong>Données collectées / Finalités / Base légale / Durée / Destinataires</strong></p>
        <p><strong>Vos droits RGPD</strong> — ...</p>
      </div>
    </details>
  </div>
</section>
```

Le CSS `.legal-item` / `.legal-content` réutilise le même pattern que FAQ (border 1px cream, summary cliquable, max-height transition 0.6s, icône `+` qui pivote en accent au open).

**Champs à compléter par l'utilisateur** (toujours rouges dashed visibles via `<span class="to-fill">`) :
- Mentions : nom légal, SIRET, code APE, TVA, adresse, hébergeur
- Confidentialité : fournisseur email, hébergeur (sous-traitants)

L'utilisateur peut publier sans compléter ces champs immédiatement — les placeholders rouges signalent visuellement ce qui reste à faire, sans casser le rendu de la landing.

### A.5.a — Charger le contexte depuis `brand.md` + réponses A.5.0 / A.5.0.bis / A.5.0.quater

**Lire en priorité les variables A.5.0** (`LANDING_LENGTH`,
`LANDING_SECTIONS_INCLUDE`, `LANDING_CTA_PRIMARY`,
`LANDING_TRUST_ELEMENTS`, `LANDING_LEAD_MAGNET`) **et A.5.0.bis**
(`HERO_LAYOUT`, `HERO_IMAGE_SOURCE`, `HERO_IMAGE_PATH`,
`LANDING_IMAGERY_STYLE`) — elles **pilotent la structure et le visuel**,
pas le ton.

**Mapping A.5.0 → décisions structure** :

| Variable | Impact dans `landing-page.md` |
|---|---|
| `LANDING_LENGTH = "courte"` | 3-4 sections après le hero. Pas de FAQ, pas de section À propos. Hero + 1 problème + 1 méthode + 1 cas client + CTA final. |
| `LANDING_LENGTH = "moyenne"` *(défaut)* | 5-7 sections. Hero + constat + méthode + qualification + cas client + FAQ + CTA final. |
| `LANDING_LENGTH = "longue"` | 8-10 sections. Ajoute différenciateur dédié, section À propos, comparaison vs alternatives, garantie. |
| `LANDING_SECTIONS_INCLUDE` inclut `Témoignages clients` | Section dédiée témoignages (3-5 verbatim) en placeholder. Sinon, intégrer 1-2 verbatim courts dans le cas client. |
| `LANDING_SECTIONS_INCLUDE` inclut `Logos clients` | Section bandeau logos clients en monochrome teal. Sinon, omettre. |
| `LANDING_SECTIONS_INCLUDE` inclut `Calendly intégré` | Le CTA final ne mène pas à un formulaire mais à un widget Calendly embed — préciser dans le bloc Prompt. |
| `LANDING_SECTIONS_INCLUDE` inclut `Comparaison vs alternatives` | Section tableau comparatif (votre offre vs DIY vs concurrent type) avant la FAQ. |
| `LANDING_CTA_PRIMARY` | Texte exact du CTA hero + CTA final + CTA inline (au lieu du défaut « Réservez un diagnostic »). |
| `LANDING_TRUST_ELEMENTS` | Liste des éléments de réassurance à intégrer (en placeholder si pas encore disponibles). |
| `LANDING_LEAD_MAGNET` (si rempli) | Section dédiée au lead magnet AVANT le CTA final, avec promesse claire du livrable et formulaire d'opt-in en placeholder. |

**Mapping A.5.0.ter → décisions portfolio** *(actif si L2 inclut Portfolio)* :

| Variable | Impact dans `landing-page.md` |
|---|---|
| `PORTFOLIO_SOURCE = "files-ready"` | Section Portfolio référence directement `./images/cas-clients/cas1.png`, etc. Si `details.md` existe, en extraire titre/secteur/année. |
| `PORTFOLIO_SOURCE = "photos-only"` | Référence les images `./images/cas-clients/casN.png` + génère copy (titre + descriptif) cohérente avec brand.md. |
| `PORTFOLIO_SOURCE = "placeholders"` | Invente 3 projets-types cohérents avec brand.md (secteur du persona Q15, ton Q1, style imagerie Q13). Marquer chaque projet `[INVENTÉ — à remplacer]`. Brief créatif 5 champs pour chaque mockup. |
| `PORTFOLIO_SOURCE = "later"` | 3 cards vides avec `[Projet 1 — à fournir : photo + titre + descriptif]` visible. Pas de brief créatif. |
| `PORTFOLIO_LAYOUT = "grid-3-cols"` | Section grille 3 colonnes, ratio 4:3 par card, mobile : empile en 1 col. |
| `PORTFOLIO_LAYOUT = "stack-vertical"` *(défaut éditorial)* | Section avec 1 projet par scroll, full-width, ratio 16:9, alterne image-texte/texte-image. |
| `PORTFOLIO_LAYOUT = "carousel"` | Section carousel horizontal, snap par card, navigation flèches + dots. |
| `PORTFOLIO_LAYOUT = "tabs"` | Section avec tabs par catégorie en haut (à compléter par l'utilisateur), 3-9 projets en grille sous chaque tab. |
| `PORTFOLIO_DEPTH = 1` (sobre) | Par projet : mockup + titre + 1 ligne descriptive. |
| `PORTFOLIO_DEPTH = 2` | Par projet : + secteur client + année. |
| `PORTFOLIO_DEPTH = 3` | Par projet : + brief original + ce qui a été livré (2-3 lignes chacun). |
| `PORTFOLIO_DEPTH = 4` | Par projet : + résultat chiffré (en placeholder si pas de chiffre validé). |

**Position de la section Portfolio** : voir A.5.0.ter — calculée selon
`HERO_LAYOUT`. Si hero full-bleed → Portfolio en section 2 (visible
immédiatement). Si hero sobre → Portfolio en section 4-5 (après méthode).

**Mapping A.5.0.bis → décisions hero & visuels** :

| Variable | Impact dans `landing-page.md` |
|---|---|
| `HERO_LAYOUT = "full-bleed"` *(défaut visuel fort)* | Section 1 Hero : background image plein écran (100vw × 100vh), texte en surimpression sur dégradé subtil cream→transparent (ou dark→transparent selon brand.md Q11). Brief image : cadrage vertical contre-plongée, 1/3 inférieur réservé au texte. |
| `HERO_LAYOUT = "split-50-50"` | Section 1 Hero : grille 2 colonnes (texte gauche 50%, image droite 50%). Brief image : cadrage carré ou portrait. Mobile : empilé texte-image. |
| `HERO_LAYOUT = "centered-image-below"` | Section 1 Hero : texte centré full-width, image en dessous ratio 16:9. Mobile-first natif. Brief image : cadrage horizontal large. |
| `HERO_LAYOUT = "text-only"` | Section 1 Hero : texte seul sur fond couleur unie (`primaryDark` ou `cream` selon brand.md Q11). Pas de brief image, eyebrow accent renforcé. |
| `HERO_LAYOUT = "text-illustration"` | Section 1 Hero : texte + illustration SVG vectorielle décorative à droite ou en arrière-plan opacity 30%. Brief illustration calibré au style brand.md Q14bis. |
| `HERO_IMAGE_SOURCE = "provided"` | Le brief hero de section 1 se contente de référencer `HERO_IMAGE_PATH` : *« Image fournie : `./images/hero.jpg` — utiliser telle quelle. Crop recommandé : [selon HERO_LAYOUT]. »* |
| `HERO_IMAGE_SOURCE = "to-generate"` | Le brief hero contient un brief créatif 5 champs complet (sujet, composition, lumière, palette, références) **prêt à copier-coller dans Midjourney / Imagen / Gemini**. |
| `HERO_IMAGE_SOURCE = "placeholder"` | Le brief hero indique `[Hero image à fournir — chemin attendu : ./images/hero.jpg]` + brief créatif de référence pour quand l'utilisateur la générera plus tard. |
| `LANDING_IMAGERY_STYLE = "brand-md-q13"` | Tous les briefs visuels (hero + autres sections) suivent `brand.md` Q13 (cohérence brand book ↔ landing). |
| `LANDING_IMAGERY_STYLE = "Photos documentaire"` | Override Q13 : lumière naturelle, sujets réels en situation, palette calibrée brand.md. |
| `LANDING_IMAGERY_STYLE = "Illustrations vectorielles"` | Override Q13 : illustrations éditoriales, palette restreinte aux tokens brand, pas de photo. |
| `LANDING_IMAGERY_STYLE = "3D / isométrique"` | Override Q13 : rendus 3D isométriques, axonometric, palette tokens brand. |
| `LANDING_IMAGERY_STYLE = "Photos duotone / monochrome"` | Override Q13 : traitement duotone (deux tokens brand), monochrome teal ou cream. |
| `LANDING_IMAGERY_STYLE = "Mix photos + illustrations"` | Permission de mixer selon section (hero=photo, process=illustration, etc.) — préciser dans chaque brief visuel. |

**Garde-fou** : si une section A.5.0 demande des données externes
(témoignages, logos, chiffres), elles apparaissent **toujours en
placeholder visible** `[À COMPLÉTER]` — ne **jamais** halluciner.

**Puis** relire dans `brand.md` les sections qui calibrent le **ton**
(pas la structure) :

Avant de générer, relire dans `brand.md` ces sections clés :

| Section brand.md | Usage dans la landing |
|---|---|
| Positionnement (Qui / Pour qui / Promesse / Différenciateur) | Hero H1+sous-titre · section Problème · section Solution |
| Identité verbale (ton dominant Q1) | Registre général de toute la copy |
| Style d'accroches (Q5) | Forme des H1 / H2 / eyebrows |
| Style de CTAs (Q6) | Texte exact des boutons CTA |
| Mots à utiliser | Vocabulaire imposé |
| Mots à éviter | Vocabulaire interdit (reformuler si nécessaire) |
| Formule signature | Hero (en variation) + CTA final (intégrale) — apparaît **2 fois** |
| Ambiance lumineuse (Q11) | Fond du hero et des sections « accent » |
| Univers visuel (Q12-Q14) | Briefs visuels (illustrations, photos, mood) |
| Icônes & illustrations (Q14bis) | Style imposé pour toutes les icônes et illustrations de la landing |
| Persona (Q15) | Tonalité de la copy, choix du vocabulaire, structure FAQ |
| Objection #1 (Q16) | 1ère question FAQ obligatoire — verbatim prospect |
| Différenciateur (Q17) | Section « Pourquoi nous » / « Méthode unique » et sous-titre hero |

### A.5.b — Choisir le template

**Lire d'abord le fichier `references/landing-page-templates.md`** —
notamment sa section *Logique de sélection*. Cette logique appliquée aux
réponses Q1/Q3/Q4 + au positionnement de `brand.md` détermine lequel des
3 templates utiliser :

| Template | Cas typique |
|---|---|
| **TEMPLATE A** — Offre freelance / SaaS B2B *(défaut)* | Q3 = Partenaire stratégique / Expert technique, Q4 = clarté/performance/confiance, vente d'une prestation ou d'un accompagnement |
| **TEMPLATE B** — Landing produit | Q3 = Exécutant fiable, Q4 = Gagner du temps, contexte logiciel/SaaS/app autonome |
| **TEMPLATE C** — Personal brand / Portfolio | Q3 = Coach inspirant OU (Q1 = Chaleureux & humain sans produit logiciel), freelance / créateur / consultant indépendant |

**En cas d'ambiguïté** (50/50 entre deux templates), poser **une seule question**
à l'utilisateur :

> Pour ta landing j'hésite entre :
> 1. Template **Offre freelance / SaaS B2B** (sections : problème → solution → bénéfices → process → qualification — pas de pricing affiché)
> 2. Template **Personal brand / Portfolio** (sections : about → services → selected work → process)
>
> Tu vends plutôt **une offre/méthode** ou **toi-même comme expert** ?

Une fois le template choisi, **ne lire que ce template** dans
`references/landing-page-templates.md` — pas les 3.

### A.5.c — Rédiger la copy (pas juste des placeholders !)

C'est l'étape **critique** qui distingue ce skill d'un wireframe vide. Pour
chaque champ de copy (H1, H2, paragraphes, CTAs, bullets, FAQ, etc.) :

1. **Hero — produire 3 variantes systématiquement** (le hero porte ~80% du
   résultat d'une landing — c'est trop risqué de n'en générer qu'une seule).
   Chaque variante = un H1 + un sous-titre. Les 3 variantes suivent
   3 angles différents :
   - **Variante A — Affirmation tranchée** (ex : *« Clarté. Méthode. Résultats. »*)
   - **Variante B — Question ouverte** (ex : *« Et si votre charte vous rapportait des clients ? »*)
   - **Variante C — Chiffre / preuve** (ex : *« 200+ marques calibrées en 14 questions. »*)

   Toutes les 3 doivent respecter Q5 (style d'accroches privilégié) mais
   chacune l'angle Q5 différemment. **Recommander la variante principale**
   en 1 ligne d'argumentaire (alignement Q1+Q5+persona). Conserver les
   2 autres dans une section **« Variantes hero (A/B test) »** en fin de
   fichier — elles servent au test post-launch.
2. **Reste de la copy** (H2 sections, paragraphes, CTAs, bullets, FAQ,
   etc.) : produire 3-5 variations en interne puis choisir la plus
   impactante. Garder en tête le persona (« Pour qui » dans brand.md).
3. **Utiliser des mots de la liste « Mots à utiliser »** chaque fois que c'est
   naturel. Bannir absolument les mots de « Mots à éviter ».
4. **Placer la formule signature** au moins **2 fois** : version condensée dans
   le hero (H1 ou sous-titre), version intégrale dans le CTA final (H2).
5. **Bullets et listes** : rythme court, parallélisme syntaxique, vocabulaire
   du persona. Pas d'adjectifs creux (« puissant », « innovant », « unique »)
   — préférer des verbes d'action et des bénéfices ancrés.
6. **CTAs** : exactement dans le style Q6. Même CTA en hero et en CTA final
   (cohérence). Variante plus douce pour le CTA secondaire.
7. **FAQ** : 4-6 questions ancrées sur les **vraies objections** du
   persona. **Inclure systématiquement l'Objection #1 (Q16)** en première
   FAQ, formulée verbatim (mots du prospect, pas mots de la marque) puis
   réponse calibrée dans le ton Q1. Compléter avec 3-5 objections
   universelles (temps, complexité, garantie, « pour mon cas ? »)
   adaptées au persona Q15.
8. **Témoignages / logos / chiffres clés** : **toujours en placeholder**
   (« [Témoignage 1 — à compléter] »). Ne jamais halluciner de noms réels.
9. **Briefs visuels** : pour chaque section qui contient un visuel, rédiger
   un **brief illustratif et créatif** suivant la structure 5-champs définie
   dans `references/landing-page-templates.md` (Règle #1) — sujet incarné,
   composition, lumière, palette, références culturelles. Ne **jamais**
   produire de brief générique style « capture produit » ou « photo
   lifestyle ». Croiser explicitement Q11 (ambiance), Q12 (référence
   d'univers), Q14 (personnalité graphique) et Q14bis (style d'icône/illu)
   pour calibrer chaque brief.
10. **Aucune mention de prix** dans le fichier — règle #0 du templates.md.
    Si une section semble appeler une mention tarifaire (services, plans,
    offres), la remplacer par un CTA de qualification (RDV / contact / devis
    à la demande).

### A.5.c.bis — Mapper les images de `/images/` aux sections

**Avant** de rédiger les briefs visuels, croiser `AVAILABLE_IMAGES`
(scanné en A.1.b) avec les sections du template retenu. Le mapping se fait
**par nom de fichier** — les conventions de naming guident automatiquement
le placement :

| Pattern de nom | Section cible |
|---|---|
| `hero.*` | Section 1 · Hero (visuel principal) |
| `photo-profil.*` (ou legacy `about*`, `portrait*`, `bio*`) | Section About / Personal brand |
| `process-1.*` → `process-N.*` | Sections process (1 image par étape) |
| `feature-1.*` → `feature-N.*` | Sections features / bénéfices |
| `cas-clients/cas1.*` → `casN.*` (ou legacy `case-1.*`, `work-1.*`) | Section Selected work / case studies |
| `testimonial-1.*` → `…-N.*` | Photos clients dans section témoignages |
| `mood-1.*` → `mood-4.*` | Slide 04 Universe & Mood (brand book Figma) |
| Autre nom | Demander à l'utilisateur quelle section, ou ranger en bas de fichier dans une section « Images disponibles non placées » |

**Règle de rédaction des briefs visuels** :

- Si une image **matche** la section : remplacer le brief créatif par
  *« Image fournie : `./images/[filename]` — utiliser telle quelle. Crop
  recommandé : [16/9 / 4/3 / 1/1 selon section]. »*
- Si **aucune image** ne matche : conserver le brief créatif 5-champs
  (sujet, composition, lumière, palette, références) **et** ajouter une
  ligne *« À déposer dans `./images/[suggested-filename].jpg`. »*
- Mix possible : une section peut avoir une image fournie ET un brief
  pour une 2ème image (ex : hero avec photo + illustration décorative).

### A.5.d — Format du fichier `landing-page.md`

Reprendre **fidèlement** la structure du template retenu dans
`references/landing-page-templates.md`, en remplaçant les `[crochets]` par la
copy rédigée. Ne pas supprimer les commentaires `<!-- … -->` en tête de
fichier — ils documentent quel template a été choisi et pourquoi.

> **Le fichier `landing-page.md` est conçu pour être copié-collé tel quel
> comme prompt** dans Claude / ChatGPT / Cursor / Framer AI / Webflow AI /
> v0 / Lovable / etc. pour générer la landing page finale en HTML, React,
> Framer ou Webflow. C'est pour ça qu'il **commence obligatoirement par un
> bloc « Prompt »** qui pose le rôle UX/UI designer spécialisé landing page
> et cadre la mission. Sans ce bloc, le brief n'est pas auto-portant et
> l'outil cible régresse vers du générique.

**Structure minimale attendue** :

```markdown
# Landing — [Nom de la marque]

<!-- TEMPLATE [A|B|C] · [Nom du template] -->
<!-- Sélectionné car : [raison en 1 ligne basée sur brand.md] -->

## ✦ Décisions structure (réponses A.5.0)

> Auto-documentation du fichier — surface les 4 choix de structure
> commerciale validés par l'utilisateur avant l'écriture. Si tu rouvres
> ce fichier 6 mois plus tard, tu sais pourquoi telle section est là ou
> pas. Ne **jamais** supprimer ce bloc en post-édition manuelle — c'est
> la mémoire de la décision.

| Décision | Valeur |
|---|---|
| **Longueur** (L1) | [courte / moyenne / longue / sur-mesure] *(défaut si vide : moyenne)* |
| **Sections obligatoires** (L2) | [liste des sections cochées, séparées par ` · `] *(défaut si vide : FAQ uniquement)* |
| **CTA primaire** (L3) | [valeur exacte L3] *(défaut si vide : Diagnostic gratuit)* |
| **Éléments de réassurance** (L4) | [liste des éléments cochés, séparés par ` · `] *(défaut si vide : Aucun)* |
| **Lead magnet** (L5, si L3 = Téléchargement) | [titre + angle] *(sinon : N/A)* |

**Sections retenues dans cette landing** : [liste numérotée des sections
finales du fichier, dérivée du croisement L1 × L2 — ex :
« 1. Hero · 2. Constat · 3. Méthode · 4. Pour qui · 5. Cas client · 6.
Pourquoi nous · 7. FAQ · 8. CTA final »]

**Sections écartées et raison** : [liste courte des sections du template
A/B/C qui ont été retirées et pourquoi — ex : « Tarifs : règle #0
non-mention prix. À propos : non coché en L2 + longueur moyenne ne le
requiert pas. »]

---

## ✦ Décisions hero & visuels (réponses A.5.0.bis)

> Surface les 3 choix sur le hero + le style d'imagerie global. Ces
> décisions overridd certains défauts de `brand.md` (notamment Q13) —
> traçabilité pour comprendre pourquoi telle image, telle composition.

| Décision | Valeur |
|---|---|
| **Layout du hero** (H1) | [full-bleed / split-50-50 / centered-image-below / text-only / text-illustration] *(défaut si vide : full-bleed)* |
| **Source image hero** (H2) | [provided / to-generate / placeholder] *(défaut si vide : placeholder si pas de `./images/hero.*`, sinon provided)* |
| **Chemin image hero** (si provided) | [chemin relatif détecté, ex : `./images/hero.jpg`] *(sinon : N/A)* |
| **Style d'imagerie pour toute la landing** (H3) | [Suivre brand.md Q13 / Photos documentaire / Illustrations vectorielles / 3D isométrique / Photos duotone / Mix] *(défaut si vide : Suivre brand.md Q13)* |

**Conséquences appliquées** : [3-4 lignes courtes — ex :
« Hero full-bleed → section 1 utilise background image 100vw × 100vh +
texte en surimpression. Image fournie → brief minimal référence
`./images/hero.jpg`. Style override en duotone → tous les briefs visuels
des sections suivantes spécifient le traitement duotone teal/cream. »]

---

## ✦ Décisions portfolio (réponses A.5.0.ter, si Portfolio coché en L2)

> Surface les 3 choix sur la section selected work. Ce bloc n'apparaît
> que si l'option Portfolio est cochée à l'étape A.5.0 L2. Pour les
> graphistes / agences créa, c'est la section qui fait la conversion —
> traçabilité critique.

| Décision | Valeur |
|---|---|
| **Source des projets** (P1) | [files-ready / photos-only / placeholders / later] *(défaut si vide : placeholders)* |
| **Layout de présentation** (P2) | [grid-3-cols / stack-vertical / carousel / tabs] *(défaut si vide : stack-vertical éditorial)* |
| **Profondeur d'info par projet** (P3) | [1 mockup+titre+1ligne / 2 +secteur+année / 3 +brief+livraison / 4 +résultat chiffré] *(défaut si vide : 1)* |
| **Position section dans la landing** | [section N — calculée selon HERO_LAYOUT, cf. A.5.0.ter] |
| **Dossier `./images/cas-clients/` détecté** | [oui (avec liste des cas trouvés) / non — invention 3 cas-types] |

**Si placeholders inventés** — liste des 3 projets-types générés :

1. **[Titre projet 1 — INVENTÉ]** · [Secteur] · [Année] — [1 ligne descriptive]
2. **[Titre projet 2 — INVENTÉ]** · [Secteur] · [Année] — [1 ligne descriptive]
3. **[Titre projet 3 — INVENTÉ]** · [Secteur] · [Année] — [1 ligne descriptive]

> Pour remplacer ces placeholders : créer `./images/cas-clients/cas1.png`
> (+ `details.md` optionnel) puis relancer le skill en mode LANDING.

---

## ✦ Prompt — à copier-coller tel quel

> **Rôle** : Tu es **UX/UI designer senior spécialisé dans les landing pages
> à fort taux de conversion**. Tu as designé +200 landings pour des SaaS B2B,
> des marques DTC et des freelances premium. Tu maîtrises la grammaire des
> hero qui accrochent en 3 secondes, les sections qui répondent aux vraies
> objections, les CTAs qui convertissent, et la hiérarchie visuelle qui
> guide l'œil sans effort. Tu connais Framer, Webflow, Tailwind, shadcn/ui,
> et tu sais quand un layout sent l'IA générique et comment l'éviter.
>
> **Mission** : Génère la landing page complète de **[Nom de la marque]** à
> partir du brief ci-dessous. La copy est **déjà rédigée et validée** — tu
> ne dois **pas la réécrire**, juste la mettre en forme. Ton job c'est le
> design : layout, hiérarchie typo, composition, micro-interactions,
> qualité visuelle premium, cohérence avec l'univers de marque décrit plus
> bas.
>
> **Contraintes non négociables** :
> - **Respecte la copy à la lettre** : pas de paraphrase, pas d'ajout, pas de suppression. Chaque H1, H2, bullet, CTA et FAQ a été calibré.
> - **Respecte les briefs visuels** de chaque section (sujet incarné, composition, lumière, palette, références culturelles). Pas de stock photo générique, pas d'abstract gradient par défaut, pas de blob coloré.
>
> - **LOGO · règle absolue (PRIORITÉ HAUTE)** :
>   1. Le logo doit toujours être inséré sous forme de **vrai SVG**, jamais comme un wordmark recréé en CSS / `<span>` / texte stylé. Recréer le wordmark en CSS (genre `<span class="wordmark">fokus<span class="dot">i</span>a</span>`) est **INTERDIT**, c'est le pire signal de génération IA paresseuse.
>   2. Source obligatoire : `./logo/logo.svg` (variante couleur) et `./logo/logo-white.svg` (variante blanche).
>   3. Méthode d'insertion recommandée pour single-file HTML :
>      - Soit `<img src="./logo/logo.svg" alt="[Nom marque]" />` et `<img src="./logo/logo-white.svg" alt="[Nom marque]" />` (les SVG vivent à côté du HTML, chargés naturellement par le navigateur).
>      - Soit SVG inliné directement dans le HTML (extraire le contenu de `<svg>...</svg>` du fichier et coller dans le markup).
>   4. Pour le toggle de variante (header au scroll : blanc → couleur), utiliser **deux `<img>` superposés** avec opacity transition CSS, jamais un re-color JS, jamais un filter CSS de couleur.
>   5. Ne jamais re-colorer, re-générer ou approximer un logo programmatiquement.
>   6. Si les fichiers `./logo/logo.svg` et `./logo/logo-white.svg` sont absents du dossier, **bloquer la génération HTML** et demander à l'utilisateur de les exporter depuis Figma vers le dossier `./logo/` du projet.
>
> - **IMAGES · règle gracieuse (PRIORITÉ HAUTE)** :
>   1. Chaque chemin référencé dans le brief (`./images/hero.jpg`, `./images/photo-profil.jpg`, `./images/cas-clients/casN.png`, etc.) doit apparaître comme un **vrai `<img src="..." />` dans le HTML**, pas comme un commentaire, pas comme une variable vide.
>   2. Pour le hero, mettre en place une **cascade de fallback en 3 niveaux** :
>      - **Niveau 1 (prioritaire)** : `src="./images/hero.jpg"` (local, contrôlé par l'utilisateur, écrase tout le reste dès qu'il existe).
>      - **Niveau 2 (présentable immédiatement)** : `data-fallback="https://images.unsplash.com/photo-XXX?w=1920&q=80&auto=format&fit=crop"` avec photo Unsplash spécifique (URL stable avec photo ID), libre de droits, cohérente avec le brief visuel du brand.md (univers Q12 + imagerie Q13). Chargée par `onerror` quand le local n'existe pas.
>      - **Niveau 3 (offline / firewall)** : composition CSS riche `.hero-fallback` (multi-gradients + verticalité + noise) quand Unsplash est inaccessible. Cf. modèle technique ci-dessous.
>      - Ajouter un **crédit photo discret** (opacity ≤ 0.32, en bas du hero, masqué par défaut) qui ne s'affiche que quand on tombe sur le niveau 2, format : *« Photo provisoire · Unsplash · à remplacer dans `./images/hero.jpg` »*. Le sélecteur CSS `.hero-img[data-tried="unsplash"] ~ .hero-img-credit { display: block; }` gère l'affichage.
>      - Pattern HTML/JS recommandé :
>        ```html
>        <img src="./images/hero.jpg"
>             data-fallback="https://images.unsplash.com/photo-1568667256549-094345857637?w=1920&q=80&auto=format&fit=crop"
>             onerror="if (!this.dataset.tried) { this.dataset.tried='unsplash'; this.src=this.dataset.fallback; } else { document.body.classList.add('no-hero-img'); }" />
>        ```
>   3. Gestion du fallback pour les **autres sections** :
>      - **Mockups portfolio** : conserver les mockups composés CSS riches (cartes éditoriales avec contenu typographique cohérent au projet) en fallback. Pas de placeholder dashed.
>      - **Portrait About** : fond couleur primary + monogramme initiales + label discret acceptable comme fallback. Pas dashed. **Pattern obligatoire** : les overlays placeholder (monogramme + label "Portrait à fournir") doivent **se masquer automatiquement** quand l'image est chargée, via `onload="this.closest('.portrait-frame').classList.add('has-photo')"` et CSS `.portrait-frame.has-photo .portrait-placeholder-mono { opacity: 0; visibility: hidden; }`. Sinon la photo se charge AVEC le monogramme par-dessus, le rendu est sale. La caption éditoriale (place + numéro) reste visible dans les deux cas, comme signature magazine premium.
>      - **Duotone subtil sur portrait** : pour fondre la photo dans la palette brand (cohérence avec Q13 "photos duotone/monochrome"), appliquer 3 couches : (1) `filter: grayscale(0.35) contrast(1.06) brightness(0.94) saturate(1.05)` sur l'`<img>`, (2) voile teal `linear-gradient(165deg, rgba(primary, 0.32), rgba(primaryDark, 0.55))` en `mix-blend-mode: multiply` qui pousse les ombres vers le primary, (3) lumière rasante + vignette `radial-gradient` en `mix-blend-mode: overlay` pour profondeur. Évite l'effet "photo de stock collée" sans recolorer agressivement.
>      - **Sections secondaires (constat, méthode)** : placeholder dashed avec chemin attendu acceptable si la section ne porte pas le visuel principal.
>   4. **Ne jamais hot-linker Unsplash sur les sections secondaires** (portrait, mockups, constat, méthode). Le hot-link Unsplash est réservé au hero, qui doit être présentable même sans image locale. Pour les autres sections, le fallback CSS suffit.
>   5. **Choisir la photo Unsplash en cohérence avec brand.md** : sélectionner un photo ID stable depuis Unsplash dont le sujet, la lumière et la palette correspondent au brief visuel hero. Pour un brand univers Q12=Architecture/Éditorial luxe + ambiance Q11=Crème : photo de bibliothèque feutrée, cabinet juridique, livres reliés, intérieur ancien lumière chaude. Pour Q12=Tech minimale : intérieur épuré, surfaces brossées, lumière froide. Pour Q12=Nature : matières brutes, lumière naturelle. Documenter le photo ID retenu en commentaire HTML pour traçabilité.
>   6. L'élément `<img>` doit toujours être présent dans le DOM dès la génération, même si l'image n'existe pas encore. Sans `<img>`, l'utilisateur ne saura jamais où déposer son image.
>   7. **Modèle technique pour le hero fallback CSS (niveau 3)** (à adapter aux tokens du brand.md) :
>      ```css
>      .hero-fallback {
>        background:
>          radial-gradient(ellipse 60% 40% at 18% 22%, rgba([cream], 0.22) 0%, transparent 55%),
>          radial-gradient(ellipse 50% 35% at 82% 30%, rgba([accent], 0.10) 0%, transparent 60%),
>          radial-gradient(ellipse 80% 60% at 50% 100%, rgba([primaryDark], 0.85) 0%, transparent 55%),
>          linear-gradient(180deg, [primaryMid] 0%, [primary] 50%, [primaryDark] 100%);
>      }
>      .hero-fallback::before { /* verticalité : pilastres / étagères */
>        background-image: linear-gradient(90deg, transparent, transparent 11%, rgba([cream], 0.035) 11.2%, ... );
>      }
>      .hero-fallback::after { /* lumière rasante diagonale */
>        background: linear-gradient(118deg, rgba([cream], 0.08) 0%, transparent 35%, ...);
>        mix-blend-mode: overlay;
>      }
>      ```
>      Adapter les positions, les angles, et le nombre de pilastres au registre Q12 (architecture, nature, tech, éditorial).
>
> - **Respecte le ton et la personnalité graphique** définis plus bas (ambiance lumineuse, univers de référence, mots à utiliser/éviter).
> - **Aucune mention de prix**, la tarification est gérée hors landing.
> - Hero impactant **dès le scroll 0** (pas de loader, pas d'animation qui retarde le H1). Mobile-first. Accessibilité AA minimum.
> - Quand un placeholder `[à compléter]` apparaît (témoignages, logos, chiffres clés), garde-le visible. Ne l'invente pas, ne hallucine aucun nom client.
>
> - **PRODUCTION-READY · liens vivants (PRIORITÉ MAXIMALE — bloque la génération)** :
>   1. **Étape A.5.0.quater BLOQUANTE** : si une seule variable de destination est manquante (`CTA_PRIMARY_URL`, `CONTACT_EMAIL`, `LEGAL_MODE`, `PORTFOLIO_LINKS_MODE`), **REFUSER de générer le HTML** et relancer une mini-question pour combler. Aucun cas légitime de génération avec des destinations incomplètes.
>   2. **Zéro `href="#"` orphelin**. Toutes les URLs sont réelles. Seule exception tolérée : `href="#"` sur le logo retour-haut-de-page (header + footer), c'est convention web standard.
>   3. **Toutes les URLs externes** (Calendly, LinkedIn, formulaires externes, Behance, Dribbble) ont systématiquement `target="_blank" rel="noopener noreferrer"`. Les `mailto:` restent dans l'onglet courant (pas de `target="_blank"`).
>   4. **CTA principal** : pointe vers la valeur de `CTA_PRIMARY_URL` (Calendly de préférence, mailto à défaut). Apparaît au minimum 3 fois dans la landing avec la même URL : hero, header nav, CTA final. Plus une 4e dans le footer (« Réserver un diagnostic »).
>   5. **Mailto avec sujet pré-rempli** : `mailto:contact@brand.fr?subject=Diagnostic%20d%27image%20[Brand]` — URL-encoder l'espace en `%20` et l'apostrophe en `%27`. Évite à l'utilisateur de devoir taper l'objet.
>   6. **Email** : utiliser exclusivement la valeur `CONTACT_EMAIL` (jamais d'email inventé). Vérifier que l'email apparaît avec `<a href="mailto:[email]">` dans le footer (pas en texte simple non cliquable).
>   7. **Single-page par défaut** : mentions légales et politique de confidentialité **intégrées en accordions au pied de landing** (id `#mentions-legales` et `#confidentialite`), **pas** dans des fichiers `.html` séparés. Ne créer des pages dédiées que si `LEGAL_MODE = "dedicated-pages"` (cas exceptionnel multi-page).
>   8. **Variables manquantes → fallback documenté, jamais d'href vide** :
>      - `LINKEDIN_URL_PERSO = null` → **retirer entièrement** le lien LinkedIn du footer (pas de tooltip "À venir", pas de span désactivé)
>      - `CTA_PRIMARY_URL` Calendly pas encore configuré → fallback automatique sur `mailto:[CONTACT_EMAIL]?subject=...`
>      - `LEGAL_MODE = "placeholder"` → accordions générés mais contenu en `<span class="to-fill">[à compléter]</span>` rouge dashed
>      - `PORTFOLIO_LINKS_MODE = "none"` → wrapper `<a>` retiré, remplacé par `<div class="portfolio-card">`
>   9. **Footer · 4 destinations minimum** (jamais d'orphelin) :
>      - Email de contact (mailto avec href réel)
>      - URL LinkedIn (target blank) OU rien si masqué
>      - CTA secondaire « Réserver un diagnostic » (CTA_PRIMARY_URL)
>      - Mentions légales + Confidentialité (ancres `#mentions-legales` + `#confidentialite` par défaut)
>
> - **ANIMATIONS CSS · règle éditoriale subtle (PRIORITÉ MOYENNE)** :
>   1. **Easing commun** : `cubic-bezier(0.16, 1, 0.3, 1)` (out-expo modifié). C'est l'easing « premium » qui démarre vif et s'adoucit. Jamais `ease-in-out` standard (trop générique), jamais `cubic-bezier` bouncy / élastique (registre toy / playful).
>   2. **Durations** : 0.6 à 1.4 seconde pour les reveals, 0.3 à 0.5s pour les hover. Lent et élégant. Tout ce qui est sous 0.3s sent le clic mécanique.
>   3. **Stagger** : 0.1 à 0.15s entre éléments d'une même séquence (3 lignes de hero, 5 chapitres, 6 cards). Au-delà de 0.2s ça devient cinématographique, en-dessous de 0.1s le stagger n'est plus perceptible.
>   4. **Patterns recommandés** (à implémenter selon le brand) :
>      - **Hero load** : 3 lignes du H1 en slide-up + fade-in, stagger 0.15s, démarrage à 0.15s.
>      - **Scroll reveal** : `opacity: 0; transform: translateY(40px)` → `0`, via `IntersectionObserver` (threshold 0.12, rootMargin `0px 0px -80px 0px`).
>      - **Stagger interne** : sur un `.reveal.stagger`, les enfants se révèlent en cascade via `:nth-child(N) { animation-delay: N*0.1s }`.
>      - **Numéros projet/méthode** : `slide-in-left` (translateX -32px → 0) quand la section devient visible. Renforce la lecture éditoriale verticale.
>      - **Eyebrows** : `letter-spacing-reveal` (0.6em → 0.3em) au reveal, donne une sensation de respiration typographique. Sublime sur les uppercase tracking-wide.
>      - **Hero image** : zoom-in subtil (`scale(1.04) → 1`) sur 2.4s au load, OU parallax léger (`translate3d(0, scrollY * 0.18, 0)`) sur scroll. Pas les deux en même temps.
>      - **Float décoratif** : `float-soft` (translateY -6px loop 2.4s) sur le scroll indicator du hero. Aussi sur des marques décoratives (étoile, point médian), jamais sur du texte.
>      - **Hover boutons accent** : flèche `→` qui glisse de 6px sur 0.4s. Background couleur transition vers accent-dark.
>      - **Hover nav links** : underline 1px qui grandit `width: 0 → 100%` sur 0.4s.
>      - **FAQ accordion** : `max-height: 0 → 800px` sur 0.6s + icône `+` qui rotate 45° vers `×` sur 0.5s avec changement de fill background → accent.
>   5. **Anti-patterns à bannir** :
>      - Animations qui se déclenchent en boucle infinie sur du contenu (anneau qui tourne, dots qui pulsent). Réservées aux **loaders et états d'attente**, jamais sur la copy.
>      - Glitch / scramble text effects sur les H1 (registre cyberpunk, hors-sujet pour éditorial luxe).
>      - Curseurs custom élaborés (gros disque qui suit la souris, trail effect). Trop showy, casse la lecture.
>      - Confetti, particles, snow falling.
>      - Parallax brutal (factor > 0.3) qui crée du motion sickness.
>   6. **Respect prefers-reduced-motion** : obligatoire. Inclure le bloc CSS :
>      ```css
>      @media (prefers-reduced-motion: reduce) {
>        *, *::before, *::after {
>          animation-duration: 0.01ms !important;
>          animation-iteration-count: 1 !important;
>          transition-duration: 0.01ms !important;
>        }
>      }
>      ```
>   7. **Hiérarchiser les animations selon le brand.md** :
>      - **Q14 = Sobre / Premium** (cas fokusia) : animations rares, lentes, beaucoup de fondu, peu de mouvement. Maximum 1 animation visible par fold.
>      - **Q14 = Audacieux / Émotionnel** : animations plus présentes, parallax marqué, reveal stagger sur tous les éléments majeurs, possibilité de cinemagraph subtil sur le hero.
>      - **Q14 = Technique** : animations qui suggèrent le système (grilles qui apparaissent, lignes qui se tracent), pas d'effet émotionnel.
>      - **Q14 = Chaleureux** : transitions très douces (1.2s+), opacités, jamais de slide brutal.
>
> - **Anti-AI-slop checklist (à vérifier avant de livrer le HTML)** :
>   - [ ] Le logo de la marque est-il un vrai SVG chargé depuis `./logo/`, ou un wordmark CSS recréé ? Si CSS recréé → corriger immédiatement.
>   - [ ] Le hero référence-t-il `./images/hero.jpg` via un `<img>` réel ? Si juste un dégradé → ajouter l'`<img>` avec fallback.
>   - [ ] Le fallback hero (quand l'image n'est pas encore déposée) est-il une **composition CSS riche** (multi-gradients + verticalité + noise) ou un placeholder dashed visible ? Si placeholder dashed → remplacer par fallback riche.
>   - [ ] Les mockups portfolio référencent-ils `./images/cas-clients/casN.png` via `<img>` ? Sinon → ajouter les `<img>` avec onerror fallback.
>   - [ ] Le portrait About référence-t-il `./images/photo-profil.jpg` via `<img>` ? Sinon → ajouter.
>   - [ ] Les overlays placeholder du portrait (monogramme initiales + label "Portrait à fournir") se masquent-ils automatiquement quand la photo est chargée (pattern `onload` + `.has-photo`) ? Sinon → ajouter le pattern, sinon le rendu final superpose le monogramme sur la photo réelle.
>   - [ ] La photo portrait a-t-elle un duotone subtil (filter grayscale + voile multiply + lumière overlay) qui la fond dans la palette brand ? Sinon → ajouter, sans quoi la photo ressort comme un corps étranger dans la composition.
>   - [ ] Présence de gradients violet, de blobs colorés, de cards rounded-2xl shadow-xl générique ? Si oui → retravailler dans la direction éditoriale du brand.md.
>   - [ ] Animations CSS présentes et calibrées au registre (hero load stagger, scroll reveal, hover accent, FAQ smooth, parallax doux) ? Si juste statique → ajouter au minimum hero load + scroll reveal.
>   - [ ] Bloc `@media (prefers-reduced-motion: reduce)` présent ? Si absent → ajouter (accessibilité).
>   - [ ] Tous les CTAs ont-ils une URL réelle (Calendly, mailto, page) ou des `href="#"` orphelins ? Si orphelin → bloquant, repasser A.5.0.quater.
>   - [ ] Tous les liens externes ont-ils `target="_blank" rel="noopener noreferrer"` ? Si manquant → ajouter (sécurité + UX).
>   - [ ] Étape A.1.f exécutée AVANT toute interview ? Scan des 4 `.txt` sources (`description-services.txt`, `infos-entreprise.txt`, `temoignages-clients.txt`, `images/cas-clients/texte-cas-clients.txt`) effectué et variables remplies ? Si non → relancer A.1.f, sinon le skill repose des questions auxquelles l'utilisateur a déjà répondu.
>   - [ ] Étape A.5.0.quater complétée AVANT toute génération HTML (sauf si pré-consommée depuis `infos-entreprise.txt`) ? Si non → arrêter, lancer la vague.
>   - [ ] Sections `<section id="legal">` avec accordions `#mentions-legales` et `#confidentialite` présentes en pied de landing (single-page) ? Si fichiers `.html` séparés générés sans demande explicite → consolider en accordions.
>   - [ ] Footer affiche-t-il les destinations minimum (email mailto, LinkedIn si disponible, CTA diagnostic, mentions+confidentialité) ? Si une manque → vérifier que la donnée a bien été collectée en A.5.0.quater, sinon appliquer le fallback documenté.
>
> **Format de sortie attendu** : [HTML+Tailwind / React+shadcn / Framer
> sections / Webflow components — à préciser par l'utilisateur au moment
> du copier-coller].
>
> Maintenant lis tout le brief ci-dessous et génère la landing.

---

> Structure + copy de landing générée depuis brand.md.
> Ton dominant : [Q1] · Style accroches : [Q5] · Style CTAs : [Q6]
> Formule signature : "[citation depuis brand.md]"

---

## 1 · Hero
[copy rédigée — pas de [placeholders] sauf pour visuels et data externe]

## 2 · [Section suivante du template]
[copy rédigée]

...

## [N] · CTA final
[copy rédigée incluant la formule signature]

---

## Footer
[copy rédigée]

---

## Variantes hero (A/B test)

> 2 alternatives au hero principal de la section 1 — à tester en post-launch.
> Toutes les 2 calibrées Q5 / Q1 / persona, mais l'angle d'attaque diffère.

### Variante B — [Angle : question ouverte / chiffre / affirmation]
**H1** : [copy]
**Sous-titre** : [copy]
**Pourquoi tester** : [1 ligne — quel angle elle teste vs la principale]

### Variante C — [Angle restant]
**H1** : [copy]
**Sous-titre** : [copy]
**Pourquoi tester** : [1 ligne]

---

## Notes de production

- **Mots à utiliser** dans toute la page : [reprise depuis brand.md]
- **Mots à éviter** : [reprise depuis brand.md]
- **Formule signature** : "[citation]" — placée en hero (variation) + CTA final (intégrale)
- **Ambiance visuelle** : [Q11] · **Univers de référence** : [Q12]
- **Icônes & illustrations** : [Q14bis — librairie + style imposés]
- **Aucune mention de prix** sur cette landing — tarification gérée
  ailleurs (RDV / page dédiée)
- **Données à compléter par l'utilisateur** :
  - [ ] Témoignages clients (section [N])
  - [ ] Logos clients (section [N])
  - [ ] Chiffres de résultats (sections [N])
  - [ ] Visuels et illustrations selon les briefs (sections [N])
```

### A.5.e — Emplacement du fichier

Écrire `landing-page.md` **à côté de `brand.md`** (même dossier projet). Ne
pas écraser un éventuel `landing-page.md` existant sans demander confirmation
— créer une sauvegarde `landing-page.md.bak-YYYYMMDD-HHMM` avant.

### A.5.f — Lint sémantique (audit auto avant annonce)

**Avant** de présenter le fichier à l'utilisateur, exécuter un audit
automatique du `landing-page.md` qu'on vient d'écrire, en le re-parsant
en mémoire et en le confrontant à `brand.md`. C'est ce qui transforme la
copy générée en copy **calibrée**.

**6 vérifications obligatoires** :

| # | Règle | Détection | Si échec |
|---|---|---|---|
| 1 | **Mots à éviter, 0 occurrence** | Pour chaque mot de « Mots à éviter » de brand.md, `count(landing-page.md, mot) === 0` (insensible casse, hors bloc Prompt) | Lister chaque mot trouvé + sa section + proposer un remplacement issu de « Mots à utiliser » |
| 2 | **Formule signature, exactement 2 occurrences** | `count(landing-page.md, formule_signature) === 2` (version condensée hero + intégrale CTA final) | Signaler si 0/1/3+. Si 0 → bloquer. Si 1 → suggérer où l'ajouter. Si 3+ → suggérer où la retirer. |
| 3 | **Mots à utiliser, ≥ 5 occurrences distinctes** | Au moins 5 mots distincts de « Mots à utiliser » présents au moins 1× | Lister les mots clés non utilisés et proposer 2-3 sections où les intégrer naturellement |
| 4 | **Pas de prix** | Aucune occurrence de `€`, `EUR`, `$`, `USD`, `prix`, `tarif`, `€/mois`, `forfait` (hors bloc Prompt et hors brief visuel) | Signaler chaque occurrence + remplacer par CTA de qualification |
| 5 | **Objection #1 (Q16) présente dans la FAQ** | La FAQ contient une question dont le sens correspond à l'Objection #1 verbatim (matching sémantique sur les mots-clés du Q16) | Ajouter la FAQ correspondante en tête de section |
| 6 | **Zéro tiret cadratin `—` et `–`** *(règle d'écriture transversale #1)* | `count(landing-page.md, '—') === 0` ET `count(landing-page.md, '–') === 0` (hors bloc Prompt, hors séparateurs markdown `---`, hors commentaires `<!-- -->`) | Lister chaque occurrence + sa ligne + proposer le remplacement contextuel (`,`, `:`, `;`, `(...)`, ` · `, ou point + nouvelle phrase) selon le tableau de la règle #1 |

**Sortie de l'audit** : un mini-rapport en 6 lignes dans la conversation
**avant** de passer à A.5.f.bis :

```
✦ Lint landing-page.md
✅ Mots à éviter : 0 occurrence
✅ Formule signature : 2 occurrences (hero §1 + CTA final §[N])
✅ Mots à utiliser : 7/8 utilisés (manque : « rituel »)
✅ Pas de prix mentionné
✅ Objection #1 (Q16) en FAQ §[N], question 1
✅ Zéro tiret cadratin (règle écriture #1)
```

Si un check échoue → **corriger automatiquement** le fichier (réécrire les
phrases concernées) puis re-lancer le lint. Maximum 2 passes ; au 3e
échec sur le même check, signaler à l'utilisateur et demander arbitrage.

### A.5.f.bis — Annonce à l'utilisateur

Après écriture **et lint validé**, annoncer en 5 lignes max :

```
✦ landing-page.md écrit & audité
Template : [A · Offre freelance / B · Produit / C · Personal brand]
Sections : [liste compacte des sections]
Hero : 1 principal + 2 variantes A/B en fin de fichier
À compléter par toi : [3-4 items — témoignages, logos, chiffres, visuels selon briefs]
Note : aucune mention de prix dans le MD (par design — tarif géré ailleurs).
→ Tu peux copier-coller le fichier entier dans Claude / Framer AI / v0 /
  Lovable pour générer la landing — le bloc « ✦ Prompt » en tête fait le
  cadrage (rôle UX/UI designer landing, contraintes, format de sortie).
```

Puis **enchaîner sur la Phase B** (génération brochure Figma) sans question
intermédiaire — sauf en mode LANDING où on s'arrête ici.

### A.5.g — Génération de `landing.html` + `landing-content.txt` (production-ready single-file)

> Étape finale après la rédaction de `landing-page.md` (en mode CRÉATION ou LANDING). Elle produit la **landing HTML production-ready** ET le fichier compagnon `landing-content.txt` qui contient uniquement les textes IA-générés (non sourcés des autres `.txt`). Ce binôme permet à l'utilisateur d'éditer le contenu sans toucher au HTML.

#### A.5.g.1 — Pré-requis pour génération HTML

Avant de générer, vérifier :

- [ ] Les SVG `./logo/logo.svg` et `./logo/logo-white.svg` existent (sinon bloquer, cf. règle LOGO).
- [ ] Toutes les variables de A.5.0.quater sont collectées (sinon relancer, cf. règle PRODUCTION-READY).
- [ ] Les 4 `.txt` sources ont été lus en A.1.f, leurs valeurs sont en mémoire.
- [ ] Si `./images/hero.jpg` absent, le fallback CSS riche + Unsplash hot-link est prêt à être inséré.

#### A.5.g.2 — Génération `landing.html`

Suivre toutes les règles documentées dans le bloc Prompt de `landing-page.md` (LOGO règle absolue, IMAGES règle gracieuse, PRODUCTION-READY liens vivants, ANIMATIONS CSS, anti-AI-slop checklist).

Single-page self-contained : `<section id="legal">` avec accordions `#mentions-legales` et `#confidentialite` en pied de landing, contenu rempli depuis `infos-entreprise.txt`.

#### A.5.g.3 — Génération `landing-content.txt`

**Objectif** : produire un fichier compagnon qui contient **uniquement** les textes que le skill a rédigés (et qui ne viennent pas des `.txt` sources). L'utilisateur peut éditer ce fichier puis demander une resynchronisation (mode SYNC) pour mettre à jour `landing.html` sans relancer l'interview.

**Règle de séparation** :

| Type de texte | Source | Présent dans landing-content.txt ? |
|---|---|---|
| Promesse, formule signature, différenciateur, persona, méthode (3 étapes), objection #1, texte CTA principal | `description-services.txt` | **Non** |
| Mentions légales, RGPD, hébergeur, URLs, email | `infos-entreprise.txt` | **Non** |
| 3 verbatim témoignages + attributions | `temoignages-clients.txt` | **Non** |
| Titres + briefs + livraisons des 3 cas | `texte-cas-clients.txt` | **Non** |
| Eyebrows de section (`SÉLECTION DE TRAVAUX`, `MÉTHODE FOKUSIA`, etc.) | IA | **Oui** |
| H1 hero condensé (version courte de la signature) | IA | **Oui** |
| H2 de section (« Trois projets, trois bascules… ») | IA | **Oui** |
| Paragraphes intro de chaque section | IA | **Oui** |
| 3 bullets du Constat (titre + description) | IA | **Oui** |
| Hook italic + 2 paragraphes À propos + 3 bullets posture | IA | **Oui** |
| Réponses FAQ Q2 à Q5 (Q1 est l'objection #1 du source) | IA | **Oui** |
| Sous-titre CTA final | IA | **Oui** |
| Variantes hero B et C (H1 + sous-titre + raison test) | IA | **Oui** |
| Eyebrow location, num signature, scroll indicator, link prefix | IA | **Oui** |

**Format `landing-content.txt`** : structure section par section, paire `Champ : valeur` simple. Lignes `#` ignorées (commentaires). Exemple complet :

```
# landing-content.txt
# Édité par toi · resync via « resynchronise la landing »

## Hero
Eyebrow location : Paris · 2026
H1 condensé : Rigueur. Méthode. Autorité.
Sous-titre : Identité visuelle pour les cabinets d'investigation…
CTA secondaire : Voir les projets récents

## Portfolio
Eyebrow section : SÉLECTION DE TRAVAUX
H2 section : Trois projets, trois bascules vers le corporate.
Paragraphe intro : Chaque mission documentée par son brief original…

## Constat
Eyebrow : CE QUE NOUS OBSERVONS
H2 : Votre expertise est en béton. Votre image, non.
Paragraphe intro : …
Bullet 1 titre : L'image dit « particulier ».
Bullet 1 description : Codes années 90…
[etc.]
```

#### A.5.g.4 — Annonce à l'utilisateur

```
✦ landing.html + landing-content.txt générés
Fichiers : landing.html (production-ready) + landing-content.txt (textes éditables)

Pour modifier les textes (typo, reformulation, ajustement) :
1. Édite landing-content.txt (ou un des 4 .txt sources)
2. Dis-moi « resynchronise la landing » → je relis tout et regénère landing.html
3. Pas besoin de relancer l'interview ni de toucher au HTML

Tous les liens sont vivants : Calendly, mailto, LinkedIn, mentions/confidentialité en accordions.
À compléter dans infos-entreprise.txt : [liste des champs vides détectés]
```

---

## Mode SYNC — Resynchronisation rapide depuis les `.txt`

> Mode déclenché par les triggers « resynchronise la landing », « mets à jour la landing depuis les txt », « regen depuis les txt », « sync content ». **Pas d'interview, pas de questions, pas de Figma.** Le skill relit les 5 `.txt` et regénère uniquement `landing.html`.

**Étapes** :

1. **Détection** : vérifier que `landing.html` et `landing-content.txt` existent à la racine. Si absent, basculer sur LANDING ou CRÉATION selon l'état.
2. **Re-scan** des 5 `.txt` (les 4 sources + `landing-content.txt`), suivant la priorité documentée en A.1.f (`.txt` > AskUserQuestion > inférence).
3. **Diff diagnostic** : comparer les valeurs lues aux valeurs précédentes (heuristique simple : compter le nombre de champs modifiés). Annonce :
   ```
   ✦ Resynchronisation détectée
     · landing-content.txt : 3 champs modifiés (Hero H1, Bullet 2 titre, FAQ Q3 réponse)
     · description-services.txt : inchangé
     · infos-entreprise.txt : 2 champs modifiés (URL Calendly, Téléphone hébergeur)
     · temoignages-clients.txt : inchangé
     · texte-cas-clients.txt : inchangé
   ```
4. **Regénération `landing.html`** en réutilisant exactement la même structure et le même CSS qu'avant. Seuls les contenus changent.
5. **Lint** rapide : check qu'aucun `href="#"` n'est apparu, que les overlays placeholder du portrait sont toujours cachés quand photo présente, que les accordions mentions/confidentialité sont remplis.
6. **Confirmation** :
   ```
   ✦ landing.html resynchronisée
   Modifications appliquées en 3 secondes. Pas d'interview. Pas de Figma touché.
   Tu peux recharger landing.html dans ton navigateur.
   ```

**Garanties du mode SYNC** :
- Aucune question posée à l'utilisateur (sauf si un `.txt` source est devenu illisible / parsé invalide).
- Aucune modification du `brand.md` ni du `landing-page.md`.
- Aucun lint sémantique des règles d'écriture (déjà validé à la génération initiale) — sauf vérification production-ready (zéro href orphelin).
- Régénération idempotente : relancer le mode SYNC sans modification donne exactement le même `landing.html`.

---

# Phase B — Génération de la brochure Figma

## Étape 1 — Charger le contexte

Lire **uniquement `brand.md`** (source de vérité unique) et en extraire,
par section :

1. **Identité** :
   - Nom de la marque + tagline (titre H1)
   - Positionnement (Qui / Pour qui / Promesse / Différenciateur)
   - Identité verbale (formule signature, mots à utiliser/éviter, CTAs)
   - Personnalité graphique (3-5 règles)

2. **Tokens — parsés depuis les tables markdown du même fichier** :
   - Table **Couleurs** → primary, primary dark, accent, atmospheric, neutres
   - Tables **Typographie** (Familles + Échelle) → display, body, h1→small
   - Table **Espacements** → xs / sm / md / lg / xl / 2xl
   - Table **Border radius** → sm / md / lg / full
   - Liste **Composition rules** → 6 règles `**Nom** — description courte`
     (utilisées telles quelles dans la slide 12 Key Elements)

> **Parsing** : Claude lit les tables markdown directement — c'est natif.
> Si une table est absente ou malformée, demander à l'utilisateur de la
> compléter dans `brand.md` avant de générer (pas de fallback silencieux).

3. **Grille de mise en page** : se référer à l'Étape 8 (« Spécifications
   par slide ») qui contient les coordonnées x/y, sizing, fonts et fills
   pour chacune des 15 slides. **C'est la source unique** de la grammaire
   visuelle — ne pas inventer de positions, ne pas relire de fichier YAML
   externe.

---

## Étape 2 — Compléter les inputs manquants en une question groupée

Si certaines infos manquent (URL Figma, handle social pour le footer,
nombre de couleurs à exposer), poser **une seule question groupée**.
Pas de va-et-vient.

Exemple :
> Pour générer le brand book j'ai besoin de :
> 1. L'URL du fichier Figma cible
> 2. Confirmer le nombre de couleurs à exposer en palette (4, 5 ou 6)
> 3. (mode BOOK sans preset dans brand.md) Confirmer le preset visuel
>    inféré : [preset] — ou en choisir un autre parmi les 5

---

## Étape 3 — Mapper YAMLs templates → slides à produire

Le brand book final contient **15 slides** 1920×1280. Chaque slide
réutilise la composition d'un YAML template, en y injectant les tokens
du brand actuel + les réponses de l'interview (Phase A.2).

| # | Slide | Tokens / contenu injectés | Slots images |
|---|---|---|---|
| 01 | Cover | Fond preset + eyebrow `IDENTITÉ DE MARQUE · [année]` + logo + nom marque XXL + accent line + formule signature | `cover` (bande droite pleine hauteur) |
| 02 | Sommaire | Liste des **5 chapitres** numéros + intitulés à gauche, descriptions + hairlines à droite | `sommaire-1`→`sommaire-5` (collage central) |
| 03 | **Verbal Identity** | Ton dominant + formule signature en grand + 2 colonnes **Mots à utiliser** / **Mots à éviter** (barrés) + 3 CTAs exemples + ligne posture | `identite` (visuel droite) |
| 04 | **Universe & Mood** | Fond immersif — 4 attributs (ambiance, référence, imagerie, personnalité) + citation manifeste | `univers-1`→`univers-6` (grille 3×2 labellisée) |
| 05 | Chapter opener — Logo | Eyebrow `CHAPITRE 02` + mot **"Logo"** géant en bas-gauche + sous-titre chapitre bas-droite | `chapitre-logo` (visuel droite) |
| 06 | Content — Logo | Variantes du logo (3 cartouches) + taille minimale + zone de protection | `logo-hero` (grand visuel haut) |
| 07 | Chapter opener — Couleur | Eyebrow `CHAPITRE 03` + mot **"Couleur"** géant | `chapitre-couleur` (bande verticale droite) |
| 08 | Content — Palette de couleurs | 4-6 **colonnes couleur 186×960** côte à côte — nom + HEX + RGB | `palette-col-<n>` (insert image dans chaque colonne, optionnel) |
| 09 | Content — Nuances | Échelle de tints `primary` et `accent` (100/80/60/40/20), **opacité pré-mélangée sur le fill** | `nuances` (visuel gauche) |
| 10 | Chapter opener — Typographie | Eyebrow `CHAPITRE 04` + mot **"Typographie"** géant | `chapitre-typo` (visuel droit, bleed haut autorisé selon preset) |
| 11 | Content — Police | 2 « Aa » géants (display + body) + alphabet complet + noms des typefaces | `police` (visuel bas-gauche, bleed selon preset) |
| 12 | Content — Hiérarchie typographique | Cascade Titre 1→Légende avec specs et phrases d'exemple | `hierarchie` (panneau droite pleine hauteur) |
| 13 | Chapter opener — Système | Eyebrow `CHAPITRE 05` + mot **"Système"** géant | `chapitre-systeme` (visuel droite) |
| 14 | Content — Éléments clés | 6 cards 3×2 depuis la liste **Composition rules** de `brand.md` | `elements` (bandeau panoramique bas) |
| 15 | Back cover | Logo centré + **"Merci"** + formule signature + crédits | aucun |

**Mots entiers, jamais coupés** : sur les chapter openers, le mot
("Logo", "Couleur", "Typographie", "Système") tient sur une seule ligne,
taille réduite si besoin.

**Règle d'or** : chaque slide doit pouvoir être lue isolément — header
section + titre + footer **page XX/15** + handle.

> **Pourquoi 2 slides en plus (03 et 04)** : rendre **visibles** dans
> le brand book les réponses de l'interview (identité verbale + univers).
> Sans ces slides, ces données restent invisibles côté Figma et le
> livrable perd sa valeur de présentation client.

---

## Étape 4 — Grammaire de mise en page (héritée des YAMLs)

Toutes les slides respectent ces règles, extraites des YAMLs :

### Format
- **Taille** : 1920 × 1280 px (format éditorial choisi pour qualité PDF
  client, identique aux YAMLs templates)
- **Padding** : 80px en haut, 80px à gauche, 80px à droite, 132px en bas
  (pour laisser place au footer)
- **Zone de contenu utile** : 1760 × 1068 px (x=80, y=80)

### Colonne gauche (slides "content")
Sur les content sections, la colonne gauche (x=80, y=160, width=347) contient :
- Titre de section en 40px regular (display) — couleur `neutral.darkest`
- Paragraphe descriptif en 20px regular (body) — issu de `brand.md`
- Texte long et aéré (gap 32px)

### Zone visuelle (droite)
À partir de x=520, contenu visuel (palette, swatches, typo, logo…).

### Slots images (toutes les slides sauf 15)
Chaque slide porte 1 à 6 slots images placés via `placeImage()` (Étape 5).
Un slot rempli = image clonée depuis le node `IMG · <slot>` du Figma
cible. Un slot vide = rectangle dégradé aux couleurs du preset, nommé
`MoodBoard · <slot>`, sur lequel l'utilisateur peut drag-drop une photo
plus tard (Figma applique l'image en fill automatiquement). Les
**débordements hors cadre** (bleed) sont autorisés sur certains slots
(police, chapitre-typo, hierarchie) **selon le preset** : `clipsContent:
true` sur la slide fait le rognage proprement.

### Footer (toutes les slides)
- À y=1228, hauteur 24
- Gauche (x=80) : **nom de la marque en capitales** (ex « MAX PICCININI »)
  en 16px regular, letter-spacing 2px — couleur adaptée au fond. Le handle
  (@marque) n'est utilisé que si l'utilisateur le demande explicitement.
- Droite (x=1757) : "page XX/15" en 16px regular — même couleur

### Chapter openers (grammaire éditoriale)
- Fond pleine couleur **selon le preset** (Étape 4bis) : luxe = tous
  sombres, pop = accent full bleed, etc.
- Eyebrow **`CHAPITRE 02`** / `03` / `04` / `05` en haut-gauche (x=80,
  y=120), 19px Semi Bold, letter-spacing 4px, couleur accent ou contrastée
- Accent line 120×4 posée au-dessus du mot géant
- Grand mot en **bas-gauche** (le bloc texte se termine vers y=1040),
  Display du preset, **mot entier sur une seule ligne — jamais coupé en
  plein milieu d'un mot**. Si le mot ne tient pas, **réduire la taille de
  police jusqu'à ce qu'il tienne** dans ~60% de la largeur utile (le slot
  image occupe la droite). Échelle indicative selon le mot :
  - 4 lettres ("Logo") → 340-400px possibles
  - 7 lettres ("Couleur", "Système") → 240-300px
  - 11 lettres ("Typographie") → 170-210px
- **Slot image à droite** (`chapitre-logo`, `chapitre-couleur`,
  `chapitre-typo`, `chapitre-systeme`) : dimensions par chapitre dans
  l'Étape 8 — c'est lui qui donne la respiration éditoriale
- Sous-titre du chapitre en **bas-droite** (x=1140, y=1086, w=680, align
  right, 22px Medium) : le contenu du chapitre en 3 items séparés par
  ` · ` (ex « Variantes · Zone de protection · Taille minimale »)
- Couleur contrastée selon le fond (cream sur sombre, primary sur accent clair)
- Pas de colonne descriptive

> **Pourquoi ne pas casser les mots** : couper "Typo / graphy" ou "Lo / go"
> donne un effet "magazine éditorial" qui peut sembler stylé mais qui :
> (1) nuit à la lisibilité pour les non-francophones / non-anglophones,
> (2) brise la grammaire typographique propre et signale "design forcé",
> (3) rend la slide ambiguë au premier regard (« c'est quoi ce mot tronqué ? »).
> On préfère un mot entier lisible, quitte à réduire la taille.

---

## Étape 4bis — Sélectionner le preset visuel (5 styles)

**La structure des 15 slides est fixe ; le rendu ne l'est pas.** La même
grammaire (slots images, chapter openers, collage sommaire) doit pouvoir
produire un brand book de maison de luxe ET un brand book de marque pop.
Le preset est la **peau** appliquée sur le squelette. Le choisir AVANT de
coder la moindre slide, et l'écrire dans `brand.md` (section Univers
visuel → `**Preset visuel**`) pour que le mode BOOK le retrouve.

### Les 5 presets

| Preset | Ambiance | Fonds (cover / content / chapters / back) | Display typique | Traitement des images & placeholders | Radius | Signature |
|---|---|---|---|---|---|---|
| **Luxe éditorial** | Sombre, dramatique, muséal | `primaryDark` partout, texte cream | Serif (Playfair, Cormorant) | Duotone teinté primary, sombres ; bleeds hors cadre autorisés (slots police, chapitre-typo, hierarchie) ; placeholder = dégradé vertical `noir→primaryDark` + voile accent 8% | 8-12 | Accent chirurgical (<10%), white space massif, verticalité |
| **Minimal tech** | Clair, précis, suisse | `white` partout, chapters : 3 `white` texte primary + 1 `accent` | Sans géométrique (Inter, Helvetica) | Images cadrées strictes, jamais de bleed, hairline border 1px ; placeholder = aplat `gris froid 4%` + grille fine accent 8% | 0-4 | Grilles visibles, numérotation, densité maîtrisée |
| **Crème organique** | Chaleureux, artisanal, matière | `cream` partout, chapters alternance `primary` / `accent` adouci | Serif doux ou sans premium (Manrope) | Images naturelles chaudes, pas de duotone ; placeholder = dégradé diagonal `cream→accent 25%` | 16-24 | Rondeur, textures, tons chauds |
| **Pop contrasté** | Énergique, franc, direct | Alternance `white` / `accent` full bleed / `noir` ; chapters tous `accent` | Display bold XXL | Images saturées, cadres pleins 4-8px `noir` ; placeholder = 2 aplats tranchés `accent`+`primary` en diagonale dure (pas de fondu) | 16-24 ou 999 (pills) | Contrastes durs, chips visibles, tailles +20% |
| **Corporate clair** | Institutionnel, rigoureux, premium | `white` + chapters `primary`, back `primaryDark` | Sans premium (Inter, Manrope) | Photo documentaire désaturée ou N&B ; placeholder = dégradé discret `primary→primaryDark` opacité 90% | 4-8 | Sobriété, alignements stricts, eyebrows tracking large |

### Sélection automatique depuis l'interview

Chaque réponse vote pour un ou plusieurs presets. Compter les voix :

| Réponse | Vote |
|---|---|
| Q11 Sombre & dramatique | Luxe éditorial |
| Q11 Clair & aéré | Minimal tech, Corporate clair |
| Q11 Crème & chaleureux | Crème organique |
| Q11 Contrasté noir/blanc | Pop contrasté |
| Q1 Confidentiel & premium | Luxe éditorial, Corporate clair |
| Q1 Direct & tranchant | Pop contrasté |
| Q1 Chaleureux & humain | Crème organique |
| Q1 Technique & expert | Minimal tech, Corporate clair |
| Q8 Serif éditorial | Luxe éditorial, Crème organique |
| Q8 Géométrique moderne | Minimal tech |
| Q8 Sans-serif premium | Corporate clair, Crème organique |
| Q8 Display dégaussé | Pop contrasté, Luxe éditorial |
| Q12 Architecture & passages | Luxe éditorial, Corporate clair |
| Q12 Nature & matières brutes | Crème organique |
| Q12 Tech minimale & grilles | Minimal tech |
| Q12 Éditorial luxe & mode | Luxe éditorial |
| Q14 Sobre / Premium | Luxe éditorial, Corporate clair |
| Q14 Audacieux | Pop contrasté |
| Q14 Chaleureux | Crème organique |
| Q14 Technique | Minimal tech |
| Q14 Émotionnel | Luxe éditorial, Crème organique |

**Décision** :
- Un preset domine de ≥2 voix → le prendre, l'annoncer en 1 ligne dans
  le récap (« Preset : Luxe éditorial, dérivé de tes réponses sombre +
  serif + premium »). Pas de question.
- Égalité ou écart de 1 voix → poser **une seule** `AskUserQuestion`
  avec les 2-3 presets en tête + 1 ligne de description chacun, le
  favori marqué *(Recommandé)*.
- Mode BOOK : lire `**Preset visuel**` dans brand.md. S'il est absent,
  inférer depuis les sections Univers visuel + Identité verbale, et
  glisser la confirmation dans la question groupée de l'Étape 2.

**Le preset pilote** : les constantes `PRESET.bg.*` (fonds par type de
slide), `PRESET.display` (famille titres), `PRESET.imageRadius`,
`PRESET.placeholderFill()` (recette de dégradé des slots vides),
`PRESET.allowBleed` (autoriser les images hors cadre), `PRESET.frame`
(bordures d'images). L'Étape 4ter affine ensuite par-dessus, réponse
par réponse.

---

## Étape 4ter — Adaptation fine selon `brand.md` (par-dessus le preset)

**Règle fondamentale** : le brand book n'a **pas un layout fixe**. Les choix
visuels (fond des slides, tailles des titres, intensité de l'accent, fonts,
densité) **s'adaptent aux réponses d'interview** stockées dans la section
*Identité verbale* + *Univers visuel* + *Typographie* de `brand.md`.

Avant de générer la première slide, **lire les valeurs suivantes** dans
`brand.md` et appliquer le mapping ci-dessous :

### A · Ambiance lumineuse (Q11) → fond dominant des slides cream-default

| Valeur Q11 | Fond Cover / Index / Logo / Palette / Shades / Type Hierarchy / Key Elements |
|---|---|
| **Crème & chaleureux** *(défaut)* | `cream` |
| **Clair & aéré** | `white` |
| **Sombre & dramatique** | `primaryDark` (texte cream) |
| **Contrasté noir/blanc** | `white` (texte primary, accents tranchés) |

Les slides *Universe & Mood*, *Typeface*, *Back Cover* gardent `primaryDark`
quelle que soit Q11 (atmosphère immersive intentionnelle).

### B · Personnalité typographique (Q8) → familles de fonts

| Valeur Q8 | Display | Body | Fallback si non chargée |
|---|---|---|---|
| **Géométrique moderne** *(défaut)* | Inter Bold | Inter Regular | — |
| **Serif éditorial** | Playfair Display Bold | Inter Regular | Inter Bold pour Display |
| **Sans-serif premium** | Manrope Bold | Inter Regular | Inter Bold pour Display |
| **Display dégaussé** | Editorial New Bold | Inter Regular | Inter Bold pour Display |

Le **preload** au démarrage (Étape 5) doit inclure la famille Display choisie
+ Inter. Si la famille n'est pas disponible, fallback explicite sur Inter et
**signaler à l'utilisateur** : « Manrope non disponible, fallback sur Inter. »

### C · Échelle typographique (Q9) → tailles H1 / Body + titre cover

| Valeur Q9 | H1 | H2 | H3 | Body | Small | Titre cover (signature XXL) |
|---|---|---|---|---|---|---|
| **Contraste fort** | 56 | 40 | 28 | 16 | 13 | **200px** |
| **Mesurée** *(défaut)* | 40 | 32 | 24 | 18 | 14 | **140px** |
| **Dramatique** | 96 | 56 | 32 | 18 | 14 | **240-280px** |
| **Subtile / uniforme** | 32 | 28 | 24 | 17 | 14 | **100px** |

### D · Usage de l'accent couleur (Q10) → intensité dans chapter openers

| Valeur Q10 | Fond chapter openers (slides 05/07/10/13) | Accent visible |
|---|---|---|
| **Chirurgical (<10%)** *(sobriété max)* | Tous en `primaryDark` (cream texte) | Accent uniquement en eyebrow, numéro, accent line |
| **Mesuré (10-20%)** | 1 chapter sur 4 en `accent`, 3 en `primary/primaryDark` | Accent ponctuel |
| **Présent (20-30%)** *(défaut)* | 2 chapter sur 4 en `accent`, 2 en `primary/primaryDark` | Alternance accent / sombre |
| **Pleine couleur (full bleed)** | **Tous les 4 chapter openers en `accent`** | Pleine couleur |

### E · Référence d'univers (Q12) → motifs décoratifs subtils

| Valeur Q12 | Motif appliqué |
|---|---|
| **Architecture & passages** | Slide 04 Universe : ajouter 1-2 lignes courbes subtiles (arches) en `accent` opacity 30% en fond |
| **Nature & matières brutes** | Slide 04 Universe : ajouter texture organique (rectangles opacity 5-10% en fond) |
| **Tech minimale & grilles** | Toutes les slides : afficher les colonnes de grille en `accent` opacity 8% en fond (subtil) |
| **Éditorial luxe & mode** | Marges plus généreuses (+20-30%), rectangles d'image placeholder en bordure |

Ces motifs sont **optionnels** et **toujours subtils** (opacity ≤ 30%). Ne
jamais ajouter un motif qui sature visuellement la slide.

### F · Personnalité graphique mots-clés (Q14, multiSelect) → densité

| Mot-clé | Impact |
|---|---|
| **Sobre** | Moins de texte, plus de white space. Cards sans shadow. Pas de chips colorés. Border radius 8 max |
| **Audacieux** | Tailles +20%, chapter openers full bleed accent, chips visibles |
| **Premium** | Border radius 4-8 (au lieu de 16-24). Tracking augmenté sur eyebrows. Marges +10% |
| **Chaleureux** | Border radius 16-24. Cream chaud dominant. Cards avec ombre douce |
| **Technique** | Grilles visibles (cf. Q12 Tech minimale). Tableaux structurés. Numérotation visible |
| **Émotionnel** | Plus d'imagerie (placeholders). Moodboard étendu. Titres XXL |

Si plusieurs mots-clés sont cochés, **prioriser la cohérence** : Sobre + Premium = layout très épuré ; Audacieux + Émotionnel = layout impactant.

### G · Ton dominant (Q1) → choix de fond cover

Q1 module **subtilement** le fond cover par-dessus Q11 :

| Q1 | Ajustement |
|---|---|
| **Confidentiel & premium** | Cover sur `cream` ou `primaryDark` (jamais `accent`) |
| **Direct & tranchant** | Cover sur fond contrasté (`primary` ou `accent` full bleed acceptable) |
| **Chaleureux & humain** | Cover sur `cream` ou tons chauds |
| **Technique & expert** | Cover sur `cream` ou `white` (rigueur), pas de fond émotionnel |

---

### Exemple : 2 identités opposées → 2 brand books visuellement différents

| Aspect | Identité A *(Confidentiel · Crème · Mesurée · Chirurgical · Sobre)* | Identité B *(Direct · Sombre · Dramatique · Pleine couleur · Audacieux)* |
|---|---|---|
| Cover | Fond `cream`, titre 140px primary | Fond `accent` ou `primaryDark`, titre 280px cream |
| Chapter openers | Tous sombres + accent ponctuel | Tous en `accent` full bleed |
| Cards Key Elements | Sans shadow, border radius 8 | Avec shadow, border radius 16+ |
| Chips Universe | Texte simple (pas de chips colorés) | Chips full bleed accent |
| Densité | Mots à utiliser : 6-8 max | Mots à utiliser : 10-12 (plus dense) |

> **Comment l'agent applique ce mapping** : avant de coder l'appel use_figma,
> il lit `brand.md` (sections Identité verbale + Univers visuel + Typographie),
> identifie chaque réponse de l'interview, et **modifie les paramètres**
> (constantes `BG_COVER`, `TITLE_SIZE`, `CHAPTER_BG_LIST`, etc.) avant
> d'appeler les helpers `solidVar()` et `makeSlide()`.

---

## Étape 5 — Génération Figma (use_figma) — code structuré

### Règle de langue — **tout le contenu visible du Figma est en français**

Tous les libellés affichés dans les slides (titres de chapitre, eyebrows,
sommaire, légendes, footer, phrases d'exemple, mentions techniques) doivent
être en **français**. Les seules exceptions tolérées :
- Noms des fonts (« Inter Display », « Manrope ») — c'est le nom officiel
- Noms des variables Figma internes (`primary`, `accent`, `cream`…) — ce sont
  des tokens techniques, jamais affichés sous forme de texte dans les slides
- Le glyphe "Aa" de la slide Police (typographique, universel)
- Codes hex / RGB / CMYK / Pantone

**Tableau de référence des libellés** (à utiliser tel quel comme `characters`
des nodes texte — ne **jamais** générer une variante anglaise ou abrégée
sans demande explicite) :

| Concept | Libellé Figma |
|---|---|
| Sommaire (slide 02) | **Sommaire** |
| Chapitre Identité | **Identité** |
| Chapitre Logo | **Logo** |
| Chapitre Couleur | **Couleur** |
| Chapitre Typographie | **Typographie** |
| Chapitre Système | **Système** |
| Titre slide Verbal Identity | **Identité verbale** *(2 lignes : "Identité" / "verbale")* |
| Titre slide Universe & Mood | **Univers & ambiance** *(2 lignes : "Univers" / "& ambiance")* |
| Slide Palette de couleurs | **Palette de couleurs** |
| Slide Nuances | **Nuances** |
| Slide Police | **Police** |
| Slide Hiérarchie typographique | **Hiérarchie typographique** |
| Slide Éléments clés | **Éléments clés** |
| Phrase « Smallest logo size » | **Taille minimale du logo** |
| Phrase « Thank you » (back cover) | **Merci** |
| Mention « Generated by brand-book skill » | **Généré par le skill brand-book** |
| Eyebrows sections (ex `IDENTITY` uppercase) | **IDENTITÉ**, **LOGO**, **COULEUR**, **TYPOGRAPHIE**, **SYSTÈME** (uppercase ASCII conservé pour les caractères accentués) |
| Phrase d'exemple type hierarchy (slide 12) | **« Nous ne livrons pas un produit, mais la solution complète. »** *(défaut — peut être remplacée par la formule signature du brand.md si présente)* |
| Sommaire lignes (slide 02) | **« 01 Identité »**, **« 02 Logo »**, **« 03 Couleur »**, **« 04 Typographie »**, **« 05 Système »** |
| Eyebrow chapter openers | **« CHAPITRE 02 »** … **« CHAPITRE 05 »** |
| Eyebrow cover | **« IDENTITÉ DE MARQUE · [année] »** |
| Footer gauche | **nom de la marque en capitales** (ex « MAX PICCININI ») |
| Footer page | **« page XX/15 »** *(le mot "page" reste en français — pas "Page" majuscule)* |

> **Pourquoi cette règle stricte** : Arnaud livre les brand books à des
> clients français. Les titres en anglais (« Typography », « Colour
> Palette ») cassent la lecture et signalent « template générique » au
> lieu de « brand book sur mesure ». La cohérence linguistique est un
> marqueur de qualité.

### Préambule : helpers et constantes

```js
// Nom de la marque — collecté en A.1.c (CRÉATION) ou lu dans le titre
// de brand.md (BOOK). JAMAIS une valeur d'exemple du SKILL.md.
const BRAND_NAME = "[Nom réel de la marque]";

// Couleurs depuis la table Couleurs de brand.md (à injecter).
// ⚠️ Les hex ci-dessous sont des EXEMPLES (projet fokusia) — toujours
// les remplacer par les valeurs réelles de brand.md.
const COLORS = {
  primary:        "#06211a",
  primaryDark:    "#04231E",
  accent:         "#f58220",
  cream:          "#F5EFE6",
  creamLight:     "#FAF6EE",
  bordeaux:       "#660000",
  white:          "#FFFFFF",
  ardoise:        "#1A1A2E",
  border:         "#E8DDD0",
};

const FONTS = [
  {family:"Inter", style:"Regular"},
  {family:"Inter", style:"Medium"},
  {family:"Inter", style:"Semi Bold"},
  {family:"Inter", style:"Bold"},
];

const hex = (h) => {
  const x = h.replace("#","");
  return { r: parseInt(x.slice(0,2),16)/255, g: parseInt(x.slice(2,4),16)/255, b: parseInt(x.slice(4,6),16)/255 };
};

// État du preset (Étape 4bis) + collecte des slots vides (Étape 9ter)
const PRESET = {
  id: "luxe",                       // luxe | minimal | creme | pop | corporate
  bg: { cover: "primaryDark", content: "primaryDark", immersive: "primaryDark",
        chapters: ["primaryDark","primaryDark","primaryDark","primaryDark"],
        back: "primaryDark" },      // ← valeurs pour Luxe éditorial ; adapter selon la table 4bis
  display: { family: "Playfair Display", style: "SemiBold" },
  imageRadius: 8,
  allowBleed: true,
  frame: null,                      // bordure des slots (hors masque). Minimal tech : {color:"border", w:1} ; Pop : {color:"<token sombre>", w:4} ; sinon null
  placeholderFill: (slot) => PLACEHOLDER_RECIPES["luxe"](slot),
};
const PLACEHOLDER_SLOTS = [];

await Promise.all(FONTS.map(f => figma.loadFontAsync(f)));
```

### Préambule : créer/réutiliser la Variable Collection « Brand Tokens »

**Obligatoire**. Toutes les couleurs des fills doivent être bindées à des
variables Figma — pas de hex brut dans les fills. Cela permet à l'utilisateur
de changer une couleur **une seule fois** dans le panneau Variables et toutes
les slides se mettent à jour.

```js
// 1. Trouver ou créer la collection
let collection = figma.variables.getLocalVariableCollections()
  .find(c => c.name === "Brand Tokens");
if (!collection) {
  collection = figma.variables.createVariableCollection("Brand Tokens");
}
const modeId = collection.modes[0].modeId;

// 2. Trouver ou créer les variables couleurs
const VAR_NAMES = ["primary","primaryDark","accent","cream","creamLight","bordeaux","white","ardoise","border"];
const vars = {};
for (const name of VAR_NAMES) {
  const existing = figma.variables.getLocalVariables("COLOR")
    .find(v => v.name === name && v.variableCollectionId === collection.id);
  const v = existing || figma.variables.createVariable(name, collection, "COLOR");
  v.setValueForMode(modeId, hex(COLORS[name]));
  vars[name] = v;
}

// 3. Helpers — TOUJOURS utiliser solidVar(), pas solid()
const solidVar = (varName) => [{
  type: "SOLID",
  color: hex(COLORS[varName]),     // fallback visuel
  boundVariables: { color: { type:"VARIABLE_ALIAS", id: vars[varName].id } }
}];

// Helper legacy uniquement pour cas exceptionnels (opacité custom, etc.)
const solid = (h) => [{type:"SOLID", color: hex(h)}];
```

> Réutilisation : si la collection existe déjà (cas typique d'une régénération),
> on **met à jour les valeurs** des variables existantes au lieu d'en créer de
> nouvelles. C'est ce qui permet l'idempotence.

### Préambule : créer la page

```js
let page = figma.root.children.find(p => p.name === "✦ Brand Book");
if (!page) { page = figma.createPage(); page.name = "✦ Brand Book"; }
await figma.setCurrentPageAsync(page);
```

### Préambule : helper pour placer le logo source réel

**Obligatoire**. Toutes les slides qui exposent le logo (01, 06, 15) doivent
utiliser ce helper. Il garantit qu'on n'utilise jamais un placeholder texte
"Logo" et qu'on s'appuie sur la map `LOGOS` peuplée à l'Étape A.1.d (depuis
les nodes `Logo couleur` / `Logo blanc` placés dans le Figma cible).

```js
// Stockés depuis l'Étape A.1.d (détection des nodes Figma)
const LOGOS = {
  color: tplColorNode,  // node Figma trouvé via findOne — obligatoire
  white: tplWhiteNode,  // node Figma trouvé via findOne — optionnel (null si absent)
};

/**
 * Place le logo de marque dans un parent (slide ou cartouche).
 * Clone le node template depuis le Figma cible, redimensionne et place.
 * @param {FrameNode} parent  - slide ou cartouche cible
 * @param {object}    opts    - {x, y, w, h, variant: "color"|"white"}
 * @returns le node logo cloné et placé
 */
function placeLogo(parent, o) {
  // o : { x, y, w?, h?, variant?: "color"|"white" }
  // Fallback vers la variante couleur si la blanche n'est pas fournie
  const tpl = (o.variant === "white" && LOGOS.white) ? LOGOS.white : LOGOS.color;
  if (!tpl) {
    throw new Error(
      "Aucun node logo disponible dans le Figma cible — colle ton logo SVG " +
      "sur Page 1 et nomme-le 'Logo couleur' avant de relancer le skill " +
      "(Étape A.1.a)."
    );
  }

  // ⚠️ Préservation OBLIGATOIRE de l'aspect ratio source.
  // L'argument {w, h} passé représente une **boundingbox**, pas la taille
  // exacte du logo. Le logo s'inscrit dans la box en mode fit-contain.
  const srcRatio = tpl.width / tpl.height;

  let w, h, xFinal, yFinal;
  if (o.w && o.h) {
    // Mode fit-contain : inscrit le logo dans la box w×h sans déformer
    const boxRatio = o.w / o.h;
    if (boxRatio > srcRatio) {
      // Box plus large que le logo → contrainte par la hauteur
      h = o.h;
      w = h * srcRatio;
    } else {
      // Box plus haute (ou ratio égal) → contrainte par la largeur
      w = o.w;
      h = w / srcRatio;
    }
    // Centrer horizontalement + verticalement dans la boundingbox
    xFinal = o.x + (o.w - w) / 2;
    yFinal = o.y + (o.h - h) / 2;
  } else if (o.w) {
    // Largeur fixée → hauteur calculée depuis le ratio source
    w = o.w;
    h = w / srcRatio;
    xFinal = o.x;
    yFinal = o.y;
  } else if (o.h) {
    // Hauteur fixée → largeur calculée depuis le ratio source
    h = o.h;
    w = h * srcRatio;
    xFinal = o.x;
    yFinal = o.y;
  } else {
    throw new Error("placeLogo() : il faut fournir au moins w ou h.");
  }

  const clone = tpl.clone();
  clone.name = "logo";
  clone.resize(w, h);
  clone.x = xFinal;
  clone.y = yFinal;
  parent.appendChild(clone);
  return clone;
}
```

**Pourquoi préserver l'aspect ratio est non négociable** : un logo
déformé (étiré horizontalement ou écrasé verticalement) **n'est plus le
logo de la marque** — c'est une violation directe de la charte qu'on est
censé fabriquer. Un client qui voit son logo squashé dans son propre
brand book ne valide pas le livrable. Bug observé sur la première
version du brand book fokusia (mai 2026) : box `w:272 h:80` sur un logo
en ratio 4.35 → hauteur forcée à 80px alors que la hauteur correcte pour
w=272 est 62.5px → écrasement vertical visible. Le fix `placeLogo()`
ci-dessus calcule automatiquement la dimension manquante depuis le ratio
source, et centre le logo dans la boundingbox quand les deux dimensions
sont fournies.

**Conséquence sur la spec des slides** (Étape 8) : les arguments `w` et
`h` passés à `placeLogo()` représentent désormais une **boundingbox**,
PAS la dimension exacte du logo affiché. Le logo s'inscrit DANS cette
box en mode fit-contain :
- `placeLogo(s1, {x:128, y:80, w:280, h:80})` → logo dans une box
  280×80, centré, ratio préservé. Si ratio source = 4.35 et boxRatio =
  3.5, le logo affiché fait `280×64.4` centré verticalement dans la box.
- `placeLogo(s6, {x:520, y:160, w:360, h:120})` → logo dans une box
  360×120, centré dans le cartouche.
- `placeLogo(s1, {x:128, y:80, w:280})` → hauteur auto depuis le ratio.

**Règle stricte** : tous les usages historiques du texte « Logo » 96px
(slide 01 Cover) ou des cartouches placeholder de la slide 06 doivent passer
par `placeLogo()`. Si `LOGOS.color` est `null` (aucun node logo détecté),
**stopper la génération** et demander à l'utilisateur de coller le logo
dans le Figma cible plutôt que de générer un brand book décoratif sans
logo réel.

**Note sur la recoloration** : si l'utilisateur ne fournit que la variante
`Logo couleur` (pas de blanche), le helper place le node couleur tel quel
sur les fonds sombres et le contraste sera dégradé. Une option avancée
consiste à parcourir les paths du clone et remplacer les fills `primary`
par `cream` programmatiquement — mais cette opération est fragile sur des
SVG complexes. La recommandation est de demander à l'utilisateur la
variante blanche dès qu'on détecte qu'elle manque.

### Préambule : helper pour placer les images (`placeImage`) — structure MASQUE

**Obligatoire pour tous les slots images.** Chaque slot est un **groupe
masque** : un rectangle « masque » exactement à la taille du slot
(`isMask = true`) + une couche de contenu au-dessus (image clonée si
fournie, sinon dégradé du preset). Jamais d'upload, jamais de base64.

> **Pourquoi un masque et pas un simple rectangle** (retour Arnaud,
> 2026-06) : avec un rectangle plein, remplacer le placeholder oblige à
> redécouper chaque image à la bonne taille, slot par slot. Trop de
> travail. Avec un masque pré-posé à la taille du slot, Arnaud glisse
> n'importe quelle image (même beaucoup plus grande) dans le groupe :
> elle est **automatiquement rognée au cadre du masque**, sans recrop.
> Il peut ensuite la déplacer ou la zoomer derrière le masque, le cadre
> ne bouge pas. Structure cible (vue dans le Figma de référence,
> node 19:2) :
>
> ```
> MoodBoard · <slot>            (groupe)
>   ├── <slot> · masque         (rect taille slot, isMask, en bas)
>   └── <slot> · image          (image clonée OU dégradé, au-dessus, masquée)
> ```

```js
// IMG_TPLS : peuplé à l'Étape A.1.a.bis (nodes "IMG · <slot>" de Page 1)
// PRESET   : sélectionné à l'Étape 4bis

/**
 * Place un slot image sous forme de GROUPE MASQUE.
 * @param {FrameNode} parent
 * @param {object} o - { slot, x, y, w, h, radius? }
 * @returns le GroupNode (nommé "MoodBoard · <slot>")
 */
function placeImage(parent, o) {
  const rad = (o.radius !== undefined) ? o.radius : PRESET.imageRadius;
  const tpl = IMG_TPLS[o.slot];

  // 1. Le MASQUE : rect exactement à la taille du slot. Sa couleur n'est
  //    pas rendue (un masque ne sert que d'alpha), on le met en blanc.
  const mask = figma.createRectangle();
  mask.resize(o.w, o.h);
  if ("cornerRadius" in mask) mask.cornerRadius = rad;
  mask.fills = [{ type: "SOLID", color: { r: 1, g: 1, b: 1 } }];
  mask.x = o.x; mask.y = o.y;
  mask.name = `${o.slot} · masque`;
  parent.appendChild(mask);          // index bas = sous le contenu

  // 2. Le CONTENU : image clonée (cadrée COVER, centrée, débordement
  //    assumé = le masque rogne) ou dégradé placeholder.
  let content;
  if (tpl) {
    content = tpl.clone();
    const sc = Math.max(o.w / tpl.width, o.h / tpl.height);
    if (content.rescale) content.rescale(sc); else content.resize(o.w, o.h);
    content.x = o.x + (o.w - content.width) / 2;
    content.y = o.y + (o.h - content.height) / 2;
  } else {
    content = figma.createRectangle();
    content.resize(o.w, o.h);
    if ("cornerRadius" in content) content.cornerRadius = rad;
    content.fills = PRESET.placeholderFill(o.slot);  // dégradé du preset
    content.x = o.x; content.y = o.y;
    PLACEHOLDER_SLOTS.push(o.slot);  // collecté pour images-brief.md
  }
  content.name = `${o.slot} · image`;
  parent.appendChild(content);       // index haut = au-dessus du masque

  // 3. Activer le masque PUIS grouper (ordre z conservé : masque en bas)
  mask.isMask = true;
  const group = figma.group([mask, content], parent);
  group.name = `MoodBoard · ${o.slot}`;

  // 4. Cadre du preset (Pop : 4px encre) — posé HORS du groupe masque
  //    pour ne jamais être rogné, et survivre au remplacement de l'image.
  if (PRESET.frame) {
    const fr = figma.createRectangle();
    fr.resize(o.w, o.h);
    if ("cornerRadius" in fr) fr.cornerRadius = rad;
    fr.fills = [];
    fr.strokes = solid(COLORS[PRESET.frame.color]);
    fr.strokeWeight = PRESET.frame.w;
    fr.strokeAlign = "INSIDE";
    fr.x = o.x; fr.y = o.y;
    fr.name = `${o.slot} · cadre`;
    parent.appendChild(fr);
  }
  return group;
}
```

**Remplacement par Arnaud (sans recrop)** : double-clic dans le groupe
`MoodBoard · <slot>` → sélectionner la couche `<slot> · image` →
« Place image » (ou glisser un fichier dessus). La nouvelle image hérite
du cadre du masque. Pour réajuster : la déplacer / la zoomer librement,
le masque garde le cadrage. Le texte posé « par-dessus » un slot (numéros
univers, labels) doit rester **hors du groupe masque** (sinon il est
rogné) : le poser après l'appel `placeImage`, en enfant direct de la slide.

**Recettes `placeholderFill()` par preset** (fills GRADIENT_LINEAR non
bindés — exception documentée, comme les shades de la slide 09 ; les
dégradés ne peuvent pas être liés à des variables) :

```js
// Helper générique : dégradé linéaire vertical entre 2 couleurs
const grad = (h1, h2, opacity2 = 1) => [{
  type: "GRADIENT_LINEAR",
  gradientTransform: [[0, 1, 0], [-1, 0, 1]],   // vertical haut → bas
  gradientStops: [
    { position: 0, color: { ...hex(h1), a: 1 } },
    { position: 1, color: { ...hex(h2), a: opacity2 } },
  ],
}];

const PLACEHOLDER_RECIPES = {
  "luxe":      (slot) => grad(COLORS.ardoise ?? "#05080F", COLORS.primaryDark),
  "minimal":   (slot) => [{ type: "SOLID", color: hex("#F2F4F7") }],
  "creme":     (slot) => grad(COLORS.cream, COLORS.accent, 0.25),
  "pop":       (slot) => [
    // 2 aplats tranchés : un rectangle accent + diagonale primary posée par-dessus
    { type: "SOLID", color: hex(COLORS.accent) },
  ],
  "corporate": (slot) => grad(COLORS.primary, COLORS.primaryDark),
};
```

> **Pourquoi un dégradé et pas un gris hachuré** : le brand book doit être
> **présentable au client même sans photos**. Un placeholder gris dashed
> dit « document inachevé » ; un dégradé dans les couleurs de la marque
> dit « zone d'ambiance », et le drag-drop d'une vraie photo par-dessus
> prend 5 secondes. Le nom de node `MoodBoard · <slot>` suffit à
> retrouver chaque emplacement (panneau Layers ou recherche Figma).

### Registre des slots images (16 slots nommés)

C'est la **table de référence** : `images-brief.md` (Étape 9ter) est
généré depuis ce registre, et l'utilisateur nomme ses nodes Figma
`IMG · <slot>` avec ces identifiants exacts.

| Slot | Slide | Position (x,y) | Taille | Rôle éditorial |
|---|---|---|---|---|
| `cover` | 01 | 1513, 0 | 407×1280 | Portrait ou visuel signature, bande verticale pleine hauteur bord droit |
| `sommaire-1`…`sommaire-4` | 02 | collage central dès 569, 167 | ~156×440-465 chacune | 4 bandes verticales masonry (hauteurs décalées) |
| `sommaire-5` | 02 | 1692, 47 | 155×151 | Petite vignette haut-droite |
| `identite` | 03 | 1408, 255 | 513×669 | Visuel d'ambiance qui incarne le ton |
| `univers-1`…`univers-6` | 04 | grille 3×2 dès 720, 440 | 346×232 chacune | Moodboard labellisé (thèmes selon Q12) |
| `chapitre-logo` | 05 | 1224, 188 | 696×677 | Visuel chapitre Logo |
| `logo-hero` | 06 | 560, 0 | 1260×844 | Grand visuel de mise en scène du logo |
| `chapitre-couleur` | 07 | 1273, 120 | 347×1020 | Bande verticale chapitre Couleur |
| `palette-col-<nom>` | 08 | dans chaque colonne, y=345 | 186×546 | Insert duotone par couleur (optionnel) |
| `nuances` | 09 | 100, 382 | 320×272 | Visuel matière sous le paragraphe |
| `chapitre-typo` | 10 | 741, -102 | 485×642 | Visuel chapitre Typographie (bleed haut si preset l'autorise) |
| `police` | 11 | -534, 404 | 968×705 | Visuel bas-gauche (bleed gauche si preset l'autorise) |
| `hierarchie` | 12 | 1239, 0 | 681×1280 | Panneau pleine hauteur bord droit |
| `chapitre-systeme` | 13 | 1353, 171 | 615×695 | Visuel chapitre Système |
| `elements` | 14 | 129, 728 | 1668×255 | Bandeau panoramique sous les 6 cards |

**Presets sans bleed** (Minimal tech, Corporate clair) : ramener les
slots débordants dans le cadre — `chapitre-typo` → (1140, 120, 600×700),
`police` → (80, 560, 480×560), `hierarchie` → (1320, 80, 520×1068).

### Layout de la grille de slides

Disposer les 15 slides en grille 2 colonnes × 8 rangées (dernière rangée
incomplète — slide 15 seule sur colonne A), avec **gap 240px** entre slides
(horizontalement ET verticalement) — l'espace généreux permet de naviguer
sans se sentir oppressé par la densité, et facilite les annotations / les
captures par section :
```
col A (x=0)     col B (x=2160)
rangée 1 :  Slide 01      Slide 02
rangée 2 :  Slide 03      Slide 04
rangée 3 :  Slide 05      Slide 06
...
rangée 7 :  Slide 13      Slide 14
rangée 8 :  Slide 15      ·
```
Soit :
- X de chaque colonne : `col A = 0`, `col B = 1920 + 240 = 2160`
- Y de chaque rangée : `r * (1280 + 240) = r * 1520`

> **Pourquoi 240px et pas 60px** : sur les anciennes versions le gap de 60px
> faisait coller les slides visuellement (l'œil les lisait comme un seul bloc).
> 240px = 12.5% de la largeur d'une slide, soit la respiration minimale pour
> que chaque slide soit lue comme un objet autonome.

---

## Étape 6 — Patron commun pour chaque slide

```js
// Noms français des slides — utilisés comme `name` du frame Figma
// (visible dans la sidebar Layers, donc traité comme du contenu utilisateur).
const SLIDE_NAMES_FR = {
  1:  "Couverture",
  2:  "Sommaire",
  3:  "Identité verbale",
  4:  "Univers & ambiance",
  5:  "Chapitre — Logo",
  6:  "Logo",
  7:  "Chapitre — Couleur",
  8:  "Palette de couleurs",
  9:  "Nuances",
  10: "Chapitre — Typographie",
  11: "Police",
  12: "Hiérarchie typographique",
  13: "Chapitre — Système",
  14: "Éléments clés",
  15: "Quatrième de couverture",
};

function makeSlide({ index, total, bg, isChapterOpener = false }) {
  const slide = figma.createFrame();
  const num = String(index).padStart(2,"0");
  slide.name = `${num} · ${SLIDE_NAMES_FR[index]}`;  // ex "01 · Couverture"
  slide.resize(1920, 1280);
  slide.fills = solidVar(bg);     // bg = nom de variable (ex "cream", "primary")
  slide.clipsContent = true;
  return slide;
}

function addFooter(slide, { brandName, page, total, color = COLORS.dark }) {
  // brandName : nom de la marque EN CAPITALES (ex "MAX PICCININI").
  // Le handle @marque n'est utilisé que sur demande explicite.
  const h = figma.createText();
  h.fontName = {family:"Inter Display", style:"Regular"};
  h.fontSize = 16;
  h.letterSpacing = { value: 2, unit: "PIXELS" };
  h.characters = brandName.toUpperCase();
  h.resize(400, 24);
  h.textAutoResize = "HEIGHT";
  h.fills = solid(color);
  h.x = 80; h.y = 1228;
  slide.appendChild(h);

  const p = figma.createText();
  p.fontName = {family:"Inter Display", style:"Regular"};
  p.fontSize = 16;
  p.characters = `page ${String(page).padStart(2,"0")}/${String(total).padStart(2,"0")}`;
  p.resize(200, 24);
  p.textAutoResize = "HEIGHT";
  p.textAlignHorizontal = "RIGHT";
  p.fills = solid(color);
  p.x = 1640; p.y = 1228;
  slide.appendChild(p);
}
```

---

## Étape 7 — Découpe en appels MCP (anti-timeout)

Ne pas tenter de générer les 15 slides en un seul appel `use_figma`.
Découper en **7 appels** :

| Appel | Slides | Notes |
|---|---|---|
| 1 | Preload fonts + Cover (01) + Sommaire (02) | Initialisation page |
| 2 | **Verbal Identity (03) + Universe & Mood (04)** | Slides interview — riches en texte |
| 3 | Chapter Logo (05) + Content Logo (06) | |
| 4 | Chapter Colour (07) + Palette (08) | Slide la plus dense — 6 colonnes |
| 5 | Shades (09) + Chapter Typo (10) | |
| 6 | Typeface (11) + Type Hierarchy (12) | |
| 7 | Chapter System (13) + Key Elements (14) + Back Cover (15) + `figma.fileThumbnailNodeId` = id de la slide 01 + `viewport.scrollAndZoomIntoView` | Finalisation |

Les slots images ne changent pas la découpe : un placeholder dégradé est
un simple rectangle, et un clone de node `IMG` est une opération locale
légère. Redéclarer les helpers (`placeImage`, recettes, `PRESET`) au
début de **chaque** appel — les scopes ne persistent pas entre appels
`use_figma` (re-scanner `IMG_TPLS` via A.1.a.bis à chaque fois).

À chaque appel, **récupérer la page courante** au lieu de la recréer :
```js
const page = figma.currentPage.name === "✦ Brand Book"
  ? figma.currentPage
  : figma.root.children.find(p => p.name === "✦ Brand Book");
figma.currentPage = page;
```

---

## Étape 8 — Spécifications par slide

### 01 · Couverture
- Fond : `PRESET.bg.cover` (luxe : `primaryDark` ; crème : `cream` ; etc.)
- **Slot image `cover`** : `placeImage(s1, { slot: "cover", x: 1513, y: 0,
  w: 407, h: 1280, radius: 0 })` — bande verticale pleine hauteur collée
  au bord droit (portrait du fondateur, matière, visuel signature). La
  poser **en premier** pour qu'elle reste sous les textes.
- **Logo source réel en haut-gauche** : `placeLogo(slide01, { x: 100,
  y: 80, w: 240, h: 72, variant: selon fond })`. **Ne jamais** remplacer
  par un texte "Logo" — placeholder inacceptable.
- Eyebrow **`IDENTITÉ DE MARQUE · [année courante]`** 14px Semi Bold,
  letter-spacing 4px, x=100 y=200, couleur accent ou contrastée
- Titre marque XXL en **bas-gauche** : 1-2 mots sur 1-2 lignes,
  **140-200px** (selon Q9) Display du preset, x=150, le bloc se termine
  vers y=890. Largeur max ~1300px (laisser respirer le slot cover).
- Accent line 120×4 (`accent`) x=160 y=930
- **Formule signature** entre guillemets français « … » en 32px Italic ou
  Regular, x=160 y=984, w=1300
- Footer : nom de marque + page 01/15

### 02 · Sommaire
- Fond : `PRESET.bg.content`
- Eyebrow **"SOMMAIRE"** 18px Semi Bold letter-spacing 4px (x=100, y=120)
- **5 lignes de chapitres**, une par chapitre, pitch vertical 168px à
  partir de y=300 :
  - Numéro **"01"…"05"** 64px Display, x=100, couleur accent
  - Intitulé (**"Identité"**, **"Logo"**, **"Couleur"**,
    **"Typographie"**, **"Système"**) 64px Display, x=260
  - Description à droite : x=1300, w=520, 18px Regular — rédigée en
    français (« Ton, formule, univers », « Variantes et zone de
    protection », « Palette et nuances », « [Display] et [Body] »,
    « Règles de composition »)
  - Hairline 520×1 sous la description (x=1300, y=ligne+70), opacité 30%
- **Collage central — 5 slots images masonry** entre les intitulés et
  les descriptions (la zone x=560…1200 est libre) :
  - `placeImage(s2, { slot: "sommaire-1", x: 569, y: 331, w: 156, h: 438 })`
  - `placeImage(s2, { slot: "sommaire-2", x: 726, y: 167, w: 155, h: 457 })`
  - `placeImage(s2, { slot: "sommaire-3", x: 882, y: 315, w: 155, h: 444 })`
  - `placeImage(s2, { slot: "sommaire-4", x: 1036, y: 167, w: 158, h: 465 })`
  - `placeImage(s2, { slot: "sommaire-5", x: 1692, y: 47, w: 155, h: 151 })`
  Les hauteurs décalées créent le rythme masonry : ne pas les aligner.
  Preset Minimal tech : remplacer le collage par 1 seule image cadrée
  (x=620, y=200, 560×400) — la densité d'images est un code éditorial,
  pas minimal.

### 03 · Identité verbale *(depuis l'interview Phase A.2)*
- Fond : `neutralLight`
- Eyebrow **`01 · IDENTITÉ`** 14px Semi Bold accent (x=80, y=80)
- Titre **"Identité\nverbale"** 64px Bold primary (x=80, y=108, w=600, h=160)
- Bandeau attributs **`TON · [valeur] — POSTURE · [valeur]`** 12px Semi Bold accent uppercase, letter-spacing 3px (y=300)
- **Formule signature** centrée (80-100px Bold primary) — la phrase issue de `brand.md`
- Ligne accent 4px de large sous la formule (1 seul accent decorative)
- Sous la formule, légende discrète Regular 16px primary
- En bas, **2 colonnes** 480px largeur, espace XL entre :
  - **Mots à utiliser** : eyebrow accent (libellé **« MOTS À UTILISER »**) + **liste 6-8 mots max** en 18px Medium primary (3 par ligne)
  - **Mots à éviter** : eyebrow accent (libellé **« MOTS À ÉVITER »**) + **liste 5-6 mots max** en 18px Regular ardoise barrés (`textDecoration: STRIKETHROUGH`)
- **3 CTAs exemples** alignés horizontalement en 16-18px Semi Bold primary avec icône `→` (jamais accent — l'accent est réservé aux vrais CTAs en production)
- **Slot image `identite`** : `placeImage(s3, { slot: "identite",
  x: 1408, y: 255, w: 513, h: 669 })` — visuel d'ambiance qui incarne le
  ton, bord droit. Limiter la formule signature et le bandeau attributs
  à w=1280 pour ne pas passer sous l'image.

> **Sobriété** : pas plus de 8 mots par colonne, pas de chips colorés, pas
> d'éléments décoratifs additionnels. La densité dilue le premium.

### 04 · Univers & ambiance *(depuis l'interview Phase A.2)*

**Layout 2 colonnes** — col gauche 540px (attributs + citation),
col droite 1120px (grille 6 photos moodboard).

- Fond : `primary` ou `primaryDark` (atmosphère immersive — la seule
  slide où le fond sombre est légitime malgré la règle « crème ≥70% »)
- Eyebrow **`01 · IDENTITÉ`** 14px Semi Bold accent (x=80, y=80)
- Titre **"Univers\n& ambiance"** 88px Bold cream sur 2 lignes (x=80, y=120, w=560)

**Colonne gauche — 4 attributs compacts** (à partir de y=440) :
  - Chaque attribut = eyebrow 11px Semi Bold accent uppercase
    (`AMBIANCE`, `RÉFÉRENCE`, `IMAGERIE`, `PERSONNALITÉ`) + valeur 24px Bold cream
  - Espacement 90px entre attributs (compact)
  - **Pas de chips colorés** — texte simple « Sobre · Premium · Technique » selon Q14.
    Si Sobre/Premium/Confidentiel, chips à proscrire (règle « accent chirurgical »).

**Colonne gauche — citation en bas** (y=840, w=560) :
  - Phrase manifeste 18px Medium accent en 3-4 lignes, lh=160
  - Ex pour fokusia : *« Aucune représentation littérale du métier
    d'investigation — on suggère par les codes du pouvoir installé. »*

**Colonne droite — Moodboard 6 slots images labellisés 3×2** :

> **Pourquoi 6 images plutôt qu'un paragraphe** : le moodboard textuel
> décrit en mots ce que les photos devraient montrer. Avec 6 slots
> `placeImage()` labellisés, la slide est un **vrai moodboard photo**
> dès que les images existent, et un moodboard d'ambiance (dégradés du
> preset) en attendant. Drag-drop d'une photo sur un slot = 5 secondes.

- **Grille 3 colonnes × 2 lignes**, slots 346×232, gap 20px, zone à
  partir de (720, 440) :
  - `placeImage(s4, { slot: "univers-1", x: 720,  y: 440, w: 346, h: 232 })`
  - `placeImage(s4, { slot: "univers-2", x: 1086, y: 440, w: 346, h: 232 })`
  - `placeImage(s4, { slot: "univers-3", x: 1452, y: 440, w: 346, h: 232 })`
  - `placeImage(s4, { slot: "univers-4", x: 720,  y: 692, w: 346, h: 232 })`
  - `placeImage(s4, { slot: "univers-5", x: 1086, y: 692, w: 346, h: 232 })`
  - `placeImage(s4, { slot: "univers-6", x: 1452, y: 692, w: 346, h: 232 })`
- **Par-dessus chaque slot** (textes posés après l'image, jamais en
  enfants du rectangle) :
  - Numéro `01`…`06` 12px Semi Bold, x=slot.x+20, y=slot.y+18,
    couleur cream (ou contrastée selon preset)
  - Label thématique 14px Medium, x=slot.x+20, y=slot.y+188 — un mot
    concret par slot, dérivé de Q12 (table ci-dessous)
  - Si l'image clonée est trop claire pour les labels, poser un voile
    `noir` opacité 25% entre l'image et les textes (rectangle même taille)
- **Labels thématiques selon brand.md Q12** (Référence d'univers) :
  - Architecture & passages → Escaliers · Cathédrale · Sculpteur · Bibliothèque · Musée · Lumière
  - Nature & matières brutes → Paysage · Texture · Matière · Geste · Détail organique · Ambiance
  - Tech minimale & grilles → Grille · Data · Interface · Lumière froide · Geste tech · Composition
  - Éditorial luxe & mode → Détail produit · Texture luxe · Portrait · Composition · Geste · Atmosphère

**Workflow utilisateur — drag-drop dans Figma** (rappelé dans
`images-brief.md`, pas sur la slide) :
1. Sélectionner une photo dans le Finder / Explorer
2. La glisser-déposer sur le node `MoodBoard · univers-X`
3. Figma applique l'image comme fill (`scaleMode: FILL`), dimensions
   conservées ; cadrage ajustable via double-clic → Crop

### 05/07/10/13 · Chapter openers
- Fond pleine couleur : **par défaut selon le preset** (luxe : tous
  `primaryDark` ; pop : tous `accent` ; sinon alternance pilotée par Q10,
  cf. Étape 4ter)
- Eyebrow **`CHAPITRE 02`** / **`CHAPITRE 03`** / **`CHAPITRE 04`** /
  **`CHAPITRE 05`** en haut-gauche (x=80, y=120), 19px Semi Bold,
  letter-spacing 4px, couleur accent (01 est réservé à Identité,
  slides 03-04)
- **Slot image à droite** (poser avant les textes) :
  - 05 : `placeImage(s5, { slot: "chapitre-logo", x: 1224, y: 188, w: 696, h: 677 })`
  - 07 : `placeImage(s7, { slot: "chapitre-couleur", x: 1273, y: 120, w: 347, h: 1020 })`
  - 10 : `placeImage(s10, { slot: "chapitre-typo", x: 741, y: -102, w: 485, h: 642 })`
    (bleed haut — si `PRESET.allowBleed` est faux : x=1140, y=120, 600×700)
  - 13 : `placeImage(s13, { slot: "chapitre-systeme", x: 1353, y: 171, w: 615, h: 695 })`
- Accent line 120×4-5 au-dessus du mot (ex x=164, y=400-640 selon hauteur du mot)
- Titre **mot français entier sur une seule ligne — jamais coupé**,
  posé en **bas-gauche** (le bloc se termine vers y=1040) :
  **"Logo"**, **"Couleur"**, **"Typographie"**, **"Système"** — Display
  du preset, couleur contrastée. Taille adaptée pour tenir à gauche du
  slot image sans débordement :
  - "Logo" (4 lettres) → 340-400px
  - "Couleur", "Système" (7 lettres) → 240-300px
  - "Typographie" (11 lettres) → 170-210px
  Si débordement au test, **réduire la taille de 20px** et réessayer
  plutôt que de casser le mot.
- Sous-titre du chapitre en **bas-droite** (x=1140, y=1086, w=680, align
  right, 22px Medium) : 3 items séparés par ` · ` —
  05 : « Variantes · Zone de protection · Taille minimale » ;
  07 : « Palette · Nuances · Usage de l'accent » ;
  10 : « [Display] · [Body] · Hiérarchie » ;
  13 : « Composition · Règles · [mot signature du preset] »

### 06 · Content — Logo
- Fond : `PRESET.bg.content`
- **Slot image `logo-hero`** (poser en premier) : `placeImage(s6,
  { slot: "logo-hero", x: 560, y: 0, w: 1260, h: 844, radius: 0 })` —
  grand visuel de mise en scène (logo en situation, matière, signalétique).
  Si le slot est en placeholder, y poser le logo par-dessus, centré :
  `placeLogo(s6, { x: 960, y: 280, w: 460, h: 280, variant: "white" })`
- Colonne gauche (x=100, w=380) : titre "Logo" 40px + paragraphe
  descriptif depuis `brand.md` (type, symbolique, règle vecteur :
  « Toujours inséré comme vrai vecteur, jamais recréé. Variante couleur
  sur fonds clairs, variante blanche sur fonds sombres. »)
- En bas : **3 cartouches 400×214 cornerRadius selon preset**, à
  y=846, x=560 / 990 / 1420, avec variantes du **logo source réel**
  (jamais un placeholder texte) :
  - Cartouche 1 : fond `white` + logo en **couleur originale** (`LOGOS.color`)
  - Cartouche 2 : fond `primaryDark` + logo en **blanc/cream** (`LOGOS.white`)
  - Cartouche 3 : fond `accent` + logo en **blanc/cream** (`LOGOS.white`)

  **Implémentation** : pour chaque cartouche, créer le rectangle de fond
  via `figma.createRectangle()` (couleur bindée à la variable), puis
  `placeLogo(cartouche, { x: 92, y: 74, w: 216, h: 66, variant: "color"|"white" })`.
  Si `LOGOS.white` est `null`, fallback automatique sur `LOGOS.color`
  avec mention au récap final. **Centrer le logo** dans chaque cartouche.

- Sous les cartouches (y=1148) : 2 blocs de spec côte à côte —
  eyebrow **"TAILLE MINIMALE"** + valeur « 96 px de large » (x=620) ;
  eyebrow **"ZONE DE PROTECTION"** + valeur « 1× la hauteur du logo »
  (x=1050)

### 08 · Palette de couleurs
- Fond : `PRESET.bg.content`
- Colonne gauche (x=100, w=430) : titre **"Palette de couleurs"** 40px +
  paragraphe règle d'usage rédigé en français (rôle de chaque couleur,
  ratio d'accent Q10)
- Sous le paragraphe : **mini-swatches de rappel** — 1 carré 86×83 par
  couleur, empilés verticalement à x=160 à partir de y=479 (gap 14px),
  cornerRadius selon preset. Rappel discret de la palette dans la marge.
- Zone droite (y=160) : 4 à 6 **colonnes 186×960px cornerRadius selon
  preset** côte à côte avec gap 16px à partir de x=620 (6 couleurs :
  x=620, 822, 1024, 1226, 1428, 1630). Chaque colonne :
  - Fond = la couleur du token (ex `primary`, `noir`, `accent`, `acier`,
    `lumière`, `bone`)
  - Nom de la couleur en haut (y=240, 24px Medium) — **en français**
    (ex « Primaire », « Noir », « Accent »). La variable Figma interne
    reste technique, ce qui s'affiche est le libellé français.
  - **Slot image intégré** (optionnel mais recommandé) :
    `placeImage(col, { slot: "palette-col-<nom>", x: 0, y: 345, w: 186,
    h: 546, radius: 0 })` — un visuel duotone dans la teinte de la
    colonne. En placeholder : dégradé de la couleur de la colonne vers
    sa version sombre (vivant même sans photo).
  - HEX en bas (y=1018, 18px) + RVB en dessous (y=1044, 13px,
    format « 10 · 26 · 48 »)
  - Texte blanc sur fond sombre, sombre sur fond clair
- 4 couleurs → colonnes 280px à partir de x=620 ; 5 couleurs → 224px.
  Les slots images suivent la largeur de leur colonne.

### 09 · Nuances
- Fond : `PRESET.bg.content` (variable bindée — slide content classique)
- Colonne gauche : titre **"Nuances"** + paragraphe sur l'usage des tints
- **Slot image `nuances`** sous le paragraphe : `placeImage(s9,
  { slot: "nuances", x: 100, y: 382, w: 320, h: 272 })` — détail de
  matière ou texture qui montre la couleur en situation
- Zone droite : **2 lignes de 5 carrés** (100 / 80 / 60 / 40 / 20 %) pour
  `primary` (ligne 1, à y=310) et `accent` (ligne 2, à y=690) — chaque carré
  **220×300 cornerRadius 8** (cornerRadius 8 = signature Sobre + Premium ;
  passer à 16 ou 24 pour Audacieux / Chaleureux).
- Label "100" / "80" / ... en gras 36px, centré horizontal, basculement de
  couleur selon opacité : cream sur les 2 carrés sombres (≥0.6), primary
  sombre sur les 3 clairs (<0.6). Pas d'opacité sur le label principal.
- Mini-label "opacity 0.x" en 12px Medium, sous le label principal, opacité
  0.7 sur le node (autorisé car élément seul sans enfants).

**Application stricte de l'opacité — technique pré-mélange** (mise à jour
2026-05) : les anciennes versions tentaient d'appliquer `opacity` sur un
fill `SOLID` lié à une variable (`{ ...solidVar(varName)[0], opacity: 0.6 }`),
mais Figma ignore ce paramètre quand la couleur vient d'une variable
**bindée** — résultat : les 5 carrés s'affichent identiques à 100%.

**Solution** : pré-calculer la couleur résultante en mélangeant la couleur
source avec le fond `cream` à l'opacité voulue, puis appliquer un fill
SOLID **non bindé**. C'est intentionnel : les shades sont des **dérivés
documentés**, pas des références à modifier dans le panneau Variables —
elles servent à montrer le RENDU visuel de l'opacité telle que les
designers l'appliqueront ensuite dans leurs outils (Figma, HTML, etc.).

```js
const SHADE_STEPS = [
  { label: "100", op: 1.0 },
  { label: "80",  op: 0.8 },
  { label: "60",  op: 0.6 },
  { label: "40",  op: 0.4 },
  { label: "20",  op: 0.2 },
];

// Mix d'une couleur source sur un fond cream à l'opacité op (0-1)
// Renvoie {r, g, b} normalisé 0-1 pour fills
function mixOnCream(srcHex, creamHex, op) {
  const s = hex(srcHex), c = hex(creamHex);
  return {
    r: op * s.r + (1 - op) * c.r,
    g: op * s.g + (1 - op) * c.g,
    b: op * s.b + (1 - op) * c.b,
  };
}

const solidObj = (obj) => [{ type: "SOLID", color: obj }];

function makeShadeRow(parent, { yRow, srcHex }) {
  SHADE_STEPS.forEach((step, i) => {
    const rect = figma.createRectangle();
    rect.resize(220, 300);
    rect.cornerRadius = 8;  // Sobre + Premium → 8 max
    rect.x = 520 + i * (220 + 16);  // gap 16px
    rect.y = yRow;

    // ✅ Pré-mélange : fill SOLID sans boundVariables
    const mixed = mixOnCream(srcHex, COLORS.cream, step.op);
    rect.fills = solidObj(mixed);

    parent.appendChild(rect);

    // Label "100" / "80" / ... — couleur adaptée selon opacité
    // Sur les 2 plus opaques (1.0 / 0.8) → cream pour contraste
    // Sur les 3 plus claires (0.6 / 0.4 / 0.2) → primary sombre
    const labelColor = step.op >= 0.6 ? COLORS.cream : COLORS.primary;

    const label = figma.createText();
    label.fontName = { family: "Inter", style: "Bold" };
    label.fontSize = 36;
    label.characters = step.label;
    label.fills = solid(labelColor);
    label.textAlignHorizontal = "CENTER";
    label.resize(220, 44);
    label.x = rect.x;
    label.y = rect.y + 128;  // centré vertical
    parent.appendChild(label);

    // Mini label "opacity 0.x" en bas (légende visuelle)
    const t2 = figma.createText();
    t2.fontName = { family: "Inter", style: "Medium" };
    t2.fontSize = 12;
    t2.characters = "opacity " + step.op.toFixed(1);
    t2.fills = solid(labelColor);
    t2.textAlignHorizontal = "CENTER";
    t2.resize(220, 18);
    t2.x = rect.x;
    t2.y = rect.y + 260;
    t2.opacity = 0.7;  // ici opacité sur le NODE = OK (label seul)
    parent.appendChild(t2);
  });
}

// srcHex = les valeurs primary / accent réelles de brand.md
makeShadeRow(slide, { yRow: 310, srcHex: COLORS.primary });
makeShadeRow(slide, { yRow: 690, srcHex: COLORS.accent });
```

**Points critiques** :
- ❌ **Ne pas faire** : `rect.fills = [{ ...solidVar(varName)[0], opacity: 0.6 }]`
  — Figma ignore `opacity` sur un fill SOLID variable-bindé. Bug observé en
  prod sur le projet brandin&ia (mai 2026).
- ❌ **Ne pas faire non plus** : `rect.opacity = 0.6` — translucidifierait
  aussi les labels enfants (perte de lisibilité sur les nuances claires).
- ✅ **Bonne approche** : pré-calculer le rendu via `mixOnCream()` et
  appliquer un fill SOLID **sans `boundVariables`**. Visuel garanti
  identique à ce qu'un designer obtiendrait avec opacité réelle.
- Le label texte doit basculer du cream (sur les 2 cellules sombres) au
  primary (sur les 3 claires) pour respecter le contraste AA. Pas
  d'opacité sur le label principal — seulement sur le mini-label
  « opacity 0.x » (légende secondaire, opacité 0.7 sur le node OK car
  il est seul).
- Sur l'opacité sur node : autorisée **uniquement** sur un node atomique
  (texte simple, ligne, micro-décoration) — JAMAIS sur un conteneur qui
  porte d'autres éléments.

> **Pourquoi pré-mélanger plutôt qu'appliquer opacité au fill** :
> la slide Nuances **documente le rendu visuel** que le designer obtiendra
> en appliquant l'opacité dans son outil (CSS, Figma propriété, etc.).
> Le designer ne consomme pas la slide comme un token à copier — il s'en
> sert comme référence visuelle. Donc on a juste besoin que les 5 carrés
> aient l'aspect correct. Pré-mélange = garantie de rendu. Variable +
> opacity sur fill = bug Figma sur fills bindés.

### 11 · Police
- Fond : `PRESET.bg.immersive` (`primary`/`primaryDark` — atmosphère)
- Colonne gauche (x=100, w=380) : titre **"Police"** 40px + paragraphe
  rédigé en français qui présente **les deux familles** (ex « Deux
  familles. [Display] porte la voix, serif éditorial des titres.
  [Body] porte l'information, sans suisse du corps. »)
- **Slot image `police`** : `placeImage(s11, { slot: "police",
  x: -534, y: 404, w: 968, h: 705 })` — bleed hors cadre gauche (rogné
  par `clipsContent`), occupe le bas-gauche sous le paragraphe. Si
  `PRESET.allowBleed` est faux : x=80, y=560, 480×560.
- Zone droite : **2 "Aa" côte à côte** — Display 350px à (620, 160),
  Body 320px à (1240, 226), couleur contrastée
- Sous chaque "Aa" (y=560) : nom de la famille 24px Medium + rôle 16px
  Regular (« Titres éditoriaux » / « Corps, rigueur suisse »)
- En bas : alphabet complet sur 2 lignes (x=620, w=1200) —
  majuscules A-Z en Display 36px (y=700), minuscules + chiffres en Body
  28px (y=780)

### 12 · Hiérarchie typographique
- Fond : `PRESET.bg.content`
- **Slot image `hierarchie`** (poser en premier) : `placeImage(s12,
  { slot: "hierarchie", x: 1239, y: 0, w: 681, h: 1280, radius: 0 })` —
  panneau pleine hauteur collé au bord droit. Si `PRESET.allowBleed`
  est faux : x=1320, y=80, 520×1068, radius preset.
- Colonne gauche (x=100, w=400) : titre **"Hiérarchie typographique"** +
  paragraphe court rédigé en français (le contraste voix / information)
- Zone centrale (x=620, w=560) : 5 niveaux empilés — chaque niveau :
  - Eyebrow de spec 15px Semi Bold, letter-spacing 2px, en français :
    **"TITRE 1 · [DISPLAY] SEMIBOLD · 64 PX"**, **"TITRE 2 · …"**,
    **"TITRE 3 · [BODY] SEMI BOLD · 26 PX"**, **"CORPS · [BODY] REGULAR
    · 18 PX"**, **"LÉGENDE · [BODY] MEDIUM · 13 PX"**
  - Phrase d'exemple à la taille réelle juste en dessous. **Par défaut** :
    courtes déclinaisons de la formule signature ou du positionnement
    (1 phrase différente par niveau, jamais du lorem). Si le brand.md
    contient une formule signature, l'utiliser sur le niveau Légende.
  - Positions indicatives : y=180, 340, 500, 620, 740 (resserrer si
    l'échelle Q9 est Dramatique)

### 14 · Éléments clés
- Fond : `PRESET.bg.content`
- Colonne gauche (x=100, w=400) : titre **"Éléments clés"** + paragraphe
  rédigé en français (« Six règles de composition qui rendent l'identité
  reproductible, du brand book à la page. »)
- Zone droite : **6 cards 384×224** en grille 3×2 à partir de (597, 236),
  gap 24px horizontal / 19px vertical, cornerRadius et shadow selon
  preset (luxe : fond légèrement éclairci du bg, sans shadow ; chaleureux :
  shadow douce). Chaque card contient une accent line 40×5 (y=+32), le
  nom de la règle 28px Display (y=+56), la description 16px Regular
  (y=+108, w=328).
- **Slot image `elements`** : `placeImage(s14, { slot: "elements",
  x: 129, y: 728, w: 1668, h: 255 })` — bandeau panoramique sous les
  cards (matière, détail, composition large)
- Chaque card : nom de la règle en 20-28px **en français** (ex « Espacement »,
  « Mise en page », « Contraste », « Accentuation », « Typographie »,
  « Atmosphère »…) issu de la liste **Composition rules** de `brand.md`.
  Si le brand.md contient les noms en anglais, les traduire automatiquement
  vers le français standardisé du DS (mapping ci-dessous) :

| Anglais (brand.md) | Français (slide 14) |
|---|---|
| Spacing | Espacement |
| Shaping | Mise en page |
| Contrast | Contraste |
| Accentuation | Accentuation |
| Typeface | Typographie |
| Padding | Marges |
| Atmosphere | Atmosphère |

### 15 · Back cover
- Fond : `PRESET.bg.back` (sombre dans tous les presets — clôture)
- **Logo source réel** centré en haut (variante blanche/cream sur fond
  sombre) via `placeLogo(slide15, { x: 810, y: 200, w: 300, h: 180,
  variant: "white" })`. Pas de placeholder texte "Logo".
- Accent line 100×4 centrée (x=910, y=470)
- Centré : **"Merci"** 160px Display du preset (y=520) — et non plus
  "Thank you"
- Sous le titre : **formule signature** entre guillemets « … » 32px,
  centrée (y=760)
- En bas centré (y=1120, 14px, letter-spacing 2px) : **"[NOM MARQUE] ·
  IDENTITÉ DE MARQUE [année] · Généré par le skill brand-book"**

---

## Étape 9 — Guardrails techniques (lire avant le code)

**① Fonts — preload massif et unique**
Toutes les fonts (Inter + Inter Display × Regular/Medium/Bold) chargées
dans le **premier** appel `use_figma`. Les appels suivants n'ont qu'à
créer les nodes — les fonts sont déjà en cache.

**② Text nodes — règle stricte**
```js
const t = figma.createText();
t.fontName = {family:"Inter Display", style:"Bold"};
t.fontSize = 200;
t.characters = "Big\ntype";       // \n pour casser le titre cover
t.resize(1200, 416);
t.textAutoResize = "HEIGHT";       // ou "WIDTH_AND_HEIGHT" si HUG
t.fills = solid(COLORS.accent);
```

**③ Footer obligatoire sur les 15 slides**
Inclure `addFooter(slide, {brandName, page: i, total: 15})` à la fin de
chaque création de slide. Aucune exception (la back cover aussi).

**④ Cornerradius — toujours 24 sur les blocs de couleur**
Les blocs de palette (slide 08), shades (09), key elements (14) ont
tous `cornerRadius: 24` ou `16` — c'est la signature visuelle du moule
éditorial choisi.

**⑤ Hardcoder zéro hex inline**
Toujours passer par `COLORS.xxx`. Si on a besoin d'une variante d'une
couleur, la définir comme constante en haut du script, pas inline.

**⑥ clipsContent = true sur chaque slide**
```js
slide.clipsContent = true;
```
Évite que le titre 200px déborde en dehors du cadre.

**⑦ Couleur de texte selon le fond**
- Fond sombre (`primary`, `primaryAlt`, `accent`) → texte `white`
- Fond clair (`neutralLight`, `white`) → texte `dark`
Tester systématiquement le contraste avant d'afficher.

**⑧ Variables Figma — OBLIGATOIRE pour TOUTES les couleurs de marque**

Règle absolue : **1 couleur = 1 Brand Token**. Tous les fills (rectangles,
frames, slides) ET toutes les couleurs de texte (`text.fills`) des éléments
qui utilisent une couleur de la palette doivent être bindés à une variable
de la collection « Brand Tokens ». Pas d'exception pour les "petits"
éléments (eyebrows, labels, micro-décorations) — c'est précisément ces
éléments qui pluent les hex en dur dans le DS.

Utiliser **`solidVar("name")`**, jamais `solid(hex)` ni un objet color brut.

```js
// ✅ BON — couleurs de marque toujours bindées
slide.fills = solidVar("cream");
rect.fills = solidVar("accent");
eyebrow.fills = solidVar("accent");     // ← jamais solid(COLORS.accent)
title.fills = solidVar("primary");      // ← jamais solid(COLORS.primary)
strike.fills = solidVar("ardoise");

// ❌ INTERDIT pour les couleurs de marque
slide.fills = solid("#F5EFE6");
eyebrow.fills = solid(COLORS.accent);   // ← le piège classique
rect.fills = [{type:"SOLID", color:{r:0.94,g:0.93,b:0.9}}];
```

**Helper txt() — discipline cv: vs c:** : dans le helper de création de
texte, paramètre `cv:` (variable bindée) à utiliser **par défaut**.
Paramètre `c:` (raw hex) **uniquement** pour les exceptions documentées
ci-dessous. Si un nouveau call utilise `c:` sur une couleur du brand
sans justification → c'est un bug à corriger, pas une exception.

```js
// ✅ BON
txt(s, {text:"03 · COULEUR", cv:"accent", ...});
txt(s, {text:"Palette", cv:"primary", ...});

// ❌ INTERDIT (couleurs de marque)
txt(s, {text:"03 · COULEUR", c:COLORS.accent, ...});
txt(s, {text:"Palette", c:COLORS.primary, ...});
```

**Bénéfices** :
- L'utilisateur change une valeur dans le panneau Variables → toutes les
  slides se mettent à jour automatiquement (titres, eyebrows, lignes
  accent, fonds, labels — partout).
- Pas de hex hardcodé qui dérive de brand.md.
- Le client peut tester un changement de couleur live sans repasser par
  une regénération.
- Audit visuel : pour vérifier qu'une couleur est bien bindée, sélectionner
  le node dans Figma → panneau de droite doit montrer une **pastille
  hexagonale violette** à côté du fill (= lié à variable). Si rond gris =
  raw hex à patcher.

**Exceptions LIMITATIVEMENT tolérées pour fill non bindé** :

1. **Shades de la slide 09 Nuances** — couleurs pré-mélangées via
   `mixOnCream()` (cf. spec slide 09). NE PAS appliquer `opacity` sur un
   fill variable-bindé : Figma ignore le paramètre, résultat = 5 carrés
   identiques à 100%. Bug observé en prod.
2. **Couleurs de debug ou markers** (cleanup garanti à la fin du script).

**Tout le reste** doit être bindé. Si une couleur n'est pas encore dans
la collection « Brand Tokens » mais devrait l'être (ex : un gris technique
récurrent), **créer la variable** plutôt que de l'écrire en raw hex.

**Pour les "opacités" sur des couleurs de marque** :

❌ **NE FONCTIONNE PAS** (Figma ignore l'opacity sur fill bindé) :
```js
rect.fills = [{ ...solidVar("primary")[0], opacity: 0.8 }];  // rend à 100%
```

✅ **Solution** : pré-mélanger avec le fond, fill SOLID non bindé :
```js
function mixOnCream(srcHex, creamHex, op) {
  const s = hex(srcHex), c = hex(creamHex);
  return {
    r: op * s.r + (1 - op) * c.r,
    g: op * s.g + (1 - op) * c.g,
    b: op * s.b + (1 - op) * c.b,
  };
}
rect.fills = [{ type:"SOLID", color: mixOnCream(COLORS.primary, COLORS.cream, 0.8) }];
```

L'autre solution acceptée (sur élément SEUL, sans enfants) : `rect.opacity = 0.8`.
Mais inutilisable dès qu'il y a un label ou un contenu enfant — ils
deviendraient translucides aussi.

**⑨ Viewport + thumbnail — finalisation cohérente**
Dans le dernier appel, zoomer sur la **cover (slide 01)** :
```js
const cover = page.children.find(c => c.name.startsWith("01 ·"));
if (cover) figma.viewport.scrollAndZoomIntoView([cover]);
```
**Important** : la propriété `figma.fileThumbnailNodeId` **n'existe pas**
dans l'API Figma Plugin (erreur fréquente). Pour que le fichier reflète la
charte dès la liste Figma, on zoom sur la cover — Figma capture la
**thumbnail naturellement** depuis le viewport visible lors de la dernière
sauvegarde.

Passer `[cover]` (tableau) — `scrollAndZoomIntoView` attend un tableau de
nodes. Ne pas passer la `page` directement, un canvas n'est pas un node
visible.

**⑩ Vérification — `figma.root.findAll()` ne traverse PAS les pages non-courantes**
En mode dynamic-page (cas typique des fichiers récents), `figma.root.findAll(n => …)`
ne renvoie que ce qui est sur la page courante. Pour vérifier la présence des
slides sur une autre page, faire `figma.root.children.find(p => …).children`
explicitement, pas un findAll global. **Symptôme** : un script semble n'avoir
rien créé alors qu'il a fonctionné — c'est juste que la vérification cherchait
au mauvais endroit.

**⑪ Idempotence du `find` page — éviter le doublonnage**
Au démarrage de chaque appel `use_figma`, faire :
```js
let page = figma.root.children.find(p => p.name === "✦ Brand Book");
if (!page) { page = figma.createPage(); page.name = "✦ Brand Book"; }
await figma.setCurrentPageAsync(page);
```
**Ne pas** modifier le nom de la page entre les appels (ex: pour debug) —
sinon le `find` au prochain appel ne matche plus → nouvelle page créée →
slides éparpillées. Si on doit marquer un état, utiliser un rectangle marker
à l'extérieur du viewport, pas le nom de la page.

**⑫ Cleanup post-génération — purger ce qui n'est pas une slide**
Le test a révélé que des frames orphelins peuvent être créés (compteur > 14
en fin de génération). Toujours faire un cleanup final qui **préserve les
15 slides ET la frame des prompts images** (Étape 9quater) :
```js
page.children.filter(c => !/^\d{2} ·/.test(c.name) && c.name !== "✦ Prompts images").forEach(c => c.remove());
```
Cela garantit 15 slides exactement, nommées en français (cf. `SLIDE_NAMES_FR`) :
`01 · Couverture` → `15 · Quatrième de couverture`, plus la frame
`✦ Prompts images` posée à côté de la grille.

> **Migration** : si on relance la skill sur un fichier qui a déjà des slides
> aux anciens noms anglais (`01 · Cover`, `15 · Back Cover`), le regex
> `/^\d{2} ·/` les détecte quand même et les conserve. Pour migrer, il suffit
> de re-générer (les anciens seront purgés au cleanup et remplacés par les
> nouveaux noms français).

**⑬ Jamais de saut de ligne au milieu d'un mot — règle stricte**
Les anciens chapter openers utilisaient `"Lo\ngo"`, `"Typo\ngraphy"`, etc.
**C'est désormais interdit.** Les chapter openers doivent afficher le mot
français **entier sur une seule ligne** :
```js
// ✅ BON
title.characters = "Typographie";
title.fontSize = 200;          // taille adaptée pour tenir dans 1760px

// ❌ INTERDIT
title.characters = "Typo\ngraphy";    // coupure dans le mot
title.characters = "Lo\ngo";
title.characters = "Typography";      // libellé anglais — interdit par la règle de langue
```

**Méthode pour s'assurer que ça tient** : créer le node texte, mesurer sa
largeur (`title.width` après assignation des `characters`), et si elle
dépasse 1760px (largeur utile = 1920 - 80 - 80), **réduire `fontSize`
par paliers de 20px** jusqu'à ce que ça rentre. Ne **jamais** introduire
de `\n` dans un mot pour faire tenir.

Référence des tailles indicatives (libellés français) :
- "Logo" (4 lettres) → 360-400px
- "Couleur" / "Système" (7 lettres) → 260-300px
- "Typographie" (11 lettres) → 190-210px

Le saut de ligne `\n` reste autorisé **entre deux mots** (ex : cover "Identité\nverbale")
mais jamais dans un mot.

**⑭ Logo source obligatoire — jamais de placeholder texte « Logo »**
Les slides 01, 06 et 15 doivent afficher le **vrai logo de la marque** placé
via `placeLogo()` (cf. helper Étape 5). **Bannir absolument** :
```js
// ❌ INTERDIT — placeholder texte qui rend le livrable inutilisable
const logoText = figma.createText();
logoText.characters = "Logo";
logoText.fontSize = 96;
```

Si la map `LOGOS` est vide (aucun node `Logo couleur` détecté sur Page 1
du Figma cible), **stopper la génération** au tout début du script et
demander à l'utilisateur de coller son logo SVG dans le Figma (Page 1,
nom = `Logo couleur`), puis de relancer le skill.
Le brand book sans logo réel n'a aucune valeur de présentation client.

**⑮ Zéro tiret cadratin dans les libellés Figma**

Application de la **règle d'écriture transversale #1**. Tous les
`text.characters = "..."` envoyés à l'API Figma doivent passer un check
préalable : si la chaîne contient `—` ou `–`, l'agent réécrit la phrase
avec un remplacement contextuel **avant** d'envoyer le code
`use_figma`. Pas d'exception, même pour les eyebrows compactes type
`« TON · CONFIDENTIEL & PREMIUM — POSTURE · PARTENAIRE STRATÉGIQUE »`.

❌ **NE PAS faire** :
```js
txt(s, {text: "Espacement — White space généreux"});
txt(s, {text: "TON · CONFIDENTIEL — POSTURE · STRATÉGIQUE"});
txt(s, {text: "Une méthode verticale — pas un studio généraliste"});
```

✅ **Bonnes réécritures** :
```js
txt(s, {text: "Espacement : white space généreux"});
txt(s, {text: "TON · CONFIDENTIEL    ·    POSTURE · STRATÉGIQUE"});  // double point médian au lieu du tiret
txt(s, {text: "Une méthode verticale. Pas un studio généraliste."});
```

**Cas typiques à patcher dans la spec slides** :
- Slide 03 Identité verbale, bandeau attributs : remplacer
  `« TON · X — POSTURE · Y »` par `« TON · X    ·    POSTURE · Y »`
  (double espace + point médian).
- Slide 14 Éléments clés, descriptions des cards : « Espacement —
  White space » devient « Espacement : white space ».
- Slide 12 Hiérarchie, spec lignes : « H1 — Inter Bold · 40pt » devient
  « H1 · Inter Bold · 40pt » ou « H1 (Inter Bold, 40pt) ».

**⑯ Slots images — discipline `placeImage()`**

Tous les emplacements visuels passent par `placeImage()` (Étape 5).
Règles non négociables :

- **Toujours un groupe masque**, jamais un rectangle plein seul. La
  structure est `MoodBoard · <slot>` (groupe) → `<slot> · masque`
  (`isMask`, taille du slot) + `<slot> · image` (contenu, au-dessus).
  C'est ce qui permet à Arnaud de remplacer l'image **sans recrop**
  (cf. Étape 5). Un slot rendu en simple rectangle est un bug à corriger.
- **Jamais de déformation** : l'image clonée est cadrée COVER (scale
  uniforme), centrée sur le slot, le masque rogne le débordement.
  Jamais d'étirement non uniforme.
- **Nommage** : le groupe s'appelle `MoodBoard · <slot>` — c'est le
  contrat de remplacement avec l'utilisateur ET la clé de la QA 9bis.
  Ne pas renommer le groupe, ne pas le dégrouper.
- **Ordre z** : les slots se posent AVANT les textes qui les chevauchent
  (cover, logo-hero, hierarchie, univers-X). Les textes restent **hors
  du groupe masque** (enfants directs de la slide), sinon ils sont rognés.
- **Cadre** : le cadre du preset (Pop : 4px encre) est un rectangle
  séparé posé HORS du groupe masque (`PRESET.frame`), pour ne pas être
  rogné et survivre au remplacement de l'image. Jamais de stroke sur le
  masque lui-même (non rendu).
- **Placeholders présentables** : dégradé du preset sur la couche image,
  pas de gris dashed, pas de label « à compléter » sur la slide — la
  liste des images à produire vit dans `images-brief.md` (Étape 9ter).
- **Bleed** : les positions négatives ou dépassant 1920 sont voulues sur
  certains slots (police, chapitre-typo, hierarchie) si
  `PRESET.allowBleed` — `clipsContent: true` de la slide rogne proprement.
  Ne pas « corriger » ces coordonnées en QA.
- **Fills gradient non bindés** : exception documentée (les dégradés ne
  peuvent pas être liés aux variables Figma) — au même titre que les
  shades de la slide 09.

**⑰ Zéro fuite des valeurs d'exemple — le livrable porte le nom du client**

Ce SKILL.md contient des noms et des hex d'exemple (fokusia, Max
Piccinini / PICCININI, #06211a, #f58220…). Ils servent à illustrer la
grammaire, **jamais à remplir un livrable**. Le nom affiché partout
(titre cover, footer, crédits slide 15, brand.md, images-brief.md) est
`BRAND_NAME`, collecté en A.1.c ou lu dans `brand.md`. Si `BRAND_NAME`
est vide au moment de générer, **stopper et demander** — ne jamais
improviser ni recycler un exemple.

Check final (dernier appel `use_figma`, avant le récap) :

```js
const LEAKS = /fokusia|piccinini/i;
const leaked = [];
for (const slide of page.children.filter(c => /^\d{2} ·/.test(c.name))) {
  slide.findAll(n => n.type === "TEXT" && LEAKS.test(n.characters))
       .forEach(n => leaked.push(`${slide.name} → "${n.characters.slice(0, 40)}"`));
}
// leaked.length > 0 → corriger immédiatement avec BRAND_NAME, re-vérifier
```

Même vigilance côté fichiers : `brand.md`, `landing-page.md`,
`images-brief.md` ne doivent contenir aucun nom d'exemple (sauf si le
client s'appelle vraiment comme ça).

---

## Étape 9bis — QA visuelle automatique des 15 slides

**Après** que les 15 slides sont écrites dans Figma et **avant** l'Étape 10
(récap final), exécuter une QA visuelle automatique. C'est ce qui empêche
de livrer un brand book avec un débordement de texte ou un contraste raté.

### 9bis.a — Re-screenshot des 15 slides

Boucler sur les 15 slides via `get_screenshot` (1 appel par slide, avec
le `nodeId` de chaque frame stocké pendant l'Étape 5). Stocker les
15 screenshots en mémoire — pas besoin de les exposer à l'utilisateur
sauf si une slide échoue à la QA.

### 9bis.b — Auto-critique sur 6 axes

Pour chaque slide, vérifier visuellement (le LLM regarde le screenshot
et juge) selon **6 axes** :

| Axe | Question concrète | KO si... |
|---|---|---|
| 1 · **Débordement de texte** | Est-ce qu'un bloc de texte sort de son conteneur ou est tronqué ? | Texte coupé en bord de frame, ellipsis `…` non voulu, ligne qui sort de la safe zone |
| 2 · **Contraste** | Le texte principal est-il lisible sur son fond (y compris sur les images) ? | Ratio < AA (4.5:1 pour body, 3:1 pour large text) — typiquement gris sur gris, label clair sur image claire sans voile |
| 3 · **Logo présent & non cassé** | Le logo apparaît-il bien sur slides 01, 06, 15 ? Est-il déformé ou pixelisé ? | Logo absent, étiré, écrasé, mauvaise variante (sombre sur sombre), pixelisation visible |
| 4 · **Footer cohérent** | Footer présent, aligné, avec bon numéro de slide ? | Footer manquant, désaligné, numéro de slide faux ou en double |
| 5 · **Slide vide / inutile** | La slide porte-t-elle réellement une information ? | Slide presque vide (1 titre + 1 fond), placeholder `[À COMPLÉTER]` visible, nom d'exemple (fokusia, Piccinini) au lieu de `BRAND_NAME`, contenu dupliqué d'une autre slide |
| 6 · **Slots images (groupe masque)** | Chaque slot attendu (registre Étape 5) est-il un groupe `MoodBoard · <slot>` avec masque + contenu ? | Slot rendu en simple rectangle (pas de masque), groupe manquant, image étirée (cercles → ovales, visages déformés), placeholder gris brut au lieu du dégradé preset, texte rogné car posé DANS le groupe masque |

Format de la sortie interne (utilisée par l'agent — pas montrée à
l'utilisateur sauf si KO) :

```js
const qaResults = [
  { slide: "01 Cover", status: "OK" },
  { slide: "02 Sommaire", status: "OK" },
  { slide: "03 Identité verbale", status: "KO",
    issues: ["1·débordement: section 'Mots à utiliser' coupée après 'rigueur,…'"] },
  // ...
];
```

### 9bis.c — Si KO : proposer un fix auto

Pour chaque slide KO, **proposer immédiatement un fix** (sans demander
à l'utilisateur — il valide à la fin) :

| Type d'échec | Fix auto |
|---|---|
| Débordement texte | Réduire taille de police d'1 cran (ex : 18→16px) **ou** scinder le texte en 2 colonnes **ou** réduire le contenu à l'essentiel |
| Contraste KO | Changer la couleur du texte vers le neutre opposé (texte foncé → texte clair) ou augmenter l'opacité |
| Logo absent | Re-tenter le `placeLogo()` avec la variante alternative (image-hash si le node clone a échoué) |
| Footer KO | Re-injecter le footer depuis le helper commun de l'Étape 6 |
| Slide vide | Re-générer la slide depuis sa spec à l'Étape 8 |
| Slot image manquant / déformé | Re-tenter `placeImage()` depuis le registre Étape 5 ; si l'image clonée est déformée, repasser le node en crop FILL ou retomber sur le placeholder dégradé |
| Label illisible sur image | Insérer un voile `noir` opacité 25% entre l'image et les textes |

Appliquer les fixes via `use_figma` (1 batch de modifications), puis
**re-screenshoter uniquement les slides corrigées** et re-valider.
Max 2 passes — au 3e échec sur la même slide, signaler à l'utilisateur
et lui demander quoi faire.

### 9bis.d — Sortie utilisateur

Si **tout est OK du 1er coup** → ne rien dire (silence = qualité), passer
direct à l'Étape 10.

Si **des fixes ont été appliqués** → 3 lignes max :

```
✦ QA visuelle : 3 slides corrigées automatiquement
  · 03 Identité verbale — texte recadré (débordement)
  · 09 Nuances — contraste accent renforcé
  · 15 4e de couverture — logo re-placé (image-hash)
```

Si **une slide reste KO après 2 fixes** → bloquer l'Étape 10 et exposer
le screenshot de la slide problématique à l'utilisateur avec 2 options :
1. Corriger manuellement et relancer la QA
2. Accepter en l'état et passer à l'Étape 10

---

## Étape 9ter — Écrire `images-brief.md` (10 prompts exécutables en direct dans Figma)

**Condition** : au moins un slot rendu en placeholder (la liste
`PLACEHOLDER_SLOTS` collectée par `placeImage()` est non vide). Si tous
les slots sont remplis, sauter cette étape.

**Intention** : ce fichier n'est pas un brief abstrait, c'est une
**liste de 10 prompts prêts à exécuter dans le plugin IA d'image de
Figma** (le générateur que l'utilisateur a directement dans Figma).
Chaque prompt est **autoportant** : sujet, composition, orientation et
direction artistique (palette hex + traitement) sont déjà dans le bloc,
rien à rajouter. L'utilisateur copie un bloc, le colle dans le plugin,
génère, et glisse l'image obtenue dans la couche `<slot> · image` du
groupe masque correspondant (le masque rogne, aucun recrop).

> **Pourquoi 10 et pas 16** (décision 2026-06) : générer 16 images
> noie l'utilisateur. On priorise les **10 slots qui portent le plus
> visuellement**, et on regroupe les séries : les 6 `univers-*` =
> **1 seul prompt-série** (6 variations du même traitement). 10 prompts
> couvrent tout le poids visuel du deck.

**Les 10 prompts prioritaires** (ordre fixe) :

| # | Slot(s) | Slide | Pourquoi prioritaire |
|---|---|---|---|
| 1 | `cover` | 01 | Première impression, pleine hauteur |
| 2 | `identite` | 03 | Incarne le ton de la marque |
| 3 | `univers-1…6` *(série)* | 04 | Le moodboard, cœur de l'univers |
| 4 | `chapitre-logo` | 05 | Ouverture de chapitre, grand format |
| 5 | `logo-hero` | 06 | Mise en scène du logo |
| 6 | `chapitre-couleur` | 07 | Bande verticale immersive |
| 7 | `chapitre-typo` | 10 | Ouverture typo, bleed |
| 8 | `hierarchie` | 12 | Panneau pleine hauteur |
| 9 | `chapitre-systeme` | 13 | Ouverture système |
| 10 | `elements` | 14 | Bandeau panoramique de clôture |

Les slots restants (`sommaire-*`, `nuances`, `police`, `palette-col-*`)
gardent leur placeholder dégradé : ils sont décoratifs, pas prioritaires.
Les mentionner en fin de fichier dans une ligne « slots secondaires :
placeholders conservés, prompts sur demande ».

Structure du fichier :

```markdown
# Images du brand book · [Nom de la marque]

> 10 prompts prêts à exécuter dans ton plugin IA d'image Figma.
> Workflow : copier un bloc → coller dans le plugin → générer →
> glisser l'image sur la couche « <slot> · image » du groupe
> « MoodBoard · <slot> » (le masque rogne, pas de recrop).
> Alternative : déposer l'image sur la page Assets, la nommer
> « IMG · <slot> », relancer le skill en mode BOOK.

## 1 · cover — Couverture (vertical 2:3)
[Prompt FR autoportant, 2-4 phrases : sujet concret + composition +
orientation explicite + « Palette : citron #E9F355, vert forêt #334C15,
ciel #94DFFC sur fonds clairs. Photo éditoriale lumière naturelle,
mouvement réel, léger grain. »]

## 2 · identite — Identité verbale (portrait 3:4)
[…]

## 3 · univers (série de 6, paysage 3:2)
> Génère 6 variations du même traitement, une par sujet :
> Foulée · Bitume · Lumière · Tracé · Souffle · Arrivée.
[Prompt de base FR + la liste des 6 sujets]

…(jusqu'à 10)…

---
Slots secondaires (placeholders dégradés conservés) : sommaire-1…5,
nuances, police, palette-col-*. Prompts disponibles sur demande.
```

> **Le `.md` reste écrit** (archive + portabilité hors Figma), mais la
> surface de travail réelle d'Arnaud est la **frame Figma de l'Étape
> 9quater** : il lit le prompt sur le canvas, le copie, le colle dans son
> plugin IA, sans jamais quitter Figma.

Règles d'écriture des prompts :
- **En français**, 2-4 phrases, sujet concret en premier (« Un escalier
  urbain gravi par un coureur au lever du jour »), puis composition,
  puis orientation, puis la DA (palette hex + traitement).
- **DA bakée dans chaque bloc** : reprendre 2-3 hex de la table Couleurs
  de `brand.md` + le traitement déduit du preset et de Q13 (photo
  lumière naturelle / duotone / N&B). Pas de bloc DA séparé à recoller.
- **Orientation explicite** entre parenthèses dans le titre ET dans le
  prompt (« cadrage vertical 2:3 ») : avec le masque, la taille pixel
  n'importe pas, seule l'orientation compte pour bien remplir le slot.
- **Série univers** : 1 prompt de base + les 6 labels thématiques de la
  slide 04 (dérivés de Q12) comme variations, « même traitement,
  série cohérente ».
- Zéro tiret cadratin (règle d'écriture #1), zéro nom d'exemple
  (guardrail ⑰).

---

## Étape 9quater — Poser les 10 prompts sur une frame Figma

**Même condition que 9ter** (au moins un slot en placeholder). Après
avoir écrit `images-brief.md`, **recopier les 10 prompts sur une frame
Figma** posée à côté de la grille de slides, sur la page `✦ Brand Book`.

> **Pourquoi dans Figma et pas seulement dans le `.md`** (demande Arnaud,
> 2026-06) : son générateur d'image IA est **dans Figma**. Si les prompts
> vivent sur une frame du canvas, le cycle complet (lire le prompt →
> copier → coller dans le plugin → déposer l'image dans le slot masque)
> se fait sans jamais sortir de Figma. Le `.md` reste écrit pour
> l'archive et la portabilité, mais la frame est la surface de travail.

**Placement** : la frame `✦ Prompts images` se pose à droite de la grille
2 colonnes, soit `x = 2 × 2160 = 4320`, `y = 0`. Largeur 1200, hauteur
auto selon le nombre de cards. Le cleanup ⑫ la préserve explicitement.

**Construction** (dans le dernier appel `use_figma`, après les slides) :

```js
// PROMPTS : tableau [{ n, slot, orient, body }] des 10 prompts (mêmes
// textes que images-brief.md). bgPrompts/ink = couleurs lisibles du preset.
function buildPromptsFrame(page, PROMPTS, C) {
  let f = page.children.find(n => n.name === "✦ Prompts images");
  if (f) f.remove();                       // idempotent : on régénère
  f = figma.createFrame();
  f.name = "✦ Prompts images";
  f.x = 4320; f.y = 0;
  f.fills = solidVar(C.bg);                // fond clair du preset
  f.layoutMode = "VERTICAL";
  f.primaryAxisSizingMode = "AUTO";
  f.counterAxisSizingMode = "FIXED";
  f.resize(1200, 100);
  f.paddingTop = f.paddingBottom = 80;
  f.paddingLeft = f.paddingRight = 80;
  f.itemSpacing = 24;
  f.clipsContent = false;

  // Titre
  const h = figma.createText();
  h.fontName = FONT_DISPLAY; h.fontSize = 56;
  h.characters = "Prompts images";
  h.fills = solidVar(C.ink); f.appendChild(h);
  const sub = figma.createText();
  sub.fontName = F.r; sub.fontSize = 17; sub.layoutSizingHorizontal = "FILL";
  sub.characters = "Copie un bloc, colle-le dans ton plugin IA, génère, "
    + "puis glisse l'image sur la couche « <slot> · image » du groupe "
    + "« MoodBoard · <slot> ». Le masque rogne, pas de recrop.";
  sub.fills = solidVar(C.ink); sub.opacity = 0.75; f.appendChild(sub);

  // 10 cards
  for (const p of PROMPTS) {
    const card = figma.createFrame();
    card.layoutMode = "VERTICAL"; card.itemSpacing = 10;
    card.paddingTop = card.paddingBottom = 28;
    card.paddingLeft = card.paddingRight = 28;
    card.cornerRadius = PRESET.imageRadius;
    card.fills = solidVar("white");
    if (PRESET.frame) { card.strokes = solid(COLORS[PRESET.frame.color]); card.strokeWeight = 2; }
    f.appendChild(card);
    card.layoutSizingHorizontal = "FILL";

    const head = figma.createText();
    head.fontName = F.sb; head.fontSize = 15;
    head.letterSpacing = { value: 1, unit: "PIXELS" };
    head.characters = `${String(p.n).padStart(2,"0")} · ${p.slot.toUpperCase()} · ${p.orient}`;
    head.fills = solidVar("primary"); card.appendChild(head);

    const body = figma.createText();
    body.fontName = F.r; body.fontSize = 18; body.lineHeight = { value: 160, unit: "PERCENT" };
    body.characters = p.body;            // texte sélectionnable = copiable
    body.fills = solidVar(C.ink);
    card.appendChild(body); body.layoutSizingHorizontal = "FILL";
  }
  return f;
}
```

**Notes d'implémentation** :
- Auto-layout vertical : la frame grandit toute seule selon le nombre de
  cards et la longueur des prompts. Pas de calcul de hauteur manuel.
- Le **texte du body est un node texte normal**, donc sélectionnable et
  copiable directement dans Figma (double-clic, Cmd+A, Cmd+C).
- `FONT_DISPLAY` = la famille Display du preset (préchargée Étape 5).
  `C = { bg, ink }` : un fond clair et une encre lisibles quel que soit
  le preset (ex Pop : bg `white`, ink `encre`).
- **Idempotent** : on supprime la frame existante avant de la recréer,
  comme pour les slides. Le cleanup ⑫ la garde (nom exact
  `✦ Prompts images`).
- Si le preset est sombre (Luxe), prendre `bg = primaryDark`, `ink =
  cream`, cards en `primary` clair : la frame reste lisible.

---

## Étape 10 — Confirmer et récapituler

```
✦ Brand Book installé pour [Nom de la marque]

Preset visuel : [Luxe éditorial | Minimal tech | Crème organique |
Pop contrasté | Corporate clair] (dérivé de [réponses clés])

Fichiers générés :
  · brand.md            (source de vérité — tokens + identité verbale + univers)
  · landing-page.md     (structure + copy de landing — template [A|B|C])
  · images-brief.md     (10 prompts FR prêts à exécuter dans le plugin IA Figma)

Page Figma créée : "✦ Brand Book"
  · 15 slides + frame "✦ Prompts images" (10 prompts copiables sur le canvas)
URL : [URL fichier]

15 slides 1920×1280 (tous les libellés Figma sont en français) :
  01 · Couverture
  02 · Sommaire
  03 · Identité verbale     →  04 · Univers & ambiance
  05 · Chapitre — Logo      →  06 · Logo
  07 · Chapitre — Couleur   →  08 · Palette de couleurs → 09 · Nuances
  10 · Chapitre — Typographie → 11 · Police → 12 · Hiérarchie typographique
  13 · Chapitre — Système   →  14 · Éléments clés → 15 · Quatrième de couverture

Tokens injectés :
  · primary  [hex] / accent [hex] / neutres
  · display  [font] / body [font]
  · 6 règles de composition
  · Identité verbale (ton, formule, mots, CTAs)  ← visibles slide 03
  · Univers visuel (mood, références, mots-clés) ← visibles slide 04

Images :
  · [X]/[total] slots remplis depuis les nodes IMG du Figma
  · [N] slots en placeholder dégradé → prompts dans images-brief.md
    (générer puis drag-drop sur les nodes "MoodBoard · …")

Prochaine étape suggérée :
→ Dans Figma : lire un prompt sur la frame "✦ Prompts images", le coller
  dans ton plugin IA, glisser l'image générée sur la couche
  "<slot> · image" du groupe masque (le masque rogne, pas de recrop)
→ Relire landing-page.md et compléter témoignages/logos/chiffres clients
→ Déposer les images manquantes dans ./images/ (chemins listés dans la
  section "Données à compléter" du landing-page.md)
→ Export PDF du brand book depuis Figma (File → Export → Slides as PDF)
→ Copier-coller landing-page.md dans Claude / Framer AI / v0 / Lovable
  pour générer la landing finale (le bloc Prompt en tête cadre la mission)
→ brand-social : visuels LinkedIn cohérents avec ce brand book
```

---

## Cas limites

**Table Composition rules absente ou < 6 entrées dans brand.md** — générer
la slide 12 avec les règles disponibles complétées par la section
**Personnalité graphique** du même `brand.md` (jusqu'à atteindre 6 cards).
Signaler à l'utilisateur que les règles gagneraient à être complétées
directement dans `brand.md`.

**Une seule couleur primaire (pas d'alt)** — réutiliser `primary` partout
où `primaryAlt` est attendu (chapter openers, palette colonne 2). Signaler
à l'utilisateur que le brand book gagnerait à avoir une 2ème couleur.

**Pas de fichier Figma fourni** — créer un nouveau fichier via
`mcp__figma__add_figma_file` si l'utilisateur le souhaite, sinon générer
uniquement un export local (HTML preview dans `outputs/`) et signaler
qu'il faudra fournir un Figma pour la version éditable.

**Aucun node logo dans le Figma cible** — **bloquer la génération du
brand book** et demander à l'utilisateur de coller son logo SVG dans le
Figma cible (Page 1) et de le nommer `Logo couleur`. Ne **jamais**
générer les slides avec un placeholder texte "Logo" 96px ou un rectangle
gris — le brand book serait inutilisable pour présenter au client. Le
seul cas acceptable de génération sans logo est si l'utilisateur le
demande explicitement (« génère sans logo, je l'intégrerai après ») —
alors afficher un rectangle hachuré 240×96 (slide 01) et 480×320
(slide 06) avec la mention claire « LOGO À INSÉRER ».

**Figma cible avec uniquement `Logo couleur`** — utiliser la
variante couleur sur tous les fonds (clairs et sombres), signaler dans
le récap final que la variante blanche n'a pas été fournie. Sur slide 01,
15 et chapter openers, mentionner `[Variante blanche à produire]` discret
en pied de slide. Ne **jamais** re-coloriser automatiquement un node Figma
existant (les SVG complexes ont souvent plusieurs fills imbriqués).

**Dossier `./images/` vide ou incomplet** — pas bloquant. La landing-page.md
listera explicitement les chemins manquants dans la section « Données à
compléter par l'utilisateur ». (Rappel : `./images/` ne concerne que la
landing — le brand book Figma se nourrit des nodes `IMG · <slot>`.)

**Aucun node `IMG · <slot>` dans le Figma cible** — cas nominal, pas
bloquant : tous les slots sont rendus en placeholders dégradés du preset
et `images-brief.md` fournit les prompts. Le brand book reste présentable
tel quel (les dégradés sont aux couleurs de la marque, pas des zones
grises).

**Node `IMG · <slot>` avec un nom de slot inconnu** (ex `IMG · héro`,
faute de frappe) — signaler au récap (« node IMG · héro ignoré : slot
inconnu, slots valides : cover, identite, … ») plutôt que d'échouer
silencieusement.

**brand.md sans formule signature** — utiliser le positionnement ou la
promesse comme phrase d'exemple dans la slide 10 Type Hierarchy. Pour le
`landing-page.md`, dériver une formule depuis la promesse Q4 en mode Q5
au lieu de laisser vide.

**Mode LANDING déclenché sans brand.md** — refuser proprement et expliquer
qu'il faut d'abord lancer le mode CRÉATION (« j'ai besoin de brand.md pour
calibrer la copy — démarrons par une interview courte »). Ne jamais inventer
de positionnement à partir de rien.

**Positionnement ambigu dans brand.md** (« Qui » ou « Pour qui » vide) —
poser une question groupée avant l'Étape A.5 pour clarifier, puis enchaîner.
Sans persona clair, la copy de la landing tombe dans le générique.

**3 templates de landing ne couvrent pas le cas** (ex : e-commerce produit
physique, association, événement) — proposer à l'utilisateur de partir du
TEMPLATE A (le plus générique) puis adapter les sections. Mentionner
explicitement dans le commentaire de tête du fichier que le template a été
détourné.

---

## Références — grammaire de mise en page intégrée

La grammaire visuelle (brand book Figma) est **entièrement intégrée à ce SKILL.md** :
- **Étape 4** : règles de mise en page communes (taille slide, padding,
  footer, slots images, chapter openers)
- **Étape 4bis** : les 5 presets visuels (Luxe éditorial, Minimal tech,
  Crème organique, Pop contrasté, Corporate clair) + sélection
- **Étape 5** : helpers `placeLogo()` / `placeImage()` + registre des
  16 slots images
- **Étape 6** : patron commun pour chaque slide (helpers, footer)
- **Étape 8** : spécifications détaillées par slide (coordonnées x/y,
  sizing, fonts, fills, structure, slots)
- **Étape 9ter** : génération de `images-brief.md` (10 prompts FR)
- **Étape 9quater** : frame Figma `✦ Prompts images` (mêmes 10 prompts,
  copiables sur le canvas pour le plugin IA)

La grammaire de la landing page est dans un fichier de référence séparé :
- **`references/landing-page-templates.md`** — 3 templates détaillés
  (Offre/SaaS B2B · Produit · Personal brand) + logique de sélection +
  règles de copy transversales. Lu par l'agent à l'Étape A.5.

Si tu veux modifier l'identité visuelle du brand book Figma (positions,
sizing, couleurs par défaut, tailles de texte), édite directement les
Étapes 4, 6 et 8 de ce SKILL.md. Si tu veux modifier la structure ou la
copy par défaut de la landing, édite `references/landing-page-templates.md`.
