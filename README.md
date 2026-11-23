# AR Glasses Virtual Try-On with Dynamic Gaussian Avatars

> **Project status:** Planning and implementation roadmap. The features described below are **proposed**, not yet implemented or experimentally validated.

## 1. Project Overview

This graduation project aims to build a practical **AR glasses virtual try-on and geometric fitting demonstration** by combining a photorealistic dynamic human-head representation with an independently designed 3D glasses model.

The main technical foundation is [GaussianAvatars](https://github.com/ShenhanQian/GaussianAvatars), which represents a human head using 3D Gaussian primitives attached to the [FLAME](https://flame.is.tue.mpg.de/) head model. We plan to reuse the original reconstruction method rather than modify its learning architecture. A pair of AR glasses, prepared in Blender and informed by a separate Zemax optical design, will be aligned with and attached to the reconstructed head **after** avatar training.

The intended outcome is an interactive demonstration in which a viewer can observe a moving head, toggle the glasses on or off, change the frame color, automatically position the glasses, and inspect their geometric fit. **Unity is optional**: the preferred initial implementation extends the GaussianAvatars Python viewing/rendering pipeline to avoid unnecessary engine-conversion work.

This is a **virtual prototype and engineering demonstration**, not a manufactured or clinically validated wearable product.

## 2. Motivation

AR eyewear development involves more than optical performance. The relative placement of the frame, optical components, eyes, and head must also be considered, especially when a user moves.

Physical prototypes are expensive and slow to revise. A virtual try-on workflow can support early design iteration before fabrication by allowing developers to:

- Visualize a proposed AR glasses model on a photorealistic 3D human head.
- Verify that the frame follows recorded head motion without obvious sliding.
- Inspect basic geometric alignment with the eyes and nose bridge.
- Identify visible intersections between the frame and a head-mesh approximation.
- Compare the effect of fit adjustments and frame colors without retraining the avatar.
- Connect a teammate's Zemax optical-layout parameters with a visual model of the intended wearable geometry.

The project's contribution is **application integration, visualization, and evaluation**, not a new Gaussian Splatting or neural-network algorithm.

## 3. Objectives and Scope

### Core objectives

1. Reproduce a working GaussianAvatars rendering pipeline using an official example or an authorized dataset.
2. Obtain a dynamic Gaussian head and corresponding FLAME geometry/pose information.
3. Import **one** Blender AR-glasses model as a separate conventional mesh asset.
4. Implement initial alignment, head-following attachment, and optional manual position adjustments.
5. Provide **one-click glasses visibility**, **frame-color switching**, and free-view camera controls.
6. Implement **automatic initial placement** and a lightweight **geometric fit check**.
7. Demonstrate coherent rendering of the Gaussian head and traditional mesh glasses, including reasonable occlusion.
8. Test the integrated application and document its limitations.

### Deliberately out of scope

- Developing or modifying the original GaussianAvatars learning algorithm.
- Building a new 4DGS architecture or conducting large-scale model comparisons.
- Facial-expression switching as a user-facing feature.
- Switching among different glasses designs as a user-facing feature.
- Manufacturing physical glasses.
- Claiming that a visualized lens reproduces real AR optical imaging.
- Real-time webcam-based face tracking in the first release.
- Mandatory Unity integration.

**Terminology note:** GaussianAvatars is a rigged dynamic Gaussian head representation. It is **not the same method** as the original CVPR 2024 *4D Gaussian Splatting for Real-Time Dynamic Scene Rendering* implementation. This project focuses on the former because the FLAME head representation is useful for eyewear placement.

## 4. Proposed System Architecture

```text
Authorized head data / official pretrained example
                    |
                    v
       GaussianAvatars reconstruction
       + FLAME geometry and head pose
                    |
                    v
          Dynamic Gaussian head
                    |
                    +----------------------------+
                    |                            |
                    v                            v
       Head-local reference frame        Per-frame head transform
                    |                            |
                    +-------------+--------------+
                                  |
Zemax optical layout             |
       |                          |
       v                          |
Blender AR-glasses mesh           |
       |                          |
       v                          v
   Model preparation ------> Auto-fit & attachment
                                  |
                                  v
                      Gaussian + mesh rendering
                                  |
                                  v
                    Interactive try-on controls
                                  |
                                  v
                    Fit checks and demo outputs
```

**Key design decision:** The Gaussian head and the glasses are separate assets. The glasses do **not** participate in GaussianAvatars training. They can be adjusted or recolored after training without reconstructing the person.

## 5. Tools and Data Sources

| Component | Proposed tool | Purpose |
| --- | --- | --- |
| Dynamic head representation | [GaussianAvatars](https://github.com/ShenhanQian/GaussianAvatars) | Reconstruct/render a Gaussian head driven by FLAME |
| Head geometry and pose | [FLAME](https://flame.is.tue.mpg.de/) | Reference geometry, landmarks/regions, and head transforms |
| Official training data | [Preprocessed NeRSemble](https://github.com/ShenhanQian/GaussianAvatars/blob/main/doc/download.md) | Lower-risk reproduction using the authors' data pipeline |
| Optional custom-video preparation | [VHAP](https://github.com/ShenhanQian/VHAP) | Estimate head tracking and export compatible-style data |
| Glasses creation | Blender | Create/import a frame, assign materials, and set physical scale |
| Optical design | Zemax OpticStudio (teammate) | Optical layout and independently verified optical parameters |
| Viewer and rendering | Python + GaussianAvatars viewer / renderer | Head playback, camera interaction, and combined visualization |
| Compute | AutoDL GPU instance | Installation, model training if needed, and offline rendering |
| Optional future integration | Unity or a 3DGS scene viewer | Additional demonstration environment if time and scope allow |

### Dataset strategy

**Baseline:** Start with the official GaussianAvatars pretrained example. Then reproduce a selected subject from the authors' **preprocessed NeRSemble** data if training is required for the graduation assessment.

**Optional extension:** Use VHAP to preprocess an appropriately licensed, consented monocular or multi-view head video. This is not guaranteed to work as a drop-in replacement: camera calibration, segmentation, FLAME tracking, sequence naming, and export formats must be checked.

Synthetic characters or game-engine footage are possible research inputs **only when their use and licensing permit it**. Rendered RGB frames alone are insufficient; camera parameters, consistent motion timing, foreground masks, and head/FLAME registration are also needed. Custom data will therefore be attempted only after the baseline is stable.

## 6. End-to-End Development Plan

### Phase 1 — Reproduce the original rendering baseline

- Set up a reproducible Linux, CUDA, PyTorch, and GaussianAvatars environment on AutoDL.
- Record the repository commit, dependency versions, and GPU configuration.
- Acquire the required FLAME assets under their respective license terms.
- Run the official pretrained head example and confirm successful rendering.
- Export a short offline animation and verify that head motion is actually changing over time.
- If appropriate, train one NeRSemble subject using the official data preparation and training instructions.

**Deliverable:** A reproducible dynamic-head render, its configuration, and a baseline demonstration video.

### Phase 2 — Prepare the glasses asset

- Obtain a legally reusable provisional glasses mesh or use the teammate's Blender model.
- Separate the **frame** from the transparent optical elements so material changes affect the intended parts only.
- Confirm model origin, orientation, dimensions, and consistent coordinate conventions.
- Define the intended eye positions and optical-element reference points for later fit checks.
- Preserve the glasses as a separate mesh rather than converting them into trained Gaussians.

**Deliverable:** One cleaned and documented glasses model with an initial head-relative transform.

### Phase 3 — Head-following attachment

- Read the FLAME head pose or derive a stable rigid head reference frame from selected head-region vertices.
- Define a constant glasses-to-head offset in the local head coordinate system.
- Update the glasses transform for every animation frame using that head pose.
- Provide simple manual controls to refine translation and rotation if automatic placement is imperfect.
- Confirm that the glasses move with the head during left/right rotation, pitch, and translation.

A simplified attachment relationship is:

```text
T_glasses_world(t) = T_head_world(t) × T_glasses_head
```

Here, `T_glasses_head` is the calibrated local placement. If the head mesh deforms, the rigid reference must be constructed from appropriate stable regions rather than from arbitrary facial-expression vertices.

**Deliverable:** A moving Gaussian head wearing a synchronized glasses mesh.

### Phase 4 — Automatic initial placement and geometric fit check

Implement a **Fit Glasses** control that estimates a plausible initial placement using FLAME reference geometry:

1. Estimate a head-centered frame from stable landmarks or mesh regions.
2. Identify approximate eye centers and the nose-bridge region.
3. Align the glasses' centerline and nose-support area to the head references.
4. Apply a sensible initial offset and preserve the intended frame/optical dimensions.
5. Evaluate basic geometric clearances and visually flag obvious intersections.
6. Allow a user to refine the proposed placement rather than assuming the automatic result is perfect.

Candidate fit-check outputs include:

- Left/right alignment of the optical reference points relative to the eye references.
- Nose-bridge distance and visible support alignment.
- Minimum clearance or penetration against an approximate FLAME head surface.
- Relative eye-to-optical-component distance using parameters supplied by the optical-design teammate.
- Whether the head-relative glasses offset remains consistent across an animation.

These measurements are **approximations based on the available head model**. FLAME is not a precision scan of the ears, scalp, or nose-contact surfaces. The fit check does not determine real-world comfort, prescription suitability, or physical safety.

**Deliverable:** Automatic placement plus an interpretable fit-status panel.

### Phase 5 — Joint rendering and interactive controls

The initial implementation will extend a Python-based viewer or rendering pipeline rather than requiring Unity.

Planned controls:

| Control | Behavior |
| --- | --- |
| Wear / Remove | Show or hide the glasses mesh without changing the Gaussian head |
| Frame color | Change the frame material color at runtime |
| Automatic fit | Recompute the initial head-relative glasses placement |
| Fit adjustment | Fine-tune translation/rotation after auto-fit |
| Playback | Play/pause recorded head motion |
| Camera | Orbit, pan, and zoom around the reconstructed head |
| Fit check | Display geometric distances and visible warnings |

**Rendering challenge:** The Gaussian head and Blender mesh use different representations. A single compositing pipeline must maintain consistent camera calibration, coordinate systems, depth ordering, and lighting. A FLAME mesh can serve as an approximate depth/occlusion proxy for the face, but hair and transparent lenses may still require special handling.

First produce a correct **offline composite**. Only then enable live interaction in the viewer. If remote GUI performance is unreliable, provide a locally runnable viewer or a recorded demonstration as a fallback.

**Deliverable:** An interactive virtual try-on prototype and a recorded backup demo.

### Phase 6 — Testing, evaluation, and final demonstration

Test the pipeline on a variety of recorded head movements and camera angles. Where feasible, use a second authorized head sequence to check that placement is not tailored to only one subject.

Planned validation:

| Area | Test | Evidence |
| --- | --- | --- |
| Head attachment | Head rotation and translation | Glasses remain aligned with the chosen head coordinate frame |
| Automatic fitting | Initial placement versus manually refined placement | Placement offsets, clearances, and before/after screenshots |
| Geometry | Glasses/frame intersections | Collision/clearance warnings and visual inspection |
| Visual compositing | Front, three-quarter, and side views | Evidence of correct/incorrect occlusion |
| Interaction | Visibility, color, camera, fit and playback controls | Functional test checklist |
| Performance | Selected rendering resolutions and display modes | FPS, GPU memory use, and responsiveness |
| Reproducibility | Repeat the documented baseline setup | Saved environment and configuration information |

The primary success condition is a **working and repeatable virtual try-on application**, not an improvement to the original reconstruction quality or benchmark scores.

## 7. Main Challenges and Mitigations

| Challenge | Why it matters | Planned mitigation |
| --- | --- | --- |
| Gaussian head and mesh integration | They use different renderers and asset types | Start with offline rendering using identical cameras; add interactive rendering later |
| Coordinate systems and physical scale | Blender, FLAME, Gaussian outputs, and Zemax may differ | Document axes, handedness, origins, and unit conversions in one place |
| Stable eyewear attachment | Poor reference frames cause drifting or visible sliding | Attach in a rigid head-local frame; validate with multiple motion sequences |
| Automatic fitting | Face geometry may not capture nose pads or ears accurately | Use a conservative initial estimate plus manual correction |
| Occlusion and transparent lenses | Incorrect depth ordering breaks realism | Use a head-mesh depth proxy, explicit render ordering, and controlled test views |
| Training environment compatibility | CUDA and native extensions can fail to build | Pin tested dependencies and keep the official example as a smoke test |
| Custom data compatibility | New videos may lack reliable FLAME/camera information | Use the official NeRSemble baseline first; treat VHAP data as an extension |
| Live demonstration stability | A remote cloud GUI may have latency or display issues | Keep pre-rendered demo videos and a verified fallback path |

## 8. Expected Outcomes

- A working GaussianAvatars baseline reproduction and documented setup.
- A dynamic photorealistic head with recorded head-motion playback.
- One Blender AR-glasses model attached to the head independently of avatar training.
- A user-facing **Wear / Remove**, **Frame Color**, and **Auto-Fit** workflow.
- Basic geometric fitting and inspection results.
- A consistent Gaussian-plus-mesh rendering workflow.
- Demonstration videos, a functional test report, and a final project presentation.

### Relationship to the broader AR project

The optical-design teammate develops the proposed AR eyewear optics in Zemax and creates the corresponding glasses geometry in Blender. This repository focuses primarily on the **Gaussian Splatting, motion-following, and virtual-fit visualization** side. Selected optical layout parameters may be used for geometric placement checks.

The originally proposed **3DGS environment reconstruction and Unity-based AR scene** remain possible later extensions, but are not prerequisites for the core try-on prototype. Any change to the approved graduation-project scope should be confirmed with the supervisor.

## 9. Evaluation Boundaries

This system can assess **visual appearance and approximate geometric relationships** in a digital prototype. It does **not** independently validate:

- Optical image quality, field of view, eye box, distortion, or see-through performance.
- Mechanical durability, weight distribution, or long-term comfort.
- Exact anatomical fit where the reconstruction lacks detailed geometry.
- Real-time tracking of a live human unless a separate tracking component is added.

Optical properties must be analyzed separately in Zemax. The planned head motion is **recorded motion playback**; it should not be described as live webcam tracking.

## 10. Responsibilities

| Gaussian Splatting / application development | Optical design / modeling |
| --- | --- |
| GaussianAvatars baseline and optional dataset training | Zemax optical system design |
| Head pose and FLAME geometry integration | Optical performance and layout verification |
| Automatic glasses placement and attachment | Blender frame and optical-component geometry |
| Viewer controls, joint rendering, and fit inspection | Design dimensions and optical reference parameters |
| Demonstration, testing, and reporting | Optical-design documentation and results |

## 11. References and Acknowledgments

- [GaussianAvatars — Official Code and Documentation](https://github.com/ShenhanQian/GaussianAvatars)
- [GaussianAvatars Project Page](https://shenhanqian.github.io/gaussian-avatars/)
- [GaussianAvatars — Data Download Instructions](https://github.com/ShenhanQian/GaussianAvatars/blob/main/doc/download.md)
- [VHAP — Versatile Head Alignment with Adaptive Appearance Priors](https://github.com/ShenhanQian/VHAP)
- [NeRSemble Dataset](https://tobias-kirschstein.github.io/nersemble/)
- [FLAME Head Model](https://flame.is.tue.mpg.de/)

**Licensing:** GaussianAvatars, FLAME, NeRSemble, any example models, and third-party eyewear assets have their own terms of use. Access, attribution, redistribution, and any commercial restrictions must be reviewed before using or distributing project artifacts.

---

*This README describes the planned engineering workflow. Implementation details, reproducibility instructions, measured results, and demo media will be added as the project progresses.*
