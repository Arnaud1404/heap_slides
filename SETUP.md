/prompt [LaTeX Architect: Beamer Structural Outline]

OBJECTIF : Générer uniquement la structure LaTeX Beamer (Outline) d'une présentation technique de 20 minutes basée sur le rapport de TER fourni (main.pdf et main.tex).

STRICTES CONTRAINTES :

    Zéro Contenu : Ne pas rédiger le corps des slides. Générer uniquement les titres des sections et des frames.

    Packages : Inclure un préambule complet avec tous les packages nécessaires pour supporter le code C (minted), les analyses de mémoire (tcolorbox), les schémas (tikz) et le thème metropolis.

    Logique : Appliquer la règle "1 Vulnérabilité = 1 Primitive".

CONFIGURATION DES PACKAGES (Préambule) :

    \documentclass[10pt]{beamer}

    \usetheme{metropolis}

    \usepackage[utf8]{inputenc}

    \usepackage[T1]{fontenc}

    \usepackage{minted} (pour les PoC C)

    \usepackage{tcolorbox} avec la library breakable (pour les dumps pwndbg)

    \usepackage{tikz} (pour les schémas de tas)

    \usepackage{amsmath, amssymb} (pour les formules de Safe Linking)

OUTLINE DES SLIDES (Titres uniquement) :

    I. Introduction et Fondamentaux (4 min)

        Slide 1 : Titre et Objectif (Exploitation de la Heap en 2026).

        Slide 2 : L'arène de jeu ptmalloc2 (Gestion via \texttt{brk} et \texttt{mmap}) .

        Slide 3 : Anatomie structurelle d'un Chunk (Métadonnées et état libre) .

    II. Le Cœur : 1 Type d'Exploit = 1 Primitive (12 min)

        Slide 4 : Use-After-Free (UAF) → Arbitrary Write (Tcache Poisoning & Safe Linking) .

        Slide 5 : Double Free → Memory Aliasing (Fastbin Dup et boucles de bin) .

        Slide 6 : Heap Overflow → Structural Corruption (Overlapping Chunks et fusion forcée) .

        Slide 7 : Corruption de Largebin → Arbitrary Pointer Write (Largebin Attack sur \texttt{bk_nextsize}) .

    III. Défenses et Perspectives (4 min)

        Slide 8 : La barrière de l'ASLR (Randomisation vs Arbitrary Read) .

        Slide 9 : Évolutions GLIBC 2.42 & 2.43 (Durcissement et suppression des Fastbins) .

        Slide 10 : Conclusion (Pivot vers la \textit{Type Confusion} applicative) .