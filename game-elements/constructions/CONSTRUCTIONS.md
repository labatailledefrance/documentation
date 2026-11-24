# Constructions (ébauche)

Liste préliminaire des constructions possibles. Les coûts sont qualitatifs (faible / moyen / élevé) ; les quantités seront définies plus tard. Les ressources citées sont définies dans [RESOURCES.md](../resources/RESOURCES.md).

Bâtisseurs possibles : [sapeurs](../units/infanterie/sapeur.md), engins du génie ([EBG](../units/vehicules-lourds/genie/ebg.md), [EGAME](../units/vehicules-lourds/genie/egame.md)…), [ouvriers](../units/soutien/equipes-genie-civil.md).

----

## Obstacles et défenses

| Construction | Matériaux | Bâtisseurs | Coût | Notes |
|---|---|---|---|---|
| Réseau de barbelés | Barbelés, piquets | Sapeurs, ouvriers | Faible | Ralentit sans arrêter. |
| Chevaux de frise | Acier ou bois | Sapeurs, ouvriers | Faible | Mobile, réutilisable. |
| Mur de sacs de sable | Sacs, terre ou sable | Sapeurs, ouvriers, infanterie | Faible | Protège les tireurs. |
| Bastion de gabions (type HESCO) | Gabions, terre | Sapeurs avec engin | Moyen | Mur rapide et solide. |
| Mur en béton préfabriqué | Blocs de béton | Engins de levage | Élevé | Durable ; nécessite des camions et une grue. |
| Fossé | — | EGAME, EBG, ouvriers | Moyen | Canalise les hordes ; consomme du carburant pour les engins. |
| Talus / merlon | Terre | EGAME, EBG | Moyen | Souvent combiné au fossé. |
| Barricade d'épaves | Véhicules épaves | EBG, dépanneurs | Faible | Utilise les épaves des routes. |
| Champ de mines / pièges | Mines et pièges, explosifs | Sapeurs | Moyen | Usage unique. |
| Poste de tir surélevé / tour de guet | Bois ou acier | Sapeurs, ouvriers | Moyen | Améliore la vue des tireurs. |
| Éclairage de périmètre | Projecteurs, électricité | Ouvriers (électriciens) | Moyen | Consomme de l'énergie. |

## Franchissement et circulation

| Construction | Matériaux | Bâtisseurs | Coût | Notes |
|---|---|---|---|---|
| Pont d'assaut | Travures | [SPRAT](../units/vehicules-lourds/genie/sprat.md) | Moyen | Pose et récupération rapides. Sur un pont détruit, il rouvre la route ; ailleurs, il crée une nouvelle traversée du réseau routier ([A-07](../../OPEN-PROBLEMS.md#a-07--recherche-de-chemin)). |
| Pont flottant / bac | Modules flottants | [PFM](../units/vehicules-lourds/genie/pfm.md) | Élevé | Grands cours d'eau. |
| Réparation de route ou de pont | Matériaux de construction | Ouvriers, engins | Variable | Rouvre un axe logistique. |
| Déblaiement d'itinéraire | — | EBG, EGAME, dépanneurs | Moyen | Dégage les épaves. |
| Zone de poser hélicoptère | Balisage | Sapeurs | Faible | Doit être sécurisée. |
| Piste sommaire | Terre compactée, matériaux | Engins, ouvriers | Élevé | Pour A400M et C-130. |

## Logistique et soutien

| Construction | Matériaux | Bâtisseurs | Coût | Notes |
|---|---|---|---|---|
| Dépôt de carburant | Citernes souples ou fixes | Sapeurs, ouvriers | Moyen | Cible sensible (incendie). |
| Dépôt de munitions | Abris, merlons | Sapeurs, ouvriers | Moyen | Doit être protégé et éloigné. |
| Atelier de maintenance | Hangar, outillage, pièces détachées | Ouvriers | Élevé | Seul lieu de réparation des véhicules. |
| Hôpital de campagne | Tentes ou modules, matériel médical | Personnel de santé, ouvriers | Élevé | Voir [hôpital de campagne](../units/soutien/hopital-de-campagne.md). |
| Cuisine / cantine | Matériel de cuisine, vivres | Ouvriers | Faible | Nourrit soldats et ouvriers. |
| Logements / abris | Tentes, modules, bâtiments réhabilités | Ouvriers | Moyen | Civils et ouvriers. |
| Groupe électrogène | Groupe, carburant | Ouvriers (électriciens) | Moyen | Consomme du gazole. |
| Point de contrôle | Barrières, sacs de sable | Sapeurs, gendarmerie | Faible | Filtre les entrées d'une zone. |
| Point de traitement de l'eau | Matériel de purification | Ouvriers | Moyen | Produit de l'eau potable. |
| Fosse commune | — | EGAME, EBG | Moyen | Reçoit les corps ; capacité limitée ; consomme du carburant pour les engins ([A-16](../../OPEN-PROBLEMS.md#a-16--gestion-des-corps)). |
| Site d'incinération | Bois | Équipes sanitaires, ouvriers | Faible | Brûle les corps ; consomme du bois. Ne se remplit jamais, mais sa fumée attire les zombies et le moral en souffre plus qu'avec une fosse. |

## Infrastructures à remettre en service

Sites existants à reconquérir puis à réparer plutôt qu'à construire :

- Usines (munitions, poudres, explosifs, pièces détachées).
- Raffineries et dépôts pétroliers.
- Centrales électriques : [CNPE](../locations/cnpe.md) et les 10 plus gros [barrages](../locations/barrages.md) (pas de centrales thermiques).
- Réseau d'eau potable.
- [Ports](../locations/ports.md), [bases aériennes](../locations/bases-aeriennes.md) et aéroports, [gares](../locations/gares.md).
- Voies ferrées obstruées ou coupées : dégagées ou réparées par le génie ([trains](../units/vehicules-lourds/trains.md)).
- [CTM de Rosnay](../locations/ctm-rosnay.md), seul centre de transmissions de la Marine en état de marche.
- Chaque lieu exige une garnison du génie pour être utilisable ([LOCATIONS.md](../locations/LOCATIONS.md#règles-communes)).

----

## Décisions

- Toutes les constructions et infrastructures ont une **barre de vie**, comme chaque bâtiment ordinaire de la carte. Les zombies, les tirs, les frappes et le souffle nucléaire l'endommagent ; elles doivent être entretenues ou réparées. Une infrastructure endommagée mais pas détruite continue de fonctionner (un barrage produit tant qu'il n'est pas rompu).
- Les zombies endommagent les constructions. En s'entassant, ils peuvent aussi les franchir. Sans humain, ils ne les attaquent pas : une base abandonnée reste intacte et peut être reprise.
- Une construction peut être démontée : 25 % de ses matériaux sont récupérés.
- Le **dépôt de munitions** et le **dépôt de carburant** remplacent la **mairie** pour leur ressource : chaque bâtiment puise au dépôt construit le plus proche qui la garde, dans sa commune ou une commune voisine, sans transport simulé ; à défaut, à la mairie de sa commune ([mairies.md](../locations/mairies.md)). Le joueur choisit d'en construire ou non, par exemple quand la mairie est loin. Le stock ne passe pas tout seul de la mairie au dépôt : le joueur le transporte. Une commune voisine est une commune qui touche celle du bâtiment ; un dépôt vide ou occupé par des zombies est sauté, au profit du suivant, puis de la mairie. Les autres ressources des colons (vivres, eau potable, médicaments, vêtements chauds, couvertures) restent à la mairie.
- Le temps de construction ne dépend pas du nombre de bâtisseurs, mais chaque construction exige un nombre minimum de bâtisseurs (à définir par construction).
- Un bâtiment effondré laisse des **gravats** : infranchissables pour les humains et les véhicules, ils doivent être dégagés par le génie avant de construire ou de passer. Les zombies les franchissent en s'entassant, comme un mur (coût de franchissement élevé).
- On peut construire **n'importe où**, y compris sous le feu et au contact des zombies, à ses risques. Mais on ne construit **ni sur des zombies, ni sur des corps** : la place doit d'abord être dégagée ([A-16](../../OPEN-PROBLEMS.md#a-16--gestion-des-corps)).
- Chaque obstacle a un **coût de franchissement** propre dans le chemin des zombies : faible pour des barbelés, élevé pour un mur ([A-05](../../OPEN-PROBLEMS.md#a-05--entassement-et-franchissement-dobstacles)). Valeurs à fixer à l'équilibrage.
