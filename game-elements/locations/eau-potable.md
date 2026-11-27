# Usines d'eau potable

## Identité

- Type : usines de production et de traitement d'eau potable, et grands réservoirs.
- Liste : **tirée des données** (BD TOPO), sans liste écrite à la main.

## Rôle en jeu

- Le **réseau d'eau potable** fait partie des infrastructures à remettre en service ([CONSTRUCTIONS.md](../constructions/CONSTRUCTIONS.md#infrastructures-à-remettre-en-service)). Une usine remise en service fournit de l'**Eau potable** aux communes qu'elle dessert, sans transport simulé.
- Elle consomme de l'**Électricité** et des ouvriers.
- Sans usine tenue, l'eau potable est transportée (citernes, bouteilles) comme toute ressource.

## Prise de contrôle

- Y installer une **garnison du génie** de 10, civil ou militaire (règle commune, voir [LOCATIONS.md](LOCATIONS.md#règles-communes)).

## Données

- BD TOPO (réservoirs, stations de traitement ; classes exactes à vérifier), OpenStreetMap en complément (`man_made=water_works`).
- Zone desservie : à décider (communes voisines, découpage comme les régions électriques).

## Décisions

- Usines tirées des données ; une usine remise en service fournit l'eau potable aux communes desservies.
- Garnison du génie : 10.

## Questions ouvertes

- Zone desservie par une usine : découpage de Voronoï, comme les régions électriques, ou communes voisines ?
