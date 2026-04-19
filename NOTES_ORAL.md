# Notes pour l'Oral - Exploitation Heap (2026)

Ce fichier regroupe les explications techniques complètes et les réponses aux questions potentielles du jury.

## Slide 4 : Use-After-Free (UAF) & Safe Linking

**Explication orale :**
* *Qu'est-ce qu'un UAF ?* C'est quand un programme libère un chunk de mémoire (`free`) mais continue d'utiliser le pointeur.
* *Tcache Poisoning :* On utilise l'UAF pour écraser le pointeur `fd` du chunk libéré, qui pointe vers le prochain bloc libre. En le remplaçant par l'adresse de notre choix (ex: sur la pile), on force le prochain appel `malloc()` à allouer à cet endroit.

**Questions pièges du jury :**
* *Q : Pourquoi le Tcache est-il si vulnérable ?*
  * **R :** Parce qu'il a été conçu pour la vitesse absolue (LIFO). Contrairement aux autres bacs (bins), il ne possédait historiquement presque aucune vérification d'intégrité de ses métadonnées. L'ajout du Safe Linking a tenté d'atténuer cela.
* *Q : Expliquez comment la formule du Safe Linking protège le système.*
  * **R :** La formule `C = (L >> 12) XOR P` masque le vrai pointeur `P`. Si l'attaquant écrit bêtement son adresse cible `P`, l'allocateur lira `C` comme un pointer, ce qui fera planter le programme (SIGSEGV).
* *Q : Comment inversez-vous l'algorithme (XOR-shift) sans clé ?*
  * **R :** L'adresse `L` et le pointeur `P` partagent souvent le même préfixe (même block mmap). Avec le décalage `>> 12`, les 12 bits de poids fort de `C` sont identiques à `P`. On XOR cette partie connue en cascade pour reconstruire le pointeur entier. Mais il faut un pointer fuité (heap leak) de base.

## Slide 5 : Double Free

**Explication orale :**
* *Le Bug Fastbin Dup :* Effectuer un *double free* directement (`free(A); free(A);`) plante le programme. Mais la vérification LIFO des Fastbins est très faible : elle ne vérifie que le *dernier* élément rentré. En intercalant une libération (`free(A); free(B); free(A);`), on passe la sécurité.
* *Conséquence (Aliasing) :* La liste devient circulaire. On obtient deux pointeurs valides (ex: `c` et `e`) qui modifient la même zone mémoire en même temps.

**Questions pièges du jury :**
* *Q : Pourquoi utiliser `calloc` plutôt que `malloc` dans votre exemple de code ?*
  * **R :** C'est indispensable pour l'exploit dans ce contexte ! `calloc` alloue de la mémoire et la met à zéro. Contrairement à `malloc`, la GLIBC force `calloc` à contourner le *Tcache*. Cela permet de cibler directement les *Fastbins* plutôt que de rester prisonnier de la liste du Tcache (surtout car sa vérification de double-free *key* est plus rude).
* *Q : Pourquoi le Tcache n'est-il pas vulnérable au Double Free d'origine ?*
  * **R :** À partir de la GLIBC 2.29, le Tcache insère une clé (`key` pointant vers la structure du tcache lui-même) dans l'ancien espace utilisateur du chunk lors de sa libération. Avant de libérer un bloc dans le Tcache, l'allocateur vérifie si cette *key* est présente. L'attaque Fastbin permet d'ignorer complètement cette liste.

## Slide 6 : Heap Overflow (À venir)
*(Notes à compléter une fois la slide générée)*

## Slide 7 : Largebin (À venir)
*(Notes à compléter une fois la slide générée)*

## Slide 8 & 9 : ASLR & GLIBC 2.42/2.43
*(Notes à compléter une fois la slide générée)*