# Cemu — Ayn Thor dual-screen fixes

Small patch set on top of [SapphireRhodonite/Cemu](https://github.com/SapphireRhodonite/Cemu)'s dual-screen Android build, focused on stability on the [Ayn Thor](https://www.ayntec.com/products/ayn-thor) (and any other Android device with a real secondary `Display`). Nothing in this fork is original emulator work — all credit for that goes to the projects below.

## Acknowledgements

- **[Cemu](https://github.com/cemu-project/Cemu)** by the Cemu team — the Wii U emulator this is built on.
- **[Android port](https://github.com/SSimco/Cemu)** by SSimco — the port of Cemu to Android that everything here depends on.
- **[Dual-screen fork](https://github.com/SapphireRhodonite/Cemu)** by SapphireRhodonite — the work that actually makes the GamePad screen show up on a second physical display via Android's `Presentation` API. Without their `PadPresentation` / external-display / screen-swap commits, none of this would exist.

This fork just adds bug fixes on top of all three.

## What's changed here

- **Dual-screen stability**: the bottom screen no longer freezes when the side menu opens, the device suspends, or you switch apps. The `Presentation` is now tied to the Activity lifecycle (`repeatOnLifecycle(RESUMED)` + `setOnDismissListener` restart), and a stale `ANativeWindow` refcount leak in the surface swap path is gone.
- **Synced with upstream**: merged ~107 commits from [SSimco/Cemu](https://github.com/SSimco/Cemu)'s `main` to pick up recent stability and compat work.
- **Paper Mario: Color Splash now boots**: shipped a default game profile to force single-core recompiler, and — more importantly — fixed a Cemu Android bug where the SAF filesystem backend would `throw` (and then `abort()` the process) on any write/create attempt instead of returning a clean error. This wasn't Color Splash specific; any title that pokes at write paths on a read-only SAF mount benefits.
- **Adreno linear-filter fallback (v0.2)**: when a texture format doesn't expose `VK_FORMAT_FEATURE_SAMPLED_IMAGE_FILTER_LINEAR_BIT` (e.g. `R32_SFLOAT` on Adreno), the Vulkan sampler is downgraded from `LINEAR` to `NEAREST` per-draw instead of producing the undefined-behavior zero-sample Adreno returns. Spec-compliance fix that helps any title sampling float buffers.
- **Minor Vulkan fix**: missing `break;` in `VulkanRenderer::GetTextureFormatInfoVK` for the `X24_G8_UINT` texture format (used by Color Splash, Resident Evil) — could matter on stricter drivers like Adreno.

## Known limitations

- **Paper Mario: Color Splash on Adreno** still has a character-rendering artifact during the paper-cutout wave animation (characters render as solid-black silhouettes / travelling black bars). The same dump works correctly on Steam Deck (RADV); current evidence points at Cemu's SPIR-V shader generator emitting more vertex output locations than Adreno's `maxVertexOutputLocations` allows. Fixing it properly requires upstream Cemu shader-gen changes. The game still boots and is playable — the artifact is cosmetic.

## APK

A prebuilt arm64-v8a release APK is on the [Releases](../../releases) page. It's signed with a debug keystore (no upstream-Cemu signature match), so if you already have a Cemu install, uninstall it first or use `adb install -r`.

Target device: **Ayn Thor** (Snapdragon 8 Gen 2, Adreno 740, Android 13). Should also work on any other Android handheld / foldable whose second screen is exposed as a separate `Display` via `DisplayManager`.

## Honest disclaimer

These fixes were authored end-to-end with [Claude Code](https://claude.com/claude-code) and validated against a single Ayn Thor + one game (Color Splash) + spot checks on Wind Waker HD. They build and run, but they have not been exhaustively tested on other titles or hardware. Treat this as a working patch set, not a production build.

If you find bugs, please file them upstream first — most of what's in here probably belongs in [SSimco/Cemu](https://github.com/SSimco/Cemu) or [SapphireRhodonite/Cemu](https://github.com/SapphireRhodonite/Cemu) rather than as a long-lived fork.

---

## About the upstream Android port

This is the Android port of **Cemu**, a Wii U emulator written in C/C++.
It is still early and experimental. Stability, performance, and features are not guaranteed, and some functionality may be missing compared to the desktop version.

There is no timeline for when this port will be finished or when new features will be implemented.

**Please do not open issues or pull requests upstream yet.**
Development is ongoing, and contributions will be welcome once the project is more stable.

For general information about Cemu, see the [official website](https://cemu.info) and [main repository](https://github.com/cemu-project/Cemu).

## License
Cemu is licensed under [Mozilla Public License 2.0](/LICENSE.txt). Exempt from this are all files in the dependencies directory for which the licenses of the original code apply as well as some individual files in the src folder, as specified in those file headers respectively.
