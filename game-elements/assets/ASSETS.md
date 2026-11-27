# Assets

Liste des assets (modèles 3D, animations, sons, icônes) à trouver pour le jeu, tirée des fiches de [UNITS.md](../units/UNITS.md), [CONSTRUCTIONS.md](../constructions/CONSTRUCTIONS.md), [ZOMBIES.md](../zombies/ZOMBIES.md), [LOCATIONS.md](../locations/LOCATIONS.md) et [RESOURCES.md](../resources/RESOURCES.md). Les fichiers eux-mêmes ne sont **jamais versionnés** : chaque asset est inscrit au manifeste de [beta1-assets](https://github.com/labatailledefrance/beta1-assets) (source, auteur, licence, hash), qui le télécharge ([F-05](../../OPEN-PROBLEMS.md#f-05--licence)).

----

## Décisions

- **Pas de génération par IA** pour les modèles 3D : essai de TRELLIS (texte vers 3D) abandonné, résultat jugé trop laid ([E-01](../../OPEN-PROBLEMS.md#e-01--volume-de-contenu)).
- Les assets viendront de **sources existantes**. Toute source convient si l'asset est en **CC0-1.0, CC-BY-4.0 ou OFL-1.1** (déjà utilisées : Sketchfab, poly.pizza, Poly Haven, OpenGameArt).
- **Direction artistique** : **low-poly**, dans l'esprit de Project Zomboid (vue isométrique lisible), mais avec des **textures moins détaillées**. C'est le critère de choix de tout asset : un modèle réaliste ou très texturé ne se mélange pas aux autres. Les bâtiments générés (fenêtres dans la texture, P-08) suivent la même direction.
- **Passe pixel art** ([E-01](../../OPEN-PROBLEMS.md#e-01--volume-de-contenu)) : elle unifie les modèles à l'écran ; un modèle plus texturé est acceptable s'il est réduit et rendu mat. Premiers assets intégrés à P-24 (Charles de Gaulle, Leclerc, Rafale, VBCI, personnages de Quaternius, textures de Poly Haven) : attributions dans le manifeste de [beta1-assets](https://github.com/labatailledefrance/beta1-assets) (provisoirement `assets/README.md`, à la racine, hors dépôt).
- **Navires civils** : il en faut, pour transporter les colons (navires de colons, unités simulées, [A-15](../../OPEN-PROBLEMS.md#a-15--civils-en-métropole--colonies)) et les importations.

## Ce qui n'est pas à chercher

Le jeu le génère lui-même à partir des données ou par le code :

- **Bâtiments** : générés au chargement depuis la BD TOPO (B-02, P-08). Seuls les **monuments** fidèles (liste écrite à la main) demanderont des modèles, plus tard.
- **Relief, routes, voies ferrées, cours d'eau** : tirés de l'IGN.
- **Gravats** des bâtiments effondrés : un relief généré sur l'emprise.
- **Symboles militaires** (APP-6) en vue stratégique : générables par code.
- **Fumée, feu, eau, traçantes** : surtout des shaders et des particules ; quelques textures de particules au plus.

## Comment lire la liste

- **Priorité** : **1** = nécessaire au premier prototype jouable ; **2** = cœur du jeu ; **3** = plus tard.
- **Variantes** : les versions d'une unité partagent en général **un modèle de base** ; une variante ne change que des pièces (tourelle, armement, cabine). Il vaut mieux trouver un modèle de base propre que dix modèles disparates.
- **Statut** : *À trouver*, *Trouvé* (candidat repéré), *Retenu*.
- Pour chaque asset retenu, noter : **source, auteur, licence, URL, modifications**. Seules les licences **CC0-1.0, CC-BY-4.0 et OFL-1.1** sont acceptées : le jeu déployé sert les assets aux joueurs ; tout autre asset est éliminé ([F-05](../../OPEN-PROBLEMS.md#f-05--licence)).

----

## 1. Personnages

Un seul squelette humain partagé par tous les personnages permet de réutiliser les mêmes animations partout.

| Asset | Variantes à couvrir | Priorité | Statut |
|---|---|---|---|
| Soldat | Tenue de combat, casque, gilet ; sac ; Légion étrangère (képi blanc hors combat, à vérifier) ; forces spéciales | 1 | À trouver |
| Zombie | Plusieurs silhouettes (homme, femme, tenues civiles déchirées) ; zombie en feu | 1 | À trouver |
| Civil (colon) | Hommes, femmes ; tenue civile, vêtements chauds | 2 | À trouver |
| Ouvrier du génie civil | Gilet de chantier, casque | 2 | À trouver |
| Gendarme, policier | Uniformes | 2 | À trouver |
| Sapeur-pompier | Tenue de feu, casque | 2 | À trouver |
| Équipe sanitaire | Combinaison de protection, masque | 2 | À trouver |
| Personnel de santé | Blouse, tenue de bloc | 3 | À trouver |
| Milicien | Tenue civile, brassard, arme sommaire | 3 | À trouver |
| Blessé, amputé | Pansements ; béquilles | 3 | À trouver |

### Animations

| Animation | Pour qui | Priorité |
|---|---|---|
| Attente, marche, course | Tous | 1 |
| Tir debout, accroupi, couché | Soldats, gendarmes, miliciens | 1 |
| Rechargement | Soldats | 2 |
| Lancer de grenade | Soldats | 2 |
| Tir au lance-roquettes, épaule | Grenadier / antichar | 2 |
| Mitrailleuse sur trépied, à deux | Mitrailleur | 2 |
| Mort, chute ; corps au sol | Tous (les corps sont générés à l'affichage) | 1 |
| Course de zombie, bras tendus | Zombie | 1 |
| Attaque, agrippement | Zombie | 1 |
| Escalade, entassement | Zombie (franchissement d'obstacles) | 2 |
| Combustion (zombie qui avance en feu) | Zombie | 3 |
| Travail : pelle, marteau, porter une charge | Sapeurs, ouvriers | 2 |
| Brancard à deux, porter un corps | Sanitaires, infirmiers | 2 |
| Lance à incendie | Pompiers | 3 |
| Lance-flammes | Spécialiste incendiaire | 3 |

## 2. Armes portées

Petits modèles accrochés à la main du personnage.

| Asset | Priorité | Statut |
|---|---|---|
| Fusil d'assaut HK416F (avec et sans lance-grenades de 40 mm) | 1 | À trouver |
| Pistolet Glock 17 | 3 | À trouver |
| Mitrailleuse légère Minimi | 2 | À trouver |
| Mitrailleuse MAG 58 sur trépied | 2 | À trouver |
| Fusil de précision SCAR-H PR ou HK417 | 2 | À trouver |
| Fusil anti-matériel PGM Hécate II | 2 | À trouver |
| Lance-roquettes AT4 CS | 2 | À trouver |
| Poste de tir Akeron MP | 3 | À trouver |
| Lance-flammes (fabriqué) | 3 | À trouver |
| Grenade à main | 3 | À trouver |
| Drone léger (télépilote) | 3 | À trouver |
| Outils : pelle, masse, brancard, sac mortuaire | 2 | À trouver |

## 3. Véhicules terrestres

| Asset | Variantes à couvrir | Priorité | Statut |
|---|---|---|---|
| Griffon (6×6) | VTT, EPC, VOA, VSS, MEPAC, génie | 1 | À trouver |
| VBCI (8×8) | VCI, VPC | 1 | À trouver |
| Camion logistique | PPT, PPLD ; TRM 10000, GBC 180 ; citerne | 1 | À trouver |
| Serval (4×4) | VPB, MCT, LAD, sol-air Mistral, génie, NRBC, guerre électronique | 2 | À trouver |
| VBL | Base, Ultima, porte-missiles, VB2L | 2 | À trouver |
| Jaguar (6×6) | Version unique | 2 | À trouver |
| Leclerc | XLR, non rénové | 2 | À trouver |
| CAESAR | Mk I, Mk II | 2 | À trouver |
| LRU | Version unique | 3 | À trouver |
| VLTP-NP (pick-up tactique) | — | 2 | À trouver |
| Fardier, quad, moto | À recenser | 3 | À trouver |
| Dépanneurs et porte-chars | DCL (sur base Leclerc), dépanneur à roues, tracteur porte-char | 3 | À trouver |
| EBG VT | — | 2 | À trouver |
| EGAME (pelle mécanique) | — | 2 | À trouver |
| SPRAT (poseur de pont) | Avec et sans travure | 3 | À trouver |
| PFM (pont flottant) | Modules à l'eau, sur camion | 3 | À trouver |
| SOUVIM | Détection, neutralisation | 3 | À trouver |
| Buffalo | — | 3 | À trouver |
| Train de fret | Locomotive électrique ; wagons plats, conteneurs, citernes | 2 | À trouver |
| Train de passagers | Rame électrique | 2 | À trouver |

### Véhicules civils et de soutien

Récupérés sur les routes, ou épaves (barricades, routes encombrées). Une version **épave** de chaque modèle est utile.

| Asset | Variantes à couvrir | Priorité | Statut |
|---|---|---|---|
| Voitures | Citadine, berline, break, SUV | 1 | À trouver |
| Utilitaires | Fourgonnette, fourgon | 2 | À trouver |
| Camions civils | Porteur, semi-remorque, citerne | 2 | À trouver |
| Bus, car | — | 2 | À trouver |
| Engins agricoles | Tracteur, remorque | 3 | À trouver |
| Engins de chantier | Pelle, chargeuse, bulldozer, grue mobile | 2 | À trouver |
| Moto, scooter | — | 3 | À trouver |
| Gendarmerie, police | Voiture de patrouille, fourgon, blindé Centaure | 2 | À trouver |
| Pompiers | Fourgon-pompe, camion-citerne, échelle | 3 | À trouver |
| Ambulance | — | 3 | À trouver |

## 4. Aéronefs

Avec le train rentré et sorti, et les rotors animés pour les hélicoptères.

| Asset | Variantes à couvrir | Priorité | Statut |
|---|---|---|---|
| Rafale | B (biplace), C, M (crosse d'appontage) | 2 | À trouver |
| Tigre | HAP, HAD | 2 | À trouver |
| NH90 Caïman | TTH (Terre), NFH (Marine) | 2 | À trouver |
| Caracal (H225M) | — | 3 | À trouver |
| Gazelle | Canon, HOT, Viviane, Mistral | 3 | À trouver |
| Guépard (H160M) | Terre, Air, Marine | 3 | À trouver |
| A400M | — | 2 | À trouver |
| C-130 | H, J, KC-130J | 3 | À trouver |
| Drones MALE | MQ-9 Reaper, Patroller | 3 | À trouver |
| Canadair CL-415 | — | 3 | À trouver |
| Dash 8 Q400 MR | — | 3 | À trouver |

## 5. Navires

Les navires de la Marine sont nommés individuellement, mais un modèle par **classe** suffit.

| Asset | Variantes à couvrir | Priorité | Statut |
|---|---|---|---|
| PHA classe Mistral | — | 2 | À trouver |
| Engins de débarquement | EDA-R, EDA-S, CTM | 2 | À trouver |
| FREMM | ASM, DA | 2 | À trouver |
| FDI | — | 3 | À trouver |
| Porte-avions Charles de Gaulle | — | 3 | À trouver |
| BRF (classe Jacques Chevallier) | — | 3 | À trouver |
| SNA classe Suffren | — | 3 | À trouver |
| Chasseurs de mines | Tripartite, drones SLAM-F | 3 | À trouver |
| Patrouilleurs et vedettes | Océanique, côtière, gendarmerie maritime, fluviale | 2 | À trouver |
| SNLE classe Triomphant | Invisibles en jeu : pas de modèle | — | — |

### Navires civils

Navires de colons et d'importations, chargés et déchargés à la main par le joueur.

| Asset | Variantes à couvrir | Priorité | Statut |
|---|---|---|---|
| Ferry (passagers et véhicules) | — | 2 | À trouver |
| Paquebot | — | 3 | À trouver |
| Cargo | Porte-conteneurs, vraquier | 2 | À trouver |
| Pétrolier | — | 2 | À trouver |
| Navire roulier | — | 3 | À trouver |
| Bateaux de pêche et de plaisance | Épaves dans les ports | 3 | À trouver |

## 6. Constructions

Chaque construction a une barre de vie : prévoir un état **endommagé** si le modèle s'y prête.

| Asset | Priorité | Statut |
|---|---|---|
| Réseau de barbelés | 1 | À trouver |
| Mur de sacs de sable | 1 | À trouver |
| Chevaux de frise | 2 | À trouver |
| Bastion de gabions (type HESCO) | 2 | À trouver |
| Mur en béton préfabriqué | 2 | À trouver |
| Fossé, talus / merlon, cratère | 2 | Généré (modification du relief, B-05) |
| Champ de mines / pièges (signalisation) | 3 | À trouver |
| Poste de tir surélevé / tour de guet | 2 | À trouver |
| Projecteur d'éclairage de périmètre | 3 | À trouver |
| Point de contrôle (barrière, guérite) | 2 | À trouver |
| Zone de poser hélicoptère (balisage) | 3 | À trouver |
| Dépôt de carburant (citernes souples ou fixes) | 2 | À trouver |
| Dépôt de munitions (abris) | 2 | À trouver |
| Atelier de maintenance (hangar) | 3 | À trouver |
| Hôpital de campagne (tentes, modules) | 2 | À trouver |
| Cuisine / cantine (tente) | 3 | À trouver |
| Logements (tentes, modules) | 2 | À trouver |
| Groupe électrogène | 3 | À trouver |
| Point de traitement de l'eau | 3 | À trouver |
| Fosse commune | 3 | Peut-être généré (relief) |
| Bûcher d'incinération | 3 | À trouver |

## 7. Cargaisons et objets

Le modèle de données emballe les ressources : conteneurs, palettes, caisses, lots (P-20).

| Asset | Priorité | Statut |
|---|---|---|
| Caisse de munitions | 1 | À trouver |
| Palette (vide, chargée) | 1 | À trouver |
| Conteneur maritime (20 et 40 pieds) | 2 | À trouver |
| Fût, jerrican | 2 | À trouver |
| Citerne souple | 3 | À trouver |
| Sacs (vivres, sable) | 2 | À trouver |
| Sac mortuaire | 2 | À trouver |
| Mobilier urbain : lampadaire, feu tricolore, abribus, poubelle, banc, panneaux | 3 | À trouver |

## 8. Végétation

Les arbres sont placés au chargement depuis les données (B-01). Le feu est simulé arbre par arbre : il faut un état **brûlé**.

| Asset | Variantes à couvrir | Priorité | Statut |
|---|---|---|---|
| Arbre feuillu | Quelques silhouettes ; brûlé | 1 | À trouver |
| Conifère | Quelques silhouettes ; brûlé | 1 | À trouver |
| Haie, buisson | — | 2 | À trouver |
| Végétation tropicale (outre-mer) | Palmier, etc. | 3 | À trouver |

## 9. Sons

| Asset | Variantes à couvrir | Priorité | Statut |
|---|---|---|---|
| Tirs d'armes légères | Par arme (HK416F, Minimi, MAG 58, Hécate II…), proche et lointain | 1 | À trouver |
| Explosions | Grenade, obus, roquette, bombe ; proche et lointain | 1 | À trouver |
| Zombies | Cris isolés, râles, rumeur de foule, course | 1 | À trouver |
| Moteurs | Véhicule à roues, chenillé, camion, hélicoptère, avion à réaction, turbopropulseur, navire | 2 | À trouver |
| Chantier | Pelle, marteau, engin, scie | 3 | À trouver |
| Feu | Incendie, lance-flammes | 3 | À trouver |
| Ambiance | Vent, pluie, ville vide, mer, oiseaux | 2 | À trouver |
| Radio | Bruits de transmission | 3 | À trouver |
| Interface | Clic, ordre donné, alerte, vote | 2 | À trouver |
| Explosion nucléaire | — | 3 | À trouver |
| Musique | Facultatif | 3 | À trouver |

## 10. Interface

| Asset | Priorité | Statut |
|---|---|---|
| Police de caractères (OFL-1.1 ou CC0-1.0, accents français) | 1 | À trouver |
| Icônes des ordres (déplacer, attaquer, patrouille, axe de frappe, construire, charger, décharger…) | 1 | À trouver |
| Icônes des ressources (une par ressource de [RESOURCES.md](../resources/RESOURCES.md)) | 2 | À trouver |
| Icônes des lieux (CTM, CNPE, barrage, base aérienne, aérodrome, gare, port, mairie, survivants, base et dépôt militaires, raffinerie et dépôt pétrolier, usine d'armement, hôpital, eau potable) | 2 | À trouver |
| Symboles des couches dynamiques (coupures, ponts à charge limitée, ports minés, alertes, météo, régions électriques) | 2 | À trouver |
| Curseurs | 2 | À trouver |

----

## Questions ouvertes

- Monuments fidèles : où trouver les modèles ? Piste : les **modéliser** en low-poly à partir des parties de bâtiments d'OpenStreetMap (Simple 3D Buildings, licence ODbL), plutôt que des scans photogrammétriques, trop détaillés pour la direction artistique. Liste à écrire ([B-02](../../OPEN-PROBLEMS.md#b-02--génération-des-bâtiments)).
