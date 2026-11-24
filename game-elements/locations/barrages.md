# Barrages hydroélectriques

## Identité

- Type : barrages et centrales hydroélectriques de métropole.
- Liste : les **10 plus gros barrages** de France, classés par **volume de la retenue**. Classement à vérifier :

| Barrage | Commune (département) |
|---|---|
| Serre-Ponçon | Rousset (Hautes-Alpes) |
| Sainte-Croix | Sainte-Croix-du-Verdon (Alpes-de-Haute-Provence) |
| Vouglans | Cernon (Jura) |
| Bort-les-Orgues | Bort-les-Orgues (Corrèze) |
| Mont-Cenis | Val-Cenis (Savoie) |
| Sarrans | Sainte-Geneviève-sur-Argence (Aveyron) |
| Monteynard | Monteynard (Isère) |
| Tignes (lac du Chevril) | Tignes (Savoie) |
| Roselend | Beaufort (Savoie) |
| Pareloup | Salles-Curan (Aveyron) |

## Rôle en jeu

- Un barrage remis en service restaure l'**Électricité** dans sa région, comme une [CNPE](cnpe.md).
- Les barrages entrent dans le découpage de Voronoï fixe des régions électriques, avec les CNPE.
- **Destruction** : un barrage a une **barre de vie** ; il faut plusieurs frappes de **bombes guidées** ou d'**artillerie** pour la vider et le rompre. Sa rupture **noie la vallée en aval** : les zombies sont dispersés, les humains, bâtiments, ponts et véhicules sont détruits. Un barrage détruit ne peut pas être reconstruit : sa région reste **dans le noir pour toujours** ([A-20](../../OPEN-PROBLEMS.md#a-20--rupture-de-barrage)).

## Prise de contrôle

- Y installer une **garnison du génie** de 10, civil ou militaire, pour le redémarrage (règle commune, voir [LOCATIONS.md](LOCATIONS.md#règles-communes)).

## Remise en service

- Comme une CNPE : apporter du **Gazole routier** pour les groupes électrogènes de secours, puis le génie fait le reste. Le barrage fonctionne ensuite indéfiniment, sans approvisionnement ni garnison.

## Données

- Barrages : BD TOPO, d'après une **liste écrite à la main** (identifiants BD TOPO) : la nature « Barrage » couvre aussi les seuils et petits ouvrages (212 dans la Manche, P-07) ; OpenStreetMap en complément.
- Retenue (volume d'eau) et relief en aval (RGE ALTI) : nécessaires pour calculer l'onde de rupture.

## Décisions

- Les barrages entrent dans le Voronoï des régions électriques.
- Seuls les 10 plus gros barrages de France, par volume de la retenue, sont dans le jeu.
- Remise en service : même règle que les CNPE.
- Un barrage a une barre de vie, comme toute construction ou infrastructure ; endommagé mais pas rompu, il continue de produire. Plusieurs frappes de bombes guidées ou d'artillerie sont nécessaires pour le rompre ; sa rupture noie la vallée en aval.
- Un barrage détruit ne peut pas être reconstruit ; sa région reste dans le noir pour toujours.
- Tout ordre de frappe visant un barrage est soumis au **vote des joueurs** ([H-04](../../OPEN-PROBLEMS.md#h-04--vote-sur-les-actions-irréversibles)).
- La liste des barrages est écrite à la main (identifiants BD TOPO), la BD TOPO classant aussi les seuils en « Barrage » (P-07).

## Questions ouvertes

- Aucune pour l'instant.
