# Unity XR 3DGS Toolkit

### 3D Gaussian Splatting for VR and Mixed Reality in Unity

[![Unity](https://img.shields.io/badge/Unity-6000.0.76f1-000000.svg?logo=unity)](https://unity.com/)
[![Meta Quest](https://img.shields.io/badge/Meta%20Quest-3-0467DF.svg?logo=meta)](https://www.meta.com/quest/quest-3/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Build Unity XR experiences with **3D Gaussian Splatting (3DGS)**. This self-contained Unity project brings together splat import, XR rendering, editor tools, example scenes, and interaction components. Start with a **VR 3DGS viewer**, integrate splats into your own XR scene, or extend the same content into **mixed reality** with physical-world registration, passthrough comparison, and spatial anchors.

Originally developed for **[MR-Compare](https://github.com/changruizhu96/MR-Compare)**, this is the **actively maintained toolkit repository** for future updates and new features. You do not need to use the paper's comparison workflow to use 3DGS in VR. For the research system, evaluation, and publication, see [Research Origins](#research-origins).

**Start here:** [Demo video](#demo-video) · [Get the Project](#get-the-project) · [VR Quick Start](#vr-quick-start) · [MR Registration](#mr-registration) · [Integration Guide](#integration-guide) · [Requirements](#requirements)

- **Project or package?** A complete Unity project with integrated packages and example scenes, not a standalone UPM-installable toolkit package.
- **Does it require a PC?** Windows PC + Meta Quest Link is the tested workflow. MR registration uses Windows `x86_64` native libraries and does not run entirely on the headset.
- **Can I use VR without MR registration?** Yes. Import and render 3DGS in an XR scene without physical-world scanning or registration.

## Demo Video

See the MR-Compare application that the toolkit grew out of: reconstructions registered to a physical office, switching between reconstruction and live passthrough, and spatially grounded comparison with the 3D Slider. This demonstrates the MR comparison workflow; physical-world registration is not required for VR-only use.

https://github.com/user-attachments/assets/d06a137d-d7df-4e2e-936a-07cc4ce427a4

[Original video (MP4, approximately 103 MB)](https://changruizhu96.github.io/media/mr-compare/office-demo.mp4) · [Watch on the project page](https://changruizhu96.github.io/work/mr-compare/#demo)

Play the video directly above. The recording shows the MR-Compare research application, not a separate evaluation of later toolkit releases.

## Use the toolkit in two ways

| Unity XR + 3DGS (VR) | Spatially grounded 3DGS (MR) |
| --- | --- |
| Import and render 3DGS assets in Unity XR | Keep the complete Unity XR rendering workflow |
| Place splats in VR scenes and application logic | Register 3DGS or meshes to the physical environment |
| View and interact with splats through an XR camera | Use depth scans or Quest's built-in environment setup as registration targets |
| Use the project without physical-world registration | Persist alignment with Quest coordinates or spatial anchors |
| Build custom VR interactions around `GaussianSplatRenderer` | Compare registered content with passthrough using Mini-Window, 3D Slider, or Switch-back |

## What is included

### Unity XR + 3DGS

- A modified [UnityGaussianSplatting](https://github.com/aras-p/UnityGaussianSplatting) package with XR rendering support.
- Unity tools for creating `GaussianSplatAsset` data and rendering it with `GaussianSplatRenderer`.
- Support for placing 3DGS content in OpenXR/Meta Quest scenes and integrating it with application logic.
- Editor tools for transforming, inspecting, filtering, and preparing Gaussian content.

### Extending 3DGS into mixed reality

**Scale-aware registration for non-metric reconstructions:** enable `Estimate Scaling` to align 3DGS or mesh sources expressed in arbitrary units to a metric physical-world target. The registration estimates a uniform scale together with rotation and translation—a **Sim(3) similarity transform**. Optional one-time coarse scale initialization helps prepare complete, high-overlap clouds for feature matching. Alignment quality still depends on geometry, overlap, and parameter settings.

- Source-point extraction from 3DGS centres or Unity meshes.
- Three ways to capture the physical target: reusable depth pre-scans, real-time depth scanning, or Quest's built-in environment setup.
- FPFH/TEASER++ coarse registration followed by GICP or VGICP refinement.
- Alignment save/load support using spatial anchors or the headset's scene mesh.
- Mini-Window, 3D Slider, and Switch-back interaction modes for comparing reconstructions with live passthrough.

## Get the Project

Install Git LFS before cloning so the bundled images and native libraries are downloaded:

```bash
git lfs install
git clone https://github.com/changruizhu96/Unity-XR-3DGS-Toolkit.git
cd Unity-XR-3DGS-Toolkit
git lfs pull
```

Open the repository root in Unity `6000.0.76f1`, let the package manager restore dependencies, and check the [requirements](#requirements). Choose the VR-only or MR registration path below. This is a complete project to open and adapt, not a single package to install through a UPM Git URL.

## Quick Start

<a id="vr-quick-start"></a>

### Option A — Use 3DGS in VR

1. Follow [Get the Project](#get-the-project) and open it in Unity `6000.0.76f1`.
2. Let Unity restore the dependencies in `Packages/manifest.json`.
3. Open `Tools > Gaussian Splats > Create GaussianSplatAsset` and import your 3DGS source.
4. For VR-only rendering, select the asset quality appropriate for your application. Use `Very High` if the same asset will later be used for MR registration, because registration reads the uncompressed Gaussian positions.
5. Add a `GaussianSplatRenderer` to an object in an XR-enabled scene and assign the created asset.
6. Position the object as normal Unity scene content and run the project through the configured OpenXR/Meta Quest workflow.
7. Integrate the renderer with your own XR interaction and application logic as needed. Physical-world scanning and registration are not required for VR-only use.

<a id="mr-registration"></a>

### Option B — Register 3DGS to the physical world in MR

**Prerequisite: an indoor, single-room scene.** This Quick Start and its default filtering preset assume your reconstruction and depth scan cover the same room with high overlap. The default selection ROI is a 10 × 10 × 10 Unity-world-unit box centered on the source object's origin. Check that the room fits inside it before baking, especially for non-metric reconstructions; adjust `ROI Size` or `Center Offset` under `Advanced Bake Settings` if needed. Later scale estimation cannot recover points already removed by the ROI.

**Metric scale is essential for MR registration:** the reconstruction must match the physical environment's dimensions, position, and orientation. The depth scan provides a metric target; enable `Estimate Scaling` if your 3DGS uses arbitrary units.

**Recommended first run: scan in `Room Scanner.unity`, press B to save, then run `Alignment Demo.unity`.** Both scenes are included: the scanner saves to the filenames the aligner already loads, and Alignment Demo is preset to `preScan`. Once your own 3DGS is imported and baked, no target-mode or filename changes are needed. See [Source Preparation Routes](#source-preparation-routes) if you have not created a reconstruction of the same physical environment yet.

**A. Scan and save the physical target**

1. Follow [Get the Project](#get-the-project), open the project in Unity `6000.0.76f1`, and connect your Quest through Meta Horizon Link. Enable spatial data access over Link in the PC app's Developer settings and accept the headset's spatial-data permission prompt when requested.
2. Open `Assets/Scenes/Room Scanner.unity`. Select `Quest Scanner`, keep `Scanner Mode` set to `Scan`, and leave the existing camera, raycast, anchor, and visualization references assigned. Under `Save Files`, keep `Point Cloud File` as `alignment_scan.bytes` and `Anchor Uuid File` as `alignment_scan.txt`.
3. Enter Unity Play Mode. In the headset, **hold the left controller's X button** while looking around and moving through the area represented by your reconstruction. Check that the point-cloud visualization covers the room; release X to pause and hold it again to collect more points.
4. **Press the right controller's B button** to create the spatial anchor and save the scan. Wait for both `Successfully saved ... points to ...` and `Anchor UUID saved to ...` in the Unity Console before stopping Play Mode. Saving is triggered by B, not by releasing X.

The files are written automatically to `Application.persistentDataPath`; no manual export or copying is needed when both scenes run from this Unity project on the same PC. Keeping the default filenames is sufficient. To keep multiple scans, choose a different pair of filenames under `Save Files` and use that same pair in the aligner; saving to an existing filename replaces that scan file.

**B. Load the target and align your 3DGS**

1. Stop Play Mode and open `Assets/Scenes/Alignment Demo.unity`.
2. Open `Tools > Gaussian Splats > Create GaussianSplatAsset`, import your 3DGS, and assign the created asset to the existing `3dgs` object's `Gaussian Splat Render` component. `Quality` defaults to `Very High` on first use; if the editor remembers a previous choice, select `Very High` for registration. The repository does not include the reconstruction shown in the demo video.
3. On that component, go to `Baking & Debug`, click **Apply Default Settings**, then **Bake & Generate Asset**. The preset enables Auto `Max Flatness`, `Density Filter`, and `Enable Selection ROI`. Applying defaults only sets parameters; Bake calculates Flatness from the current splats and generates the baked data, assigning it to the renderer automatically. Wait for the bake to finish. The indoor single-room ROI prerequisite above applies; make any necessary ROI adjustment after applying defaults and before baking.
4. Leave the `Aligner` object's preset settings in place: `Align`, `preScan`, `GaussianSplating`, and `Use Baked Data`. Its `Gaussian Source Renderer` already points to the existing `3dgs` object, and the scan filenames, anchor holder, point-cloud VFX, and `SpatialAnchor` save reference are assigned. Only change the renderer reference if you created a different source object, or the filenames if you renamed the scan in stage A.
5. Enter Play Mode in the scanned physical space. The aligner automatically loads `alignment_scan.bytes` and `alignment_scan.txt`, then runs the first alignment once the anchor and points are ready. The demo already enables TEASER++ scale estimation, one-time coarse scale initialization with 20% sampling, GICP refinement, and repeated alignment. You do not need to scan again or press Y to start this first pass. If the Console reports missing files or an anchor-localization failure, check the saved filename pair and return to the scanned space before proceeding.
6. Inspect the reconstruction against passthrough. If it is not aligned correctly, **press the left controller's Y button** to realign using the current source transform and the retained target. In current testing, some initially unsuccessful alignments became visually correct after several realignments; this is not a guarantee for every scene.

This is the recommended two-scene path. [Choosing a Registration Target](#choosing-a-registration-target) explains the other supported workflows, including same-session scanning and Quest's built-in environment setup. See [Environment Depth Scanning Performance](#environment-depth-scanning-performance) for the observed scan-time GPU load.

### Visual Comparison Controls

After alignment, use the visual comparison modes to inspect the registered reconstruction against the real environment:

1. Press the left controller menu/start button to open or close the comparison UI.
2. Select one of the three comparison modes:
   - **Mini-Window:** move the controller or tracked hand horizontally to move the local comparison window across the scene.
   - **3D Slider:** move the controller or tracked hand horizontally to move the boundary between the reconstruction and the real environment.
   - **Switch-back:** press the right controller `B` button to switch the reconstruction display on or off.

The comparison modes can target either a `GaussianSplatRenderer` or a standard `MeshRenderer`.

## Integration Guide

Start from this Unity project and adapt the parts your application needs. The following are source locations, not independently installable modules; copying a single script into another project is not sufficient to reproduce the configured workflow.

| What you want to build | Starting point | MR registration needed? |
| --- | --- | --- |
| A VR splat viewer or an XR scene containing 3DGS | `Packages/package/Runtime/GaussianSplatRenderer.cs` and `GaussianSplatAsset.cs`; use the asset creator under `Packages/package/Editor/` | No |
| Tools to inspect or prepare splat assets | The Gaussian Splats editor tools and `Gaussian Splat Render > Baking & Debug` | No; registration-specific filtering is optional for VR-only use |
| A physical-environment point-cloud capture workflow | `Assets/Scenes/Room Scanner.unity` and `Assets/Scripts/Quest Scan/Quest3PointCloudScanner.cs` | No source registration during capture; requires Quest environment scanning setup |
| Spatially registered 3DGS or meshes in MR | `Assets/Scenes/Alignment Demo.unity` and `Assets/Scripts/Registration/All In One Registration.cs` | Yes |
| Passthrough comparison after alignment | `Assets/Scripts/Visual Comparison/` and the comparison controls in `Alignment Demo.unity` | Use aligned content for physical-world comparison |

For VR-only development, begin with the renderer and your XR camera/interaction setup; a physical-room scan and `AllInOneRegistration` are not required. For MR development, start with the supplied alignment scene so the scanner, Meta scene references, and comparison controls remain connected while you replace the source reconstruction.

### How MR registration extends the renderer

The MR path extracts registration points from Gaussian centres or mesh geometry, collects target geometry from a depth scan or the headset's environment setup, and estimates an alignment using FPFH/TEASER++ followed by GICP/VGICP. The transform is applied to the rendered source and can be saved relative to Quest scene references or a spatial anchor.

![MR registration pipeline inherited from MR-Compare: source and target point collection, filtering, coarse registration, fine registration, and XR interaction](docs/workflow.png)

The diagram is the current workflow figure from the [MR-Compare project page](https://changruizhu96.github.io/work/mr-compare/#method). It shows the MR comparison application inherited from MR-Compare, one use of the toolkit rather than a prerequisite for VR rendering. See the [demo video](#demo-video) for the in-headset result.

## Project Status

Unity XR 3DGS Toolkit is a reusable Unity research toolkit rather than a finished consumer application. The repository includes the Unity XR/3DGS workflow, example scenes, MR registration pipeline, bundled native registration libraries, and interaction techniques. Users provide their own reconstructed 3DGS or mesh assets, and some workflows require scene-specific setup and parameter tuning.

“Self-contained” refers to the integrated Unity workflow: the XR renderer, environment capture, registration pipeline, and interaction techniques are maintained in one project. Unity and Meta packages are restored through the package manager, and the current registration backend uses bundled Windows `x86_64` native libraries tested through Meta Quest Link. Registration is not executed entirely on the headset.

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
- Windows editor/runtime environment for the native registration plugin
- [Microsoft Visual C++ Redistributable v14, x64](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170) for the native registration DLLs

Make sure the Quest headset OS/runtime is compatible with the Meta Quest Link version used on the development PC. Mismatched Quest Link and headset OS versions can cause environment depth, scene, or OpenXR behavior to differ from the tested setup.

For the most stable experience, disable the **Public Test Channel (PTC)** in the Meta Quest Link PC app. PTC releases may introduce runtime changes or regressions before the corresponding general release, which can affect Quest Link, OpenXR, MRUK scene data, or Environment Depth behavior. Use the general-release channel unless you specifically need to test a preview runtime.

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

For the MR registration path, import your own reconstructed environment; the toolkit does not ship with a ready-to-use reconstructed scene. VR-only users can start with [VR Quick Start](#vr-quick-start) without preparing a matching physical-room scan.

Supported source types:

- **3D Gaussian Splatting**: use `Tools > Gaussian Splats > Create GaussianSplatAsset`, use `Quality: Very High` (the first-use default), create the asset, and assign its `GaussianSplatRenderer` to the aligner. Previously saved quality choices are retained. Registration relies on the uncompressed `Very High` position data to read Gaussian centres correctly; this requirement does not apply to VR-only rendering.
- **Mesh**: import a reconstructed mesh and assign its `MeshFilter` to the aligner.

The current Inspector enum uses the spelling `GaussianSplating`; this refers to the 3D Gaussian Splatting source mode.

### Source Preparation Routes

Prepare a reconstruction of the physical environment you intend to scan and align:

- **Fast mobile route — [Scaniverse](https://scaniverse.com/):** capture and process a Gaussian splat directly on a phone, with few reconstruction steps.
- **Desktop route with metric calibration — [RealityScan](https://www.realityscan.com/download):** [align the source images](https://rshelp.capturingreality.com/en-US/tutorials/quickstart_2.htm), establish metric scale with [control points and distance constraints](https://rshelp.capturingreality.com/en-US/tutorials/scaling.htm), then export a mesh or pass the calibrated cameras and images to a desktop 3DGS pipeline.
- **Non-metric end-to-end workflow — e.g. Luma AI:** if the exported reconstruction uses arbitrary units, enable `AllInOneRegistration > Estimate Scaling`. For complete source and target clouds with high overlap, optional `Coarse Scale Alignment` initializes scale before the first applied alignment; later Y realignments skip this initialization.

### Source Scale Requirement

The most important requirement for 3DGS registration is scale. The 3DGS scene should be reconstructed at real-world scale, or at least be close enough that TEASER++ can estimate a usable scale correction.

For a desktop 3DGS handoff, preserve RealityScan's calibrated camera poses and metric scale when moving the images into a dedicated generator such as LichtFeld Studio, Nerfstudio's gsplat workflow, or Postshot.

If the imported 3DGS scale is inaccurate, first resize the source to a roughly correct scale in Unity, then enable TEASER++ `Estimate Scaling`. Alternatively, when the filtered source and metric target both cover the same complete scene with high overlap, enable `Coarse Scale Alignment` under `Estimate Scaling` to estimate an initial size ratio before native downsampling and feature matching. Newly added components leave coarse scaling off; the included Alignment Demo enables both scale options with 20% sampling. Partial scans and strongly different sampling densities can bias this ratio; retain manual scale preparation for those cases.

### 3DGS Point Cloud Preprocessing

Before matching a 3DGS source against the Quest target, clean the source points used for registration. The goal is to extract a stable, mostly foreground point cloud from the 3DGS asset, instead of registering against floaters, skyboxes, background geometry, or low-quality splats.

Use the `Baking & Debug` section on the `Gaussian Splat Render` component.

Important settings:

- `Apply Default Settings`: restores the indoor single-room bake preset without starting a bake. It enables Auto Flatness, density filtering, and ROI; resets minimum opacity to `0`, grid spacing to `0.05`, minimum neighbors to `2`, and coordinate normalization to off; and restores a 10 × 10 × 10 box centered on the renderer's origin, with no center override or offset. Apply it before any scene-specific adjustments.
- `Max Flatness`: defaults to Auto. Clicking Bake calculates the median splat scale ratio before filtering. Open `Advanced Bake Settings` and disable `Auto Max Flatness` to enter a manual threshold.
- `Density Filter`: defaults to on, with grid spacing `0.05` and minimum neighbors `2`. These are a starting preset, not an automatic estimate of metric scale; adjust them under `Advanced Bake Settings` if the source units or density differ substantially.
- `Enable Selection ROI`: defaults to on for new renderers and is enabled in Alignment Demo; existing components retain their serialized choice. The default box is 10 × 10 × 10 world units, centered on the renderer's origin. It restricts baked points to the room of interest; check its coverage for large, offset, or non-metric scenes.

Auto Flatness runs only when Bake is clicked, using the existing GPU export/readback and CPU median calculation. It adds no per-frame work during alignment. The displayed `Capture + Save` time excludes this preliminary Auto calculation.

Recommended preprocessing steps:

1. Select the 3DGS object in Unity.
2. Open the `Gaussian Splat Render` component.
3. Go to `Baking & Debug` and click **Apply Default Settings** to restore the indoor single-room preset. Check that the ROI contains the room; adjust its size or center under `Advanced Bake Settings` if necessary.
4. Click **Bake & Generate Asset** once. The renderer receives the generated baked asset automatically.
5. Inspect the baked point cloud; adjust the advanced filters and rebake only if relevant geometry is missing or unwanted background remains.
6. Use that renderer as the `AllInOneRegistration` source with `Use Baked Data` enabled. Alignment Demo already has these references assigned for its existing `3dgs` object.

If registration fails, inspect the source and target point clouds first. In most failed cases, the cause is one of the following:

- the source scale is not close to real-world scale;
- the source includes too much background or skybox;
- the source and target point clouds do not share enough overlap;
- the source point cloud is too noisy for stable FPFH correspondences;
- the TEASER++ voxel size or GICP/VGICP parameters are not appropriate for the scene scale.

## Choosing a Registration Target

The toolkit supports three target-capture workflows:

- **Depth pre-scan (recommended Quick Start):** capture and save a target in `Room Scanner.unity`, then reuse it in `Alignment Demo.unity`.
- **Real-time depth scan and align:** capture target points and align the reconstruction in the same scene and session.
- **Quest 3 built-in environment setup:** complete `Settings > Environment Setup > Space Setup` directly on the headset. Quest assists with scanning the room, and the toolkit uses the resulting scene mesh without a separate toolkit depth scan. See [Meta's official assisted Space Setup guide](https://www.meta.com/help/quest/853707093070890/).

These workflows map to four `Target Format` modes in `AllInOneRegistration`. For the built-in Quest environment setup workflow, `effectMesh` and the legacy `roomMesh` mode serve the same purpose: using the headset's captured scene mesh as the target. They are separate implementation options for current and older Meta scene components, rather than separate capture workflows.

`preScan` and `realTimeScan` use the same underlying target-generation and registration path: the Quest Environment Depth scanner produces a point cloud, which is then aligned with the reconstructed source through the same TEASER/GICP pipeline. The difference is how the workflow is organized. `preScan` saves the scan and anchor for a later alignment session, while `realTimeScan` passes the scanner points directly to alignment in the same scene and session.

When `Allow Repeated Alignment` is enabled, pressing `Y` after the initial alignment reruns registration with the retained target data in `preScan`, `roomMesh`, and `effectMesh` modes. In `realTimeScan`, X can add scan coverage before Y runs the next alignment. Repeated alignment refreshes the source point cloud in its current pose, so each pass acts as an incremental refinement.

The included **Alignment Demo is preset to `preScan`** with `alignment_scan.bytes` / `alignment_scan.txt`, the scan anchor and point-cloud VFX references, `SpatialAnchor` alignment saving, and the existing `3dgs` source renderer. It is ready to load the output of Room Scanner once you import and bake your own source. Its embedded scanner and Meta scene references remain available if you explicitly switch to another target mode.

Newly added `AllInOneRegistration` components still start with `Align`, `effectMesh`, `GaussianSplating`, TEASER rough alignment, GICP refinement, baked source data, repeated alignment enabled, and automatic alignment saving disabled. Custom scenes require their own target references; use the included demo for the prepared Pre-Scan workflow.

| Target format | Recommended use | Target data | Main tradeoff |
| --- | --- | --- | --- |
| `preScan` | Separated, reusable scan/alignment workflow | Environment Depth scanner point cloud loaded with its saved anchor data | Requires a separate scan/save/load stage |
| `realTimeScan` | Integrated scan/alignment workflow | The same Environment Depth scanner point cloud passed directly from memory | Fewer handoff steps; scanner and 3DGS rendering must be isolated at runtime |
| `effectMesh` | Quest built-in environment setup; recommended for new scenes using this workflow | Global scene mesh exposed through Effect Mesh | No separate toolkit scan; target detail and accuracy depend on the headset's captured mesh |
| `roomMesh` (legacy) | The same built-in environment setup workflow in older MRUK scenes | Scene mesh exposed through Room Mesh Event | Retained for compatibility; use `effectMesh` for new scenes |

## Detailed Usage

### Quest Environment Setup Workflow

Use this workflow to capture the room through Quest's own environment setup and use its scene mesh as the registration target. The steps below use the current `effectMesh` mode; `roomMesh` is the legacy implementation of this workflow. Registration accuracy depends on the accuracy, coverage, and geometric detail of the headset's captured mesh.

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

In the current tested setup with Meta Horizon Link `208.0.0.84.535`, GPU utilization can reach approximately **90% while Environment Depth scanning is active**, causing frame drops during target point-cloud capture. This may be a Meta Depth API/runtime performance issue; the root cause has not been isolated.

In the included Alignment Demo, pressing **Y** to finish target capture and begin registration automatically deactivates the scanner and its `EnvironmentDepthManager`, shutting down the Depth API. The scan-related slowdown was no longer observed during subsequent alignment and visual comparison in this setup. Pressing **X** to resume scanning reactivates depth capture and may bring back the slowdown. Loading a saved Pre-Scan target for alignment does not require active depth capture.

### Separated Workflow: Pre-Scan / Quest 3 Scanner

This workflow uses the same Environment Depth point-cloud target and registration pipeline as `realTimeScan`, but separates scanning from alignment. The Room Scanner saves the points and spatial anchor first, and the Alignment Demo loads them later. This makes the target reusable and keeps scanner collection separate from 3DGS rendering and alignment. Registration quality depends on scan coverage and settings rather than on choosing `preScan` instead of `realTimeScan`.

1. Open `Assets/Scenes/Room Scanner.unity`.
2. Run the scene on Quest.
3. Hold `X` to scan the physical environment.
4. Press `B` to create a spatial anchor and save the scanned point cloud. With the included scene's defaults, this writes `alignment_scan.bytes` and `alignment_scan.txt` to `Application.persistentDataPath`. Wait for both point-cloud and anchor-file save confirmations before stopping Play Mode.
5. Open `Assets/Scenes/Alignment Demo.unity`.
6. Keep the demo's `Workflow Mode = Align` preset.
7. Keep its `Target Format = preScan` preset; choose this mode explicitly only when configuring a custom scene or switching back from another workflow.
8. Keep `Point Cloud Target File = alignment_scan.bytes` and `Anchor Uuid File = alignment_scan.txt`; if you renamed either file in the scanner, enter the matching name here.
9. Keep the included scene's anchor-holder and point-cloud VFX references, or assign them when building a custom scene.
10. Configure the source type and source object.
11. Use `SpatialAnchor` as the alignment reference mode if you want the result to persist relative to the saved scan anchor.
12. Run registration.

The scanner stores:

- point cloud data;
- spatial anchor UUID data.

### Real-Time Scan Workflow

This workflow integrates the same Environment Depth scanner and point-cloud registration path used by `preScan` into one scene and session. Instead of saving and reloading the target, it passes the accumulated scanner points directly from memory to alignment. The toolkit suspends 3DGS rendering while scanning and deactivates the scanner before alignment. Prefer **Pre-Scan / Quest 3 Scanner** when you want a reusable saved target or separate scan/alignment stages; use **Quest Environment Setup** when you want the headset to prepare the scene mesh without a separate toolkit scan.

1. Open `Assets/Scenes/Alignment Demo.unity`.
2. Set `Target Format` to `realTimeScan`.
3. Enable `Allow Repeated Alignment` if the user should be able to refine the result with additional X/Y cycles.
4. The included demo scene already has its in-scene Quest scanner assigned to `Scanner Target`. No scanner setup is required unless you are building a custom scene.
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
- `Estimate Scaling`: estimate the residual scale difference. Prepare the source near metric scale manually or, for complete high-overlap clouds, use the option below.
- `Coarse Scale Alignment`: visible only with `Estimate Scaling` enabled. Approximate each cloud's geometric median and 90th-percentile radius from deterministic stratified samples, then pre-scale the temporary source buffer around its estimated center. The CPU preparation runs on a worker thread, reuses the DLL input buffer, and introduces no GPU transfer. TEASER++ still estimates residual scale; the final transform includes both scaling stages and the scale pivot. Logs report coarse scale, residual scale, combined scale/transformation matrix, and preparation time (including worker scheduling). Invalid preparation, an invalid scale, or a missing/non-finite native matrix aborts this optional path without changing the rendered source transform. Native non-convergence alone emits a warning and applies the finite candidate, matching the existing registration behavior; this does not certify alignment quality. Saving, if enabled, still saves the applied candidate.
- Coarse scaling runs only before this component's first applied registration. Later realignments retain the current source scale and use the existing TEASER++/GICP path. An attempt rejected before applying its transform does not consume initialization; applying a candidate does, even if native convergence is false. Enabling coarse scaling after a registration was already applied does not rerun initialization. A fresh component/scene run resets this state; merely toggling the option or rescanning does not. This improves initialization but does not guarantee a correct result in one attempt.
- `Coarse Scale Sample (%)`: visible when coarse scale alignment is enabled. Percentage of each cloud to sample, default 5%, range 0.1–100%. Each cloud uses `min(pointCount, max(3, ceil(pointCount * percentage / 100)))` points; 100% uses all points. There is no fixed point-count cap. The fixed-seed sampling picks one point per index stratum; it is a Monte Carlo approximation when subsampling, not spatially uniform sampling. Larger percentages increase CPU time and memory and can reduce sampling variation, but cannot correct density or coverage bias. The default is an engineering starting point, not a validated optimum. This setting controls how many points are sampled; the radius statistic remains the 90th percentile. This replaces the old sample-count setting; old count values are not converted into percentages, and the new field starts at 5%.

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

## Known Limitations

- The native registration plugin is currently provided as Windows `x86_64` DLLs.
- The repository does not include user-specific reconstructed scenes.
- Large 3DGS files, meshes, point clouds, or native binaries may require Git LFS or external release assets.
- Registration quality depends on overlap, scale consistency, target point-cloud or effect-mesh accuracy, source geometry quality, and parameter tuning.
- Active Environment Depth scanning may cause high GPU utilization and frame drops in the tested Link runtime; see [Environment Depth Scanning Performance](#environment-depth-scanning-performance). The included scan-to-alignment handoff disables depth capture automatically.
- The project may require scene-specific setup before it can be used as a turnkey application.

## Research Origins

The toolkit originated in **[MR-Compare](https://github.com/changruizhu96/MR-Compare)**, a research system for spatially grounded comparison of reconstructed and physical environments. The toolkit starts from that implementation with a fresh repository history. The repositories have different roles: MR-Compare documents the research system; this repository maintains reusable Unity XR/3DGS workflows and future toolkit development.

**MR-Compare: A Mixed-Reality Framework for Spatially Grounded Visual Comparison of 3D Gaussian Splatting and Mesh Reconstructions with the Physical Environment**

**IEEE ISMAR 2026 · TVCG Paper** · [Best Paper Award · Honorable Mention](https://www.ieeeismar.net/2026/awards/)

[Research repository](https://github.com/changruizhu96/MR-Compare) · [Paper on arXiv](https://arxiv.org/abs/2607.20325) · [Project page and demo](https://changruizhu96.github.io/work/mr-compare/)

The publication and award refer to MR-Compare, not to separate evaluations of later toolkit releases. The current toolkit remains a complete Unity project; a standalone UPM distribution is not provided.

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

Please also acknowledge the upstream projects listed above and cite the reconstruction, registration, or MR systems used in your workflow. Citation is an academic request and is not an additional condition of the MIT License.

## License

Unity XR 3DGS Toolkit continues the MR-Compare codebase under the [MIT License](LICENSE). Copyright (c) 2026 Changrui Zhu.

The license applies to the original MR Compare software and accompanying documentation in this repository, except where a file or directory carries a separate license or copyright notice.

Under the MIT License, you may:

- use the software for academic, research, educational, private, or commercial purposes;
- copy, modify, merge, publish, and distribute the software;
- sublicense or sell copies of the software; and
- include the software in open-source or proprietary derivative work.

When redistributing the software or a substantial portion of it, you must retain the original copyright notice and MIT permission notice. The software is provided **as is**, without warranty; see the full [LICENSE](LICENSE) text for the legally controlling terms.

The MIT License does not replace or override licenses for third-party material. In particular:

- the modified Unity Gaussian Splatting package remains subject to its bundled [MIT license](Packages/package/LICENSE.md);
- the adapted [Depth Scanner Project](https://github.com/Appletea0673/Depth-Scanner-Project) is distributed under its upstream MIT license; and
- Meta XR SDK, Meta MR Utility Kit, Unity packages, native registration libraries, and other dependencies remain subject to their respective licenses and terms.

User-supplied datasets, scans, 3DGS assets, meshes, model checkpoints, and other imported content are not covered by the MR Compare license. Users are responsible for ensuring that they have permission to use and redistribute those materials.
