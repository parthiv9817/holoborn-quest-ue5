# HoloBorn — Quest 3 Client (UE5)

Mixed-reality avatar generator for Meta Quest 3, built in Unreal Engine 5.

User presses a button on the Quest controller → headset bursts 5 photos via the passthrough camera → the photos are uploaded to the Mac backend → the backend portraitizes and ships them to a RunPod GPU → TRELLIS generates a 3D GLB → the headset downloads the GLB and spawns it as a hologram anchored in the room.

This is the **client** — the UE5 app that runs on the headset. The backend lives separately at [`parthiv9817/holoborn-server`](https://github.com/parthiv9817/holoborn-server).

## Status

🪦 **ARCHIVED / ABANDONED (2026-05-04). This path is dead — do not revive it.**

The UE5 migration was abandoned after it hit an **unfixable engine regression**: on Intel-Mac +
UE 5.5.4, the standalone Quest APK fails at engine init with `Missing global shader
FScreenPSsRGBSourceMipLevelArray's permutation 0` — a binary cooker regression for
`VULKAN_ES31_ANDROID` that **no `.ini` setting reaches** (verified across 10+ config-matrix builds).
There are zero documented Intel-Mac + UE5 + standalone-Quest successes globally, and Meta itself
routes Mac Quest devs to Unity. The project pivoted to Unity on 2026-05-05; the shipped client lives
at **[parthiv9817/holoborn-quest-unity](https://github.com/parthiv9817/holoborn-quest-unity)**.

Kept for historical reference only (toolchain notes, plugin recon, version pins below). Immersive VR
*did* run on-device (Build #5) before the cooker wall — the blocker was purely the shader cook.

The original Unity client was lost when the dev Mac died on ~April 25, 2026 (compiled APK survived on
the headset, source did not). Founder push + lost code + Vision Pro on the longer roadmap → UE5 was
attempted instead of an immediate Unity rebuild. It didn't work out (see above).

The migration roadmap and decision history live in the backend repo's diaries:

- [`diaries/ue5-roadmap.md`](https://github.com/parthiv9817/holoborn-server/blob/main/diaries/ue5-roadmap.md) — 8-phase roadmap, exit gates, predicted blockers
- [`diaries/ue5-vs-unity-brief.md`](https://github.com/parthiv9817/holoborn-server/blob/main/diaries/ue5-vs-unity-brief.md) — full UE5-vs-Unity decision brief
- [`diaries/2026-04-28.md`](https://github.com/parthiv9817/holoborn-server/blob/main/diaries/2026-04-28.md) — the day the migration was committed

## Architecture (target)

```
Quest 3 (UE5 app, this repo)
  │  A button → burst 5 JPEG frames from passthrough camera
  │  POST /generate-multiview (multipart, 5 JPEGs + metadata)
  ▼
holoborn-server (Mac, FastAPI, separate repo)
  │  burst average → portraitize → RunPod GPU → TRELLIS GLB
  ▼
Quest 3
  │  poll /generate/{task_id}/status until complete
  │  GET /avatars/{task_id}.glb
  │  load via glTFRuntime → spawn AglTFRuntimeAssetActor
  │  anchor to floor in passthrough
```

## Pinned versions

| Tool | Pin | Reason |
|------|-----|--------|
| Unreal Engine | **5.5.4** | Meta-tested combo with MetaXR v78 |
| MetaXR plugin | **v78** | Official Meta build for UE 5.5 |
| Android NDK | **27.2.12479018** | Pinned per Epic's `SetupAndroid.command` (5.5+ correct) |
| JDK | **OpenJDK 21.0.3** | Bundled with Android Studio Koala JBR |
| Android Studio | **Koala 2024.1.1 Patch 2** | Last fully Intel-Mac-supported stable Koala |
| SDK Platform | **android-34** | Quest store target |
| Build-Tools | **35.0.1** | Latest stable for Quest store rules |
| Min SDK | **29** | Quest store rules |
| Target SDK | **34** | Quest store rules |
| Xcode | **16.2** | Mac Metal compile, last fully Intel-supported version |

## Plugin layout (target — not yet populated)

```
HoloBornUE5/
├── HoloBornUE5.uproject
├── Source/HoloBornUE5/             # game code (mostly Blueprint, one C++ class for HTTP multipart)
└── Plugins/
    ├── glTFRuntime/                 # github.com/rdeioris/glTFRuntime — runtime GLB loading (Epic MegaGrant)
    └── AndroidCamera2Plugin/        # parthiv9817/UnrealAndroidCamera2Plugin — fork of tark146 (1280x960 + BT.601 fix)
```

The plugin pattern in UE5 is "drop the source folder into `Plugins/`, the engine compiles it into your project's binary at build time" — different from Unity's pre-compiled `.unitypackage` model.

## Build target

- **Quest 3 standalone** (Snapdragon XR2 Gen 2, ARM64, AOSP / Meta Horizon OS)
- Mobile Forward Renderer
- Mobile HDR off
- MSAA 4x
- Output: `*.apk` sideloaded via `adb install` or Meta Quest Developer Hub

## License

MIT (or whatever the founder picks closer to ship).
