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
- Adaptation tailes pour format Beamer (`\resizebox`, `width=...`)

**Environnements code & debug (cf. template `slides.tex`)**
- Code source C : utiliser `\begin{ccode}` (et `\end{ccode}`)
- Sorties memory/gdb : utiliser `\begin{pwndbg}{...}`
- Conservation couleurs inline (`\textcolor{pwndbgred}{...}`)

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