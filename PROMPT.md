# Instructions Génération `slides.tex`

**Source de données stricte**
- `main.tex` 
- Fichiers inclus (`sections/*.tex`)
- Exclusion autres sources

**Contraintes de rédaction**
- Style télégraphique obligatoire
- Zéro phrase complète
- Listes à puces systématiques (`\begin{itemize}`)
- Mots-clés et concepts isolés

**Réutilisation ressources (Obligatoire)**
- Extraction diagrammes `\begin{tikzpicture}` depuis rapport
- Intégration images `\includegraphics` (dossier `images/`)
- Maintien balises mathématiques (`$ ... $`)
- **Échappement LaTeX** : S'assurer que les caractères spéciaux (notamment le souligné `_`) sont correctement échappés dans le texte (`\_`) pour éviter les erreurs de compilation hors des environnements `minted` ou `verbatim`.
- Adaptation tailes pour format Beamer (`\resizebox`, `width=...`)

**Environnements code & debug (cf. template `slides.tex`)**
- Code source C : utiliser `\begin{ccode}` (et `\end{ccode}`)
- Sorties memory/gdb : utiliser `\begin{pwndbg}{...}`
- Conservation couleurs inline (`\textcolor{pwndbgred}{...}`)
- **Priorité Démo/Code** : Intégrer systématiquement des extraits de code (`ccode`) ou des sorties de debug (`pwndbg`) pour illustrer chaque primitive technique. Préférer le code concret aux longues explications textuelles.

**Mapping Sections -> Slides**
- **Introduction** : Synthèse `01_rappels.tex` / `02_malloc.tex`
- **Exploitation (Primitives)** : Synthèse `04_exploitation*.tex`
- **Mitigations (ASLR / GLIBC)** : Extractions sections de défense / conclusion

**Génération de notes d'accompagnement (Obligatoire)**
- Pour chaque slide générée, mettre à jour `NOTES_ORAL.md`.
- **Contenu des notes** :
    - Explication orale détaillée du concept technique (phrases complètes).
    - Section "Questions pièges du jury" : anticiper les interrogations sur les détails d'implémentation (ex: pourquoi `calloc`, spécificités des versions GLIBC, calculs d'offsets).
    - Définition des termes complexes non détaillés sur la slide.
- **Format** : Markdown structuré avec titres correspondant aux titres des slides.

**Gestion des espaces (Overfull \vbox)**
- Aucun débordement vertical toléré (`Overfull \vbox`).
- Si le contenu est dense, utiliser l'environnement `columns` (ex. `0.48\textwidth`) pour le texte en haut de la slide.
- Garder le code (`ccode`) et les traces de debug (`pwndbg`) en format plein format horizontal (full width), placés en dessous du texte ou sur une slide séparée, pour éviter les retours à la ligne illisibles. Ne pas mettre le code et les traces dans des colonnes étroites.
- Si cela cause un dépassement vertical, scinder systématiquement sur plusieurs slides (ex: `Concept - Code (1/2)`, `Concept - Traces (2/2)`).
- Ajuster dynamiquement les espacements internes (`\vspace{-0.2em}`, tailles de police, formatage horizontal dans pwndbg) pour tout compacter sans générer d'erreurs de compilation LaTeX.
- Ne pas utiliser le terme 'aliasing' ou ses dérivés.
- **Mise en relief visuelle** : Utiliser des rectangles rouges fins pour encadrer les sections de code ou de dump mémoire critiques (ex: `@\makebox[0pt][l]{\fboxsep=0pt\color{red}\fbox{\phantom{TEXT}}}TEXT@`). Ne pas utiliser de fond coloré (`colorbox`) pour préserver l'alignement. L'utiliser avec parcimonie pour attirer l'œil sur le point critique de la slide (ex: offset modifié, fonction détournée).

**Exceptions de dictionnaire (English Dictionary Exception)**
- Conserver certains termes techniques spécifiques en anglais tels que `fake_size` et `dangling pointer`. Ne pas les traduire en français.

