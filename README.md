# La Bataille de France

**La Bataille de France** : jeu vidéo open source de reconquête de la France face aux zombies, en 2030. Le projet est en phase de **conception** : ce dépôt en contient la documentation.

## Sommaire

- [Problèmes ouverts](OPEN-PROBLEMS.md) : problèmes techniques et de conception, avec leurs décisions.
- [Unités](game-elements/units/UNITS.md), [ressources](game-elements/resources/RESOURCES.md), [systèmes](game-elements/systemes/SYSTEMES.md), [zombies](game-elements/zombies/ZOMBIES.md), [constructions](game-elements/constructions/CONSTRUCTIONS.md), [lieux](game-elements/locations/LOCATIONS.md).
- Prototypes techniques : dépôt [labatailledefrance/prototypes](https://github.com/labatailledefrance/prototypes).
- [Développement](developpement/DEVELOPPEMENT.md) : dépôts du code, standard, pipelines, déploiement, contribution.

----

## Scénario


Le jeu se déroule en France, en 2030. L'humanité est en train de perdre, mais elle peut encore gagner : non pas grâce à des super-armes ou à des batailles épiques, mais par une planification rigoureuse, une logistique solide et de lourds sacrifices.

La métropole est entièrement perdue : sa population a été zombifiée, à l'exception de quelques rares poches isolées de survivants. L'outre-mer a survécu intact. Avant l'effondrement, une évacuation par la mer y a conduit environ 7 millions de personnes : environ 10 millions d'humains y vivent désormais.

Une force d'invasion a été rassemblée. Elle doit débarquer en Normandie avec pour objectif de reprendre Brest et de reprendre le contrôle de la FOST et de sa principale base d'opérations, puis, à partir de cette tête de pont, de reconquérir le territoire national. Le jeu commence au débarquement.

Des navires chargés de civils attendent au large de Brest. Le joueur les débarque et les installe où il veut sur le territoire reconquis, puis par tout port repris. Ces **colons** sont la main-d'œuvre de la reconquête : il faut leur reprendre des villes et des villages, les fortifier, les protéger et les approvisionner. Une colonie qui tombe est perdue, et ses habitants rejoignent les zombies.

----

## Gameplay

### Échelle :

- Le monde est à l'échelle 1:1 : distances, superficies, bâtiments et terrain correspondent à la réalité.
- Seules la France métropolitaine et les outre-mer sont simulées (sans les Terres australes et antarctiques françaises ni Clipperton, inhabitables, ni les collectivités du Pacifique : Nouvelle-Calédonie, Polynésie française, Wallis-et-Futuna). Le reste du globe est affiché, grisé ; les navires naviguent néanmoins sur toutes les mers, à vitesse réelle.
- Aucun écran de chargement en jeu : le joueur peut zoomer pour suivre l'action d'un soldat individuel, et dézoomer jusqu'à l'espace. En quelques mouvements de souris, le joueur doit pouvoir déplacer la caméra de Paris à La Réunion sans accroc.
- Contrôle direct : le joueur doit pouvoir contrôler certaines unités individuelles (chars, soldats, etc.), à l'exception des unités complexes (avions, etc.) qui sont contrôlées automatiquement par le jeu.
- Simulation en temps réel : le jeu n'est pas au tour par tour. Tout se passe en temps réel, **sans accélération** : le temps de jeu est le temps réel ([A-02](OPEN-PROBLEMS.md#a-02--accélération-du-temps)).
- L'intégralité de l'arsenal français est disponible : la majeure partie de l'armée est partie pour l'outre-mer. Tout l'arsenal français reste disponible, mais pas dans les quantités d'avant la guerre contre les zombies.

### Multijoueur

- Le jeu est **multijoueur seulement** et se joue dans le **navigateur**. Il n'y a qu'**une partie officielle**, sur un serveur unique ; le code étant libre, d'autres peuvent héberger la leur.
- **Pas de compte**, pour le moment : n'importe qui peut rejoindre la partie, anonymement, et l'influencer.
- **Tout le monde commande tout** : toutes les unités, tous les lieux et tous les stocks sont communs ; le dernier ordre l'emporte. Dans la documentation, « le joueur » désigne n'importe quel joueur.
- Les actions **irréversibles** (frappe nucléaire, destruction d'un barrage) sont soumises au **vote des joueurs**.
- Le monde est **persistant** : il continue de tourner quand personne n'est connecté. L'administrateur du serveur peut le remettre à zéro (nouvelle saison) ; chaque mise à jour du serveur le remet à zéro.
- Détails : [OPEN-PROBLEMS.md, section H](OPEN-PROBLEMS.md#h-multijoueur-et-serveur).

### Boucle de reconquête

Elle n'est pas scriptée : elle émerge des règles de base.

- Pour prendre une ville, le joueur y construit une base avancée. Il lui faut quatre chaînes logistiques entre l'arrière et le front (voir ci-dessous).
- Ces besoins le poussent à rapprocher de la zone de combat les bases arrière qui les alimentent.
- Chaque zombie abattu laisse un corps : au pied des défenses, les corps forment des rampes ; ailleurs, ils rendent malade. Il faut les évacuer et les brûler, mais la fumée de l'incinérateur attire les zombies. Plus il tourne, plus ils arrivent, et plus le combat s'intensifie.
- Le joueur doit donc sans cesse réévaluer ses positions et les améliorer, en jonglant entre les besoins du combat.
- Il ne peut pas s'y soustraire : c'est ce combat qui libère la surface habitable nécessaire pour remettre en marche les usines qui alimentent la bataille.

### Cœur du jeu : les quatre chaînes logistiques

Le cœur de la complexité du jeu est la logistique entre la base arrière et la zone de combat. Elle suit quatre axes, qui tous doivent fonctionner en même temps :

1. **Munitions et matériel** : apporter munitions et matériel aux combattants, remorquer les véhicules endommagés, remplacer au front les combattants et les véhicules perdus.
2. **Sanitaire** : évacuer les blessés (évasan), relever les combattants dont le moral est bas, les soigner et les renvoyer au combat ([A-18](OPEN-PROBLEMS.md#a-18--chaîne-sanitaire--blessures-et-usure)).
3. **Construction** : réparer et construire les structures en zone de combat.
4. **Évacuation des corps** : évacuer le volume considérable de cadavres vers les fosses communes ou les incinérateurs ([A-16](OPEN-PROBLEMS.md#a-16--gestion-des-corps)).

Toute règle nouvelle devrait nourrir au moins l'un de ces axes ; une unité ou une construction qui n'en nourrit aucun est suspecte.

----

Exigences identifiées :

- Code Rust portable
- Jeu en navigateur : client en WebAssembly, rendu en WebGPU seul
- Multijoueur seulement : une partie officielle, sur un serveur unique qui fait autorité (pile native Kubernetes)
- Sans compte (pour le moment)
- Jeu 100 % open source : code sous AGPL-3.0-or-later, documentation sous CC-BY-SA-4.0, contributions sous DCO ; assets tiers en CC0-1.0, CC-BY-4.0 ou OFL-1.1 ([F-05](OPEN-PROBLEMS.md#f-05--licence))
- Données géographiques issues de l'IGN (BD TOPO, RGE ALTI ; Licence Ouverte Etalab), OpenStreetMap en complément (licence ODbL : attribution obligatoire)

Choix de conception :

- Vue isométrique
- Direction artistique low-poly, dans l'esprit de Project Zomboid, avec des textures peu détaillées ([E-01](OPEN-PROBLEMS.md#e-01--volume-de-contenu))
- API/ABI pour le modding avancé, côté serveur seulement ([F-03](OPEN-PROBLEMS.md#f-03--modding))
- Facilité de traduction
