# TP2 – Niveau de collecte (Unreal Engine 5)

Petit niveau en greyboxing (template Third Person) où le joueur doit récupérer
**3 coffres** et **10 pièces** pour terminer le niveau.

## Contrôles

| Touche | Action |
|---|---|
| W A S D | Se déplacer |
| Souris | Caméra |
| Espace | Sauter |
| **E** | Interagir (coffres, levier) |

## Condition de victoire

Le niveau est terminé seulement quand le joueur possède **en même temps**
3 coffres **ET** 10 pièces. Un Print String « NIVEAU TERMINÉ ! » s'affiche et
le jeu se met en pause (`Set Game Paused`).

## Architecture (séparation des responsabilités)

| Asset | Type | Responsabilité |
|---|---|---|
| `IA_Interact` | Input Action | Action « Interagir » liée à la touche E dans `IMC_Default` |
| `BPI_Interactable` | Blueprint Interface | Contrat commun pour tout objet activable avec E |
| `BP_ThirdPersonCharacter` | Character | Lit l'input E et appelle `Interact` sur l'objet interactif à proximité |
| `BP_Chest` | Actor | Coffre réutilisable : ouverture animée du couvercle (Timeline), une seule fois |
| `BP_Coin` | Actor | Pièce réutilisable : ramassée au contact, puis détruite |
| `PS_Collector` | PlayerState | Stocke `CoinCount` / `ChestCount`, affiche les Print String, diffuse `OnCollectiblesChanged` |
| `GM_Collect` | GameMode | Écoute le Player State et valide la condition de victoire (pause) |
| `BP_Lever` | Actor | Élément interactif : actionné avec E, ouvre la porte ciblée (`TargetDoor`) |
| `BP_Door` | Actor | Porte qui descend dans le sol par animation (Timeline) |
| `BP_Lava` | Actor | Zone de lave décorative (greyboxing) autour de l'île |
| `WBP_HUD` | Widget Blueprint | HUD (bonus) : affiche Pièces x / 10 et Coffres x / 3 en lisant le Player State |

### Communication entre Blueprints

```
Personnage --(interface Interact)--> Coffre / Levier
Coffre / Pièce --(AddChest / AddCoin)--> PS_Collector
PS_Collector --(Event Dispatcher OnCollectiblesChanged)--> GM_Collect (victoire)
Levier --(référence TargetDoor)--> Porte
WBP_HUD --(lecture seule, binding)--> PS_Collector
```

Les objets ne se connaissent pas entre eux : le personnage ne sait pas ce
qu'est un coffre (il passe par l'interface), et les coffres/pièces ne savent
pas comment on gagne (c'est le rôle du Game Mode).

## Design du niveau

Les pièces sont cachées pour qu'on n'en voie presque aucune depuis le départ.

- **Départ** : ruelles et village de caisses qui bloquent la vue, levier — 1 pièce derrière les caisses, 1 au bout de la ruelle
- **Salle fermée** (porte rouge) : ouverte seulement par le levier — 1 coffre + 2 pièces
- **Salle 2** : 1 pièce dans un coin, et 1 pièce **sur le toit**, accessible par un escalier de caisses
- **Parcours de plateformes** : blocs à franchir en sautant — 2 pièces
- **Mer de lave** : on saute de pierre en pierre — 1 pièce sur une pierre, 1 coffre sur l'île au bout
- **Tour** : grand bloc accessible par une longue rampe — 1 pièce sur la rampe + 1 coffre au sommet

## HUD (bonus)

`WBP_HUD` est créé au `BeginPlay` du personnage (`Create Widget` → `Add to Viewport`).
Ses textes sont liés (binding) aux compteurs de `PS_Collector`. Les Print String
exigés par l'énoncé sont conservés.

Tout est construit en greyboxing (formes de base et matériaux simples créés dans le projet).

## Ouvrir le projet

1. Cloner le dépôt (avec Git LFS installé : `git lfs install`)
2. Ouvrir le fichier `.uproject` avec Unreal Engine 5
3. Lancer le niveau `L_Collect` et appuyer sur Play
