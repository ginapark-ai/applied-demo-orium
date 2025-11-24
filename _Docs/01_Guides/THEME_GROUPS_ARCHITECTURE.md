# 🏗️ Theme Groups Architecture

How to structure tokens for independent Figma mode switching

## Core Concept: Theme Groups

A **Theme Group** is a set of alternate token values that switch as a unit.

```
Theme Group "Brand" has:
├── Theme "Default"    → Load 01_Brand/Default.json
├── Theme "Brand2"     → Load 01_Brand/Brand2.json

Theme Group "Theme" has:
├── Theme "Light"      → Load 03_Themes/Light.json
├── Theme "Dark"       → Load 03_Themes/Dark.json

Theme Group "Spacing" has:
├── Theme "Default"    → Load 02_Layout.json
├── Theme "Compact"    → Load 02_Layout.json + 04_Responsive/Compact.json
├── Theme "Spacious"   → Load 02_Layout.json + 04_Responsive/Spacious.json
```

## Design Principle

**Each theme group solves ONE problem:**

- **Brand** → Which brand's colors/typography?
- **Theme** → Light or dark?
- **Spacing** → How dense is the layout?

Users toggle each independently - no compound themes needed.

## Current Structure

### $themes.json Organization

```json
[
  // BRAND THEME GROUP - Switch brand colors & typography
  {
    "id": "brand-default",
    "name": "Default",
    "group": "Brand",
    "selectedTokenSets": {
      "_Base/Value": "enabled",
      "01_Brand/Default": "enabled",      // ← Brand-specific colors/type
      "02_Layout": "enabled",             // ← Shared spacing
      "05_Interactions/States": "enabled",
      "07_Components/Compositions": "enabled"
    }
  },
  {
    "id": "brand-brand2",
    "name": "Brand2",
    "group": "Brand",
    "selectedTokenSets": {
      "_Base/Value": "enabled",
      "01_Brand/Brand2": "enabled",       // ← Different brand colors/type
      "02_Layout": "enabled",             // ← Same spacing
      "05_Interactions/States": "enabled",
      "07_Components/Compositions": "enabled"
    }
  },

  // THEME THEME GROUP - Switch color theme
  {
    "id": "theme-light",
    "name": "Light",
    "group": "Theme",
    "selectedTokenSets": {
      "_Base/Value": "enabled",
      "03_Themes/Light": "enabled",       // ← Light colors only
      "05_Interactions/States": "enabled",
      "07_Components/Compositions": "enabled"
    }
  },
  {
    "id": "theme-dark",
    "name": "Dark",
    "group": "Theme",
    "selectedTokenSets": {
      "_Base/Value": "enabled",
      "03_Themes/Dark": "enabled",        // ← Dark colors only
      "05_Interactions/States": "enabled",
      "07_Components/Compositions": "enabled"
    }
  },

  // SPACING THEME GROUP - Switch layout density
  {
    "id": "spacing-default",
    "name": "Default",
    "group": "Spacing",
    "selectedTokenSets": {
      "_Base/Value": "enabled",
      "02_Layout": "enabled",             // ← Base spacing
      "05_Interactions/States": "enabled",
      "07_Components/Compositions": "enabled"
    }
  },
  {
    "id": "spacing-compact",
    "name": "Compact",
    "group": "Spacing",
    "selectedTokenSets": {
      "_Base/Value": "enabled",
      "02_Layout": "enabled",             // ← Load base first
      "04_Responsive/Compact": "enabled", // ← Override with compact (0.75x)
      "05_Interactions/States": "enabled",
      "07_Components/Compositions": "enabled"
    }
  },
  {
    "id": "spacing-spacious",
    "name": "Spacious",
    "group": "Spacing",
    "selectedTokenSets": {
      "_Base/Value": "enabled",
      "02_Layout": "enabled",             // ← Load base first
      "04_Responsive/Spacious": "enabled", // ← Override with spacious (1.25x)
      "05_Interactions/States": "enabled",
      "07_Components/Compositions": "enabled"
    }
  }
]
```

## Export to Figma

When synced, each `group` becomes a **Variable Collection**:

```
Figma File Variables:

┌─ Collection: Brand
│  ├─ Mode: Default
│  └─ Mode: Brand2
│
├─ Collection: Theme
│  ├─ Mode: Light
│  └─ Mode: Dark
│
└─ Collection: Spacing
   ├─ Mode: Default
   ├─ Mode: Compact
   └─ Mode: Spacious
```

## Token Resolution

### Load Order

When a theme is selected, token sets load in this order:

```
1. _Base/Value              (primitives - never changes)
2. 01_Brand/Default or Brand2   (brand colors/typography)
3. 02_Layout                (base spacing/radius)
4. 03_Themes/Light or Dark  (color overrides)
5. 04_Responsive/*          (spacing/radius overrides)
6. 05_Interactions/States   (state tokens)
7. 07_Components            (component compositions)
```

**"Last set wins" principle:** Token defined in set 5 overrides set 4, which overrides set 3, etc.

### Example: Compact + Dark + Brand2

```
Theme selections:
  • Spacing → "Compact"
  • Theme → "Dark"
  • Brand → "Brand2"

Load order:
1. _Base/Value
2. 01_Brand/Brand2         ← Brand2 colors + typography
3. 02_Layout               ← Base spacing-8 = 8px
4. 03_Themes/Dark          ← Dark color overrides
5. 04_Responsive/Compact   ← Compact spacing-8 = 7px (WINS!)
6. 05_Interactions/States
7. 07_Components

Result:
  color-primary = Brand2 dark color
  spacing-8 = 7px
```

## Adding New Variants

### Scenario 1: Add Brand3

**Step 1: Create brand file**
```json
// Tokens/New/01_Brand/Brand3.json
{
  "color": { "brandPrimary": { ... } },
  "typography": { "fontFamily": { ... } }
}
```

**Step 2: Add theme**
```json
// Add to $themes.json
{
  "id": "brand-brand3",
  "name": "Brand3",
  "group": "Brand",
  "selectedTokenSets": {
    "01_Brand/Brand3": "enabled",
    // ... rest of sets
  }
}
```

**Step 3: Sync to Figma**

Result: Brand collection now has 3 modes (Default, Brand2, Brand3)

### Scenario 2: Brand2-specific Responsive Typography

If Brand2 needs different font sizes at Compact spacing:

**Step 1: Create responsive typography**
```json
// Tokens/New/04_Responsive/CompactType.json
{
  "typography": {
    "heading": {
      "fontSize": { "value": "{fontSize.24 * 0.85}", "type": "dimension" }
    }
  }
}
```

**Step 2: Update spacing theme**
```json
{
  "id": "spacing-compact",
  "name": "Compact",
  "group": "Spacing",
  "selectedTokenSets": {
    "_Base/Value": "enabled",
    "02_Layout": "enabled",
    "04_Responsive/Compact": "enabled",
    "04_Responsive/CompactType": "enabled",  // ← Add this
    "05_Interactions/States": "enabled",
    "07_Components/Compositions": "enabled"
  }
}
```

**Step 3: Sync to Figma**

Result: When Spacing → Compact, typography sizes also adjust!

### Scenario 3: Add 4th Spacing Mode (UltraDense)

**Step 1: Create spacing file**
```json
// Tokens/New/04_Responsive/UltraDense.json
{
  "spacing": { "spacing-8": { "value": "{spacing.8 * 0.5}", "type": "dimension" } }
}
```

**Step 2: Add theme**
```json
{
  "id": "spacing-ultradense",
  "name": "UltraDense",
  "group": "Spacing",
  "selectedTokenSets": {
    "_Base/Value": "enabled",
    "02_Layout": "enabled",
    "04_Responsive/UltraDense": "enabled",
    "05_Interactions/States": "enabled",
    "07_Components/Compositions": "enabled"
  }
}
```

**Step 3: Sync**

Result: Spacing collection now has 4 modes!

## Developer Export

Using @tokens-studio/sd-transforms:

```bash
# Export each collection as separate file
output/
├── Brand/
│   ├── default.json
│   ├── brand2.json
│   └── brand3.json
├── Theme/
│   ├── light.json
│   └── dark.json
└── Spacing/
    ├── default.json
    ├── compact.json
    └── spacious.json
```

Developers use CSS custom properties:

```css
/* Load all base variables */
:root {
  --brand-primary: var(--Brand-default-brandPrimary);
  --color-background: var(--Theme-light-background);
  --spacing-8: var(--Spacing-default-spacing8);
}

/* Toggle brand with class */
.brand2 {
  --brand-primary: var(--Brand-brand2-brandPrimary);
}

/* Toggle theme */
@media (prefers-color-scheme: dark) {
  :root {
    --color-background: var(--Theme-dark-background);
  }
}

/* Toggle spacing */
body.compact {
  --spacing-8: var(--Spacing-compact-spacing8);
}
```

## Best Practices

1. **Keep groups independent** - Each group solves one problem
2. **Use descriptive names** - `"Default"`, `"Brand2"`, `"Dark"` are clear
3. **Load shared sets in all themes** - `_Base/Value`, `05_Interactions/States`, etc.
4. **Override in order** - Load base first, then overrides
5. **Document per theme** - Add descriptions to each theme object
6. **Test combinations** - Toggle each mode to verify tokens resolve correctly

## Common Pitfalls

❌ **Too many tokens in one set** → Hard to override
```json
// BAD: All colors in one file
"03_Color.json": { 100+ color tokens }
```

✅ **Organize by purpose** → Easy to swap
```json
// GOOD: Separate by brand/theme
"01_Brand/Default.json"
"01_Brand/Brand2.json"
"03_Themes/Light.json"
"03_Themes/Dark.json"
```

---

❌ **Loading unrelated sets** → Confusing dependencies
```json
// BAD: Why does Brand theme load Interactions?
"selectedTokenSets": {
  "01_Brand/Default": "enabled",
  "05_Interactions/States": "enabled"  // ← Unrelated
}
```

✅ **Load shared foundations** → Clear separation of concerns
```json
// GOOD: All themes load shared sets
"selectedTokenSets": {
  "_Base/Value": "enabled",           // ← Shared by all
  "01_Brand/Default": "enabled",      // ← Brand-specific
  "05_Interactions/States": "enabled" // ← Shared by all
}
```

## Summary

- **Theme Groups** = Independent switching dimensions
- **Each group** = One collection in Figma
- **Each mode** = One "way" to style that dimension
- **Token resolution** = Last set wins, load order matters
- **Scaling** = Add new theme to group, sync, done!

No 12 combined themes. No complex permutations. Just **independent mode toggles**.

