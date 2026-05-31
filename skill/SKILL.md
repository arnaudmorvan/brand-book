---
name: brand-book
description: >
  Pipeline charte de marque. Interview 18 questions (5 vagues : positionnement,
  ton, typo, univers, persona), écrit `brand.md` et `landing-page.md` (copy
  rédigée + 3 templates auto : offre/SaaS B2B, produit, personal brand —
  prêt à coller comme prompt), génère brochure Figma 15 slides 1920×1280
  dont le design s'adapte aux réponses. Détecte automatiquement les logos
  (nodes "Logo couleur" + "Logo blanc" sur Page 1 du fichier Figma cible)
  et scanne `./images/` du dossier projet — pas d'upload manuel. CRÉATION : "scan ma charte", "crée le brand system", "j'ai un
  logo", "génère brand.md", "fais le brand book", "brand setup", "structure
  ma landing", "page de vente de la marque". BOOK (brand.md existe) :
  "brand guidelines propre", "brand book Figma", "transforme mes tokens en
  brand book". LANDING (brand.md existe) : "refais la landing", "structure
  ma page de vente", "génère landing-page.md". Pour produire du contenu
  (LinkedIn, PPTX) sans brand.md — lancer CRÉATION d'abord.
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
| Nom de la marque | Oui | Demander si absent |
| Couleur(s) principale(s) | Oui (min. 1 hex) | Demander si absent, ou extraire visuellement depuis le node `Logo couleur` via `get_screenshot` (rendu Figma natif) |
| Positionnement / cible court | Recommandé | Brief, profil LinkedIn, conversation |

Si nom ou couleurs manquent, poser **une seule question groupée** pour
tout récupérer d'un coup.

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

### A.1.e — Préparer les images pour Figma et la landing

Pour chaque image présente dans `AVAILABLE_IMAGES`, **uploader vers Figma**
au début de la Phase B (un seul appel `upload_assets` batché) et stocker
les hashs dans une map :

```js
const IMAGE_HASHES = {
  "hero.jpg": "abc123…",
  "photo-profil.jpg": "def456…",
  // ...
};
```

**Usage dans le brand book Figma** : la slide 04 Universe & Mood peut
piocher dans ces images pour son moodboard (3-4 visuels en grille) si elles
correspondent à l'ambiance Q11+Q12. Sinon, garder les blocs colorés du
template.

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
> 2. Le handle à afficher en footer (ex: @nom-de-la-marque)
> 3. Confirmer le nombre de couleurs à exposer en palette (4, 5 ou 6)

---

## Étape 3 — Mapper YAMLs templates → slides à produire

Le brand book final contient **15 slides** 1920×1280. Chaque slide
réutilise la composition d'un YAML template, en y injectant les tokens
du brand actuel + les réponses de l'interview (Phase A.2).

| # | Slide | Tokens / contenu injectés |
|---|---|---|
| 01 | Cover | `cream` ou `primaryDark` (bg) + `accent` (eyebrow) + nom marque (Bold) + tagline + formule signature XXL |
| 02 | Sommaire | Liste des **5 chapitres** avec numéros 01/02/03/04/05 — fond `cream` |
| 03 | **Verbal Identity** | Ton dominant + formule signature en grand (H1 80-120px) + 2 colonnes **Mots à utiliser** / **Mots à éviter** (barrés) + 3 CTAs exemples + ligne posture |
| 04 | **Universe & Mood** | Fond `primary` ou `primaryDark` (immersif) — référence univers + ambiance + imagerie + 3-5 chips mots-clés + paragraphe moodboard depuis `brand.md` |
| 05 | Chapter opener — Logo | Fond `accent`, titre **"Logo"** sur une seule ligne (mot entier, jamais coupé), taille adaptée pour tenir dans la largeur disponible + sommaire chapitre |
| 06 | Content — Logo | Variantes du logo (couleur, blanc, sur primary) + safe zone + min size "128px / 35mm" |
| 07 | Chapter opener — Couleur | Fond `primary`, titre **"Couleur"** sur une seule ligne (jamais coupé) |
| 08 | Content — Palette de couleurs | 4-6 **colonnes 224-251×960 cornerRadius 24** côte à côte — chaque colonne = un fond couleur + nom + CMYK + HEX + RGB |
| 09 | Content — Nuances | Échelle de tints pour `primary` et `accent` (100/80/60/40/20) — 2 lignes de 5 carrés cornerRadius 24, **opacité vraiment appliquée sur le fill** |
| 10 | Chapter opener — Typographie | Fond `accent`, titre **"Typographie"** sur une seule ligne (jamais coupé) |
| 11 | Content — Police | Fond `primary` ou `primaryDark` — "Aa" géant 480px + alphabet complet + nom de la typeface |
| 12 | Content — Hiérarchie typographique | Cascade H1→Small avec specs (taille / interligne / graisse) et phrases d'exemple issues de `brand.md` |
| 13 | Chapter opener — Système | Fond `primaryDark`, titre **"Système"** sur une seule ligne (jamais coupé) |
| 14 | Content — Éléments clés | 6 cards 3×2 (cornerRadius 16) depuis la liste **Composition rules** de `brand.md` |
| 15 | Back cover | Fond `primaryDark` + formule signature en titre XXL + crédits + mention "brand-book skill" |

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

### Footer (toutes les slides)
- À y=1228, hauteur 24
- Gauche (x=80) : handle en 16px regular — couleur `neutral.darkest`
- Droite (x=1757) : "page XX/15" en 16px regular — même couleur

### Chapter openers
- Fond pleine couleur (`accent`, `primary`, `primary.dark` en alternance)
- Grand titre Inter Display Bold, **mot entier sur une seule ligne — jamais
  coupé en plein milieu d'un mot** (ex : "Logo", "Colour", "Typography",
  "System"). Si le mot ne tient pas à 240px, **réduire la taille de police
  jusqu'à ce qu'il tienne** dans la largeur utile (1760px) plutôt que le
  casser. Échelle indicative selon le mot :
  - 4 lettres ("Logo") → 360-400px possibles
  - 7 lettres ("Couleur", "Système") → 260-300px
  - 11 lettres ("Typographie") → 190-210px
- Couleur contrastée selon le fond (cream sur sombre, primary sur accent clair)
- Sommaire 4 lignes en bas (01/02/03/04 + intitulés)
- Pas de colonne descriptive

> **Pourquoi ne pas casser les mots** : couper "Typo / graphy" ou "Lo / go"
> donne un effet "magazine éditorial" qui peut sembler stylé mais qui :
> (1) nuit à la lisibilité pour les non-francophones / non-anglophones,
> (2) brise la grammaire typographique propre et signale "design forcé",
> (3) rend la slide ambiguë au premier regard (« c'est quoi ce mot tronqué ? »).
> On préfère un mot entier lisible, quitte à réduire la taille.

---

## Étape 4bis — Adaptation du design selon `brand.md`

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
| Footer page | **« page XX/15 »** *(le mot "page" reste en français — pas "Page" majuscule)* |

> **Pourquoi cette règle stricte** : Arnaud livre les brand books à des
> clients français. Les titres en anglais (« Typography », « Colour
> Palette ») cassent la lecture et signalent « template générique » au
> lieu de « brand book sur mesure ». La cohérence linguistique est un
> marqueur de qualité.

### Préambule : helpers et constantes

```js
// Couleurs depuis la table Couleurs de brand.md (à injecter)
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

function addFooter(slide, { handle, page, total, color = COLORS.dark }) {
  const h = figma.createText();
  h.fontName = {family:"Inter Display", style:"Regular"};
  h.fontSize = 16;
  h.characters = handle;
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
- Fond : `primary` (foncé, ex #06211a)
- **Logo source réel en haut-gauche** : appeler le helper
  `placeLogo(slide01, { x: 128, y: 80, w: 240, h: 96, variant: "white" })`
  défini au préambule de l'Étape 6. Hauteur 96px, largeur proportionnelle
  au ratio du logo (ne pas écraser).
  **Variante** : sur ce fond sombre, le helper utilise `LOGOS.white` si
  fourni dans le Figma cible, sinon fallback automatique sur `LOGOS.color`
  avec une mention au récap final.
  **Ne jamais** remplacer par un texte "Logo" 96px — c'est un placeholder
  inacceptable qui rend le brand book inutilisable.
- Tagline orange (`accent`) en milieu 30px Inter Display Medium, x=128 y=656
- Titre marque XXL : 2 mots sur 2 lignes, **200px Inter Display Regular**,
  couleur `white`/`accent`, x=128 y=736

### 02 · Sommaire
- Fond : `neutralLight`
- Header gauche : **"Sommaire"** 40px display (et non plus "Index")
- Liste : **5 lignes** — **"01 Identité"**, **"02 Logo"**, **"03 Couleur"**,
  **"04 Typographie"**, **"05 Système"** en 64px display + numéro accent à gauche
- Chaque ligne avec une mini-description à droite (16-20px Regular) — la
  description est rédigée en français (ex : « Ton, formule, univers »,
  « Variantes et zone de protection », « Palette et nuances », etc.)

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

**Colonne droite — Moodboard 6 rectangles 3×2** (NOUVEAU 2026-05) :

> **Pourquoi 6 rectangles plutôt qu'un paragraphe** : le moodboard
> textuel décrit en mots ce que les photos devraient montrer. C'est
> utile pour briefer une banque image, pas pour valider visuellement
> l'ambiance avec le client. Avec 6 placeholders prêts pour drag-drop,
> l'utilisateur transforme la slide en **vrai moodboard photo** en
> 30 secondes : il drag-drop ses 6 photos depuis son ordinateur sur les
> rectangles, Figma applique automatiquement l'image comme fill.

- Eyebrow **`MOODBOARD · 6 photos à déposer`** 11px Semi Bold accent
  letter-spacing 30 (x=720, y=80)
- **Grille 3 colonnes × 2 lignes**, gap 16px :
  - Zone : x=720 à x=1840 (largeur 1120)
  - Rectangle width : `(1120 - 2 × 16) / 3 = 362.67` → arrondi à `362`
  - Rectangle height : `240` (ratio ~3:2 paysage)
  - Position : `x = 720 + col × (362 + 16)`, `y = 140 + row × (240 + 16)`
- **Style des rectangles** (placeholder) :
  - `cornerRadius: 4`
  - Fill cream à 8% opacity (signal visuel "vide à remplir" sur fond sombre)
  - Stroke `accent` 1px **dashed** `[6, 6]` (signal "à compléter")
- **Contenu de chaque rectangle** (centré) :
  - Label thématique 14px Medium cream — ex : `1 · Architecture`,
    `2 · Matériaux`, `3 · Lumière`, `4 · Détail / objet`,
    `5 · Portrait situation`, `6 · Texture / ambiance`
  - Sous-label 11px Regular accent : **« Glisser une image ici »**
- **Adaptation des labels à brand.md Q12** (Référence d'univers) :
  - Architecture & passages → Architecture · Matériaux · Lumière · Détail · Portrait · Texture
  - Nature & matières brutes → Paysage · Texture · Matière · Geste · Détail organique · Ambiance
  - Tech minimale & grilles → Grille · Texture data · Détail interface · Lumière froide · Geste tech · Composition
  - Éditorial luxe & mode → Détail produit · Texture luxe · Portrait · Composition · Geste · Atmosphère

**Légende sous la grille** (y=696, w=1120) :
- 13px Regular cream lh=160 : *« Glisser-déposer 6 photos depuis votre
  ordinateur sur chaque rectangle. Figma applique automatiquement
  l'image comme fill. Brief suggéré : [reprendre les 3-4 attributs
  Univers de brand.md en 1 phrase]. »*

**Workflow utilisateur — drag-drop dans Figma** :
1. Ouvrir la slide 04 dans Figma
2. Sélectionner une photo dans le Finder / Explorer
3. Glisser-déposer sur le rectangle cible
4. Figma applique automatiquement l'image comme fill, le rectangle
   garde ses dimensions, l'image est en `scaleMode: FILL` par défaut
5. Pour ajuster le cadrage : double-clic sur le rectangle → onglet
   "Crop" dans le panneau de droite

### 05/07/10/13 · Chapter openers- Fond pleine couleur alterné : 05=`accent`, 07=`primaryAlt`, 10=`accent`,
  13=`primary`
- Numéro de chapitre en haut-gauche : **"02" / "03" / "04" / "05"** en 96px blanc
  (01 est désormais réservé à Identité sur les slides 03-04)
- Titre **mot français entier sur une seule ligne — jamais coupé** :
  **"Logo"**, **"Couleur"**, **"Typographie"**, **"Système"** — Inter Display
  Bold, blanc. Taille adaptée pour tenir dans la largeur utile (1760px)
  sans débordement :
  - "Logo" (4 lettres) → 360-400px
  - "Couleur", "Système" (7 lettres) → 260-300px
  - "Typographie" (11 lettres) → 190-210px
  Si le `clipsContent: true` détecte un débordement lors du test, **réduire
  la taille de 20px** et réessayer plutôt que de casser le mot.
- Liste des sections du chapitre en bas à droite (24px Medium) — rédigée
  en français

### 06 · Content — Logo
- Fond : `neutralLight`
- Colonne gauche (x=80) : titre "Logo" 40px + paragraphe descriptif depuis `brand.md`
- Zone droite : **3 cartouches 480×320 cornerRadius 24** avec variantes du
  **logo source réel** (jamais un placeholder texte) :
  - Cartouche 1 : fond `white` + logo en **couleur originale** (`LOGOS.color`)
  - Cartouche 2 : fond `accent` + logo en **blanc/cream** (`LOGOS.white`)
  - Cartouche 3 : fond `primary` + logo en **blanc/cream** (`LOGOS.white`)

  **Implémentation** : pour chaque cartouche, créer le rectangle de fond
  via `figma.createRectangle()` (avec la couleur de fond bindée à la
  variable correspondante), puis appeler le helper
  `placeLogo(cartouche, { x: 64, y: 64, w: 352, h: 192, variant: "color"|"white" })`
  défini au préambule de l'Étape 6. Le helper clone les nodes Figma de
  `LOGOS` (détectés sur Page 1 à l'Étape A.1.a). Si `LOGOS.white`
  est `null`, fallback automatique sur `LOGOS.color` avec mention au récap
  final. **Centrer le logo** dans chaque cartouche (padding interne ≥ 64px).

- En bas : **"Taille minimale du logo"** + spec "128px / 35px" + ligne séparatrice

### 08 · Palette de couleurs
- Fond : `neutralLight`
- Colonne gauche : titre **"Palette de couleurs"** 40px + paragraphe règle
  d'usage rédigé en français
- Zone droite (x=520, y=160) : 4 à 6 **colonnes 251×960px cornerRadius 24**
  côte à côte avec gap 16px. Chaque colonne :
  - Fond = la couleur du token (ex `accent`, `primaryAlt`, `primary`, `white`)
  - Nom de la couleur en haut (24px Medium) — **en français** (ex « Primaire »,
    « Accent », « Crème », « Bordeaux »). Le nom de la variable Figma interne
    (`primary`, `accent`, etc.) reste technique mais ce qui s'affiche est le
    libellé français correspondant.
  - Bloc CMYK / HEX / RGB labels (x=24, y=80) — les libellés "CMJN" / "HEX" /
    "RVB" en français (et non "CMYK" / "RGB")
  - Bloc valeurs CMYK / HEX / RGB values (x=81, y=80)
  - Texte blanc sur fond sombre, sombre sur fond clair

### 09 · Nuances
- Fond : `cream` (variable bindée — slide content classique)
- Colonne gauche : titre **"Nuances"** + paragraphe sur l'usage des tints
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

// Pour fokusia : srcHex = "#0A2B32" et "#E73737"
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
- Fond : `primary` (foncé)
- Colonne gauche : titre **"Police"** 40px blanc + paragraphe sur la police
  rédigé en français (ex « Une typographie géométrique moderne, à la fois
  technique et chaleureuse, qui structure l'identité sans la dater. »)
- Zone droite : **"Aa" géant ~480px** en Inter Display Bold blanc, centré
- En bas : nom de la police (ex "Inter Display") + alphabet complet
  (a-z A-Z 0-9) en 28px Medium

### 12 · Hiérarchie typographique
- Fond : `neutralLight`
- Colonne gauche : titre **"Hiérarchie typographique"** + paragraphe court
  rédigé en français
- Zone droite : 5 lignes empilées (H1 56pt, H2 40pt, H3 36pt, H4 28pt,
  Sous-titre 24pt) — chaque ligne contient :
  - Spec gauche en français : **"Titre 1 — Inter Display Regular, 56pt/64pt,
    interlettrage +4"** (16px Medium). Les noms "Titre 1/2/3/4" et "Sous-titre"
    remplacent "Headline 1/2/3/4" et "Subheadline".
  - Phrase d'exemple à la taille H1/H2/H3/H4 réelle, couleur `dark`. **Par
    défaut** : **« Nous ne livrons pas un produit, mais la solution complète. »**
    Si le brand.md contient une formule signature, l'utiliser à la place
    (mais toujours en français).

### 14 · Éléments clés
- Fond : `neutralLight`
- Colonne gauche : titre **"Éléments clés"** + paragraphe rédigé en français
- Zone droite : **6 cards 280×240 cornerRadius 16** en grille 3×2 avec
  shadow douce (`#a8d8f5` ou neutre clair, blur 20, offset 0/8)
- Chaque card : nom de la règle en 20px Bold **en français** (ex « Espacement »,
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
- Fond : `primary`
- **Logo source réel** centré en haut (variante blanche/cream sur fond sombre)
  via `placeLogo(slide15, { x: 800, y: 320, w: 320, h: 120, variant: "white" })`.
  Pas de placeholder texte "Logo".
- Centré : **"Merci"** 120px Inter Display Bold accent (y=560) — et non plus
  "Thank you"
- En bas : handle + version + **"Généré par le skill brand-book"**

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
Inclure `addFooter(slide, {handle, page: i, total: 15})` à la fin de
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
en fin de génération). Toujours faire un cleanup final :
```js
page.children.filter(c => !/^\d{2} ·/.test(c.name)).forEach(c => c.remove());
```
Cela garantit 15 slides exactement, nommées en français (cf. `SLIDE_NAMES_FR`) :
`01 · Couverture` → `15 · Quatrième de couverture`.

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

### 9bis.b — Auto-critique sur 5 axes

Pour chaque slide, vérifier visuellement (le LLM regarde le screenshot
et juge) selon **5 axes** :

| Axe | Question concrète | KO si... |
|---|---|---|
| 1 · **Débordement de texte** | Est-ce qu'un bloc de texte sort de son conteneur ou est tronqué ? | Texte coupé en bord de frame, ellipsis `…` non voulu, ligne qui sort de la safe zone |
| 2 · **Contraste** | Le texte principal est-il lisible sur son fond ? | Ratio < AA (4.5:1 pour body, 3:1 pour large text) — typiquement gris sur gris, accent sur primary clair |
| 3 · **Logo présent & non cassé** | Le logo apparaît-il bien sur slides 01, 06, 15 ? Est-il déformé ou pixelisé ? | Logo absent, étiré, écrasé, mauvaise variante (sombre sur sombre), pixelisation visible |
| 4 · **Footer cohérent** | Footer présent, aligné, avec bon numéro de slide ? | Footer manquant, désaligné, numéro de slide faux ou en double |
| 5 · **Slide vide / inutile** | La slide porte-t-elle réellement une information ? | Slide presque vide (1 titre + 1 fond), placeholder `[À COMPLÉTER]` visible, contenu dupliqué d'une autre slide |

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

## Étape 10 — Confirmer et récapituler

```
✦ Brand Book installé pour [Nom de la marque]

Fichiers générés :
  · brand.md            (source de vérité — tokens + identité verbale + univers)
  · landing-page.md     (structure + copy de landing — template [A|B|C])

Page Figma créée : "✦ Brand Book"
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

Prochaine étape suggérée :
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
compléter par l'utilisateur ». Le brand book Figma générera la slide 04
Universe & Mood avec ses blocs colorés par défaut (pas de moodboard photo).

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
  footer, colonne descriptive, chapter openers)
- **Étape 6** : patron commun pour chaque slide (helpers, footer)
- **Étape 8** : spécifications détaillées par slide (coordonnées x/y,
  sizing, fonts, fills, structure)

La grammaire de la landing page est dans un fichier de référence séparé :
- **`references/landing-page-templates.md`** — 3 templates détaillés
  (Offre/SaaS B2B · Produit · Personal brand) + logique de sélection +
  règles de copy transversales. Lu par l'agent à l'Étape A.5.

Si tu veux modifier l'identité visuelle du brand book Figma (positions,
sizing, couleurs par défaut, tailles de texte), édite directement les
Étapes 4, 6 et 8 de ce SKILL.md. Si tu veux modifier la structure ou la
copy par défaut de la landing, édite `references/landing-page-templates.md`.
