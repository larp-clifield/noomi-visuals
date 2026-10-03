# NoomiClone Visual Modding Toolkit

Phone-first, offline research scripts for inspecting the Unity assets in an Android **NoomiClone** build using **Termux, Debian (proot-distro), and UnityPy**.

This is an unofficial toolkit—not a game, APK, asset pack, or one-click installer. It is intended to help a user inspect their own game copy, export preview images, trace material references, and validate local test patches.

> **Important:** Bring your own legally obtained APK/Data. This repository should contain scripts and documentation only. Do not upload or redistribute the game APK, extracted textures/audio, `libil2cpp.so`, `global-metadata.dat`, Unity `.assets` files, or a combined dump archive.

## What is here

- `scripts/noomi_scan.py` — read-only inventory of Unity containers and selected objects; can export Texture2D and Sprite PNG previews.
- `scripts/noomi_green.py` — experimental, size-preserving test patch for the serialized `noomi` Material color. It writes new split files to a separate folder and never edits the input Data directory or creates a complete APK.
- `scripts/noomi_check_apk.py` — read-only check of the color and split files inside a rebuilt APK.
- `scripts/noomi_trace_refs.py` — read-only trace of character MeshRenderer material references in `level1` and `level19`.
- `scripts/noomi_shader_check.py` — read-only inspection of the `noomi` shader's serialized properties in an APK.
- `scripts/noomi_extract_il2cpp.py` — extracts IL2CPP analysis inputs from a user-supplied APK for local analysis. The extracted binaries are not part of this repository.

## Tested target and current findings

The initial target inspected was an Android NoomiClone build using Unity `6000.3.9f1`; the scripts were tested with UnityPy `1.25.3` in Debian under Termux on Android ARM64.

The read-only inventory found 155 containers and 1,053 selected objects: 125 Texture2D, 124 Sprite, 66 Material, 22 Camera, 22 RenderSettings, 693 MeshRenderer, and 1 AudioClip. These counts describe that one tested build; they are not guaranteed for other versions.

The character renderers in `level1` and `level19` reference `sharedassets1.assets` Material `noomi` (PathID 3). That material's shader exposes `_Color`. A green test value was verified inside a rebuilt APK, but changing the serialized material alone did **not** turn the in-game character green. IL2CPP metadata for the tested build contains a `NoomiPalette`, a `NoomiSkinNum` preference, and `SingleColorCustomizationItem.Apply(GameObject)`. The game has a runtime skin/customization system that can apply a saved skin color over the asset's starting color. In-game appearance must be tested; an asset-only color edit is not a guaranteed runtime override.

## PNG preview export

With the existing Termux/Debian environment and UnityPy venv:

```bash
source ~/unity-env/bin/activate
python scripts/noomi_scan.py /path/to/Data \
  --out /path/to/unity_scan_png --png --sprites
```

For the phone layout used during testing:

```bash
python /mnt/phone/Download/noomi_scan.py \
  /mnt/phone/visualels/Data \
  --out /mnt/phone/visualels/unity_scan_png \
  --png --sprites
```

`--png` exports Texture2D previews; adding `--sprites` also exports Sprite previews. The output must be outside the source `Data` folder. Some compressed/HDR texture formats are converted for PNG preview, so the exported preview is not necessarily a lossless reimport file. The script keeps existing PNG previews rather than overwriting them; use a fresh output folder for a clean export.

## Test the material-color patch

```bash
python scripts/noomi_green.py /path/to/Data \
  --out /path/to/green_patch --color '#00FF00'
python scripts/noomi_check_apk.py /path/to/rebuilt.apk \
  --patch /path/to/green_patch
```

The patch script only creates replacement asset files. It does not update an installed app, rebuild/sign an APK, or change the original Data folder. Keep a full backup, preserve every Unity split part, and test offline.

## Phone workflow

The scripts expect Python with `UnityPy==1.25.3` already installed in the Debian `unity-env` environment. On Termux, enter the Debian container with the storage bind used for the test device, then activate the venv:

```bash
proot-distro login --bind "$HOME/storage/shared:/mnt/phone" debian
source ~/unity-env/bin/activate
```

The exact Data path and build layout vary by device and game version. Do not assume an asset filename or PathID is universal.

## Public-repository hygiene

Keep game-owned binaries and generated dumps out of Git history. `.gitignore` excludes common local outputs, but it does not untrack a file that was already committed. If a proprietary archive has already been pushed publicly, remove it from the repository and consider making the repository private or recreating it cleanly; deleting a file in a later commit does not erase earlier Git history or copies made by others.

No license is included yet. Choose a license only for original scripts/documentation you have the right to license; that license cannot grant rights to NoomiClone or Unity-owned files.

## Русский

Это несуществующий «конвейер атласов»: репозиторий посвящён **офлайн-анализу Unity-ресурсов NoomiClone с телефона через Termux/Debian/UnityPy**, PNG-превью, трассировке ссылок на материалы и проверке локальных тестовых патчей. Сам APK и файлы игры в публичный репозиторий включать не следует.
