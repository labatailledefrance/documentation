# Centrales nucléaires (CNPE)

## Identité

- Type : centre nucléaire de production d'électricité (CNPE), exploité par EDF.
- Liste : les centrales en service en 2030 (Fessenheim est fermée depuis 2020).

| CNPE | Département |
|---|---|
| Belleville | Cher |
| Blayais | Gironde |
| Bugey | Ain |
| Cattenom | Moselle |
| Chinon | Indre-et-Loire |
| Chooz | Ardennes |
| Civaux | Vienne |
| Cruas | Ardèche |
| Dampierre | Loiret |
| Flamanville | Manche |
| Golfech | Tarn-et-Garonne |
| Gravelines | Nord |
| Nogent | Aube |
| Paluel | Seine-Maritime |
| Penly | Seine-Maritime |
| Saint-Alban | Isère |
| Saint-Laurent | Loir-et-Cher |
| Tricastin | Drôme |

Liste vérifiée pour 2030 (2026-11-27) : aucune fermeture n'est prévue avant 2030, l'État ayant renoncé aux fermetures de la PPE 2 et prolongé le parc ; Flamanville 3 (EPR) est en service depuis décembre 2024 (57 réacteurs). Les EPR2 (Penly, Gravelines) ne produiront pas avant 2030.

## Rôle en jeu

- Une centrale redémarrée restaure l'**Électricité** dans sa région : usines, hôpitaux, recharge des **Batteries et piles** ([RESOURCES.md](../resources/RESOURCES.md#énergie)), traction des trains électriques ([gares.md](gares.md)), [CTM de Rosnay](ctm-rosnay.md).
- **Région** : découpage de Voronoï **fixe** autour de toutes les centrales électriques du jeu (CNPE et [barrages](barrages.md)), calculé une fois à la préparation de la carte. Chaque commune est rattachée à la centrale la plus proche de sa **mairie** ; les régions suivent donc les limites de communes. Une région dont la centrale est éteinte reste sans électricité, même si la voisine tourne : c'est voulu, pour inciter à libérer toutes les centrales.
- La région est alimentée **d'un bloc** : le réseau (lignes, postes) n'est pas simulé.
- Toutes les communes de la région reçoivent l'électricité, qu'elles soient libérées ou non.
- Les centrales thermiques ne sont pas dans le jeu : les approvisionner serait trop lourd à gérer.

## Prise de contrôle

- Y installer une **garnison du génie** de 100, civil ou militaire, pour le redémarrage (règle commune, voir [LOCATIONS.md](LOCATIONS.md#règles-communes)). Elle peut repartir ensuite.

## Remise en service

1. Apporter du **Gazole routier** pour les groupes électrogènes de secours de la centrale.
2. Le génie s'occupe du reste : [équipes de génie civil](../units/soutien/equipes-genie-civil.md) ou génie militaire ([sapeurs](../units/infanterie/sapeur.md)), qui ont les mêmes compétences de génie.
3. Une fois redémarrée, la centrale fonctionne **indéfiniment**, sans approvisionnement régulier ni garnison.

## Données

- Emprise des sites : BD TOPO, d'après une **liste écrite à la main** (identifiants BD TOPO) : la nature « Centrale électrique » couvre aussi de petits ouvrages (P-07).
- Régions : calculées à la préparation de la carte, à partir des mairies et des limites de communes (IGN).

## Décisions

- Redémarrer une CNPE restaure l'électricité dans sa région.
- Pour la redémarrer, il suffit d'apporter du gazole pour les groupes électrogènes de secours ; le génie, civil ou militaire, fait le reste.
- Régions électriques : Voronoï autour des CNPE et des barrages, chaque commune rattachée à la centrale la plus proche de sa mairie.
- Une centrale redémarrée ne consomme rien.
- Toutes les communes d'une région alimentée reçoivent l'électricité, libérées ou non.
- Pas de centrales thermiques dans le jeu.
- Garnison du génie : 100, pour le redémarrage seulement.
- Voronoï fixe, pour inciter à libérer toutes les centrales ; région alimentée d'un bloc.
- Outre-mer : électricité disponible d'office.
- Pas d'accident nucléaire : la radioactivité n'est pas simulée dans le jeu ([G-05](../../OPEN-PROBLEMS.md#g-05--effets-des-armes-nucléaires)).
- La liste des CNPE est écrite à la main (identifiants BD TOPO), la BD TOPO ne distinguant pas les CNPE des petites centrales (P-07).

## Questions ouvertes

- Aucune pour l'instant.
