# Systèmes de simulation

Règles transverses décidées en concevant les unités. Elles s'appliquent à tout le jeu, pas à une unité en particulier.

----

## Multijoueur

Décidé avec [H-01 à H-07](../../OPEN-PROBLEMS.md#h-multijoueur-et-serveur).

- **Une seule partie officielle**, multijoueur seulement, sur un serveur unique qui fait autorité ; les joueurs y accèdent depuis leur navigateur ([H-01](../../OPEN-PROBLEMS.md#h-01--architecture--serveur-unique-faisant-autorité)).
- **Anonymat** : aucun compte, pour le moment ; n'importe qui peut rejoindre la partie et l'influencer ([H-02](../../OPEN-PROBLEMS.md#h-02--identité-et-abus)).
- **Tout le monde commande tout** : toute unité, tout lieu et tout stock appartiennent à tous les joueurs. Aucun verrou : le dernier ordre l'emporte, contrôle direct compris. Dans les fiches, « le joueur » désigne n'importe quel joueur ([H-03](../../OPEN-PROBLEMS.md#h-03--commandement-partagé)).
- **Vote** : les actions irréversibles (frappe nucléaire, frappe sur un barrage, liste close) sont soumises au vote des joueurs connectés : majorité des voix exprimées, une voix par adresse IP, 2 minutes, égalité = refus ([H-04](../../OPEN-PROBLEMS.md#h-04--vote-sur-les-actions-irréversibles)).
- **Monde persistant** : il tourne en temps réel, même quand personne n'est connecté. L'administrateur du serveur peut le remettre à zéro (nouvelle saison) ; toute mise à jour du serveur le remet à zéro ([H-01](../../OPEN-PROBLEMS.md#h-01--architecture--serveur-unique-faisant-autorité), [H-07](../../OPEN-PROBLEMS.md#h-07--remise-à-zéro-du-monde)).

## Monde

- **Échelle 1:1** : le monde est représenté à l'échelle réelle. Distances, superficies, durées de trajet et consommations en découlent directement.
- **Temps réel** : le temps de jeu est le temps réel, **sans accélération** ([A-02](../../OPEN-PROBLEMS.md#a-02--accélération-du-temps)). Les distances comptent donc pleinement : un convoi Cherbourg–Brest, c'est plusieurs heures de route ; une traversée depuis l'outre-mer, des semaines.
- **Zone simulée** : la France métropolitaine et les outre-mer uniquement, **sans les TAAF** (Kerguelen, Crozet, Amsterdam et Saint-Paul, îles Éparses, Terre Adélie) ni Clipperton, inhabitables, **ni le Pacifique** (Nouvelle-Calédonie, Polynésie française, Wallis-et-Futuna), que les données de l'IGN ne couvrent pas ([C-01](../../OPEN-PROBLEMS.md#c-01--précision-numérique)). En dehors, rien n'est simulé, pour des raisons de performance : le globe reste affiché, mais ces surfaces sont grisées.
- **Mer** : les navires naviguent réellement sur toutes les mers, hors de la zone simulée comprise, sur les vraies routes maritimes, avec durée et consommation. Les canaux de Suez et de Panama sont praticables, sans être simulés ([G-01](../../OPEN-PROBLEMS.md#g-01--frontières-avec-les-zones-non-simulées)).
- **Ralentissement** : si le calcul ne suit plus, le temps de jeu ralentit sous ×1, pour tous les joueurs à la fois ; les combats ne sont jamais simplifiés ([A-02](../../OPEN-PROBLEMS.md#a-02--accélération-du-temps)).
- **Cours fermées** : la génération des bâtiments perce un passage entre chaque cour fermée et la rue la plus proche ([B-02](../../OPEN-PROBLEMS.md#b-02--génération-des-bâtiments)).
- **Bâtiments** : chaque bâtiment de chaque ville est présent. Ils sont **générés au chargement**, de façon déterministe, à partir des données de l'IGN (BD TOPO) : emprise réelle extrudée sur le relief, hauteur de la BD TOPO (à défaut, étages ou médiane locale), toit plat, à deux ou quatre pans ou en mansarde selon la hauteur du toit, fenêtres dans la texture, porches aux passages percés ; jamais stockés. Les églises, châteaux, forts, phares et moulins ont des gabarits par nature ; seuls les monuments d'une **liste écrite à la main** sont modélisés fidèlement ([B-02](../../OPEN-PROBLEMS.md#b-02--génération-des-bâtiments), [P-08](https://github.com/fafapiti/prototypes/blob/main/p08-generation-batiments/README.md)).
- **Routes** : le réseau routier réel est reproduit (IGN BD TOPO), chemins et routes empierrées compris. Les véhicules y suivent des **itinéraires** calculés sur tout le réseau national (contraction hierarchies personnalisables, [A-07](../../OPEN-PROBLEMS.md#a-07--recherche-de-chemin), [P-09](https://github.com/fafapiti/prototypes/blob/main/p09-chemin-routier/README.md)). Il n'y a plus de code de la route : les sens uniques, les ronds-points et les interdictions de tourner sont ignorés ; seules les **chaussées séparées** (autoroutes, voies rapides, bretelles) ont un sens.
- **Relief** : IGN RGE ALTI, stocké en nœuds de 2 m au décimètre près ([P-07](https://github.com/fafapiti/prototypes/blob/main/p07-chaine-manche/README.md)).
- **Données du monde** : les tiles sont précalculées par le projet à partir des données de l'IGN et de l'INSEE, et installées sur le serveur (35 à 41 Go pour la France) ; le client les télécharge à la demande et les garde en cache ([H-06](../../OPEN-PROBLEMS.md#h-06--diffusion-des-tiles)). Le jeu ne relit jamais les données brutes ; il recalcule au chargement ce qui est local et rapide (bâti de 1 m, capacités, arbres, bâtiments), le reste est précalculé (zone accessible, passages, habitants par commune, mairies) ([B-01](../../OPEN-PROBLEMS.md#b-01--chaîne-de-traitement-des-données-ign)).
- **Côte** : la mer commence au trait de côte des plus hautes eaux ; les estrans sont de la terre, que la marée recouvre. Les îles étrangères proches (Jersey, Guernesey, îles des Antilles) sont grisées.
- **Ponts** : chaque pont a une charge limite. Un véhicule trop lourd ne peut pas le franchir sans risque : ses itinéraires l'évitent. Les données ne la donnent presque jamais : elle vient d'une heuristique (importance de la route portée, nombre de voies, longueur, ce que le pont franchit), avec les restrictions réelles de la BD TOPO et des valeurs écrites à la main pour quelques grands ouvrages ([B-03](../../OPEN-PROBLEMS.md#b-03--charge-limite-des-ponts)). Les grands axes portent les convois les plus lourds ; les ponts des petites routes refusent les chars. Un pont du jeu est une **traversée** : ses tabliers (un par sens sur une voie rapide) sont détruits ou réparés ensemble.
- **Coupures** : un pont détruit, des gravats, une épave ou un barrage du génie coupent la route ; les itinéraires qui l'empruntaient sont recalculés aussitôt. Une route dégagée ou un pont du génie la rouvrent.
- **Cours d'eau** : profondeur arbitraire, sans estimation à partir des données.

## Repères et précision

Décidé avec [P-06](https://github.com/fafapiti/prototypes/blob/main/p06-origine-flottante/README.md) ([C-01](../../OPEN-PROBLEMS.md#c-01--précision-numérique)).

- **Zones de projection** : Lambert-93 en métropole, Corse comprise ; UTM ou Lambert local pour chaque territoire (Antilles, Guyane, Réunion, Mayotte, Saint-Pierre-et-Miquelon). Le Pacifique n'est pas dans le jeu (voir *Monde*). Projection d'usage de chaque territoire à vérifier.
- **Position dans la simulation** : une tile (zone, ix, iy) et une position locale en flottant 32 bits, **centrée** sur la tile (±500 m). Le calcul ne dépend pas de l'endroit du monde, et le passage d'une tile à l'autre est exact.
- **Navires en mer** : position géocentrique (ECEF) en 64 bits, hors des tiles ; à 30 km d'une côte (valeur de départ), le navire entre dans les tiles du territoire, sans saut.
- **Placement d'une tile sur la Terre** : au second ordre (courbure comprise) ; un plan tangent serait faux de 39 mm aux coins.
- **Rendu** : relatif à la caméra ; l'origine du rendu est recalée quand la caméra s'en éloigne de plus de 5 km. Les foules sont rendues par instanciation ; environ 100 000 entités rendues au plus. Les bâtiments sont rendus en morceaux fusionnés de 250 m, envoyés au GPU par une file limitée en octets par frame ([C-05](../../OPEN-PROBLEMS.md#c-05--rendu-des-foules)).
- **Globe** : découpé en morceaux, chacun avec son origine locale ; de près, le sol des tiles le remplace.
- **Temps** : compteur de ticks entier sur 64 bits ; les animations (eau, fumée) reçoivent des phases calculées depuis les ticks, jamais un temps absolu en flottant. Une date relative dans une fiche compacte (dernière mise à jour d'un colon) peut rester sur 32 bits, comparée par soustraction modulaire.
- **Instantané du monde** (pris par le serveur toutes les 5 minutes, relu seulement par la même version) : positions exactes dans les tiles actives, 1/64 m dans les tiles inertes ([A-13](../../OPEN-PROBLEMS.md#a-13--sauvegardes)).

## Simulation par tiles

- Le monde est découpé en tiles d'environ 1 km (taille à confirmer), sur une grille alignée sur le Lambert-93 en métropole, et sur la projection de chaque territoire outre-mer (voir *Repères et précision*).
- **Tile active** : des humains au contact possible de zombies, ou voisine d'une telle tile. Simulation complète, zombies en agents individuels.
- **Tile inerte** : zombies seuls. État stocké (nombre de zombies, barre de vie des bâtiments endommagés, part brûlée…). Une tile qui a déjà été active garde tout : position de chaque zombie (redevenu errant) et carte des corps sur disque, un fichier compressé et immuable par tile (environ 4 octets par zombie), auquel une sauvegarde de partie fait référence par lien physique ([A-13](../../OPEN-PROBLEMS.md#a-13--sauvegardes)) ; constructions et leur état en mémoire, pour l'affichage. Sans humain, les zombies n'attaquent pas les constructions : une base abandonnée reste intacte et peut être reprise.
- **Affichage** : les tiles sont invisibles à l'écran. Une tile inactive est affichée avec ses constructions, sans ses zombies, légèrement grisée (brouillard de guerre).
- **Activation et désactivation** : une tile reste active environ une minute de jeu après la disparition de sa raison d'être active. À l'activation loin de tout humain, ses zombies naissent en veille profonde. L'activation est **étalée** sur environ une à quatre secondes de jeu (terrain, placement ou restauration, création des zombies par lots) ; pendant ce temps, la tile reste inerte pour ses voisines. Les zombies sont placés dans la **zone accessible** de la tile, précalculée par la chaîne de données ([P-04](https://github.com/fafapiti/prototypes/blob/main/p04-activation/README.md)).
- **Frontières** : un zombie qui sort de sa tile passe dans la voisine. Un zombie **sans cible** ne sort pas vers une tile non active : il est renvoyé. Un zombie en poursuite traverse. Une tile active compte au plus **65 000 zombies** ; au-delà, elle refuse les entrées et les zombies attendent au bord, comme devant une case pleine.
- **Bâtiments** : un bâtiment est un contenant, comme un véhicule. Humains et zombies y entrent et en sortent ; tous les humains à l'intérieur peuvent tirer par les fenêtres. Capacité selon l'emprise et la hauteur. Les zombies n'y entrent que s'il contient des humains, et en sortent aussitôt qu'il n'en contient plus ; à l'activation d'une tile, tous sont placés dans les rues. Ils repèrent les humains par les fenêtres ou au bruit des tirs. Chaque bâtiment a une porte, sur la case accessible au pied de ses murs la plus proche de la rue, calculée par la chaîne de données : les zombies entrent par elle, et elle représente le bâtiment occupé dans la perception des zombies (un seul point, qui porte le nombre de tireurs). Le combat à l'intérieur est auto-résolu chaque seconde (chance de tuer un zombie ou d'exposer un humain, selon les effectifs, les armes et les munitions, consommées dans le dépôt de la mairie de la commune ; sans munitions, les occupants ne se battent plus). Une seule porte par bâtiment. L'intérieur n'est pas simulé ([B-02](../../OPEN-PROBLEMS.md#b-02--génération-des-bâtiments)).
- **Migration** : les zombies des tiles inertes passent lentement d'une tile à l'autre, attirés par les stimuli (bruit, fumée d'incinération). Une tile sauvegardée n'est pas relue : ses arrivées et départs sont notés dans un journal, rejoué à son activation (arrivées près du bord d'entrée, départs les plus proches du bord de sortie). Une tile inerte de 40 000 zombies n'en reçoit plus ; les autres restent sur place.
- **Feu** : continue dans les tiles inertes, de façon agrégée.
- **Aéronefs** : n'activent une tile qu'à leurs points d'action (frappe, largage, poser).
- **Tirs à longue portée** : un tir observé ou une frappe sur une tile inerte l'active.
- **Perception des zombies** : un arbre k-d des humains par tile, reconstruit à chaque tick ; environ 4 fois par seconde pour une tile très peuplée ([P-02](https://github.com/fafapiti/prototypes/blob/main/p02-perception/README.md)). Le coût ne dépend pas de la distance de vue. Un zombie dont la vue déborde de sa tile interroge aussi les arbres des tiles voisines. La distance de vue est un attribut de la tile, tiré de sa part de bâti à la préparation de la carte.
- **Déplacement des zombies** : chaque tile calcule deux flow fields de 2 m (proies, bruits), recalculés une fois par seconde en étalant le calcul sur les ticks de la seconde ; après un obstacle, recalcul partiel s'il est petit. Les tiles voisines échangent leurs bordures à chaque recalcul, avec l'horodatage de la dernière observation, comme l'alerte relayée ([P-03](https://github.com/fafapiti/prototypes/blob/main/p03-flow-fields/README.md), [A-07](../../OPEN-PROBLEMS.md#a-07--recherche-de-chemin)).
- **Index des zombies** : une grille dense de cases de 2 m par tile (séparation, relais, capacité), reconstruite à chaque tick ; places réservées exactement sur toutes les cases ([A-05](../../OPEN-PROBLEMS.md#a-05--entassement-et-franchissement-dobstacles)).
- **Tile sûre** : des humains sans zombie à portée (outre-mer, zones reconquises). Économie simulée, unités du joueur individuelles, population civile en dossiers par bâtiment (voir *Colonisation*), visible individuellement dehors quand la caméra s'approche. Redevient active dès qu'un zombie arrive à portée.
- **Métropole** : entièrement zombifiée. Seules subsistent de rares poches isolées de survivants (moins de 1 000 personnes chacune), connues dès le début et qui sont des objectifs de jeu ([A-10](../../OPEN-PROBLEMS.md#a-10--population-civile), [liste](../locations/survivants.md)).
- **Densité de zombies** : proportionnelle à la population, moins les évacués. Les habitants des carreaux de 200 m de l'INSEE sont répartis sur les bâtiments résidentiels, puis complétés commune par commune jusqu'à la population légale ; chaque fragment tile × commune en reçoit la somme ([A-10](../../OPEN-PROBLEMS.md#a-10--population-civile), [A-19](../../OPEN-PROBLEMS.md#a-19--suivi-des-zombies-par-commune)).
- **Outre-mer** : totalement exempts de zombies ; toutes leurs tiles sont sûres. Environ 10 millions d'habitants, dont environ 7 millions de réfugiés évacués de métropole.

## Colonisation

- Des navires de civils attendent au large de Brest. Le joueur débarque les colons par Brest, puis par **tout port reconquis** (donc déminé), et les installe **où il veut** sur le territoire reconquis ([A-15](../../OPEN-PROBLEMS.md#a-15--civils-en-métropole--colonies)).
- **Transport** : les navires de colons sont des unités simulées. Le joueur embarque les colons manuellement, les fait voyager à vitesse réelle et les débarque manuellement. Le rythme d'arrivée est limité par la flotte disponible.
- Les colons fournissent la **main-d'œuvre** de la production locale.
- **Chute d'une colonie** : émergente. Les zombies entrent, l'infection se propage selon les règles normales, et les civils deviennent des zombies.
- Une colonie entourée de zones nettoyées est en tiles sûres ; une colonie isolée au contact des zombies fait vivre des tiles actives à sa périphérie.
- **Population** : pas d'âge modélisé, tous les colons ont les mêmes règles. Le temps ne passe pas pour eux (ni vieillissement, ni naissances) : la population ne change que par les débarquements, les morts et les zombifications.
- **Pas d'objet « colonie »** : une colonie n'est que des colons logés dans des bâtiments, sur la surface de communes.
- **Dépôt de la commune** : la [mairie](../locations/mairies.md) de chaque commune est le dépôt de toutes les ressources de ses colons (vivres, eau potable, médicaments, vêtements chauds, couvertures, munitions). Chaque bâtiment de la commune y puise directement, sans transport simulé, pour tous ses occupants : colons, ouvriers et soldats logés (munitions comprises). Une unité en opération hors d'un bâtiment n'est ravitaillée que par sa chaîne logistique. La mairie est le dépôt **par défaut** : chaque bâtiment puise chaque ressource au **dépôt construit le plus proche** qui la garde ([dépôt de munitions, dépôt de carburant](../constructions/CONSTRUCTIONS.md#logistique-et-soutien)), dans sa commune ou une commune voisine ; à défaut, à la mairie de sa commune. Les vivres, l'eau potable, les médicaments, les vêtements chauds et les couvertures restent à la mairie. Le stock ne change pas de dépôt tout seul : le joueur le transporte, comme toute cargaison. Une commune voisine est une commune qui **touche** celle du bâtiment (graphe d'adjacence précalculé par la chaîne de données). Un dépôt vide ou occupé par des zombies est sauté : le bâtiment puise au suivant qui a du stock et n'est pas occupé, puis à la mairie. Capacité illimitée. Une mairie endommagée doit être réparée par le génie pour servir de nouveau ; une mairie effondrée est reconstruite à l'identique, par exception, avec les règles d'une construction. Une mairie occupée par des zombies garde son stock sur place, mais personne n'y puise tant qu'elle n'est pas reprise. Une seule mairie par commune (la mairie principale) ; à Paris, Lyon et Marseille, une par arrondissement, qui compte aussi comme une commune pour le statut. Sans mairie dans les données, le plus grand bâtiment de la commune, public ou privé, en tient lieu.
- **Capacité** : un bâtiment loge un colon par 35 m² de plancher et offre un poste de travail par 20 m² (valeurs de départ, tirées de la BD TOPO : emprise, étages, logements, usage) ([P-05](https://github.com/fafapiti/prototypes/blob/main/p05-civils/README.md)).
- **Représentation** : chaque colon a son propre dossier (identité, logement, travail, jauges, expérience, compétences, maladie), rangé dans le dossier du bâtiment où il se trouve, hors de l'ECS. Pas d'agrégat statistique : chacun garde son identité ([A-15](../../OPEN-PROBLEMS.md#a-15--civils-en-métropole--colonies)).
- **Jauges des civils** : mises à jour toutes les 10 s en tile sûre (30 s au-delà de 2 millions de colons ; 30 à 60 s outre-mer, dans les tiles sans colonie ni caméra), sur l'intervalle exact écoulé. Même période pour un colon individualisé.
- **Individualisation** : seuls les colons **dehors** deviennent des entités (marche sous la caméra, rue d'une tile active). Jamais les occupants d'un bâtiment, même au contact des zombies ; un chantier extérieur en tile sûre reste agrégé comme un bâtiment. Retour au dossier 30 s après, avec 100 m d'hystérésis sur le rayon de la caméra.
- **Trajets** : à pied sous la caméra (A\* reprenable sur plusieurs ticks) ; ailleurs, ou sans chemin à pied, trajet abstrait, en tile sûre seulement. En tile active, un colon sans chemin à pied reste dans son bâtiment.
- **Sauvegarde** : dossiers des colons sans perte, dans le fichier de leur tile ([A-13](../../OPEN-PROBLEMS.md#a-13--sauvegardes)).

## Lieux et territoire

- **Lieux d'intérêt** : sites réels qui donnent une capacité au joueur ([liste](../locations/LOCATIONS.md)) : CTM de Rosnay (tirs de M51), centrales nucléaires et barrages (électricité régionale), bases aériennes et aéroports, gares et ports (logistique aérienne, ferroviaire et navale).
- **Garnison du génie** : un lieu n'est utilisable qu'avec une garnison du génie, militaire ou civil : 10 pour une gare, 100 pour un grand port (10 pour un port de pêche ou de plaisance), 100 pour un aéroport ou une base aérienne, 100 pour une CNPE, 10 pour les autres lieux ; aucune pour une piste sommaire, qui ne reçoit que les avions adaptés (A400M, C-130). Un port mixte est un grand port dès qu'il a un quai de commerce ou militaire. La garnison suffit, même avec des zombies dans l'emprise ; sous l'effectif minimal, le lieu est aussitôt inutilisable. Pour les centrales, elle n'est exigée qu'au redémarrage. Un lieu à cheval sur plusieurs communes appartient à toutes.
- **Génie** : le génie militaire et le génie civil ont les mêmes compétences de génie ; le militaire a en plus des compétences de combat.
- **Territoire (cosmétique)** : le statut des communes n'a aucun effet sur le jeu ; il colore la carte en vue simplifiée. Une commune est libérée (bleue) quand au moins 10 colons habitent sa [mairie](../locations/mairies.md) et qu'aucun zombie n'est sur sa surface ; elle passe au jaune dès qu'un zombie revient, y compris par migration entre tiles inertes ([A-19](../../OPEN-PROBLEMS.md#a-19--suivi-des-zombies-par-commune)) ; elle redevient rouge sans colons dans la mairie ou sans humain sur sa surface.
- **Électricité** : une [centrale nucléaire](../locations/cnpe.md) ou un [barrage](../locations/barrages.md) remis en service alimente sa région, définie par un découpage de Voronoï **fixe** (chaque commune rattachée à la centrale la plus proche de sa mairie) : une région dont la centrale est éteinte reste dans le noir, pour inciter à libérer toutes les centrales. Toute la région est alimentée d'un bloc (réseau non simulé), communes libérées ou non. L'outre-mer a l'électricité d'office. Pour redémarrer une CNPE, il suffit d'y livrer du **Gazole routier** pour ses groupes électrogènes de secours ; le génie fait le reste. Elle fonctionne ensuite indéfiniment, sans approvisionnement ([G-04](../../OPEN-PROBLEMS.md#g-04--énergie)). Pas de centrales thermiques.
- **Pistes** : chaque aéronef a une longueur de piste minimale ; les gros avions ne se posent pas partout ([bases aériennes](../locations/bases-aeriennes.md)). Les aéroports civils comptent comme des bases.
- **Quais** : chaque navire a un tirant d'eau et une longueur ; il n'accoste que dans un port assez profond et aux quais assez longs, et la marée limite l'accès aux ports à marée ([ports](../locations/ports.md)).
- **Trains** : [unités simulées](../units/vehicules-lourds/trains.md) qui roulent sur les voies et fonctionnent comme des véhicules (chargement, déchargement, destruction) ; les zombies ne peuvent ni les arrêter ni y entrer. Deux modèles, fret et passagers, non recomposables ; chaque gare a une chance sur cinq d'en avoir un au début. Tout le monde peut conduire ; le matériel trouvé est en état impeccable. Le réseau ferré est intact au début. **Tous les trains sont électriques** : un trajet n'est possible que s'il est entièrement dans des régions alimentées. Une voie obstruée ou coupée doit être dégagée ou réparée par le génie ; les rails peuvent être détruits.
- **Barrages** : les 10 plus gros barrages de France, par volume de retenue, sont dans le jeu. Un barrage a une barre de vie, comme toute infrastructure, et produit tant qu'il n'est pas rompu : plusieurs frappes de bombe guidée ou d'artillerie le rompent, sans reconstruction possible, et sa région reste dans le noir pour toujours. La crue précalculée noie la vallée en aval ; elle relocalise les zombies touchés au hasard sur le périmètre de la zone inondée et détruit humains (sans corps), bâtiments, ponts et véhicules ([A-20](../../OPEN-PROBLEMS.md#a-20--rupture-de-barrage)). Tout ordre de frappe visant un barrage est soumis au **vote des joueurs** ([H-04](../../OPEN-PROBLEMS.md#h-04--vote-sur-les-actions-irréversibles)).

## Périmètre

- **Contenu exhaustif** : chaque version de chaque véhicule de l'arsenal est incluse. Les bâtiments de la Marine sont nommés individuellement.
- **Pas de politique** : la légitimité politique, l'opinion publique et les décisions politiques ne sont pas simulées. L'arme nucléaire est une arme comme une autre, soumise à ses propres conditions d'emploi (voir plus bas).
- **Hors périmètre pour l'instant** : l'ordre public dans les zones reconquises (pillages, émeutes). À revoir quand le jeu sera jouable.
- **Trop bas niveau, non simulé** : la surchauffe et le changement de canon des armes, la réactivité aux ordres, l'usure du corps des zombies.
- **Non simulé** : les animaux.

## Soldats

- Chaque soldat a ses propres jauges de **fatigue**, de **moral**, de **faim** et de **soif**. Elles évoluent **une fois par seconde** de jeu, pas à chaque tick : elles changent lentement, et cela divise leur coût par 20 (P-00).
- **Événements ponctuels** (mort d'un civil à proximité, chef tué…) : ils sont mis en file et appliqués aux jauges lors de la mise à jour suivante, soit au plus une seconde de jeu plus tard. Leurs effets s'additionnent : l'ordre des événements au cours d'une même seconde ne change pas le résultat.
- **Débandade** : évaluée à la même mise à jour d'une seconde, quand le moral passe sous un seuil.
- **Commandement** : un chef de groupe améliore le moral et la précision de son groupe.
- **Mort de civils** : la mort d'un civil à proximité d'une unité fait baisser son moral (événement ponctuel, appliqué à la mise à jour suivante des jauges).
- **Usure psychologique** : jauge lente, distincte du moral, mise à jour avec les autres. Elle monte avec l'exposition au combat, les pertes et la manipulation des corps, et ne baisse qu'au repos en tile sûre. Au-delà d'un seuil, le soldat devient **blessé psychique** : évacué vers une tile sûre, il est inapte environ un jour de jeu, puis revient avec une usure juste sous le seuil (il rechute vite sans repos supplémentaire). La perte du chef de groupe augmente l'usure du groupe. Elle impose la relève des unités ([A-18](../../OPEN-PROBLEMS.md#a-18--chaîne-sanitaire--blessures-et-usure)).
- **Blessures** : les zombies n'en font pas (tout contact zombifie). Elles sont d'origine humaine et accidentelle : appuis-feux trop proches, tirs amis, brûlures et souffle d'une arme nucléaire, accidents de véhicules, de chantier et de manipulation des corps, brûlures, effondrements (pour les humains proches du bâtiment ; ses occupants, eux, meurent). La fatigue en augmente le risque. Le manque de sommeil n'est pas simulé à part : la fatigue le couvre.
- **Blessés** : un blessé grave meurt s'il n'est pas opéré à temps ; l'hôpital de campagne vaut donc par sa distance au front. Soignés, les blessés retournent au combat, après une inaptitude plus longue que celle d'un blessé psychique ; un amputé est réaffecté à l'arrière, militaire non combattant, à un poste pertinent pour son métier (un infirmier de combat en hôpital de campagne, par exemple), et perd son expérience. Sans poste pertinent (un mitrailleur, par exemple), il devient **blessé de guerre** : sans utilité, c'est une charge à gérer. Il consomme vivres, eau potable, logement et médicaments comme un colon, et peut être rapatrié en outre-mer par navire. Il n'agit pas sur le moral ni sur l'usure des autres. Un blessé mort de ses blessures ne se relève pas.
- **Expérience** : chaque soldat a une expérience individuelle, qui améliore sa précision et sa résistance à l'usure. Elle ne se gagne qu'au combat. Tout le monde commence la partie sans expérience.
- **Ouvriers et colons** : l'usure et l'expérience s'appliquent aussi à eux. Par exception, leur expérience se gagne par le travail (chantiers, manipulation des corps) et améliore leur rendement et leur résistance à l'usure.
- **Changement de rôle** : l'expérience suit la personne (colon devenu ouvrier ou milicien, milicien devenu soldat régulier).
- **Formation** : les milices peuvent devenir des unités régulières ; leurs miliciens gardent leur expérience. Les forces spéciales ne peuvent pas être formées pendant la partie : c'est une ressource finie.
- **Compétences** (ouvriers, civils) : gérées lorsqu'elles sont directement liées aux ressources du jeu (électricien, soudeur, chimiste…).

## Corps et maladies

- **Corps** (zombies et humains) : comptés par case de 10 m, générés à l'affichage ; ils ne se décomposent pas et ne sont pas contagieux. Ce sont des cargaisons : ils doivent être évacués avant de construire à leur place, puis traités en fosse commune ou par incinération. La fumée de l'incinération attire les zombies, avec une portée qui dépend du vent et de la météo, y compris depuis les tiles inertes voisines ([A-16](../../OPEN-PROBLEMS.md#a-16--gestion-des-corps)).
- **Décontamination** : le joueur désigne une zone, une équipe s'en charge seule. Tout le monde peut le faire, mais seules les [équipes sanitaires](../units/soutien/equipes-sanitaires.md) ne risquent pas de tomber malades. La tâche baisse le moral ; personne ne s'y habitue.
- **Maladies** : une carte de risque sanitaire par tile (cases de 50 à 100 m, mise à jour chaque minute de jeu) additionne corps non traités, eaux usées, eau non potable et surpopulation ; le froid est calculé pour chaque personne (météo, abri chauffé ou non, vêtements chauds, couvertures). Chance de tomber malade = risque du lieu × froid × faim et fatigue, tirée à la mise à jour des jauges. Pas de contagion entre personnes, pas de prévention ([A-17](../../OPEN-PROBLEMS.md#a-17--maladies)).
- **Malades** : une seule maladie générique. Soignés par des médicaments, sinon chance de mourir ; le froid ne tue que par la maladie. Un ouvrier ou un civil malade ne travaille pas ; un soldat malade est diminué.

## Traits de régiment

- Un régiment peut porter des traits qui modifient le comportement de ses soldats.
- Premier exemple : la Légion étrangère (moral élevé, résistance à la débandade ; pas d'effet sur l'usure).
- Liste des traits à définir.

## Véhicules

- Un véhicule est un contenant : chaque membre d'équipage et chaque passager est un soldat individuel.
- Les passagers peuvent tirer depuis l'intérieur lorsque le véhicule le permet (trappes).
- Un véhicule immobilisé (faute de carburant, par exemple) peut servir de position défensive.
- **Véhicules civils trouvés** : chacun a un état (batterie, carburant, clés, dégâts). Le carburant ne peut pas être siphonné d'un véhicule à l'autre.
- Un humain dans un véhicule blindé fermé est à l'abri du contact des zombies, mais pas des effets d'une arme nucléaire.
- **Réparation** : uniquement en atelier. Les véhicules en panne ou endommagés doivent être dépannés et ramenés.
- **Itinéraires** : chaque véhicule appartient à une **classe d'itinéraire** selon sa masse, son gabarit et sa motricité : véhicule léger, camion, porte-char chargé, chenillé lourd (liste révisable, [P-09](https://github.com/fafapiti/prototypes/blob/main/p09-chemin-routier/README.md)). La classe fixe les routes permises (chemins pour les camions et les chenillés, routes revêtues seulement pour les porte-chars), les ponts franchissables, les passages trop étroits ou trop bas, et la vitesse. Les itinéraires se calculent en tâche de fond ; une coupure ne fait replanifier que les véhicules dont la route passe par elle.

## Constructions

- **Barre de vie** : toutes les constructions et infrastructures (lieux compris : barrages, centrales, gares…) en ont une, ainsi que chaque bâtiment ordinaire généré depuis la BD TOPO. Les zombies, les tirs, les frappes et le souffle nucléaire la font baisser. Une infrastructure endommagée mais pas détruite continue de fonctionner ; un bâtiment endommagé garde sa capacité et sa protection. À zéro, un bâtiment s'effondre (seule une mairie peut être reconstruite, [mairies.md](../locations/mairies.md)) : ses occupants meurent, et ses **gravats** (son emprise élargie selon sa hauteur) bloquent le passage des humains et des véhicules, et la construction, tant que le génie ne les a pas dégagés ; les zombies les franchissent en s'entassant, comme un mur (coût de franchissement élevé, A-05) ([B-02](../../OPEN-PROBLEMS.md#b-02--génération-des-bâtiments)).
- Les zombies endommagent les constructions et peuvent les franchir en s'entassant.
- Le démontage d'une construction rend 25 % de ses matériaux.
- Le temps de construction ne dépend pas du nombre de bâtisseurs, mais un nombre minimum est requis.
- On ne construit ni sur des zombies, ni sur des corps, ni sur des gravats : la place doit d'abord être dégagée.
- **Aucune restriction de lieu** : le joueur peut envoyer construire ou évacuer des corps n'importe où, y compris sous le feu et au contact des zombies, à ses risques.
- Chaque obstacle a un coût de franchissement propre dans le chemin des zombies (A-05).

## Cargaison et logistique

- **Quatre chaînes** entre l'arrière et le front, cœur du jeu : munitions et matériel, sanitaire, construction, évacuation des corps ([README](../../README.md#cœur-du-jeu--les-quatre-chaînes-logistiques)).
- La cargaison est gérée au détail : chaque objet est visible individuellement.
- Chaque objet a une **masse** et un **volume**, qui déterminent ce qu'un véhicule, un navire ou un avion peut emporter.
- Chaque soldat a un **inventaire individuel limité**, que le port d'un sac à dos augmente.
- Les objets identiques sont regroupés en lots. Mécanique **emballer / déballer** : on emballe en caisses, palettes ou conteneurs pour transporter, on déballe pour distribuer ou inspecter.
- Les palettes et les conteneurs peuvent être inspectés pour voir leur contenu.
- Le joueur voit et gère la cargaison des navires (PHA notamment). Le chargement et le déchargement des navires de colons et d'importations sont manuels.

## Combat

- **Pas d'auto-résolution** : toute unité du joueur est réellement simulée, même loin de la caméra. **Seule exception** : le combat à l'intérieur d'un bâtiment, qui est auto-résolu (B-02).
- **Tirs** : modèle probabiliste partout (chance de toucher la tête selon l'arme, la distance, l'entraînement…).
- **Explosions** : effets calculés par zone.

## Appuis-feux

- Les frappes d'artillerie et aériennes sont possibles sans JTAC, avec une précision dégradée.
- Avec un JTAC, elles sont précises.

## Largages

- Les parachutistes et le matériel largués tombent dans une **zone de probabilité** autour de l'axe de largage. Le matériel peut se perdre.

## Feu

- Le feu est simulé objet par objet : chaque arbre, chaque objet peut brûler.
- La simulation doit tenir des incendies de la taille des plus grands feux français (de l'ordre de la superficie de Bordeaux).
- Les zombies en feu continuent d'avancer et propagent l'incendie, jusqu'à ce que le feu détruise leur tête et les tue.

## Météo et marées

- Un cycle météo et un cycle de marées sont simulés.
- Météo générée de façon déterministe à partir de normales climatiques ; marées calculées par harmoniques.
- Les débarquements ne sont possibles que sur les plages, et seulement en conditions favorables.
- Dans un port à marée, les navires de fort tirant d'eau n'entrent et ne sortent qu'à certaines heures ([ports](../locations/ports.md)).

## Dissuasion nucléaire

- **Vote** : toute frappe nucléaire (M51 ou ASMP-A) est soumise au **vote des joueurs** ; les délais de tir courent à partir du vote ([H-04](../../OPEN-PROBLEMS.md#h-04--vote-sur-les-actions-irréversibles)).
- **Pas de radioactivité** : ni retombées, ni zones contaminées, ni accident de centrale ([G-05](../../OPEN-PROBLEMS.md#g-05--effets-des-armes-nucléaires)).
- **Effets** : explosion aérienne, rayons tirés de Nukemap. TNO (100 kt) : boule de feu 500 m, rayonnement thermique 4 km, souffle 9 km. TNA (300 kt) : 600 m, 7 km, 13 km.
- **Effets par zone** :
  - **Boule de feu** : tout est détruit ; tous les zombies et tous les humains sont tués.
  - **Rayonnement thermique** : allume des incendies (feu simulé objet par objet), blesse (brûlures) les humains exposés et enflamme les zombies, qui continuent d'avancer jusqu'à ce que le feu détruise leur tête. Un bâtiment protège ses occupants du rayonnement.
  - **Souffle** : les bâtiments et constructions sont endommagés (barre de vie) ; les humains sont blessés ; les zombies restent intacts, sur place.
  - Un véhicule blindé fermé ne protège ni du rayonnement thermique ni du souffle.

- Les missiles ASMP-A et leurs têtes TNA (300 kt) ont été transférés sur le Charles de Gaulle. Seul un Rafale M parti du porte-avions peut les tirer.
- Les SNLE sont en patrouille, invisibles en jeu et sans besoins logistiques ; leur base à Saint-Pierre-et-Miquelon n'est qu'un élément de contexte (île non aménagée, [fiche](../units/naval/snle-triomphant.md)). Leurs missiles M51 (têtes TNO de 100 kt) ne sont utilisables que si le joueur contrôle le [CTM de Rosnay](../locations/ctm-rosnay.md), seul centre de transmissions de la Marine en état de marche. Un ordre de tir met 5 minutes à être transmis, puis le missile met 20 minutes à arriver ; un tir ordonné part même si le CTM est perdu entre-temps. Durée de vol fixe, quelle que soit la cible ; plusieurs tirs possibles en parallèle. Pour émettre, le CTM a besoin d'une garnison du génie et d'électricité : celle de sa région si elle est alimentée, sinon du gazole pour ses groupes électrogènes.

## État initial de la partie

- Tous les ports de métropole sont minés, sauf Brest, déminé avant le débarquement.
- Les stocks d'armes et de munitions existants sont présents, en quantités réduites.
- Aucun soldat n'a d'expérience, forces spéciales et Légion comprises ([A-18](../../OPEN-PROBLEMS.md#a-18--chaîne-sanitaire--blessures-et-usure)).
- **Approvisionnement** : au début, tout vient de l'outre-mer par bateau. Les ressources que la France ne peut pas produire (pétrole brut, électronique, composants étrangers dont les pièces américaines) sont fournies par des pays non spécifiés, non simulés ([G-03](../../OPEN-PROBLEMS.md#g-03--importations)). Leur livraison n'est pas simulée : elles s'ajoutent aux stocks des ports d'outre-mer environ toutes les deux semaines. Aucun quota sur le transport : le joueur charge, achemine à vitesse réelle et décharge ses navires lui-même.
- **Fin de partie** : aucune condition de victoire ni de défaite pour le moment ; le jeu tourne indéfiniment (G-07).
