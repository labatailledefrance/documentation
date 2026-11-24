# Unités

Index des unités jouables, tirées de l'arsenal français réel des années 2030. Chaque unité a sa propre fiche (versions, armement, équipement, besoins logistiques…).

Documents liés :

- [RESOURCES.md](../resources/RESOURCES.md) : ressources citées dans les fiches.
- [SYSTEMES.md](../systemes/SYSTEMES.md) : règles de simulation transverses.
- [ZOMBIES.md](../zombies/ZOMBIES.md) : comportement des zombies.
- [CONSTRUCTIONS.md](../constructions/CONSTRUCTIONS.md) : constructions possibles (ébauche).

Les statistiques chiffrées seront définies plus tard ; les fiches utilisent des niveaux qualitatifs (faible / moyen / élevé).

----

## Légende

- **[D]** : contrôle direct par le joueur.
- **[A]** : unité pilotée par le jeu ; le joueur lui donne des ordres (voir ci-dessous).

----

## Décisions de conception

### Contenu

Chaque version de chaque véhicule est incluse. Les bâtiments de la Marine sont nommés individuellement.

### Granularité

Un véhicule est un contenant : chaque membre d'équipage et chaque passager est un soldat individuel, contrôlable une fois descendu. Un Griffon, c'est un pilote, un chef d'engin et 8 fantassins, chacun avec sa propre fiche d'infanterie.

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

- Chaque véhicule terrestre appartient à une **classe d'itinéraire** : véhicule léger, camion, porte-char chargé, chenillé lourd ([P-09](../../../prototypes/p09-chemin-routier/README.md), [SYSTEMES](../systemes/SYSTEMES.md#véhicules)). Une classe regroupe les véhicules de masse, de gabarit et de motricité voisins ; on ne crée pas une classe par modèle (chaque classe coûte une métrique en mémoire). Liste révisable.
- Les fiches indiquent la classe dans **Mobilité**.

### Génie militaire et génie civil

Le génie militaire ([sapeurs](infanterie/sapeur.md)) et le génie civil ([équipes de génie civil](soutien/equipes-genie-civil.md)) ont les mêmes compétences de génie : remise en service des infrastructures (centrales nucléaires, notamment), garnison des lieux (CTM, centrales, bases aériennes, gares, ports) ([LOCATIONS.md](../locations/LOCATIONS.md#règles-communes)). Le génie militaire a en plus des compétences de combat.

### Pas de politique

La politique n'est pas simulée. L'arme nucléaire est utilisable comme une arme, sous ses propres conditions d'emploi, dont le **vote des joueurs** (voir [SYSTEMES.md](../systemes/SYSTEMES.md#dissuasion-nucléaire)).

----

## Infanterie (soldats individuels, tous [D])

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

## Véhicules légers ([D])

- [VBL](vehicules-legers/vbl.md)
- [Serval](vehicules-legers/serval.md)
- [Griffon](vehicules-legers/griffon.md)
- [VBCI](vehicules-legers/vbci.md)
- [Jaguar](vehicules-legers/jaguar.md)
- [Fardier, quad et moto](vehicules-legers/fardier-quad-moto.md)
- [Camions logistiques](vehicules-legers/camions-logistiques.md)
- [Véhicules civils récupérés](vehicules-legers/vehicules-civils.md)

## Véhicules lourds ([D])

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

## Naval (majoritairement [A])

- [PHA Mistral](naval/pha-mistral.md)
- [Charles de Gaulle](naval/charles-de-gaulle.md)
- [FREMM](naval/fremm.md)
- [FDI](naval/fdi.md)
- [BRF (ravitailleurs)](naval/brf.md)
- [Engins de débarquement](naval/engins-de-debarquement.md) ([D] pendant le débarquement)
- [SNA Suffren](naval/sna-suffren.md)
- [SNLE Triomphant](naval/snle-triomphant.md)
- [Chasseurs de mines](naval/chasseurs-de-mines.md)
- [Patrouilleurs et vedettes](naval/patrouilleurs.md)

## Soutien et civils

- [Gendarmerie et police](soutien/gendarmerie-police.md)
- [Sapeurs-pompiers](soutien/sapeurs-pompiers.md)
- [Équipes sanitaires](soutien/equipes-sanitaires.md)
- [Hôpital de campagne](soutien/hopital-de-campagne.md)
- [Civils](soutien/civils.md)
- [Milices](soutien/milices.md)
- [Équipes de génie civil](soutien/equipes-genie-civil.md)
