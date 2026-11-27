# Développement

Comment le code de **La Bataille de France** est organisé, vérifié, publié et déployé. La conception du jeu est dans le [README](../README.md) et les [problèmes ouverts](../OPEN-PROBLEMS.md) ; cette page décrit la fabrique.

Pour contribuer, lire d'abord [CONTRIBUTING.md](https://github.com/labatailledefrance/.github/blob/main/CONTRIBUTING.md) : langue, licence et DCO, commits signés, règles du code, assets. Cette page ne le répète pas.

## Dépôts

Tous publics, dans l'organisation [`labatailledefrance`](https://github.com/labatailledefrance).

| Dépôt | Rôle |
|---|---|
| [.github](https://github.com/labatailledefrance/.github) | CONTRIBUTING, SECURITY, modèles d'issues et de PR ; **workflows réutilisables** appelés par les autres dépôts |
| [beta1-common](https://github.com/labatailledefrance/beta1-common) | Bibliothèque partagée : positions (`geo`), `Reader` et `Writer` binaires (`codec`), messages et handshake (`protocol`), aléatoire déterministe (`rng`) ; bientôt le format des tiles |
| [beta1-server](https://github.com/labatailledefrance/beta1-server) | Serveur de simulation : Bevy sans rendu, 20 Hz, WebSocket ; il fait autorité |
| [beta1-client](https://github.com/labatailledefrance/beta1-client) | Client navigateur : Bevy en WebAssembly, WebGPU seul ; il affiche et envoie des ordres |
| [beta1-datachain](https://github.com/labatailledefrance/beta1-datachain) | Chaîne de données : IGN et INSEE vers tiles de 1 km et graphe routier ; image lancée en Job par le chart `game` (en construction) |
| [beta1-assets](https://github.com/labatailledefrance/beta1-assets) | Manifeste et scripts des assets tiers ; aucun asset dans git (en construction) |
| [beta1-helm](https://github.com/labatailledefrance/beta1-helm) | Chart `game` : un monde sur Kubernetes (serveur, client, tiles et leur Job de production, ingress, NetworkPolicies) |
| [documentation](https://github.com/labatailledefrance/documentation) | Conception du jeu (ce dépôt) |
| [prototypes](https://github.com/labatailledefrance/prototypes) | Prototypes techniques jetables (P-00 à P-26), dont on porte le code |
| [agent](https://github.com/labatailledefrance/agent) | Configuration de Claude Code (instructions, skills, agents) |

Dépendances entre dépôts :

```
                 beta1-common (tag vX.Y.Z)
               /        |          \
   beta1-server   beta1-client   beta1-datachain ──> tiles
         \              /                              |
          images ghcr.io            beta1-assets ──> assets
                \                                      |
                 beta1-helm (chart game) <─────────────┘
```

`server`, `client` et `datachain` épinglent `beta1-common` sur **le même tag** (`version = "=0.2.0", git = …, tag = "v0.2.0"`). Le protocole porte un **fingerprint** (hash BLAKE3 de ses sources, calculé par `build.rs`) que client et serveur comparent au handshake : un client d'une autre version est refusé. Monter `beta1-common`, c'est donc livrer client et serveur ensemble, ce qui est déjà la règle : toute mise à jour du serveur remet le monde à zéro ([H-01](../OPEN-PROBLEMS.md#h-01--architecture--serveur-unique-faisant-autorité)).

## Standard

Les règles sont dans CONTRIBUTING ; voici ce qui les fait respecter.

| Règle | Vérifiée par |
|---|---|
| Format (`max_width = 120`) | `cargo fmt --check` |
| Aucun avertissement | `cargo clippy --all-targets -- -D warnings`, `RUSTFLAGS=-D warnings` |
| Pas d'`unsafe` | `unsafe_code = "forbid"` dans `Cargo.toml`, dont la CI contrôle la présence |
| Dépendances : licence compatible AGPL, pas de vulnérabilité connue, sources autorisées | `cargo deny check` |
| Dépendances publiées depuis au moins deux semaines | revue de la PR ; Dependabot attend 14 jours (`cooldown`) |
| Actions GitHub épinglées par SHA | zizmor |
| Déterminisme (même hash d'état à 1 et 16 threads) | `--determinism` du serveur, en CI |
| Le décodeur ne panique sur aucune entrée | fuzzing (`cargo fuzz`) de `beta1-common` |
| Pas de secret dans l'historique | gitleaks |
| Ordre des systèmes explicite (aucune ambiguïté) | détection d'ambiguïté de Bevy en erreur, au démarrage du serveur en CI |
| Catalogue valide (champs connus, références, bornes) | `beta1-server --check` |
| JSON Schema du catalogue à jour | régénéré en CI et comparé à la version du dépôt |

L'analogie : chaque dépôt est un service, `.github` est sa **bibliothèque de pipelines** partagée, versionnée comme une dépendance. Un dépôt appelle `labatailledefrance/.github/.github/workflows/rust.yml@<SHA> # v2.0.0` : changer de version de pipeline est une PR explicite, jamais une dérive silencieuse.

## Organisation du serveur

Décisions : [E-02](../OPEN-PROBLEMS.md#e-02--format-des-données), [F-01](../OPEN-PROBLEMS.md#f-01--moteur), [F-03](../OPEN-PROBLEMS.md#f-03--modding).

### Données et code

Ce qui **décrit** le jeu est en données ; ce qui le **fait tourner** est en Rust. Comme dans Kubernetes : la ressource est déclarative, le controller est compilé.

- **Catalogue** : un fichier TOML par objet (unité et ses versions et postes, arme, munition, véhicule, ressource, recette, construction, lieu, paramètres des zombies, scénario et état initial), référencé par identifiant. Les textes affichés sont des clés de traduction (Fluent, français seul au début, E-05).
- **Validation stricte au démarrage** : champs inconnus refusés, références et bornes vérifiées ; le serveur ne démarre pas sur une erreur. `beta1-server --check` en CI ; JSON Schema versionné dans le dépôt, vérifié à jour en CI, pour l'éditeur.
- **Compilation au chargement** en une ressource `Catalogue` indexée par entiers ; son hash entre dans le hash d'état. Le client le reçoit au handshake (types dans `beta1-common`).
- **Pas de rechargement à chaud** : une donnée changée, c'est un redémarrage, donc un monde remis à zéro.
- **Tout est mod**, à la manière de Factorio : le contenu de base est le mod `base` (`beta1-server/mods/base/`, dans l'image), chargé comme les autres. Chaque mod a un `mod.toml` (identifiant, version, dépendances) ; l'ordre suit les dépendances, puis l'ordre déclaré. Les mods d'un opérateur arrivent par une PVC, listés dans les values du chart `game` ; la partie officielle ne charge que `base`. Pas de code de mod chargé à l'exécution ; un mod de comportement est un fork.
- **Instantanés** : ils portent le hash du catalogue ; un hash différent empêche la reprise (A-13).

### Plugins et phases

```
beta1-server/src/
  main.rs          CLI, assemblage des plugins
  app.rs           les phases et leur ordre : le seul endroit où lire le flux d'un tick
  catalogue/       chargement des mods, fusion et validation
  tiles/           actives, inertes, sûres ; terrain
  zombies/         perception, relais, champs, déplacement
  unites/          ordres, IA (D-02), déplacement
  combat/
  logistique/
  routes/          CCH, tâches de fond à tick fixé
  reseau/          WebSocket, handshake, send queue, quota des ordres coûteux
  sauvegarde/
  determinisme/    hash d'état
mods/base/         le contenu de base : mod.toml, TOML, textes Fluent
```

- Un **plugin par domaine** : il possède ses composants, ses systèmes et ses messages ; il lit les composants des autres, n'écrit que les siens (règle de revue).
- Les plugins ne s'appellent pas : ils échangent des **messages** (`combat` émet `ZombieAbattu`, `logistique` le consomme).
- Un tick suit des **phases** dans un ordre fixe : Entrees, Tiles, Perception, Decision, Mouvement, Combat, Logistique, Nettoyage, Sortie.

```rust
#[derive(SystemSet, Debug, Clone, PartialEq, Eq, Hash)]
pub enum Phase { Entrees, Tiles, Perception, Decision, Mouvement, Combat, Logistique, Nettoyage, Sortie }

app.configure_sets(FixedUpdate, (Phase::Entrees, Phase::Tiles, /* … */ Phase::Sortie).chain());
```

- **Détection d'ambiguïté** en erreur : deux systèmes qui touchent la même donnée sans ordre déclaré empêchent le démarrage.
- Chaque plugin se teste seul, dans une `App` minimale avec un petit catalogue de test.

## Pipelines

### Sur chaque PR

| Workflow | Contrôles |
|---|---|
| `rust` | format, clippy, tests (`nextest`), `cargo deny`, test de déterminisme ; pour le client, clippy et construction en `wasm32` |
| `securite` | zizmor (workflows), gitleaks (secrets, tout l'historique), Semgrep CE (SAST), hadolint et trivy (Dockerfiles, configuration) |
| `codeql` | CodeQL (Rust, Actions) |
| `dco` | `Signed-off-by` de l'auteur sur chaque commit |
| `chart` | `helm lint`, kubeconform, kube-linter (`beta1-helm`) |
| fuzzing | 60 s sur une PR, 30 min chaque nuit (`beta1-common`) |

Chaque dépôt appelle ceux qui le concernent, et son ruleset sur `main` les **exige** : une PR qui en échoue un ne peut pas être fusionnée.

| Dépôts | Checks requis |
|---|---|
| `beta1-common`, `beta1-server`, `beta1-client` | `rust`, `securite`, `codeql` (Rust, Actions), `dco` ; fuzzing pour `beta1-common` |
| `beta1-helm` | `chart`, `securite`, `codeql` (Actions), `dco` |
| `.github`, `beta1-datachain`, `beta1-assets`, `documentation`, `prototypes`, `agent` | `securite`, `codeql` (Actions), `dco` |

### Après fusion ou sur un tag

- **Images** (`image`) : construites sur une base `dhi.io` (Docker Hardened Images), publiées sur `ghcr.io/labatailledefrance/<dépôt>` avec SBOM et provenance, scannées par trivy (les vulnérabilités critiques ou élevées corrigeables bloquent), **signées** par cosign sans clé (identité OIDC du workflow) et attestées (SLSA v1). Les PR n'en construisent pas : celles des forks n'ont pas les secrets de dhi.io.
- **Chart** (`chart`) : publié en OCI sur `ghcr.io/labatailledefrance/charts/game` et signé de la même façon.

La signature tient lieu d'approbation : l'image a été produite par **notre** workflow, depuis **notre** dépôt. Elle est vérifiée au déploiement.

## Protection des dépôts

- Ruleset **« main protégée »** sur chaque dépôt : PR obligatoire, checks requis verts, commits **signés**, historique **linéaire**, ni suppression ni force-push, fils de discussion résolus. Aucune exception, administrateurs compris.
- Fusion en **squash** : GitHub signe le commit fusionné.
- Les workflows d'un **contributeur extérieur** ne tournent qu'après approbation d'un mainteneur : une PR ne peut pas exécuter de code sur la CI sans qu'on l'ait lue.
- Workflows en `permissions: {}` par défaut ; chaque job demande le minimum (`contents: read`, `packages: write` pour publier, `id-token: write` pour signer).

## Déployer

Le monde officiel tourne sur `srv.labatailledefrance.fr` (k3s, ingress Traefik), namespace `labatailledefrance`. Le cluster n'a pas de contrôle d'admission : la vérification se fait **avant** l'installation, à la main.

1. Relever les digests des images publiées par la CI de `beta1-server`, `beta1-client` et `beta1-datachain`.
2. Vérifier les signatures du chart et des trois images avec **cosign ≥ 3** (la version des distributions ne lit pas les signatures en bundle OCI).
3. Sur srv : `KUBECONFIG=/etc/rancher/k3s/k3s.yaml helm upgrade game oci://ghcr.io/labatailledefrance/charts/game …`, images **par digest**.

### Tiles

Tout le jeu tient dans un namespace ([H-06](../OPEN-PROBLEMS.md#h-06--diffusion-des-tiles)) :

```
Job datachain-<hash> ──écrit──> PVC sources (~720 Go)   PVC tiles (~100 Go)
  (seul pod qui sort, en 443)                              │ lecture seule
                                           ┌───────────────┴──────────────┐
                                     server (jeu)                  tiles (nginx)
                                attendent <hash>/COMPLET        cache immutable
Traefik : /  → client, /tiles/ → tiles, /ws → server
```

La publication est adressée par ses **entrées** (manifeste des sources, digest de l'image `beta1-datachain`) : le chart sait quel dossier servir avant qu'il existe. Le Job tourne à chaque `helm install` ou `upgrade` ; il sort aussitôt si le dossier existe, sinon il le produit à part puis le renomme. La première génération dure des heures : `helm` rend la main, le jeu attend. Les deux PVC survivent à `helm uninstall`.

Commandes complètes : [README de beta1-helm](https://github.com/labatailledefrance/beta1-helm/blob/main/README.md). Une mise à jour du serveur remet le monde à zéro (H-01).

## Contribuer

Fork, branche, `git commit -s -S`, PR vers `main`, CI verte : le détail est dans [CONTRIBUTING.md](https://github.com/labatailledefrance/.github/blob/main/CONTRIBUTING.md), les vulnérabilités se signalent selon [SECURITY.md](https://github.com/labatailledefrance/.github/blob/main/SECURITY.md).
