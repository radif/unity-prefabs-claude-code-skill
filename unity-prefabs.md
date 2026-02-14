# Unity Prefab YAML Building Guide

A universal reference for building and modifying Unity prefab `.prefab` files directly in YAML. Covers document structure, component wiring, nested prefabs, UI recipes, and a modification checklist. Works with any Unity 6+ project using Claude Code.

---

## 1. Prefab YAML Fundamentals

### Document Structure

Every prefab is a multi-document YAML file. Each document starts with `--- !u!<type_id> &<file_id>` and represents one entity:

| Type ID | Entity | Description |
|---|---|---|
| `!u!1` | GameObject | Container with component list, name, layer, active state |
| `!u!4` | Transform | 3D transform (position, rotation, scale, parent-child) |
| `!u!224` | RectTransform | UI transform (anchors, pivot, size delta) — extends Transform |
| `!u!222` | CanvasRenderer | Required for any visible UI element |
| `!u!114` | MonoBehaviour | Any script component (Image, Button, custom scripts, etc.) |
| `!u!1001` | PrefabInstance | Nested prefab reference with property overrides |

### GameObject + Components Pattern

Every element is a GameObject listing its components, plus separate documents for each component:

```yaml
--- !u!1 &<GO_ID>
GameObject:
  m_ObjectHideFlags: 0
  m_CorrespondingSourceObject: {fileID: 0}
  m_PrefabInstance: {fileID: 0}
  m_PrefabAsset: {fileID: 0}
  serializedVersion: 6
  m_Component:
  - component: {fileID: <TRANSFORM_ID>}
  - component: {fileID: <COMPONENT_ID>}
  m_Layer: 0
  m_Name: MyElement
  m_TagString: Untagged
  m_Icon: {fileID: 0}
  m_NavMeshLayer: 0
  m_StaticEditorFlags: 0
  m_IsActive: 1
```

### Parent-Child Wiring

Bidirectional — parent lists children, each child points back:

```yaml
# Parent's Transform/RectTransform
m_Children:
- {fileID: <CHILD1_TRANSFORM_ID>}
- {fileID: <CHILD2_TRANSFORM_ID>}

# Child's Transform/RectTransform
m_Father: {fileID: <PARENT_TRANSFORM_ID>}
```

### FileID Rules

- Use unique large integers (15–19 digits)
- Check existing fileIDs in the prefab to avoid collisions
- YAML blocks must be inserted in **fileID-sorted order** within the prefab file

---

## 2. 3D Transform vs RectTransform

**3D elements** (characters, props, game world objects) use Transform (`!u!4`). **UI elements** use RectTransform (`!u!224`), which extends Transform with anchors, pivot, and size delta. Choose based on the element type — never mix them.

---

## 3. Component GUIDs — Unity Built-in

These GUIDs are stable across all Unity projects:

```
Image:              {fileID: 11500000, guid: fe87c0e1cc204ed48ad3b37840f39efc, type: 3}
TextMeshProUGUI:    {fileID: 11500000, guid: f4688fdb7df04437aeb418b961361dc5, type: 3}
Button:             {fileID: 11500000, guid: 4e29b1a8efbd4b44bb3f3716e73f07ff, type: 3}
TMP_Dropdown:       {fileID: 11500000, guid: 7b743370ac3e4ec2a1668f5455a8ef8a, type: 3}
Toggle:             {fileID: 11500000, guid: 9085046f02f69544eb97fd06b6048fe2, type: 3}
ScrollRect:         {fileID: 11500000, guid: 1aa08ab6e0800fa44ae55d278d1423e3, type: 3}
Scrollbar:          {fileID: 11500000, guid: 2a4db7a114972834c8e4117be1d82ba3, type: 3}
Mask:               {fileID: 11500000, guid: 31a19414c41e5ae4aae2af33fee712f6, type: 3}
```

> **Project-specific components:** Add your own script GUIDs here. Find them in the `.meta` file next to your script (e.g., `MyScript.cs.meta` → `guid: abcdef1234567890...`).

---

## 4. Standard Unity Sprites

Built-in sprites (guid: `0000000000000000f000000000000000`):

```
10901:  Checkmark
10905:  UISprite (Background, sliced)
10907:  InputFieldBackground
10915:  DropdownArrow
10917:  UIMask
```

Usage in YAML:
```yaml
m_Sprite: {fileID: 10905, guid: 0000000000000000f000000000000000, type: 0}
```

---

## 5. Font Reference

Replace the placeholder GUID with your project's TMP font asset (found in the `.meta` file next to the font `.asset`):

```yaml
m_fontAsset:      {fileID: 11400000, guid: <YOUR_FONT_ASSET_GUID>, type: 2}
m_sharedMaterial: {fileID: 2180264, guid: <YOUR_FONT_ASSET_GUID>, type: 2}
```

---

## 6. Standard Selectable Colors Block

Buttons, Toggles, Dropdowns, and Scrollbars all share this color transition block:

```yaml
m_Navigation:
  m_Mode: 3
  m_WrapAround: 0
  m_SelectOnUp: {fileID: 0}
  m_SelectOnDown: {fileID: 0}
  m_SelectOnLeft: {fileID: 0}
  m_SelectOnRight: {fileID: 0}
m_Transition: 1
m_Colors:
  m_NormalColor: {r: 1, g: 1, b: 1, a: 1}
  m_HighlightedColor: {r: 0.9607843, g: 0.9607843, b: 0.9607843, a: 1}
  m_PressedColor: {r: 0.78431374, g: 0.78431374, b: 0.78431374, a: 1}
  m_SelectedColor: {r: 0.9607843, g: 0.9607843, b: 0.9607843, a: 1}
  m_DisabledColor: {r: 0.78431374, g: 0.78431374, b: 0.78431374, a: 0.5019608}
  m_ColorMultiplier: 1
  m_FadeDuration: 0.1
m_SpriteState:
  m_HighlightedSprite: {fileID: 0}
  m_PressedSprite: {fileID: 0}
  m_SelectedSprite: {fileID: 0}
  m_DisabledSprite: {fileID: 0}
m_AnimationTriggers:
  m_NormalTrigger: Normal
  m_HighlightedTrigger: Highlighted
  m_PressedTrigger: Pressed
  m_SelectedTrigger: Selected
  m_DisabledTrigger: Disabled
m_Interactable: 1
```

---

## 7. Component Templates

### Minimal UI GameObject (RectTransform)

```yaml
--- !u!1 &<GO_ID>
GameObject:
  m_ObjectHideFlags: 0
  m_CorrespondingSourceObject: {fileID: 0}
  m_PrefabInstance: {fileID: 0}
  m_PrefabAsset: {fileID: 0}
  serializedVersion: 6
  m_Component:
  - component: {fileID: <RT_ID>}
  - component: {fileID: <CR_ID>}
  - component: {fileID: <COMP_ID>}
  m_Layer: 0
  m_Name: MyElement
  m_TagString: Untagged
  m_Icon: {fileID: 0}
  m_NavMeshLayer: 0
  m_StaticEditorFlags: 0
  m_IsActive: 1
--- !u!224 &<RT_ID>
RectTransform:
  m_ObjectHideFlags: 0
  m_CorrespondingSourceObject: {fileID: 0}
  m_PrefabInstance: {fileID: 0}
  m_PrefabAsset: {fileID: 0}
  m_GameObject: {fileID: <GO_ID>}
  m_LocalRotation: {x: 0, y: 0, z: 0, w: 1}
  m_LocalPosition: {x: 0, y: 0, z: 0}
  m_LocalScale: {x: 1, y: 1, z: 1}
  m_ConstrainProportionsScale: 0
  m_Children: []
  m_Father: {fileID: <PARENT_RT_ID>}
  m_LocalEulerAnglesHint: {x: 0, y: 0, z: 0}
  m_AnchorMin: {x: 0, y: 0}
  m_AnchorMax: {x: 1, y: 1}
  m_AnchoredPosition: {x: 0, y: 0}
  m_SizeDelta: {x: 0, y: 0}
  m_Pivot: {x: 0.5, y: 0.5}
--- !u!222 &<CR_ID>
CanvasRenderer:
  m_ObjectHideFlags: 0
  m_CorrespondingSourceObject: {fileID: 0}
  m_PrefabInstance: {fileID: 0}
  m_PrefabAsset: {fileID: 0}
  m_GameObject: {fileID: <GO_ID>}
  m_CullTransparentMesh: 1
```

### Minimal 3D GameObject (Transform)

```yaml
--- !u!1 &<GO_ID>
GameObject:
  m_ObjectHideFlags: 0
  m_CorrespondingSourceObject: {fileID: 0}
  m_PrefabInstance: {fileID: 0}
  m_PrefabAsset: {fileID: 0}
  serializedVersion: 6
  m_Component:
  - component: {fileID: <T_ID>}
  m_Layer: 0
  m_Name: MyObject
  m_TagString: Untagged
  m_Icon: {fileID: 0}
  m_NavMeshLayer: 0
  m_StaticEditorFlags: 0
  m_IsActive: 1
--- !u!4 &<T_ID>
Transform:
  m_ObjectHideFlags: 0
  m_CorrespondingSourceObject: {fileID: 0}
  m_PrefabInstance: {fileID: 0}
  m_PrefabAsset: {fileID: 0}
  m_GameObject: {fileID: <GO_ID>}
  m_LocalRotation: {x: 0, y: 0, z: 0, w: 1}
  m_LocalPosition: {x: 0, y: 0, z: 0}
  m_LocalScale: {x: 1, y: 1, z: 1}
  m_ConstrainProportionsScale: 0
  m_Children: []
  m_Father: {fileID: <PARENT_T_ID>}
```

### Image Component

```yaml
--- !u!114 &<IMG_ID>
MonoBehaviour:
  m_ObjectHideFlags: 0
  m_CorrespondingSourceObject: {fileID: 0}
  m_PrefabInstance: {fileID: 0}
  m_PrefabAsset: {fileID: 0}
  m_GameObject: {fileID: <GO_ID>}
  m_Enabled: 1
  m_EditorHideFlags: 0
  m_Script: {fileID: 11500000, guid: fe87c0e1cc204ed48ad3b37840f39efc, type: 3}
  m_Name:
  m_EditorClassIdentifier:
  m_Material: {fileID: 0}
  m_Color: {r: 1, g: 1, b: 1, a: 1}
  m_RaycastTarget: 1
  m_RaycastPadding: {x: 0, y: 0, z: 0, w: 0}
  m_Maskable: 1
  m_OnCullStateChanged:
    m_PersistentCalls:
      m_Calls: []
  m_Sprite: {fileID: 10905, guid: 0000000000000000f000000000000000, type: 0}
  m_Type: 1
  m_PreserveAspect: 0
  m_FillCenter: 1
  m_FillMethod: 4
  m_FillAmount: 1
  m_FillClockwise: 1
  m_FillOrigin: 0
  m_UseSpriteMesh: 0
  m_PixelsPerUnitMultiplier: 1
```

### TextMeshProUGUI Component

```yaml
--- !u!114 &<TMP_ID>
MonoBehaviour:
  m_ObjectHideFlags: 0
  m_CorrespondingSourceObject: {fileID: 0}
  m_PrefabInstance: {fileID: 0}
  m_PrefabAsset: {fileID: 0}
  m_GameObject: {fileID: <GO_ID>}
  m_Enabled: 1
  m_EditorHideFlags: 0
  m_Script: {fileID: 11500000, guid: f4688fdb7df04437aeb418b961361dc5, type: 3}
  m_Name:
  m_EditorClassIdentifier:
  m_Material: {fileID: 0}
  m_Color: {r: 1, g: 1, b: 1, a: 1}
  m_RaycastTarget: 1
  m_RaycastPadding: {x: 0, y: 0, z: 0, w: 0}
  m_Maskable: 1
  m_OnCullStateChanged:
    m_PersistentCalls:
      m_Calls: []
  m_text: Label
  m_isRightToLeft: 0
  m_fontAsset: {fileID: 11400000, guid: <YOUR_FONT_ASSET_GUID>, type: 2}
  m_sharedMaterial: {fileID: 2180264, guid: <YOUR_FONT_ASSET_GUID>, type: 2}
  m_fontSharedMaterials: []
  m_fontMaterial: {fileID: 0}
  m_fontMaterials: []
  m_fontColor32:
    serializedVersion: 2
    rgba: 4294967295
  m_fontColor: {r: 1, g: 1, b: 1, a: 1}
  m_enableVertexGradient: 0
  m_colorMode: 3
  m_fontColorGradient:
    topLeft: {r: 1, g: 1, b: 1, a: 1}
    topRight: {r: 1, g: 1, b: 1, a: 1}
    bottomLeft: {r: 1, g: 1, b: 1, a: 1}
    bottomRight: {r: 1, g: 1, b: 1, a: 1}
  m_fontColorGradientPreset: {fileID: 0}
  m_spriteAsset: {fileID: 0}
  m_tintAllSprites: 0
  m_StyleSheet: {fileID: 0}
  m_TextStyleHashCode: -1183493901
  m_overrideHtmlColors: 0
  m_faceColor:
    serializedVersion: 2
    rgba: 4294967295
  m_fontSize: 40
  m_fontSizeBase: 40
  m_fontWeight: 400
  m_enableAutoSizing: 0
  m_fontSizeMin: 18
  m_fontSizeMax: 72
  m_fontStyle: 0
  m_HorizontalAlignment: 2
  m_VerticalAlignment: 512
  m_textAlignment: 65535
  m_characterSpacing: 0
  m_wordSpacing: 0
  m_lineSpacing: 0
  m_lineSpacingMax: 0
  m_paragraphSpacing: 0
  m_charWidthMaxAdj: 0
  m_TextWrappingMode: 1
  m_wordWrappingRatios: 0.4
  m_overflowMode: 0
  m_linkedTextComponent: {fileID: 0}
  parentLinkedComponent: {fileID: 0}
  m_enableKerning: 0
  m_ActiveFontFeatures: 6e72656b
  m_enableExtraPadding: 0
  checkPaddingRequired: 0
  m_isRichText: 1
  m_EmojiFallbackSupport: 1
  m_parseCtrlCharacters: 1
  m_isOrthographic: 1
  m_isCullingEnabled: 0
  m_horizontalMapping: 0
  m_verticalMapping: 0
  m_uvLineOffset: 0
  m_geometrySortingOrder: 0
  m_IsTextObjectScaleStatic: 0
  m_VertexBufferAutoSizeReduction: 0
  m_useMaxVisibleDescender: 1
  m_pageToDisplay: 1
  m_margin: {x: 0, y: 0, z: 0, w: 0}
  m_isUsingLegacyAnimationComponent: 0
  m_isVolumetricText: 0
  m_hasFontAssetChanged: 0
  m_baseMaterial: {fileID: 0}
  m_maskOffset: {x: 0, y: 0, z: 0, w: 0}
```

---

## 8. Nested Prefab References (PrefabInstance + Stripped Entries)

When a prefab contains another prefab as a child (e.g., a UI cell containing a shared button), Unity serializes it as a **PrefabInstance** block (`!u!1001`) plus **stripped entries** — NOT as inline GameObjects/components.

### PrefabInstance Block

```yaml
--- !u!1001 &<PREFAB_INSTANCE_ID>
PrefabInstance:
  m_ObjectHideFlags: 0
  serializedVersion: 2
  m_Modification:
    serializedVersion: 3
    m_TransformParent: {fileID: <PARENT_TRANSFORM_ID>}
    m_Modifications:
    - target: {fileID: <SOURCE_COMPONENT_ID>, guid: <SOURCE_PREFAB_GUID>, type: 3}
      propertyPath: m_Name
      value: PlayButton
      objectReference: {fileID: 0}
    # ... more property overrides ...
    m_RemovedComponents: []
    m_RemovedGameObjects: []
    m_AddedGameObjects: []
    m_AddedComponents: []
  m_SourcePrefab: {fileID: 100100000, guid: <SOURCE_PREFAB_GUID>, type: 3}
```

**Key points:**
- `m_TransformParent` — wires the nested prefab's root into the outer hierarchy
- `m_Modifications` — each entry overrides a single property on a source component
- `m_SourcePrefab` — always `{fileID: 100100000, guid: <GUID>, type: 3}`
- Components inside the nested prefab are NOT expanded inline

### Stripped Entries

To reference a component from a nested PrefabInstance in SerializeFields or event targets, create a **stripped entry** — a minimal YAML document with the `stripped` keyword that creates a local fileID alias:

```yaml
# Stripped GameObject
--- !u!1 &<LOCAL_FILEID> stripped
GameObject:
  m_CorrespondingSourceObject: {fileID: <SOURCE_GO_ID>, guid: <SOURCE_GUID>, type: 3}
  m_PrefabInstance: {fileID: <PREFAB_INSTANCE_ID>}
  m_PrefabAsset: {fileID: 0}

# Stripped RectTransform
--- !u!224 &<LOCAL_FILEID> stripped
RectTransform:
  m_CorrespondingSourceObject: {fileID: <SOURCE_RT_ID>, guid: <SOURCE_GUID>, type: 3}
  m_PrefabInstance: {fileID: <PREFAB_INSTANCE_ID>}
  m_PrefabAsset: {fileID: 0}

# Stripped Transform (for 3D prefabs)
--- !u!4 &<LOCAL_FILEID> stripped
Transform:
  m_CorrespondingSourceObject: {fileID: <SOURCE_T_ID>, guid: <SOURCE_GUID>, type: 3}
  m_PrefabInstance: {fileID: <PREFAB_INSTANCE_ID>}
  m_PrefabAsset: {fileID: 0}

# Stripped MonoBehaviour (e.g., Button component)
--- !u!114 &<LOCAL_FILEID> stripped
MonoBehaviour:
  m_CorrespondingSourceObject: {fileID: <SOURCE_COMP_ID>, guid: <SOURCE_GUID>, type: 3}
  m_PrefabInstance: {fileID: <PREFAB_INSTANCE_ID>}
  m_PrefabAsset: {fileID: 0}
  m_GameObject: {fileID: <STRIPPED_GO_ID>}
  m_Enabled: 1
  m_EditorHideFlags: 0
  m_Script: {fileID: 11500000, guid: <SCRIPT_GUID>, type: 3}
  m_Name:
  m_EditorClassIdentifier:
```

**Key points:**
- The `stripped` keyword on the `---` line is mandatory
- `m_CorrespondingSourceObject` points to the original component's fileID inside the source prefab
- `m_PrefabInstance` points to the `!u!1001` PrefabInstance block
- The local fileID is what you use in SerializeField references
- MonoBehaviour stripped entries include `m_GameObject`, `m_Script`; Transform/RectTransform stripped entries are minimal (3 fields)

### Wiring SerializeFields to Nested Prefab Components

Point the serialized field at the **stripped entry's** local fileID:

```yaml
# Outer prefab's script component
--- !u!114 &<SCRIPT_COMP_ID>
MonoBehaviour:
  playButton: {fileID: <STRIPPED_BUTTON_LOCAL_ID>}   # points to stripped Button entry
```

---

## 9. Event Wiring Convention

UI callbacks are wired as **persistent UnityEvents in the prefab**, not programmatically via `AddListener()`. Callback methods should be:
- **`public`** so they appear in the Inspector's function picker
- **Parameterless** — read the component's current value directly (e.g., `dropdown.value`)
- Registered with `m_Mode: 1` (void) in the persistent call

```yaml
m_OnValueChanged:  # or m_OnClick for buttons
  m_PersistentCalls:
    m_Calls:
    - m_Target: {fileID: <SCRIPT_COMPONENT_ID>}
      m_TargetAssemblyTypeName: ClassName, Assembly-CSharp
      m_MethodName: OnMethodName
      m_Mode: 1
      m_Arguments:
        m_ObjectArgument: {fileID: 0}
        m_ObjectArgumentAssemblyTypeName: UnityEngine.Object, UnityEngine
        m_IntArgument: 0
        m_FloatArgument: 0
        m_StringArgument:
        m_BoolArgument: 0
      m_CallState: 2
```

---

## 10. Nested Prefab Event Callback Retargeting

### Two-Layer Button Pattern

Many projects use shared button prefabs that have two click-handling components:
1. **Unity Button** (guid `4e29b1a8efbd4b44bb3f3716e73f07ff`) — built-in `m_OnClick`
2. **Custom button wrapper** (your project's script) — wraps audio/haptics and provides its own `onClick` UnityEvent

The custom wrapper's `onClick` is the primary handler. Clear the Unity Button's `m_OnClick` (size=0) and configure the wrapper's `onClick`:

```yaml
# In PrefabInstance m_Modifications:

# Step 1: Clear Unity Button's m_OnClick
- target: {fileID: <SOURCE_BUTTON_ID>, guid: <SOURCE_GUID>, type: 3}
  propertyPath: m_OnClick.m_PersistentCalls.m_Calls.Array.size
  value: 0
  objectReference: {fileID: 0}

# Step 2: Configure custom wrapper's onClick
- target: {fileID: <SOURCE_WRAPPER_ID>, guid: <SOURCE_GUID>, type: 3}
  propertyPath: onClick.m_PersistentCalls.m_Calls.Array.size
  value: 1
  objectReference: {fileID: 0}
- target: {fileID: <SOURCE_WRAPPER_ID>, guid: <SOURCE_GUID>, type: 3}
  propertyPath: onClick.m_PersistentCalls.m_Calls.Array.data[0].m_Target
  value:
  objectReference: {fileID: <OUTER_SCRIPT_COMPONENT_ID>}
- target: {fileID: <SOURCE_WRAPPER_ID>, guid: <SOURCE_GUID>, type: 3}
  propertyPath: onClick.m_PersistentCalls.m_Calls.Array.data[0].m_MethodName
  value: OnButtonTapped
  objectReference: {fileID: 0}
- target: {fileID: <SOURCE_WRAPPER_ID>, guid: <SOURCE_GUID>, type: 3}
  propertyPath: onClick.m_PersistentCalls.m_Calls.Array.data[0].m_TargetAssemblyTypeName
  value: MyClass, Assembly-CSharp
  objectReference: {fileID: 0}
- target: {fileID: <SOURCE_WRAPPER_ID>, guid: <SOURCE_GUID>, type: 3}
  propertyPath: onClick.m_PersistentCalls.m_Calls.Array.data[0].m_Mode
  value: 1
  objectReference: {fileID: 0}
- target: {fileID: <SOURCE_WRAPPER_ID>, guid: <SOURCE_GUID>, type: 3}
  propertyPath: onClick.m_PersistentCalls.m_Calls.Array.data[0].m_CallState
  value: 2
  objectReference: {fileID: 0}
- target: {fileID: <SOURCE_WRAPPER_ID>, guid: <SOURCE_GUID>, type: 3}
  propertyPath: onClick.m_PersistentCalls.m_Calls.Array.data[0].m_Arguments.m_ObjectArgumentAssemblyTypeName
  value: UnityEngine.Object, UnityEngine
  objectReference: {fileID: 0}
```

**Critical details:**
- `m_Target` uses `objectReference` (not `value`) to point at the outer prefab's script component
- All other fields use `value` (not `objectReference`)
- Unity Button uses `m_OnClick.m_PersistentCalls...`, a custom wrapper typically uses `onClick.m_PersistentCalls...` (no `m_` prefix on the field name) — check your wrapper's serialized field name

---

## 11. Adding Components to Nested Prefabs

To add a new component to a nested prefab's GameObject, use `m_AddedComponents`:

```yaml
# In PrefabInstance:
m_AddedComponents:
- targetCorrespondingSourceObject: {fileID: <SOURCE_GO_ID>, guid: <SOURCE_GUID>, type: 3}
  insertIndex: -1
  addedObject: {fileID: <NEW_COMPONENT_ID>}

# Normal (non-stripped) component document:
--- !u!114 &<NEW_COMPONENT_ID>
MonoBehaviour:
  m_ObjectHideFlags: 0
  m_CorrespondingSourceObject: {fileID: 0}
  m_PrefabInstance: {fileID: 0}
  m_PrefabAsset: {fileID: 0}
  m_GameObject: {fileID: <STRIPPED_GO_ID>}
  m_Enabled: 1
  m_EditorHideFlags: 0
  m_Script: {fileID: 11500000, guid: <SCRIPT_GUID>, type: 3}
  # ... component fields ...
```

---

## 12. TMP_Dropdown Hierarchy Recipe

A TMP_Dropdown requires **12 GameObjects** in this hierarchy:

```
Dropdown (GO + RectTransform + CanvasRenderer + Image + TMP_Dropdown)
├── Label (GO + RectTransform + CanvasRenderer + TextMeshProUGUI)      ← m_CaptionText
├── Arrow (GO + RectTransform + CanvasRenderer + Image[DropdownArrow])
└── Template (GO + RectTransform + CanvasRenderer + Image + ScrollRect) ← m_Template, m_IsActive: 0
    ├── Viewport (GO + RectTransform + CanvasRenderer + Mask + Image[UIMask])
    │   └── Content (GO + RectTransform only)
    │       └── Item (GO + RectTransform + Toggle)
    │           ├── Item Background (GO + RectTransform + CanvasRenderer + Image[none]) ← Toggle.m_TargetGraphic
    │           ├── Item Checkmark (GO + RectTransform + CanvasRenderer + Image[Checkmark]) ← Toggle.graphic
    │           └── Item Label (GO + RectTransform + CanvasRenderer + TextMeshProUGUI) ← m_ItemText
    └── Scrollbar (GO + RectTransform + CanvasRenderer + Image + Scrollbar)
        └── Sliding Area (GO + RectTransform only)
            └── Handle (GO + RectTransform + CanvasRenderer + Image) ← Scrollbar.m_HandleRect
```

### Key Wiring

**TMP_Dropdown component:**
- `m_TargetGraphic` → Dropdown's own Image
- `m_Template` → Template's RectTransform
- `m_CaptionText` → Label's TextMeshProUGUI
- `m_ItemText` → Item Label's TextMeshProUGUI

**ScrollRect (on Template):**
- `m_Content` → Content's RectTransform
- `m_Viewport` → Viewport's RectTransform
- `m_VerticalScrollbar` → Scrollbar's Scrollbar component

**Scrollbar:**
- `m_TargetGraphic` → Handle's Image
- `m_HandleRect` → Handle's RectTransform
- `m_Direction: 2` (Bottom-to-Top)

**Toggle (on Item):**
- `m_TargetGraphic` → Item Background's Image
- `graphic` → Item Checkmark's Image

**Template specifics:**
- `m_IsActive: 0` (starts hidden)
- Dropdown root Image: `m_Color` alpha 0 for transparent background, or 1 for visible
- Viewport Image: uses UIMask sprite (10917)
- Arrow Image: uses DropdownArrow sprite (10915)
- Item Checkmark Image: uses Checkmark sprite (10901)

> **Canvas sorting for dropdown overlays:** If the dropdown template must render above its parent canvas, add a canvas-order-sorting component to the Template GameObject. The sorting order value should exceed the parent canvas order. This is project-specific — use your own sorting script or Unity's built-in Canvas `overrideSorting` + `sortingOrder`.

---

## 13. Canvas Sorting for Overlays

Dropdown templates and popup elements that must render above the parent canvas need a sorting override. Two approaches:

### A. Unity Built-in (Canvas component override)

Add a Canvas component to the overlay GameObject with `overrideSorting: 1` and a high `sortingOrder`.

### B. Custom Sorting Script

If your project has a canvas-order-sorting script, add it as a MonoBehaviour:

```yaml
--- !u!114 &<SORTER_ID>
MonoBehaviour:
  m_ObjectHideFlags: 0
  m_CorrespondingSourceObject: {fileID: 0}
  m_PrefabInstance: {fileID: 0}
  m_PrefabAsset: {fileID: 0}
  m_GameObject: {fileID: <TEMPLATE_GO_ID>}
  m_Enabled: 1
  m_EditorHideFlags: 0
  m_Script: {fileID: 11500000, guid: <YOUR_SORTING_SCRIPT_GUID>, type: 3}
  m_Name:
  m_EditorClassIdentifier:
  sortingOrderToOverride: 32531
```

> Replace `<YOUR_SORTING_SCRIPT_GUID>` with the GUID from your sorting script's `.meta` file. Adjust `sortingOrderToOverride` or field name to match your implementation.

---

## 14. Modification Checklist

When modifying any prefab, verify all 10 steps:

1. **Update parent's `m_Children` list** when adding/removing child GameObjects
2. **Set `m_Father`** on every new child's Transform/RectTransform
3. **Wire component cross-references** (m_TargetGraphic, m_Template, m_CaptionText, etc.)
4. **Update serialized fields** on the controlling script (e.g., `playButton: {fileID: <ID>}`)
5. **Insert YAML blocks in fileID-sorted order** within the prefab file
6. **Use unique fileIDs** — check existing IDs in the prefab before choosing new ones
7. **Create stripped entries** for any nested prefab component referenced by SerializeFields or events
8. **Retarget nested button callbacks via PropertyModifications** — override in `m_Modifications`, don't edit source prefab
9. **Handle both Button and custom wrapper layers** when retargeting nested buttons — clear `m_OnClick` (size=0), configure wrapper `onClick`
10. **Use `objectReference`** (not `value`) for `m_Target` in event callback PropertyModifications
