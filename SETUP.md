/prompt [LaTeX Architect: Beamer Structural Outline]

OBJECTIF : Générer uniquement la structure LaTeX Beamer (Outline) d'une présentation technique de 20 minutes basée sur le rapport de TER fourni, en utilisant exclusivement main.tex et les fichiers du dossier sections comme sources.

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

        Slide 1 : Objectif.

        Slide 2 : ptmalloc2.

        Slide 3 : Chunk.

    II. Le Cœur : 1 Type d'Exploit = 1 Primitive (12 min)

        Slide 4 : UAF.

        Slide 5 : Double Free.

        Slide 6 : Heap Overflow.

        Slide 7 : Largebin.

    III. Défenses et Perspectives (4 min)

        Slide 8 : ASLR.

        Slide 9 : GLIBC 2.42 / 2.43.

        Slide 10 : Conclusion.