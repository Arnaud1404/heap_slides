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

## Slide 6 & 7 : Heap Overflow & Overlapping

**Explication orale :**
* *Code de démonstration :* On alloue trois chunks. L'overflow sur `p2` permet de modifier son header pour inclure l'espace de `p3`. Lors du `free(p2)`, l'allocateur enregistre un pack mémoire de 0x581 octets.
* *L'exploit :* En réallouant `p4` avec cette taille corrompue, on obtient un pointeur qui chevauche `p3`. Le `memset` sur `p4` écrase ainsi silencieusement les données de `p3`.
* *Visualisation Debug :* On voit dans `pwndbg` la taille corrompue `0x581` en rouge. Ensuite, la mémoire est remplie de `0x34` (le caractère '4' en ASCII), prouvant que `p4` modifie bien la zone de `p3`.

**Questions pièges du jury :**
* *Q : Pourquoi le `malloc(0x10)` à la ligne 4 du code ?*
    * **R :** C'est un "guard chunk". Il sert à éviter que le `p3` libéré ne soit consolidé avec le Top Chunk, ce qui rendrait l'overlapping impossible à démontrer simplement.
* *Q : Comment le système détecte-t-il cette corruption maintenant ?*
    * **R :** La GLIBC 2.41 calcule l'adresse théorique du chunk suivant (`p2 + size`). Elle vérifie si à cet endroit, le champ `prev_size` est cohérent. Ici, il ne le serait plus car il pointerait bien après `p3`.

## Slide 7 : Heap Overflow Applicatif (Data-Only Attack)

**Explication orale :**
* *Contexte :* On vient de voir que l'overlap pouvait cibler les métadonnées. Mais la GLIBC rend la manipulation de pointeurs de plus en plus difficile.
* *Le principe applicatif :* On place un buffer manipulable juste avant une structure vitale dans la même zone mémoire. Un simple dépassement suffit. Pas besoin de toucher `size` ou `prev_size` ni de faire un `free`.
* *L'exemple concret :* Écraser un champ `is_admin` d'une structure voisine par un overflow. L'OS laisse passer : notre programme écrit légitimement dans ses propres portions allouées.

**Questions pièges du jury :**
* *Q : Pourquoi nommer ça "Data-Only Attack" ?*
    * **R :** Parce qu'on ne cherche pas à exécuter du code malveillant en redirigeant le flux d'exécution (Control Flow). On modifie simplement de la donnée passive vérifiée par la suite.
* *Q : Quelles sont les défenses contre ce type spécifique d'attaque applicative (hors vtables) ?*
    * **R :** C'est très complexe pour la GLIBC qui ne voit qu'un bloc de mémoire. Il faut un durcissement lié au langage, typiquement avec le Memory Tagging côté hardware (architectures CHERI) pour associer finement chaque variable à son buffer, ou adopter des langages dits "Memory Safe" (comme Rust).

## Slide 8 & 9 : Largebin & ASLR (À venir)
*(Notes à compléter par Kamiel)*

## Slide 10 : Évolutions GLIBC 2.42 & 2.43

**Explication orale :**
* *Contexte :* La GLIBC évolue rapidement. Les techniques que l'on vient de voir sont sévèrement freinées par les versions `2.42` et `2.43` qui arrivent dans les distributions récentes (Ubuntu 26.04).
* *GLIBC 2.42 :* La faille classique du Largebin Attack (que Kamiel expliquera) permettait d'écrire une adresse n'importe où en corrompant `bk_nextsize`. La version 2.42 ajoute une vérification vitale : elle s'assure que le chunk pointé par `fd_nextsize` pointe bien en retour vers notre chunk via son `bk_nextsize`. Si c'est asymétrique, le programme crash.
* *GLIBC 2.43 :* C'est une refonte majeure. Les Fastbins historiques, souvent utilisés pour bypasser les sécurités du Tcache (comme on l'a vu pour le Double Free), sont **supprimés du code**. Tout passe par Tcache et Smallbins, qui sont protégés par le Safe Linking.

**Questions pièges du jury :**
* *Q : Si la 2.43 supprime les Fastbins, votre exploit Double Free fonctionnera-t-il encore ?*
    * **R :** Plus du tout sous sa forme originale. Il faudra trouver des corruptions mémoire qui forcent le passage dans d'autres bacs non protégés, ou utiliser d'anciennes versions pour les systèmes legacy non mis à jour.
* *Q : Pourquoi ces versions ne sont-elles pas partout aujourd'hui ?*
    * **R :** La GLIBC est attachée à l'OS. Les serveurs LTS (qui tournent en production) restent sur des anciennes versions pour la stabilité. Seuls les "rolling release" comme Arch Linux l'ont immédiatement.

## Slide 11 : Conclusion - Data-Only Attacks

**Explication orale :**
* *Bilan :* Traverser toutes ces sécurités (Safe Linking, validations de taille, intégrité des listes) demande aujourd'hui de cumuler beaucoup trop de fuites d'informations (Heap leak, Libc leak, etc.). 
* *Le Pivot Applicatif :* En 2026, l'attaquant ne cherche plus à prendre le contrôle du pointeur retourné par `malloc()`. À la place, on utilise un bug du Heap (comme notre Overlapping chunks) pour **écraser les données mêmes de l'application**.
* *Exemple d'attaque :* On va placer une structure critique (comme un contexte utilisateur avec un booléen `is_admin`) juste à côté d'un buffer vulnérable. En écrasant le champ `is_admin` via un débordement, l'attaquant s'octroie les droits d'administration. C'est ce qu'on appelle une "Data-Only Attack".
* *Conclusion :* Le Heap redevient ce qu'il est : de la mémoire pure. Il n'est plus forcément la cible de l'exécution de code (`RIP`), mais le moyen de corrompre la logique métier du programme, sans jamais déclencher les sécurités de la GLIBC.

**Questions pièges du jury :**
* *Q : Pourquoi ces "Data-Only Attacks" sont-elles si difficiles à bloquer ?*
    * **R :** Parce que, du point de vue de la GLIBC et du système d'exploitation, le programme écrase légalement un espace mémoire qui lui appartient. L'allocateur ne peut pas dissocier un champ `is_admin` d'un champ de texte. Seul le code source applicatif a la notion de "métier".
* *Q : Comment vous défendez-vous contre le pivot applicatif ?*
    * **R :** C'est le plus difficile. Les défenses se tournent vers le CFI (Control Flow Integrity) si l'on attaque des pointeurs de fonctions, ou des architectures comme CHERI qui "taggent" finement la mémoire en hardware pour isoler les sections critiques.