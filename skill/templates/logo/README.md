# /logo/

Déposer ici les deux variantes du logo en SVG vectoriel exportées depuis Figma :

- `logo.svg` — variante couleur originale (pour fonds clairs). **Obligatoire.**
- `logo-white.svg` — variante blanche / cream (pour fonds sombres). **Obligatoire.**

Le skill consomme ces deux fichiers directement comme source de vérité pour les logos affichés dans la landing HTML. Il ne re-colorise jamais un logo programmatiquement.

Si l'un des deux fichiers manque, la génération HTML est bloquée jusqu'au dépôt.
