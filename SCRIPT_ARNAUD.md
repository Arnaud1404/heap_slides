# Script Oral - Arnaud (10 minutes)

Ce script utilise un ton neutre et factuel, adapté à une présentation technique universitaire.

---

## 🟢 Transition (Après Kamiel)

- **[Transition]** "Merci Kamiel. Après avoir vu le fonctionnement théorique de l'allocateur, nous allons maintenant aborder les techniques d'exploitation concrètes."
- "Je vais vous présenter les bases de l'exploitation actuelle, qui repose sur trois types de vulnérabilités et sur l'organisation de la mémoire, ce qu'on appelle le Feng Shui de la heap."

---

## 📽️ Slide 12 : Les bases d'un exploit

- **[Slide]** "Un exploit n'est pas le fruit du hasard, c'est une méthode qui se déroule généralement en trois étapes."
- **[POINTER - Colonne de gauche]** "On commence par identifier un bug initial, comme un Use-After-Free, un Double Free ou un débordement classique (overflow)."
- "Ensuite, on utilise le Feng Shui pour organiser la mémoire et placer nos données aux bons endroits."
- "En combinant ces éléments, on parvient à détourner le flux d'exécution pour prendre le contrôle du programme."

---

## 📽️ Slide 13 : Use-After-Free (UAF) & Tcache Poisoning

- **[Slide]** "L'attaque la plus courante sur le Tcache est le Poisoning, souvent basé sur une vulnérabilité de type Use-After-Free."
- "Le principe est le suivant : on libère un bloc de mémoire, mais le programme conserve un pointeur vers celui-ci, ce qui permet de continuer à le modifier."
- **[REGARDER LE CODE - Ligne 5]** "Ici, nous appliquons l'empoisonnement : nous écrivons notre adresse cible dans le pointeur `fd`. Comme on le voit sur le schéma à droite, on remplace le lien normal par l'adresse de notre cible."
- **[POINTER - Flèche Arbitrary Write]** "Le premier `malloc` retire `p` de la liste. L'allocateur regarde alors le pointeur `fd` que nous avons falsifié pour trouver le bloc suivant."
- "Résultat : la liste pointe maintenant sur notre cible. Le deuxième `malloc` nous renvoie alors directement l'adresse choisie, nous permettant d'écrire n'importe où en mémoire."

---

## 📽️ Slide 14 : Défense - Safe Linking

- **[Slide]** "Depuis 2020, la GLIBC intègre une protection nommée **Safe Linking** pour contrer cette technique."
- "L'idée est de ne plus stocker les pointeurs en clair, mais de les masquer avec une opération XOR."
- **[POINTER - Formule mathématique]** "Si on écrit une adresse directement sans tenir compte de ce masquage, le système détecte une incohérence et le programme s'arrête."
- "Pour réussir l'attaque, il faut d'abord obtenir un **Heap Leak**, c'est-à-dire une fuite mémoire pour récupérer la clé de chiffrement."

---

## 📽️ Slide 15 : Inversion de Safe Linking

- **[Slide]** "Ce masquage n'est pas une protection absolue car il repose sur un algorithme simple (XOR-shift)."
- **[LOOK AT - Schéma en cascade]** "Comme on le voit sur ce schéma, l'adresse du bloc est utilisée pour masquer le pointeur. On peut donc inverser l'opération bit après bit."
- "Une fois la clé reconstruite, on peut chiffrer l'adresse que l'on veut atteindre et ainsi passer la sécurité du Safe Linking."

---

## 📽️ Slide 16 : Double Free - Fastbin Dup

- **[Slide]** "La deuxième technique utilise le bug du **Double Free**, qui consiste à libérer deux fois le même bloc de mémoire."
- "L'objectif est de créer une boucle dans les listes de l'allocateur pour obtenir plusieurs accès au même emplacement."
- **[POINTER - Liste fastbin]** "Dans les Fastbins, en libérant le bloc A, puis le bloc B, et enfin à nouveau le bloc A, on peut tromper la vérification de sécurité."
- "On finit par avoir deux pointeurs sur la même zone mémoire, ce qui permet de modifier des données sensibles sans être détecté."

---

## 📽️ Slide 17 : Heap Overflow & Overlapping

- **[Slide]** "Le troisième point concerne le **Heap Overflow**, qui est un débordement de mémoire classique."
- "Si une application ne vérifie pas la taille des données saisies, on peut déborder d'un buffer et modifier le champ **size** du bloc voisin."
- **[POINTER - Diagramme de droite]** "Comme on le voit avec la flèche rouge d'overflow, on modifie la taille du chunk `p2` pour lui faire croire qu'il inclut aussi le chunk `p3`."
- "Lorsqu'on libère puis réalloue cet espace, l'allocateur nous donne le chunk `p4` qui **chevauche** `p3`."
- **[REGARDER LE CODE - Ligne terminale]** "On peut alors écraser silencieusement les données du chunk `p3`, qui est pourtant toujours considéré comme actif par l'application."

---

## 📽️ Slide 18-20 : Largebin Attack

- **[Slides]** "Cette technique est plus complexe car elle manipule les listes doublement chaînées des Largebins."
- "Le principe reste le même : on détourne un mécanisme interne de l'allocateur pour transformer une modification de pointeur en une écriture forcée en mémoire."
- "C'est une attaque qui demande plus de conditions mais qui permet des écritures très précises."

---

## 📽️ Slide 21 : Exploitation Applicative (Type Confusion)

- **[Slide]** "Aujourd'hui, face au renforcement des sécurités de la GLIBC, on voit apparaître des attaques dites **applicatives** (ou Data-Only)."
- "Au lieu d'attaquer les structures internes de l'allocateur, on s'attaque directement aux données métiers du programme."
- **[REGARDER LE CODE - Struct task_t]** "Dans cet exemple, on utilise une confusion de type pour remplacer un message par une structure de tâche."
- "Le programme pense lire une chaîne de caractères, mais il exécute en réalité l'adresse d'une fonction que nous avons injectée."

---

## 📽️ Slide 22 : Évolutions - GLIBC 2.42 & 2.43

- **[Slide]** "Pour finir sur les évolutions récentes, sachez que les versions 2.42 et 2.43 de la GLIBC ferment plusieurs de ces failles."
- "La 2.42 ajoute des vérifications pour bloquer la Largebin Attack, et la 2.43 supprime les Fastbins, qui étaient une source historique de vulnérabilités."
- "Cela montre que l'exploitation des métadonnées devient de plus en plus difficile."

---

## 📽️ Slide 23 : Conclusion

- **[Slide]** "Pour conclure, s'il est devenu très complexe de prendre le contrôle direct du processeur via la heap, le risque reste présent."
- "Le danger s'est déplacé vers la **corruption de données applicatives**, où l'on modifie le comportement du logiciel sans casser les règles de la mémoire."
- "Merci de votre attention, nous sommes disponibles pour répondre à vos questions."
