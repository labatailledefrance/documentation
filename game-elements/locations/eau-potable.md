# Usines d'eau potable

## Identité

- Type : usines de production et de traitement d'eau potable, et grands réservoirs.
- Liste : **tirée des données** (BD TOPO), sans liste écrite à la main.

## Rôle en jeu

- Le **réseau d'eau potable** fait partie des infrastructures à remettre en service ([CONSTRUCTIONS.md](../constructions/CONSTRUCTIONS.md#infrastructures-à-remettre-en-service)). Une usine remise en service **couvre directement** le besoin en **Eau potable** des communes qu'elle dessert : elles n'ont plus besoin de livraison d'eau, et rien ne passe par le stock de la [mairie](mairies.md).
- Elle consomme de l'**Électricité** et des ouvriers. Dans une région éteinte, elle peut tourner sur **groupes électrogènes**, alimentés en **Gazole routier**, comme le [CTM de Rosnay](ctm-rosnay.md).
- **Zone desservie** : découpage de Voronoï **fixe**, comme les régions électriques (chaque commune rattachée à l'usine la plus proche de sa mairie).
- L'**outre-mer** a l'eau potable d'office, comme l'électricité.
- Sans usine tenue, l'eau potable est transportée (citernes, bouteilles) comme toute ressource.

## Prise de contrôle

- Y installer une **garnison du génie** de 10, civil ou militaire (règle commune, voir [LOCATIONS.md](LOCATIONS.md#règles-communes)).

## Données

- BD TOPO (réservoirs, stations de traitement ; classes exactes à vérifier), OpenStreetMap en complément (`man_made=water_works`).

## Décisions

- Usines tirées des données ; une usine remise en service fournit l'eau potable aux communes desservies.
- Garnison du génie : 10.
- L'usine couvre directement le besoin des communes desservies, sans passer par la mairie (2026-11-27, recommandation adoptée par défaut, révisable).
- Zone desservie : Voronoï fixe, comme les régions électriques (2026-11-27, recommandation adoptée par défaut, révisable).
- Outre-mer : eau potable d'office (2026-11-27, recommandation adoptée par défaut, révisable).

## Questions ouvertes

- Aucune pour l'instant.
