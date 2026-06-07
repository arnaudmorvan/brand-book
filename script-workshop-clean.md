# Script Workshop — IA × Branding

---

## INTRO

Salut à tous et bienvenue à ce workshop sur l'IA × Figma — comment créer une image de marque facilement avec l'IA. On commence par un rapide recap du contexte, puis on passe directement à la pratique.

Quelques mots sur moi : je suis designer freelance et j'accompagne des startups pour implémenter l'IA dans leurs workflows, notamment sur la partie design — pour augmenter l'efficacité des PO, des product managers et des équipes créatives.

---

## 3 NOTIONS CLÉS

**Le LLM — c'est le cerveau.**
Il prédit les mots suivants. Il sait presque tout… mais il ne connaît pas ta marque. Il a été entraîné sur des données publiques, pas sur tes intentions ni ton univers. C'est à toi de le nourrir.

**La skill — c'est une compétence.**
Un ensemble d'instructions pré-configurées pour que l'IA comprenne un contexte précis. Aujourd'hui, on travaille avec une skill dédiée au branding — Claude va comprendre ta marque, mémoriser son univers et s'adresser à elle de façon cohérente sur tous les supports.

**Le contexte — c'est la différence.**
Même question, 5 contextes différents = 5 résultats très différents. Plus tu donnes de contexte, plus l'IA est précise.

---

## OBJECTIF DU WORKSHOP

On part de votre logo pour créer une image de marque complète, avec un brand book généré directement dans Figma. Deux temps :

1. **Définir l'identité** — Claude pose des questions, vous répondez, vous décidez.
2. **Générer le brand book** — Claude s'appuie sur votre logo et vos réponses pour créer le brand book dans Figma.

Vous repartez avec quelque chose d'utilisable, et vous pouvez reproduire ce process seul·e, pour vous ou pour vos clients.

---

## ÉTAPE 1 — CONFIGURATION

### Connecter Figma à Claude

On va dans **Cowork → Customize → Connecteurs → +**, on tape "Figma", on clique sur le connecteur. Il propose d'installer une extension — faites-le. Une fois installée, on active la connexion. C'est le pont entre Claude et Figma.

### Créer un projet Cowork

**Cowork → Nouveau projet → Utiliser un dossier existant.** Vous sélectionnez votre dossier projet sur votre ordinateur — celui avec votre logo et vos images de marque. Claude va pouvoir accéder à vos assets et interagir avec eux tout au long de la session.

### Télécharger le dossier modèle

Si vous n'avez pas eu le temps de préparer l'arborescence avant la session : on a mis à disposition un dossier complet sur GitHub. Téléchargez le zip, extrayez-le, remplacez les fichiers de logo par les vôtres, et sélectionnez ce dossier dans Cowork.

### Installer la skill brand-book

**Customize → Compétences → +**, glissez-déposez le fichier `.skill` partagé dans le chat. La skill apparaît dans votre liste de compétences — c'est elle qui guide tout le process.

### Récap avant de lancer

- Passez sur le modèle **Claude Opus 4.8** — ça fait une vraie différence sur la qualité des sorties.
- Ouvrez votre fichier Figma avec le **logo couleur** et le **logo blanc** sur la Page 1.

---

## ÉTAPE 2 — LANCER LE BRAND BOOK

On va dans Figma, clic droit sur le fichier, **Copier le lien**. On revient dans Cowork, bien dans notre projet, et on tape :

```
/brand-book
```

Claude demande deux choses : le lien de votre fichier Figma, et un texte de contexte sur votre marque. Collez le texte récapitulatif qu'on a préparé — quelques phrases sur qui vous êtes, votre univers, votre ton. C'est ce qui permet à Claude de comprendre votre marque avant de poser ses questions.

---

## ÉTAPE 3 — RÉPONDRE AUX QUESTIONS

Claude pose trois vagues de questions pour définir votre identité. Prenez le temps d'y répondre — c'est la partie la plus importante. Plus vos réponses sont précises, plus le brand book sera juste.

Exemple : *"Quel est votre ton ?"* — professionnel, informel… Ce choix va adapter toutes les communications de la marque dans le brand book.

Si vous n'avez pas toutes les réponses, répondez avec ce que vous avez. Vous pourrez toujours relancer `/brand-book` plus tard avec des réponses affinées.

---

## ÉTAPE 4 — LE FICHIER BRAND.MD

Claude génère un fichier `brand.md` : votre identité de marque formalisée — positionnement, promesse, différenciateur, identité visuelle. Ce fichier est entièrement modifiable. Reformulez ce qui ne vous semble pas juste.

Deux usages :
- **Pour vous** → éditez directement, c'est votre document de référence.
- **Pour un client** → ce fichier fait partie du livrable. Une identité formalisée en texte, réutilisable sur tous les supports.

---

## ÉTAPE 5 — GÉNÉRER LES IMAGES

Claude vous fournit une liste de **10 prompts prioritaires** pour générer des images cohérentes avec votre identité. Ces prompts sont construits à partir de vos réponses — ils reprennent votre univers visuel, vos valeurs, votre ton.

Dans Figma : **Make → Image**. Collez un prompt, choisissez votre moteur. Testez le même prompt avec deux moteurs pour voir lequel colle le mieux à votre univers.

Une fois les images générées, collez-les dans les placeholders du brand book — **Cmd+R** pour remplacer le placeholder par votre image.

---

## ÉTAPE 6 — LE BRAND BOOK DANS FIGMA

Claude a créé une page "Brand" dans votre fichier Figma. Vous allez trouver :

- **15 à 17 slides** composées à partir de vos réponses
- **Une palette de couleurs** avec des variables Figma — modifiez une variable, tout le brand book se met à jour en cascade
- **Typographie, nuances, modes de contraste**
- **Placeholders** prêts à accueillir vos images

La mise en page n'est pas figée — vous avez toute la souplesse de Figma pour ajuster et affiner.

---

## ÉTAPE 7 — GÉNÉRER LA LANDING PAGE

Installez la deuxième skill : **Frontend Design**. Même process — **Customize → Compétences → +**, glissez le fichier `.skill`.

Ouvrez un **nouveau chat** dans Cowork, restez dans votre projet, et tapez :

```
/frontend-design création de landing page
```

Claude s'appuie sur `brand.md` et `landing-page.md` pour générer une trame HTML complète : positionnement, services, témoignages clients si vous en avez renseigné.

---

## ÉTAPE 8 — AFFINER LA LANDING PAGE

Ouvrez le fichier HTML dans Chrome. Vous avez une première version de votre landing page. Ce qui ne vous convient pas, vous le demandez à Claude directement dans le chat :

*"Remplace le titre de la section Services"*, *"Change l'image hero"*, *"Ajoute un témoignage client"*…

---

## ÉTAPE 9 — METTRE EN LIGNE AVEC NETLIFY

Créez un compte sur [netlify.com](https://netlify.com) si ce n'est pas fait. Ensuite : **glissez-déposez** votre dossier projet dans l'interface Netlify. Votre site est en ligne en quelques secondes, avec une URL publique.

---

## CLÔTURE

Vous avez configuré Claude avec Figma, défini une identité de marque complète, généré un brand book visuel, créé une landing page et mis le tout en production. Merci à tous !

---
