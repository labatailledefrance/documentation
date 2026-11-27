# Unités

Index des unités jouables, tirées de l'arsenal français réel des années 2030. Chaque unité a sa propre fiche (versions, armement, équipement, besoins logistiques…).

Documents liés :

- [RESOURCES.md](../resources/RESOURCES.md) : ressources citées dans les fiches.
- [SYSTEMES.md](../systemes/SYSTEMES.md) : règles de simulation transverses.
- [ZOMBIES.md](../zombies/ZOMBIES.md) : comportement des zombies.
- [CONSTRUCTIONS.md](../constructions/CONSTRUCTIONS.md) : constructions possibles (ébauche).

Les statistiques chiffrées seront définies plus tard, dans le catalogue TOML de `beta1-server` qui met ces fiches en œuvre ([E-02](../../OPEN-PROBLEMS.md#e-02--format-des-données)) ; les fiches restent la conception et utilisent des niveaux qualitatifs (faible / moyen / élevé).

----

## Légende

Aucune unité n'est pilotée manuellement (voir [Pas de pilotage manuel](#pas-de-pilotage-manuel)). La légende indique le type d'ordres :

- **[O]** : ordres d'unité : le joueur donne une destination ou une mission ; l'unité, même seule, choisit son chemin et ses cibles.
- **[A]** : ordres de trajectoire (aéronefs, navires) : waypoints, patrouilles, axes de frappe (voir ci-dessous).

----

## Décisions de conception

### Contenu

Chaque version de chaque véhicule est incluse. Les bâtiments de la Marine sont nommés individuellement.

### Granularité

Un véhicule est un contenant : chaque membre d'équipage et chaque passager est un soldat individuel, commandable une fois descendu. Un Griffon, c'est un pilote, un chef d'engin et 8 fantassins, chacun avec sa propre fiche d'infanterie.

### Pas de pilotage manuel

- **Décision** : le joueur **ne pilote aucune unité** : pas de conduite d'un véhicule au clavier ou à la manette, pas de vue à la première ou à la troisième personne. Il peut toujours **commander une unité individuelle** (un soldat, un char, un drone MALE) en lui donnant des ordres, que l'unité exécute seule ([D-02](../../OPEN-PROBLEMS.md#d-02--ia-des-unités-amies)).
- **Décision** : **pas de micro-management**. Le joueur ne désigne **ni cible ni zone de tir** à une unité : l'unité connaît mieux sa situation que lui, et **engage ce qu'elle veut**.
- **Exceptions** : les **missions de feu à distance** et les **destructions** sont des ordres du joueur ([SYSTEMES](../systemes/SYSTEMES.md#appuis-feux)) :
  - artillerie ([CAESAR](vehicules-lourds/caesar.md), [LRU](vehicules-lourds/lru.md)) sur une **zone désignée par le joueur** ;
  - **axes de frappe** des aéronefs ; **appui-feu naval** sur une zone côtière ; l'**appui-feu à la demande** est demandé par le joueur ;
  - **destruction d'une infrastructure précise** (pont, rails, bâtiment) : toujours un ordre du joueur, **jamais une décision de l'unité** ;
  - **frappe nucléaire** et **frappe sur un barrage** : cible désignée par le joueur, puis soumise au vote.
- **Décision** : **un seul ordre à la fois** : pas de file d'ordres. Un nouvel ordre remplace le précédent. Comptent pour un seul ordre : un trajet par waypoints, une patrouille, une **livraison** (charger en A, aller, décharger en B), un ordre permanent ([SYSTEMES](../systemes/SYSTEMES.md#cargaison-et-logistique)). **Exception** : un SNLE peut mener plusieurs tirs nucléaires en parallèle, sur plusieurs cibles.
- **Décision** : un **ordre de groupe** (donné au chef de groupe, à une section, à une compagnie) **remplace l'ordre en cours** de chacun de ses membres, par les ordres issus de sa décomposition par l'IA du chef (voir *Chaîne de commandement et IA*).
- **Décision** : une unité **sans ordre tient sa position et se défend**.
- **Pourquoi** : le serveur reçoit des **ordres ponctuels** plutôt qu'un flux continu d'entrées, sensible à la latence ; et, sans verrou, plusieurs joueurs ne se disputent pas le volant d'une même unité ([H-03](../../OPEN-PROBLEMS.md#h-03--commandement-partagé)).

### Chaîne de commandement et IA

- **Échelons réels** : équipe, groupe, section, compagnie, régiment ; bâtiment, groupe naval ; patrouille aérienne. Décrits dans le catalogue ([D-01](../../OPEN-PROBLEMS.md#d-01--du-soldat-à-larmée)).
- Un ordre donné à un échelon est **décomposé par l'IA du chef** en ordres aux échelons inférieurs (répartition du secteur, formation). Si le chef est tué, le suivant dans l'ordre de commandement prend le relais. Un ordre donné directement à un échelon inférieur survit aux passages du chef, jusqu'au prochain ordre.
- Les joueurs peuvent **détacher ou rattacher** des unités : c'est un ordre comme un autre (révisable).
- **IA** : un ordre est un **état désiré**, que l'unité poursuit par une boucle de réconciliation ; ses **réflexes** (riposter, se mettre à couvert, se replier, se ravitailler) sont choisis par des scores d'utilité et suspendent l'ordre sans l'effacer. Les seuils (repli, munitions basses, pertes) sont des données du catalogue, par type d'unité ([D-02](../../OPEN-PROBLEMS.md#d-02--ia-des-unités-amies), révisable).

### Contrôle des unités aériennes (et navales)

Le joueur ne pilote pas les aéronefs. Il leur donne des ordres :

- **Waypoints** : trajet point par point.
- **Patrouilles stationnaires** : trajectoire précise au-dessus d'une zone (cercle, huit, hippodrome…).
- **Axes de frappe** : droite de longueur arbitraire le long de laquelle l'appareil tire, bombarde ou largue (parachutistes, matériel, eau).

### Efficacité réaliste

Le manque de pertinence d'une grande partie de l'arsenal face aux zombies est voulu. Le joueur dispose des stocks réels (obus flèche pour les chars, missiles antichars, radars…), avec leur efficacité réelle. C'est à lui de s'adapter : fabriquer des munitions adaptées et les livrer sur place.

### Logistique complète

Les unités non combattantes sont au cœur du jeu. Toute munition tirée doit avoir été produite, à partir de matières premières qu'il faut se procurer, puis livrée. Les ouvriers doivent être nourris. Sacrifier des civils aux zombies est possible ; le coût réel est la perte de main-d'œuvre.

### Classes d'itinéraire

- Chaque véhicule terrestre appartient à une **classe d'itinéraire** : véhicule léger, camion, porte-char chargé, chenillé lourd ([P-09](https://github.com/labatailledefrance/prototypes/blob/main/p09-chemin-routier/README.md), [SYSTEMES](../systemes/SYSTEMES.md#véhicules)). Une classe regroupe les véhicules de masse, de gabarit et de motricité voisins ; on ne crée pas une classe par modèle (chaque classe coûte une métrique en mémoire). Liste révisable.
- Les fiches indiquent la classe dans **Mobilité**.

### Génie militaire et génie civil

Le génie militaire ([sapeurs](infanterie/sapeur.md)) et le génie civil ([équipes de génie civil](soutien/equipes-genie-civil.md)) ont les mêmes compétences de génie : remise en service des infrastructures (centrales nucléaires, notamment), garnison des lieux (CTM, centrales, bases aériennes, gares, ports) ([LOCATIONS.md](../locations/LOCATIONS.md#règles-communes)). Le génie militaire a en plus des compétences de combat.

### Pas de politique

La politique n'est pas simulée. L'arme nucléaire est utilisable comme une arme, sous ses propres conditions d'emploi, dont le **vote des joueurs** (voir [SYSTEMES.md](../systemes/SYSTEMES.md#dissuasion-nucléaire)).

----

## Infanterie (soldats individuels, tous [O])

- [Fusilier](infanterie/fusilier.md)
- [Chef de groupe](infanterie/chef-de-groupe.md)
- [Tireur de précision](infanterie/tireur-de-precision.md)
- [Tireur d'élite](infanterie/tireur-elite.md)
- [Mitrailleur](infanterie/mitrailleur.md)
- [Grenadier / antichar](infanterie/grenadier-antichar.md)
- [Spécialiste incendiaire](infanterie/specialiste-incendiaire.md)
- [Infirmier de combat](infanterie/infirmier.md)
- [Sapeur](infanterie/sapeur.md)
- [Opérateur radio / JTAC](infanterie/operateur-radio-jtac.md)
- [Télépilote](infanterie/telepilote.md)
- [Forces spéciales](infanterie/forces-speciales.md)
- [Légion étrangère](infanterie/legion-etrangere.md) (trait de régiment)

## Véhicules légers ([O])

- [VBL](vehicules-legers/vbl.md)
- [Serval](vehicules-legers/serval.md)
- [Griffon](vehicules-legers/griffon.md)
- [VBCI](vehicules-legers/vbci.md)
- [Jaguar](vehicules-legers/jaguar.md)
- [Fardier, quad et moto](vehicules-legers/fardier-quad-moto.md)
- [Camions logistiques](vehicules-legers/camions-logistiques.md)
- [Véhicules civils récupérés](vehicules-legers/vehicules-civils.md)

## Véhicules lourds ([O])

- [Leclerc XLR](vehicules-lourds/leclerc.md)
- [CAESAR](vehicules-lourds/caesar.md)
- [LRU](vehicules-lourds/lru.md)
- [Dépanneurs et porte-chars](vehicules-lourds/depanneurs-porte-chars.md)
- [Trains](vehicules-lourds/trains.md)

### Génie

- [EBG](vehicules-lourds/genie/ebg.md)
- [SPRAT](vehicules-lourds/genie/sprat.md)
- [PFM](vehicules-lourds/genie/pfm.md)
- [SOUVIM](vehicules-lourds/genie/souvim.md)
- [Buffalo](vehicules-lourds/genie/buffalo.md)
- [EGAME](vehicules-lourds/genie/egame.md)

## Aérien ([A])

- [Tigre](aerien/tigre.md)
- [NH90 Caïman](aerien/nh90-caiman.md)
- [Caracal](aerien/caracal.md)
- [Gazelle](aerien/gazelle.md)
- [Guépard](aerien/guepard.md)
- [Rafale](aerien/rafale.md)
- [A400M](aerien/a400m.md)
- [C-130](aerien/c130.md)
- [Drones MALE (Reaper, Patroller)](aerien/drones-male.md)
- [Canadair](aerien/canadair.md)
- [Dash 8 bombardier d'eau](aerien/dash8.md)

## Naval ([A])

- [PHA Mistral](naval/pha-mistral.md)
- [Charles de Gaulle](naval/charles-de-gaulle.md)
- [FREMM](naval/fremm.md)
- [FDI](naval/fdi.md)
- [BRF (ravitailleurs)](naval/brf.md)
- [Engins de débarquement](naval/engins-de-debarquement.md)
- [SNA Suffren](naval/sna-suffren.md)
- [SNLE Triomphant](naval/snle-triomphant.md)
- [Chasseurs de mines](naval/chasseurs-de-mines.md)
- [Patrouilleurs et vedettes](naval/patrouilleurs.md)

## Soutien et civils

- [Gendarmerie et police](soutien/gendarmerie-police.md)
- [Sapeurs-pompiers](soutien/sapeurs-pompiers.md)
- [Équipes sanitaires](soutien/equipes-sanitaires.md)
- [Hôpital de campagne](soutien/hopital-de-campagne.md) (construction, pas une unité)
- [Civils](soutien/civils.md)
- [Milices](soutien/milices.md)
- [Équipes de génie civil](soutien/equipes-genie-civil.md)
