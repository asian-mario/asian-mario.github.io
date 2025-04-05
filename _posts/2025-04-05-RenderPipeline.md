---
layout: post

title: "ROUGE_2 : Rendering Pipeline + More"

date: 2025-04-05

author: "AsianMario"

categories: projects

tags: [proj]

image: rouge2.jpg
---

# ROUGE_2 : Basic 2D to Batched Rendering

## Initial Basic 2D Renderer and USP

A lot has happened since the past log, skipping few menial and minor commits, involving setting up multiple-API support along with the creation of a foundational 2D pipeline with textures and cameras, todays focus is the renderer and its progress. If you are interested on all of the commits made in this project, view the commit list [here](https://github.com/asian-mario/ROUGE2/commits/main/)

As the 2D Renderer is able to display the output in layers, different layers could be ordered in front of eachother, seperating them like a orthodox game engine (such as a GameLayer and a GUILayer).

In a new layer in the `SandboxApp` (where all layers are created and hosted) called `Sandbox2D` I tested a variety of new and experimental features.

Our starting point is going to be commit `49865a0` as I created a new `Renderer2D` module that makes it easy to draw textured quads (and other primititives in the future) in a 2D space, this intitial implementation used two shaders, one for flats and one for texturing, which exposed simple API calls such as `Renderer2D::DrawQuad()` for drawing both types of quads, for example:

```
Renderer2D::BeginScene(camera);
Renderer2D::DrawQuad({0.0f, 0.0f}, {1.0f, 1.0f}, {0.9f, 0.2f, 0.1f, 1.0f});       // colored quad
Renderer2D::DrawQuad({1.0f, -0.5f}, {0.8f, 0.8f}, {0.2f, 0.7f, 0.1f, 1.0f});      // another colored quad
Renderer2D::DrawQuad({0.0f, 0.0f, -0.5f}, {10.0f, 10.0f}, bgTexture, /*tint=*/1); // textured quad (background)
Renderer2D::DrawQuad({0.3f, 1.0f, 0.1f}, {0.8f, 0.8f}, spriteTexture, /*tint=*/1);
```

Under the hood, the render engine was setting up a quad vertex buffer, which would upload one quad at a time using the `RenderCommand::DrawIndexed()` function per quad, while this solution was functional it had a massive performance impact as each quad required its own draw call, ill-ideal when it comes to scenes which needed more than a few quads to render.

The next commit revamped the pipeline to use a single shader for all 2D drawing which elimanted the need to swap between a color shader and texture shader when rendering the different type of quads. Instead, the texture shader was modified to accept a colour tint uniform and default to a pure white texture (non-tinted) for coloured shapes, this meant all flat coloured shapes would be given a 'slate-texture' which essentially is a tiny 1x1 white texture which is generated bu the engine and bound whenever drawing asolid-colored quad, allowing a unified shader to handle the textured and solid quads. This new commit would then allow R2 to drop the `FlatColor.glsl` shader in favour for the one-size fits all `texture.glsl` shader. Furthermore, the engine's OpenGL texture class was also extended to support the creation of textures from scratch and to set data (used to intialize the 'empty' white texture). This now allowed quads to simply toggle a tint color and bind an actual sprite or ane mpty white texture with no shader program changes, this cleanup reduced state changes and siplified the API which discontinued the usage of SetColor or binding different shaders for different types of quads

## Integrating a Profiling/Instrumentation System (OSVI)

As performance and ensuring that the engine is running as smoothly as possible is critical, any bottlenecks need to be cought early before their damage to performance is too crtitical. To do this I implemented [OSVI](https://github.com/asian-mario/OSVI).

OSVI was created and implemented alongside the engine, as it started with a simple scoped timer utility that pushed any timing results into a list, which any layer then displayed in an ImGui window in each frame, this gave out a readout of frame times for certain code sections which could be changed by the developer, although this short-term solution was quickly superseded. 

Specifically in `8b4a174`, I fully integrated a full-suite OSVI profiler which enabled lightweight porfiling for recording profiling events and writing them into a JSON file to be able to be analyzed in tracing  tools. The engine now marks the beginning and end of key phases such as the initialization, runtime and shutdown and other important scope timings. In the `EntryPoint.h` I ensured that every stage of the engine had its own seperate profile which in-turn created seperate files to be created and analyzed later on. Furthermore, each render loop iteration and other important subsystems such as camera updates is instrumented with `OSVI_PROFILE_FUNCTION()` or `OSVI_PROFILE_SCOPE(name)` depending on the scope needed to be recorded. This detailed instrumentation for all things ROUGE2 from `Run()` loops to buffer vinds exposed an unexpected delay in GLFW window creation which took ~600ms which was later fixed.

During the OSVI integration, all remenants of ImGui diagnostics in favour for OSVI's global instrumentor, the on-screen diagnostics window is removed as the focus shifted to the offline profiling via JSON traces as this was a much more scalable option that dumping numbers on the screen on each frame. Although, this does not mean `spdlog` is considered to be useless as it still serves as a warning, error and trace systems to minor subsystems or when more information is required. The usage of OSVI as a whole served immensely helpful for the next major stage of the engine - optimizing the rendering process.

## Particles & API Refinements

With the core 2D rendering pipeline in place and profiling tools now available, I tested the current pipeline with a simple particle system, the idea was to have a pool of particles which could be emitted, updated and rendered efficiently, a worthwhile test for the engine's capability to handle many moving sprites and objects/indices in the area.

This introduced `ROUGE2::FX` with the `ParticleSystem` class along with a simple randomizer to generate random values for particle variation. By defaultm the particle system maintains a pool of 1000 particles of `Particle` structs, where an `Emit` function takes a set of `ParticleProps` which contained the position, velocity, color range, lifetime, etc. This would be used to initialize a new particle from the pool, particles are updated each frame as their positions alter based on velocity and reducing their remaining life, once a particle's `LifeRemaining` drops to 0 the particle is inactive and is added back to the pool.

In order to render these particles, I extended the Renderer2D API to support the existence of *rotated quads* and flexible tiling. A new `Renderer2D::DrawRotQuad()` overload was added that accepted a rotation angle in degrees, in addition to the position, size and colour/texture. In the background, the rotation is applied by calculating the quad's transform on the CPU before submitting the vertices (pre-batching implementation). In addition, the quad-drawing functions were refined to take a `tileScale` float variable. These API improvements made the function signatures more intuitive and allowed the user to specify the parameters the user needed, which meant any factors which were not needed such as tiling scale could simply be omitted from the function call.

In the Sandbox2D layer, I integrated the new particle system in order to get a visualization of its performance, on each update particles are emitted with random directions and colours as the particle system iterates through active particles and draws them via Render2D calls.

```
particleSystem.OnUpdate(ts);

Renderer2D::BeginScene(camera);
for (auto& particle : particleSystem) {
    // Compute interpolated color and size based on life remaining
    float lifeFactor = particle.LifeRemaining / particle.LifeTime;
    glm::vec4 color = glm::lerp(particle.ColorEnd, particle.ColorBegin, lifeFactor);
    float size = glm::lerp(particle.SizeEnd, particle.SizeBegin, lifeFactor);
    // Draw the particle as a rotated quad
    Renderer2D::DrawRotQuad(particle.Position, { size, size }, particle.Rotation, color);
}
Renderer2D::EndScene();
```

This stress test proved useful - with hundreds of particles being emitted and recycled, this allowed the engine to push a lot more draw calls which showed the need for optimization as the engine started dropping below 60fps with only a few hundred indices being rendered on screen. Although the shader swaps had been removed, issuing one draw call per quad made the engine extremely CPU-bound, being later confirmed by OSVI.

## Batch Renderer for 2D Rendering

The next series of commits starting roughly around July tackled this issue head-on by introducing a batch-renderer. This was a huge change as the entire 2D renderer had to be revamped, the idea was to buffer up as many quads into a single large vrtex buffer possible and send them in one go to drastically reduce the draw calls.

The initial basic batch renderer refactored Renderer2D to accumulate quads in a batch with a defined maximum limiting the number of quads per batch, also corresponding to the max vertices and indices, allocated in a large dynamic vertex buffer. Furthermore, `Renderer2D::BeginScene()` was changed to just reset a buffer pointer and `EndScene()` to flush the batch and issue the draw call when ready. Each call to `DrawQuad no longer immediately submitted to the GPU and instead wrote the quad's vertex data into the mapped buffer and icnremented a quad counter, if the batch buffer filled up to the maximum the renderer would flush the current batch and start a new one.

This initially was a disaster as one of the commits lead to the destruction of the whole shader binding API which caused visual artefacts during runtime, this was re-written in the subsequent commit. 

A limitation of the initial batch renderer was that it didn't support multiple textures in one batch, meaning that if you drew quads with two different texture assets, the renderer would flush the batch when the texture changed since a single draw call can only bind one texture at a time. In `a2e8bbf` solved this by also introducing a *texture index* in the vertex data and an array of texture slots in the shader. Now the renderer could bind, for example, 8 textures at once to different texture units and each quad vertex carries an index telling the shader which texture slot to sample from during the drawing process. The fragment shader was updated to sample from an array of samplers `u_Textures[]` using the texture index. The Renderer2D class maintains a list of textures in the current patch and if a new quad uses a textures that not yet in the list (presuming there is a free slot) it assigns it to the next slots. If all slots are filled then the current batch is flushed and a new batch is started. This method of batch rendering is only applicable to NVIDIA GPU's as AMD GPU's will have to initialize each array in the fragment shader before usage, as the current implimentation would cause a crash if any textures were to be binded. 

Finally, after re-implementing the particle system, the Renderer could draw 10000 quads in one draw call which massively improved performance, increasing it from the initial 40fps for 10000 quads in the scene to 800fps for the same number of quads in the scene.

## Optimizations and Cleanup

After the batch renderer change, the commits after was all about polish and optimizations. I optimized the batch renderer further and added internal stats tracking. Moreover, I seperated a `StartNewBatch()` function from the batch renderer to prevent duplicate code when a batch is flushed and restarted.

Additionally, I made sure that the large index buffer for all possible quad indices is created once and reused instead of regenerated which saved a bit of overhead, I also switched the dynamic vertex buffer usage to a more efficient strategy, using `glMapBufferRange` with a persistent mapped pointer which avoids extra allocations.

Well, that was it! Further steps is too add a tilemap for a tile-based system for creating simple maps in the game engine. Stay tuned for more!