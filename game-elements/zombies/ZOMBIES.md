# Zombies

Comportement des zombies, tel que décidé jusqu'ici. Document à compléter.

----

## Infection

- **Exposition** : tout contact entre un zombie et un humain.
- **10 secondes** séparent l'exposition de la zombification.
- Un soldat ou un civil exposé est donc perdu : aucun soin n'est possible.
- Un humain à l'intérieur d'un véhicule blindé fermé est à l'abri de tout contact (mais pas des effets d'une arme nucléaire).
- Un humain mort **sans contact** avec un zombie (blessure, accident, maladie) **ne se relève pas** ([A-18](../../OPEN-PROBLEMS.md#a-18--chaîne-sanitaire--blessures-et-usure)).

## Perception

- Chaque zombie a un **rayon de détection** (sons).
- Chaque zombie a un **angle de vue** et une **distance de vue maximale**. Elle est d'environ **400 m en ville** et va jusqu'à **800 m en terrain découvert** ; elle est **fixée par tile**, selon la part de bâti de la tile, calculée à la préparation de la carte ; elle reste toujours inférieure à la largeur de la couronne de tiles actives, environ 1 km ([A-01](../../OPEN-PROBLEMS.md#a-01--nombre-de-zombies--simulation-par-tiles)). En ville, les bâtiments coupent la plupart des vues bien avant. Le coût de la recherche ne dépend plus de la distance (P-02) ; le surcoût vient des zombies qui ne dorment plus et de ceux qui courent plus tôt.
- Le bruit attire les zombies : par exemple, les tirs d'une batterie d'artillerie attirent les hordes vers elle.
- Un humain dans un bâtiment est **perçu** : vu par les fenêtres, entendu quand il tire. Pour les zombies, un bâtiment occupé est **un seul point, à sa porte**, qui porte le nombre de ses tireurs ; ses occupants n'y sont pas des points séparés ([A-15](../../OPEN-PROBLEMS.md#a-15--civils-en-métropole--colonies), [B-02](../../OPEN-PROBLEMS.md#b-02--génération-des-bâtiments)).
- Les **tirs par les fenêtres** font du bruit comme des tirs dehors, **avec la même portée** (le bâtiment ne l'atténue pas) : ils alimentent le champ « bruits » et attirent les zombies.
- La **fumée** de l'incinération des corps attire aussi les zombies, d'autant plus loin que le vent et la météo la portent ; elle attire aussi ceux des tiles inertes voisines ([A-16](../../OPEN-PROBLEMS.md#a-16--gestion-des-corps)).
- Les drones n'attirent pas les zombies.
- Un zombie perd la trace d'une cible dès qu'elle sort de ses zones de détection (ouïe et vue), à la fréquence de perception près. Il ne s'arrête pas pour autant : voir *Dernière position connue*.
- **Propagation de l'alerte** : un zombie qui poursuit une cible la transmet à ses voisins proches, qui la transmettent à leur tour. L'alerte traverse ainsi une foule de proche en proche, comme une onde. L'alerte passe d'un zombie à ses voisins **à chaque tick** (environ 20 fois par seconde) : une vague porte ainsi jusqu'à environ 150 m d'une source pendant la fraîcheur de l'alerte (P-02 : à la fréquence de perception, elle ne porterait plus qu'à environ 36 m).
- Un zombie qui suit une alerte transmise ne perçoit pas lui-même la cible. Il la perd **dès que la source la perd** : quand plus aucun zombie ne la perçoit directement. Une alerte relayée reste valable **3 secondes** après la dernière observation directe.
- **Fréquence de perception** : environ 4 fois par seconde pour un zombie actif (en poursuite, attiré, suivant une alerte). Un zombie perd donc une cible jusqu'à un quart de seconde après sa sortie du champ.
- **Mise en veille** : un zombie sans stimulus (errance) est simulé à fréquence réduite, avec les mêmes règles : il perçoit environ une fois par seconde et se déplace par pas plus longs. Il se réveille dès qu'un stimulus apparaît ([A-03](../../OPEN-PROBLEMS.md#a-03--unités-dispersées-sur-tout-le-territoire)).
- **Veille profonde** : un zombie plus loin de tout humain que sa distance de vue ne se déplace plus qu'une fois par seconde, sans collision avec les autres zombies. Il repasse en veille normale dès qu'un humain approche ou que la caméra le regarde.
- **Dernière position connue** : après avoir perdu une trace, qu'il ait perçu la cible lui-même ou suivi une alerte, un zombie continue vers la dernière position connue de sa cible pendant **environ une minute**, puis reprend son errance. Un nouveau stimulus l'en détourne.

## Déplacement

- Les zombies **courent** et donnent toujours tout ce qu'ils ont.
- Ils **entrent dans les bâtiments** qui contiennent des humains, et seulement ceux-là, par leur porte ; ils en sortent aussitôt qu'il n'y en a plus. Un bâtiment est un contenant : l'intérieur n'est pas simulé, et un combat à l'intérieur est auto-résolu chaque seconde ; un humain exposé devient zombie au bout de 10 s, comme dehors ([B-02](../../OPEN-PROBLEMS.md#b-02--génération-des-bâtiments)).
- **Aucun épuisement** : ils sont en permanence à 100 % de leurs capacités. L'usure du corps (pieds usés à force de courir, etc.) n'est pas simulée.
- Ils franchissent les cours d'eau **en marchant au fond** : l'eau ne constitue pas une protection.
- Sous l'eau, ils ne peuvent pas attaquer un bateau ni un véhicule qui traverse un gué.
- Ils se marchent dessus : en s'entassant, ils franchissent les obstacles (murs, fossés, barbelés). Les corps des zombies abattus au pied d'un obstacle servent de rampe aux suivants, tant qu'ils ne sont pas évacués. Le nombre de zombies par case est exact partout : une case pleine n'accepte plus personne (réservation des places, [A-05](../../OPEN-PROBLEMS.md#a-05--entassement-et-franchissement-dobstacles)).
- **Chemin** : un zombie qui a une cible descend un **flow field**, une carte des distances à pied calculée pour toute la tile, à cases de 2 m ([P-03](https://github.com/labatailledefrance/prototypes/blob/main/p03-flow-fields/README.md)). Il contourne les îlots et longe les rues sans raser les façades. Sous 15 m de sa cible, il va droit sur elle. Son pas mêle la pente du champ et un **écart individuel**, pour que la foule s'étale au lieu d'avancer en colonne (P-24, révisable). Les champs ne couvrent que les tiles actives et leur couronne.
- **Limite de la simulation** : un zombie sans cible (errance) ne quitte pas la zone simulée vers une tile inerte ; il fait demi-tour. Un zombie qui poursuit une cible la suit partout ([A-01](../../OPEN-PROBLEMS.md#a-01--nombre-de-zombies--simulation-par-tiles), [P-04](https://github.com/labatailledefrance/prototypes/blob/main/p04-activation/README.md)).
- **Obstacles** : dans le calcul du chemin, chaque obstacle a un coût de franchissement qui lui est propre (barbelés faible, mur et gravats d'un bâtiment effondré élevés). La foule contourne un obstacle quand le détour est court, et l'attaque en s'entassant quand il est long.

## Mort

- Un zombie ne meurt que d'une destruction de la **tête** (tir, explosion, feu…).
- Un zombie abattu laisse un **corps**, qu'il faut évacuer et traiter ([A-16](../../OPEN-PROBLEMS.md#a-16--gestion-des-corps)). Un corps n'est pas contagieux : il ne transmet pas la zombification, seulement des maladies (A-17).
- L'écrasement (par un véhicule, par exemple) ne compte pas comme une destruction de la tête : un zombie écrasé reste actif.
- Les zombies ne peuvent pas arrêter un [train](../units/vehicules-lourds/trains.md), même en s'entassant sur la voie, ni monter dans ses wagons.

## Dégâts aux structures

- Les zombies endommagent les structures et les constructions (coups, masse), mais **seulement en présence d'humains** : sans humain, ils ne les attaquent pas ([A-01](../../OPEN-PROBLEMS.md#a-01--nombre-de-zombies--simulation-par-tiles)).

## Feu

- **Arme nucléaire** : la boule de feu tue tous les zombies ; le rayonnement thermique les enflamme ; le souffle les laisse intacts, sur place ([G-05](../../OPEN-PROBLEMS.md#g-05--effets-des-armes-nucléaires)).
- Un zombie en feu continue d'avancer et propage l'incendie, jusqu'à ce que le feu détruise sa tête et le tue.

## Comportement de horde

- **Hordes émergentes** : une horde n'est pas un objet du jeu. C'est simplement un ensemble de zombies voisins qui se transmettent l'alerte (voir *Perception*). Pas de meneur, pas de périmètre à calculer, rien à maintenir quand une horde se divise ou fusionne.
- Les zombies d'une horde suivent le flow field de leur cible ([A-07](../../OPEN-PROBLEMS.md#a-07--recherche-de-chemin)). Chaque tile active (et sa couronne) a deux champs, partagés par tous ses zombies :
  - **proies** : vers les humains observés par au moins un zombie depuis moins d'une minute, à la position où ils ont été vus. C'est la règle de la dernière position connue : la horde suit ce qu'elle sait, pas la vérité. Un zombie va vers l'humain observé le plus proche à pied. Recalculé une fois par seconde ;
  - **bruits** : vers les bruits (tirs, y compris par les fenêtres d'un bâtiment) et les fumées ; recalculé quand ils changent.

## Hors simulation

- Les animaux ne sont pas simulés.
