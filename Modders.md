---

# 🌌 Wide Horizons - Scaling API Developer Guide

Wide Horizons dynamically scales the **Starsector galaxy** (sector width/height, constellation count, etc.).
To keep your systems aligned when scaling is active, register them with the API instead of placing them manually.

This ensures:

* ✔️ Correct scaled positions
* ✔️ Entities (stations, jump points, relays) stay aligned
* ✔️ Hyperspace is cleared/refilled around systems
* ✔️ Compatibility with Nexerelin & other mods

---

## 🚀 Quick Start

Add a `compileOnly` dependency on Wide Horizons and call:

```java
import org.widehorizons.api.WideHorizonsAPI;

// Register a system at its vanilla reference position
WideHorizonsAPI.registerSystem("my_system_id", 12000f, -8000f);
```

If your mod has a **capital / main hub** you want treated as a **Core World** for scaling purposes:

```java
// Mark your capital as a core world (uses the player's WH core scaling rules)
WideHorizonsAPI.registerCoreSystem("my_capital_system_id");
```

---

## 📦 API Methods

| Method                                                            | Description                                                            |
| ----------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `registerSystem(String id, float x, float y)`                     | Register one system at vanilla reference coordinates                   |
| `registerConstellation(String anchorId, float x, float y)`        | Move an entire constellation so the anchor ends up at (x,y)            |
| `registerConstellationAsIs(String anchorId)`                      | Register a constellation *as-is*, using current positions as reference |
| `registerConstellationAsIsByTag(String tag)`                      | Same as above, but search anchor by tag                                |
| `Vector2f getScaledPosition(float x, float y)`                    | Compute scaled coordinates without registering                         |
| `Vector2f getScaledPosition(float x, float y, boolean coreWorld)` | Same, but apply core-world scaling rules                               |
| `registerCoreSystem(String systemId)`                             | **NEW:** Explicitly mark a system as “core-like” for scaling           |
| `unregisterCoreSystem(String systemId)`                           | **NEW:** Remove a previously registered core system                    |
| `boolean isCoreSystem(StarSystemAPI system)`                      | **NEW:** Query whether WH currently treats a system as core            |

> ℹ️ **What counts as “Core” in WH?**
> Wide Horizons recognizes vanilla core worlds, any system flagged with vanilla core tags (`THEME_CORE*`), systems having visible populated markets, plus anything you explicitly mark via `registerCoreSystem(...)`. Marking your capital is the most reliable cross-mod approach.

---

## 🕒 When to Call

* Call **after your system is generated** (ID & entities exist).
* Call **before** you manually move the system again.
* Typical places:

  * At the end of your system generator `generate()`
  * In `onNewGame()` / `onNewGameAfterProcGen()` / `onNewGameAfterEconomyLoad()` (once the system exists)

👉 The API is idempotent — duplicate registrations are ignored with a log warning.

---

## 🌌 Coordinate System

* Galaxy center: `(0, 0)`
* **Vanilla reference coords:** positions in a vanilla sector (164 000 × 104 000)
* Wide Horizons multiplies these by the configured **sector scale**
* **Core Worlds** can use a different **core scale** (player setting)

---

## 🔧 Examples

### Register a single system

```java
WideHorizonsAPI.registerSystem("elysium", -5000f, 8000f);
```

### Register a constellation

```java
WideHorizonsAPI.registerConstellation("arcadia", 20000f, -12000f);
```

### Use current positions as reference

```java
WideHorizonsAPI.registerConstellationAsIs("frostbite");
```

### Get scaled coords directly

```java
Vector2f pos = WideHorizonsAPI.getScaledPosition(10000f, -6000f);
system.getLocation().set(pos.x, pos.y);
```

### **NEW:** Mark your capital as a Core World

```java
// Ensure your capital follows the core scaling bucket (not the rim bucket)
WideHorizonsAPI.registerCoreSystem("my_capital");
```

### **NEW:** Query core status (for debugging / logic)

```java
boolean isCore = WideHorizonsAPI.isCoreSystem(system);
```

### **NEW:** Undo explicit core registration (rarely needed)

```java
WideHorizonsAPI.unregisterCoreSystem("my_capital");
```

---

## ✅ What You Get Automatically

* Correct **core vs. rim** scaling factors applied
* Static entities re-aligned relative to their star
* Hyperspace cleared/refilled around systems
* Discovery & map markers updated

---

## 🧭 Core Worlds & Player Settings

* Players control the magnitude of **core scaling** via the existing Wide Horizons Luna setting (`wh_coreWorldScaleMultiplier`).
* By calling `registerCoreSystem("your_capital")`, you **assign your capital to the core bucket** — then the player’s core-scaling preference is applied to it (e.g., “Vanilla” = no scaling, or a specific multiplier).
* You don’t need to add custom settings — just register the system to ensure it’s treated as core consistently across random/Corvus starts and mod mixes.

---

## ⚠️ Best Practices

**Do**

* Register once per system/constellation
* Pass vanilla reference coords where possible
* Use `registerConstellationAsIs` for dynamically placed clusters
* **Mark exactly your capital/main hub** as core (don’t mark everything)

**Don’t**

* Don’t move systems manually after registering
* Don’t register the same system twice
* Don’t rely solely on vanilla tags in modded environments — use `registerCoreSystem(...)` to be future-proof

---

## 🔄 Reflection Safe-Call (Optional, no hard dep)

```java
if (Global.getSettings().getModManager().isModEnabled("WideHorizons")) {
    try {
        ClassLoader cl = Global.getSettings().getScriptClassLoader();
        Class<?> api = cl.loadClass("org.widehorizons.api.WideHorizonsAPI");

        // registerSystem
        api.getMethod("registerSystem", String.class, float.class, float.class)
           .invoke(null, "my_system_id", 12000f, -8000f);

        // NEW: registerCoreSystem
        api.getMethod("registerCoreSystem", String.class)
           .invoke(null, "my_capital");

        // OPTIONAL: isCoreSystem
        boolean core = (Boolean) api.getMethod("isCoreSystem", StarSystemAPI.class)
                                    .invoke(null, Global.getSector().getStarSystem("my_capital"));
    } catch (Throwable ignored) {}
}
```

---

## 🛠 Troubleshooting

* **System didn’t move:** check ID, ensure the call happens *after* system creation
* **Moved twice:** remove your own repositioning after registration
* **Hyperspace looks wrong:** ensure registration before the WH scaling pass
* **Core flag ignored?** Make sure you call `registerCoreSystem(...)` after the system exists (post-generation), and before scaling runs; avoid unregistering unless you know why

---

## TL;DR

```java
// Single system
WideHorizonsAPI.registerSystem("my_system_id", VANILLA_X, VANILLA_Y);

// Whole constellation
WideHorizonsAPI.registerConstellation("anchor_system_id", TARGET_X, TARGET_Y);

// Adopt current positions
WideHorizonsAPI.registerConstellationAsIs("anchor_system_id");

// NEW: ensure your capital is treated as a Core World
WideHorizonsAPI.registerCoreSystem("my_capital_system_id");
```

That’s all you need for **scale-aware placement** including **Core World** handling with Wide Horizons ✨
