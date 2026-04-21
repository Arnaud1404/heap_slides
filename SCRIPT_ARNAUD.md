# Script Oral - Arnaud (10 minutes)

> **PRÉAMBULE : TON DE LA PRÉSENTATION**
> Ce script est rédigé et doit être prononcé selon les directives suivantes :
> - **Ton** : Strictement neutre, concis et factuel.
> - **Vocabulaire** : Technique et précis (aucun adjectif familier ou exagéré, pas de formulations "farfelues").
> - **Profondeur** : Justifie de manière didactique les subtilités (comme l'utilisation de `calloc` pour contourner le Tcache, ou le calcul `0x80 - 2`) en s'appuyant directement sur le rapport technique (`main.tex`).

---

## 🟢 Transition (Après Kamiel)

- **[Transition]** "Après l'analyse des mécanismes de tri par Kamiel, nous allons étudier l'application concrète des primitives d'exploitation."
- "L'exploitation du tas repose sur un enchaînement d'étapes, impliquant une corruption initiale de la mémoire, suivie d'une manipulation de son agencement, appelée *Heap Feng Shui*."

---

## 📽️ Slide 12 : Les bases d'un exploit

- **[Slide]** "Une exploitation s'effectue généralement en trois phases."
- **[POINTER - Liste des étapes]** "La première étape consiste à obtenir une primitive de corruption mémoire : l'utilisation d'un pointeur après libération (Use-After-Free), une double libération (Double Free) ou un débordement de buffer (Heap Overflow)."
- "La seconde étape repose sur le contrôle de l'état du tas. En déterminant l'ordre des allocations et libérations, il est possible de positionner des structures cibles à côté des blocs vulnérables."
- "L'association de ces deux éléments permet de modifier des données critiques ou de dévier le flux d'exécution."

---

## 📽️ Slide 13 : Use-After-Free (UAF) & Tcache Poisoning

- **[Slide]** "Le Tcache est souvent ciblé via une vulnérabilité Use-After-Free (UAF)."
- "Le problème survient lorsqu'un programme conserve et utilise un pointeur vers un `chunk` qui a déjà été libéré avec `free`."
- **[REGARDER LE CODE - Ligne 5]** "Dans cet exemple, le UAF permet d'écraser le pointeur `fd` du bloc libéré. Jusqu'à récemment, le Tcache, conçu comme une liste simplement chaînée rapide, ne vérifiait pas la validité de ces pointeurs."
- **[POINTER - Schéma Tcache vers Target]** "En remplaçant `fd` par une adresse voulue, la structure de la liste est modifiée. Lors de l'allocation suivante, la tête de liste pointe vers l'adresse injectée."
- "La dernière allocation (`v = malloc(...)`) retourne alors un pointeur sur cette adresse, offrant une lecture et écriture arbitraire en mémoire."

---

## 📽️ Slide 14 : Défense - Safe Linking

- **[Slide]** "Pour empêcher le Tcache Poisoning, la version 2.32 de la GLIBC a introduit le **Safe Linking**."
- "Cette protection masque les pointeurs stockés dans les listes simplement chaînées."
- **[POINTER - Formule mathématique]** "Le pointeur est chiffré par l'opération logique `(L >> 12) ^ P`, où `L` est l'adresse où le pointeur est stocké et `P` l'adresse mémoire cible."
- "Toute modification directe du pointeur produit un déchiffrement incorrect. L'allocateur tente alors d'accéder à une adresse invalide, causant l'arrêt du programme (SIGSEGV)."
- "Pour contourner ce mécanisme, il devient indispensable de provoquer une fuite d'information (Heap Leak), souvent facilitée par une stratégie de saturation de la mémoire comme le Heap Spraying, afin d'exposer les adresses internes."

---

## 📽️ Slide 15 : Inversion de Safe Linking

- **[Slide]** "Le Safe Linking repose sur un XOR, une opération symétrique et réversible."
- **[LOOK AT - Schéma en cascade]** "Comme `L` et `P` partagent généralement les 12 mêmes bits de poids fort (sur la même page mémoire), un attaquant peut calculer la clé de chiffrement."
- "Une fois l'adresse de base obtenue grâce au Heap Leak, l'attaquant peut chiffrer son propre pointeur puis l'injecter correctement dans les listes."

---

## 📽️ Slide 16 : Double Free - Fastbin Dup

- **[Slide]** "Le **Double Free** consiste à libérer deux fois la même adresse mémoire pour fausser les listes de l'allocateur."
- "La GLIBC bloque les doubles libérations consécutives. Toutefois, en intercalant une autre libération (`free(A); free(B); free(A)`), on contourne la vérification LIFO des Fastbins qui cible seulement le dernier bloc inséré."
- **[EXPLICATION DU CODE - calloc]** "L'utilisation de `calloc` ici est volontaire. Contrairement à `malloc`, `calloc` met la mémoire à zéro et ignore le Tcache pour chercher dans les Fastbins. On évite ainsi les vérifications strictes (notamment le mécanisme de clé anti-double-free) du Tcache."
- **[POINTER - Schéma Fastbin Dup (Boucle)]** "Cette séquence crée une liste circulaire. L'allocateur retourne alors deux pointeurs (ici `c` et `e`) qui référencent la même zone mémoire, permettant l'écrasement de données utilisées ailleurs dans le programme."

---

## 📽️ Slide 17 : Heap Overflow & Overlapping

- **[Slide]** "Le **Heap Overflow** consiste à déborder d'un buffer pour écraser les métadonnées du bloc suivant."
- "Dans cet exemple, l'allocation de `0x80 - 2` est calibrée pour que les données de l'utilisateur arrivent exactement à la limite du bloc voisin sans laisser d'espace vide."
- **[POINTER - Diagramme de droite]** "En écrivant un octet de trop, on modifie le champ `size` du bloc `p2`. On lui donne une valeur falsifiée pour lui faire croire qu'il est plus grand et qu'il englobe aussi le bloc `p3`."
- "Après avoir libéré `p2`, l'allocateur voit un seul grand espace libre. La réallocation de `p4` avec cette nouvelle taille couvre alors physiquement l'emplacement de `p3`."
- **[REGARDER LE CODE - Ligne terminale]** "On se retrouve avec deux pointeurs sur la même zone : on peut modifier `p3` en écrivant normalement dans `p4`."

---

## 📽️ Slide 21 : Exploitation Applicative (Type Confusion)

- **[Slide]** "Suite à l'ajout continu de vérifications sur les métadonnées par la GLIBC, les attaques se tournent vers les données applicatives (ou Data-Only)."
- "Le but n'est plus de corrompre les structures internes de `malloc`, mais de manipuler directement les objets définis par le développeur."
- **[REGARDER LE CODE - Structs definition]** "La Confusion de type exploite la réutilisation de la mémoire. Si on libère un objet `task_t` sans supprimer ses accès (donc un Use After Free), l'allocateur peut réallouer le même espace pour stocker un objet `msg_t`."
- **[POINTER - Ligne 12]** "En écrivant dans le buffer de chaîne de caractères du message, on écrase en fait le pointeur de fonction de l'ancienne tâche. Lorsque la tâche est exécutée, le flux est dévié vers la fonction de notre choix, sans alerter les sécurités de l'allocateur."

---

## 📽️ Slide 22 : Évolutions - GLIBC 2.42 & 2.43

- **[Slide]** "Les versions récentes de la GLIBC intègrent de nouvelles vérifications pour limiter ces techniques."
- "**GLIBC 2.42** : Cette version bloque la Largebin Attack. Lors de l'ajout d'un bloc, le système vérifie que les pointeurs `fd_nextsize` et `bk_nextsize` sont cohérents et forment une boucle valide (`P->bk_nextsize->fd_nextsize == P`). En cas d'incohérence, le programme s'arrête."
- "**GLIBC 2.43** : Cette version supprime totalement les **Fastbins**. Ces listes, manquant de vérifications solides, sont abandonnées pour le Tcache et les Smallbins."
- "De plus, le Safe Linking s'applique désormais de manière globale, sécurisant une plus grande part des listes internes."

---

## 🏁 Conclusion (Orale)

- "En résumé, l'exploitation des métadonnées de la GLIBC est devenue une discipline de haute précision, nécessitant quasi systématiquement des fuites d'information préalables."
- "La tendance actuelle montre un déplacement du champ de bataille : des structures internes de l'allocateur vers la corruption directe des données métier de l'application."
- "Nous vous remercions pour votre attention et sommes à votre disposition pour vos questions."
