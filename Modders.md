# 🌌 Wide Horizons – Scaling API Developer Guide

Wide Horizons dynamically scales the **Starsector galaxy** (sector width/height, constellation count, etc.).  
To keep your systems aligned when scaling is active, register them with the API instead of placing them manually.  

This ensures:
- ✔️ Correct scaled positions  
- ✔️ Entities (stations, jump points, relays) stay aligned  
- ✔️ Hyperspace is cleared/refilled around systems  
- ✔️ Compatibility with Nexerelin & other mods  

---

## 🚀 Quick Start

Add a `compileOnly` dependency on Wide Horizons and call:

```java
import org.widehorizons.api.WideHorizonsAPI;

// Register a system at its vanilla reference position
WideHorizonsAPI.registerSystem("my_system_id", 12000f, -8000f);
```

---

## 📦 API Methods

| Method | Description |
|--------|-------------|
| `registerSystem(String id, float x, float y)` | Register one system at vanilla reference coordinates |
| `registerConstellation(String anchorId, float x, float y)` | Move an entire constellation so the anchor ends up at (x,y) |
| `registerConstellationAsIs(String anchorId)` | Register a constellation *as-is*, using current positions as reference |
| `registerConstellationAsIsByTag(String tag)` | Same as above, but search anchor by tag |
| `Vector2f getScaledPosition(float x, float y)` | Compute scaled coordinates without registering |
| `Vector2f getScaledPosition(float x, float y, boolean coreWorld)` | Same, but apply core-world scaling rules |

---

## 🕒 When to Call

- Call **after your system is generated** (ID & entities exist).  
- Call **before** you manually move the system again.  
- Typical places:
  - At the end of your system generator `generate()`  
  - Inside `onNewGame()` or `onNewGameAfterProcGen()`  

👉 The API is idempotent — duplicate registrations are ignored with a log warning.

---

## 🌌 Coordinate System

- Galaxy center: `(0, 0)`  
- **Vanilla reference coords:** positions in a vanilla sector (164 000 × 104 000)  
- Wide Horizons multiplies these by the configured **sector scale**  
- Core worlds can use a different scaling factor  

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

---

## ✅ What You Get Automatically

- Core vs. rim scaling factors applied correctly  
- Static entities re-aligned relative to their star  
- Hyperspace cleared/refilled around systems  
- Discovery & map markers updated  

---

## ⚠️ Best Practices

**Do**
- Register once per system/constellation  
- Pass vanilla reference coords where possible  
- Use `registerConstellationAsIs` for dynamically placed clusters  

**Don’t**
- Don’t move systems manually after registering  
- Don’t register the same system twice  
- Don’t worry about hyperspace cleanup — it’s automatic  

---

## 🔄 Reflection Safe-Call (Optional)

If you don’t want a hard dependency:

```java
if (Global.getSettings().getModManager().isModEnabled("WideHorizons")) {
    try {
        Class<?> api = Global.getSettings().getScriptClassLoader()
            .loadClass("org.widehorizons.api.WideHorizonsAPI");
        api.getMethod("registerSystem", String.class, float.class, float.class)
           .invoke(null, "my_system_id", 12000f, -8000f);
    } catch (Throwable ignored) {}
}
```

---

## 🛠 Troubleshooting

- **System didn’t move:** check ID, ensure call after system creation  
- **Moved twice:** remove your own repositioning after registration  
- **Hyperspace looks wrong:** ensure registration before Wide Horizons scaling pass  

---

## TL;DR

```java
// Single system
WideHorizonsAPI.registerSystem("my_system_id", VANILLA_X, VANILLA_Y);

// Whole constellation
WideHorizonsAPI.registerConstellation("anchor_system_id", TARGET_X, TARGET_Y);

// Adopt current positions
WideHorizonsAPI.registerConstellationAsIs("anchor_system_id");
```

That’s all you need for **scale-aware placement** with Wide Horizons ✨
