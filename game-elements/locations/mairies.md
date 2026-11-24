# Mairies et communes

## Identité

- Type : mairie de chaque commune, et surface de la commune.
- Liste : toutes les communes, tirées des données ; pas de liste écrite à la main.

## Rôle en jeu

- **Statut cosmétique** : le statut d'une commune n'a aucun effet sur le jeu. Il sert à voir d'un coup d'œil le territoire libéré, en vue simplifiée (carte dézoomée, [C-02](../../OPEN-PROBLEMS.md#c-02--niveaux-de-vue)), où la surface de la commune prend la couleur de son état.
- **Dépôt de la commune** : le bâtiment de la mairie est le dépôt de toutes les ressources des colons de la commune : vivres, eau potable, médicaments, vêtements chauds, couvertures et munitions. Chaque bâtiment de la commune y puise directement (consommation des colons, combat des occupants), sans transport simulé ([A-15](../../OPEN-PROBLEMS.md#a-15--civils-en-métropole--colonies), [B-02](../../OPEN-PROBLEMS.md#b-02--génération-des-bâtiments)). Il n'y a pas d'objet « colonie » : l'unité d'approvisionnement est la commune. C'est le seul effet de la mairie sur le jeu ; le statut de la commune, lui, reste cosmétique.
- **Qui y puise** : les colons, et aussi les soldats et les ouvriers logés dans un bâtiment de la commune, munitions comprises. Une unité en opération hors d'un bâtiment n'est ravitaillée que par sa chaîne logistique.
- **Dépôt par défaut** : la mairie garde ce qu'aucun dépôt construit ne garde. Chaque bâtiment puise chaque ressource au **dépôt construit le plus proche** qui la garde ([dépôt de munitions, dépôt de carburant](../constructions/CONSTRUCTIONS.md#logistique-et-soutien)), dans sa commune ou une commune voisine ; à défaut, à la mairie de sa commune. Les vivres, l'eau potable, les médicaments, les vêtements chauds et les couvertures restent à la mairie. Le stock ne change pas de dépôt tout seul : le joueur le transporte, comme toute cargaison. Une commune voisine est une commune qui **touche** celle du bâtiment (graphe d'adjacence précalculé par la chaîne de données). Un dépôt vide ou occupé par des zombies est sauté : le bâtiment puise au suivant qui a du stock et n'est pas occupé, puis à la mairie. Le joueur construit un dépôt s'il le juge utile, par exemple quand la mairie est loin.
- **Capacité** : illimitée ; la masse et le volume ne comptent que pendant le transport.
- **Mairie détruite** : elle ne sert plus de dépôt tant que le génie ne l'a pas réparée. Effondrée, elle est **reconstruite à l'identique**, avec les règles d'une construction (matériaux, nombre minimum de bâtisseurs, durée fixe) : c'est la seule exception à l'effondrement définitif des bâtiments ([B-02](../../OPEN-PROBLEMS.md#b-02--génération-des-bâtiments)).
- **Mairie tombée aux zombies** : son stock reste sur place, à reprendre ; tant que des zombies l'occupent, aucun bâtiment n'y puise.
- **Une mairie par commune** : la mairie principale (pas les mairies annexes des communes nouvelles). À Paris, Lyon et Marseille, chaque **arrondissement** compte comme une commune, pour le dépôt comme pour le statut, avec sa mairie d'arrondissement.
- **Adjacence** : la chaîne de données précalcule, pour chaque commune, la liste des communes qui la touchent.
- **Commune sans mairie dans les données** : la chaîne de données désigne le **plus grand bâtiment** de la commune, public ou privé.
- On libère la **commune**, pas la mairie : la mairie sert de point d'ancrage, la commune est libérée quand **au moins 10 colons habitent sa mairie** et qu'**aucun zombie** ne se trouve sur sa surface.

## États d'une commune

L'état est une simple fonction de la situation, recalculée à chaque changement ; il n'y a pas d'historique à stocker.

| Couleur | État | Condition |
|---|---|---|
| Rouge | Zombie | Moins de 10 colons habitent la mairie (état initial de toute commune de métropole). |
| Jaune | Contestée | Au moins 10 colons dans la mairie, et au moins un zombie sur la surface. |
| Bleu | Libérée | Au moins 10 colons dans la mairie, et aucun zombie sur la surface. |

- Un zombie qui arrive par migration entre tiles inertes compte ([A-19](../../OPEN-PROBLEMS.md#a-19--suivi-des-zombies-par-commune)).
- Des soldats seuls ne suffisent pas : une commune nettoyée sans colons dans la mairie reste rouge.
- Les communes d'outre-mer sont bleues dès le début (aucun zombie).

## Données

- Limites des communes : IGN (ADMIN EXPRESS ou BD TOPO) ; arrondissements municipaux pour Paris, Lyon et Marseille.
- Position de la mairie : BD TOPO, zone « Mairie » sans détail ou « Hôtel de ville » (les mairies de communes déléguées sont écartées) ; bâtiment de la mairie : le plus grand dont le centroïde est dans la zone, sinon le plus proche à moins de 100 m. Dans la Manche, 444 communes sur 445 ont leur mairie dans la BD TOPO ([P-07](../../../prototypes/p07-chaine-manche/README.md#9-lieux-dintérêt-de-locations-dans-la-manche-resultats-lieuxcsv)). À Paris, Lyon et Marseille, la mairie d'arrondissement porte le code de la commune : elle est rattachée à son arrondissement par sa position. Sert aussi au découpage des régions électriques ([cnpe.md](cnpe.md)).

## Décisions

- Le statut d'une commune est cosmétique ; il n'influence pas le gameplay.
- La mairie est le dépôt de toutes les ressources des colons de sa commune (vivres, eau potable, médicaments, vêtements chauds, couvertures, munitions) ; les bâtiments de la commune y puisent sans transport simulé.
- Pas d'objet « colonie » : l'unité d'approvisionnement est la commune.
- Une mairie détruite doit être réparée pour servir de dépôt ; effondrée, elle est reconstruite à l'identique.
- Une mairie tombée aux zombies garde son stock sur place ; personne n'y puise tant qu'elle n'est pas reprise.
- Capacité du dépôt illimitée.
- Soldats et ouvriers logés dans la commune puisent aussi au dépôt, munitions comprises ; une unité en opération passe par sa chaîne logistique.
- Une mairie par commune, la principale ; à Paris, Lyon et Marseille, chaque arrondissement compte comme une commune (dépôt et statut).
- Sans mairie dans les données, le plus grand bâtiment de la commune, public ou privé, en tient lieu.
- La mairie est le dépôt par défaut : pour chaque ressource, un bâtiment puise au dépôt construit le plus proche qui la garde (dépôt de munitions, dépôt de carburant), dans sa commune ou une voisine, sinon à la mairie. Le joueur transporte lui-même le stock d'un dépôt à l'autre.
- Commune voisine : une commune qui touche celle du bâtiment. Un dépôt vide ou occupé par des zombies est sauté.
- Vivres, eau potable, médicaments, vêtements chauds et couvertures : toujours à la mairie.
- Une mairie effondrée se reconstruit avec les règles d'une construction.
- On libère la commune : elle est libérée si au moins 10 colons habitent sa mairie et qu'aucun zombie ne se trouve sur sa surface.
- Couleurs en vue dézoomée : rouge (zombie), jaune (contestée), bleu (libérée).
- État = fonction de la situation : moins de 10 colons dans la mairie → rouge ; sinon un zombie sur la surface → jaune ; sinon bleu.
- Un zombie arrivé par migration entre tiles inertes compte.
- Paris, Lyon, Marseille : chaque mairie d'arrondissement est rattachée à son arrondissement par sa position (dans son polygone), son code INSEE étant celui de la commune (P-07).

## Questions ouvertes

Voir [P-05](../../../prototypes/p05-civils/README.md#questions-ouvertes).
