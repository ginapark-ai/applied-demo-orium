# 🎨 Figma Modes Guide - VehicleOS Design System

Using Tokens Studio **Theme Switching** with Figma Variables

> Based on [Tokens Studio Theme Switching Guide](https://docs.tokens.studio/manage-themes/simple-switch-guide)

## Overview

Instead of managing 12 separate themes or a complex master theme, we use **Theme Groups** that export as independent **Figma Variable Collections**. Each collection has its own modes that designers toggle independently.

## How It Works

### 3 Independent Variable Collections

When synced to Figma, you get **3 separate Variable Collections** with independent modes:

```
┌─────────────────────────────────────────────────────────────┐
│ VARIABLE COLLECTION: Brand                                  │
├─────────────────────────────────────────────────────────────┤
│ Modes: [Default] [Brand2]                                   │
│ Purpose: Switch brand colors & typography                   │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ VARIABLE COLLECTION: Theme                                  │
├─────────────────────────────────────────────────────────────┤
│ Modes: [Light] [Dark]                                       │
│ Purpose: Switch color theme                                 │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ VARIABLE COLLECTION: Spacing                                │
├─────────────────────────────────────────────────────────────┤
│ Modes: [Default] [Compact] [Spacious]                       │
│ Purpose: Switch layout density                              │
└─────────────────────────────────────────────────────────────┘
```

### How Token Resolution Works

When you select modes, tokens resolve using **"last set wins"** logic:

```
Example: spacing-8 token

1. _Base/Value loads          → spacing-8 = 8px
2. Brand mode loads           → (no override)
3. Theme mode loads           → (no override)
4. Spacing/Compact loads      → spacing-8 = 7px ✓ (WINS)

Result: spacing-8 = 7px
```

All 3 collections are **independent** - they load together but don't conflict.

## Usage in Figma

### Step 1: Sync Tokens to Figma

1. Open Figma Tokens Studio plugin
2. Sync your token repository (GitHub, GitLab, etc.)
3. Plugin creates 3 Variable Collections:
   - ✅ Brand (modes: Default, Brand2)
   - ✅ Theme (modes: Light, Dark)
   - ✅ Spacing (modes: Default, Compact, Spacious)

### Step 2: Use Modes

In your Figma file:

1. **Select a component** or create a design
2. **Assign variables** to colors, spacing, etc.
3. **Toggle modes independently**:
   - Brand Collection → Click to switch brand
   - Theme Collection → Click to switch colors
   - Spacing Collection → Click to switch density

Each mode group works **completely independently** - no combinations to manage!

### Example Usage

**Scenario: Design a card for Brand2, Dark mode, Compact spacing**

```
Step 1: Assign brand-color-primary to card background
Step 2: Assign spacing-8 to card padding
Step 3: Toggle modes:
  • Brand mode     → Brand2 ✓
  • Theme mode     → Dark ✓
  • Spacing mode   → Compact ✓

Result:
  • Card background = Brand2 dark color
  • Card padding = 7px (compact spacing-8)
```

**Switch to Light mode?** Just toggle Theme collection mode to "Light" - everything updates!

## Theme Group Reference

### Brand Theme Group
| Theme | Colors | Typography | Spacing |
|-------|--------|------------|---------|
| Default | Default Brand colors | Default typography | Base layout |
| Brand2 | Brand2 colors | Brand2 typography | Base layout |

**Files enabled:** `01_Brand/Default` or `01_Brand/Brand2`

### Theme Group (Color Themes)
| Theme | Color Overrides | Purpose |
|-------|-----------------|---------|
| Light | Light mode colors | Bright background, dark text |
| Dark | Dark mode colors | Dark background, light text |

**Files enabled:** `03_Themes/Light` or `03_Themes/Dark`

### Spacing Group
| Theme | Spacing Multiplier | Use Case |
|-------|-------------------|----------|
| Default | 1.0x (base) | Standard layouts |
| Compact | 0.75x | Dense/mobile layouts |
| Spacious | 1.25x | Generous/desktop layouts |

**Files enabled:** `02_Layout` + `04_Responsive/Compact` or `04_Responsive/Spacious`

## Scaling: Adding New Variants

### Add Brand3

1. **Create `01_Brand/Brand3.json`** with Brand3 colors/typography
2. **Create new theme in $themes.json**:
   ```json
   {
     "id": "brand-brand3",
     "name": "Brand3",
     "group": "Brand",
     "selectedTokenSets": {
       "01_Brand/Brand3": "enabled",
       // ... other sets ...
     }
   }
   ```
3. **Sync to Figma** → Brand collection now has 3 modes!

### Add Responsive Typography

If Brand2 needs different typography sizes at Compact spacing:

1. **Create `04_Responsive/CompactTypography.json`** with Brand2 type overrides
2. **Create new theme**:
   ```json
   {
     "id": "spacing-compact-brand2",
     "name": "Compact",
     "group": "Spacing-Brand2",
     "selectedTokenSets": {
       "04_Responsive/Compact": "enabled",
       "04_Responsive/CompactTypography": "enabled"
     }
   }
   ```
3. **Sync to Figma** → New Spacing-Brand2 collection!

## JSON Structure

### Theme Group in $themes.json

```json
{
  "id": "brand-default",
  "name": "Default",
  "group": "Brand",
  "description": "Default brand - colors and typography",
  "selectedTokenSets": {
    "_Base/Value": "enabled",
    "01_Brand/Default": "enabled",
    "02_Layout": "enabled",
    "05_Interactions/States": "enabled",
    "07_Components/Compositions": "enabled"
  }
}
```

**Key Points:**
- `group`: Becomes Variable Collection name in Figma
- `name`: Becomes Mode name in Figma
- `selectedTokenSets`: Which token files load for this mode
- All token sets load together (no exclusions = simplicity!)

## Why This Approach?

### ✅ Benefits Over 12 Combined Themes

| Aspect | 12 Themes | Theme Groups (Modes) |
|--------|-----------|----------------------|
| **Flexibility** | Fixed combinations | Independent modes |
| **Switching** | Pick new theme (reload) | Toggle any mode |
| **Adding Brand3** | Create 6 new themes | Add 1 new theme |
| **Figma Collections** | 1 complex collection | 3 simple collections |
| **Developer Export** | Complex permutations | Each mode independent |
| **Discoverability** | 12 choices overwhelming | 3 groups of 2-3 modes |

### ✅ How Modes Export to Code

When developers export:

```bash
# Each collection exports independently
output/
  ├── Brand/
  │   ├── default.json    # Default brand tokens
  │   └── brand2.json     # Brand2 tokens
  ├── Theme/
  │   ├── light.json      # Light mode tokens
  │   └── dark.json       # Dark mode tokens
  └── Spacing/
      ├── default.json    # Base spacing
      ├── compact.json    # Compact spacing
      └── spacious.json   # Spacious spacing
```

Developers use these in code:
```javascript
// CSS variables
:root {
  // Load base tokens
  --color-primary: var(--Brand-default);
  --spacing-8: var(--Spacing-default);
}

// Toggle modes via class
body.dark {
  --color-primary: var(--Theme-dark);
}

body.compact-layout {
  --spacing-8: var(--Spacing-compact);
}
```

## FAQ

**Q: What if I want responsive typography to change with Spacing mode?**
A: Create separate token sets (`04_Responsive/CompactType.json`) and include them in the Spacing theme. When Spacing mode changes, typography updates too.

**Q: Can I have 4+ mode groups?**
A: Yes! Create more theme groups in $themes.json. Each becomes a separate collection in Figma.

**Q: What's the difference between this and the old 12 themes?**
A: Old approach = designer picks 1 theme, locks into all 3 dimensions. New approach = designer toggles each dimension independently.

**Q: Do modes require Figma Tokens Studio Pro?**
A: Theme Switching is a Pro feature, but exporting to Figma Variables (free) works with the exported data.

**Q: Can I have more than one mode active per collection?**
A: No - Figma Variables allow only 1 mode active per collection at a time. This is the "either this or that" logic.

## Resources

- [Tokens Studio - Themes that Switch](https://docs.tokens.studio/manage-themes/simple-switch-guide)
- [Figma Variables Documentation](https://help.figma.com/hc/en-us/articles/15343816063012)
- [Design Token Fundamentals](./TOKEN_FUNDAMENTALS.md)

## Next Steps

1. **Sync to Figma** - Push this configuration to your remote repo
2. **Test in Figma** - Open Figma Tokens plugin, load tokens, see 3 collections
3. **Toggle modes** - Try switching between Brand/Theme/Spacing
4. **Apply to components** - Assign variables to component properties
5. **Export to code** - Use SD-Transforms to convert modes for development

---

**Questions?** Check the Tokens Studio docs or review `$themes.json` for current theme definitions.
