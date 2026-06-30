---
title: A Not-So-Simple Game Engine
description: "While iterating on the voxel engine, I ended up improving the engine itself. In this post, I go over the new features, architectural changes, and design decisions behind them."
slug: engine-001
date: 2026-07-01
categories:
    - Game Engine
tags:
    - Game Engine
image: sponza.png
---

## Preface

After the last post, I finally started working on the voxel logic itself (which will be covered in the next post!). However, while doing so, I realized how coupled some parts of the engine were, as well as the limitations I would likely face in the future.

At the same time, I fell into the usual rabbit hole of feature creep and refactoring, improving things along the way. Initially, it was just small tweaks, which I simply updated in the previous post. But after a while, I realized that the larger changes deserved their own update, so I think it is worth covering them here.

## Engine structural changes

In this post, I won’t go into small refactors, fixes, or minor improvements. While the devil is in the details, both the engine and voxel system are still in early development and are likely to change significantly in the future, so those types of changes are to be expected. With that in mind, let's go folder by folder through the engine and see what has changed.

### Assets

#### New ownership model

Initially, the asset manager stored every asset using **std::shared_ptr**. Since assets are owned exclusively by the asset manager and accessed elsewhere through lightweight AssetHandles rather than shared ownership, I replaced **std::shared_ptr** with **std::unique_ptr**. This better expresses the ownership model, removes unnecessary reference counting overhead, and improves performance.

#### Model loading

A few improvements were also made to make model loading more robust and consistent:

- **Default asset values.** Missing or incomplete resources now fall back to sensible defaults. For example, missing textures are replaced with a checkerboard texture and missing materials use a default material, allowing models to load gracefully instead of failing at runtime.
- **MikkTSpace tangent generation.** Tangent space is generated automatically whenever a glTF asset does not provide tangents, ensuring normal maps behave consistently across different tools while allowing assets without precomputed tangents to render correctly.

    ![Missing textures](missing_textures.png)

#### Animated models

Model loading was extended to support animated glTF assets, in addition to static geometry, materials, and textures.

When a glTF file contains a **skeleton**, the engine builds a bone hierarchy stored as a flat structure, where each bone stores its parent index, inverse bind matrix, and rest pose. **Animation clips** are loaded as collections of channels, where each channel contains translation, rotation, and scale keyframes for a single bone.

This allows animations to be evaluated at runtime by the **Animator**, which reconstructs poses from the imported data.

> For a deeper explanation of skeletal animation, I recommend reading [this article](https://learnopengl.com/Guest-Articles/2020/Skeletal-Animation).

![Animated model](animated_model.png)

#### Cubemaps support

I also implemented cubemap support from scratch. In my previous engine, textures and cubemaps were unified under a single texture asset, which led to a lot of conditional logic and made the system harder to maintain.

A **Cubemap** asset now exclusively handles loading the six faces, validating them, and creating the GPU cubemap texture.

Rendering is handled independently by **SkyboxRenderer**, which draws a unit cube directly on the GPU (no imported mesh or CPU geometry), removes translation from the view matrix so the skybox stays centered on the camera, and samples the cubemap in the fragment shader to render an infinitely distant environment.

![Cubemap](cubemap.png)

### Core

#### New config file format

The most significant change in this folder was changing config file parsing from INI to TOML, thus switching from [simpleini](https://github.com/brofield/simpleini) to [toml++](https://github.com/marzer/tomlplusplus). I found that with the first I was maintaining a lot of code just to parse the INI file. After looking at toml++, I found it to be a more modern and flexible format, and its API just clicked with me more.

#### New update loop

I moved away from updating everything directly in the application loop. The application is now responsible only for platform-level concerns such as windowing, event polling, timing, and frame orchestration, while simulation is handled in **Level**.

**Level** acts as a simulation coordinator for the game world. It orchestrates the update of the scene, gameplay systems, and related simulation logic, using a player-intent-based input model to decouple simulation from direct input devices.

> The update loop now uses a more robust fixed timestep approach for handling delta time, based on the approach described in [Fix Your Timestep](https://gafferongames.com/post/fix_your_timestep/).

### Render

#### Decoupling scene from renderer

One of the important architectural improvements in the engine was decoupling the scene from the renderer by introducing a transformation layer. Scene objects are no longer directly consumed by rendering systems; instead, they are converted into rendering-specific data structures that describe what should be rendered, not how it should be rendered.

#### Rendering pipeline refactor

Rendering is the part of the engine that has changed the most (and could potentially be separated into its own module). The rendering pipeline underwent a significant refactor, particularly around transparency handling and the render queue system. This redesign was driven by both the integration of animation and research into more efficient and scalable transparency techniques.

Transparency is explicitly defined in the material system and influences how geometry is classified during submission. Each renderable is assigned to a specific path depending on its state, including whether it is opaque, animated, or transparent, and in the latter case whether it uses **CPU sorting** or **OIT**.

Sorted transparency uses a CPU-based depth sorting approach where objects are rendered back to front. It is efficient for simple cases but becomes expensive with large numbers of transparent objects and does not handle intersecting geometry correctly.

**OIT (Order Independent Transparency)** is used for particles and other complex transparent effects. It avoids sorting by using accumulation and revealage buffers in a dedicated framebuffer setup, trading CPU cost for a controlled GPU cost.

> For more information on **OIT**, I recommend reading [this article](https://learnopengl.com/Guest-Articles/2020/OIT/Introduction).

The render queue separates static opaque batches, animated geometry, and transparent submissions into distinct internal structures. Transparent geometry is further split into sorted and OIT paths at submission time, while animated geometry bypasses instancing and is processed per draw call. This structure allows the renderer to maintain batching efficiency for static geometry while supporting more complex rendering cases.

#### Framebuffer and post-processing

I also introduced framebuffer support into the engine, enabling more advanced rendering techniques such as OIT accumulation, MSAA, and post-processing effects.

![Edge detection post-processing effect](edge_detection.png)

In my previous engine, full-screen post-processing effects were implemented using a screen-aligned quad. Here, I switched to using a full-screen triangle, which is a more efficient and modern approach in OpenGL.

This approach avoids the need for vertex buffers for screen geometry and guarantees full-screen coverage without artifacts caused by diagonal edges in a quad.

> A detailed discussion of the full-screen triangle technique can be found here:
https://stackoverflow.com/questions/2588875/whats-the-best-way-to-draw-a-fullscreen-quad-in-opengl-3-2/51625078
and an in-depth performance analysis here:
https://wallisc.github.io/rendering/2021/04/18/Fullscreen-Pass.html

> A few debugging views were also added to visualize framebuffer contents, which is useful for debugging post-processing effects and rendering issues.

<table>
  <tr>
    <td align="center">
      <img src="normals.png" width="92%"><br>
      <em>Normals view</em>
    </td>
    <td align="center">
      <img src="material_ids.png" width="100%"><br>
      <em>Material IDs view</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="ndotl.png" width="91%"><br>
      <em>NDOTL(normal dot light) view</em>
    </td>
    <td align="center">
      <img src="linear_depth.png" width="100%"><br>
      <em>Linear Depth view</em>
    </td>
  </tr>
</table>

#### Lighting system

The lighting shader was refactored to support a variable number of lights instead of a fixed limit, improving scalability with scene complexity. I also introduced a hybrid approach combining Blinn-Phong and PBR-style metallic-roughness inputs.

> The system is still a work in progress, and I plan to further improve it in the future, potentially exploring more scalable approaches such as deferred rendering for handling scenes with many lights more efficiently (likely for the engine side, while the voxel renderer may explore alternative approaches such as flood lighting and other specialized techniques).

### Scene

#### First-person camera and Visibility Mask

The initial camera implementation was changed to support first-person movement by using a visibility mask to filter out the player mesh. The idea of using masks/layers to filter objects in the scene is something I learned from Godot, and I found it to be very useful.

It also generalizes naturally to other cases, such as excluding UI or particles from specific render passes, or filtering objects for effects like water reflections and shadow maps.

I also improved the third-person camera to behave more like an orbit camera instead of using a fixed offset from the player, which provides more flexible control over camera movement.

> The third-person camera does not currently prevent clipping through walls, as it does not include collision handling. A physics-aware camera will likely be added later when more complex environments such as caves are introduced.

#### Animation system

At runtime, each animated entity owns an **Animator** and an **AnimationController**. The **AnimationController** is a small locomotion state machine that selects which animation should play (for example idle, walk, or run), while the **Animator** performs the actual animation playback.

During each update, the **Animator** advances the current animation time, samples the keyframes for each animated bone, optionally blends between two poses during transitions, and builds the final bone palette. These bone matrices are then uploaded to the GPU, where the vertex shader performs skinning to produce the final animated mesh.

![Animated fox](fox.gif)

#### Root motion

Root motion was added as an optional movement source for animated entities, allowing movement to be driven either by the controller or by animation data, depending on the use case.

## Conclusion

After reading (*hopefully*) all of this, you might be wondering why I’ve been implementing ideas from the README and previous posts, since I’ve admittedly drifted a little from the main objective of developing a voxel world.

When working on engine features that are not strictly voxel-related (and will be covered in future posts), some are generic enough to belong in most engines, whether voxel-based or traditional 2D/3D. Systems such as audio, UI, and particles fall into this category, and I will end up implementing them at some point.

Others, such as lighting, as mentioned earlier, tend to be more specific to voxel engines and often require completely different approaches. For example, instead of cubemaps, voxel engines often rely on procedural sky systems.

Animation and post-processing sit somewhere in between. While not strictly voxel-specific, they were added earlier to support longer-term goals such as animated world elements and a flexible rendering pipeline for future features like shadows and advanced effects.

Let’s close this post here. In the next one, I will shift focus back to the voxel engine and the progress made so far, returning to the main objective of the blog.