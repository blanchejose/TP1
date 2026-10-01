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
3 coffres **ET** 10 pièces. Un message s'affiche et le jeu se met en pause.

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
| `BP_Lever` | Actor | Élément interactif : actionné avec E, ouvre la porte ciblée |
| `BP_Door` | Actor | Porte qui s'ouvre par animation (Timeline) |
| `BP_MovingPlatform` | Actor | Plateforme mobile pour le défi de platforming |
| `WBP_HUD` / `BP_HUD` | Widget / HUD | **Bonus** : compteurs à l'écran + invite d'interaction |

### Communication entre Blueprints

```
Personnage --(interface Interact)--> Coffre / Levier
Coffre / Pièce --(AddChest / AddCoin)--> PS_Collector
PS_Collector --(Event Dispatcher OnCollectiblesChanged)--> GM_Collect (victoire)
                                                       --> WBP_HUD   (affichage)
Levier --(référence TargetDoor)--> Porte
```

Les objets ne se connaissent pas entre eux : le personnage ne sait pas ce
qu'est un coffre (il passe par l'interface), et les coffres/pièces ne savent
pas comment on gagne (c'est le rôle du Game Mode).

## Design du niveau

- **Place centrale** (spawn) : 3 pièces + levier
- **Salle fermée** derrière la porte rouge (ouverte par le levier) : coffre 2 + 2 pièces
- **Parcours de plateformes** : coffre 1 + 3 pièces
- **Tour** accessible par plateforme mobile : coffre 3 + 2 pièces

## Installation

1. Cloner le dépôt (avec Git LFS installé : `git lfs install`)
2. Ouvrir le fichier `.uproject` avec Unreal Engine 5
3. Lancer `L_Collect` et appuyer sur Play
