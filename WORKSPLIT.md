# 🚀 Répartition des Slides : Exploitation Heap 2026

**Objectif :** 20 minutes (10 min/personne).
**Stack :** LaTeX Beamer (`metropolis`), `minted`, `tcolorbox`.
**Workflow :** Branches Git séparées, merge final ce soir.

---

## 🏗️ Kamiel : Théorie, Fondations et ASLR (10 min)

### Section I : Introduction et Fondamentaux
* **Slide 1 : Objectifs du TER**
    * Présentation du sujet et périmètre de recherche (GLIBC 2.41).
* **Slide 2 : L'arène `ptmalloc2`**
    * Gestion via `brk()` (contigu) et `mmap()` (anonyme).
    * Hiérarchie de recyclage : Tcache, Bins (Fast, Small, Large, Unsorted).
* **Slide 3 : Anatomie d'un Chunk**
    * Structure physique : `prev_size`, `size` et bits d'état.
    * Métadonnées en état libre : apparition des pointeurs `fd` et `bk`.

### Section II : Primitive Moderne (Largebin)
* **Slide 8 : Largebin Attack**
    * Corruption des pointeurs `bk_nextsize` via UAF.
    * Primitive : Écriture d'adresse arbitraire via le tri de l'Unsorted Bin.

### Section III : Défenses
* **Slide 9 : La barrière de l'ASLR**
    * Randomisation des bases (Libc, Heap, Stack).
    * Nécessité de l'Arbitrary Read (Leak) pour recalculer les offsets réels.

---

## ⚔️ Arnaud : Exploitation Offensive et Futur (10 min)

### Section II : Primitives d'Exploitation (Suite)
* **Slide 4 : Use-After-Free (UAF) & Safe Linking**
    * Empoisonnement du Tcache via corruption de `fd`.
    * Algorithme du Safe Linking : $P \oplus (L \gg 12)$.
* **Slide 5 : Double Free**
    * Fastbin Dup et création de boucles circulaires ($A \to B \to A$).
    * Bypass de la protection `key` du Tcache via les Fastbins.
* **Slide 6 : Heap Overflow & Overlapping**
    * Corruption du champ `size` pour étendre un chunk.
    * Primitive : Aliasing mémoire (deux pointeurs sur une même zone).
* **Slide 7 : Heap Overflow Applicatif (Data-Only)**
    * Corruption de structures métier C adjacentes (ex: variables de droits).
    * Bypass total des métadonnées `malloc` face aux durcissements GLIBC.

### Section III : Perspectives et Conclusion
* **Slide 10 : Évolutions GLIBC 2.42 & 2.43**
    * 2.42 : Vérification de cohérence `nextsize` (neutralisation Largebin Attack).
    * 2.43 : Suppression physique des Fastbins au profit du Tcache/Smallbins.
* **Slide 11 : Conclusion**
    * Pivot vers les Data-Only Attacks.