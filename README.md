# AE Align Anchor Point (v1.1.0)

[![Live Browser Simulator](https://img.shields.io/badge/Live%20Demo-AE%20ScriptUI%20Simulator-2c8cff?style=for-the-badge&logo=javascript)](https://work2survive.github.io/AE_JS_builder/?jsx=AE_Align_Anchor_Point.jsx&autobuild=true)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)
[![Adobe After Effects](https://img.shields.io/badge/Adobe%20After%20Effects-CS6%20--%20CC%202026%2B-9999FF?style=for-the-badge&logo=adobeaftereffects)](https://www.adobe.com/products/aftereffects.html)
[![Built with Antigravity](https://img.shields.io/badge/Built%20With-Antigravity-4285F4?style=for-the-badge&logo=google)](https://deepmind.google/)

> **Interactive Web Simulator**: Test, customize, and preview this tool directly in your browser without launching After Effects via the [AE JS & ScriptUI Simulator](https://work2survive.github.io/AE_JS_builder/?jsx=AE_Align_Anchor_Point.jsx&autobuild=true).

---

## 📖 Overview

**AE Align Anchor Point** is an Adobe After Effects ScriptUI panel and live expression engine that aligns the anchor point of any composition, text box (paragraph or point text), vector shape, solid, footage, or null layer to Left, Center, or Right (X) and Top, Center, or Bottom (Y) with **zero visual jump or shift** of content on screen.

Moving an anchor point natively in After Effects shifts the layer's visual position across the composition. **AE Align Anchor Point** completely eliminates this drift by pairing the Anchor Point with a scale-safe and rotation-safe Position compensator expression (or static keyframe bake), keeping the layer's rendered visual content strictly locked in place.

---

## 🌟 Key Features

1. **Zero-Shift Content Invariance**:
   - Mathematically couples Anchor Point and Position so that moving the anchor point never displaces the visual content on screen.
   - Scale-safe ($S / 100$) and rotation-safe ($R(\theta)$) compensation prevents drift even if the layer is already scaled or rotated.

2. **Dual Calculation Engines**:
   - **`sourceRectAtTime(time, false)`**: Dynamically measures the exact rendered bounding box of glyphs, ascenders, descenders, and vector paths.
   - **Stated Width & Height**: Aligns to declared paragraph box text dimensions (`boxTextSize`), solid sizes (`layer.width`, `layer.height`), or precomp canvas dimensions (`source.width`, `source.height`).

3. **Guideline Vector Crosshair Reticle**:
   - Creates a procedural vector crosshair reticle linked as a child layer at position `[0, 0]` with anchor point `[0, 0]`.
   - Automatically follows the parent layer's anchor point through all movements, rotations, and scales.
   - Marked as a guide layer (`guideLayer = true`): clearly visible in the composition working viewer, but completely non-renderable in production renders, Adobe Media Encoder, and nested precompositions.

4. **Layer State Persistence**:
   - Stores the active alignment parameters directly on the layer using standard After Effects effect controls (`Anchor Align X`, `Anchor Align Y`, `Anchor Bounds Mode`, `Anchor Base Point`).
   - Selecting a previously aligned layer automatically restores and reflects its alignment settings in the UI.

5. **Quick 3x3 Preset Visual Matrix**:
   - Dedicated 9-button grid (`Top-Left`, `Center`, `Bot-Right`, etc.) for one-click alignment.

6. **Dual Execution Modes**:
   - **Live Dynamic Expressions**: Anchor point and position remain continuously reactive to text edits, font size changes, and content resizing.
   - **One-Time Static Bake**: Instantly applies static coordinates without active expressions running in the timeline.

---

## 🖥️ ScriptUI Panel Layout

```text
┌────────────────────────────────────────────────────────┐
│ ⚓ AE ALIGN ANCHOR POINT v1.1.0                         │
│ Zero-Shift Anchor Aligner with State Effect & Reticle  │
├────────────────────────────────────────────────────────┤
│ Target Selection:                                      │
│ 'Title_Text' [Text Layer, Anchor: 230, 90]             │
│ [ 🔄 Refresh Selection ]                                │
├────────────────────────────────────────────────────────┤
│ Bounds Calculation Method:                             │
│ (•) sourceRectAtTime() (Dynamic)  ( ) Stated Dimensions│
├────────────────────────────────────────────────────────┤
│ Horizontal Alignment (X):                              │
│ ( ) Left Edge      (•) Center        ( ) Right Edge    │
├────────────────────────────────────────────────────────┤
│ Vertical Alignment (Y):                                │
│ ( ) Top Edge       (•) Center        ( ) Bottom Edge   │
├────────────────────────────────────────────────────────┤
│ Quick 9-Point Presets (Visual Matrix):                 │
│ [ Top-Left   ]  [ Top-Center ]  [ Top-Right   ]        │
│ [ Mid-Left   ]  [ Center     ]  [ Mid-Right   ]        │
│ [ Bot-Left   ]  [ Bot-Center ]  [ Bot-Right   ]        │
├────────────────────────────────────────────────────────┤
│ Anchor Cross Indicator (Child Layer at [0, 0]):        │
│ [x] Enable Child Crosshair Reticle                     │
│ [x] Guide Layer (non-renderable in exports)            │
│ Size (px): [ 30 ]  Color: [ Cyan                    ▼] │
├────────────────────────────────────────────────────────┤
│ Execution Mode:                                        │
│ (•) Live Dynamic Expressions  ( ) One-Time Static Bake │
│ [ ⚓ Align Anchor Point (Keep Content in Place)       ] │
│ [ ✕ Remove Crosshairs & Expressions                  ] │
│ Ready.                                                 │
│ GitHub: work2survive/AE-Align-AnchorPoint              │
└────────────────────────────────────────────────────────┘
```

---

## ⚙️ Mathematical Derivation & Zero-Shift Content Mechanics

### 1. The Anchor Point Coordinate Relationship
In After Effects, every 2D layer's visual content is transformed according to the classic affine transformation:
$$\mathbf{X}_{\text{comp}} = \mathbf{P} + \mathbf{R}(\theta) \cdot \mathbf{S} \cdot (\mathbf{x}_{\text{layer}} - \mathbf{A})$$

Where:
- $\mathbf{P} = [P_x, P_y]^T$ is the layer's **Position** in composition space.
- $\mathbf{A} = [A_x, A_y]^T$ is the layer's **Anchor Point** in layer local space.
- $\mathbf{x}_{\text{layer}}$ is an arbitrary point within the layer's content (e.g. top-left corner of text glyphs).
- $\mathbf{S} = \begin{bmatrix} s_x/100 & 0 \\ 0 & s_y/100 \end{bmatrix}$ is the scale matrix.
- $\mathbf{R}(\theta) = \begin{bmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{bmatrix}$ is the 2D rotation matrix.

### 2. The Zero-Shift Invariance Condition
When moving the anchor point from $\mathbf{A}_{\text{base}}$ to $\mathbf{A}_{\text{target}}$, the content point $\mathbf{x}_{\text{layer}}$ must satisfy:
$$\mathbf{X}_{\text{comp}}(\text{new}) = \mathbf{X}_{\text{comp}}(\text{old})$$

$$\mathbf{P}_{\text{new}} + \mathbf{R} \cdot \mathbf{S} \cdot (\mathbf{x} - \mathbf{A}_{\text{target}}) = \mathbf{P}_{\text{base}} + \mathbf{R} \cdot \mathbf{S} \cdot (\mathbf{x} - \mathbf{A}_{\text{base}})$$

Subtracting like terms yields the universal compensation formula:
$$\mathbf{P}_{\text{new}} = \mathbf{P}_{\text{base}} + \mathbf{R}(\theta) \cdot \mathbf{S} \cdot (\mathbf{A}_{\text{target}} - \mathbf{A}_{\text{base}})$$

Let $\Delta \mathbf{A} = \mathbf{A}_{\text{target}} - \mathbf{A}_{\text{base}}$. When rotation $\theta = 0$:
$$P_x(\text{new}) = P_x(\text{base}) + \Delta A_x \cdot \frac{s_x}{100}$$
$$P_y(\text{new}) = P_y(\text{base}) + \Delta A_y \cdot \frac{s_y}{100}$$

When rotation $\theta \neq 0$:
$$\Delta P_x = \Delta A_x \cdot \frac{s_x}{100} \cdot \cos\theta - \Delta A_y \cdot \frac{s_y}{100} \cdot \sin\theta$$
$$\Delta P_y = \Delta A_x \cdot \frac{s_x}{100} \cdot \sin\theta + \Delta A_y \cdot \frac{s_y}{100} \cdot \cos\theta$$
$$\mathbf{P}_{\text{new}} = \mathbf{P}_{\text{base}} + [\Delta P_x, \Delta P_y]^T$$

### 3. The Special Case: Base Anchor at $[0, 0]$
When a layer's base anchor point was $[0, 0]$ (common for text layers and shape layers created at the origin), $\mathbf{A}_{\text{base}} = [0, 0]$, which simplifies $\Delta \mathbf{A} = \mathbf{A}_{\text{target}}$.
In that case, the position expression reduces to:
```javascript
const r = sourceRectAtTime(time, false);
const s = [transform.scale[0] / 100, transform.scale[1] / 100];
value + [r.left * s[0], r.top * s[1]];
```

---

## 🎯 The Vector Crosshair Reticle & Child Layer Mechanics

In After Effects, parenting child layers introduces a crucial coordinate property:
> **The origin `[0, 0]` of a child layer in parent space is located at the parent layer's Anchor Point, NOT its top-left corner.**

When layer $B$ is parented to layer $A$ ($B.\text{parent} = A$):
$$\mathbf{X}_{B,\text{comp}} = \mathbf{P}_A + \mathbf{R}_A \cdot \mathbf{S}_A \cdot (\mathbf{P}_B - \mathbf{A}_A) + \dots$$
When $B$'s local position is explicitly set to $[0, 0]$ and $B$'s anchor point is $[0, 0]$:
$$\mathbf{X}_{B,\text{comp}} = \mathbf{P}_A$$
Because $\mathbf{P}_A$ is the screen position of layer $A$'s anchor point, layer $B$ is physically pinned to layer $A$'s anchor point under all scales, rotations, and animations.

### Why `guideLayer = true` Matters
Setting `crossLayer.guideLayer = true`:
1. Renders the reticle directly in the active After Effects composition viewer so the motion designer can see the anchor pivot in real time.
2. Marks the layer with a hash icon (`#`) in the timeline.
3. Automatically suppresses the layer during final rendering in the Render Queue, Adobe Media Encoder, and when the composition is nested as a precomp into other compositions.

---

## 🔬 Deep-Dive: Anchor Points in After Effects vs. JavaScript Emulation

When building tools that run both natively in Adobe After Effects (ExtendScript ES3 engine) and inside JavaScript browser emulators (like the [AE JS & ScriptUI Simulator](https://work2survive.github.io/AE_JS_builder/)), several critical runtime differences emerge:

### 1. Operator Overloading in Vector Math
- **After Effects Native Engine**: The AE expression engine (both the Modern JavaScript engine and Legacy ExtendScript) natively overloads mathematical operators for arrays and vectors:
  ```javascript
  // Valid in native After Effects expressions:
  const s = transform.scale / 100;           // Evaluates to [1, 1]
  const pos = value + [r.left, r.top];       // Vector addition
  ```
- **Pure JavaScript / V8 / Web Browsers**: Pure JavaScript does not support operator overloading:
  - `[100, 100] / 100` evaluates to `NaN`!
  - `[960, 540] + [10, 20]` evaluates to `"960,54010,20"` (string concatenation)!
- **Universal Parity Rule**: Expressions must always index components explicitly:
  ```javascript
  const s = [transform.scale[0] / 100, transform.scale[1] / 100];
  const newPos = [value[0] + dx, value[1] + dy];
  ```
  This component-wise syntax is 100% valid in native AE and strictly valid in JavaScript environments.

### 2. Multi-Dimensional Property Array Indexing
- **In After Effects**: Multi-dimensional properties (e.g. `Position`, `Scale`, `Anchor Point`) can be indexed directly:
  ```javascript
  transform.scale[0]; // Returns 100
  ```
- **In JavaScript Emulation**: Properties in JS object models are instances of a `Property` class. Accessing `transform.scale[0]` returns `undefined` unless the class implements numeric index getters:
  ```javascript
  get 0() { const v = this.value; return Array.isArray(v) ? v[0] : v; }
  get 1() { const v = this.value; return Array.isArray(v) ? v[1] : undefined; }
  get 2() { const v = this.value; return Array.isArray(v) ? v[2] : undefined; }
  get length() { const v = this.value; return Array.isArray(v) ? v.length : 1; }
  ```

### 3. Parent Space Transform Matrix Separation
- **In After Effects**:
  - The parent matrix transformation for child layers is:
    $$M_{\text{parent}} = T(\text{position}) \cdot R(\text{rotation}) \cdot S(\text{scale})$$
  - The local offset $-T(\text{anchor})$ applies **only** to the parent layer's own internal drawing content, **not** to child layers!
- **In JS Emulators**:
  - A common bug is computing a single monolithic `getWorldMatrix()` that includes $-T(\text{anchor})$ and using it for child layers. This causes child layers at $[0, 0]$ to be pinned to the top-left corner instead of the anchor point.
  - Fix: Decouple `getParentSpaceMatrix()` from `getWorldMatrix()`.

### 4. Expression Circular Dependency & Base Value Extraction (`origA`)
- **In After Effects**:
  - If `transform.anchorPoint` has an expression attached, querying `transform.anchorPoint.value` from a `position` expression returns the **post-expression evaluated value**, NOT the raw static base value!
  - If the position expression computes $\Delta \mathbf{A} = \mathbf{A}_{\text{target}} - \text{transform.anchorPoint.value}$, both terms become identical, causing $\Delta \mathbf{A} = 0$!
- **Solution**:
  - Store the baseline anchor point $\mathbf{A}_{\text{base}}$ into an effect control (`Anchor Base Point`), or embed the static base coordinates directly into the position expression:
    ```javascript
    const origA = [230, 90]; // Statically referenced baseline
    const dx = (targetX - origA[0]) * s[0];
    ```

### 5. `sourceRectAtTime()` vs. `boxTextSize`
- **`sourceRectAtTime(time, false)`**:
  - Measures the tight visual bounding box of rendered character paths.
  - Ascenders (like `h`, `d`, `k`), descenders (like `g`, `y`, `p`), and capital letters influence `r.top` and `r.height`.
  - For empty or whitespace lines, dimensions collapse.
- **`boxTextSize`**:
  - Represents the declared spatial container box of paragraph text.
  - In native AE: Accessed via `thisLayer.text.sourceText.value.boxTextSize`.
  - In modern AE JS expressions:
    ```javascript
    const sDoc = thisLayer.text ? thisLayer.text.sourceText.value : null;
    const boxW = sDoc && sDoc.boxTextSize ? sDoc.boxTextSize[0] : thisLayer.width;
    ```

---

## 📦 Installation & Setup

### Option 1: Dockable ScriptUI Panel in After Effects
1. Download [`AE_Align_Anchor_Point.jsx`](AE_Align_Anchor_Point.jsx).
2. Place the `.jsx` file inside your After Effects **ScriptUI Panels** directory:
   - **Windows**: `C:\Program Files\Adobe\Adobe After Effects <Version>\Support Files\Scripts\ScriptUI Panels\`
   - **macOS**: `/Applications/Adobe After Effects <Version>/Scripts/ScriptUI Panels/`
3. Restart After Effects or click **Window → AE_Align_Anchor_Point.jsx**.
4. Dock the panel anywhere into your AE workspace layout.

### Option 2: Live Browser Emulator
Test and run the panel directly in your browser without Adobe After Effects:
👉 **[Launch Live Demo in AE ScriptUI Simulator](https://work2survive.github.io/AE_JS_builder/?jsx=AE_Align_Anchor_Point.jsx&autobuild=true)**

---

## 📄 License & Credits

- **Author**: Co-authored & architected with **Google DeepMind Antigravity**.
- **Repository**: [https://github.com/work2survive/AE-Align-AnchorPoint](https://github.com/work2survive/AE-Align-AnchorPoint)
- **License**: [MIT License](LICENSE)