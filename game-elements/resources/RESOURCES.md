# Ressources

Liste commune des ressources du jeu. Les fiches d'unités reprennent ces noms exacts, afin que la chaîne logistique reste lisible : quelle unité dépend de quoi, et ce qui s'arrête quand un stock s'épuise.

Les quantités et les taux de consommation seront définis dans le mod `base`, en tests de jeu ([D-06](../../OPEN-PROBLEMS.md#d-06--équilibrage)).

----

## Principes

- **Tout ce qui est consommé a été produit, transporté et livré.** Une munition tirée a d'abord été fabriquée à partir de matières premières, puis acheminée jusqu'à l'unité.
- **Importations** : le pétrole brut, l'électronique et les composants que la France ne fabrique pas (dont les pièces américaines) sont fournis par des pays non spécifiés. Leur livraison n'est pas simulée : ils s'ajoutent aux stocks des ports d'outre-mer environ toutes les deux semaines. Le joueur les charge ensuite sur ses navires, les achemine à vitesse réelle et les décharge ; la logistique reste complète ([G-03](../../OPEN-PROBLEMS.md#g-03--importations)). Au début de la partie, tout vient de l'outre-mer.
- **Les stocks existants sont présents dans le jeu.** Au début de la partie, le joueur dispose des stocks réels de l'arsenal français (dépôts, soutes, navires), en quantités réduites par la guerre.
- **L'efficacité est fidèle à la réalité.** Une munition conçue contre des blindés reste peu efficace contre des zombies. C'est au joueur de s'adapter, en fabriquant des munitions adaptées et en les livrant sur place.
- **La main-d'œuvre est une ressource.** Les soldats, les ouvriers et les civils doivent être nourris, soignés et logés. Sacrifier des civils a un coût réel : la perte de main-d'œuvre.
- **Chaque objet a une masse et un volume.** La cargaison est gérée au détail (voir [SYSTEMES.md](../systemes/SYSTEMES.md)).

----

## Carburants

| Ressource | Utilisateurs | Notes |
|---|---|---|
| **F-34** | Véhicules terrestres, aéronefs de l'armée de Terre et de l'armée de l'Air | Kérosène militaire (équivalent JP-8). Carburant unique terre-air. |
| **F-44** | Aéronefs embarqués (Rafale M, hélicoptères de la Marine) | Kérosène à point éclair élevé (équivalent JP-5), exigé à bord des navires. |
| **F-76** | Navires à propulsion classique | Gazole marine. |
| **Gazole routier** | Véhicules civils, groupes électrogènes (dont ceux du CTM de Rosnay hors région alimentée, et des centrales à redémarrer) | Stocks civils récupérables dans les stations et les dépôts. Utilisable en dépannage par certains véhicules militaires. |
| **Essence** | Motos, quads, petits moteurs, engins civils | Également matière première des armes incendiaires improvisées. |
| **Jet A-1** | Aviation civile, Canadair | Proche du F-34 ; stocks dans les aéroports civils. |
| **Combustible nucléaire** | Charles de Gaulle, SNA | Rechargement très rare, hors logistique courante. |

## Énergie

- **Électricité** : réseau, groupes électrogènes, centrales. Une [centrale nucléaire](../locations/cnpe.md) ou un [barrage](../locations/barrages.md) remis en service alimente sa région indéfiniment, sans approvisionnement. Pas de centrales thermiques dans le jeu. L'outre-mer a l'électricité d'office. Indispensable aux usines, aux hôpitaux et à la recharge des batteries.
- **Batteries et piles** : radios, vision nocturne, drones, désignateurs laser.
- **Bois** : combustible de l'incinération des corps et du chauffage.

## Munitions

### Armes légères

- **9×19 mm** : pistolets.
- **5,56×45 mm OTAN** : HK416F, Minimi.
- **7,62×51 mm OTAN** : HK417, MAG 58, mitrailleuses de tourelleau.
- **12,7×99 mm** : mitrailleuses lourdes, Hécate II.
- **Calibre 12** : fusils à pompe (dotation limitée ; candidat sérieux pour la production adaptée).
- **40 mm (grenades)** : lance-grenades individuels et automatiques.
- **Grenades à main** : à fragmentation, fumigènes, lacrymogènes, incendiaires.

### Canons

- **20 mm** : canons d'appoint (navires, anciens hélicoptères).
- **25×137 mm** : VBCI.
- **30×113 mm** : Tigre.
- **30×150 mm** : Rafale.
- **40 mm CTA** : Jaguar (munition télescopée ; fabricable en France).
- **76 mm** : frégates.

### Artillerie et chars

- **Obus de mortier de 81 et 120 mm**.
- **Obus de 155 mm** : explosifs, fumigènes, éclairants, antichars à sous-munitions (BONUS), guidés.
- **Obus de 120 mm pour char** : flèche (APFSDS, « sabot ») et explosif.
- **Charges propulsives** : modulaires pour le 155 mm.

### Roquettes et missiles

- **Roquettes de 68 mm** : hélicoptères.
- **Roquettes de 227 mm (GMLRS)** : LRU.
- **Missiles antichars** : Akeron MP, Hellfire.
- **Missiles sol-air et air-air** : Mistral, MICA, Meteor, Aster.
- **Missiles antinavires et de croisière** : Exocet, MdCN, SCALP.
- **Torpilles** : MU90, F21.

### Bombes

- **Bombes guidées** : AASM Hammer, GBU.

### Armes nucléaires

- **ASMP-A (tête TNA, 300 kt)** : stockés sur le Charles de Gaulle, tirés uniquement par des Rafale M partis du porte-avions. Explosion aérienne : boule de feu 600 m, rayonnement thermique 7 km, souffle 13 km.
- **M51 (têtes TNO, 100 kt)** : emportés par les SNLE ; explosion aérienne : boule de feu 500 m, rayonnement thermique 4 km, souffle 9 km ; utilisables seulement si le joueur contrôle le [CTM de Rosnay](../locations/ctm-rosnay.md).
- Pas de dimension politique : leur emploi est un choix tactique comme un autre.

### Explosifs du génie

- **Explosifs de démolition** (pains de plastic, charges).
- **Mines et pièges**.
- **Détonateurs et cordeau**.

### Munitions adaptées (à fabriquer)

Munitions absentes ou rares dans les stocks réels, que le joueur peut produire pour mieux combattre les zombies. Liste ouverte :

- Obus de 120 mm à mitraille (« canister ») pour les chars.
- Obus de 25, 30 et 40 mm à fragmentation ou à éclatement programmé, en plus grand nombre.
- Obus de 155 mm à fragmentation optimisée contre les hordes.
- Cartouches de calibre 12 à chevrotine.
- Munitions et grenades incendiaires.
- Carburant épaissi pour lance-flammes.

## Vivres et eau

- **Rations de combat**.
- **Vivres frais et en vrac** : cuisines de campagne, cantines des usines.
- **Eau potable**.
- **Eau non potable** : lutte contre les incendies, décontamination, industrie.
- **Retardant** : produit de lutte contre les incendies, chargé au sol (Dash 8).

## Santé

- **Matériel médical** : pansements, garrots, kits de suture.
- **Médicaments** : antalgiques, antibiotiques, anesthésiques.
- **Sang et produits sanguins**.

## Hygiène et traitement des corps

- **Sacs mortuaires** : évacuation des corps ([A-16](../../OPEN-PROBLEMS.md#a-16--gestion-des-corps)).

## Habillement

- **Vêtements chauds** et **couvertures** : protègent du froid, donc des maladies ([A-17](../../OPEN-PROBLEMS.md#a-17--maladies)).

## Maintenance

- **Pièces détachées** : moteurs, transmissions, électronique. Les pièces américaines (Reaper notamment) ne sont pas produites en France : elles sont importées (voir *Importations*).
- Les réparations se font en atelier.
- **Pneus** et **chenilles**.
- **Tubes de canon** : ils s'usent à chaque tir et doivent être remplacés.
- **Lubrifiants et liquides techniques**.

## Matières premières et production

Chaînes de production à détailler plus tard. Premières briques :

- **Pétrole brut** → raffinage → carburants.
- **Acier**, **cuivre**, **laiton**, **plomb** → corps d'obus, douilles, balles.
- **Nitrocellulose, poudres et propergols** → charges propulsives.
- **Explosifs** (TNT, hexogène…) → chargement des obus et des charges du génie.
- **Produits chimiques et électronique** → fusées d'obus, munitions guidées.
- **Sites de production** : usines, raffineries, arsenaux. Ils doivent être reconquis, alimentés en énergie et en main-d'œuvre.

## Main-d'œuvre

- **Militaires** : combattants et soutien.
- **Ouvriers** : production, réparation, infrastructures. Compétences gérées lorsqu'elles sont liées aux ressources (électricien, soudeur, chimiste…).
- **Civils** : colons débarqués de l'outre-mer, et survivants des rares poches secourues. Ils consomment des vivres et de l'eau, peuvent travailler ou être mobilisés, et peuvent être sacrifiés : la main-d'œuvre perdue est le vrai coût. Pas d'âge modélisé.
- **Capacité** : la main-d'œuvre se loge et travaille dans les bâtiments : un colon logé par 35 m² de plancher, un poste de travail par 20 m² (valeurs de départ, [SYSTEMES.md](../systemes/SYSTEMES.md#colonisation)).
