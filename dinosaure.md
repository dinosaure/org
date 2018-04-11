DONE extraire rabin d'ocaml-git et en faire une librairie (duff)
================================================================

CLOSED: \[2018-04-06 ven. 14:32\]

DONE Faire un README.md de duff
===============================

CLOSED: \[2018-04-09 lun. 16:52\]

TODO Faire une release de duff
==============================

DONE Implémenter un fuzzer pour duff
====================================

CLOSED: \[2018-04-06 ven. 15:56\]

Le fuzzer est lancer sur 163.172.129.132.

DONE faire le README.md de radis
================================

CLOSED: \[2018-04-06 ven. 14:28\]

Corriger les fautes en anglais
------------------------------

TODO Faire une release de radis
===============================

DONE Implémenter la fonction `remove`
=====================================

CLOSED: \[2018-04-06 ven. 15:06\]

Son implémentation est faite avec un trick sur les exceptions. Bon, il
faut savoir que Radis n'a pas été fait pour enlever des éléments. La
solution est donc de lever une exception `Empty` quand on trouve \`L (k,
v) when k = key\` et sur les noeuds, on rattrape cet exception pour
affiner l'arbre:

-   B (l, r, \_, \_) -&gt; l ou r (en fonction de la direction)
-   T (m, k, v) -&gt; -&gt; L (k, v)

C'est pas propre mais bon, il y a le `remove`.

TODO extraire RFC1951 de decompress
===================================

\[ \] contraindre decompress d'utiliser `camlzip.1.07`
------------------------------------------------------

Une relecture de decompress m'amène à faire:

-   une factorisation du code comme j'ai fait dans le PACK décodeur. En
    effet, `get_bits`, `get_byte`, etc. sont implémentés plusieurs fois
    dans le code. Il s'agit de rajouter un argument `ctor` pour savoir
    sur quelle ADT utiliser.
-   Dans `Dictionary.inflate.get`, on vérifie `t.bits` pour ensuite
    exécuter `peek_bits` qui fait la même vérification. C'est une
    duplication des vérifications que je viens de supprimer (l'impact
    sur les performances devrait être marginal cependant).
-   `Dictionary.inflate` pose un gros problème. En effet, lorsqu'il
    s'agit de répéter l'opération `loop` après avoir lu quelques bits
    avec `get`, la valeur n'est pas repris correctement par le `k` avec
    ses arguments. C'est hyper bizarre.

    Bon en réalité c'était des noms de variables qui correspondaient pas
    à ce qu'ils faisaient mais l'ordre était toujours bon. Genre:
    -   \`get (fun src dst t -&gt; loop src dst t)\` // partial
        application
    -   \`get (fun v src dst t -&gt; loop v src dst t)\`
-   Je viens de me rendre compte la `window` peut être accessible à
    partir de `t` au lieu de le passer aux fonctions. Cela peut être une
    optimisation car, dans certains cas, on n'alloue pas des closures.

La PR est disponible
[ici](https://github.com/mirage/decompress/pull/41). Le diff est pas
trop lisible cependant.

TODO fixer Decompress.Inflate avec un random input
==================================================

Fuzzer encore
=============

Il faut attendre la [PR de Gabriel](https://github.com/stedolan/crowbar/pull/36) pour faire ce travail.
-------------------------------------------------------------------------------------------------------

TODO Optimiser ocaml-git
========================

TODO Trouver pourquoi on avait pas trouver le bug par rapport aux \000 dans les noms dans les trees
===================================================================================================

TODO Faire le serveur
=====================

\[ \] Fuzzer l'encoder et le décoder Smart
------------------------------------------

\[ \] Faire le moteur de négociation
------------------------------------

\[ \] Faire une abstraction du serveur (TCP pour l'instant)
-----------------------------------------------------------

Implémenter un *call-by-need* dans ocaml-git
============================================

L'idée est de ne pas obtenir l'objet Git dès qu'on souhaite juste le
manipuler (`Value.t`) mais de l'obtenir seulement quand on souhaite
accéder à une information à l'intérieur (comme `Value.Commit.tree`).

On peut imaginer cette définition:

``` {.ocaml}
type lazy =
  | Pack of { hash : Hash.t; offset : int64 } (* identifiant du PACK et son offset dans le dit-PACK *)
  | Loose
and t =
  | Loaded of [ `Commit of Commit.t | ... ]
  | Unloaded of lazy
```

Bon après, je sais pas (et je pense pas) que cela soit vraiment
efficace. On est déjà dans une politique *call-by-need* dans le sens où
on charge les objets seulement quand on les demande explicitement.

Ici, il s'agit d'affiner un peu plus le *call-by-need* et de faire les
opérations nécessaires seulement quand on souhaite non seulement obtenir
l'objet mais aussi obtenir les informations qu'il contient - maintenant
est ce que ce n'est pas déjà le cas ?

Le retour de Thomas: Il voudrait raffiner le parser en collectant les
informations non pas d'un block comme c'est le cas mais petit à petit.
On pourrait s'en sortir avec `Angstrom` en splittant le parser en
plusieurs morceaux et en modifiant l'interface `Commit.D` pour notifier
dès qu'on a décoder le `tree` (puisque c'est spécifiquement celui ci qui
nous intéresse) et garder l'état du parser pour le faire continuer si
l'utilisateur demande plus d'informations.

Il est vrai que dans le format du commit, le `tree` est la première
information et on a ensuite les parents - qui sont toutes les deux des
informations relatives au parcours du DAG. Donc on peut imaginer que
cela puisse être intéressant - on évite notamment de décompresser au
meilleur des cas les autres valeurs et le message.

Bref, il faudrait s'intéresser à la question mais elle serait spécifique
en réalité au `Commit`, répercuter le code sur les `Blob` n'a pas de
sens par exemple.

TODO Regarder mirage-lambda et y participer
===========================================

Passer Mr. MIME à Angstrom
==========================

J'ai eu une idée de GADT.

``` {.ocaml}
type 'a field
  | Content_type : content_type field
  | Msg_id : msg_id field

type res =
  [ `Await of decoder
  | `Header of ('v field, 'v)
  | `End ]
```

En gros, le décodeur va s'arrêter à chaque *fields* de l'e-mail et
donner sa valeur. Ce sera super bien typé grâce au GADT `field`. Il
suffira d'une fonction `continue` pour passer au `field` suivant ou au
`body`. Cependant, la question du `body` (entre `multipart/alternate` ou
simple `multipart`) ce pose toujours.

Gérer l'*encoding* des e-mails (normaliser un *encoding* vers de l'UTF-8)
=========================================================================

Camomile fait déjà le *mapping* entre les *encodings* et l'unicode.
Cependant, en regardant le code, c'est à la fois complexe, redondant et
certainements inutiles. Un simple exemple, la structure permettant de
mapper un code d'un encoding vers un autre est implémenter dans `Tbl31`:
le code est juste immonde - un patricia tree suffirait largement (bien
entendu, il faudrait faire des benchmarks mais on y gagnerais en
lisibilité).

Bon selon dbuenzli, Camomile supporterait que unicode 3 et dépends de
fichiers externes. Deux erreurs qu'il ne faudrait pas reproduire.

TODO Ce forcer à utiliser org-mode (2 mois de tests)
====================================================

TODO Avoir deux serveur uDNS (un sur intel et un sur arm) et configurer son PC sur ces serveur
==============================================================================================

Ce qui est compliqué, c'est que les ressources pour utiliser udns sont
inexistante et il faut pousser hannes pour faire un tutoriel.

Implémenter Lwt~sequence~ avec CFML ou Why3 pour ocaml-tcpip
============================================================

Lwt~sequence~ va devenir obsolète, cela peut donc être une bonne
opportunité de passer à du code prouvé