# Problèmes ouverts

Liste des problèmes techniques et de conception à résoudre pour que le jeu soit réalisable. Chaque problème a un identifiant pour qu'on puisse y faire référence, et un statut :

- **Ouvert** : rien de décidé.
- **Piste** : une solution est envisagée, mais pas validée.
- **Prototype** : à valider par un prototype technique.
- **Décidé** : réglé ; la décision est reportée dans le document concerné.

Principe général proposé : **ne simuler finement que là où ça compte**, et simuler de façon agrégée partout ailleurs.

----

## A. Simulation

### A-01 — Nombre de zombies : simulation par tiles

- **Problème** : des dizaines de millions de zombies (la France compte environ 68 millions d'habitants), qui courent en permanence avec ouïe et cône de vue. Impossible de tous les simuler individuellement.
- **Décision** : le monde est découpé en tiles d'environ 1 km (taille à confirmer par prototype), sur une grille alignée sur le Lambert-93, comme les carreaux de l'INSEE. Outre-mer, la grille suit la projection du territoire (C-01).
  - Une tile est **active** si elle contient des humains au contact possible de zombies, ou si elle touche une telle tile. Les zombies y sont des agents complets.
  - Les autres tiles à zombies sont **inertes** : un état stocké (« X zombies, tel bâtiment détruit, X % brûlé… »).
  - **Migration** : les zombies des tiles inertes se déplacent lentement de tile en tile, attirés par les stimuli (bruit, fumée…). Le monde n'est jamais figé. Un incinérateur de corps qui tourne attire ainsi les zombies des tiles inertes voisines, et peut en activer de nouvelles (A-16). Dans une tile sauvegardée, un zombie qui arrive apparaît près du bord par lequel il entre, et ceux qui partent sont les plus proches du bord de sortie. Une tile inerte qui compte 40 000 zombies n'en reçoit plus : les envois refusés restent dans leur tile de départ (P-04).
  - **Activation / désactivation** : les compteurs deviennent des agents placés de façon déterministe et crédible (bâtiments, rues), et inversement, sans perte.
  - **Désactivation** : une tile qui a été active **sauvegarde tout** : la position de chaque zombie, sa carte des corps, ses **constructions et leur état**. Les zombies en poursuite ou relayés redeviennent **errants**. Une tile jamais activée reste un simple compteur. Les tiles sauvegardées sont **stockées sur disque** et chargées à l'activation (A-13).
  - **Constructions dans une tile inerte** : sans humain, les zombies n'attaquent pas les constructions. Une base avancée abandonnée dans une tile inerte reste donc intacte et peut être reprise. Les constructions restent **en mémoire** (quelques octets chacune) pour l'affichage sous le brouillard ; seuls les zombies et la carte des corps vont sur disque (P-04).
  - **Affichage** : les tiles sont une division logique, invisible à l'écran ; la caméra passe librement de l'une à l'autre. Une tile inactive est affichée avec ses constructions, mais **sans ses zombies**, légèrement grisée : c'est le brouillard de guerre.
  - **Délai de désactivation** : une tile reste active environ une minute de jeu après la disparition de sa raison d'être active, pour éviter les bascules répétées. Maintenu après P-04 : une bascule ne coûte rien de visible.
  - **Bord réfléchissant** : un zombie **sans cible** ne sort pas vers une tile non active (inerte ou en cours d'activation) ; un zombie en poursuite traverse (P-04).
  - **Capacité** : une tile active compte au plus **65 000 zombies** ; au-delà, un passage de frontière vers elle est refusé et les zombies attendent au bord, dans leur tile, comme devant une case pleine (A-05, P-04).
  - **Activation loin des humains** : les zombies d'une tile qui s'active hors de portée de tout humain naissent directement en **veille profonde**.
  - **Aéronefs** : en vol, ils n'activent aucune tile, sauf à leurs points d'action (frappe, largage, poser).
  - **Feu** : il continue dans les tiles inertes, de façon agrégée (« X % de la tile brûlé »).
  - **Tirs à longue portée** : un tir observé ou une frappe d'artillerie sur une tile inerte l'active, comme pour les aéronefs.
- **Contraintes** : la distance de vue maximale d'un zombie doit rester inférieure à la largeur de la couronne (environ 1 km). Une tile de centre-ville dense peut contenir plus de 20 000 zombies : c'est le cas de Paris, pire cas à mesurer pour la reconquête d'une grande ville.
- **Prototype P-01** ([résultats](https://github.com/fafapiti/prototypes/blob/main/p01-tile-paris/README.md), emprises réelles de la BD TOPO, relais d'alerte et veille) : une tile parisienne de 20 000 zombies coûte 1,0 ms (calme) à 1,7 ms (assaut) par tick sur la machine de référence. Environ 12 à 28 tiles de ce type tiennent à ×1 selon le scénario, 1 à 3 à ×10. Le coût est du calcul, linéaire en nombre de tiles ; la perception en représente encore 45 % au calme. Le relais gagne 30 à 50 % en combat, la veille 25 % au calme.
- **Décision** : **veille profonde**. Un zombie plus loin de tout humain que sa distance de vue sort de l'index spatial et ne se déplace qu'une fois par seconde, sans collision avec les autres zombies. Il repasse en veille normale dès qu'un humain approche ou que la caméra le regarde (P-01 : l'index pèse 31 % du coût d'une tile calme).
- **Décision** : la densité de zombies est proportionnelle à la population (A-10).
- **Constat** (P-01) : un front permanent autour de Paris (environ 93 tiles avec zombies) dépassait le budget d'un facteur trois. C'est l'une des raisons de l'abandon de Paris fortifié au profit des colonies (A-15) : le joueur choisit désormais la taille de son front.
- **Prototype P-02** ([résultats](https://github.com/fafapiti/prototypes/blob/main/p02-perception/README.md), structure de perception, humains mobiles) : un **arbre k-d** des humains par tile, reconstruit à chaque tick, divise le coût d'une requête de perception par 8 à 13 et rend ce coût indépendant de la distance de vue. Avec le filtre du relais et la veille profonde (−23 % au calme), une tile parisienne coûte 0,5 ms (calme) à 0,9 ms (dispersé) par tick en coût marginal : environ 44 tiles calmes, 20 dispersées ou 24 en assaut à ×1. Le premier poste au calme devient la reconstruction de la grille des zombies (45 %).
- **Décision** (P-02) : la perception interroge un **arbre k-d des humains par tile**, reconstruit à chaque tick : ouïe d'abord, puis vue, du plus proche au plus loin.
- **Décision** (P-02) : la **distance de vue** n'est plus plafonnée pour des raisons de coût : environ 400 m en ville, jusqu'à 800 m en terrain découvert, toujours sous la largeur de la couronne. Elle est **fixée par tile**, selon sa part de bâti, calculée une fois à la préparation de la carte (B-01) ; le seuil de veille profonde de la tile s'en déduit. Conséquence : la veille profonde commence au-delà de la distance de vue, et non plus à quelques centaines de mètres.
- **Décision** (P-02) : pour une **tile à forte densité d'humains** (colonie de plusieurs milliers de civils), l'index des humains est reconstruit environ 4 fois par seconde : les zombies perçoivent des positions vieilles d'au plus un quart de seconde. Depuis P-05, un bâtiment occupé n'y figure que par **un point à sa porte** (A-15), et seuls les colons dehors y sont individuels.
- **Constat** (P-02, complément) : comme la veille profonde commence au-delà de la vue, la vue a un coût indirect. Une tile calme coûte 33 % de plus avec une vue de 400 m, 59 % de plus à 800 m, faute de zombies endormis ; en assaut, 5 % et 16 %.
- **Décision** (P-02) : **perception entre tiles** : chaque tile garde son propre arbre ; un zombie interroge aussi les arbres des tiles voisines que sa vue atteint. Les constructions restent locales et parallèles ; un arbre lointain est écarté sur sa seule boîte englobante.
- **Décision** (P-02) : la recherche de relais entre zombies reste faite **à chaque tick**, malgré son coût en combat (10 à 20 % du tick) : la faire moins souvent réduirait fortement la portée des hordes ([ZOMBIES.md](game-elements/zombies/ZOMBIES.md#perception)).
- **Prototype P-03** ([résultats](https://github.com/fafapiti/prototypes/blob/main/p03-flow-fields/README.md), flow fields, index des zombies, réservation partout, vue de 400 m) : environ 40 tiles calmes, 18 à 20 dispersées, 28 en assaut à ×1 (2, 1 et 1 à ×10), mieux que P-02 à la même vue. La grille dense compacte (u16, sans curseurs) divise par 3,6 la reconstruction de l'index des zombies à 16 tiles ; le premier poste devient le déplacement (36 % au calme, 40 à 46 % en combat).
- **Décision** (P-03) : index des zombies en **grille dense de 2 m en u16, sans tableau de curseurs** ; grille triée et grille creuse écartées (requêtes 2 à 3 fois plus lentes), cases de 4 m écartées (densité non bornée).
- **Prototype P-04** ([résultats](https://github.com/fafapiti/prototypes/blob/main/p04-activation/README.md), bloc de 55 tiles réelles de Paris, environ 690 000 zombies) : l'activation et la désactivation sont **exactes** (conservation vérifiée à chaque tick, 366 restaurations identiques octet pour octet, constructions et corps compris) et **déterministes** (identiques de 1 à 16 threads). Étalée sur environ 27 ticks, une activation ne dépasse jamais 3 ms par tick ; d'un seul tenant, le tick monte à 12 à 18 ms. Une colonne dense traverse 10 km à ×1 sans tick au-dessus de 7,3 ms. Le délai de désactivation n'a pas d'enjeu de coût : une bascule complète ne se voit pas dans le p99.
- **Constat** (P-04) : au bord d'une tile non active, les zombies **errants** fuient en sens unique (ils y entrent par diffusion et n'en ressortent jamais) : 10 % de la couronne en 20 min. Un **bord réfléchissant** pour les zombies sans cible supprime la fuite ; un zombie en poursuite traverse toujours.
- **Constat** (P-04) : la **migration par journal** fonctionne sans relire les sauvegardes (233 000 migrants en 70 min) : un départ retire les zombies les plus proches du bord de sortie, une arrivée les place dans une bande de 20 m le long du bord d'entrée ; le journal est rejoué, étalé, à l'activation. Sans borne, un incinérateur concentre plus de 65 000 zombies dans une tile.
- **Décision** (P-04) : **bord réfléchissant** pour les zombies sans cible ; **saturation** de la migration à 40 000 zombies par tile inerte ; **capacité** de 65 000 zombies par tile active ; délai de désactivation de 60 s maintenu ; constructions des tiles inertes résidentes en mémoire ; activation étalée (voir [F-02](#f-02--performances-de-la-simulation)).
- **Constat** (P-07, [résultats](https://github.com/fafapiti/prototypes/blob/main/p07-chaine-manche/README.md)) : les carreaux INSEE (Filosofi 2021) donnent jusqu'à **42 600 habitants** dans une tile parisienne (21 300 en médiane ; 50 tiles sur 105 au-dessus de 20 000), soit environ 38 000 zombies après évacuation : le double du pire cas de P-01 à P-04, au-delà de la saturation de migration (40 000), sous la capacité d'une tile active (65 000). En France, 69 carreaux de 1 km dépassent 20 000 habitants. La Manche n'en a pas plus de 6 600.
- **Constat** (P-07) : la grille des carreaux de l'INSEE n'est **pas** celle du Lambert-93 depuis 2017 : elle est en ETRS89-LAEA (EPSG 3035). La grille des tiles reste le Lambert-93 ; les carreaux sont redistribués sur les bâtiments (A-10).
- **Constat** (P-07) : la zone accessible se calcule par **étiquetage distribué** (composantes par tile, union-find sur les bords) : 0,04 s de fusion pour la Manche, mémoire bornée quelle que soit la surface. Critère d'accès : composante traversée par une route ou un chemin, ou la plus grande (sinon les îles n'auraient pas de zombies). 99,98 % des cases libres de la Manche sont accessibles, 93,9 % à Paris avant percement des cours.
- **Décision** (P-07) : saturation de la migration (40 000) et capacité d'une tile active (65 000) **maintenues** : la tile la plus dense de France part d'environ 38 000 zombies, sous les deux seuils.
- **Statut** : Décidé dans le principe ; Prototype (priorité 1) pour la taille des tiles ; coût d'une tile urbaine dense mesuré (P-01) ; structure de perception décidée (P-02) ; index des zombies décidé (P-03) ; activation, désactivation, frontières et migration décidées (P-04).

### A-14 — Territoires sans zombies (outre-mer, zones reconquises)

- **Problème** : les outre-mer comptent environ 10 millions d'habitants dans le scénario, dont environ 7 millions de réfugiés évacués de métropole. Avec la seule règle « humains → tile active », ils seraient simulés en agents complets en permanence. Mais ne pas les simuler serait dommage.
- **Décision** : troisième état de tile, **sûre** : des humains, sans zombie à portée.
  - Économie complète (production, logistique, main-d'œuvre, consommation).
  - Unités du joueur simulées individuellement.
  - Population civile en **dossiers par bâtiment** (A-15) ; seuls les colons dehors deviennent des entités, quand la caméra s'approche.
  - Dès qu'un zombie arrive à portée, la tile redevient active : pas d'auto-résolution, puisqu'aucun combat n'a lieu dans une tile sûre.
- **Effet** : les zones reconquises de métropole deviennent aussi des tiles sûres, ce qui limite la croissance du coût en fin de partie (A-03).
- **Décision** : les outre-mer sont totalement exempts de zombies ; toutes leurs tiles sont sûres.
- **Constat** (P-05, [résultats](https://github.com/fafapiti/prototypes/blob/main/p05-civils/README.md)) : en dossiers par bâtiment, 3 millions de civils coûtent 0,15 ms par tick avec des jauges toutes les 10 s. Les 10 millions d'humains d'outre-mer coûteraient environ 0,5 ms (estimation linéaire, non mesurée) et 500 Mo, au-delà du budget de ×100 (0,25 ms). Piste : une période plus longue (30 à 60 s) pour les tiles sans caméra ni colonie.
- **Décision** (P-05) : les civils d'outre-mer sont aussi en dossiers par bâtiment ; leurs jauges sont mises à jour toutes les **30 à 60 s** dans les tiles sans colonie ni caméra, sur l'intervalle exact écoulé.
- **Statut** : Décidé.

### A-15 — Civils en métropole : colonies

- **Problème** : les civils de métropole doivent être simulés (faim, soif, fatigue, moral), approvisionnés et protégés, sans faire exploser le coût de la simulation.
- **Décision** : **colonisation**. Des navires de civils attendent au large de Brest. Le joueur débarque les colons par Brest, puis par **tout port reconquis**, et les installe **où il veut** sur le territoire reconquis : villes, villages, quartiers.
  - Les **navires de colons sont des unités simulées** : le joueur embarque manuellement les colons en outre-mer, les fait voyager à vitesse réelle, puis les débarque. Le rythme d'arrivée est **limité par la flotte disponible**.
  - Les colons sont la **main-d'œuvre** de la production locale (ouvriers).
  - Le joueur fortifie les quartiers qu'il occupe et approvisionne ses colonies.
  - **Chute d'une colonie** : ses civils deviennent des zombies (perte totale). La chute est **émergente** : les zombies entrent et l'infection se propage selon les règles normales ; aucune règle ne la déclenche.
  - Effet recherché : une raison réelle de reprendre les villes, au lieu de foncer vers les objectifs stratégiques.
- **Effet sur le coût** : une colonie isolée au milieu des zombies a des tiles actives à sa périphérie ; une colonie entourée de zones nettoyées est en tiles sûres (A-14). Le joueur choisit donc la taille de son front, et nettoyer autour d'une colonie la rend moins chère à simuler.
- **Décision** (P-05) : les civils sont rangés en **dossiers compacts par bâtiment**, hors de l'ECS : un enregistrement par colon (identité, logement, travail, jauges, expérience, compétences, maladie), dans le dossier du lieu où il se trouve. Analogie : un pool de workers qui ne réveille que les dossiers arrivés à échéance, au lieu d'un processus par client. Les soldats restent toujours individuels.
- **Abandonné** (P-05) : l'**agrégat statistique** (effectifs et moyennes) : il perd l'identité, l'expérience et les compétences (A-18), et ses résultats divergent (froid et maladie non linéaires).
- **Décision** (P-05) : seuls les colons **dehors** deviennent des entités : ceux qui marchent sous la caméra et ceux qui sont dans la rue d'une tile active. Les occupants d'un bâtiment ne sont **jamais** individualisés, ni quand la tile devient active, ni quand un zombie arrive à portée : l'intérieur n'est pas simulé et le combat s'y auto-résout sur les dossiers (B-02).
- **Décision** (P-05) : en tile sûre, un **chantier extérieur** est un lieu agrégé comme un bâtiment : ses ouvriers ne deviennent des entités que sous la caméra.
- **Décision** (P-05) : un colon individualisé retourne à son dossier **30 s** après la fin de sa raison d'être individuel ; le rayon de sortie de la caméra dépasse son rayon d'entrée de **100 m** (hystérésis), pour qu'un léger mouvement de caméra ne fasse pas osciller les bâtiments.
- **Décision** (P-05) : les jauges des civils sont mises à jour toutes les **10 s** en tile sûre (**30 s** au-delà de 2 millions de colons), sur l'intervalle exact écoulé ; un colon individualisé garde la **même période** que dans son dossier, ce qui garantit qu'un aller-retour dossier → entité → dossier ne change rien.
- **Abandonné** : un million de survivants dans les 20 arrondissements de Paris, avec le périphérique comme ligne de défense. P-01 a montré que ce front permanent (125 tiles actives) dépassait le budget d'un facteur trois.
- **Constat** (P-01, géométrie réelle de la BD TOPO, avant l'abandon) : le périphérique fait environ 36 km. Il active en permanence **125 tiles** : 43 qu'il traverse, 32 voisines intérieures sans zombies et 50 voisines extérieures. Restent **environ 93 tiles avec zombies**, soit environ 70 ms par tick même au calme avec la veille, pour un budget de 25 ms : **trois fois trop**. Un assaut sur tout le périmètre à la fois (environ 130 ms) n'est pas tenable à ×1. Leviers : densité de zombies hors de Paris, veille profonde, P-02 ; voir [P-01](https://github.com/fafapiti/prototypes/blob/main/p01-tile-paris/README.md#analyse-test-2).
- **Constat** (P-05, [résultats](https://github.com/fafapiti/prototypes/blob/main/p05-civils/README.md)) : sur les bâtiments réels de la BD TOPO, avec des jauges, des maladies, une consommation, un cycle quotidien et des trajets abstraits, **l'agrégat statistique n'est pas nécessaire**. Des **dossiers compacts par bâtiment** (32 octets par colon, hors de l'ECS) font exactement le même calcul qu'une entité par colon (même somme de contrôle) :
  - jauges toutes les 10 s : 1 million de colons en **0,08 ms par tick** (un tiers du budget de ×100), 3 millions en 0,15 ms ;
  - jauges à 1 Hz : 0,25 ms à 1 million, soit **tout** le budget de ×100 ;
  - une entité par colon coûte 2,5 à 12 fois plus, selon que les entités sont rangées par phase ou non, et 3 fois plus de mémoire (150 octets contre 45 par colon).
- **Constat** (P-05) : un **agrégat statistique** (effectifs et moyennes) coûte 4 à 5 fois moins que les dossiers, mais il perd l'identité, l'expérience et les compétences de chacun (A-18). Ses résultats divergent aussi : 63 % de morts de plus en hiver, parce que le froid et la maladie ne sont pas linéaires.
- **Constat** (P-05) : individualiser puis rendre à l'agrégat est **exact** : l'état final est identique, colon par colon, à celui d'une simulation sans caméra. Un bâtiment s'individualise en 1 tick, pour 80 à 290 ns par colon. Mais une caméra qui individualise les bâtiments dans un rayon de 250 m fait jusqu'à 49 500 entités pour 250 à 320 colons visibles dehors : à l'intérieur d'un bâtiment (B-02), une entité ne fait rien de plus qu'un dossier.
- **Constat** (P-05) : pendant un assaut sur une colonie isolée de 30 000 colons, la chute **émerge** : 571 bâtiments sur 574 tombent en 30 min sans arme ; avec 5 % de miliciens, la colonie perd 29 bâtiments puis tient ; avec 20 %, aucun zombie n'atteint une porte. Elle n'a demandé aucune individualisation des occupants. Conservation exacte, une entité zombie par zombifié.
- **Décision** : **pas d'âge** : la part des enfants et des personnes âgées n'est pas modélisée, tous les colons ont les mêmes règles. Le temps ne passe pas pour la population : ni vieillissement, ni naissances, ni morts de vieillesse ; elle ne change que par les débarquements, les morts et les zombifications.
- **Décision** : une **colonie n'est pas un objet du jeu** : ce sont des colons logés dans des bâtiments, sur la surface de communes. L'unité d'approvisionnement est la **commune** : sa **mairie** sert de **dépôt** pour toutes les ressources de ses colons (vivres, eau potable, médicaments, vêtements chauds, couvertures, munitions), et chaque bâtiment de la commune y puise directement, sans transport simulé ([mairies.md](game-elements/locations/mairies.md)). Les soldats et les ouvriers logés dans la commune y puisent aussi, munitions comprises ; une unité en opération hors d'un bâtiment, elle, n'est ravitaillée que par sa chaîne logistique. Dépôt de capacité illimitée ; inaccessible tant que la mairie est occupée par des zombies. La mairie n'est que le dépôt **par défaut** : chaque bâtiment puise chaque ressource au **dépôt construit le plus proche** qui la garde ([dépôt de munitions, dépôt de carburant](game-elements/constructions/CONSTRUCTIONS.md#logistique-et-soutien)), dans sa commune ou une commune voisine ; à défaut, à la mairie de sa commune. Les vivres, l'eau potable, les médicaments, les vêtements chauds et les couvertures restent à la mairie. Le stock ne change pas de dépôt tout seul : le joueur le transporte, comme toute cargaison. Une commune voisine est une commune qui **touche** celle du bâtiment (graphe d'adjacence précalculé par la chaîne de données). Un dépôt vide ou occupé par des zombies est sauté : le bâtiment puise au suivant qui a du stock et n'est pas occupé, puis à la mairie.
- **Statut** : Décidé.

### A-16 — Gestion des corps

- **Problème** : chaque zombie abattu laisse un corps, soit des millions sur une partie. Les corps encombrent les rues et les emplacements à construire, et causent des maladies s'ils restent sur place (A-17).
- **Décision** : les corps des **humains** morts (combat, maladie) sont traités comme ceux des zombies.
- **Décision** : le corps d'un zombie abattu n'est **pas contagieux** au contact (pas de zombification) : c'est seulement un **risque sanitaire** (A-17).
- **Décision** : les corps doivent être **déplacés** avant de construire à leur place ([CONSTRUCTIONS.md](game-elements/constructions/CONSTRUCTIONS.md#décisions)).
- **Décision** : les corps sont **comptés, pas des entités** : une carte des corps par tile (un compteur par case de 10 m, environ 20 ko par tile), conservée telle quelle quand la tile redevient inerte (les rampes de corps restent en place ; A-01). À l'affichage, les corps sont générés à partir du compteur, avec une graine, comme les bâtiments. Analogie : un compteur d'occupation par bloc de disque, pas un inode par fichier.
- **Décision** : un corps **ne se décompose pas** : il compte jusqu'à ce qu'il soit traité.
- **Décision** : deux traitements seulement, la **fosse commune** et l'**incinération** ([CONSTRUCTIONS.md](game-elements/constructions/CONSTRUCTIONS.md#logistique-et-soutien)).
- **Décision** : **décontamination automatique** : le joueur pose un waypoint sur une zone, une équipe évacue et traite les corps seule (D-03). Tout le monde peut décontaminer, mais seules les [équipes sanitaires](game-elements/units/soutien/equipes-sanitaires.md) le font sans risque pour leur santé physique ; les autres risquent de tomber malades (A-17).
- **Décision** : manipuler et traiter les corps **baisse le moral** de ceux qui le font, différemment selon le traitement ; **personne ne s'y habitue**. Ne pas le faire cause des maladies.
- **Décision** : les corps entassés au pied d'un obstacle servent de **rampe** aux zombies (A-05).
- **Décision** : un corps est une **cargaison** (masse et volume) : porté à bras ou chargé dans un camion, vers une fosse commune ou un site d'incinération.
- **Décision** : la **fumée de l'incinération attire les zombies**, comme le bruit ([ZOMBIES.md](game-elements/zombies/ZOMBIES.md#perception)). Sa portée dépend du **vent** et de la **météo** (A-11). Elle attire aussi les zombies des tiles inertes voisines (migration, A-01).
- **Décision** : **aucune restriction de lieu** : le joueur peut envoyer n'importe qui évacuer des corps (ou construire) n'importe où, y compris sous les balles et au contact des zombies, à ses risques.
- **Décision** : la fosse commune et l'incinération ne diffèrent que par trois points. L'**incinération** pèse plus lourd sur le moral et fait de la fumée, mais ne se remplit jamais. La **fosse commune** pèse moins sur le moral et ne fait pas de fumée, mais se remplit.
- **Décision** : l'évacuation des corps fait partie du combat. Sans elle, les rampes de corps et la maladie gagnent ; avec elle, l'incinérateur attire de nouvelles hordes. C'est l'un des moteurs de la boucle de reconquête ([README.md](README.md#boucle-de-reconquête)).
- **Statut** : Décidé.

### A-17 — Maladies

- **Problème** : soldats, ouvriers et colons vivent dehors, dans le froid, au milieu des corps, avec une eau et une hygiène incertaines. Les maladies doivent peser sur la main-d'œuvre sans demander de simuler chaque microbe.
- **Décision** : le **froid** rend malade ; les **corps non traités** aussi (A-16), ainsi que la décontamination faite par d'autres que les équipes sanitaires.
- **Décision** : un malade se soigne avec des **médicaments** ([RESOURCES.md](game-elements/resources/RESOURCES.md#santé)) ; sans médicaments, il a une **chance de mourir**. Un ouvrier ou un civil malade **ne peut pas travailler** ; un soldat malade est **diminué**, pas inapte.
- **Décision** : une **carte de risque sanitaire** par tile, champ scalaire **grossier** (cases de 50 à 100 m) et **lent** (mis à jour chaque minute de jeu). Elle additionne les sources locales : corps non traités, eaux usées, eau non potable consommée, surpopulation des logements. Un léger flou fait déborder le risque autour de chaque source. Analogie : une carte de chaleur des vulnérabilités, recalculée périodiquement à partir de l'inventaire. Coût négligeable.
- **Décision** : **pas de contagion** entre personnes ; **pas de prévention** (ni vaccins, ni constructions d'hygiène). On agit sur les causes : corps, eau, logement, froid.
- **Décision** : **une seule maladie générique**.
- **Décision** : le froid ne tue que **par la maladie** (pas d'hypothermie simulée).
- **Décision** : les **vêtements chauds** et les **couvertures** protègent du froid ([RESOURCES.md](game-elements/resources/RESOURCES.md#habillement)).
- **Décision** : le **froid est calculé par personne**, pas stocké dans la carte : il dépend de la météo du lieu, de l'abri (chauffé ou non) et de l'équipement (vêtements chauds, couvertures). Chance de tomber malade = risque de la carte au lieu où l'on se trouve × facteur de froid de la personne × faim et fatigue, tirée à la mise à jour des jauges (P-00 : une fois par seconde, ou moins).
- **Piste** : « malade » est un **champ d'état** (F-02), pas un composant marqueur.
- **Constat** (P-05, [résultats](https://github.com/fafapiti/prototypes/blob/main/p05-civils/README.md#4-hiver)) : la carte de risque par tile (cases de 100 m, une tile par tick, chacune une fois par minute) et le froid calculé par personne coûtent moins de 0,01 ms par tick. En hiver à −5 °C, sans chauffage, sur 100 000 colons et 7 jours :
  - sans vêtements chauds, couvertures ni médicaments : 5,2 % de morts et 17,5 % de main-d'œuvre disponible en moins ;
  - avec vêtements et couvertures, sans médicaments : 3 fois moins de morts ;
  - avec tout : aucune.
  Les corps non évacués font monter le risque jour après jour ; la boucle corps → maladie apparaît sans règle dédiée.
- **Statut** : Décidé (principes).

### A-18 — Chaîne sanitaire : blessures et usure

- **Problème** : tout contact avec un zombie zombifie en 10 secondes : les zombies ne font donc **aucun blessé**. Sans autre source, la chaîne sanitaire (évasan, hôpital de campagne), l'un des quatre axes logistiques du jeu ([README](README.md#cœur-du-jeu--les-quatre-chaînes-logistiques)), n'a presque rien à faire.
- **Décision** : les blessures sont **d'origine humaine et accidentelle** : éclats de nos propres tirs d'appui trop proches, tirs amis, accidents de véhicules, de chantier et de manipulation des corps, brûlures, effondrements. La **fatigue** en augmente le risque.
- **Décision** : **usure psychologique**, jauge lente distincte du moral. Elle monte avec l'exposition au combat, les pertes et la manipulation des corps, et ne baisse qu'**au repos en tile sûre**. Au-delà d'un seuil, le soldat devient **blessé psychique** : inapte, il faut l'évacuer. L'usure impose la **relève** des unités.
- **Décision** : un blessé psychique est **évacué vers une tile sûre** et inapte **environ un jour de jeu**. Il ne revient pas vite, mais bien plus vite qu'un blessé grave (amputé, par exemple). Il revient avec une usure **juste sous le seuil** : sans repos supplémentaire, il rechute vite.
- **Décision** : un **amputé** ne revient pas au combat : il est **réaffecté** à l'arrière. Il **reste militaire, mais non combattant**, et **perd son expérience**. Son nouveau poste doit être **pertinent pour son métier** : un infirmier de combat amputé travaille en hôpital de campagne, par exemple. Pas de rôle générique « réaffecté ». Sans poste pertinent (un mitrailleur, par exemple), il passe dans la catégorie **blessé de guerre** : sans utilité, c'est une **charge** que le joueur doit gérer.
- **Décision** : un **blessé de guerre** consomme vivres, eau potable, logement et médicaments comme un colon. Il peut être **rapatrié en outre-mer** par navire, où il occupe une place à bord. Sa présence **n'agit pas** sur le moral ni sur l'usure des autres.
- **Décision** : l'**usure** et l'**expérience** s'appliquent aussi aux **ouvriers** et aux **colons**. Par exception, leur expérience se gagne **par le travail** (chantiers, manipulation des corps) ; elle améliore leur **rendement** et leur **résistance à l'usure**.
- **Décision** : l'expérience suit la personne quand elle change de rôle : un colon qui devient ouvrier ou milicien, un milicien qui devient soldat régulier **gardent leur expérience**.
- **Décision** : la **perte du chef de groupe** augmente l'usure du groupe, en plus de baisser son moral.
- **Décision** : **délai de soin** : un blessé grave **meurt s'il n'est pas opéré à temps**. L'hôpital de campagne vaut donc par sa **distance au front**.
- **Décision** : **expérience individuelle** des soldats : elle améliore la précision et la résistance à l'usure. Elle ne se gagne **qu'au combat**. Un blessé soigné vaut donc plus qu'une recrue. En début de partie, **tout le monde est sans expérience**, forces spéciales et Légion comprises.
- **Décision** : la Légion étrangère résiste à la **débandade** ; son trait ne touche pas l'usure.
- **Décision** : un humain mort **sans contact avec un zombie** (blessé mort de ses blessures, par exemple) **ne se relève pas** ([ZOMBIES.md](game-elements/zombies/ZOMBIES.md#infection)).
- **Décision** : le **manque de sommeil** n'est pas simulé à part : la jauge de fatigue le couvre.
- **Statut** : Décidé (principes).

### A-19 — Suivi des zombies par commune

- **Problème** : le statut des communes est cosmétique, mais une commune passe au jaune dès qu'un zombie revient sur sa surface, y compris par migration entre tiles inertes ([mairies.md](game-elements/locations/mairies.md)). Or une tile inerte ne stocke qu'un nombre de zombies, et les limites de communes ne suivent pas la grille des tiles : une tile de 1 km est souvent à cheval sur deux ou trois communes.
- **Décision** : découper chaque tile en fragments tile × commune à la préparation de la carte ; une tile inerte stocke un compteur par fragment, et le journal de migration note le fragment d'arrivée (le plus proche du bord d'entrée). Une commune est sans zombie quand tous ses fragments sont à zéro : un compteur par commune, mis à jour à chaque arrivée ou départ, suffit pour l'affichage. Coût négligeable en mémoire.
- **Constat** (P-07, [résultats](https://github.com/fafapiti/prototypes/blob/main/p07-chaine-manche/README.md)) : 59 % des tiles terrestres de la Manche ont plus d'un fragment de commune (jusqu'à 5). Les fragments coûtent 0,4 ko par tile (liste et carte à 2 m) ; le voisinage des communes (1 338 paires dans la Manche) et la mairie de chaque commune tiennent dans un fichier de 11 ko par département.
- **Piste** (P-07) : répartition initiale au prorata des habitants des bâtiments de chaque fragment (carreaux de 200 m répartis sur les bâtiments résidentiels).
- **Décision** (P-07) : répartition initiale au prorata des habitants des bâtiments de chaque fragment (A-10).
- **Statut** : Décidé.

### A-20 — Rupture de barrage

- **Problème** : un [barrage](game-elements/locations/barrages.md) détruit noie la vallée en aval. Il faut propager une onde de crue sur des dizaines de kilomètres, à l'échelle 1:1, avec ses effets sur les humains, les zombies, les bâtiments, les ponts et les véhicules. Une simulation de fluide en temps réel sur cette surface est hors de portée.
- **Décision** : la carte d'inondation de chaque barrage est **précalculée** dans la chaîne de données : front de l'onde et hauteur d'eau au cours du temps, par un modèle d'écoulement en eaux peu profondes sur le RGE ALTI. En jeu, la crue est rejouée comme un front qui avance à vitesse réelle et active les tiles qu'il touche. Les plans particuliers d'intervention (PPI) des grands barrages peuvent servir de contrôle (à vérifier : accès public).
- **Décision** : le barrage a une **barre de vie** : plusieurs frappes de bombe guidée ou d'artillerie sont nécessaires pour le rompre. Il n'est pas reconstructible ; sa région reste dans le noir pour toujours.
- **Décision** : les zombies touchés par le front sont **dispersés** : relocalisés aléatoirement sur le **périmètre** de la surface de crue précalculée (pas tués). Humains, bâtiments, ponts et véhicules sur le passage sont **détruits** ; les corps des humains tués **disparaissent** (pas de cargaison à évacuer).
- **Piste** : le périmètre est précalculé avec la carte d'inondation, restreint à la zone accessible des tiles (pas de relocalisation dans l'eau ou dans un bâtiment) ; les tiles qui reçoivent des zombies s'activent ou reçoivent une entrée de journal de migration, selon leur état.
- **Statut** : Décidé.

### A-02 — Accélération du temps

- **Problème** : à l'échelle 1:1, les trajets sont longs (traversées de l'outre-mer, convois de plusieurs heures). L'accélération du temps envisagée au départ (×100) demandait 100 fois plus de calcul par seconde réelle, et ses grands pas de temps rendaient la simulation instable.
- **Constat** (P-01) : pendant un combat en ville dense, ×10 n'est tenable que pour une seule tile active ; avec une dizaine de tiles, l'accélération doit retomber vers ×1.
- **Décision** (abandonnée) : accélération de ×100 ou plus hors combat, avec retour automatique vers ×1 au premier contact. Les constats « à ×10 » ou « au budget de ×100 » de P-01 à P-05 datent de cette décision.
- **Décision** : **pas d'accélération**. Le monde est partagé par tous les joueurs sur un serveur unique ([H-01](#h-01--architecture--serveur-unique-faisant-autorité)) : une seule horloge, **temps réel**, en permanence. Les traversées entre l'outre-mer et la métropole durent de 10 jours à 6 semaines réelles (estimation à vérifier), un convoi Cherbourg–Brest plusieurs heures. Le monde tourne même quand personne n'est connecté : les unités doivent agir seules ([D-02](#d-02--ia-des-unités-amies), [D-03](#d-03--logistique-automatisée)).
- **Conséquence** : le budget du tick n'est plus divisé par l'accélération ni partagé avec le rendu. Il est fixé sur le serveur, à ×1 ([F-02](#f-02--performances-de-la-simulation)) ; le nombre de tiles actives tenables est à remesurer (P-14).
- **Constat** (P-02, [résultats](https://github.com/fafapiti/prototypes/blob/main/p02-perception/README.md)) : avec un arbre k-d pour la perception, ×10 tient pour 4 tiles calmes, 2 tiles en assaut et 1 tile dispersée.
- **Constat** (P-03, [résultats](https://github.com/fafapiti/prototypes/blob/main/p03-flow-fields/README.md)) : avec les flow fields et la vue de 400 m, ×10 tient pour 2 tiles calmes ou en repli, 1 tile dispersée ou en assaut. Tout calcul lourd d'une tile (champ, obstacle) doit être étalé sur plusieurs ticks : d'un seul tenant, il double le p99 d'une tile seule.
- **Constat** (P-04, [résultats](https://github.com/fafapiti/prototypes/blob/main/p04-activation/README.md)) : au budget de ×10, une activation étalée avec des budgets divisés par 4 (environ 75 ticks, soit 0,4 s de temps réel) n'ajoute que 0,3 à 0,4 ms au p99 en banlieue ; d'un seul tenant, elle coûte 12 à 14 ms, 5 fois le budget. Les budgets du chantier se divisent par le facteur d'accélération.
- **Constat** (P-06, [résultats](https://github.com/fafapiti/prototypes/blob/main/p06-origine-flottante/README.md#6-horloge-des-shaders-resultats-tempscsv)) : un temps de jeu en f32 absolu dans les shaders devient visible après 36 h de jeu (22 min de temps réel à ×100) et fige la plupart des frames après une semaine. Le `globals.time` de Bevy (modulo 3 600 s) saute à chaque bouclage pour toute animation dont la période ne divise pas 3 600 s. Des phases calculées sur le CPU depuis les ticks entiers sont exactes quelle que soit la durée de la partie.
- **Décision** (P-06) : le temps de jeu est un compteur de **ticks en u64** ; les shaders ne reçoivent que des **phases** calculées sur le CPU depuis ces ticks (`tick mod période`), jamais un temps absolu en f32. Exception : une **date relative** dans une fiche compacte (la dernière mise à jour d'un colon) reste en u32, comparée par **soustraction modulaire** (`wrapping_sub`), comme les numéros de séquence TCP : juste tant que l'écart entre deux mises à jour reste sous 6,8 ans.
- **Décision** : quand le calcul ne suit plus, le **temps de jeu ralentit** sous ×1 (×0,5, ×0,25…), comme un serveur qui freine ses clients pour ne pas tomber. Les combats ne sont jamais simplifiés. Le ralentissement touche **tous les joueurs à la fois** : c'est une soupape de sécurité, pas un régime normal.
- **Constat** (P-09, [résultats](https://github.com/fafapiti/prototypes/blob/main/p09-chemin-routier/README.md#5-charge-de-jeu)) : 1 000 convois dont 567 touchés par 100 coupures replanifient en 9 à 12 ms de temps réel en tâche de fond (15 ou 4 threads), soit environ une seconde de temps de jeu à ×100 ; dans la boucle, au budget de 3 ms par tick, il faudrait 14 ticks. La recherche de chemin routière ne pèse donc pas sur l'accélération, à condition de tourner hors du tick.
- **Statut** : Décidé (pas d'accélération, temps réel) ; Prototype (capacité du serveur, P-14).

### A-03 — Unités dispersées sur tout le territoire

- **Réponse partielle** : la simulation par tiles (A-01) et les tiles sûres (A-14) bornent le coût : seules les tiles au contact possible entre humains et zombies sont simulées finement.
- **Problème** : chaque unité du joueur crée une zone de simulation fine autour d'elle. Plus l'armée est dispersée (convois, garnisons, patrouilles), plus le coût augmente, même loin de la caméra.
- **Pistes** (sans auto-résolution) :
  - Mise en veille : une unité sans zombie ni événement dans ses zones de détection ne coûte presque rien ; elle se réveille dès qu'un stimulus apparaît.
  - Fréquence de simulation adaptative : les zones calmes sont mises à jour moins souvent, avec les mêmes règles.
  - Le modèle de combat probabiliste (A-04) réduit fortement le coût de chaque affrontement.
  - Parallélisme : chaque zone active est indépendante et peut tourner sur son propre cœur.
- **Décision** : pas d'auto-résolution. Toute unité du joueur est réellement simulée, où qu'elle soit. Il faut trouver d'autres solutions.
- **Exception** : le combat **à l'intérieur d'un bâtiment** est auto-résolu (B-02) : l'intérieur n'est pas simulé.
- **Décision** : **mise en veille** des zombies sans stimulus : ils sont simulés à fréquence réduite, avec les mêmes règles (perception et déplacement moins fréquents). Ils se réveillent dès qu'un stimulus apparaît ([ZOMBIES.md](game-elements/zombies/ZOMBIES.md#perception)).
- **Décision** : **veille profonde** des zombies loin de tout humain : hors de l'index spatial, déplacement une fois par seconde, sans collision entre zombies (A-01).
- **Constat** (P-02, [résultats](https://github.com/fafapiti/prototypes/blob/main/p02-perception/README.md)) : la veille profonde, testée par une carte de proximité des humains lue en O(1), endort 39 % des zombies d'une tile calme et gagne 23 % du tick. Le reliquat vient de la grille des zombies, dont le coût dépend du nombre de cases plus que du nombre de zombies.
- **Décision** : quand le calcul ne suit plus à ×1, le **temps de jeu ralentit** sous ×1 (×0,5, ×0,25…) plutôt que de simplifier les combats (A-02).
- **Constat** (P-09, [résultats](https://github.com/fafapiti/prototypes/blob/main/p09-chemin-routier/README.md#8-suivi-en-jeu)) : un convoi en tile inerte n'a besoin que de son itinéraire (liste d'arcs) et des durées cumulées : sa position à un instant donné se lit en 10 ns par recherche dichotomique, sans géométrie. En tile active, la géométrie vient des morceaux des tiles traversées (0,1 ms pour 80 km), en positions locales f32 exactes.
- **Statut** : Ouvert (contrainte forte).

### A-04 — Combat et blessures

- **Problème** : un zombie ne meurt que d'une destruction de la tête ; il faut donc savoir où chaque projectile touche. Des milliers de tireurs (mitrailleuses, artillerie) rendent le calcul balle par balle très coûteux.
- **Pistes** : balistique complète près de la caméra ; modèle probabiliste (chance de tir à la tête selon l'arme, la distance et l'entraînement) ailleurs. Explosions et fragmentation traitées par zone.
- **Décision** : modèle probabiliste partout (chance de toucher la tête selon l'arme, la distance, l'entraînement…). Les explosions fonctionnent par zone.
- **Statut** : Décidé.

### A-05 — Entassement et franchissement d'obstacles

- **Problème** : les zombies se marchent dessus pour franchir les obstacles. Une vraie physique de pile de corps est hors de portée.
- **Solution** : quand la densité contre un obstacle dépasse un seuil, l'obstacle devient franchissable (rampe virtuelle) ; les dégâts aux structures sont calculés de la même façon.
- **Décision** : seuil de densité contre un obstacle ; au-delà, l'obstacle devient franchissable, et les dégâts aux structures suivent la même logique.
- **Constat** (P-01) : une capacité par case de 4 m², avec admission progressive, coûte presque rien. Mais sans réservation, plusieurs zombies entrent dans la même case au même tick : la densité locale atteint 105 zombies sur 4 m² au point de contact. Le seuil d'entassement demandera une admission exacte, au moins sur les cases au contact.
- **Décision** (remplacée par P-03) : réservation exacte des places sur les cases au contact d'un obstacle ou d'humains ; admission progressive ailleurs.
- **Constat** (P-03, [résultats](https://github.com/fafapiti/prototypes/blob/main/p03-flow-fields/README.md#réservation-des-places-resultatscsv-section-d)) : l'admission progressive laisse monter la densité jusqu'à 301 zombies sur 4 m² en assaut. La réservation au contact ne la borne pas : les humains bougent, et une case remplie avant d'être au contact le devient ensuite (55 au contact, 134 ailleurs). La réservation **partout**, déterministe (demandes triées puis admises case par case), tient exactement la capacité sur toutes les cases pour 3 à 4 % du tick, autant que la réservation au contact. Dans les flow fields, une barricade est une case chère mais franchissable : la foule s'entasse devant elle quand le détour est long.
- **Décision** (P-03) : **réservation exacte des places sur toutes les cases** ; l'admission progressive disparaît.
- **Décision** (P-03) : dans les flow fields, chaque obstacle a un **coût de franchissement** qui lui est propre (barbelés faible, mur et gravats élevés ; 40 fois une rue pour une barricade de référence, à régler à l'équilibrage, D-06). La foule attaque l'obstacle quand le détour est plus long que ce coût.
- **Décision** (P-03) : on ne construit **ni sur des zombies, ni sur des corps** : la place doit d'abord être dégagée ([A-16](#a-16--gestion-des-corps)).
- **Décision** : les **corps** des zombies abattus au pied d'un obstacle s'y accumulent et servent de **rampe** aux suivants. Comme ils ne se décomposent pas, la rampe reste tant qu'ils ne sont pas évacués ([A-16](#a-16--gestion-des-corps)).
- **Statut** : Décidé.

### A-06 — Feu

- **Problème** : le feu doit être simulé objet par objet (chaque arbre, chaque objet), y compris pour des incendies de plusieurs dizaines de km². Les forêts françaises comptent plusieurs milliards d'arbres.
- **Solution** : seules les zones en feu sont simulées finement (automate cellulaire et objets) ; le reste est un état stocké (intact / brûlé), sans coût tant que rien ne s'y passe. Interaction avec le vent (météo) et avec les zombies en feu.
- **Décision** : simulation fine uniquement dans les zones en feu ; état stocké ailleurs.
- **Statut** : Décidé.

### A-07 — Recherche de chemin

- **Problème** : des véhicules et des soldats se déplacent sur le réseau routier réel (des millions de tronçons), avec ponts à charge limite, épaves et destructions qui modifient le réseau.
- **Solution** : recherche de chemin hiérarchique (contraction hierarchies ou équivalent) mise à jour quand le réseau change ; flow fields pour les hordes.
- **Décision** : recherche de chemin hiérarchique mise à jour à chaque modification du réseau ; flow fields pour les hordes.
- **Prototype P-03** ([résultats](https://github.com/fafapiti/prototypes/blob/main/p03-flow-fields/README.md)) : sur les tiles réelles, un champ de distance à **2 m** (règle de la majorité, coût plus élevé le long des murs) rend le mouvement crédible : 0,07 % de bloqués hors foule et course à pleine vitesse, contre 65 % de bloqués en ligne droite ; 1 m n'apporte rien pour 4 fois le coût, 4 à 5 m bloquent 2 à 3 % des zombies et effacent 6 à 9 % des rues. Un champ coûte 2,7 ms sur un thread, quel que soit le nombre de sources, et 1 Mo.
- **Décision** (P-03) : flow fields à **2 m**, règle de la majorité, coût plus élevé le long des murs ; ligne droite vers la cible sous 15 m seulement.
- **Décision** (P-03) : **deux champs par tile**, à sources multiples : « proies » (humains observés dans la dernière minute, à leur position observée, ce qui applique la règle de la dernière position connue) et « bruits ». Recalcul des proies à **1 Hz, étalé** sur la seconde en double tampon (écart de 0,5 % avec un champ frais). Après un obstacle, **recalcul partiel** exact s'il invalide peu de cases (17 fois plus rapide pour une rue barrée), sinon calcul complet étalé. Entre tiles, **échange des bordures** à chaque recalcul, valeurs **horodatées** comme l'alerte relayée : exact en 5 recalculs, et une source disparue s'efface en un seul (sans horodatage : comptage à l'infini).
- **Constat** (P-05, [résultats](https://github.com/fafapiti/prototypes/blob/main/p05-civils/README.md#2-survol-resultats-survolcsv)) : la marche des colons sous la caméra, par A* sur une grille de 2 m, coûte 1,4 ms par chemin en moyenne et jusqu'à 9,4 ms ; d'un seul tenant, un chemin long fait passer un tick à 10-12 ms. Pour 7 à 9 % des portes, aucun chemin n'est trouvé dans la limite d'expansions.
- **Décision** (P-05) : le chemin à pied d'un colon est un **A\* reprenable** : il s'interrompt à la fin de son budget d'expansions et reprend au tick suivant, comme un calcul lourd étalé (F-02).
- **Décision** (P-05) : un colon sans chemin à pied vers sa destination fait un **trajet abstrait** (invisible, durée calculée), comme hors de la caméra, mais **seulement en tile sûre**. En tile active, un colon sans chemin à pied **reste dans son bâtiment**.
- **Constat** (P-09, [résultats](https://github.com/fafapiti/prototypes/blob/main/p09-chemin-routier/README.md)) : sur le réseau carrossable de la Manche recousu depuis les tiles (153 000 nœuds, 181 000 arêtes) et sur Paris (23 000 nœuds), les **CH personnalisables (CCH)** répondent en 35 à 60 µs en médiane (100 à 140 µs au p99), chemin complet compris, **exactes** sur 10 000 paires et quatre classes de véhicules ; Dijkstra en prend 7 à 10 ms, A\* et le bidirectionnel à peine moins, ALT 1 à 2 ms. Les CH classiques sont un peu plus rapides en requête (30 µs), mais leur prétraitement se refait **par classe** et à **chaque modification** (0,3 à 2,8 s pour la Manche). La CCH sépare la topologie (ordre par dissection emboîtée : 4,5 s, une fois, dans la chaîne) des métriques (personnalisation : 8 ms par classe sur 16 threads, 20 ms sur un). Une coupure se répercute par **personnalisation partielle** (écrite dans le prototype) en 0,1 à 0,9 ms, 1 000 coupures en 11 ms, identique bit à bit à une personnalisation complète. Analogie : la topologie OSPF se calcule une fois, seuls les coûts des liens changent.
- **Constat** (P-09) : la BD TOPO 3 n'a pas de table de nœuds : les intersections sont les extrémités communes des tronçons. Les morceaux des tiles se recousent exactement par (identifiant, rang) : longueur égale à celle de P-07 (écart de 2 × 10⁻⁶, quantification au 1/16 m), aucun doublon ni rupture, même graphe quel que soit le découpage (tiles, pseudo-départements), le nombre de threads ou l'ordre de lecture.
- **Constat** (P-09) : à Paris, respecter tous les sens uniques allonge les trajets de 11 % en moyenne et rend 3 % des paires impossibles ; dans la Manche, l'effet est négligeable (0,5 %).
- **Décision** (P-09, [décisions](https://github.com/fafapiti/prototypes/blob/main/p09-chemin-routier/README.md#décisions)) : **CCH** sur un **graphe national unique**. La chaîne calcule l'ordre (dissection emboîtée) et livre un fichier du graphe ; le jeu reconstruit la structure et personnalise une **métrique par classe d'itinéraire** au chargement, en tâche de fond. Une modification du réseau passe par une **personnalisation partielle** ; seuls les convois dont l'itinéraire restant emprunte une arête coupée replanifient, **en tâche de fond**, jamais dans le tick.
- **Décision** (P-09) : le **sens de circulation** n'est respecté que sur les **chaussées séparées** (autoroutes, routes à 2 chaussées, bretelles) ; les sens uniques urbains, les ronds-points et les interdictions de tourner sont ignorés : il n'y a plus de code de la route.
- **Décision** (P-09) : une **nouvelle traversée** (pont du génie hors d'un pont existant) ajoute une arête : la structure CCH est reconstruite avec le même ordre, en tâche de fond (60 ms pour la Manche). Poser un pont du génie sur un pont détruit n'est qu'un changement de métrique.
- **Statut** : Décidé.

### A-08 — Eau et franchissement des cours d'eau

- **Problème** : les zombies marchent au fond des cours d'eau. Il faut connaître la profondeur, que les données ne fournissent pas pour les rivières.
- **Voir aussi** : G-02 (la mer).
- **Décision** : pas d'estimation : profondeur arbitraire pour les cours d'eau (valeur à fixer).
- **Statut** : Décidé.

### A-09 — Logistique et production

- **Problème** : chaque objet a une masse et un volume, et les palettes et conteneurs sont inspectables. Toute la chaîne (matières premières → production → transport → consommation) est simulée.
- **Pistes** : regrouper les objets identiques en lots (une caisse de 1 000 cartouches, pas 1 000 objets) ; détailler à l'inspection seulement.
- **Décision** : objets identiques regroupés en lots, avec une mécanique « emballer / déballer » : on emballe des objets en caisses, palettes ou conteneurs pour le transport, et on les déballe à l'arrivée ou pour les inspecter.
- **Statut** : Décidé.

### A-10 — Population civile

- **Problème** : il faut répartir les survivants et les zombies au début de la partie, de façon crédible.
- **Décision** : la population de métropole est **entièrement zombifiée**. Seules subsistent de rares **poches isolées** de survivants (moins de 1 000 personnes chacune) : ce sont des objectifs de jeu, **connus dès le début**. Leur liste est dans [survivants.md](game-elements/locations/survivants.md).
- **Décision** : environ **10 millions d'humains en outre-mer**, dont environ 7 millions de réfugiés évacués de métropole avant l'effondrement. Le réalisme démographique n'est pas recherché.
- **Décision** : la population civile de métropole vient de la **colonisation** (voir A-15).
- **Décision** : la densité de zombies est **proportionnelle à la population** : le nombre initial de zombies par tile est tiré des données INSEE carroyées (carreaux de 200 m et de 1 km, librement réutilisables), moins les évacués.
- **Abandonné** : survivants regroupés dans Paris intra-muros, derrière le périphérique (remplacé par la colonisation ; voir A-15).
- **Constat** (P-07, [résultats](https://github.com/fafapiti/prototypes/blob/main/p07-chaine-manche/README.md)) : répartition dasymétrique des carreaux de 200 m sur les bâtiments résidentiels, au prorata du plancher, par la méthode du plus fort reste : somme exacte, indépendante de l'ordre, identique pour deux départements voisins. 1 % des habitants de la Manche tombent dans un carreau sans bâtiment et sont posés sur des points. Filosofi compte 1,5 % (Manche) à 3,4 % (Paris) d'habitants de moins que la population légale (sans les communautés, à vérifier).
- **Piste** (P-07) : la chaîne stocke les **habitants** par fragment ; le nombre de zombies (habitants moins les évacués) se calcule à la création de la partie.
- **Décision** (P-07) : les habitants initiaux viennent des **carreaux de 200 m** de l'INSEE (Filosofi), répartis sur les bâtiments résidentiels au prorata du plancher (plus fort reste), puis **complétés, commune par commune, jusqu'à la population légale** (population des communautés absente de Filosofi), avec la même répartition. La chaîne stocke les habitants ; les zombies (habitants moins les évacués) se calculent à la création de la partie.
- **Statut** : Décidé.

### A-11 — Météo et marées

- **Problème** : un cycle météo et un cycle de marées conditionnent les débarquements et le feu (vent).
- **Pistes** : météo générée de façon déterministe à partir de normales climatiques ; marées calculées par harmoniques à partir de données publiques (SHOM, sous réserve de licence).
- **Décision** : météo générée de façon déterministe à partir de normales climatiques ; marées calculées par harmoniques (licence des données SHOM à vérifier, voir B-06).
- **Statut** : Décidé.

### A-12 — Déterminisme et multijoueur

- **Problème** : le choix d'un multijoueur, et la façon de le faire (simulation déterministe en lockstep, ou serveur faisant autorité), change toute l'architecture. Le déterminisme est aussi utile pour rejouer une partie et déboguer.
- **Décision** : le jeu est **multijoueur seulement**, sur un **serveur unique faisant autorité** ([H-01](#h-01--architecture--serveur-unique-faisant-autorité)). Pas de lockstep : il est impossible avec des clients navigateur anonymes qui arrivent et partent à tout moment. Le client ne simule rien : il affiche l'état reçu et envoie des ordres.
- **Conséquence** : le déterminisme n'est plus nécessaire au multijoueur ; il reste utile pour rejouer et déboguer (P-18), et la chaîne de données doit rester déterministe (P-07).
- **Constat** (P-09, [résultats](https://github.com/fafapiti/prototypes/blob/main/p09-chemin-routier/README.md#7-déterminisme)) : le graphe routier, l'ordre CCH, les sept métriques et les itinéraires (CCH et Dijkstra) ont les mêmes empreintes à 1 et 16 threads, dans trois ordres de lecture des tiles, sur deux exécutions. Temps en millisecondes entières, égalités départagées par identifiant ; aucune fonction de la bibliothèque mathématique (seulement `sqrt`).
- **Statut** : Décidé (serveur faisant autorité) ; Piste (déterminisme pour le débogage).

### A-13 — Sauvegardes

- **Problème** : le monde entier peut être modifié (bâtiments détruits, forêts brûlées, constructions, objets déplacés).
- **Piste** : monde de base régénéré de façon déterministe à partir des données ; la sauvegarde ne contient que les différences.
- **Décision** : les tiles qui ont été actives sont sauvegardées entièrement (zombies, corps, constructions) et stockées sur disque ; elles font partie de la sauvegarde (A-01).
- **Constat** (P-04, [résultats](https://github.com/fafapiti/prototypes/blob/main/p04-activation/README.md#banc-tile-par-tile-un-thread-resultats-banccsv)) : un instantané **normalisé** (positions au 1/64 m, cap au 1/256 de tour, tous errants, liste triée), compressé en **zstd**, fait environ **4 octets par zombie** : 51 ko pour une tile dense de 12 300 zombies, corps et constructions compris. Instantané 0,6 ms, encodage 0,25 ms, écriture 0,1 ms en tâche de fond (`fsync` : 5 à 15 ms). Ordre de grandeur : 1 000 tiles visitées font 50 Mo. Deux états identiques donnent deux fichiers identiques.
- **Décision** (P-04) : ce format pour les tiles, avec une somme de contrôle vérifiée à la lecture, **un fichier par tile**, écrit en tâche de fond.
- **Décision** (P-04) : les fichiers de tiles sont **immuables** (une nouvelle version est un nouveau fichier) et écrits **sans `fsync`** pendant la partie. Une **sauvegarde de partie** est un **manifeste** (liste des versions de tiles et état global) plus des **liens physiques** vers les fichiers de tiles, avec un `fsync` à ce moment-là seulement. Rien n'est recopié ; un fichier disparaît quand plus aucune sauvegarde ni la partie en cours ne le référence. Un crash peut perdre les dernières désactivations, jamais une sauvegarde de partie. Analogie : un instantané copy-on-write d'un système de fichiers.
- **Décision** : les sauvegardes de partie sont sur le **même système de fichiers** que la partie en cours, qui doit gérer les liens physiques. Sinon (autre disque, FAT32 ou exFAT, clé USB), la sauvegarde **est refusée**, avec un message qui en donne la raison ; pas de copie de repli.
- **Constat** (P-05, [résultats](https://github.com/fafapiti/prototypes/blob/main/p05-civils/README.md#sauvegarde-resultats-sauvegardecsv)) : les dossiers des colons, normalisés (triés par identifiant, écarts en varint) et compressés en zstd, font environ **16 octets par colon** sans perte, 173 ko pour une tile médiane.
- **Décision** (P-05) : les colons sont sauvegardés **sans perte** dans le fichier de leur tile, au même format que les zombies (somme de contrôle vérifiée à la lecture).
- **Constat** (P-06, [résultats](https://github.com/fafapiti/prototypes/blob/main/p06-origine-flottante/README.md#9-simulation-selon-le-repère-resultats-simucsv-resultats-derivecsv-resultats-sauvegardecsv)) : au 1/64 m, une position coûte 3,2 octets compressés en tile + position locale (comme en grille absolue), 4,9 en ECEF. Les f32 exacts coûtent 6,2 octets : c'est le prix d'une sauvegarde de partie dont le rechargement reproduit exactement la partie continuée (au 1/64 m, les positions des tiles actives sont arrondies au rechargement, et la partie diverge).
- **Décision** (P-06) : dans une sauvegarde de partie, les positions des **tiles actives** sont en **f32 exacts** (environ 6 octets) ; un rechargement reproduit exactement la partie continuée. Les tiles inertes restent au 1/64 m (instantané normalisé de P-04).
- **Constat** (P-07, [résultats](https://github.com/fafapiti/prototypes/blob/main/p07-chaine-manche/README.md)) : la chaîne de données est **déterministe** : mêmes octets dans les paquets de tiles à 1, 4, 8 et 16 threads et d'une campagne à l'autre (altitudes en centimètres entiers, listes triées, union-find à racine minimale, plus fort reste). Le monde de base peut donc être régénéré à l'identique et une sauvegarde ne garder que les différences, à condition de figer l'édition des sources et la version de zstd. Réserve : les projections utilisent la bibliothèque mathématique du système (P-18 pour les autres plateformes).
- **Constat** (P-08, [résultats](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md)) : la génération des bâtiments est déterministe : mêmes octets de maillage sur 1, 4 et 16 threads, dans l'ordre direct, inverse ou mélangé, sur deux exécutions, sans aucune fonction de la bibliothèque mathématique du système (entiers, `+ − × ÷`, `sqrt`). La sauvegarde des bâtiments modifiés (identifiant, état, barre de vie) fait 46 octets par tile dense pour 1 % de bâtiments modifiés, 1,4 à 2,4 ko si tous le sont ; le volume et les gravats se régénèrent.
- **Décision** (P-08, [décisions](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md#décisions)) : les bâtiments ne sont pas sauvegardés ; seuls les **bâtiments modifiés** le sont (identifiant, état, barre de vie), dans le fichier de leur tile.
- **Constat** (P-09, [résultats](https://github.com/fafapiti/prototypes/blob/main/p09-chemin-routier/README.md#8-suivi-en-jeu)) : une modification du réseau se désigne de façon stable par (identifiant BD TOPO du tronçon, rang de début, état) : 3 octets compressés par modification. Un itinéraire restant fait environ 700 octets compressés pour 80 km.
- **Décision** (P-09) : la sauvegarde garde la **liste des modifications du réseau** (arêtes coupées, ponts détruits, ponts du génie) et l'**itinéraire restant** de chaque unité en mouvement ; graphe et métriques se régénèrent. Un itinéraire n'est pas recalculé au chargement : un convoi qui ne replanifie pas après une réouverture garderait sinon un autre chemin.
- **Décision** (serveur unique) : le joueur ne sauvegarde ni ne recharge rien. Une **sauvegarde de partie** devient un **instantané du monde** pris par le serveur, à intervalle régulier (fréquence à décider) ; c'est de l'exploitation, comme la sauvegarde d'une base de données. Le schéma manifeste + liens physiques s'applique tel quel ; le volume persistant du serveur doit donc gérer les liens physiques (ext4 ou XFS local, pas un partage réseau).
- **Conséquence** : les tiles étant écrites sans `fsync`, un arrêt brutal du serveur ramène le monde au **dernier instantané**. Sa fréquence fixe la perte maximale.
- **Décision** (recommandation adoptée par défaut, révisable) : un instantané **toutes les 5 minutes**. Il ne recopie rien (liens physiques et manifeste) : son coût est le `fsync` des tiles écrites depuis le précédent, à mesurer sur les disques durs chiffrés du serveur (P-19).
- **Décision** : une **mise à jour du serveur remet le monde à zéro** ([H-01](#h-01--architecture--serveur-unique-faisant-autorité)). Un instantané n'est donc relu que par la version qui l'a écrit : il ne sert qu'à reprendre après un arrêt ou un redémarrage (panne, redémarrage du nœud). Aucune migration de format n'est jamais nécessaire.
- **Constat** (serveur) : le volume racine (XFS sur LUKS, 15 To) est en **RAID 0** sur les deux disques durs de 8 To : il n'est pas redondant. La perte d'un disque perd le monde ; vu la décision précédente, elle équivaut à une remise à zéro.
- **Statut** : Décidé (tiles, instantanés du monde toutes les 5 minutes, bâtiments modifiés) ; Piste (autres différences du monde de base).

## B. Monde et données

### B-01 — Chaîne de traitement des données IGN

- **Problème** : convertir la BD TOPO (bâtiments avec hauteur, routes, ponts, végétation, hydrographie) et le RGE ALTI (relief, 1 à 5 m) en tuiles de jeu, pour la métropole et les outre-mer. Volume de plusieurs dizaines de Go en entrée.
- **Décision** (P-02) : la chaîne calcule pour chaque tile sa **part de bâti**, qui fixe la distance de vue des zombies et le seuil de veille profonde de la tile ([A-01](#a-01--nombre-de-zombies--simulation-par-tiles)). Le jeu ne la recalcule pas.
- **Constat** (P-04, [résultats](https://github.com/fafapiti/prototypes/blob/main/p04-activation/README.md#chaîne-de-données--zone-accessible-resultats-precalculcsv)) : la **zone accessible** des zombies (plus grande zone de rues connexe, où ils sont placés à l'activation) doit être calculée sur tout le territoire d'un coup. Tile par tile, elle perd 0,8 % des rues d'un bloc parisien et jusqu'à 14 % d'une tile, et coûte 45 à 60 ms à l'activation. Sur le bloc entier : 2,5 s pour 55 tiles, 26 à 46 ko par tile urbaine dense. Le terrain des flow fields, lui, ne se stocke pas (141 ko par tile pour 6 ms de calcul étalé).
- **Décision** (P-04) : la chaîne précalcule la **zone accessible** sur tout le territoire, et la stocke par tile dans les **données distribuées**. Sa taille hors des villes denses reste à mesurer.
- **Prototype P-07** ([résultats](https://github.com/fafapiti/prototypes/blob/main/p07-chaine-manche/README.md)) : la chaîne en Rust transforme la BD TOPO, le RGE ALTI et les carreaux INSEE de la Manche en 14 076 tiles (dont 7 808 de mer à 30 km) en 66 s sur la machine de référence, avec 11 Go de mémoire. Le relief est lu en flux dans l'archive 7z de l'IGN (6 701 dalles, 43 Go d'ASC, aucune place disque temporaire) ; seule la BD TOPO doit être décompressée (2,5 Go pour la Manche). Une tile fait 60 ko en rural, 120 ko en ville, 210 ko en centre dense ; le relief en fait 80 % en rural. Mêmes octets de 1 à 16 threads.
- **Constat** (P-07) : pour la France entière (hors Pacifique), environ 800 000 tiles, **35 à 41 Go** (zstd 19 ou 9), **1,9 h** de calcul sur la machine de référence. À l'installation, il faudrait télécharger **594 Go** de sources (BD TOPO 32 Go, RGE ALTI 1 m 561 Go), ou 71 Go en se contentant d'un relief de 5 m, plus 2 h de calcul et 11 Go de mémoire.
- **Constat** (P-07) : l'extraction départementale de la BD TOPO a une **zone tampon d'environ 5 km** chez les voisins. Avec une règle d'appartenance commune (la tile appartient au département qui contient son centre), on traite les départements un par un, sans les recoudre : la zone tampon sert de halo.
- **Piste** (P-07) : **traitement avant distribution**, par le projet (intégration continue, à chaque édition de la BD TOPO), un processus par département ; tiles rangées en paquets de 16 × 16 avec index, sections zstd séparées (niveau 19), empreinte BLAKE3 par tile.
- **Piste** (P-07) : contenu d'une tile : en-tête (part de bâti, habitants), relief (nœuds de **2 m**, bords compris, quantifiés à 10 cm : 48 ko, 3 cm d'erreur RMS ; 1 m : 150 ko ; 5 m : 12 ko mais talus et digues effacés), bâtiments possédés, franchissable de 2 m (murs et passages compris), cases hors zone, réseau coupé aux bords, ponts, occupation du sol à 2 m, fragments de communes, lieux. **Précalculés** : zone accessible, cours et passages, habitants par fragment, mairies, voisinage des communes. **Recalculés au chargement** : bâti de 1 m (0,1 à 5 ms), capacités, arbres (0,5 ms), terrain des flow fields.
- **Décision** (P-07, [décisions](https://github.com/fafapiti/prototypes/blob/main/p07-chaine-manche/README.md#décisions)) : traitement **avant distribution**. Le projet lance la chaîne (en Rust, un processus par département, la zone tampon de l'IGN servant de halo) à chaque édition de la BD TOPO ; le résultat (35 à 41 Go pour la France hors Pacifique) est installé sur le **serveur**.
- **Décision** (jeu en navigateur) : plus d'installation. Le client télécharge les tiles **à la demande**, selon la position de la caméra, depuis le serveur du jeu ; il les garde dans un cache persistant du navigateur ([H-06](#h-06--diffusion-des-tiles)).
- **Décision** (P-07) : **format** : paquets de 16 × 16 tiles avec index ; par tile, un en-tête (part de bâti, habitants, terre ou mer), des sections compressées séparément en **zstd niveau 19**, une empreinte BLAKE3. Les tiles de pleine mer (à moins de 30 km d'une côte, sans terre) n'ont **aucun fichier** : une tile absente est de la mer.
- **Décision** (P-07) : **relief** en nœuds de **2 m**, bords compris, quantifiés à **10 cm**.
- **Décision** (P-07) : **précalculés** par la chaîne : zone accessible, cours fermées et passages percés, franchissable de 2 m, habitants par fragment de commune, mairies, voisinage des communes, part de bâti. **Recalculés au chargement** : bâti de 1 m, capacité des bâtiments, arbres, terrain des flow fields.
- **Décision** (P-07) : la **mer** est définie par le trait de côte des plus hautes eaux (BD TOPO, `limite_terre_mer`) et l'absence de relief, pas par les limites des communes ; les **estrans** sont de la terre dans les tiles (la marée les recouvre en jeu, A-11). Les îles étrangères proches (Jersey, Guernesey, îles des Antilles) sont **grisées**, comme le reste du monde (G-01). À vérifier dans la chaîne : P-07 utilisait encore les communes.
- **Constat** (P-08, [résultats](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md)) : la section `bati` manque de la **hauteur du toit** (altitude maximale moins minimale du toit, BD TOPO : connue pour 76 % des bâtiments de la Manche, 31 % à Paris et petite couronne), qui donne la forme du toit, et des **matériaux** des murs et du toit (la moitié des bâtiments). Avec la porte (B-02), environ 3 octets par bâtiment : +8 à 16 % de la section, environ 0,2 Go pour la France. 49 emprises sur 720 000 sont **auto-sécantes** (aire faussée, donc capacité et population).
- **Piste** (P-08) : ajouter ces trois attributs au format ; nettoyer les emprises auto-sécantes dans la chaîne (`i_overlay`, en entiers).
- **Décision** (P-08, [décisions](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md#décisions)) : la section `bati` reçoit la **hauteur du toit** (dm), les **matériaux** des murs et du toit (un octet chacun) et la **porte** (rang de l'arête extérieure, abscisse au 1/16 m), environ 3 octets par bâtiment ; la chaîne **nettoie les emprises auto-sécantes**.
- **Constat** (P-09, [résultats](https://github.com/fafapiti/prototypes/blob/main/p09-chemin-routier/README.md)) : reconstruire le graphe routier depuis les tiles prend 1,4 s pour la Manche (environ 2 min pour la France) ; le livrer coûte 2,2 Mo compressés pour la Manche (graphe et ordre CCH), environ 0,25 Go pour la France, moins de 1 % des tiles. Il manque aux tiles le lien entre un pont et les tronçons qu'il porte (aujourd'hui retrouvé par la géométrie, 2 286 ponts associés sur 2 591 qui portent une route), et l'attribut « fictif » des tronçons.
- **Décision** (P-09, [décisions](https://github.com/fafapiti/prototypes/blob/main/p09-chemin-routier/README.md#décisions)) : la chaîne produit, après tous les départements, un **fichier national du réseau routier** (graphe carrossable, charge des ouvrages, ordre CCH). La section `ponts` reçoit les **tronçons portés**, le **franchissement** (tabliers d'une même traversée regroupés) et la **charge calculée** ; la section `reseau` reçoit le bit **fictif**. Les interdictions de tourner ne sont pas ajoutées.
- **Statut** : Décidé (P-07).

### B-02 — Génération des bâtiments

- **Problème** : chaque bâtiment est généré à partir de son emprise et de sa hauteur, de façon déterministe (même graine, même bâtiment), sauf les monuments.
- **Décision** : un bâtiment se comporte comme un **véhicule** : c'est un **contenant**. Les humains (soldats, civils) et les zombies peuvent y entrer et en sortir ; les humains tirent par les fenêtres. L'intérieur **n'est pas simulé** : pas d'étages, pas de pièces, pas de portes. Un combat entre humains et zombies dans un même bâtiment est **auto-résolu** (seule exception à la règle de A-03). Conséquence : pour le déplacement, la vue et les flow fields, un bâtiment reste un bloc plein, comme dans P-01 à P-03, dont les mesures restent valables.
- **Décision** : la **capacité** d'un bâtiment dépend de son emprise et de sa hauteur (BD TOPO).
- **Décision** : **auto-résolution** : à chaque seconde, chance de tuer un zombie ou d'exposer un humain, selon le nombre d'humains armés, leurs armes et le nombre de zombies. Un humain exposé devient zombie au bout de 10 s, comme dehors.
- **Décision** (P-05) : ces chances dépendent de l'**arme**, de la **munition** et du nombre de combattants (valeurs à définir) ; les munitions tirées sont **consommées** dans le stock de la **mairie de la commune** où se trouve le bâtiment ([mairies.md](game-elements/locations/mairies.md)).
- **Décision** : **sans munitions**, les occupants ne se battent plus : ils ne tuent plus de zombies, seuls les zombies agissent.
- **Décision** : **une seule porte** par bâtiment, même grand.
- **Décision** (P-05) : chaque bâtiment a une **porte**, générée avec lui (P-08). C'est par elle que les zombies entrent, et c'est elle qui représente le bâtiment occupé dans l'index des humains (A-15). En attendant P-08, les prototypes prennent la case accessible la plus proche.
- **Décision** : les zombies n'entrent dans un bâtiment **que s'il contient des humains**. Les humains à l'intérieur sont **perçus** (vus par les fenêtres, entendus quand ils tirent). **Tous** peuvent tirer par les fenêtres.
- **Décision** : quand un bâtiment n'a plus d'humains (morts, zombifiés ou sortis), ses zombies en **sortent aussitôt**. À l'activation d'une tile, tous les zombies sont placés **dans les rues**.
- **À décider** : liste des monuments modélisés fidèlement.
- **Constat** (P-01) : la BD TOPO ne contient ni porches, ni passages sous immeuble. Dans les tiles denses de Paris, 18 à 30 % de l'espace non bâti est une cour fermée, inaccessible depuis la rue.
- **Décision** : la génération **perce automatiquement un passage** entre chaque cour fermée et la rue la plus proche.
- **Décision** (P-03) : les passages de moins de 2 m de large (environ 1 % des rues, invisibles pour les flow fields de 2 m) sont **percés ou élargis** à la génération, comme les porches.
- **Décision** : chaque bâtiment a une **barre de vie** (tirs, frappes, souffle nucléaire). Seuls les bâtiments endommagés stockent la leur ; les autres sont intacts par défaut.
- **Décision** : un bâtiment endommagé garde sa capacité et sa protection. À zéro, il s'effondre : ses occupants meurent, et il laisse des **gravats** infranchissables pour les humains et les véhicules, que le génie doit dégager pour construire dessus ou traverser. Les zombies les franchissent en s'entassant, comme un mur. Les humains proches du bâtiment, hors occupants, peuvent être blessés.
- **Décision** : exception, une **mairie** effondrée est **reconstruite à l'identique** par le génie, pour redevenir le dépôt de sa commune, avec les règles d'une construction (matériaux, nombre minimum de bâtisseurs, durée fixe) ([mairies.md](game-elements/locations/mairies.md)).
- **Constat** (P-07, [résultats](https://github.com/fafapiti/prototypes/blob/main/p07-chaine-manche/README.md)) : la chaîne perce les cours fermées (au moins 16 m², passage de 60 m au plus à travers le bâti, plus court chemin) : 99,8 % des 32 721 cours de Paris et 89 % des 1 865 cours de la Manche ; il reste 0,18 % de la zone hors d'atteinte à Paris. Les murs de la BD TOPO ferment peu de cours. Les cases percées sont stockées avec la tile, pour que la génération (P-08) y place un porche.
- **Prototype P-08** ([résultats](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md)) : sur 720 092 bâtiments réels (Manche, Paris), un bâtiment est généré depuis son emprise réelle (trous compris), posé sur le relief de 2 m avec un soubassement de 3 m au plus ; hauteur de la BD TOPO avec un repli local (étages, sinon médiane de la tile pour le même usage et la même classe d'emprise : 0,2 m d'erreur médiane dans la Manche, 1,6 m à Paris, contre 2 et 11 m pour une constante) ; toit décrit comme une fonction affine par morceaux sur le rectangle englobant orienté (plat, deux pans, quatre pans, mansarde), choisi d'après la hauteur du toit de la BD TOPO (connue pour 76 % des bâtiments de la Manche, 15 % à Paris) ; fenêtres dans la texture (travées de 3 m et niveaux), rien à stocker. Aucun échec ; 7 × 10⁻⁵ des cases de bâti sans volume ; 9 à 16 µs par bâtiment. Limite : les bâtiments à cour n'ont de pente que côté rue (squelette droit à étudier).
- **Constat** (P-08) : la porte posée sur la case accessible au pied des murs la plus proche de la rue (à pied) donne sur la zone accessible pour 97,6 % des bâtiments de Paris et 99,6 % de la Manche, aucun sans solution ; elle est deux fois plus près de la rue que par la règle provisoire de P-05 (case accessible la plus proche du centroïde), qui place une porte sur quinze à plus de 3 m du bâtiment à Paris. Elle demande la zone accessible et la grille franchissable des tiles voisines, et 5 à 9 ms de parcours en largeur par tile.
- **Piste** (P-08) : porte calculée par la **chaîne de données** et stockée avec le bâtiment (rang de l'arête et abscisse, environ 1,6 octet).
- **Constat** (P-08) : les porches générés aux cases percées par P-07 (bas du bâtiment = emprise moins les cases, opération booléenne en entiers) couvrent 96 % des cases percées de la Manche et 99,7 % de celles de Paris (27 % des bâtiments parisiens ont un porche).
- **Constat** (P-08) : la nature BD TOPO ne désigne pas les monuments : « Monument » recouvre à Paris des statues (3 m² en médiane), « Église » mêle 835 églises de village de la Manche et l'abbaye du Mont-Saint-Michel, que le gabarit rend en bloc massif.
- **Piste** (P-08) : monuments modélisés fidèlement = **liste écrite à la main** d'identifiants BD TOPO ; gabarits par nature (église, château, fort, phare, moulin) pour les autres.
- **Constat** (P-08) : un bâtiment endommagé ne change que de couleur ; effondré, il est remplacé par des gravats sur ses cases de 2 m dilatées selon sa hauteur, qui deviennent infranchissables (14 µs pour la grille). Un bloc par case est trop lourd pour les grands bâtiments (un hangar de 3,5 ha : 103 000 triangles) : fusionner les gravats d'un bâtiment.
- **Décision** (P-08, [décisions](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md#décisions)) : **modèle** d'un bâtiment de P-08 : emprise réelle extrudée, posée sur le relief de 2 m avec un soubassement de hauteur bornée ; hauteur de la BD TOPO, sinon étages, sinon altitudes du toit, sinon médiane de la tile pour le même usage et la même classe d'emprise, sinon une constante par usage ; toit (plat, deux pans, quatre pans, mansarde) choisi d'après la hauteur du toit de la BD TOPO, décrit sur le rectangle englobant orienté (pas de squelette droit) ; fenêtres dans la texture, recalculables, jamais stockées ; porches par opération booléenne en entiers aux cases percées.
- **Décision** (P-08) : la **porte** est sur la case accessible au pied d'un mur extérieur la plus proche de la rue à pied ; calculée par la **chaîne de données** et stockée avec le bâtiment. Sans case accessible au pied des murs (0,4 à 2,4 % des bâtiments), elle est au point du mur le plus proche de la case accessible la plus proche du centroïde (règle de P-05).
- **Décision** (P-08) : **monuments** modélisés fidèlement : une **liste écrite à la main** d'identifiants BD TOPO ; une liste courte (quelques dizaines de sites nationaux), sans téléchargement de la base Mérimée ; les églises, chapelles, châteaux, forts, phares et moulins ont des **gabarits par nature** de forme simple (nef et clocher, corps et tours d'angle…), générés comme les autres bâtiments. Un bâtiment s'affiche endommagé dès que sa barre de vie est entamée.
- **Décision** (P-08) : un bâtiment **endommagé** ne change que de couleur ; **effondré**, il laisse un seul relief de gravats sur son emprise dilatée selon sa hauteur (0,3 × la hauteur, valeurs de départ), dont les cases deviennent infranchissables pour les humains et les véhicules.
- **Statut** : Décidé (bâtiments contenants, auto-résolution, passages, barre de vie, modèle, porte, monuments) ; liste des monuments à écrire.

### B-03 — Charge limite des ponts

- **Problème** : la charge limite est rarement renseignée dans les données.
- **Piste** : heuristique selon le type de pont, la classe de route et la largeur ; valeurs réelles pour les grands ouvrages.
- **Constat** (P-07, [résultats](https://github.com/fafapiti/prototypes/blob/main/p07-chaine-manche/README.md)) : dans la Manche, **1 pont sur 3 292** a une restriction de poids (37 tronçons de route sur 195 784). La largeur de la route portée est connue pour 53 % des ponts, son importance pour tous ceux qui portent une route ; 701 ouvrages ne portent aucun tronçon. Ce que le pont franchit se déduit des objets au sol (eau pour 2 711, voie ferrée pour 232).
- **Piste** (P-07) : heuristique (importance, largeur, longueur, ce qui est franchi), avec des valeurs écrites à la main pour quelques grands ouvrages ; la donnée réelle est quasi absente.
- **Décision** (P-07) : charge limite tirée d'une **heuristique** (importance, largeur et longueur de la route portée, ce que le pont franchit), avec des **valeurs réelles écrites à la main** pour quelques grands ouvrages. Valeurs à fixer.
- **Statut** : Décidé (principe) ; valeurs à fixer.

### B-04 — Végétation

- **Problème** : chaque arbre est un objet (le feu le demande), mais les données ne décrivent que des surfaces boisées.
- **Piste** : arbres générés de façon déterministe à partir des surfaces forestières et de leur essence (BD Forêt).
- **Constat** (P-07, [résultats](https://github.com/fafapiti/prototypes/blob/main/p07-chaine-manche/README.md)) : la BD TOPO distingue forêts de feuillus, de conifères, mixtes, bois, haies (233 000 surfaces et 902 000 lignes dans la Manche), landes, vergers, vignes et peupleraies, sans essences fines (BD Forêt, non téléchargée). Une couche d'occupation à 2 m (un octet par case) fait 6 à 11 ko par tile. Tirer les arbres au chargement par hachage coûte 0,5 ms par tile (8 000 arbres en médiane en rural, 38 000 au plus) ; les stocker coûterait 33 ko par tile.
- **Décision** (P-07) : les arbres sont **générés au chargement**, de façon déterministe (hachage), à partir de la couche d'occupation à 2 m (classes de la BD TOPO : feuillus, conifères, mixte, bois, haie, lande, verger, vigne, peupleraie) ; ils ne sont pas stockés.
- **Statut** : Décidé (arbres générés au chargement) ; Ouvert (essences fines : BD Forêt, densités).

### B-05 — Destructions et modifications du terrain

- **Problème** : cratères, bâtiments effondrés, ponts détruits, fossés et talus creusés par le génie modifient le terrain, la recherche de chemin et le rendu.
- **Constat** (P-08, [résultats](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md)) : effondrer un bâtiment (régénération en 0,03 à 2 ms, cases de gravats bloquées en 14 µs, refusion du morceau de 250 m qui le contient en 0,02 à 0,5 ms) est bon marché ; seule la géométrie des gravats en blocs de 2 m est trop lourde pour les grands bâtiments.
- **Décision** (P-08, [décisions](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md#décisions)) : un bâtiment effondré se régénère depuis son état (gravats et cases bloquées compris) ; seul son état est sauvegardé (A-13).
- **Constat** (P-09, [résultats](https://github.com/fafapiti/prototypes/blob/main/p09-chemin-routier/README.md#4-modifications)) : couper une route ou détruire un pont se répercute sur les itinéraires en moins d'une milliseconde par classe (personnalisation partielle de la CCH), 1 000 coupures en 11 ms ; la réouverture rend exactement la métrique d'origine. La BD TOPO décrit un pont **par tablier** : détruire un pont d'une 2 × 2 voies ne coupe qu'un sens.
- **Décision** (P-09) : un **pont du jeu** est une traversée (tabliers regroupés par la chaîne) : il est détruit, endommagé ou réparé d'un bloc. Gravats, épaves et barrages du génie coupent les arêtes qu'ils recouvrent.
- **Statut** : Décidé (bâtiments effondrés, ponts détruits et routes coupées dans la recherche de chemin) ; Ouvert (cratères, fossés et talus).

### B-06 — Licences des données

- **Problème** : vérifier la compatibilité de chaque source avec un projet 100 % open source : IGN (Licence Ouverte Etalab), OpenStreetMap (ODbL, attribution et partage à l'identique), INSEE, SHOM.
- **Constat** (P-07, [résultats](https://github.com/fafapiti/prototypes/blob/main/p07-chaine-manche/README.md)) : BD TOPO et RGE ALTI sous Licence Ouverte Etalab (version à vérifier) ; carreaux Filosofi de l'INSEE sous Licence Ouverte 2.0 (data.gouv.fr). L'IGN ne couvre pas le Pacifique (Nouvelle-Calédonie, Polynésie, Wallis-et-Futuna), qui a ses propres services (Te Fenua en Polynésie : MNT publiés, bâti à vérifier, limites des communes en ODbL) : le Pacifique a été retiré du jeu ([C-01](#c-01--précision-numérique)). Les tirants d'eau des ports manquent (SHOM).
- **Statut** : Ouvert.

## C. Rendu et caméra

### C-01 — Précision numérique

- **Problème** : à l'échelle 1:1, les flottants 32 bits perdent la précision au centimètre à quelques kilomètres de l'origine.
- **Piste** : coordonnées 64 bits pour la simulation, origine flottante centrée sur la caméra pour le rendu.
- **Prototype P-06** ([résultats](https://github.com/fafapiti/prototypes/blob/main/p06-origine-flottante/README.md)) : le rendu relatif à la caméra ramène l'écart entre la position calculée et la position rendue sous 0,02 px (0,1 mm) à Brest, à Tahiti et au point le plus éloigné de Paris, l'**île Walpole** (Nouvelle-Calédonie, 16 866 km), confirmé par des captures du rendu réel. Sans origine flottante, le monde f32 tremble de 8 px à Brest, 190 px à Tahiti et 300 px à Walpole (origine à Paris), 140 à 190 px partout (origine au centre de la Terre).
- **Constat** (P-06, repère de la simulation) : trois repères comparés sur 1 million de marcheurs à 20 Hz. **Tile + position locale f32** : 8 octets, 2,3 ms par tick, même résultat au bit près quel que soit l'endroit du monde, passage de tile exact avec des coordonnées centrées (±500 m). **Grille projetée en f64** : 16 octets, 2,6 ms, résultat dépendant de l'endroit. **ECEF f64** : 24 octets, 42 ms, car il faut projeter à chaque tick pour trouver la tile et la case du flow field.
- **Constat** (P-06, repère d'une tile) : posée par son plan tangent, une tile s'écarte de 39 mm de la Terre à ses coins (courbure) et de sa voisine de 7 mm au plus ; au second ordre (Taylor au centre), de moins de 0,01 mm. Un navire qui passe de la tile d'un port à l'ECEF en mer saute alors de 0,0 mm (19 mm en plan tangent). L'antiméridien et les fuseaux de la Polynésie ne posent aucun problème.
- **Constat** (P-06, recalage) : à 100 000 modèles, recaler à chaque frame coûte 2,3 ms de CPU et 8,6 ms de frame, et fait trembler les grands maillages dont l'origine est loin (morceaux de globe) ; au-delà d'un seuil, 0,25 ms, avec un à-coup à chaque recalage (2,3 ms ; 104 à 111 ms de frame à 1 million de modèles). Jusqu'à 10 km entre la caméra et l'origine, l'écart reste sous 0,15 px ; à 100 km, 1,3 px. La crate `big_space` est précise mais n'est pas mûre (version candidate pour Bevy 0.20, remplace la propagation de Bevy, axes géocentriques). La hiérarchie de Bevy (un parent par tile) repropage tous les enfants d'un parent dès qu'un seul bouge.
- **Constat** (P-06, profondeur) : avec la profondeur inversée de Bevy et des sommets relatifs à la caméra, deux surfaces se départagent dès 3 × 10⁻⁷ fois la distance (0,1 mm à 30 m, 10 m vu de 20 000 km) : pas de z-fighting à craindre. Les ombres en cascade se redessinent légèrement (0,2 à 0,5 % des pixels) à chaque recalage.
- **Décision** (P-06) : simulation en **tile + position locale f32 centrée** sur la tile (±500 m) ; tile = (zone de projection, ix, iy). Les **navires en mer** sont en **ECEF f64** ; ils changent de zone par l'ECEF, à 30 km des côtes (valeur de départ).
- **Décision** (P-06) : **zones de projection** : Lambert-93 en métropole (Corse comprise) ; UTM ou Lambert local par territoire, une zone par fuseau en Polynésie (UTM 5S à 8S). Projection d'usage de chaque territoire à vérifier. Depuis P-07, le Pacifique n'est plus dans le jeu : restent les zones des Antilles, de la Guyane, de La Réunion, de Mayotte et de Saint-Pierre-et-Miquelon.
- **Décision** : **pas de TAAF** dans le jeu (Kerguelen, Crozet, Amsterdam et Saint-Paul, îles Éparses, Terre Adélie) : inhabitables. **Pas de Clipperton** non plus (atoll inhabité).
- **Décision** (P-07) : **pas de Pacifique** dans le jeu (Nouvelle-Calédonie, Polynésie française, Wallis-et-Futuna) : l'IGN ne les couvre pas (ni BD TOPO, ni RGE ALTI), et leurs données (services géographiques locaux, OpenStreetMap) n'ont ni le même contenu ni les mêmes licences. Un jeu de données homogène vaut ce retrait. Les zones de projection de ces territoires, mesurées par P-06, ne servent plus ; le point le plus éloigné de Paris devient La Réunion (environ 9 400 km), et la précision mesurée par P-06 à 16 866 km reste un majorant. Conséquence : les SNLE, basés à Tahiti, sont désormais basés à **Saint-Pierre-et-Miquelon** ([fiche](game-elements/units/naval/snle-triomphant.md)).
- **Décision** (P-06) : repère de chaque tile au **second ordre** (Taylor au centre), pas en plan tangent.
- **Décision** (P-06) : **rendu relatif à la caméra**, origine **recalée au-delà de 5 km** de la caméra, par le jeu ; ni `big_space`, ni hiérarchie de Bevy pour les modèles.
- **Piste** (P-06) : caler les cascades d'ombres sur leur grille de texels, pour supprimer le saut des bords d'ombre au recalage ; à étudier dans P-10.
- **Statut** : Décidé.

### C-02 — Niveaux de vue

- **Problème** : la vue isométrique ne permet pas de dézoomer jusqu'à l'espace.
- **Proposition** : au-delà d'un certain zoom, passage en **vue simplifiée** : carte 2D de type carte routière. Voir aussi C-03.
- **Décision** : en vue simplifiée, la surface de chaque commune est colorée selon son état : rouge (zombie), jaune (contestée), bleu (libérée). Ce statut est purement cosmétique, sans effet sur le jeu. Règles de passage dans [mairies.md](game-elements/locations/mairies.md).
- **À décider** : seuil de bascule ; style de la carte ; projection (Lambert-93 pour la métropole, projections locales pour les outre-mer) ; affichage des distances Paris–La Réunion (globe ou changement de carte).
- **Constat** (P-06, [résultats](https://github.com/fafapiti/prototypes/blob/main/p06-origine-flottante/README.md#2-précision-du-rendu-relecture-de-la-projection-resultats-precisioncsv)) : un zoom continu du soldat (25 m) au globe (30 000 km) reste précis à toutes les distances avec l'origine flottante. Un globe découpé en morceaux de 625 km à origine locale est sous 0,5 px au-delà de 1 à 10 km de la caméra ; plus près, il faut afficher le sol des tiles. Zones de projection vérifiées dans PROJ : Lambert-93 (EPSG 2154), UTM 20N, 22N, 40S, 38S, 21N, UTM 5S à 8S en Polynésie, Lambert Nouvelle-Calédonie (3163), UTM 1S (Wallis-et-Futuna) ; projection d'usage de chaque territoire à vérifier.
- **Décision** (P-06) : sous 1 à 10 km d'altitude de caméra, le **sol des tiles** est affiché ; au-dessus, le globe découpé (C-03). Le seuil exact de bascule reste à fixer (P-10).
- **Constat** (P-08, [résultats](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md)) : trois niveaux de bâtiments : vue isométrique complète (82 triangles par bâtiment dans la Manche, 155 à Paris) ; vue lointaine (emprise simplifiée à 1 m, sans fenêtres, porches ni portes : 50 et 78 triangles) ; carte (emprise plate : 3 à 6 triangles).
- **Piste** (P-08) : pas de bâtiments en 3D dans la vue simplifiée : le bâti de 1 m, déjà recalculé au chargement (P-07), sert de texture de carte.
- **Décision** (P-08, [décisions](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md#décisions)) : bâtiments en trois niveaux : vue isométrique complète ; vue lointaine (emprise simplifiée, sans fenêtres, porches ni portes) ; **aucun bâtiment en 3D en vue simplifiée**, où le bâti de 1 m sert de texture de carte.
- **Statut** : Piste.

### C-03 — Globe

- **Problème** : le globe est affiché, hors zone simulée en gris. Il faut une transition entre la carte 2D et le globe.
- **Piste** : la carte 2D est plaquée sur le globe au dézoom maximal.
- **Constat** (P-06, [résultats](https://github.com/fafapiti/prototypes/blob/main/p06-origine-flottante/README.md#analyse)) : un globe dont les sommets sont relatifs au centre de la Terre est faux de 74 px de près et de 2 px à 1 km : il doit être découpé en morceaux, chacun avec son origine locale (comme une tile). Avec des morceaux de 625 km, il est précis au-delà de 1 à 10 km de la caméra.
- **Décision** (P-06) : le globe est **découpé en morceaux à origine locale**, comme les tiles ; il n'est pas affiché de près (sol des tiles, C-02).
- **Statut** : Ouvert.

### C-04 — Streaming sans écran de chargement

- **Problème** : déplacer la caméra de Paris à La Réunion sans attente, en chargeant les tuiles à la volée.
- **Piste** : la vue simplifiée (C-02) réduit le problème : au-delà du seuil, seules des tuiles 2D légères sont nécessaires ; les tuiles 3D se chargent pendant le zoom avant.
- **Constat** (P-07, [résultats](https://github.com/fafapiti/prototypes/blob/main/p07-chaine-manche/README.md)) : charger une tile coûte 3 ms (rural) à 8 ms (dense) sur un thread : lecture 0,02 ms, relief 2 ms (premier poste), bâti de 1 m recalculé jusqu'à 5 ms. L'en-tête seul (part de bâti, habitants, terre ou mer) se lit sans rien décompresser, en 0,02 ms : la vue simplifiée n'a besoin que de lui. Une tile fait 60 ko en moyenne : un survol à basse altitude lit environ 100 ko/s.
- **Piste** (P-07) : chargement étalé comme l'activation de P-04 (relief et bâti dans des ticks différents, en tâche de fond) ; décodage du relief par bandes indépendantes s'il faut le paralléliser.
- **Décision** (P-07) : le chargement d'une tile est **étalé** sur plusieurs ticks, en tâche de fond, comme l'activation (P-04) ; la vue simplifiée ne lit que les en-têtes.
- **Constat** (P-08, [résultats](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md)) : générer les bâtiments d'une tile en tâche de fond (0,6 à 110 ms) suit le survol sans jamais manquer une tile proche, jusqu'à 1 km/s en vue lointaine (1 306 tiles en 69 s). Les à-coups viennent de l'**envoi au GPU** : Bevy recopie dans la même frame tous les maillages ajoutés, avant son limiteur d'octets ; une tile fusionnée de 15 Mo fait 20 frames au-dessus du budget par vol de 12 km à Paris. Des morceaux de 250 m envoyés par une file de 2 Mo par frame n'en font aucune ; construire le maillage de Bevy dans la tâche ramène le thread principal de 14,5 à 0,01 ms par tile.
- **Piste** (P-08) : bâtiments **générés au chargement** (jamais stockés : 28 à 45 Go pour la France), en tâche de fond ; maillages découpés en morceaux de 1 à 2 Mo et envoyés par une file limitée en octets par frame.
- **Décision** (P-08, [décisions](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md#décisions)) : les bâtiments sont **générés au chargement**, en tâche de fond, jamais stockés ; leurs maillages sont découpés en assets de 1 à 2 Mo et envoyés au GPU par une **file limitée en octets par frame**.
- **Décision** (jeu en navigateur) : le streaming passe désormais par le **réseau** : les tiles viennent du serveur ([H-06](#h-06--diffusion-des-tiles)), l'état des unités et des zombies aussi ([H-05](#h-05--débit-réseau-et-gestion-dintérêt)). Le client génère toujours les bâtiments lui-même, en WebAssembly ([F-04](#f-04--plateformes-cibles)).
- **Statut** : Piste.

### C-05 — Rendu des foules

- **Problème** : des milliers de zombies visibles à l'écran en vue isométrique.
- **Pistes** : instanciation GPU, imposteurs (sprites) à distance, animation sur GPU. En vue simplifiée, représentation par densités (cartes de chaleur).
- **Constat** (P-06, [résultats](https://github.com/fafapiti/prototypes/blob/main/p06-origine-flottante/README.md#8-cadence-de-rendu-resultats-cadencecsv)) : avec une entité Bevy par modèle rendu, la frame prend 3,4 ms pour 100 000 modèles et 22 ms pour 1 million, sans aucun recalage ; chaque recalage de l'origine coûte en plus une frame de 104 à 111 ms à 1 million.
- **Décision** (P-06) : les foules et les modèles nombreux passent par l'**instanciation** (un tampon de positions relatives à l'origine, rempli depuis la simulation) ; environ **100 000 entités rendues** au plus.
- **Constat** (P-08, [résultats](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md)) : pour les bâtiments, 49 tiles parisiennes (12,4 millions de triangles) se rendent en 4 ms en morceaux fusionnés de 250 m (832 entités), 3 ms au format compressé de Bevy 0.20 (mémoire GPU divisée par deux, sans changer de shader) ; une entité par bâtiment fait 76 000 entités, des pics de 48 ms et 75 % de mémoire en plus. L'instanciation ne sert pas aux volumes, tous différents.
- **Décision** (P-08, [décisions](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md#décisions)) : bâtiments rendus en **morceaux de 250 m fusionnés**, au format compressé de Bevy ; ni entité par bâtiment, ni instanciation des volumes.
- **Statut** : Prototype.

### C-06 — Lisibilité en ville

- **Problème** : en vue isométrique, les bâtiments masquent les unités.
- **Pistes** : bâtiments rendus transparents ou découpés autour des unités ; silhouettes visibles à travers les murs.
- **Constat** (P-08, [résultats](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md)) : refusionner un morceau pour masquer un bâtiment coûte 10 à 30 ms de mise à jour : la transparence et la découpe autour des unités doivent se faire dans le shader.
- **Piste** (P-08) : indice du bâtiment par sommet (4 octets) et table d'états par morceau lue par le shader, ou découpe géométrique autour de l'unité ; à mesurer dans P-12.
- **Décision** (P-08, [décisions](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md#décisions)) : chaque sommet des morceaux porte l'**indice de son bâtiment** (rang sur 16 bits dans le morceau), pour la transparence et la découpe dans le shader ; méthode à choisir dans P-12.
- **Statut** : Piste.

### C-07 — Feu et fumée à grande échelle

- **Problème** : rendre des incendies de plusieurs km² avec un coût raisonnable.
- **Statut** : Ouvert.

## D. Interface et jouabilité

### D-01 — Du soldat à l'armée

- **Problème** : le joueur peut contrôler chaque soldat, mais commande aussi des dizaines de milliers d'hommes sur tout le territoire. Il lui faut une chaîne de commandement : ordres donnés à une section, une compagnie, un régiment, et exécutés automatiquement.
- **Statut** : Ouvert (central).

### D-02 — IA des unités amies

- **Problème** : toutes les unités que le joueur ne contrôle pas directement doivent agir avec compétence : se défendre, se ravitailler, respecter les ordres, fuir quand il le faut. Cela inclut aéronefs et navires (waypoints, patrouilles, axes de frappe).
- **Conséquence** (temps réel, monde persistant) : le monde tourne sans personne de connecté ; une unité doit pouvoir tenir des heures, voire des jours, sans qu'aucun joueur ne la regarde (A-02).
- **Statut** : Ouvert (central).

### D-03 — Logistique automatisée

- **Problème** : le joueur ne peut pas organiser chaque camion à la main sur toute la France.
- **Pistes** : lignes de ravitaillement automatiques entre dépôts et unités, que le joueur configure (priorités, escortes, itinéraires).
- **Pistes** : les lieux contrôlés ouvrent des lignes à grande capacité : de [gare](game-elements/locations/gares.md) à gare, de [port](game-elements/locations/ports.md) à port, de [base aérienne](game-elements/locations/bases-aeriennes.md) à base.
- **Conséquence** (temps réel, monde persistant) : une traversée dure des semaines réelles et aucun joueur ne suit un convoi de bout en bout. Les lignes automatiques deviennent le mode normal de la logistique, pas un confort (A-02).
- **Statut** : Ouvert (central).

### D-04 — Alertes et information

- **Problème** : avec tout le territoire en jeu, le joueur doit savoir ce qui se passe ailleurs (convoi attaqué, unité à court de munitions, incendie).
- **Conséquence** (multijoueur) : les alertes sont **communes** à tous les joueurs ; un joueur qui arrive doit comprendre en quelques instants où l'on a besoin de lui (résumé de l'état du front, alertes en attente).
- **Statut** : Ouvert.

### D-05 — Inspection de la cargaison

- **Problème** : interface pour inspecter palettes, conteneurs, soutes de navires et inventaires des soldats sans noyer le joueur.
- **Statut** : Ouvert.

### D-06 — Équilibrage

- **Problème** : l'efficacité réaliste (obus flèche inutiles, missiles gaspillés) est voulue, mais le jeu doit rester jouable et intéressant. Les chiffres viendront plus tard ; il faudra des tests de jeu.
- **Statut** : Ouvert.

## E. Contenu

### E-01 — Volume de contenu

- **Problème** : chaque version de chaque véhicule, chaque navire nommé, les monuments : énorme travail de modèles 3D et de données.
- **Pistes** : niveau de détail adapté à la vue isométrique ; contributions de la communauté (open source).
- **Statut** : Ouvert.

### E-02 — Format des données

- **Problème** : les fiches d'unités doivent devenir des fichiers de données lus par le jeu (et par les mods), avec validation.
- **Piste** : format texte lisible (RON ou TOML), un fichier par unité, schéma validé automatiquement.
- **Statut** : Piste.

### E-03 — Exactitude des informations

- **Problème** : versions, dotations et noms de navires en 2030 à vérifier (marqués « à vérifier » dans les fiches).
- **Statut** : Ouvert.

### E-04 — Noms et marques réels

- **Problème** : certains noms d'armes et de véhicules sont des marques déposées (HK416, Rafale…). Les utiliser dans un jeu distribué peut poser un problème juridique.
- **Statut** : Ouvert (à faire vérifier).

### E-05 — Traduction

- **Piste** : Project Fluent (crate `fluent`) ; toutes les chaînes, y compris les noms d'unités, externalisées dès le début.
- **Statut** : Piste.

## F. Architecture et technique

### F-01 — Moteur

- **Problème** : moteur existant (Bevy, par exemple) ou moteur sur mesure. Bevy évolue vite (API instable) mais offre un ECS performant ; le sur-mesure coûte beaucoup plus cher.
- **Prototype P-00** ([résultats](https://github.com/fafapiti/prototypes/blob/main/p00-socle-ecs/README.md)) : l'ECS de Bevy, en multi-cœur, égale du Rust écrit à la main (≈ 2 ms par tick pour 1 million d'entités), si l'on groupe les composants mis à jour ensemble et contourne la détection de changement là où elle ne sert pas. La limite est la bande passante mémoire, pas le moteur : 10 millions d'agents complets par tick ne tiennent dans le budget avec aucune solution.
- **Décision** : le jeu est construit sur **Bevy** complet (ECS, rendu, fenêtres, entrées). La simulation utilise l'**exécuteur multi-thread** (systèmes en parallèle, requêtes en `par_iter_mut`).
- **Décision** (multijoueur, navigateur) : deux binaires issus du même code. Le **serveur** utilise Bevy sans rendu (ECS, exécuteur multi-thread, `MinimalPlugins`) : il porte toute la simulation. Le **client** utilise Bevy complet compilé en **WebAssembly**, rendu en WebGPU : il affiche, il ne simule pas ([F-04](#f-04--plateformes-cibles)).
- **Conséquence** : l'API de Bevy change à chaque version ; la version est figée dans `Cargo.lock` et les montées de version sont des chantiers planifiés.
- **Statut** : Décidé.

### F-02 — Performances de la simulation

- **Pistes** : ECS, parallélisme multi-cœur, calcul sur GPU pour les foules et le feu.
- **Décision** (abandonnée) : machine de référence unique, AMD Ryzen 7 5700X (8 cœurs, 16 threads), 32 Go de DDR4, NVIDIA RTX 3060 12 Go, qui fixait le budget d'agents.
- **Décision** (multijoueur) : deux références.
  - **Serveur** (`srv.chelsea486mhz.fr`) : AMD **EPYC 7351P** (16 cœurs, 32 threads), 128 Go de mémoire, deux disques durs de 8 To en RAID 0. Il fixe le **budget d'agents** : le tick entier (50 ms à 20 Hz) est à la simulation, sans rendu ni accélération (A-02).
  - **Client** : un navigateur sur la machine Ryzen 7 5700X + RTX 3060. Elle fixe le budget de **rendu**.
- **Conséquence** : les prototypes déjà mesurés sur la machine Ryzen restent valables en relatif ; les budgets « à ×10 » sont caducs.
- **Constat** (serveur, relevé) : 8 canaux de DDR4-2666 (8 × 16 Go), machine dédiée au jeu. Les contraintes de production (NUMA, options de compilation) attendront un prototype jouable.
- **Décision** (P-00) : les données mises à jour ensemble sont **groupées dans un même composant** (par exemple les quatre jauges d'un soldat dans un composant `Jauges`). Moins de colonnes à parcourir et d'horodatages à écrire : à 1 million d'entités, 2,4 ms au lieu de 3,4 ms par tick, et 15 % de mémoire en moins.
- **Constat** (P-00) : la limite est la bande passante mémoire, pas le processeur. Au-delà d'environ 1 million d'entités par tick, ajouter des cœurs ne sert plus ; la conception doit réduire le volume de données touché à chaque tick.
- **Constat** (P-00, test 2) : avec des types d'entités variés (nombreux archétypes), les requêtes parallèles de Bevy s'effondrent avec le découpage par défaut (×200 sur une tile de 20 000 entités et 1 024 archétypes). Une taille de lot minimale (`min_batch_size`, ≈ 4 096) ramène le surcoût à presque rien.
- **Décision** (P-00, test 2) : toute requête parallèle fixe une **taille de lot minimale**.
- **Décision** (P-01) : taille de lot minimale d'environ **1 024 entités** (et au plus *entités / (2 × threads)* pour les petites requêtes). 4 096 laissait la plupart des threads inactifs sur une tile seule.
- **Constat** (P-01) : sur une tile seule (20 000 entités), un lot minimal de 4 096 ne crée que 5 lots, donc 5 threads occupés sur 16 : le tick double (3,0 ms au lieu de 1,6). De 64 à 1 024, aucune différence ; avec 16 tiles, la taille n'a plus d'effet.
- **Constat** (P-01) : la simulation des zombies d'une tile dense est limitée par le calcul, pas par la mémoire (×7,5 sur 8 cœurs) ; la perception en est le premier poste.
- **Constat** (P-02, [résultats](https://github.com/fafapiti/prototypes/blob/main/p02-perception/README.md)) : avec un arbre k-d par tile, une requête de perception coûte 0,2 à 0,4 µs ; la recherche de relais entre zombies, faite à chaque tick, coûte désormais plus en combat que les perceptions qu'elle évite (10 à 20 % du tick). Passage à l'échelle : ×9,5 sur 16 threads. Le résultat est identique quelle que soit la structure d'index, ce que vérifie une somme de contrôle.
- **Constat** (P-03, [résultats](https://github.com/fafapiti/prototypes/blob/main/p03-flow-fields/README.md#index-des-zombies)) : la reconstruction de la grille des zombies est limitée par ses **accès mémoire aléatoires** dans des tableaux qui débordent du cache (2,5 Mo par tile), pas par son calcul. En u16 et sans tableau de curseurs (0,53 Mo), sa phase baisse de 72 % à 16 tiles et les autres phases accélèrent de 9 à 17 % (cache moins pollué). Règle : réduire les octets touchés par tick avant de chercher un meilleur algorithme. Par ailleurs, en combat, 8 threads font aussi bien que 16 : le SMT n'apporte rien à ces phases.
- **Décision** (P-03) : tout **calcul lourd d'une tile** (flow field, recalcul après un obstacle) est **étalé sur plusieurs ticks**, en double tampon : d'un seul tenant, il tombe sur un seul thread et double le p99 d'une tile seule.
- **Décision** (P-00, test 3) : les **états fréquents** (blessé, en feu, en fuite, à couvert…) sont des **champs** dans un composant. Les **composants marqueurs** sont réservés à ce qui change rarement et définit l'entité (type d'entité, mort, transformation en zombie). Chaque ajout ou retrait de composant coûte 0,15 à 0,5 µs, en série : à 1 million d'entités, 10 % de changements par tick coûtent 43 ms avec des marqueurs, contre 0,7 ms avec un champ.
- **Constat** (P-04, [résultats](https://github.com/fafapiti/prototypes/blob/main/p04-activation/README.md#analyse)) : trois pièges de l'ECS coûtaient plus cher que l'activation des tiles elle-même. (1) Une boucle qui pilote le `World` de Bevy à la main doit appeler `clear_trackers` à chaque tick, comme le fait l'`App` : sinon les journaux de composants retirés grossissent sans fin, et un `despawn` coûte jusqu'à 14 ms quand ils doublent. (2) Quand le nombre d'entités franchit un palier, Bevy recopie ses tables (naissance de 6 ms au lieu de 1,5) : un **préchauffage** au démarrage (créer puis détruire le maximum attendu d'entités) l'évite. (3) Tout calcul lourd doit avoir un coût estimé avant son premier passage, sinon il tombe d'un seul tenant.
- **Constat** (P-04) : créer ou détruire des entités se fait **par lots** (environ 2 000 par tick) : 20 000 entités d'un coup coûtent jusqu'à 10 ms.
- **Constat** (P-05, [résultats](https://github.com/fafapiti/prototypes/blob/main/p05-civils/README.md#analyse)) : pour une population nombreuse et lente (les civils), le coût d'une entité ECS par individu tient d'abord au **parcours** : lire 1 million d'entités pour en mettre à jour 4 500 coûte 0,96 ms. Les ranger dans 20 tables par phase ramène le coût à 0,21 ms ; des dossiers compacts par lieu, triés par phase et hors de l'ECS, à 0,08 ms, pour le même calcul (somme de contrôle identique). Règle : une donnée mise à jour rarement se range par échéance de mise à jour, comme un index sur la date d'échéance.
- **Constat** (P-05) : dans le binaire portable (x86-64 de base, sans SSE4.1), `f32::floor` et `rem_euclid` sont des appels de bibliothèque. Cinq par colon coûtaient plus d'un tiers d'une mise à jour (41 ns contre 25 ns avec des troncatures sur des valeurs positives).
- **Décision** (P-05) : une population nombreuse et lente (les civils) vit **hors de l'ECS**, en dossiers compacts rangés par échéance de mise à jour ; seule la part qui doit se déplacer ou être vue devient des entités (A-15).
- **Constat** (P-06, [résultats](https://github.com/fafapiti/prototypes/blob/main/p06-origine-flottante/README.md#analyse)) : la **hiérarchie** de Bevy repropage tous les enfants d'un parent dès qu'un seul enfant bouge (à 1 million de modèles répartis sous 100 parents, 30 ms par frame au lieu de 1,6 ms). Les `Transform` de rendu modifiés coûtent deux fois : propagation, puis réextraction et envoi au GPU (12 ms de frame pour 100 000 modèles réécrits à chaque frame, contre 3,4 ms). À 1 million d'entités rendues, la frame prend 22 ms sans rien réécrire : le nombre d'entités rendues est une limite en soi (C-05).
- **Constat** (P-06) : sur la machine de référence, la RTX 3060 n'accepte que les surfaces Wayland (pas X11) ; Bevy doit être compilé avec le support Wayland de winit. Sous Wayland, une fenêtre est cadencée par le compositeur : les temps de frame se mesurent hors écran. Bevy ne dessine rien pendant la compilation asynchrone des shaders (quelques secondes).
- **Constat** (P-06) : le compteur de ticks d'un colon de P-05 (`u32`) déborde après 6,8 ans de jeu à 20 Hz ; la partie est sans fin (G-07).
- **Décision** (P-06) : les compteurs de ticks sont en **u64** ; pas de hiérarchie de Bevy pour les modèles rendus ; foules rendues par instanciation (C-05). La date de mise à jour d'un colon reste un u32 relatif, comparé par soustraction modulaire : la fiche garde ses 32 octets (A-02).
- **Décision** (P-04) : les entités sont créées et détruites **par lots** étalés sur plusieurs ticks ; les tables de l'ECS sont **préchauffées** au démarrage pour **800 000 zombies**, ce que le budget de ×1 permet (environ 40 tiles × 20 000) ; la boucle de simulation vide les journaux de l'ECS (`clear_trackers`) à chaque tick ; les tiles actives réutilisent un **pool** de slots (aucune allocation en régime établi).
- **Constat** (P-08, [résultats](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md)) : générer les bâtiments d'une tile dense d'un seul tenant prend 38 à 80 ms ; étalé, au plus 3,08 ms par tick (10 à 18 ticks). Paralléliser une tile sur ses bâtiments est instable (12 à 29 ms sur 16 threads, contre 88 sur un) ; paralléliser sur les tiles donne × 5 à × 8.
- **Statut** : Décidé.

### F-03 — Modding

- **Problème** : Rust n'a pas d'ABI stable.
- **Décision** : la sécurité des mods n'est pas un objectif : le jeu, open source et sans éditeur reconnu, est lui-même du code non vérifié pour le joueur. Pas de sandbox de sécurité à concevoir.
- **Pistes** : mods en WebAssembly (`wasmtime`) pour la portabilité et le multi-langage, ou mods natifs via une ABI C ; mods de données (E-02) sans code. Attention au coût des appels de mods dans les boucles critiques.
- **Conséquence** (serveur unique) : un mod qui change la simulation ne peut tourner que sur le **serveur**, et la partie officielle n'en charge que si son opérateur le décide. En pratique, modifier le jeu officiel, c'est proposer un changement au code ; les mods servent aux parties hébergées par d'autres.
- **Statut** : Piste.

### F-04 — Plateformes cibles

- **Décision** : le client est un **jeu en navigateur**, en **WebAssembly**, rendu en **WebGPU seul** (pas de repli WebGL2 : l'instanciation massive de C-05 et les compute shaders en ont besoin). Le serveur tourne sous **Linux**.
- **Contraintes** (à vérifier pour la version de Bevy figée) :
  - Bevy en WebAssembly est **mono-thread** : les threads exigent `SharedArrayBuffer`, donc l'isolation cross-origin (en-têtes COOP et COEP, posés par l'ingress), et l'exécuteur de Bevy ne les exploite pas. La génération des bâtiments (P-08 : 38 à 80 ms par tile dense sur un thread) est étalée sur plusieurs frames ou confiée à des Web Workers.
  - Mémoire limitée à **4 Go** (wasm32).
  - WebGPU sous Linux et Wayland (machine de référence) : support par Chrome et Firefox à vérifier en premier.
- **Statut** : Décidé (navigateur, WebGPU) ; Prototype (P-23).

### F-05 — Licence

- **Statut** : Ouvert (à voir plus tard).

## G. Trous de conception

### G-01 — Frontières avec les zones non simulées

- **Problème** : rien n'est simulé hors de France, mais la métropole a des frontières terrestres (Belgique, Luxembourg, Allemagne, Suisse, Italie, Espagne, Andorre, Monaco). La Guyane touche le Brésil et le Suriname. Saint-Martin partage son île avec Sint Maarten.
- **À décider** : que se passe-t-il à ces frontières ? Mur infranchissable, flux de zombies entrants, ou rien ?
- **Décision** (mer) : les **navires naviguent réellement en mer**, hors de la zone simulée, sur les vraies routes maritimes, avec durée et consommation. Les terres étrangères restent grisées. Les canaux de **Suez** et de **Panama** sont praticables, sans être simulés.
- **Décision** (P-06) : en mer, un navire est positionné en ECEF f64 ; à 30 km d'une côte (valeur de départ), il entre dans les tiles de la zone de projection du territoire. Le changement de représentation est sans saut (0,0 mm mesuré) ; l'antiméridien ne pose aucun problème ([P-06](https://github.com/fafapiti/prototypes/blob/main/p06-origine-flottante/README.md#analyse)).
- **Constat** (P-07, [résultats](https://github.com/fafapiti/prototypes/blob/main/p07-chaine-manche/README.md)) : la Manche compte 7 808 tiles de pleine mer dans sa bande de 30 km (0,2 ko chacune) ; elles peuvent ne pas être stockées. Les îles étrangères proches (Jersey, Guernesey) sont aujourd'hui classées en mer par la chaîne, qui définit la mer comme « hors de toute commune » ; cette règle met aussi en mer les herbus de la baie du Mont-Saint-Michel. Piste : la mer par le trait de côte des plus hautes eaux (couche `limite_terre_mer`) et l'absence de relief.
- **Décision** (P-07) : les îles étrangères à moins de 30 km d'une côte française (Jersey, Guernesey, îles des Antilles) sont **grisées**, comme le reste du monde ; seule leur mer est navigable.
- **Statut** : Décidé (mer) ; Ouvert (frontières terrestres).

### G-02 — Les zombies et la mer

- **Problème** : les zombies marchent au fond de l'eau. Peuvent-ils traverser la Manche, ou rejoindre une île d'outre-mer, ou un navire au mouillage, en marchant au fond de la mer ? Si oui, l'outre-mer n'est pas un refuge.
- **À décider** : profondeur maximale franchissable, ou interdiction de la mer.
- **Décision** (P-07) : les **estrans** sont de la terre dans les tiles ; la marée (A-11) les recouvre en jeu. La question du fond de la mer reste ouverte.
- **Statut** : Ouvert.

### G-03 — Importations

- **Problème** : rien n'étant simulé hors de France, il n'y a pas d'importations. La France produit très peu de pétrole brut ; les carburants deviennent des stocks finis, sauf production locale. Certains composants (électronique des munitions guidées, pièces étrangères) deviennent impossibles à produire.
- **Décision** : au début, tout vient de l'outre-mer par bateau. Les ressources que la France ne peut pas produire sont **livrées par des pays non spécifiés**, pour simplifier. Ces pays ne sont pas simulés.
- **Décision** : ressources importées : **pétrole brut**, **électronique**, et **composants que la France ne fabrique pas**, y compris les pièces de rechange américaines (Reaper notamment).
- **Décision** : **aucun quota**. Le joueur charge manuellement le navire, le fait voyager à vitesse réelle, puis le décharge manuellement. Le volume est limité par la flotte disponible et par la durée des trajets.
- **Décision** : la livraison par les pays étrangers **n'est pas simulée**. Les ressources importées sont ajoutées aux ressources disponibles environ **toutes les deux semaines** (réelles : le temps de jeu est le temps réel, A-02). Le joueur les achemine ensuite par ses propres navires.
- **Décision** : les ressources importées apparaissent dans les **ports d'outre-mer**. Le joueur doit les faire traverser jusqu'en métropole.
- **À décider** : quantité de chaque incrément.
- **Statut** : Décidé (principe, liste, rythme, lieu) ; Ouvert (quantités).

### G-04 — Énergie

- **Problème** : état des centrales électriques (dont les réacteurs nucléaires) en 2030 dans le scénario ; peut-on les remettre en service, avec quel personnel ?
- **Décision** : une [centrale nucléaire (CNPE)](game-elements/locations/cnpe.md) redémarrée restaure l'électricité dans sa région. Pour la redémarrer, il suffit d'y livrer du gazole pour ses groupes électrogènes de secours ; le génie fait le reste.
- **Décision** : régions électriques par découpage de Voronoï autour des CNPE et des [barrages](game-elements/locations/barrages.md), chaque commune rattachée à la centrale la plus proche de sa mairie. Toute la région est alimentée, communes libérées ou non.
- **Décision** : une centrale redémarrée fonctionne indéfiniment, sans approvisionnement. Le génie militaire et le génie civil peuvent la redémarrer ; garnison de 100 pour une CNPE.
- **Décision** : pas de centrales thermiques dans le jeu (trop lourdes à approvisionner).
- **Décision** : Voronoï **fixe**, pour inciter à libérer toutes les centrales ; région alimentée d'un bloc, réseau non simulé. Garnison exigée au redémarrage seulement. Les 10 plus gros barrages suivent la même règle que les CNPE. L'outre-mer a l'électricité d'office.
- **Statut** : Décidé.

### G-05 — Effets des armes nucléaires

- **Problème** : ASMP-A (300 kt) et M51 (100 kt) sont utilisables. Faut-il simuler le souffle, l'incendie, les retombées radioactives, les zones contaminées ?
- **Décision** : **pas de radioactivité** dans le jeu : ni retombées, ni zones contaminées, ni accident de centrale nucléaire.
- **Décision** : explosion **aérienne** (airburst), rayons tirés de Nukemap :

| Tête | Boule de feu | Rayonnement thermique | Souffle |
|---|---|---|---|
| TNO (M51, 100 kt) | 500 m | 4 km | 9 km |
| TNA (ASMP-A, 300 kt) | 600 m | 7 km | 13 km |

- **Décision** : effets par zone :
  - **Boule de feu** : tout est détruit ; tous les zombies et tous les humains sont tués.
  - **Rayonnement thermique** : allume des incendies (feu simulé objet par objet), blesse (brûlures) les humains exposés et enflamme les zombies, qui continuent d'avancer jusqu'à ce que le feu détruise leur tête. Un bâtiment protège ses occupants du rayonnement.
  - **Souffle** : les bâtiments et constructions sont endommagés (barre de vie) ; les humains sont blessés ; les zombies restent intacts, sur place.
  - Un véhicule blindé fermé ne protège ni du rayonnement thermique ni du souffle.
- **Décision** (multijoueur) : toute frappe nucléaire est soumise au **vote des joueurs** ([H-04](#h-04--vote-sur-les-actions-irréversibles)).
- **Statut** : Décidé.

### G-06 — État initial du monde

- **Problème** : que s'est-il passé entre le début de l'épidémie et 2030 ? Où sont les dépôts, les stocks, les zones détruites ? Quel est l'état des routes, des ponts, des usines ?
- **Décidé** : évacuation d'environ 7 millions de personnes vers l'outre-mer ; métropole entièrement zombifiée, sauf de rares poches isolées de survivants (A-10).
- **Statut** : Ouvert.

### G-07 — Conditions de victoire et de défaite

- **Problème** : quand la partie est-elle gagnée (reconquête totale, objectifs) ou perdue ?
- **Décision** : pour le moment, **aucune condition de victoire ni de défaite**. Le jeu tourne indéfiniment.
- **Décision** (serveur unique) : le monde peut être **remis à zéro** (nouvelle saison, [H-07](#h-07--remise-à-zéro-du-monde)).
- **Statut** : Décidé (pour le moment).

## H. Multijoueur et serveur

### H-01 — Architecture : serveur unique faisant autorité

- **Décision** : le jeu est **multijoueur seulement**. Il n'y a qu'**une partie officielle**, sur un serveur unique : `srv.chelsea486mhz.fr` (32 CPU, 128 Go de mémoire, 14 To de disque dur, 1 Gbit/s montant et descendant). Le code reste libre : d'autres peuvent héberger leur propre partie.
- **Décision** : le serveur **fait autorité** (A-12) : un seul processus de simulation porte tout le monde. Les clients, dans le navigateur, sont des terminaux : ils reçoivent l'état et envoient des ordres. Comme un client ne calcule rien, il ne peut pas tricher sur la simulation.
- **Décision** : pile **native Kubernetes**. L'**ingress** porte TLS et la protection réseau (débit et connexions par adresse IP, voir H-02).
- **Constat** (serveur) : un seul nœud **k3s** (v1.36), ingress **Traefik** (celui de k3s). Ses middlewares couvrent les limites par IP (`rateLimit`, `inFlightReq`) et les en-têtes COOP et COEP (F-04). Traefik ne voit que les requêtes et les connexions, pas les messages à l'intérieur d'un WebSocket : les quotas sur les ordres restent dans le serveur du jeu (H-02).
- **Piste** : trois charges distinctes :
  - la **simulation** : `StatefulSet` à **une seule réplique**, avec un volume persistant local (ext4 ou XFS, liens physiques, A-13). Un monde, c'est un processus : on ne le met pas à l'échelle horizontalement, comme une base de données primaire unique ;
  - le **service des tiles** : fichiers statiques immuables (H-06), servis à part pour ne pas charger la simulation ;
  - le **client** : fichiers statiques (WebAssembly, assets).
- **Décision** : ingress **Traefik** ; tiles et client servis par **nginx**, hors du processus de simulation.
- **Piste** : transport des ordres et de l'état en **WebSocket** ; WebTransport (QUIC, sans blocage en tête de file) à vérifier côté navigateurs et côté ingress.
- **Décision** : une **mise à jour du serveur = une remise à zéro du monde** (H-07). Une version du serveur dure le temps d'une saison.
- **Conséquences** :
  - Aucune migration : ni du format des instantanés (A-13), ni du protocole. Client et serveur ont **toujours la même version** ; à une mise à jour, les clients sont déconnectés et rechargent le client.
  - Un correctif urgent efface lui aussi le monde : les mises à jour se regroupent.
  - Un redémarrage **sans** changement de version (panne, redémarrage du nœud, déplacement du pod) reprend sur le dernier instantané.
- **Statut** : Décidé (serveur unique, autorité, Kubernetes, mise à jour = remise à zéro) ; Piste (découpage, transport).

### H-02 — Identité et abus

- **Décision** (pour le moment) : **aucun compte**, **anonymat total**. N'importe qui peut rejoindre la partie et l'influencer ; le jeu ne connaît aucune identité, même pseudonyme.
- **Décision** : la protection contre les abus est faite au niveau **réseau**, par l'**ingress** : limites de débit et de connexions par adresse IP.
- **Conséquence** : le jeu ne peut ni bannir un joueur, ni lui attribuer un ordre, ni limiter ses actions dans la durée : seule l'adresse IP le désigne, au niveau de l'ingress. Limites connues : plusieurs joueurs derrière une même adresse (CGNAT, entreprise) ; un seul joueur derrière beaucoup d'adresses (IPv6, VPN).
- **Piste** : les ordres coûteux pour le serveur (calcul d'itinéraire, ordres groupés à des milliers d'unités) ont en plus un **quota par connexion** dans le serveur, que l'ingress ne voit pas.
- **Statut** : Décidé (pour le moment) ; Piste (quotas applicatifs).

### H-03 — Commandement partagé

- **Décision** : **tout le monde commande tout**. Toute unité, tout lieu et tout stock appartiennent à tous les joueurs ; n'importe qui peut donner n'importe quel ordre, à tout moment.
- **Convention** : dans les fiches, « le joueur » désigne **n'importe quel joueur**.
- **Décision** : **aucun verrou** sur une unité. Le **dernier ordre l'emporte**, contrôle direct compris : un joueur peut prendre une unité tenue par un autre.
- **Piste** : l'interface montre qu'une unité est en contrôle direct, et depuis combien de temps son dernier ordre a été donné.
- **Statut** : Décidé.

### H-04 — Vote sur les actions irréversibles

- **Décision** : les actions **irréversibles** sont soumises au **vote des joueurs**.
- **Liste** (première version) :
  - frappe nucléaire, M51 ou ASMP-A ([G-05](#g-05--effets-des-armes-nucléaires)) ;
  - frappe sur un [barrage](game-elements/locations/barrages.md), dont la rupture noie la vallée ([A-20](#a-20--rupture-de-barrage)).
- **Décision** : la liste est **close** : aucun autre vote (la remise à zéro n'en est pas un, H-07).
- **Décision** : votent les **joueurs connectés** ; la **majorité** des voix exprimées l'emporte ; **une voix par identité**.
- **Décision** (recommandation adoptée par défaut, révisable) : sans compte (H-02), l'identité d'un votant est son **adresse IP**, transmise par l'ingress (le serveur du jeu ne fait confiance qu'à Traefik pour cet en-tête). Elle n'est gardée que le temps du vote. Une connexion ou un onglet ne peut pas servir d'identité : il suffit d'en ouvrir plusieurs.
- **Décision** (recommandation adoptée par défaut, révisable) : vote de **2 minutes** ; en cas d'égalité, la frappe est **refusée** ; il faut au moins une voix pour. Les délais de tir (CTM de Rosnay) courent à partir du vote acquis.
- **Statut** : Décidé.

### H-05 — Débit réseau et gestion d'intérêt

- **Constat** : 1 Gbit/s montant pour **1 000 joueurs simultanés** (objectif), sans CDN, donne environ **1 Mbit/s** (125 ko/s) par joueur en moyenne, **tiles comprises**. Un survol à basse altitude lit à lui seul environ 100 ko/s de tiles (C-04, P-07).
- **Piste** : **gestion d'intérêt** : chaque client ne reçoit que ce que sa caméra voit, comme un abonnement pub/sub par zone. Positions quantifiées et envoyées en différences ; fréquence réduite loin de la caméra ; en vue simplifiée (C-02), des agrégats (densités) plutôt que des agents.
- **Décision** (recommandation adoptée par défaut, révisable) : quand le lien sature, le serveur envoie par **ordre de priorité**, comme une QoS à files prioritaires sur un routeur : (1) accusés des ordres et votes ; (2) état des unités et des zombies proches de la caméra ; (3) tiles ; (4) état lointain, dégradé en premier (fréquence réduite, agrégats).
- **À décider** : budget par joueur entre l'état et les tiles (après P-22).
- **Statut** : Ouvert ; Prototype (P-22).

### H-06 — Diffusion des tiles

- **Décision** : les tiles sont servies par le serveur du jeu, **à la demande**, en HTTP statique ; il n'y a plus d'installation (B-01). Les paquets de tiles sont immuables et identifiés par leur empreinte BLAKE3 : ils se mettent en cache sans expiration (`Cache-Control: immutable`).
- **Décision** : pas de CDN ni de miroir.
- **Piste** : cache persistant dans le navigateur (OPFS), qui évite de retélécharger une région déjà visitée ; la place accordée par le navigateur est à mesurer. Les tiles de base (environ 40 Go) tiennent dans le cache mémoire du serveur.
- **Statut** : Décidé (à la demande, sans CDN) ; Prototype (P-22).

### H-07 — Remise à zéro du monde

- **Décision** : le monde peut être **remis à zéro** (nouvelle saison). Le monde de base (tiles de la chaîne) est immuable : une remise à zéro repart de lui et archive l'état de la partie terminée (instantané, A-13).
- **Décision** : la remise à zéro est décidée par l'**administrateur** du serveur (Linux ou Kubernetes), sans vote. Toute mise à jour du serveur en est une (H-01).
- **Statut** : Décidé.

----

## Prototypes

La liste détaillée des prototypes (objectifs, méthodes, critères de réussite) est dans [prototypes/README.md](https://github.com/fafapiti/prototypes/blob/main/README.md).
