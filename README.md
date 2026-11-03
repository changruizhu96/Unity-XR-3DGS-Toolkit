# Unity XR 3DGS Toolkit

### 3D Gaussian Splatting for VR and Mixed Reality in Unity

[![Unity](https://img.shields.io/badge/Unity-6000.0.76f1-000000.svg?logo=unity)](https://unity.com/)
[![Meta Quest](https://img.shields.io/badge/Meta%20Quest-3-0467DF.svg?logo=meta)](https://www.meta.com/quest/quest-3/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Import and render **3D Gaussian Splatting (3DGS)** in Unity XR, or align reconstructions with the physical world for mixed reality. This is a complete Unity project, tested on **Windows PC + Meta Quest Link**.

This toolkit maintains and extends the Unity XR/3DGS tools developed for **[MR-Compare](https://github.com/changruizhu96/MR-Compare)**, a research system for aligning and visually comparing 3DGS and mesh reconstructions with real environments. Its **IEEE TVCG** paper was presented at **[IEEE ISMAR 2026](https://www.ieeeismar.net/2026/papers/)** and received a **[Best Paper Award Honorable Mention](https://www.ieeeismar.net/2026/awards/)**.

[VR Quick Start](#vr-quick-start) · [MR Quick Start](#mr-registration) · [Demo video](#demo-video) · [Features](#what-is-included) · [Requirements](#requirements)

## Demo Video

MR-Compare demo: a reconstruction aligned with a physical office and compared against passthrough.

https://github.com/user-attachments/assets/d06a137d-d7df-4e2e-936a-07cc4ce427a4

[Original video (MP4, approximately 103 MB)](https://changruizhu96.github.io/media/mr-compare/office-demo.mp4) · [Watch on the project page](https://changruizhu96.github.io/work/mr-compare/#demo)

## Get the Project

Install Git LFS, then clone:

```bash
git lfs install
git clone https://github.com/changruizhu96/Unity-XR-3DGS-Toolkit.git
cd Unity-XR-3DGS-Toolkit
git lfs pull
```

Open the project in **Unity 6000.0.76f1** and wait for packages to restore. Check the [requirements](#requirements), then follow one path below.

## Quick Start

<a id="vr-quick-start"></a>

### Option A — Use 3DGS in VR

1. Connect your Quest through Meta Horizon Link.
2. Open `Tools > Gaussian Splats > Create GaussianSplatAsset` and import your 3DGS.
3. Add a `GaussianSplatRenderer` to an object in your XR scene and assign the created asset.
4. Position the object and enter Play Mode. No room scan or registration is needed.

<a id="mr-registration"></a>

### Option B — Register 3DGS to the physical world in MR

**Prerequisite: an indoor, single-room scene.** Reconstruct the same room you will scan with Quest. **Metric scale is essential for MR**—start with a metric reconstruction or enable scale estimation.

**Choose your source workflow:**

- **Recommended fastest start — [Scaniverse](https://scaniverse.com/):** scan the room on your phone, generate a [metric-scale Gaussian splat](https://www.nianticspatial.com/products/reconstruct), and export it for Unity. This is our recommended first-run route, with no separate desktop training pipeline to set up.
- **For higher visual quality — [RealityScan](https://www.realityscan.com/download) + desktop 3DGS:** capture high-quality photos, align them in RealityScan, set metric scale using [known distances](https://rshelp.capturingreality.com/en-US/tutorials/scaling.htm), then train 3DGS from the calibrated cameras and images. Preserve that scale during export; this route gives you more control over capture and training.
- **Already using Luma AI or another non-metric end-to-end tool?** Import its 3DGS and enable `AllInOneRegistration > Estimate Scaling`. `Coarse Scale Alignment` is checked by default and becomes active with scale estimation; uncheck it for partial scans or substantially different scene coverage. Inspect the result and use **Y** to realign if needed.

For metric-scale reconstructions, leave `Estimate Scaling` disabled to preserve the source scale. It is disabled by default. Keep the correct scale when importing into Unity.

**Then scan and align with the included scenes:**

1. **Connect Quest through Meta Horizon Link.** Enable spatial data access in the PC app's Developer settings and accept the headset's spatial-data permission prompt.
2. **Scan:** open `Assets/Scenes/Room Scanner.unity`, enter Play Mode, and hold or repeatedly press the left controller's **X** while moving around the room. Release X to pause continuous scanning.
3. **Save:** press the right controller's **B**. Wait for both point-cloud and anchor UUID save confirmations in the Console, then stop Play Mode.
4. **Import:** open `Assets/Scenes/Alignment Demo.unity`. Use `Tools > Gaussian Splats > Create GaussianSplatAsset` with `Quality = Very High`, then assign the asset to the existing `3dgs` object's renderer.
5. **Bake:** under `Baking & Debug`, click **Apply Default Settings**, check that the ROI covers the room, then click **Bake & Generate Asset** and wait for completion. [ROI and filter details](#3dgs-point-cloud-preprocessing).
6. **Align:** enter Play Mode in the scanned room. The saved scan loads and alignment starts automatically. If it looks incorrect, press the left controller's **Y** to realign; multiple attempts may be needed.

Keep the supplied scene settings: scan filenames, target mode, and references are already configured. No file copying is needed between these scenes on the same PC. For other target workflows or troubleshooting, see [Detailed Usage](#detailed-usage).

### Visual Comparison Controls

After alignment, use the visual comparison modes to inspect the registered reconstruction against the real environment:

1. Press the left controller menu/start button to open or close the comparison UI.
2. Select one of the three comparison modes:
   - **Mini-Window:** move the controller or tracked hand horizontally to move the local comparison window across the scene.
   - **3D Slider:** move the controller or tracked hand horizontally to move the boundary between the reconstruction and the real environment.
   - **Switch-back:** press the right controller `B` button to switch the reconstruction display on or off.

The comparison modes can target either a `GaussianSplatRenderer` or a standard `MeshRenderer`.

## What is included

### Unity XR + 3DGS

- A modified [UnityGaussianSplatting](https://github.com/aras-p/UnityGaussianSplatting) package with XR rendering support.
- Unity tools for creating `GaussianSplatAsset` data and rendering it with `GaussianSplatRenderer`.
- Support for placing 3DGS content in OpenXR/Meta Quest scenes and integrating it with application logic.
- Editor tools for transforming, inspecting, filtering, and preparing Gaussian content.

### Extending 3DGS into mixed reality

**Non-metric 3DGS and meshes are supported:** `Estimate Scaling` estimates uniform scale, rotation, and translation—a **Sim(3) similarity transform**. Optional coarse scale initialization prepares complete, high-overlap clouds for matching.

- Source-point extraction from 3DGS centres or Unity meshes.
- Three ways to capture the physical target: reusable depth pre-scans, real-time depth scanning, or Quest's built-in environment setup.
- FPFH/TEASER++ coarse registration followed by GICP or VGICP refinement.
- Alignment save/load support using spatial anchors or the headset's scene mesh.
- Mini-Window, 3D Slider, and Switch-back interaction modes for comparing reconstructions with live passthrough.

## Integration Guide

Start from the included scenes and adapt the components your application needs:

| What you want to build | Starting point | MR registration needed? |
| --- | --- | --- |
| A VR splat viewer or an XR scene containing 3DGS | `Packages/package/Runtime/GaussianSplatRenderer.cs` and `GaussianSplatAsset.cs`; use the asset creator under `Packages/package/Editor/` | No |
| Tools to inspect or prepare splat assets | The Gaussian Splats editor tools and `Gaussian Splat Render > Baking & Debug` | No; registration-specific filtering is optional for VR-only use |
| A physical-environment point-cloud capture workflow | `Assets/Scenes/Room Scanner.unity` and `Assets/Scripts/Quest Scan/Quest3PointCloudScanner.cs` | No source registration during capture; requires Quest environment scanning setup |
| Spatially registered 3DGS or meshes in MR | `Assets/Scenes/Alignment Demo.unity` and `Assets/Scripts/Registration/All In One Registration.cs` | Yes |
| Passthrough comparison after alignment | `Assets/Scripts/Visual Comparison/` and the comparison controls in `Alignment Demo.unity` | Use aligned content for physical-world comparison |

For VR development, begin with the renderer and your XR camera/interaction setup. For MR development, replace the source reconstruction in Alignment Demo to reuse its scanner, Meta scene references, and comparison controls.

### How MR registration extends the renderer

The MR path extracts registration points from Gaussian centres or mesh geometry, collects target geometry from a depth scan or the headset's environment setup, and estimates an alignment using FPFH/TEASER++ followed by GICP/VGICP. The transform is applied to the rendered source and can be saved relative to Quest scene references or a spatial anchor.

![MR registration pipeline inherited from MR-Compare: source and target point collection, filtering, coarse registration, fine registration, and XR interaction](docs/workflow.png)

Registration pipeline from [MR-Compare](https://changruizhu96.github.io/work/mr-compare/#method). See the [demo video](#demo-video) for the in-headset result.

## Requirements

- Git and Git LFS for the bundled binary assets and native libraries
- Unity `6000.0.76f1`
- Meta Quest 3 or compatible Meta XR device
- Meta Quest Link-based development and testing workflow
- Current tested PC runtime: **Meta Horizon Link `208.0.0.84.535`** (previous Meta Quest Link versions tested: `v77`, `v85`, `v201`, and `v203`)
- Meta XR Core SDK `203.0.0`
- Meta MR Utility Kit `203.0.0`
- Meta XR Interaction SDK OVR `201.0.0`
- Meta OpenXR `2.5.1`
- Unity OpenXR `1.16.1`
- Universal Render Pipeline `17.0.4`
- Windows PC for the native `x86_64` registration backend, connected to Quest through Link
- [Microsoft Visual C++ Redistributable v14, x64](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170) for the native registration DLLs

Use compatible Quest headset and PC Link versions. We recommend the general-release Link channel with **Public Test Channel (PTC)** disabled.

Native registration depends on DLLs under:

```text
Assets/Plugins/x86_64/
```

These include PCL-related libraries and `TeaserDll_final.dll`.

## Repository Layout

```text
Assets/Scenes/
  Room Scanner.unity        Quest point cloud scanning scene
  Alignment Demo.unity      Registration and visual comparison scene

Assets/Scripts/Quest Scan/
  Quest 3 point cloud scanning and spatial-anchor-based saving

Assets/Scripts/Registration/
  Source/target loading, TEASER++/GICP/VGICP registration, alignment save/load

Assets/Scripts/Visual Comparison/
  Registered object management and in-headset comparison modes

Assets/Plugins/x86_64/
  Native registration dependencies

Packages/package/
  Modified Unity Gaussian Splatting package
```

## Preparing Your Source Scene

For MR registration, import a reconstruction of the physical environment you will scan. For a VR viewer, follow [VR Quick Start](#vr-quick-start).

Supported source types:

- **3D Gaussian Splatting**: use `Tools > Gaussian Splats > Create GaussianSplatAsset`, select `Quality: Very High`, create the asset, and assign its `GaussianSplatRenderer` to the aligner. Very High provides the uncompressed positions used for registration. The importer defaults to this quality on first use and remembers later choices.
- **Mesh**: import a reconstructed mesh and assign its `MeshFilter` to the aligner.

The current Inspector enum uses the spelling `GaussianSplating`; this refers to the 3D Gaussian Splatting source mode.

### Source Preparation Routes

Prepare a reconstruction of the physical environment you intend to scan and align:

- **Recommended fastest start — [Scaniverse](https://scaniverse.com/):** capture and process a [metric-scale Gaussian splat](https://www.nianticspatial.com/products/reconstruct) directly on a phone, then export it for Unity. This is the recommended first-run route.
- **Desktop route with metric calibration — [RealityScan](https://www.realityscan.com/download):** [align the source images](https://rshelp.capturingreality.com/en-US/tutorials/quickstart_2.htm), establish metric scale with [control points and distance constraints](https://rshelp.capturingreality.com/en-US/tutorials/scaling.htm), then export a mesh or pass the calibrated cameras and images to a desktop 3DGS pipeline.
- **Non-metric end-to-end workflow — e.g. Luma AI:** if the exported reconstruction uses arbitrary units, enable `AllInOneRegistration > Estimate Scaling`. For complete source and target clouds with high overlap, optional `Coarse Scale Alignment` initializes scale before the first applied alignment; later Y realignments skip this initialization.

### Source Scale Requirement

The most important requirement for 3DGS registration is scale. The 3DGS scene should be reconstructed at real-world scale, or at least be close enough that TEASER++ can estimate a usable scale correction.

For a desktop 3DGS handoff, preserve RealityScan's calibrated camera poses and metric scale when moving the images into a dedicated generator such as LichtFeld Studio, Nerfstudio's gsplat workflow, or Postshot.

Keep `Estimate Scaling` disabled for metric-scale input; enable it for non-metric input to estimate scale along with rotation and translation. `Coarse Scale Alignment` is checked by default, but only takes effect when `Estimate Scaling` is enabled. It estimates an initial size ratio before native downsampling and feature matching when the filtered source and metric target cover the same complete scene with high overlap. Partial scans, substantially different coverage, and strongly different sampling densities can bias this ratio; uncheck coarse scaling and prepare the source near metric scale manually in those cases. See `Coarse Scale Sample (%)` below for sampling settings.

### 3DGS Point Cloud Preprocessing

Before matching a 3DGS source against the Quest target, clean the source points used for registration. The goal is to extract a stable, mostly foreground point cloud from the 3DGS asset, instead of registering against floaters, skyboxes, background geometry, or low-quality splats.

Use the `Baking & Debug` section on the `Gaussian Splat Render` component.

Important settings:

- `Apply Default Settings`: restores the indoor single-room bake preset without starting a bake. It enables Auto Flatness, density filtering, and ROI; resets minimum opacity to `0`, grid spacing to `0.05`, minimum neighbors to `2`, and coordinate normalization to off; and restores a 10 × 10 × 10 box centered on the renderer's origin, with no center override or offset. Apply it before any scene-specific adjustments.
- `Max Flatness`: defaults to Auto. Clicking Bake calculates the median splat scale ratio before filtering. Open `Advanced Bake Settings` and disable `Auto Max Flatness` to enter a manual threshold.
- `Density Filter`: defaults to on, with grid spacing `0.05` and minimum neighbors `2`. Adjust these under `Advanced Bake Settings` to suit the source's units and point density.
- `Enable Selection ROI`: enabled by default. It restricts baked points to the room of interest using a 10 × 10 × 10 world-unit box centered on the renderer's origin. Check its coverage for large, offset, or non-metric scenes. To restore this setup in an existing scene, click **Apply Default Settings** before adjusting the box.

Auto Flatness runs when Bake is clicked. The displayed `Capture + Save` time covers filtering and saving; Auto Flatness is calculated beforehand.

Recommended preprocessing steps:

1. Select the 3DGS object in Unity.
2. Open the `Gaussian Splat Render` component.
3. Go to `Baking & Debug` and click **Apply Default Settings** to restore the indoor single-room preset. Check that the ROI contains the room; adjust its size or center under `Advanced Bake Settings` if necessary.
4. Click **Bake & Generate Asset** once. The renderer receives the generated baked asset automatically.
5. Inspect the baked point cloud; adjust the advanced filters and rebake only if relevant geometry is missing or unwanted background remains.
6. Use that renderer as the `AllInOneRegistration` source with `Use Baked Data` enabled. Alignment Demo already has these references assigned for its existing `3dgs` object.

If registration fails, inspect the source and target point clouds for:

- a large source/target scale difference;
- unwanted background or skybox points;
- insufficient overlap;
- noise affecting FPFH correspondences;
- TEASER++ voxel size or GICP/VGICP settings that need adjusting for the scene scale.

## Choosing a Registration Target

The toolkit supports three target-capture workflows:

- **Depth pre-scan (recommended Quick Start):** capture and save a target in `Room Scanner.unity`, then reuse it in `Alignment Demo.unity`.
- **Real-time depth scan and align:** capture target points and align the reconstruction in the same scene and session.
- **Quest 3 built-in environment setup:** complete `Settings > Environment Setup > Space Setup` directly on the headset. Quest assists with scanning the room, and the toolkit uses the resulting scene mesh without a separate toolkit depth scan. See [Meta's official assisted Space Setup guide](https://www.meta.com/help/quest/853707093070890/).

These workflows map to four `Target Format` modes in `AllInOneRegistration`. Quest environment setup uses `effectMesh`, or the legacy `roomMesh` option for older Meta scene components.

`preScan` and `realTimeScan` use the same underlying target-generation and registration path: the Quest Environment Depth scanner produces a point cloud, which is then aligned with the reconstructed source through the same TEASER/GICP pipeline. The difference is how the workflow is organized. `preScan` saves the scan and anchor for a later alignment session, while `realTimeScan` passes the scanner points directly to alignment in the same scene and session.

When `Allow Repeated Alignment` is enabled, pressing `Y` after the initial alignment reruns registration with the retained target data in `preScan`, `roomMesh`, and `effectMesh` modes. In `realTimeScan`, X can add scan coverage before Y runs the next alignment. Repeated alignment refreshes the source point cloud in its current pose, so each pass acts as an incremental refinement.

The included **Alignment Demo is preset to `preScan`** with `alignment_scan.bytes` / `alignment_scan.txt`, the scan anchor and point-cloud VFX references, `SpatialAnchor` alignment saving, and the existing `3dgs` source renderer. It is ready to load the output of Room Scanner once you import and bake your own source. Its embedded scanner and Meta scene references remain available if you explicitly switch to another target mode.

For custom scenes, the `AllInOneRegistration` defaults are `Align`, `effectMesh`, `GaussianSplating`, TEASER rough alignment, GICP refinement, baked source data, repeated alignment enabled, and automatic alignment saving disabled. Choose the target mode and assign its required references, or start from Alignment Demo for the prepared Pre-Scan workflow.

| Target format | Recommended use | Target data | Main tradeoff |
| --- | --- | --- | --- |
| `preScan` | Separated, reusable scan/alignment workflow | Environment Depth scanner point cloud loaded with its saved anchor data | Requires a separate scan/save/load stage |
| `realTimeScan` | Integrated scan/alignment workflow | The same Environment Depth scanner point cloud passed directly from memory | Fewer handoff steps; scanner and 3DGS rendering must be isolated at runtime |
| `effectMesh` | Quest built-in environment setup; recommended for new scenes using this workflow | Global scene mesh exposed through Effect Mesh | No separate toolkit scan; target detail and accuracy depend on the headset's captured mesh |
| `roomMesh` (legacy) | The same built-in environment setup workflow in older MRUK scenes | Scene mesh exposed through Room Mesh Event | Retained for compatibility; use `effectMesh` for new scenes |

## Detailed Usage

### Quest Environment Setup Workflow

Capture the room through Quest's environment setup, then use its global scene mesh as the `effectMesh` registration target.

1. Complete `Settings > Environment Setup > Space Setup` directly on the Quest headset, following [Meta's official assisted Space Setup guide](https://www.meta.com/help/quest/853707093070890/).
2. Open `Assets/Scenes/Alignment Demo.unity`.
3. Add or enable your reconstructed source object in the scene.
4. Select the object that contains `AllInOneRegistration`.
5. Set `Workflow Mode` to `Align`.
6. Confirm that the source 3DGS or mesh is close to real-world scale.
7. If using 3DGS, preprocess the registration points with `Baking & Debug`.
8. Set `Target Format` to `effectMesh`.
9. Assign the `Effect Mesh Event Target`.
10. Make sure the effect mesh configuration includes the global mesh.
11. Set `Source Type` to either `GaussianSplating` or `Mesh`.
12. Assign the matching source reference.
13. Configure the selection region if you only want to align part of the source model.
14. Choose registration parameters:
    - use `Teaser` when the initial offset is large;
    - use GICP/VGICP refinement when the source and target already have reasonable overlap.
15. Enable `Is Saving` if you want to reuse the result later.
16. Set `Alignment Reference Mode` to `EffectMesh`.
17. Run the scene and wait for the effect mesh target to load.
18. After registration completes, inspect the registered source with the visual comparison tools.

This workflow avoids a separate toolkit depth scan by using the headset's captured scene mesh.

### Environment Depth Scanning Performance

Both included scenes use `Scan Interval = 0.1` seconds: holding X runs one scan, waits 0.1 seconds, and repeats while X remains held (roughly at most 10 scans/second at normal time scale; scan time and frame scheduling lower the actual rate). Repeated presses share the same scan coroutine rather than starting parallel scans. Very rapid taps may not each produce a separate scan.

At the scene preset `Density = high (128)`, each scan synchronously calls raycast 128 × 128 = **16,384 times** before yielding. Keyframe selection happens after capture, so its 0.35-second minimum interval does not throttle this work. Increase `Scan Interval` to scan less frequently, or reduce `Density` to lower the work per scan.

With Meta Horizon Link `208.0.0.84.535`, we observed GPU utilization reaching approximately **90% during Environment Depth scanning**, with frame drops during target capture. We suspect a Meta Depth API/runtime issue.

In the included Alignment Demo, pressing **Y** to finish target capture and begin registration automatically deactivates the scanner and its `EnvironmentDepthManager`, shutting down the Depth API. The scan-related slowdown was no longer observed during subsequent alignment and visual comparison in this setup. Pressing **X** to resume scanning reactivates depth capture and may bring back the slowdown. Loading a saved Pre-Scan target for alignment does not require active depth capture.

### Separated Workflow: Pre-Scan / Quest 3 Scanner

Room Scanner saves the target points and spatial anchor; Alignment Demo loads them for registration. Use this workflow to reuse a saved room scan across alignment sessions.

1. Open `Assets/Scenes/Room Scanner.unity`.
2. Run the scene on Quest.
3. Hold or repeatedly press `X` to scan the physical environment.
4. Press `B` to create a spatial anchor and save the scanned point cloud. With the included scene's defaults, this writes `alignment_scan.bytes` and `alignment_scan.txt` to `Application.persistentDataPath`. Wait for both point-cloud and anchor-file save confirmations before stopping Play Mode.
5. Open `Assets/Scenes/Alignment Demo.unity`.
6. Keep the demo's `Workflow Mode = Align` preset.
7. Keep its `Target Format = preScan` preset; choose this mode explicitly only when configuring a custom scene or switching back from another workflow.
8. Keep `Point Cloud Target File = alignment_scan.bytes` and `Anchor Uuid File = alignment_scan.txt`; if you renamed either file in the scanner, enter the matching name here.
9. Keep the included scene's anchor-holder and point-cloud VFX references, or assign them when building a custom scene.
10. Configure the source type and source object.
11. Use `SpatialAnchor` as the alignment reference mode if you want the result to persist relative to the saved scan anchor.
12. Run registration.

Both scenes use the same `Application.persistentDataPath` on the same PC. To keep multiple scans, change the filename pair in Room Scanner and enter the matching names in Alignment Demo; saving to an existing filename replaces that scan. Before stopping the scanner, wait for both `Successfully saved ... points to ...` and `Anchor UUID saved to ...` in the Console.

If loading fails, check the filename pair and return to the scanned room so the anchor can localize. If alignment looks incorrect, press **Y** to realign. In our testing, several realignments resolved some initially unsuccessful alignments.

The scanner stores:

- point cloud data;
- spatial anchor UUID data.

### Real-Time Scan Workflow

This workflow integrates the same Environment Depth scanner and point-cloud registration path used by `preScan` into one scene and session. Instead of saving and reloading the target, it passes the accumulated scanner points directly from memory to alignment. The toolkit suspends 3DGS rendering while scanning and deactivates the scanner before alignment. Prefer **Pre-Scan / Quest 3 Scanner** when you want a reusable saved target or separate scan/alignment stages; use **Quest Environment Setup** when you want the headset to prepare the scene mesh without a separate toolkit scan.

1. Open `Assets/Scenes/Alignment Demo.unity`.
2. Set `Target Format` to `realTimeScan`.
3. Enable `Allow Repeated Alignment` if the user should be able to refine the result with additional X/Y cycles.
4. Keep the demo's assigned `Scanner Target`, or assign your scanner when building a custom scene.
5. Run the scene on Quest.
6. Press or hold `X` to start scanning, depending on the active scanner setup.
7. Press `Y` to extract scanner points and trigger registration.
8. If the alignment is not satisfactory, press `X` again. 3DGS rendering is suspended before the embedded scanner is reactivated, and new scan points are added to the retained scan. Press `Y` again to align against the updated target.

## Registration Settings Guide

### TEASER++ Coarse Registration

Enable TEASER++ when the source starts far from the target, the initial rotation is uncertain, or the correspondence set contains outliers. It provides a robust coarse alignment before local refinement.

Tune these parameters first:

- `Voxel Size Teaser`: larger values improve robustness and speed at the cost of detail.
- `Noise Bound Teaser`: increase this for noisier extracted points.
- `Normal Radius Teaser` and `FPFH Radius Teaser`: set these relative to the physical scene scale.
- `Estimate Scaling`: disabled by default. Keep it disabled for metric-scale input; enable it for non-metric input to estimate the residual scale difference.
- `Coarse Scale Alignment`: checked by default, but only active when `Estimate Scaling` is enabled. Estimates an initial size ratio from each cloud's 90th-percentile radius about its approximate geometric median, then scales the source points before feature matching. TEASER++ estimates the remaining scale difference. Use this for complete clouds with high overlap; uncheck it for partial scans or substantially different coverage and prepare the source near metric scale manually.
- Coarse scaling runs before the first applied registration in a scene session. Later **Y** realignments refine the current source pose through TEASER++/GICP. Restart the scene to initialize coarse scale again.
- `Coarse Scale Sample (%)`: percentage of each cloud used for the estimate. Default: 5%; Alignment Demo preset: 20%. The range is 0.1–100%, with 100% using all points. Sampling uses a fixed seed and index strata. Higher percentages reduce sampling variation at greater CPU and memory cost; different coverage or point densities can still bias the size ratio.

Coarse preparation runs on a CPU worker using the existing input buffer. Logs show the coarse, residual, and combined scales, the transformation matrix, and preparation time. Invalid scale or matrix data stops the coarse-scaling path before applying a transform. A finite native candidate is applied even when native convergence is false, and saved when saving is enabled; inspect the resulting pose and use **Y** to refine it.

### GICP/VGICP Fine Registration

Use GICP/VGICP after coarse alignment, once source and target overlap. VGICP can be more stable for large or noisy point clouds; smaller voxels favour detail, while larger voxels favour speed and robustness.

## Alignment Save/Load Workflow

`AllInOneRegistration` supports two workflow modes:

- `Align`: run registration and optionally save the alignment result.
- `Load`: load a previously saved alignment result and apply it to the source object.

Supported alignment reference modes:

- `SpatialAnchor`: save/load the alignment relative to a spatial anchor.
- `RoomMesh` (legacy): save/load the alignment relative to the legacy room mesh.
- `EffectMesh`: save/load the alignment relative to the current effect mesh. This reference can be used with any registration target format because it is resolved independently from the point cloud or mesh used for registration.

Use `EffectMesh` when you want the alignment to be tied to the Quest's scanned scene mesh rather than to a separately scanned point cloud. The scene must contain an `EffectMeshEvent` configured to provide the global mesh; `Alignment Demo.unity` already includes this setup.

To load a saved alignment:

1. Keep the same source type and source object assignment used during alignment.
2. Set `Workflow Mode` to `Load`.
3. Select the same `Alignment Reference Mode` used when saving.
4. Assign the matching reference: anchor files and holder for `SpatialAnchor`, Room Mesh Event for `RoomMesh`, or Effect Mesh Event for `EffectMesh`.
5. Set the saved alignment file name and run the scene.

The transform is restored relative to the selected reference rather than as a fixed world-space pose.

## Third-Party Components

This project includes modified or adapted work from the following projects:

- 3D Gaussian Splatting support is modified from [aras-p/UnityGaussianSplatting](https://github.com/aras-p/UnityGaussianSplatting).
- The Quest 3 scanner workflow is modified from [Appletea0673/Depth-Scanner-Project](https://github.com/Appletea0673/Depth-Scanner-Project).

Additional runtime functionality depends on Meta XR SDK, Meta MR Utility Kit, Unity OpenXR packages, and native registration libraries.

See [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) for bundled copyright and license notices, including the native TEASER++, PCL, and LZ4 binaries. Package-manager dependencies remain subject to their own package licenses and terms.

## Research Origins

The toolkit builds on **[MR-Compare](https://github.com/changruizhu96/MR-Compare)**, extending its registration, rendering, and comparison tools for Unity XR development.

**MR-Compare: A Mixed-Reality Framework for Spatially Grounded Visual Comparison of 3D Gaussian Splatting and Mesh Reconstructions with the Physical Environment**

**IEEE ISMAR 2026 · TVCG Paper** · [Best Paper Award · Honorable Mention](https://www.ieeeismar.net/2026/awards/)

[Research repository](https://github.com/changruizhu96/MR-Compare) · [Paper on arXiv](https://arxiv.org/abs/2607.20325) · [Project page and demo](https://changruizhu96.github.io/work/mr-compare/)

## Citation and Acknowledgements

If this toolkit contributes to an academic publication, please cite its research foundation, the [MR-Compare paper](https://arxiv.org/abs/2607.20325):

```bibtex
@misc{zhu2026mrcompare,
      title={MR-Compare: A Mixed-Reality Framework for Spatially Grounded Visual Comparison of 3D Gaussian Splatting and Mesh Reconstructions with the Physical Environment},
      author={Changrui Zhu and Ernst Kruijff and Pengju Zhang and Simon Julier},
      year={2026},
      eprint={2607.20325},
      archivePrefix={arXiv},
      primaryClass={cs.GR},
      url={https://arxiv.org/abs/2607.20325},
}
```

Please also acknowledge the upstream projects and cite the reconstruction and registration methods used in your work.

## License

[MIT License](LICENSE). Copyright (c) 2026 Changrui Zhu. Retain the copyright and permission notice when redistributing the software.

Third-party components retain their respective licenses; see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) and the [Unity Gaussian Splatting package license](Packages/package/LICENSE.md).
