# **SkyVox — Procedural Voxel Sky**

A procedural, texture-free skybox for Unity URP. Voxel-based volumetric clouds, a full day/night cycle, and a runtime profile system for blending between named looks — weather, times of day, moods — without hand animation.

---

## **Introduction**

- No external textures — sky, sun, moon, stars and clouds are all generated in the shader.
- Square sun, moon and stars by design, with voxel clouds rendered through a 3D-DDA raymarcher for sharp, noise-free edges.
- A profile system lets you tag looks to a time of day and have them blend in automatically, or drive them manually from your own gameplay code.
- Built for Unity 2022 LTS and newer, URP 14+.

---

## **Getting Started**

1. Create an empty GameObject in your scene (e.g. `Sky`).
2. Add both components to it: **Component > SkyVox > Sky Setup** and **Component > SkyVox > Sky Controller**.
3. On **Sky Setup**, assign:
   - **Default Material** — the SkyVox material included with the package.
   - **Sun Light** / **Moon Light** — your scene's directional lights.
4. Click **Check / Fix Skybox Material** in the Sky Setup inspector.
5. Move the **Time Of Day** slider on Sky Controller. Sky, sun, moon and clouds update live in the Scene view.

---

## **Setup**

`SkyVoxSetup` is the only place you assign references. It has three fields: **Default Material**, **Sun Light**, **Moon Light**. On enable (and whenever you change a field), it automatically pushes those references into `SkyVoxController` on the same GameObject — Setup and Controller must live together.

The Controller's own material/light fields are hidden in the Inspector on purpose. Always assign them through Setup, never directly on the Controller.

**Check / Fix Skybox Material** compares the scene's current `RenderSettings.skybox` shader against SkyVox's shader. If it's already correct, nothing changes. Otherwise, it assigns your Default Material. You can also call this from your own scripts:

```csharp
GetComponent<SkyVoxSetup>().FixSkyboxMaterial();
```

> Note: the shader itself is still registered internally as `VoxelNest/VoxelSky` — this is only a shader name, unrelated to the `SkyVox` C# namespace used everywhere else.

If the Controller is enabled without a material assigned, it logs a console warning reminding you to add Setup and assign a Default Material.

---

## **Modifying the Sky**

All visual parameters live in one place: the **Data** field on `SkyVoxController` (a `SkyVoxData` object). It can be edited directly in Edit Mode or Play Mode for instant preview.

**Time** (on the Controller, outside Data):

| Field | Range | Description |
|---|---|---|
| Time Of Day | 0–24 | 0 = midnight, 6 = sunrise, 12 = noon, 18 = sunset. |
| Auto Advance | — | Advances Time Of Day every frame. |
| Day Duration Seconds | — | Real seconds per full day/night cycle. Only applies when Auto Advance is on. |

**Light Intensity**

| Field | Range | Description |
|---|---|---|
| Sun Intensity | 0–8 | Sun light intensity at full daylight. |
| Moon Intensity | 0–3 | Moon light intensity at full night. |

**Sky Colors**

| Field | Description |
|---|---|
| Sky Zenith | Sky color straight up at midday. |
| Sky Horizon | Sky color at the horizon during daytime. |
| Sky Sunset | Warm tint blended in near sunrise and sunset. |
| Sky Night | Sky color at night. |
| Ground Color | Color below the horizon line. |

**Sky Blending**

| Field | Range | Description |
|---|---|---|
| Horizon Falloff | 1–10 | Sharpness of the zenith-to-horizon gradient. Lower is softer. |
| Sunset Band Width | 0.01–0.5 | How close the sun needs to be to the horizon before the sunset tint kicks in. |
| Day Night Width | 0.02–0.5 | Length of the sunrise/sunset transition. Also drives the light intensity crossfade. |
| Atmosphere Thickness | 0–2 | Haze strength near the horizon. |

**Sun**

| Field | Range | Description |
|---|---|---|
| Sun Color | — | Color of the sun disc. |
| Sun Size | 0.001–0.15 | Angular size of the sun disc. |
| Sun Glow Color | — | Color of the sun's glow. |
| Sun Glow Size | 0.01–0.8 | Angular size of the sun's glow. |
| Sun Glow Falloff | 0.5–8 | How tightly the glow hugs the disc. Higher is tighter. |

**Moon**

| Field | Range | Description |
|---|---|---|
| Moon Color | — | Color of the moon disc. |
| Moon Size | 0.001–0.08 | Angular size of the moon disc. |
| Moon Glow Color | — | Color of the moon's glow. |
| Moon Glow Size | 0.01–0.8 | Angular size of the moon's glow. |
| Moon Glow Falloff | 0.5–8 | How tightly the glow hugs the disc. Higher is tighter. |

**Stars**

| Field | Range | Description |
|---|---|---|
| Stars Amount | 0–1 | Amount of stars in the sky. Higher means more stars. |
| Stars Size | 0.2–3.0 | Star size. |
| Stars Intensity | 0–5 | Star brightness. |
| Stars Twinkle | 0–3 | How much stars flicker. Above 1 overshoots for a punchier effect. |
| Stars Twinkle Speed | 0–5 | How fast stars flicker. |

**Cloud Shape**

| Field | Range | Description |
|---|---|---|
| Cloud Coverage | 0–1 | Fraction of the sky filled with cloud. |
| Cloud Altitude Low | 250–2000 | Bottom of the cloud layer, world units. |
| Cloud Altitude High | 250–2000 | Top of the cloud layer, world units. |
| Cloud Voxel Size | 0.03–64 | Size of one cloud block before Scale is applied. |
| Cloud Scale | 0.2–4 | Scales cloud sampling. Changing this or Voxel Size reshuffles which cells are solid. |

**Cloud Motion**

| Field | Range | Description |
|---|---|---|
| Cloud Speed | 0–50 | Wind scroll speed. |
| Cloud Wind Angle | 0–360 | Wind direction, degrees. |

**Cloud Appearance**

| Field | Range | Description |
|---|---|---|
| Cloud Density | 0.1–6 | How fast clouds go opaque through their depth. Higher is more solid. |
| Cloud Softness | 0–1 | Width of the sky-to-cloud transition. 0 keeps voxel edges crisp. |
| Cloud Opacity | 0–1 | Master opacity for the whole cloud layer. Handy for fades. |
| Cloud Color Light | — | Cloud color on sunlit faces. |
| Cloud Color Shadow | — | Cloud color in self-shadow. |
| Cloud Shadow Strength | 0–1 | Self-shadow strength. 0 disables it. |

**Cloud Quality**

| Field | Range | Description |
|---|---|---|
| Cloud Steps | 16–64 | Raymarch quality baseline. Real step count scales with view angle and is clamped internally. |
| Cloud Shadow Step | 0.01–64 | Minimum spacing between self-shadow samples, world units. |
| Cloud Horizon Fade | 0–1 | How far above the horizon clouds start fading in. |

**Fog** (Unity's built-in scene fog)

| Field | Range | Description |
|---|---|---|
| Fog Enabled | — | Enables Unity's built-in scene fog (Lighting > Environment > Fog). |
| Fog Color | — | Fog color. Try matching this to your horizon color for a seamless blend. |
| Fog Intensity | 0–0.3 | Fog density (Exponential Squared). |

**Output**

| Field | Range | Description |
|---|---|---|
| Exposure | 0–4 | Final brightness multiplier for the whole sky. |

---

## **Creating Profiles**

A `SkyVoxProfile` is a ScriptableObject that stores a `SkyVoxData` look tagged to one or more times of day via **Time Tags** (a `[Flags]` enum — a profile can apply to several tags at once).

| Tag | Hour Range |
|---|---|
| Night | 21:00 – 05:00 |
| Dawn | 05:00 – 07:00 |
| Morning | 07:00 – 10:00 |
| Late Morning | 10:00 – 12:00 |
| Midday | 12:00 – 14:00 |
| Afternoon | 14:00 – 17:00 |
| Late Day | 17:00 – 19:00 |
| Dusk | 19:00 – 21:00 |

**Two ways to create a profile:**

- **From the Controller** — tweak Data live in the scene, then in the **Profile Tools** section of the Sky Controller inspector click **Save As New**. Choose a location; a new asset is created from the current Data and automatically added to the Profiles list.
- **Directly** — right-click in the Project window: **Create > SkyVox > Sky Profile**.

**Editing an existing profile's look:** select the profile asset itself in the Project window — its Data fields are fully editable right there, with the same headers as on the Controller. No need to route changes back through the Controller.

**Loading a profile onto the Controller:** assign it as **Target Profile** and click **Load Target** — this clones that profile's Data onto the live Data field, so you can preview it in the scene or use it as a starting point for a new one.

> Profiles created via the Project window's Create menu aren't in the Profiles list automatically — drag them in by hand if you want Linear/Shuffle to use them.

---

## **Profile Transitions**

**Profile Mode** on the Controller:

- **Off** — Data is only touched by hand, `SetWeather`, or `TransitionWeather`. The Profiles list is ignored.
- **Manual** — nothing changes on its own. You decide when to switch, via `SetProfile` / `SetProfileByIndex`.
- **Linear** — automatically advances through the Profiles list, but only to a profile whose Time Tags include the current tag. If more than one profile qualifies, the one that's been inactive longest is chosen, so profiles sharing a tag rotate fairly instead of one blocking the others.
- **Shuffle** — same tag-matching rule, but the next profile is picked at random among the candidates (never repeating the one that just ended, unless it's the only match).

If no profile in the list matches the current time tag, the sky simply keeps its current look until one does — nothing pops or resets.

**Blend duration:**

- **Blend Ratio** (0.01–0.5) — used when Auto Advance is on. The transition takes this fraction of a full day cycle, keeping transition length proportional to how fast time is moving.
- **Manual Blend Duration** — fixed real-time seconds, used whenever Auto Advance is off, or whenever Profile Mode is Manual.

Every field in `SkyVoxData` eases smoothly between the two looks, except **Cloud Voxel Size**, **Cloud Scale**, and **Fog Enabled**, which are structural switches rather than continuous values — they change at the midpoint of the blend. If Voxel Size or Scale actually differ between the two profiles, the clouds briefly dip in opacity around that midpoint to hide the switch.

**Events:** `onProfileChanged` (`UnityEvent<SkyVoxProfile>`) fires the moment a new blend target is chosen — hook it in the Inspector or from code to trigger your own effects.

---

## **API**

```csharp
using SkyVox;

SkyVoxController sky = GetComponent<SkyVoxController>();

// Jump straight to a specific hour (0–24), skipping any blend.
sky.SetTime(14.5f);

// Switch profiles by reference — safe even if list order changes.
sky.SetProfile(myEveningProfile);

// Switch by index into the Profiles list.
sky.SetProfileByIndex(2);

// Apply one of the four built-in weather presets immediately.
sky.SetWeather(SkyVoxController.WeatherPreset.Stormy);

// Smoothly animate coverage/density over time. Play mode only.
sky.TransitionWeather(toCoverage: 0.9f, toDensity: 5f, duration: 8f);

// Read current state.
SkyVoxProfile active = sky.ActiveProfile;   // null if none has been set yet
SkyVoxTimeTag tag     = sky.CurrentTimeTag; // recomputed live from Time Of Day

// React to profile changes.
sky.onProfileChanged.AddListener(profile => Debug.Log($"Now using {profile.name}"));
```

**SkyVoxController**

| Member | Description |
|---|---|
| `SetTime(float hour)` | Sets Time Of Day immediately (wraps to 0–24) and refreshes the sky. |
| `SetProfile(SkyVoxProfile profile)` | Switches to the given profile by reference. Logs a warning and does nothing if it isn't in Profiles. |
| `SetProfileByIndex(int index)` | Switches to the profile at that index in Profiles. Does nothing if out of range or null. |
| `SetWeather(WeatherPreset preset)` | Instantly sets cloud coverage, density and colors to one of `Clear`, `PartlyCloudy`, `Overcast`, `Stormy`. |
| `TransitionWeather(float toCoverage, float toDensity, float duration)` | Eases cloud coverage and density to the given values over `duration` seconds. Play mode only. |
| `ActiveProfile` (get) | The profile currently blended toward or active, or null if none has been selected yet. |
| `CurrentTimeTag` (get) | The `SkyVoxTimeTag` for the current Time Of Day, recomputed on access. |
| `onProfileChanged` | `UnityEvent<SkyVoxProfile>`, invoked whenever a new profile becomes the blend target — including automatic Linear/Shuffle switches. |
| `data` | Public `SkyVoxData` — the live look. Safe to edit directly when Profile Mode is Off; while a blend is in progress, it's overwritten every frame. |

**SkyVoxData**

```csharp
SkyVoxData copy = sky.data.Clone();             // independent copy
SkyVoxData mid  = SkyVoxData.Lerp(a, b, 0.5f);  // blend two looks yourself
```

| Member | Description |
|---|---|
| `Clone()` | Returns an independent copy of the look. |
| `Lerp(SkyVoxData a, SkyVoxData b, float t)` (static) | Returns a new `SkyVoxData` blended between `a` and `b`, using the same easing and structural-field handling used internally for profile transitions. |

**SkyVoxTimeUtil**

```csharp
SkyVoxTimeTag tag = SkyVoxTimeUtil.GetTag(19.5f); // Dusk
```

| Member | Description |
|---|---|
| `GetTag(float hour)` (static) | Returns the `SkyVoxTimeTag` for any hour, with no Controller instance required. |

**SkyVoxSetup**

| Member | Description |
|---|---|
| `FixSkyboxMaterial()` | Assigns Default Material to the scene skybox if it isn't already using the SkyVox shader, and re-syncs light references to the Controller. |
