# Guide pas à pas – TP2 Unreal Engine

Suis les étapes **dans l'ordre**. Compile (bouton **Compile**) et sauvegarde
(**Save**) chaque Blueprint après l'avoir modifié.

> Astuce : dans un Event Graph, fais **clic droit** dans le vide et tape le
> nom du nœud (ex. « Print String ») pour l'ajouter.

---

## ÉTAPE 1 – Créer le projet

1. Ouvre Unreal Engine 5 → **Games** → **Third Person**.
2. Choisis **Blueprint**, décoche **Starter Content**.
3. Nomme le projet `TP2_Collect` → **Create**.
4. Dans le **Content Browser**, clic droit sur `Content` → **New Folder** → `TP2`.
5. Dans `TP2`, crée les dossiers : `Blueprints`, `Input`, `UI`, `Materials`, `Maps`.

## ÉTAPE 2 – Créer le niveau

1. **File → New Level → Basic** → **Create**.
2. **File → Save Current Level As** → `TP2/Maps/L_Collect`.
3. **Edit → Project Settings → Maps & Modes** :
   - Editor Startup Map = `L_Collect`
   - Game Default Map = `L_Collect`

## ÉTAPE 3 – Matériaux (greyboxing)

Pour chaque matériau : clic droit dans `TP2/Materials` → **Material**.

| Nom | Dans le matériau |
|---|---|
| `M_Grey` | Maintiens **3** + clic → couleur gris → brancher dans **Base Color** |
| `M_Gold` | Maintiens **3** + clic → jaune → **Base Color** ; Maintiens **1** + clic → valeur 1 → **Metallic** |
| `M_Wood` | **3** + clic → brun → **Base Color** |
| `M_Red` | **3** + clic → rouge → **Base Color** |

Clique **Save** à chaque fois.

## ÉTAPE 4 – Input Action pour la touche E

1. Clic droit dans `TP2/Input` → **Input → Input Action** → nom `IA_Interact`.
2. Ouvre-la : **Value Type = Digital (bool)**. Save.
3. Ouvre `Content/ThirdPerson/Input/IMC_Default`.
4. **Mappings** → clique **+** → choisis `IA_Interact`.
5. Clique l'icône clavier à côté → appuie sur **E**. Save.

## ÉTAPE 5 – Interface `BPI_Interactable`

1. Clic droit dans `TP2/Blueprints` → **Blueprint → Blueprint Interface** → `BPI_Interactable`.
2. Renomme la fonction existante (`NewFunction_0`) en **`Interact`**.
3. Dans **Details → Inputs** → **+** → nom `InstigatorPawn`, type **Pawn (Object Reference)**.
4. Compile, Save.

## ÉTAPE 6 – Player State `PS_Collector`

1. Clic droit → **Blueprint Class** → **All Classes** → cherche **PlayerState** → `PS_Collector`.
2. Panneau **My Blueprint** → **Variables** **+** :
   - `CoinCount` : Integer, valeur par défaut 0
   - `ChestCount` : Integer, valeur par défaut 0
3. **Event Dispatchers** **+** → `OnCollectiblesChanged`.
   Dans Details → **Inputs** : `Coins` (Integer), `Chests` (Integer).

### Fonction `AddCoin`
**Functions +** → `AddCoin`, puis relie :

```
[AddCoin]
  → SET CoinCount = (GET CoinCount + 1)          (nœud "Increment Int" possible)
  → Print String
       In String = Append("Pièces : ", CoinCount, " / 10")
       Duration = 3
  → Call OnCollectiblesChanged (Coins = CoinCount, Chests = ChestCount)
```

### Fonction `AddChest`
Pareil :

```
[AddChest]
  → SET ChestCount = ChestCount + 1
  → Print String  Append("Coffres : ", ChestCount, " / 3")  Duration = 3
  → Call OnCollectiblesChanged (Coins = CoinCount, Chests = ChestCount)
```

> Pour `Append`, clique **Add pin** pour avoir 3 entrées. Brancher un Integer
> dans une entrée String crée automatiquement la conversion.

Compile, Save.

## ÉTAPE 7 – Game Mode `GM_Collect`

1. Clic droit → **Blueprint Class** → **Game Mode Base** → `GM_Collect`.
2. **Class Defaults** :
   - Default Pawn Class = `BP_ThirdPersonCharacter`
   - Player State Class = `PS_Collector`
   - HUD Class = `BP_HUD` (à remplir après l'étape 13, bonus)
3. Variables : `RequiredCoins` (Integer = 10), `RequiredChests` (Integer = 3),
   `bLevelComplete` (Boolean = false).

### Event Graph

```
Event OnPostLogin (New Player)
  → New Player ▸ Get Player State
  → Cast To PS_Collector
  → Bind Event to On Collectibles Changed
        Event ◄── (tire le fil rouge "Event" → "Add Custom Event") nom : CheckVictory
```

```
CheckVictory (Coins, Chests)
  → Branch
       Condition = AND(
            Coins  >= RequiredCoins,
            Chests >= RequiredChests,
            NOT bLevelComplete )
  True →
     SET bLevelComplete = true
     → Print String
          "NIVEAU TERMINÉ ! 3 coffres et 10 pièces récupérés !"
          Duration = 15, Text Color = vert
     → Set Game Paused (Paused = true)
```

> Le **AND** est obligatoire : seulement les pièces ou seulement les coffres
> ne termine pas le niveau.

4. Dans le niveau : **Window → World Settings** → **GameMode Override = `GM_Collect`**.

## ÉTAPE 8 – Le personnage (touche E)

Ouvre `Content/ThirdPerson/Blueprints/BP_ThirdPersonCharacter` → Event Graph :

```
EnhancedInputAction IA_Interact   (sortie "Started")
  → Get Overlapping Actors (Class Filter = Actor)
  → For Each Loop with Break
       Loop Body →
          Does Object Implement Interface
               Test Object = Array Element
               Interface   = BPI_Interactable
          → Branch
               True → Interact (Message)   ← la version avec l'enveloppe
                         Target         = Array Element
                         InstigatorPawn = Self
                      → Break   (relie au pin "Break" du For Each)
```

Compile, Save.

## ÉTAPE 9 – Le coffre `BP_Chest`

1. Clic droit → **Blueprint Class** → **Actor** → `BP_Chest`.
2. **Class Settings** → **Interfaces → Add** → `BPI_Interactable`.

### Composants (bouton **+ Add**)

| Nom | Type | Parent | Location | Scale | Matériau |
|---|---|---|---|---|---|
| `Base` | Cube | DefaultSceneRoot | (0, 0, 25) | (1.0, 0.6, 0.5) | M_Wood |
| `LidPivot` | Scene | DefaultSceneRoot | (-50, 0, 50) | 1 | – |
| `Lid` | Cube | **LidPivot** | (50, 0, 7.5) | (1.0, 0.6, 0.15) | M_Wood |
| `InteractionZone` | Sphere Collision | DefaultSceneRoot | (0,0,0) | Sphere Radius = 200 | – |
| `PromptText` | Text Render | DefaultSceneRoot | (0, 0, 120) | – | Text = « [E] Ouvrir » |

- `PromptText` : **Visible = false** (décoché), Horizontal Alignment = Center.
- Le couvercle doit tourner autour de `LidPivot` (la charnière à l'arrière).

### Variable
`bIsOpened` : Boolean = false.

### Timeline
Clic droit → **Add Timeline** → `TL_OpenLid`. Double-clic dessus :
- Length = **1.0**
- **+ Track → Float Track** nommée `Alpha`
- Ajoute 2 points (Shift + clic) : (Time 0, Value 0) et (Time 1, Value 1)
- Clic droit sur les points → **Auto** (mouvement doux)

### Event Graph

```
Event Interact (InstigatorPawn)        ← clic droit "Event Interact"
  → Branch (Condition = bIsOpened)
       False →
          SET bIsOpened = true
          → PromptText ▸ Set Visibility (false)
          → TL_OpenLid  (pin "Play from Start")
          → InstigatorPawn ▸ Get Player State
          → Cast To PS_Collector
          → AddChest
```

```
TL_OpenLid  Update →
   LidPivot ▸ Set Relative Rotation
        New Rotation = Lerp (Rotator)
             A = (0, 0, 0)
             B = (0, -110, 0)     ← Pitch -110 (essaie +110 si ça s'ouvre à l'envers)
             Alpha = Alpha (de la timeline)
```

```
InteractionZone ▸ On Component Begin Overlap
  → Cast To BP_ThirdPersonCharacter (Other Actor)
  → Branch (NOT bIsOpened)
       True → PromptText ▸ Set Visibility (true)

InteractionZone ▸ On Component End Overlap
  → Cast To BP_ThirdPersonCharacter (Other Actor)
  → PromptText ▸ Set Visibility (false)
```

Compile, Save.

## ÉTAPE 10 – La pièce `BP_Coin`

1. **Blueprint Class → Actor** → `BP_Coin`.

| Nom | Type | Réglages |
|---|---|---|
| `Sphere` | Sphere Collision | Glisse-le sur DefaultSceneRoot pour qu'il devienne la racine. Radius = 60 |
| `Mesh` | Cylinder | Rotation X = 90, Scale (0.6, 0.6, 0.08), M_Gold, Collision Presets = **NoCollision** |
| `RotatingMovement` | Rotating Movement | Rotation Rate = (0, 0, 180) |

### Event Graph

```
Sphere ▸ On Component Begin Overlap (Other Actor)
  → Cast To Pawn (Object = Other Actor)
  → Is Player Controlled → Branch
       True →
          Do Once
          → (As Pawn) ▸ Get Player State
          → Cast To PS_Collector
          → AddCoin
          → Destroy Actor (Self)
```

Compile, Save.

## ÉTAPE 11 – La porte `BP_Door`

1. **Actor** → `BP_Door`.
2. Composant `Door` : Cube, Scale (0.3, 3, 4), Location Z = 200, M_Red.
3. Variable `StartLocation` (Vector).
4. Timeline `TL_Door` : Length 1.5, Float Track `Alpha` 0 → 1.

```
Event BeginPlay → SET StartLocation = Get Actor Location

Clic droit → Add Custom Event → "OpenDoor"
   (un Custom Event et non une fonction : les Timelines ne marchent pas
    dans une fonction)
   → TL_Door (Play)

TL_Door Update →
   Set Actor Location
       New Location = Lerp (Vector)
            A = StartLocation
            B = StartLocation + (0, 0, -420)
            Alpha = Alpha
```

Compile, Save.

## ÉTAPE 12 – Le levier `BP_Lever` (élément interactif)

1. **Actor** → `BP_Lever`. **Class Settings → Interfaces** → `BPI_Interactable`.

| Nom | Type | Parent | Réglages |
|---|---|---|---|
| `Base` | Cube | Root | Scale (0.5, 0.5, 0.2), M_Grey |
| `HandlePivot` | Scene | Root | Location Z = 20 |
| `Handle` | Cylinder | HandlePivot | Location Z = 50, Scale (0.1, 0.1, 1.0), M_Red |
| `InteractionZone` | Sphere Collision | Root | Radius 150 |
| `PromptText` | Text Render | Root | « [E] Activer », Z = 150, invisible |

2. Variables :
   - `TargetDoor` : type **BP_Door (Object Reference)**, coche l'**œil** (Instance Editable)
   - `bUsed` : Boolean
3. Timeline `TL_Lever` : 0.5 s, Alpha 0 → 1.

```
Event Interact
  → Branch (bUsed)
       False →
          SET bUsed = true
          → PromptText ▸ Set Visibility (false)
          → TL_Lever (Play from Start)
          → Is Valid (TargetDoor) → TargetDoor ▸ OpenDoor

TL_Lever Update →
   HandlePivot ▸ Set Relative Rotation
       Lerp (Rotator) A = (0, 45, 0)  B = (0, -45, 0)  Alpha
```

Ajoute les mêmes Begin/End Overlap que le coffre pour `PromptText`
(avec `NOT bUsed`).

## ÉTAPE 12b – Plateforme mobile `BP_MovingPlatform` (platforming)

1. **Actor** → `BP_MovingPlatform`.
2. Composant `Platform` : Cube, Scale (3, 3, 0.3), M_Grey.
3. Composant **InterpToMovement** :
   - Duration = 3
   - Behaviour Type = **Ping Pong**
   - Control Points : [0] = (0, 0, 0), [1] = (0, 0, 600)

Aucun code nécessaire.

## ÉTAPE 13 – BONUS : HUD

### `WBP_HUD`
1. Clic droit dans `TP2/UI` → **User Interface → Widget Blueprint** → `WBP_HUD`.
2. Designer :
   - **Canvas Panel**
   - **Vertical Box** ancrée en haut à gauche, contenant :
     - Text `TxtCoins` : « Pièces : 0 / 10 » (coche **Is Variable**)
     - Text `TxtChests` : « Coffres : 0 / 3 » (coche **Is Variable**)
   - Text `TxtPrompt` ancré en bas au centre : « Appuie sur E pour interagir »,
     **Is Variable**, Visibility = **Hidden**
3. Graph :

```
Event Construct
  → Get Owning Player ▸ Get Player State
  → Cast To PS_Collector
  → Bind Event to On Collectibles Changed  → Custom Event "UpdateCounters"

UpdateCounters (Coins, Chests)
  → TxtCoins  ▸ SetText( Format Text "Pièces : {0} / 10"  {0}=Coins )
  → TxtChests ▸ SetText( Format Text "Coffres : {0} / 3"  {0}=Chests )
```

Fonction `SetPromptVisible (bVisible : Boolean)` :
```
→ Select (Visible si true, Hidden si false) → TxtPrompt ▸ Set Visibility
```

### `BP_HUD`
1. **Blueprint Class → HUD** → `BP_HUD`.
2. Variable `HUDWidget` (type WBP_HUD).

```
Event BeginPlay
  → Create Widget (Class = WBP_HUD, Owning Player = Get Owning Player Controller)
  → SET HUDWidget
  → Add to Viewport
```

3. Retourne dans `GM_Collect` → **HUD Class = BP_HUD**.

4. (Optionnel) Dans les overlaps de `BP_Chest` et `BP_Lever`, ajoute :
```
Get Player Controller (0) ▸ Get HUD → Cast To BP_HUD → HUDWidget ▸ SetPromptVisible(true/false)
```

## ÉTAPE 14 – Construire le niveau

Glisse des **Cubes** (Place Actors → Shapes) et ajuste-les avec **R** (scale),
**W** (déplacer), **E** (rotation). Applique `M_Grey`.

```
                  [ TOUR (haut 600) ]  Coffre 3 + 2 pièces
                         ▲ plateforme mobile
                         │
 [ PARCOURS ] ←──── [ PLACE CENTRALE / SPAWN ] ────→ [ PORTE ROUGE ]
  5 cubes de           3 pièces + Levier                  │
  hauteurs                                           [ SALLE FERMÉE ]
  croissantes                                        Coffre 2 + 2 pièces
  Coffre 1 + 3 pièces
```

Placement :
- **Place centrale** : sol 3000 × 3000, Player Start, 3 × `BP_Coin`, `BP_Lever`
- **Salle fermée** : 4 murs + `BP_Door` dans l'ouverture, `BP_Chest` + 2 × `BP_Coin`.
  Sélectionne le levier → Details → **Target Door** → pipette → clique la porte.
- **Parcours** : 5 cubes espacés de ~300, chaque cube 100 plus haut,
  `BP_Chest` au sommet + 3 × `BP_Coin` sur les cubes
- **Tour** : bloc haut de 600, `BP_MovingPlatform` au pied,
  `BP_Chest` + 2 × `BP_Coin` en haut
- Murs autour de toute la zone pour ne pas tomber

**Total à vérifier : 10 × BP_Coin et 3 × BP_Chest.**

## ÉTAPE 15 – Tests

- [ ] Prendre seulement les 3 coffres → pas de fin
- [ ] Prendre seulement les 10 pièces → pas de fin
- [ ] Spammer E sur un coffre ouvert → compteur inchangé
- [ ] E loin d'un coffre → rien
- [ ] Le couvercle tourne réellement
- [ ] 3 coffres + 10 pièces → message de fin + pause

## ÉTAPE 16 – Remise sur GitHub

1. Copie `.gitignore`, `.gitattributes` et `README.md` de ce dossier à la
   racine de ton projet Unreal (là où se trouve `TP2_Collect.uproject`).
2. Installe Git LFS (https://git-lfs.com), puis dans le dossier du projet :

```
git init
git lfs install
git add .
git commit -m "TP2 - niveau de collecte"
git branch -M main
git remote add origin https://github.com/<ton-nom>/<ton-repo>.git
git push -u origin main
```
