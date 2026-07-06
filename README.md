# **SkyVox — Procedural Voxel Sky**

Procedural, texture-free skybox for Unity URP: voxel volumetric clouds, a day/night cycle, and a profile system for blending between named looks — weather, times of day — at runtime.

---

## **Getting Started**

1. Drag the `SkyVox` prefab (in `Prefabs/`) into your scene.
2. On its **Sky Setup** component, assign **Sun Light** and **Moon Light** to your scene's directional lights.
3. Click **Check / Fix Skybox Material**.
4. Move **Time Of Day** on **Sky Controller** to preview.

---

## **Setup**

`SkyVoxSetup` holds the material and light references and pushes them into `SkyVoxController` on the same GameObject automatically. Always assign lights and material through Setup, not the Controller — its reference fields are hidden for this reason.

**Check / Fix Skybox Material** checks the scene's current skybox material and replaces it with the assigned Default Material if needed.

---

## **Modifying the Sky**

All visual parameters live in `SkyVoxController.data` (a `SkyVoxData` object), editable directly in Edit Mode or Play Mode.

**Time** (on the Controller, outside Data)
- Time Of Day — 0 = midnight, 12 = noon.
- Auto Advance — advances Time Of Day every frame.
- Day Duration Seconds — real seconds per full cycle, used when Auto Advance is on.

**Light Intensity**
- Sun Intensity, Moon Intensity

**Sky Colors**
- Sky Zenith, Sky Horizon, Sky Sunset, Sky Night, Ground Color

**Sky Blending**
- Horizon Falloff — gradient sharpness.
- Sunset Band Width — how close to the horizon the sun needs to be for the sunset tint.
- Day Night Width — length of the sunrise/sunset transition; also drives the light intensity crossfade.
- Atmosphere Thickness — haze near the horizon.

**Sun**
- Sun Color, Sun Size, Sun Glow Color, Sun Glow Size, Sun Glow Falloff

**Moon**
- Moon Color, Moon Size, Moon Glow Color, Moon Glow Size, Moon Glow Falloff

**Stars**
- Stars Amount, Stars Size, Stars Intensity, Stars Twinkle, Stars Twinkle Speed

**Cloud Shape**
- Cloud Coverage, Cloud Altitude Low, Cloud Altitude High, Cloud Voxel Size, Cloud Scale — Voxel Size and Scale together set block size; changing either reshuffles which cells are solid.

**Cloud Motion**
- Cloud Speed, Cloud Wind Angle

**Cloud Appearance**
- Cloud Density, Cloud Softness, Cloud Opacity, Cloud Color Light, Cloud Color Shadow, Cloud Shadow Strength

**Cloud Quality**
- Cloud Steps — raymarch quality baseline; real step count scales with view angle internally.
- Cloud Shadow Step — minimum spacing between self-shadow samples.
- Cloud Horizon Fade — how far above the horizon clouds start fading in.

**Fog** (Unity's built-in scene fog)
- Fog Enabled, Fog Color, Fog Intensity

**Output**
- Exposure

---

## **Creating Profiles**

A `SkyVoxProfile` is a ScriptableObject: a `SkyVoxData` look tagged with one or more **Time Tags** (`Night, Dawn, Morning, LateMorning, Midday, Afternoon, LateDay, Dusk` — `[Flags]`, each covering a fixed hour range).

- **From the Controller** — tweak Data live, then click **Save As New** in the Sky Controller inspector. Creates the asset and adds it to Profiles.
- **Directly** — Project window: **Create > SkyVox > Sky Profile**.

Edit a profile's look by selecting its asset directly — same Data fields as the Controller.

**Load Target** (with a profile assigned as Target Profile) clones that profile's Data onto the live Data field.

Profiles created via the Create menu aren't added to the Profiles list automatically — drag them in for Linear/Shuffle to use them.

---

## **Profile Transitions**

**Profile Mode**
- **Off** — Data only changes by hand, `SetWeather`, or `TransitionWeather`.
- **Manual** — changes only via `SetProfile` / `SetProfileByIndex`.
- **Linear** — advances through Profiles to the next one whose Time Tags match the current tag; among matches, picks whichever has been inactive longest.
- **Shuffle** — same tag matching, picks randomly among matches.

If no profile matches the current tag, the sky keeps its current look until one does.

**Blend duration**
- **Blend Ratio** (used when Auto Advance is on) — transition takes this fraction of a full day cycle.
- **Manual Blend Duration** — fixed real-time seconds, used when Auto Advance is off or Profile Mode is Manual.

Every field blends between the two looks except **Cloud Voxel Size**, **Cloud Scale**, and **Fog Enabled**, which switch at the blend's midpoint. If Voxel Size or Scale differ between the two profiles, cloud opacity dips briefly around that midpoint to mask the switch.

`onProfileChanged` (`UnityEvent<SkyVoxProfile>`) fires whenever a new blend target is chosen.

---

## **API**

```csharp
using SkyVox;

SkyVoxController sky = GetComponent<SkyVoxController>();

sky.SetTime(14.5f);
sky.SetProfile(myEveningProfile);
sky.SetProfileByIndex(2);
sky.SetWeather(SkyVoxController.WeatherPreset.Stormy);
sky.TransitionWeather(toCoverage: 0.9f, toDensity: 5f, duration: 8f);

SkyVoxProfile active = sky.ActiveProfile;
SkyVoxTimeTag tag = sky.CurrentTimeTag;

sky.onProfileChanged.AddListener(profile => Debug.Log(profile.name));
```

| Member | Description |
|---|---|
| `SetTime(float hour)` | Sets Time Of Day, wraps to 0–24. |
| `SetProfile(SkyVoxProfile)` | Switches to a profile by reference. |
| `SetProfileByIndex(int)` | Switches to a profile by index in Profiles. |
| `SetWeather(WeatherPreset)` | Applies `Clear`, `PartlyCloudy`, `Overcast`, or `Stormy` immediately. |
| `TransitionWeather(float, float, float)` | Eases cloud coverage/density over time. Play mode only. |
| `ActiveProfile` | Current or blend-target profile, null if none set. |
| `CurrentTimeTag` | Time tag for the current Time Of Day. |
| `data` | The live `SkyVoxData`. Overwritten each frame while a blend is active. |
| `SkyVoxData.Clone()` | Independent copy of a look. |
| `SkyVoxData.Lerp(a, b, t)` | Blends two looks (static). |
| `SkyVoxTimeUtil.GetTag(hour)` | Time tag for any hour (static). |
| `SkyVoxSetup.FixSkyboxMaterial()` | Assigns Default Material to the scene skybox if needed. |
