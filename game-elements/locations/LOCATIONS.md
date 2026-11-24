# Lieux

Index des lieux d'intérêt de la carte. Ce sont des sites réels, tirés des données (IGN en priorité, OpenStreetMap en complément), qui donnent au joueur une capacité nouvelle une fois tenus par une garnison : frappe nucléaire, électricité, logistique aérienne, ferroviaire ou navale.

----

## Liste

| Lieu | Fiche | Ce que le lieu apporte | Garnison du génie |
|---|---|---|---|
| CTM de Rosnay | [ctm-rosnay.md](ctm-rosnay.md) | Ordres de tir des M51 des SNLE. | 10 |
| Centrales nucléaires (CNPE) | [cnpe.md](cnpe.md) | Électricité dans leur région, une fois redémarrées. | 100, pour le redémarrage seulement |
| Barrages hydroélectriques | [barrages.md](barrages.md) | Électricité dans leur région ; destruction possible pour noyer la vallée. | 10, pour le redémarrage seulement |
| Bases aériennes et aéroports | [bases-aeriennes.md](bases-aeriennes.md) | Stationnement d'aéronefs et logistique aérienne, selon la longueur de piste. | 100 ; aucune pour une piste sommaire |
| Gares | [gares.md](gares.md) | Logistique ferroviaire de gare à gare. | 10 |
| Ports | [ports.md](ports.md) | Logistique navale et débarquement des colons, selon le tirant d'eau, les quais et la marée. | 100 (commerce, militaire) ; 10 (pêche, plaisance) |
| Mairies et communes | [mairies.md](mairies.md) | Affichage du territoire libéré (cosmétique) ; dépôt des ressources des colons de la commune. | — |
| Poches de survivants | [survivants.md](survivants.md) | Civils secourus. | — |

## Structure d'une fiche de lieu

Identité, Rôle en jeu, Prise de contrôle, Remise en service (s'il y a lieu), Données, Décisions, Questions ouvertes.

## Règles communes

- Les lieux sont **connus dès le début** de la partie et visibles sur la carte, y compris en vue simplifiée ([C-02](../../OPEN-PROBLEMS.md#c-02--niveaux-de-vue)).
- **Garnison du génie** : un lieu n'est utilisable que si une garnison du génie, d'un effectif minimal propre au type de lieu (tableau ci-dessus), s'y trouve. La garnison suffit : des zombies dans l'emprise n'empêchent pas l'usage. Dès que la garnison passe sous l'effectif minimal (pertes, blessés), le lieu est inutilisable. Le génie militaire ([sapeurs](../units/infanterie/sapeur.md)) et les [équipes de génie civil](../units/soutien/equipes-genie-civil.md) ont les mêmes compétences de génie ; le premier a en plus des compétences militaires. Les deux comptent pour la garnison.
- **Communes** : un lieu appartient à **toutes** les communes qu'il touche (Roissy, une grande base, un port).
- **Statut des communes** : la couleur d'une commune (rouge, jaune, bleu) est **purement cosmétique**. Elle n'a aucun effet sur le jeu, et en particulier pas sur l'usage des lieux ([mairies.md](mairies.md)).
- Chaque lieu a une **barre de vie**, comme toute construction ; endommagé mais pas détruit, il continue de fonctionner.
- Un lieu tombé aux mains des zombies reste intact : sans humain, les zombies n'attaquent pas les constructions ([SYSTEMES.md](../systemes/SYSTEMES.md#simulation-par-tiles)). Il peut être repris.
- Les infrastructures à remettre en service sont listées dans [CONSTRUCTIONS.md](../constructions/CONSTRUCTIONS.md#infrastructures-à-remettre-en-service).

## Décisions

- Un lieu est utilisable quand une garnison du génie, militaire ou civil, de l'effectif minimal requis s'y trouve : 10 pour une gare, 100 pour un grand port (commerce, militaire), 10 pour un port de pêche ou de plaisance, 100 pour un aéroport ou une base aérienne, 100 pour une CNPE, 10 pour les autres lieux.
- La garnison suffit, même avec des zombies dans l'emprise. Sous l'effectif minimal, le lieu est aussitôt inutilisable.
- Centrales (CNPE, barrages) : la garnison n'est exigée que pour le redémarrage.
- Une piste sommaire construite par le joueur n'exige aucune garnison.
- Un lieu à cheval sur plusieurs communes appartient à toutes.
- Le statut des communes est cosmétique et n'influence pas le gameplay.

## Questions ouvertes

- Aucune pour l'instant.
