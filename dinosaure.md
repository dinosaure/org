DONE extraire rabin d'ocaml-git et en faire une librairie (duff)
================================================================

CLOSED: \[2018-04-06 ven. 14:32\]

DONE Faire un README.md de duff
===============================

CLOSED: \[2018-04-09 lun. 16:52\]

DONE Faire une release de duff
==============================

CLOSED: \[2018-04-21 sam. 01:14\]

DONE Implémenter un fuzzer pour duff
====================================

CLOSED: \[2018-04-06 ven. 15:56\]

DONE Faire un binaire de duff
=============================

CLOSED: \[2018-04-13 ven. 17:50\]

Le fuzzer est lancer sur 163.172.129.132.

DONE faire le README.md de radis
================================

CLOSED: \[2018-04-06 ven. 14:28\]

Corriger les fautes en anglais
------------------------------

DONE Faire une release de radis
===============================

CLOSED: \[2018-05-07 lun. 16:19\]

Il s'agit de regarder surtout le licence et il semble qu'elle soit
soumis à la LGPL 2.1.

DONE Implémenter la fonction `remove`
=====================================

CLOSED: \[2018-04-06 ven. 15:06\]

Son implémentation est faite avec un trick sur les exceptions. Bon, il
faut savoir que Radis n'a pas été fait pour enlever des éléments. La
solution est donc de lever une exception `Empty` quand on trouve \`L (k,
v) when k = key\` et sur les noeuds, on rattrape cet exception pour
affiner l'arbre:

-   B (l, r, \_, \_) -&gt; l ou r (en fonction de la direction)
-   T (m, k, v) -&gt; L (k, v)

C'est pas propre mais bon, il y a le `remove`.

DONE Faire en sorte que `radis` respect `Map.S`
===============================================

CLOSED: \[2018-04-22 dim. 15:46\]

DONE Optimisation de `radis`
============================

CLOSED: \[2018-05-06 dim. 15:58\]

Sur ce plan, j'ai changé la fonction `compare` qui, à mon sens, fait le
plus gros du travail. On fait d'abord une comparaison sur les tailles
des chaines pour savoir si on doit retourner:

-   Contain (dans le cas où `length(a) < length(b)`) et si le contenu
    est équivalent
-   Prefix (dans le cas où `length(a) > length(b)`) et si le contenu est
    équivalent

Dans les autres cas, on est face à `Inf` ou `Sup` (dépends des
caractères)

Enfin, le dernier cas, le *pire*, c'est quand `a = b`.

L'impact ne semble pas si significatif et j'ai besoin de faire un autre
*benchmark* avec `Core_bench` pour avoir un meilleur outil de
comparaison.

Cette optimisation est disponible
[ici](https://github.com/dinosaure/radis/pull/1).

Je viens de faire un *benchmark* avec `Core_bench` et on arrive aux
mêmes constatations que avec `Benchmark`. `radis` est plus rapide que
`Map` ce qui est une bonne nouvelle mais moins rapide que `Hashtbl`.
Bref, il s'agit de faire une *release* maintenant.

DONE Implémenter des tests sur `radis`
======================================

CLOSED: \[2018-05-06 dim. 15:58\]

J'ai juste implémenter un test très basique qui rajoute de manière
*random* des clés et qui les recherche ensuite. Simple, basique.

DONE Implémenter la fonction `update` dans `radis`
==================================================

CLOSED: \[2018-05-07 lun. 14:17\]

DONE extraire RFC1951 de decompress
===================================

CLOSED: \[2018-04-13 ven. 03:22\]

TODO \[ \] contraindre decompress d'utiliser `camlzip.1.07`
-----------------------------------------------------------

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

L'extraction est fini, j'en ai profité pour review un petit peu le code.
On a cependant une question:

Pouvons nous utiliser l'*overload* de l'opérateur `.%{}` (et restreindre
les versions d'OCaml où Decompress compile ou utiliser directement
`Array.unsafe_set` et `Array.unsafe_get`. C'est plus une question sur la
lisibilité du code (et rien n'est prouvé au niveau des performances)
qu'autre chose. Bref, il faut regarder cela plus précisement.

DONE fixer Decompress.Inflate avec un random input
==================================================

CLOSED: \[2018-04-19 jeu. 16:44\]

Bon, il semble que le problème essentiel soit que Decompress n'est
toujours pas invalidé le contenu et attends un peu plus entant
qu'*input*. Le comportement est disponible avec:

``` {.bash}
$ echo -n "a" > input
$ ./dpipe < input > /dev/null
^C
```

Dans ce contexte, puisque Decompress ne vérifie pas le *header* (TODO),
il attends quelque chose. Cependant, le *refiller* retourne `0` et le
considère pas comme étant la fin du fichier - notamment parce que dans
un *stream* depuis une *socket*, ce n'est pas le cas.

Donc le problème concerne plus de comment coder le `refiller` que
Decompress. Bonne nouvelle.

Cependant, il y a tout de même une *infinite loop* bien après avec un
contenu *random*. Donc Decompress est bien trop permissif pour l'instant
ce qui n'est pas le cas de `zlib`, on est bien face à un bug.

DONE Vérifier RFC1951 sur les derniers bytes
============================================

CLOSED: \[2018-04-14 sam. 20:31\]

En effet, puisque RFC1951 n'est pas forcément aligné, il nous faut
vérifier proprement si on a bien écrit les derniers bytes nécessaires
qui devrait se retrouver dans `hold` (et signaler à l'utilisateur
combien de bits sont libres).

DONE Vérifier le *header* dans decompress
=========================================

CLOSED: \[2018-04-19 jeu. 16:44\]

TODO Release de decompress et rfc1951
=====================================

TODO Fuzzer `encore`
====================

DONE Il faut attendre la [PR de Gabriel](https://github.com/stedolan/crowbar/pull/36) pour faire ce travail.
------------------------------------------------------------------------------------------------------------

CLOSED: \[2018-05-14 lun. 12:03\]

La pull-request est disponible
[ici](https://github.com/dinosaure/encore/pull/3). Cependant, on a un
`Out_of_memory` quand on lance `crowbar`. Il faudrait inspecter
pourquoi.

TODO Optimiser ocaml-git
========================

DONE Utiliser `Hashtbl` à la place de `Map`
-------------------------------------------

CLOSED: \[2018-04-30 lun. 22:46\]

DONE Revoir `Pack_info` et réutiliser son code pour l'analyze d'un flux PACK
----------------------------------------------------------------------------

CLOSED: \[2018-04-30 lun. 22:46\]

-   \[X\] factoriser et nettoyer le code
-   \[X\] renommer les fonctions disponibles en `first_pass` et
    `second_pass`
-   \[ \] réutiliser ces fonctions dans `Store` et `Mem`
-   \[X\] optimiser ces *pass*
-   \[X\] optimiser l'accès aux *meta-data* d'une *entry* d'un fichier
    PACK
-   \[X\] refaire la première *pass* (aggréger objets delta-ifiés et
    non delta-ifié)
-   \[X\] refaire la deuxième *pass* (résoudre tout les
    objets delta-ifiés)
-   \[X\] refaire la troisième *pass* (génération d'un nouveau fichier
    PACK non-/thin/)

    Sur ce dernier point, il me semble que obtenir la taille de l'objet
    était plus long pour un objet delta-ifié puisqu'on attendait un état
    au décodeur Hunk se qui impliquait qu'on commence (et même
    qu'on termine) la décompression de l'*entry*. Bref, ce n'est plus le
    cas maintenant.

Bon il faut mettre un peu à plat la logique des PACKs.

Il y a 2 façon de trouver un fichier PACK dans un dépôt git:

-   Il est déjà présent dans le dépôts — dans ce cas, aucune analyze
    n'est faite, mais on a le fichier IDX disposible (normalement)
-   Il est obtenu par le réseau — on a pas de fichier IDX mais pendant
    la transmission, on peut y faire une première analyze

Il faut bien dissocier les deux manières pour comprendre leurs
évolutions. Finalement, dans le `Pack_engine`, on peut dissocier 4
états:

-   le fichier PACK existes (avec un fichier IDX)
-   le fichier PACK est chargé (ainsi que son fichier IDX)
-   le fichier PACK est chargé (ainsi que son fichier IDX) et on a déjà
    fait une première *pass*
-   le fichier PACK est chargé, le fichier IDX n'est plus chargé, on a
    traité tout les objets → on a une `Hashtbl` équivalente au fichier
    IDX (et, sur la vitesse, il est préférable d'utiliser
    cette dernière)

    On sait aussi (puisqu'on a résolu tout les objets) si le PACK est
    *thin* ou pas
-   on a tout les éléments pour extraire n'importe qu'elle objet du
    fichier PACK et on sait (ou on a fait en sorte que) il n'est pas
    *thin*

Dans le dernier cas, on est certains de pouvoir extraire TOUT les objets
du fichier PACK sans AUCUNE allocation (puisqu'elles ont été faite au
préalable).

Arriver à cette situation à cependant un coup comme c'est le cas dans la
version courante d'`ocaml-git`. Il faut en gros extraire tout les objets
du fichier PACK pour ensuite être certain de la situation et promouvoir
l'état au dernier status. On peut déterminer 2 (voir 3) phases:

-   La première consiste à décompresser (seulement) les objets qui ne
    sont pas delta-ifiés et d'aggréger hash et *checksum* de ces objets
    dans une `Hashtbl`.

    Cette phase permet aussi de connaître le **possible** *path* de
    delta-ification (qui, dans le cas le plus commun, correspond au vrai
    *path*). Je veux dire quoi par là. Un objet delta-ifié pointe
    forcément sur un objet (OBJ~REFDELTA~ ou OBJ~OFSDELTA~).
    Normallement, cette source ce trouve **avant** le dit objet.

    -   Dans le cas d'OBJ~OFSDELTA~, c'est assez simple de reconstruire
        la chaîne et de dire que le dit objet à besoin d'une application
        avec l'objet source qui est **déjà** présent dans notre
        `Hashtbl` qui fait office de fichier IDX partiel. Dans ce cas,
        on considère que sa profondeur est celle de l'objet source + 1
        (cette dernière ayant **déjà** été calculé).

    -   Dans le cas d'OBJ~REFDELTA~, c'est plus compliqué. Il peut
        arrivé qu'on ait déjà traiter une **base** (donc qui n'est
        pas delta-ifié) ayant le hash référencé. Cependant, rien ne nous
        assure que ça soit bien le cas - et la documentation est putain
        de pas clair sur ça.

        En cela, si on a de la chance, on peut considérer que la
        profondeur de l'objet est de 1 (vu qu'on ne pourra pas aller
        plus loin dans la delta-ification - peut être que l'objet source
        est lui même delta-ifié mais si c'est le cas, dans la première
        *pass*, on le retrouvera pas vu qu'on aura pas son hash, c'est
        pour cette raison que le note comme étant `Unresolved`).

        Dans le pire cas, cela reste une mystère.

    Ainsi, dans cette *pass*, on a une vu partiel qui peut aider à
    limiter les allocations nécessaires pour extraire les objets du
    fichier PACK mais on ne pourra pas les limiter. L'idée est donc de
    *s'aider* des paths calculés pour donner une allocation
    **partielle** nécessaire au stockage des `hunks`.

    NOTE: La fonction `get` de `Unpack` attends donc à ce que chaque
    niveau soient bien calculés mais qu'il n'est pas nécessaire d'avoir
    tout les niveaux.

-   La deuxième consiste à ce qu'on ait résolu tout les objets
    delta-ifié ce qui veut dire que tout les *paths* devrait être tout
    résolus - à chaque fois qu'on extrait un objet, on obtient aussi son
    *path* résolu.

    En cela, on peut savoir l'allocation nécessaire pour chaque niveau
    pour les *hunks* premièrement et et on peut aussi savoir si le PACK
    est *thin* ou pas - c'est à dire si il demande un objet extérieur
    ou pas.

    Dans le cas où il n'est pas *thin*, on est dans la situation où on
    sait exactement ce qu'on a besoin pour extraire tout les objets. Si
    il est *thin*, puisqu'il y a un (ou plusieurs) objet extérieur au
    PACK requis pour la reconstruction d'un ou plusieurs objets, il
    s'agira forcément d'allouer cet objet (je parle d'allocation
    puisqu'on devra forcément avoir des buffers extérieurs au scope des
    buffers qui sont déjà utilisés) pour reconstruire finalement
    l'objet cible.

-   Et la troisième phase enfin consiste à re-/packer/ le fichier PACK
    pour qu'il ne soit plus *thin*.

Donc avec 2 *entry point*, 5 états et 3 *pass*, je pense qu'on est bon
au niveau de la granularité du traitement des fichiers PACK. Cela semble
carrément compliqué mais il faut aussi saisir que ce n'est pas dans
cette logique que fonctionne `git` (qui reste un utilitaire de commande
qui ne cherche pas à gérer dans un contexte `memory bounded`
l'extraction des objets).

DONE Optimization des fichiers PACK déjà disponibles dans le dépôts
-------------------------------------------------------------------

CLOSED: \[2018-04-26 jeu. 01:49\]

NOTE: une petite illumination sur la gestion du fichier PACK. En effet,
lorsque le fichier PACK est présent dans le dépôts (donc le premier
*entry point*), on peut présupposer qu'il y a aussi le fichier IDX. Dans
ce cas, lors de notre première *pass*, on peut résoudre tout les *path*.

En effet, ce qui casse les *path* est spécifiquement OBJ~REFDELTA~
puisqu'on a pas calculer tout les hashes du dit fichier PACK. Cependant,
le fichier IDX associé peut nous aider à savoir si le hash spécifié par
OBJ~REFDELTA~ est dans le fichier PACK ou non.

En sachant ça, on peut mettre savoir la position absolue de la source et
continuer à construire le *path*. En cela, en une seule *pass*, on peut
calculer le *path* de delta-ification de tout les objets (quand bien
même ils utilise OBJ~REFDELTA~ ou pas), savoir si le fichier PACK est
*thin* (dans le cas où un OBJ~REFDELTA~ n'est pas référencé par le
fichier IDX) ou pas et passer directement au *state* `Resolved`.

Bien entendu, il faut faire cette *pass* (et donc lire la totalité du
fichier PACK) ce qui n'est pas forcément ce que souhaite un utilisateur
dans un contexte *client*. Cependant, cela élude la question de la
promotion des états pour un fichier PACK présent qui, au pire (et
encore, avoir un fichier PACK *thin* déjà disponible dans le dépôts
n'est pas autorisé), doit passer par 2 *pass* (la première et la
troisième pour passer d'un PACK *thin* à un PACK non-/thin/).

Le code est disponibke
[ici](https://github.com/mirage/ocaml-git/pull/292).

DONE Corriger l'accès dan un fichier PACK pour connaitre la taille d'un objet
-----------------------------------------------------------------------------

CLOSED: \[2018-05-03 jeu. 18:15\]

Il me semblait que pour obtenir la taille d'un objet, il s'agissait
juste de lire le *variable length* et c'était bon mais ce dernier
informe juste de la taille de l'*entry* decompressé (et pas de l'objet).
Pour un objet delta-ifié, il faut donc bien, au moins, décompressé le
début pour obtenir la taille réelle de l'objet.

J'ai donc *revert* un de mes commits.

DONE Faire une documentation sur le `Pack_engine`
-------------------------------------------------

CLOSED: \[2018-05-14 lun. 12:04\]

DONE Faire un test avec un PACK *thin*
--------------------------------------

CLOSED: \[2018-05-16 mer. 11:08\]

L'extraction d'un *thin* PACK est pas facile puisque Git le normalize
automatiquement et on retrouve donc un PACK total dans
`.git/objects/pack`. Pour ce faire j'ai donc fait 2 dépôts `decompress`

``` {.bash}
$ git clone https://github.com/mirage/decompress.git decompress_partial
$ cd decompress_partial
$ git checkout 0.5
$ git checkout -b new_master
$ git branch -D master
$ git branch -mv new_master master
$ git fsck
$ git gc
$ cd ..
$ git clone https://github.com/mirage/decompress.git decompress_full
$ cd decompress_full
$ git daemon --reuseaddr --verbose --base-path=. --export-all
```

Dans un autre shell:

``` {.bash}
$ cd decompress_partial
$ git fetch-pack --thin --keep git://localhost/ HEAD
```

Et enfin dans un dernier shell:

``` {.bash}
$ sudo ngrep -xX -d lo port 9418 and dst host localhost &> log
```

Ainsi, dans `log`, on a l'échange du fichier PACK *thin*. Il s'agit de
l'extraire désormais. Il commence par `PACK` et chaque *chunks* sont
dans un *pkt-line* qui commence par une valeur hexa-decimal (dans le cas
présent \#2005) suivi de \#01 si `sideband` est activé (ce qui devrait
être le cas), il faut supprimer ces petits blocks et on a le fichier
PACK.

Et ouais c'est la mort ...

La pull-request est disponible
[ici](https://github.com/mirage/ocaml-git/pull/298/).

TODO Regarder le bottle-neck quand on clone avec `ocaml-git`
------------------------------------------------------------

En très gros, le *process* qui prends le plus de temps, c'est la seocnde
*pass* (25s). Il prendrait 93 % du temps (sans la réception) alors que
la première *pass* semble assez rapide (2s). Donc c'est ici qu'il faut
optimisé. Ces ordres (en temps) sont fait avec les logs, c'est pour ça
que c'est long - mais même sans, c'est long.

Il s'agirait peut être de mettre un cache LRU (puisque c'est le cas dans
Git).

Bon, la compilation du *pattern-matching* dans la fonction
`Unpack.next_object` contient un *when* et `landmarks` me signate que
c'est la fonction la plus longue, j'ai donc supprimé le *when* est
utilisé un simple `if`. Selon `landmarks`, on passe donc de 61m cycles à
53M cyles (M pour million).

La dernière barrière semble être `decompress` et par extension, le
calcul du *checksum* Adler-32 (qui était déjà le problème dans
`decompress`). Bref, il faut faire une librairie qui permet de switcher
entre une implémentation en C et une implémentation en OCaml.

### TODO Faire une librairie "à la `digestif`" qui implémente Adler-32 et CRC-32 (le nom sera `checkseum`)

### DONE Faire un PPX qui re-écrit un type int32 selon l'architecture (produire un int natif - 64 bits - ou int32)

CLOSED: \[2018-05-22 mar. 17:04\]

Bon alors PPX c'est de la merde j'ai donc fait une autre librairie qui
implémente un entier et qui, selon l'architecture, utilise `int` ou
`int32`. Le type est bien entendu abstrait.

TODO Faire une librairie par dessus `ocaml-git` qui résouds les *scheme* (ssh, http, tcp) et exécute les bons modules
=====================================================================================================================

On pourrait ici utiliser un type ouvert pour l'étendre avec `http` dans
`git-http` et `ssh` dans `git-unix`. Bref, c'est une solution mais il
faut bien faire attention.

TODO Fixer le test avec les fichiers PACK *thin*
================================================

TODO Fixer le bug de l'atomique `move` et du dossier `temp` dans `ocaml-git`
============================================================================

TODO Trouver pourquoi on avait pas trouver le bug par rapport aux \000 dans les noms dans les trees
===================================================================================================

TODO Extraire la partie qui sérialize une seule *entry* dans l'implémentation du PACK
=====================================================================================

TODO Permettre d'encoder une seule *entry* pour `wodan`
=======================================================

TODO Faire le serveur
=====================

-   \[ \] Fuzzer l'encoder et le décoder Smart
-   \[ \] Faire le moteur de négociation
-   \[ \] Faire une abstraction du serveur (TCP pour l'instant)

Implémenter un *call-by-need* dans ocaml-git

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

DONE Fixer l'*issue* de Mindy [ici](https://github.com/mirage/ocaml-git/issues/294) quand on *clone*
====================================================================================================

CLOSED: \[2018-05-07 lun. 16:18\]

DONE Intégrer `duff` dans `ocaml-git`
=====================================

CLOSED: \[2018-05-07 lun. 16:55\]

DONE Regarder mirage-lambda et y participer
===========================================

CLOSED: \[2018-05-06 dim. 02:22\]

TODO Tester si mirage-lambda est idempotent
===========================================

DONE S'agit t'il de tester `eval(decode(encode(expr))) = eval(expr)` ?
----------------------------------------------------------------------

CLOSED: \[2018-05-11 ven. 14:38\]

La pull-request est disponible
[ici](https://github.com/mirage/mirage-lambda/pull/10). Le problème
concerne surtout le fait qu'on soit obligé d'intégrer le *fuzzer* pour
avoir accès aux types abstraits de la libraries. `ppx-import` devrait
résoudre le problème, mais je sais pas en vrai.

DONE Fuzzer `mirage-lambda`
---------------------------

CLOSED: \[2018-05-14 lun. 12:02\]

DONE Fixer les tests de `mirage-lambda`
---------------------------------------

CLOSED: \[2018-05-14 lun. 12:02\]

DONE Rajouter un .travis.yml dans `mirage-lambda` avec `autoci`
---------------------------------------------------------------

CLOSED: \[2018-05-14 lun. 12:03\]

TODO Utiliser `protobuf` pour émettre et recevoir du code dans `mirage-lambda`
==============================================================================

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

Faire un outil d'importation des tables de mappings entre *charset* et unicode
------------------------------------------------------------------------------

-   \[-\] outil d'importation des tables ISO8859
    -   \[X\] Extraire les informations des tables
    -   \[ \] Extraire les auteurs
-   \[ \] outil d'importation des tables VENDORS

Il semble que ISO8859 partage le même format pour le *mapping* (format
A). Il semnle aussi que les VENDORS, eux, ne partagent pas
spécifiquement le même format dans le *header* et les valeurs n'ont pas
forcément pas la même représentation.

On peut faire un lexer/parser qui puisse accepter les différents
formats. Cependant, extraire les informations comme le nom, la date ou
encore les auteurs semble être plus difficile.

TODO Ce forcer à utiliser org-mode (2 mois de tests)
====================================================

TODO Avoir deux serveur uDNS (un sur intel et un sur arm) et configurer son PC sur ces serveur
==============================================================================================

Ce qui est compliqué, c'est que les ressources pour utiliser udns sont
inexistante et il faut pousser hannes pour faire un tutoriel.

TODO Implémenter Lwt~sequence~ avec CFML ou Why3 pour ocaml-tcpip
=================================================================

Instrumentaliser la génération du code OCaml par CFML pour *opamiser* le projet
-------------------------------------------------------------------------------

Lwt~sequence~ va devenir obsolète, cela peut donc être une bonne
opportunité de passer à du code prouvé

Un papier de Filliâtre (de 2003, hal-00789533) infirme la possibilité de
prover une liste doublement chaînée avec Why3. Il faudrait donc se
tourner vers CFML - les ressources sont moins accessibles cependant.

Apres discussion, Armael veut bien se lancer dans le projet. Le problème
semble être une notion d'*ownership* dans les nodes pour éviter de les
utiliser si on les a supprimé (data-race condition).

DONE Exporter les parsers d'Emile
=================================

CLOSED: \[2018-04-20 ven. 15:06\]

Dans mon implémentation des fichiers *database* des charset avec
l'unicode, il est nécessaire (pour éviter la duplication de code) que
d'exporter les parsers d'emile.

DONE Meilleur documentation pour Emile
======================================

CLOSED: \[2018-04-19 jeu. 14:57\]

TODO Fixer `quoted_string` (Emile)
==================================

DONE Faire une issue sur [bigstringaf](https://github.com/inhabitedtype/bigstringaf) pour connaitre et déblayer la situation avec `Cstruct` spécifiquement
==========================================================================================================================================================

CLOSED: \[2018-04-19 jeu. 15:02\]

Bon c'est une issue un peu délicate où il s'agit de comprendre la
situation de chacun par rapport au problème initial qu'est le module
`Bigarray`. En soit, la vrai question est de savoir si `bigstringaf`
devrait remplacer `Cstruct.t`, avoir un support pour MirageOS, et porter
les différentes besoins par rapport à `Bigarray`. La liste est celle-ci:

Implémentation C (utilisant `memcpy` et/ou `memmove`).
------------------------------------------------------

La situation est que `Bigarray` utilise `memmove`. `Cstruct` peut ne pas
correspondre puisque de `Bigarray` vers `Bigarray`, il utilise `memmove`
(`Decompress` requiert la sémantique de `memcpy`, qu'importe si c'est un
`Bigarray` ou une `String`).

[ocaml-memcpy](https://github.com/yallop/ocaml-memcpy) par @yallop
propose déjà cette implémentation dirigé par `ocaml-ctypes` ce qui ne le
rends pas forcément favorable au niveau des petites librairies comme
`Angstrom` et `Decompress`.

La question est pourtant difficile puisque une implémentation en C
demande à ce qu'elle fonctionne sur MirageOS et si nous pouvions éviter
du code C, ce serait pas mal (ce que fait `Decompress`). Même si cela
reste du code trivial, le diable se cache dans les détails.

Implémentation en OCaml
-----------------------

C'est l'option prise par `Decompress`. Bien entendu, il y a un impact
sur les performances mais des différents benchmarks que j'ai fait, ce
n'est pas le premier problème concernant `Decompress` (qui concerne plus
le calcul Adler-32).

Pour `Angstrom`, bien entendu (et il me semble qu'il y a un
*benchmark*), l'exécution devrait être plus rapide et dans le contexte
majoritaire où `Angstrom` se retrouvé à utiliser `blit` (et donc
`memmove~/~memcpy`) était pour passer d'un input (qui maintenant est
contraint à être un `Bigarray`) vers une `string`. Dans l'implémentation
de `Cstruct`, dans ce cas précis, il est utilisé `memcpy` avec du code
C.

Cependant, la question à propos de MirageOS a déjà été réglé sur cette
librairie. La question est donc de savoir pourquoi il y a eu un
changement depuis `Cstruct` vers un *stuff* interne, puis, ensuite vers
`bigstringaf`. Bien entendu, ceci devrait être en rapport avec les
performances.

Bound-check
-----------

Une critique pourtant qui s'applique à `Cstruct` est le *bound-check*.
De mon point de vue, il est nécessaire même dans des contextes où on
peut être sûr que le *bound-check* ne soit pas nécessaire. Je préfère
échouer que de corrompre l'information.

Le coût du *bound-check* n'a jamais vraiment été démontré (aucune trace
de *benchmark* pour `Angstrom`). Cela vaut vraiment le coût ? Une
question sans réponse.

Disgression
-----------

Il y a bien entendu la question avec `js_of_ocaml` et `bucklescript`.
Mais flemme, `Cstruct` à un support de `js_of_ocaml` mais pas de
`bucklescript`. Le reste des librairies n'en n'ont pas.

Conclusion
----------

Il ne s'agit pas d'être bêtement contre `bigstringaf` mais de faire un
état des lieux et de choisir le meilleur pour tout le monde. Je serais
ravi qu'il n'y est qu'une seule librairie qui fasse correctement le
boulot.

Cependant, la question ne peut pas être prise à la légère et si
`bigstringaf` devait remplir cette tâche, il lui incombe d'avoir non
seulement un support pour les différents cas (spécialement à propos de
MirageOS).

Ce que je veux dire, c'est `bigstringaf` ne serait pas uniquement de la
volonté d'`Angstrom` et que cette dernière devrait s'occuper d'un plus
large rôle (et cela rajouterais des responsabilités). Bref, c'est
surtout trouver une solution qui puisse convenir et si choisir
`bigstringaf` comme librairie de base pour des plus gros projets (comme
`ocaml-git`) est sans risque.

Bon on peut dire que le problème est réglé avec [cette
PR](https://github.com/inhabitedtype/bigstringaf/pull/10).

TODO refaire `callypige` **URGENT**
===================================

Le code C vient de Google et est sous licence BSD-3. On peut donc
l'importer dans un paquet opam en gardant bien le *header*. L'idée est
de faire comme `digestif` est proposé une implémentation en C et en
OCaml - le code OCaml devrait être hyper lent vu qu'on utiliser `Int64`.

Sinon, @samoht conseille de proposer une génération pseudo-automatique
de *curve25519* par `coq-fiat` et d'en faire un paquet. J'étais plutôt
dans l'idée d'instrumentaliser `coq-fiat` dans un autre projet (*punto*)
pour être à jour mais cela semble être un peu *overkill* (sur
l'installation, on dépendra de `coq` et la génération semble être très
coûteuse).

TODO Avoir `ocaml-tls` sur `async`
==================================

DONE Faire la documentation de `digestif`
=========================================

CLOSED: \[2018-05-16 mer. 11:12\]

C'est surtout à cause du trick sur la production de 2 librairies qui
fait que la génération d'une documentation avec `topkg` et `ocamlbuild`
est impossible.

La pull-request est disponible
[ici](https://github.com/mirage/digestif/pull/27/).

TODO Passer `radis` en MIT pour la prochaine release
====================================================

TODO Implémenter (voir l'e-mail de Joe) les fonctions itératives sur les *hash* pour `digestif`
===============================================================================================

TODO Et faire une release de `digestif`
---------------------------------------
