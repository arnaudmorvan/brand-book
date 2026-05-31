# Landing · fokusia

<!-- TEMPLATE C · Personal brand / Portfolio -->
<!-- Sélectionné car : graphiste freelance solo (Isabelle Marques), portfolio coché, section À propos cochée, vente d'une prestation d'identité visuelle. Hybride avec Template A pour la section méthode/constat. -->

## ✦ Décisions structure (réponses A.5.0)

| Décision | Valeur |
|---|---|
| **Longueur** (L1) | Moyenne |
| **Sections obligatoires** (L2) | Portfolio · 3 projets phares · Témoignages clients · FAQ · À propos |
| **CTA primaire** (L3) | Diagnostic / audit gratuit |
| **Éléments de réassurance** (L4) | Témoignages texte |
| **Lead magnet** (L5) | N/A |

**Sections retenues dans cette landing** :
1. Hero · 2. Portfolio · 3 projets phares · 3. Constat · 4. Méthode · 5. À propos · 6. Témoignages · 7. FAQ · 8. CTA final

**Sections écartées et raison** : Tarifs (règle #0, aucune mention de pricing). Logos clients (non coché en L4). Garantie (non coché en L2, ne colle pas au cycle B2B premium long). Comparaison vs alternatives (non coché en L2, traité indirectement dans Constat et FAQ).

---

## ✦ Décisions hero & visuels (réponses A.5.0.bis)

| Décision | Valeur |
|---|---|
| **Layout du hero** (H1) | Full-bleed (image plein écran + texte en surimpression) |
| **Source image hero** (H2) | À générer via brief créatif Midjourney / Imagen |
| **Chemin image hero** | À déposer dans `./images/hero.jpg` une fois générée |
| **Style d'imagerie global** (H3) | Suivre `brand.md` Q13 (photos duotone teal/crème) |

**Conséquences appliquées** :
Hero full-bleed : la section 1 utilise un background image 100vw × 100vh, texte en surimpression sur dégradé cream→transparent inversé (teal foncé en bas pour contraste). Cadrage vertical contre-plongée, 1/3 inférieur réservé au texte. Image à générer via brief Midjourney 5 champs (cf. section 1). Tous les briefs visuels suivants spécifient le traitement duotone `#0A2B32` × `#F5EFE6` pour cohérence brand book ↔ landing. Portfolio remonté en section 2 (juste après le hero, car hero full-bleed porte déjà le visuel fort, le visiteur arrive et voit le travail immédiatement).

---

## ✦ Décisions portfolio (réponses A.5.0.ter)

| Décision | Valeur |
|---|---|
| **Source des projets** (P1) | placeholders (3 projets-types inventés cohérents avec brand.md) |
| **Layout de présentation** (P2) | Stack vertical (éditorial, immersif) |
| **Profondeur d'info par projet** (P3) | Mockup + titre + brief + livraison |
| **Position section dans la landing** | Section 2 (juste après hero, car hero full-bleed porte le visuel fort) |
| **Dossier `./projects/` détecté** | Non, invention 3 projets-types |

**3 projets-types inventés** :

1. **Cabinet Dérivière & Associés** *(INVENTÉ)* · Investigation corporate · 2025 : refonte d'identité pour un cabinet d'investigation privée qui bascule du marché des particuliers vers le B2B grands comptes.
2. **Maître Lavanchy, avocate cybercriminalité** *(INVENTÉ)* · Avocats · 2025 : identité visuelle d'un cabinet d'avocate spécialisée en cybercriminalité et atteintes à la réputation.
3. **Argentic Security** *(INVENTÉ)* · Cybersécurité B2B · 2024 : repositionnement d'une ESN de cybersécurité pour clients directions juridiques et CSO de grands comptes.

> Pour remplacer ces placeholders, créer `./projects/project-1/cover.jpg` (et `details.md` optionnel) puis relancer le skill en mode LANDING.

---

## ✦ Prompt, à copier-coller tel quel

> **Rôle** : Vous êtes **UX/UI designer senior spécialisé dans les landing pages à fort taux de conversion**. Vous avez designé plus de 200 landings pour des SaaS B2B, des marques DTC et des freelances premium. Vous maîtrisez la grammaire des hero qui accrochent en 3 secondes, les sections qui répondent aux vraies objections, les CTAs qui convertissent, et la hiérarchie visuelle qui guide l'œil sans effort. Vous connaissez Framer, Webflow, Tailwind, shadcn/ui, et vous savez quand un layout sent l'IA générique et comment l'éviter.
>
> **Mission** : Générez la landing page complète de **fokusia** (Isabelle Marques, graphiste indépendante spécialisée des secteurs où la discrétion fait l'autorité) à partir du brief ci-dessous. La copy est **déjà rédigée et validée**. Vous ne devez **pas la réécrire**, juste la mettre en forme. Votre job c'est le design : layout, hiérarchie typo, composition, micro-interactions, qualité visuelle premium, cohérence avec l'univers de marque décrit plus bas.
>
> **Contraintes non négociables** :
> - Respectez la copy à la lettre. Pas de paraphrase, pas d'ajout, pas de suppression. Chaque H1, H2, bullet, CTA et FAQ a été calibré sur le persona (dirigeant de cabinet d'investigation qui veut basculer corporate).
> - Respectez les briefs visuels de chaque section (sujet incarné, composition, lumière, palette, références culturelles). Aucun stock photo générique, aucun gradient abstrait par défaut, aucun blob coloré.
> - Logos : utilisez les fichiers fournis dans `./logo/Logo.svg` (variante couleur) et `./logo/Logo white.svg` (variante blanche). Ne re-colorez jamais ces fichiers programmatiquement.
> - Images : utilisez les chemins indiqués dans chaque section (`./images/...`). Si un chemin n'existe pas encore, affichez un placeholder visible avec le nom du fichier attendu. Ne remplacez jamais par des stock photos, ne générez pas d'illustration IA pour les chemins déjà référencés.
> - Respectez le ton et la personnalité graphique : crème dominante (≥70%), teal `#0A2B32` pour structure, accent rouge `#E73737` strictement chirurgical (CTAs uniquement, accent line, numéros). Codes éditorial luxe, marges généreuses, typographie Playfair Display + Inter.
> - Aucune mention de pricing sur la landing. La tarification est gérée hors landing (RDV de qualification).
> - Hero impactant dès le scroll 0 (pas de loader, pas d'animation qui retarde le H1). Mobile-first. Accessibilité AA minimum.
> - Quand un placeholder `[à compléter]` apparaît (témoignages, projets, chiffres), gardez-le visible. Ne l'inventez pas, ne hallucinez aucun nom client réel.
>
> **Format de sortie attendu** : à préciser par l'utilisateur au moment du copier-coller (HTML+Tailwind / React+shadcn / Framer sections / Webflow components).
>
> Maintenant, lisez tout le brief ci-dessous et générez la landing.

---

> Ton dominant : Confidentiel & premium · Style accroches : Affirmation tranchée · Style CTAs : Phrase complète
> Formule signature placée 2 fois : variation courte en hero + intégrale en CTA final.

---

## 1 · Hero

**Layout** : full-bleed. Background image 100vw × 100vh. Texte centré horizontalement, en bas du viewport (1/3 inférieur). Dégradé linéaire teal `#0A2B32` opacité 0 à 70% du haut vers le bas pour garantir le contraste du texte cream. Logo blanc en haut à gauche (1/2 hauteur du header).

**Brief visuel hero (à générer Midjourney / Imagen)** :
- **Sujet incarné** : intérieur d'un cabinet juridique parisien feutré, étagères de livres reliés en arrière-plan flou, lumière indirecte de fin d'après-midi rasante sur un bureau ancien en bois sombre. Aucun visage, aucun corps frontal. Suggestion : main posée sur un dossier en cuir bordeaux, ou simple présence d'un sous-main crème avec un stylo plume posé.
- **Composition** : cadrage vertical 9:16, contre-plongée légère, profondeur de champ très courte (sujet net au centre, arrière-plan complètement flou). 1/3 supérieur : atmosphère lumineuse de bibliothèque. 1/3 central : sujet. 1/3 inférieur : zone d'ombre douce réservée à la surimpression texte.
- **Lumière** : indirecte, rasante, chaude. Tonalité fin d'après-midi (golden hour intérieure). Pas de néon, pas de lumière clinique.
- **Palette** : duotone teal `#0A2B32` × crème `#F5EFE6`. Conserver une touche de bordeaux profond `#5C1414` sur l'élément central (cuir, reliure, encre).
- **Références culturelles** : photographies intérieures de Massimo Listri (bibliothèques institutionnelles), François Halard pour Architectural Digest, séries éditoriales du M Le Monde sur les cabinets juridiques parisiens. À éviter : tout ce qui évoque l'imagerie d'espionnage cliché, les loupes, les empreintes, les nuits orageuses.

À déposer dans `./images/hero.jpg` une fois générée.

**Eyebrow** *(12px Inter Bold uppercase, tracking +3, accent rouge)* :
GRAPHISTE · INVESTIGATION · AVOCATS · CYBERSÉCURITÉ

**H1** *(Playfair Display Bold 56-72px, cream)*, **VARIANTE RECOMMANDÉE**, version condensée de la signature :

Rigueur. Méthode. Autorité.

**Sous-titre** *(Inter Regular 20-22px, cream à 85% opacité)* :

Identité visuelle pour les cabinets d'investigation, les avocats en cybercriminalité et les entreprises de cybersécurité qui veulent décrocher les contrats corporate.

**CTA principal** *(bouton accent rouge `#E73737` sur cream, label cream, 16px Inter Semi Bold)* :
Réservez votre diagnostic d'image

**Lien CTA secondaire** *(underline accent, 14px Inter Medium)* :
Voir les projets récents →

---

## 2 · Portfolio · 3 projets phares (selected work)

**Layout** : stack vertical éditorial. 1 projet par scroll, full-width. Alternance gauche / droite : projet 1 image-gauche / texte-droite, projet 2 image-droite / texte-gauche, projet 3 image-gauche / texte-droite. Fond `cream`. Marges latérales 10% mini. Espacement entre projets : 2xl (96-128px).

**Eyebrow section** *(uppercase, tracking +3, accent)* : SÉLECTION DE TRAVAUX

**H2 section** *(Playfair Display Bold 40px, primary)* :
Trois projets, trois bascules vers le corporate.

---

### Projet 1

**Brief visuel projet 1** :
- Mockup éditorial sobre : carte de visite + papeterie posées sur un sous-main crème, lumière rasante. Si possible, brand book ouvert montrant deux pages-spread.
- Traitement duotone teal `#0A2B32` × crème `#F5EFE6`. Aucune marque visible autre que la maquette présentée.
- Référence : mockups éditoriaux de Fanny Coulez, Atelier Carvalho Bernau, Trafik Studio.
- À générer ou photographier. Déposer dans `./images/projects/project-1.jpg`.

**Eyebrow projet** *(12px accent uppercase)* : 01 · CABINET D'INVESTIGATION CORPORATE · 2025

**H3 projet** *(Playfair Display Semi Bold 28-32px)* :
*(INVENTÉ, à remplacer)* Cabinet Dérivière & Associés.

**Brief original** *(Inter Regular 18px, lh 1.6, body)* :
Cabinet d'investigation parisien historique, vingt ans d'expérience sur le marché des particuliers (recherche, vérification, infidélité). Volonté de basculer 70% du chiffre vers le corporate (DRH, direction juridique, cabinets d'avocats d'affaires) en deux ans. Identité visuelle existante datée, codes années 90, illisible pour un acheteur grand compte.

**Ce qui a été livré** *(Inter Regular 18px)* :
Refonte complète du logotype, palette éditoriale crème / teal / bordeaux, charte typographique Playfair Display + Inter, brand book de 32 pages, papeterie corporate (papier en-tête, carte de visite, dossier client), modèle de proposition commerciale et template de rapport d'investigation. Premier contrat grand compte signé deux mois après le lancement.

---

### Projet 2

**Brief visuel projet 2** :
- Mockup d'identité d'avocate : monogramme en pied de mail, signature email, site web mobile en haut de page. Sobriété maximale.
- Traitement duotone, palette restreinte teal / crème, accent ponctuel rouge sur un seul élément (point de signature, surligneur).
- Référence : identités d'avocats récentes de Mucca Design (NY), Studio Frith, Foreign Policy Design.
- À générer ou photographier. Déposer dans `./images/projects/project-2.jpg`.

**Eyebrow projet** *(12px accent uppercase)* : 02 · AVOCATE CYBERCRIMINALITÉ · 2025

**H3 projet** *(Playfair Display Semi Bold 28-32px)* :
*(INVENTÉ, à remplacer)* Maître Lavanchy, cabinet en cybercriminalité.

**Brief original** *(Inter Regular 18px)* :
Avocate indépendante spécialisée en cybercriminalité, atteintes à la e-reputation et fraude numérique. Clientèle visée : dirigeants victimes d'usurpation, entreprises post-incident, particuliers haut patrimoine. Besoin d'une identité qui inspire confiance immédiate et discrétion totale.

**Ce qui a été livré** *(Inter Regular 18px)* :
Monogramme épuré, charte couleur à deux teintes plus accent chirurgical, mise en page systématique des correspondances (mémos, conclusions, factures d'honoraires), site one-pager statique aux codes éditoriaux. Première recommandation d'un cabinet d'avocats associé survenue dans le trimestre.

---

### Projet 3

**Brief visuel projet 3** :
- Mockup d'écran MacBook montrant une slide de pitch corporate (cybersécurité B2B) + carte de visite en premier plan.
- Traitement duotone, codes plus tech (grille subtile visible sur le mockup) mais toujours dans le registre éditorial luxe.
- Référence : decks récents de cabinets de conseil en cybersécurité haut de gamme (Wiz, Sygnia branding).
- À générer ou photographier. Déposer dans `./images/projects/project-3.jpg`.

**Eyebrow projet** *(12px accent uppercase)* : 03 · ESN CYBERSÉCURITÉ B2B · 2024

**H3 projet** *(Playfair Display Semi Bold 28-32px)* :
*(INVENTÉ, à remplacer)* Argentic Security, conseil cybersécurité grand compte.

**Brief original** *(Inter Regular 18px)* :
ESN de cybersécurité, 12 consultants senior, cible exclusive : CSO et directions juridiques de grands comptes français. Identité initiale techno-générique (bleu vif, dégradés, icônes data) qui les classait dans la même catégorie que des outils SaaS. Volonté d'un repositionnement « cabinet de conseil » et non « éditeur ».

**Ce qui a été livré** *(Inter Regular 18px)* :
Refonte logotype, palette monochrome teal + accent rouge ponctuel, système de slides corporate complet, template de rapport d'audit, charte LinkedIn pour les consultants. Le repositionnement a permis d'aligner le positionnement commercial sur celui d'un cabinet de conseil traditionnel.

---

## 3 · Constat

**Brief visuel constat** :
- Détail rapproché d'une carte de visite datée (police années 90, mise en page surchargée) posée à côté d'une carte sobre contemporaine, sur fond crème. Composition diptyque.
- Traitement duotone teal / crème, pas de couleur agressive.
- À générer ou photographier. Déposer dans `./images/constat-cards.jpg`.

**Layout** : section 2 colonnes. À gauche : visuel diptyque. À droite : H2 + paragraphe + 3 bullets. Fond `cream`.

**Eyebrow** *(uppercase, tracking +3, accent)* : CE QUE NOUS OBSERVONS

**H2** *(Playfair Display Bold 40px, primary)* :
Votre expertise est en béton. Votre image, non.

**Paragraphe** *(Inter Regular 20px, lh 1.6, body)* :
La plupart des cabinets d'investigation et des cabinets d'avocats spécialisés en cybercriminalité que nous rencontrons partagent le même constat. Leurs interlocuteurs corporate les jugent en moins de quinze secondes. Sur leur site, sur leur carte de visite, sur leur première slide. La lecture est faite avant même qu'ils aient ouvert la bouche.

**Bullets** *(3 items, icône outline 1.5 stroke `#0A2B32`, label Inter Medium 18px, description Inter Regular 16px)* :

- **L'image dit « particulier ».** Codes années 90, logo générique, papeterie sans hiérarchie. Le DRH ferme l'onglet avant le premier rendez-vous.
- **L'image dit « caricature ».** Loupes, ombres, typographie noire et rouge agressive. Représentation littérale qui empêche d'être pris au sérieux par une direction juridique.
- **L'image dit « technique seulement ».** Bleu vif, dégradés, icônes data. La cybersécurité confondue avec un éditeur SaaS, le positionnement commercial aligné en conséquence.

---

## 4 · Méthode

**Brief visuel méthode** :
- Photographie d'un sous-main de bureau avec trois feuilles A4 imprimées posées en arc de cercle, chaque feuille une étape de la méthode (1, 2, 3). Stylo plume au centre. Lumière rasante.
- Traitement duotone teal / crème.
- À générer ou photographier. Déposer dans `./images/methode-diptyque.jpg`.

**Layout** : section 3 colonnes en grille, fond `creamLight`. En-tête de section centré au-dessus. Cartes des 3 étapes avec radius `md:8` (sobre, Premium).

**Eyebrow** *(uppercase, tracking +3, accent)* : MÉTHODE FOKUSIA

**H2** *(Playfair Display Bold 40px, primary, centré)* :
Trois étapes pour passer du marché des particuliers à la lecture corporate.

**Paragraphe** *(Inter Regular 18px, lh 1.6, body, centré, max-width 720px)* :
La méthode fokusia tient sur trois mouvements. Aucun jargon, aucune slide superflue. Vous savez à chaque étape ce qui sort, ce qui rentre, et ce que vous validez.

### Étape 01 · Lecture stratégique

**Numéro** *(96px Playfair Display Bold accent)* : 01

**H3** *(Playfair Display Semi Bold 24-28px, primary)* :
Lecture stratégique de votre cabinet.

**Description** *(Inter Regular 16-18px, lh 1.6)* :
Audit de votre image actuelle vue par un acheteur corporate. Lecture des trois points de contact prioritaires : site, carte de visite, première slide de proposition. Diagnostic restitué en un document de quatre pages avec décisions claires.

### Étape 02 · Construction de l'autorité visuelle

**Numéro** *(96px)* : 02

**H3** :
Construction de votre identité corporate.

**Description** :
Logotype, palette, typographie, principes de mise en page. Production du brand book de référence et de la papeterie corporate (papier en-tête, carte, dossier, signature email). Une seule itération validée, pas d'allers-retours en boucle.

### Étape 03 · Activation sur vos points de contact

**Numéro** *(96px)* : 03

**H3** :
Activation sur vos points de contact commerciaux.

**Description** :
Application sur votre site, votre modèle de proposition commerciale, votre template de rapport. Vous repartez avec une image cohérente sur tous les supports que voient vos prospects corporate.

---

## 5 · À propos

**Brief visuel à propos** :
- Portrait éditorial d'Isabelle (ou main travaillant un document, si pas de photo disponible). Lumière naturelle indirecte, tonalité crème chaude, profondeur de champ courte. Aucune posture marketing souriante : registre éditorial sobre, regard hors champ ou attention au document.
- Traitement duotone teal / crème.
- À déposer dans `./images/about-portrait.jpg`. Si pas disponible : placeholder avec brief visuel intégral.

**Layout** : section 2 colonnes, fond `cream`. À gauche : portrait portrait 4:5. À droite : eyebrow + H2 + 2-3 paragraphes + 3-4 bullets sur la posture.

**Eyebrow** *(uppercase, tracking +3, accent)* : À PROPOS D'ISABELLE

**H2** *(Playfair Display Bold 40px, primary)* :
Une graphiste qui comprend les codes de la discrétion.

**Paragraphes** *(Inter Regular 18-20px, lh 1.6)* :

Isabelle Marques accompagne depuis [N] années les professionnels qui exercent dans les secteurs où l'image se construit par soustraction. Investigation privée corporate, avocats en cybercriminalité, conseil en cybersécurité. Trois métiers où montrer trop tue le contrat, où la rigueur visuelle est la première preuve de la rigueur méthodologique.

Sa posture n'est pas celle d'un studio créatif généraliste qui découvre votre secteur sur votre dossier. C'est celle d'une partenaire stratégique qui connaît déjà les contraintes du secret professionnel, la grammaire visuelle des cabinets d'avocats d'affaires, et les codes de lecture des directions juridiques.

**Bullets posture** *(icône outline 1.5 stroke, label Inter Medium 18px, description Inter Regular 16px)* :

- **Spécialiste sectorielle, pas généraliste.** Les codes de l'investigation, du juridique et de la cybersécurité ne s'improvisent pas projet par projet.
- **Méthode et cadre, pas créativité débridée.** Vous savez ce qui sort à chaque étape, ce qui se valide, ce qui ne se rediscute pas.
- **Cycle court, pas accompagnement sans fin.** L'objectif n'est pas de faire vivre un chantier, c'est de vous remettre une image que vous oubliez ensuite.

---

## 6 · Témoignages

**Brief visuel témoignages** : pas de visuel principal. Section purement typographique sur fond `cream`. Citations en grand Playfair Display, attribution en Inter Medium plus petite.

**Layout** : section centrée, max-width 1100px, 3 témoignages empilés verticalement avec séparateurs accent line. Eyebrow + H2 en en-tête. Marges xl (40-64px) entre témoignages.

**Eyebrow** *(uppercase, tracking +3, accent)* : RETOURS DE CLIENTS

**H2** *(Playfair Display Bold 40px, primary, centré)* :
Ce que les cabinets accompagnés disent de la collaboration.

### Témoignage 1

**Citation** *(Playfair Display Italic 28-32px, primary, lh 1.4)* :
« [À COMPLÉTER, verbatim client : 3 ou 4 lignes sur la transformation perçue de l'image, le résultat business obtenu, ou la posture d'Isabelle. Anonymiser si secret professionnel.] »

**Attribution** *(Inter Medium 14px, accent)* :
Directeur de cabinet d'investigation, Paris · 2025

### Témoignage 2

**Citation** :
« [À COMPLÉTER, verbatim avocat ou direction de cabinet juridique.] »

**Attribution** :
Avocate associée, Lyon · 2025

### Témoignage 3

**Citation** :
« [À COMPLÉTER, verbatim dirigeant ESN cybersécurité.] »

**Attribution** :
Dirigeant ESN cybersécurité, Région parisienne · 2024

---

## 7 · FAQ

**Layout** : section centrée, max-width 880px, fond `creamLight`. Accordions sobres (radius `sm:4`, séparateurs `border` discrets, plus + ou flèche outline 1.5 stroke à droite). Eyebrow + H2 en en-tête.

**Eyebrow** *(uppercase, tracking +3, accent)* : QUESTIONS FRÉQUENTES

**H2** *(Playfair Display Bold 40px, primary, centré)* :
Ce que demandent les dirigeants avant de réserver un diagnostic.

### Question 1 *(Objection #1 du brand.md, verbatim prospect)*

**Q** *(Inter Semi Bold 20px)* :
J'ai pas le temps de m'en occuper. C'est combien de mois de mobilisation ?

**R** *(Inter Regular 18px, lh 1.6)* :
La méthode fokusia est calibrée précisément pour les dirigeants solo qui sont sur le terrain. Le diagnostic d'image se restitue en 45 minutes. La construction de l'identité corporate demande deux rendez-vous d'une heure de votre côté, étalés sur trois à quatre semaines, pas davantage. Les allers-retours créatifs en boucle ne font pas partie de la méthode : une seule itération est validée, et on avance.

### Question 2

**Q** :
Vous comprenez vraiment notre secteur, ou c'est un argument marketing ?

**R** :
fokusia accompagne exclusivement trois secteurs : l'investigation privée corporate, les cabinets d'avocats en cybercriminalité, et les entreprises de cybersécurité. Les autres demandes sont systématiquement déclinées. La spécialisation se traduit concrètement par la connaissance des codes de discrétion, la lecture corporate attendue par les DRH et les directions juridiques, et la culture du secret professionnel qui n'est jamais transgressée dans les visuels ou les supports.

### Question 3

**Q** :
Mon image actuelle a coûté cher. Pourquoi je devrais tout refaire ?

**R** :
Le diagnostic d'image initial répond précisément à cette question. Dans certains cas, une refonte complète n'est pas nécessaire : trois ou quatre supports stratégiques suffisent à corriger la lecture corporate. Dans d'autres, la refonte est inévitable parce que l'identité actuelle bloque les rendez-vous grands comptes. La décision se prend après le diagnostic, jamais avant.

### Question 4

**Q** :
Vous travaillez avec des cabinets concurrents directs ? Quelle confidentialité ?

**R** :
La règle est simple. fokusia ne travaille jamais avec deux cabinets directement concurrents sur le même marché géographique et le même segment. Chaque mission fait l'objet d'un engagement écrit de non-concurrence pour la durée de la collaboration et les douze mois suivants. La liste des cabinets accompagnés n'est communiquée à personne, sauf accord explicite.

### Question 5

**Q** :
Vous livrez seule ou en équipe ?

**R** :
La méthode est pilotée et exécutée par Isabelle Marques en direct. Vous échangez avec une seule interlocutrice du diagnostic à la livraison. Pour les productions techniques annexes (impression, développement web ponctuel), des partenaires spécialisés interviennent sous brief précis, jamais sur la stratégie d'image elle-même.

---

## 8 · CTA final

**Brief visuel CTA final** :
- Fond pleine couleur `primaryDark` (`#061A1F`). Texte cream. Logo blanc centré au-dessus du titre. Pas d'image, pas de décoration : section purement typographique.

**Layout** : section centrée, padding 2xl (96-128px), fond `primaryDark`. Logo blanc en haut centré (1/3 de hauteur). H1 manifeste reprenant la formule signature intégrale. Sous-titre + CTA principal. Section finale avant le footer.

**Eyebrow** *(uppercase, tracking +3, accent)* : DIAGNOSTIC D'IMAGE

**H1 manifeste** *(Playfair Display Bold 56-72px, cream, centré, max-width 1100px)* :
Je traduis la rigueur de l'investigation en autorité visuelle.

**Sous-titre** *(Inter Regular 22px, lh 1.6, cream à 85%, max-width 720px, centré)* :
Quarante-cinq minutes en visio. Un diagnostic restitué par écrit. Aucune obligation de poursuivre. Vous repartez avec une lecture précise de votre image vue par un acheteur corporate.

**CTA principal** *(bouton accent `#E73737`, label cream, Inter Semi Bold 18px, padding xl, radius `md:8`)* :
Réservez votre diagnostic d'image

**Lien secondaire** *(underline cream à 70%, Inter Medium 14px, en dessous du CTA principal)* :
Ou écrivez à isabelle@fokusia.fr →

---

## Footer

**Layout** : section finale, padding xl, fond `primaryDark` (continuité du CTA final), border-top accent line 1px largeur 80px centrée.

**Structure 3 colonnes** *(Inter Regular 14px, cream à 70%)* :

- **Colonne 1** : Logo blanc petite taille + tagline courte *(Inter Regular 14px, cream à 70%)* : « Identité visuelle pour cabinets d'investigation, avocats et cybersécurité. »
- **Colonne 2** : Contact, isabelle@fokusia.fr · LinkedIn · Paris.
- **Colonne 3** : Mentions légales · Politique de confidentialité · © 2026 fokusia. Tous droits réservés.

---

## Variantes hero (A/B test)

> 2 alternatives au hero principal de la section 1, à tester en post-launch.

### Variante B · Angle question ouverte

**H1** *(Playfair Display Bold 56-72px, cream)* :
Et si votre image valait
vos contrats corporate ?

**Sous-titre** *(Inter Regular 20-22px, cream à 85%)* :
La graphiste qui accompagne les cabinets d'investigation, les avocats en cybercriminalité et les entreprises de cybersécurité dans leur bascule vers le B2B grand compte.

**Pourquoi tester** : engage par une projection directe sur la conséquence business. Fonctionne sur les dirigeants qui ont déjà raté un appel d'offres récemment.

### Variante C · Angle affirmation longue

**H1** *(Playfair Display Bold 56-72px, cream)* :
La rigueur de votre méthode
mérite la même lecture visuelle.

**Sous-titre** *(Inter Regular 20-22px, cream à 85%)* :
Identité visuelle pour les professionnels de la discrétion qui veulent décrocher les contrats corporate. Investigation. Avocats. Cybersécurité.

**Pourquoi tester** : registre affirmation longue plus narratif que la version recommandée (trois mots). Pour audience qui scroll lentement, lit avant de cliquer.

---

## Notes de production

- **Mots à utiliser dans toute la page** : rigueur, autorité, méthode, discrétion, codes, institution, crédibilité, corporate, structure, professionnalisme, sobriété, précision, réassurance, posture, lecture, cadre. Tous présents dans cette landing.
- **Mots interdits** : se référer à la section « Mots à éviter » de `brand.md` (buzzwords corporate, familiarités forcées, vocabulaire cliché du métier et termes anxiogènes).
- **Formule signature** : exactement 2 occurrences. Variation condensée en hero section 1 (« Rigueur. Méthode. Autorité. »). Intégrale en CTA final section 8.
- **Ambiance visuelle** : crème & chaleureux (≥70%), teal `#0A2B32` pour la structure, accent rouge `#E73737` chirurgical (CTAs uniquement, accent line, numéros). Codes éditorial luxe.
- **Univers de référence** : éditorial luxe & mode (M Le Monde, T Magazine, cabinets juridiques rue de la Loi, banques privées genevoises).
- **Icônes & illustrations** : librairie Phosphor light ou Lucide (à figer), stroke 1.5 px, jamais en accent rouge sauf action critique. Aucune illustration vectorielle décorative.
- **Aucune mention de pricing** sur cette landing. Tarification gérée ailleurs (RDV de qualification).
- **Données à compléter par l'utilisateur** :
  - [ ] 3 témoignages clients verbatim (section 6, anonymisés si secret professionnel)
  - [ ] 3 visuels mockup pour les projets phares (section 2, sauf si remplacement des projets-types par vrais projets)
  - [ ] Hero image générée via brief Midjourney (section 1) à déposer dans `./images/hero.jpg`
  - [ ] Visuel constat diptyque (section 3) dans `./images/constat-cards.jpg`
  - [ ] Visuel méthode (section 4) dans `./images/methode-diptyque.jpg`
  - [ ] Portrait Isabelle ou main travaillant (section 5) dans `./images/about-portrait.jpg`
  - [ ] Ancienneté à insérer dans la section À propos (« depuis [N] années »)
  - [ ] Email de contact à confirmer (isabelle@fokusia.fr utilisé par défaut)
  - [ ] Lien LinkedIn à insérer dans le footer
