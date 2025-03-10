.. _ocean-simulation-section:

Ocean Simulation
================

The following sections cover the major elements of the ocean simulation. All of these can be directly controlled with
user input, as covered in this video:

.. _adding-inputs-video:

.. youtube:: sQIakAjSq4Y

   Basics of Adding Ocean Inputs

.. tip::

   For inputs, you are not limited to only using a :link:`MeshRenderer <{UnityDocsLinkBase}class-MeshRenderer.html>`.
   Almost any renderer can be used like a :link:`TrailRenderer <{UnityDocsLinkBase}class-TrailRenderer.html>`, :link:`LineRenderer <{UnityDocsLinkBase}class-LineRenderer.html>` or :link:`ParticleSystem <{UnityDocsLinkBase}class-ParticleSystem.html>`.

The following shaders can be used with any ocean input:

-  **Scale By Factor** scales the ocean data between zero and one inclusive.
   It is multiplicative, which can be inverted, so zero becomes no data and one leaves the data unchanged.


.. _animated-waves-section:

Animated Waves
--------------

Overview
^^^^^^^^

The Animated Waves simulation contains the animated surface shape.
This typically contains the ocean waves (see the Wave conditions section below), but can be modified as required.
For example, parts of the water can be pushed down below geometry if required.

The animated waves sim can be configured by assigning an Animated Waves Sim Settings asset to the OceanRenderer script in your scene (:menuselection:`Create --> Crest --> Animated Wave Sim Settings`).

The waves will be dampened/attenuated in shallow water if a *Sea Floor Depth* LOD data is used (see :ref:`sea-floor-depth-section`).
The amount that waves are attenuated is configurable using the *Attenuation In Shallows* setting.


.. _animated_waves_settings:

Simulation Settings
^^^^^^^^^^^^^^^^^^^

All of the settings below refer to the *Animated Waves Sim Settings* asset.

-  **Attenuation In Shallows** - How much waves are dampened in shallow water.
-  **Shallows Max Depth** - Any water deeper than this will receive full wave strength.
   The lower the value, the less effective the depth cache will be at attenuating very large waves.
   Set to the maximum value (1,000) to disable.
-  **Collision Source** - Where to obtain ocean shape on CPU for physics / gameplay.
-  **Max Query Count** - Maximum number of wave queries that can be performed when using ComputeShaderQueries.
-  **Ping Pong Combine Pass** - Whether to use a graphics shader for combining the wave cascades together.
   Disabling this uses a compute shader instead which doesn't need to copy back and forth between targets, but it may not work on some GPUs, in particular pre-DX11.3 hardware, which do not support typed UAV loads.
   The fail behaviour is a flat ocean.
-  **Render Texture Graphics Format** - The render texture format to use for the wave simulation.
   Consider using higher precision (like R32G32B32A32_SFloat) if you see tearing or wierd normals.
   You may encounter this issue if you use any of the *Set Water Height* inputs.


User Inputs
^^^^^^^^^^^

.. admonition:: Preview

   Splines now support animated waves. Please see :ref:`wave-splines-section` for general spline information.

To add some shape, add some geometry into the world which when rendered from a top down perspective will draw the desired displacements.
Then assign the *Register Anim Waves Input* script which will tag it for rendering into the shape.
This is demonstrated in :numref:`adding-inputs-video`

There is an example in the *boat.unity* scene, GameObject *wp0*, where a smoothstep bump is added to the water shape.
This is an efficient way to generate dynamic shape.
This renders with additive blend, but other blending modes are possible such as alpha blend, multiplicative blending, and min or max blending, which give powerful control over the shape.

The following input shaders are provided under *Crest/Inputs/Animated Waves*:

-  **Add From Texture** allows any kind of shape added to the surface from a texture.
   Can ether be a heightmap texture (1 channel) or a 3 channel XYZ displacement texture.
   Optionally the alpha channel can be used to write to subsurface scattering which increases the amount of light emitted from the water volume, which is useful for approximating light scattering.
-  **Scale By Factor** scales the waves by a factor where zero is no waves and one leaves waves unchanged.
   Useful for reducing waves.
-  **Set Base Water Height Using Geometry** allows the sea level (average water height) to be offset some amount.
   The top surface of the geometry will provide the water height, and the waves will apply on top.
-  **Push Water Under Convex Hull** pushes the water underneath the geometry.
   Can be used to define a volume of space which should stay 'dry'.
-  **Set Water Height Using Geometry** snaps the water surface to the top surface of the geometry.
   Will override any waves.
-  **Wave Particle** is a 'bump' of water.
   Many bumps can be combined to make interesting effects such as wakes for boats or choppy water.
   Based loosely on http://www.cemyuksel.com/research/waveparticles/.

.. _dynamic-waves-section:

Dynamic Waves
-------------

Overview
^^^^^^^^

Crest includes a multi-resolution dynamic wave simulation, which allows objects like boats to interact with the water.

To turn on this feature, enable the *Create Dynamic Wave Sim* option on the *OceanRenderer* script, and to configure the sim, create or assign a *Dynamic Wave Sim Settings* asset on the *Sim Settings Dynamic Waves* option.

Dynamic ripples from interacting objects can be generated by placing one or more spheres under the object to approximate the object's shape.
To do so, attach one or more *SphereWaterInteraction* components to children with the object and set the *Radius* parameter.

Non-spherical objects can be approximated with multiple spheres, for an example see the *Spinner* object in the *boat.unity* example scene which is composed of multiple sphere interactions.
The intensity of the interaction can be scaled using the *Weight* setting.
For an example of usages in boats, search for GameObjects with "InteractionSphere" in their name in the *boat.unity* scene.

The dynamic wave simulation is added on top of the animated FFT waves to give the final shape.


.. _dynamic_waves_settings:

Simulation Settings
^^^^^^^^^^^^^^^^^^^

All of the settings below refer to the *Dynamic Wave Sim Settings* asset.

-  **Simulation Frequency** - Frequency to run the dynamic wave sim, in updates per second.
   Lower frequencies can be more efficient but may limit wave speed or lead to visible jitter.
   Default is 60 updates per second.

-  **Damping** - How much energy is dissipated each frame.
   Helps sim stability, but limits how far ripples will propagate.
   Set this as large as possible/acceptable.
   Default is 0.05.

-  **Courant Number** - Stability control.
   Lower values means more stable sim, but may slow down some dynamic waves.
   This value should be set as large as possible until sim instabilities/flickering begin to appear.
   Default is 0.7.

-  **Horiz Displace** - Induce horizontal displacements to sharpen simulated waves.

-  **Displace Clamp** - Clamp displacement to help prevent self-intersection in steep waves.
   Zero means unclamped.

-  **Gravity Multiplier** - Multiplier for gravity.
   More gravity means dynamic waves will travel faster.

-  **Attenuation in Shallows** - How much waves are dampened in shallow water.

The *OceanDebugGUI* script gives the debug overlay in the example content scenes and reports the number of sim steps taken each frame.


User Inputs
^^^^^^^^^^^

The recommended approach to injecting forces into the dynamic wave simulation is to use the *SphereWaterInteraction* component as described above.
This component will compute a robust interaction force between a sphere and the water, and multiple spheres can be composed to model non-spherical shapes.

However it is possible to inject custom forces into the simulation. The following input shader is provided under *Crest/Inputs/Dynamic Waves*:

-  **Add Bump** adds a round force to pull the surface up (or push it down).
   This can be moved around to create interesting effects.


.. _foam-section:

Foam
----

Overview
^^^^^^^^

Crest simulates foam getting generated by choppy water (*pinched*) wave crests) and in shallow water to approximate foam from splashes at shoreline.
Each update (default is 30 updates per second), the foam values are reduced to model gradual dissipation of foam over time.

To turn on this feature, enable the *Create Foam Sim* option on the *OceanRenderer* script, and ensure the *Enable* option is ticked in the Foam group on the ocean material.

To configure the foam sim, create a *Foam Sim Settings* asset by right clicking the a folder in the *Project* window and selecting *Create/Crest/Foam Sim Settings*, and assigning it to the OceanRenderer component in your scene.


User Inputs
^^^^^^^^^^^

.. admonition:: Preview

   Splines now support foam. Please see :ref:`wave-splines-section` for general spline information.

Crest supports inputing any foam into the system, which can be helpful for fine tuning where foam is placed.
To place foam, add some geometry into the world at the area where foam should be added.
Then assign the *RegisterFoamInput* script which will tag it for rendering into the shape, and apply a material with a shader of type *Crest/Inputs/Foam/...*.
See the *DepositFoamTex* object in the *whirlpool.unity* scene for an example.

The process for adding inputs is demonstrated in this :numref:`adding-inputs-video`.

The following input shaders are provided under *Crest/Inputs/Foam*:

-  **Add From Texture** adds foam values read from a user provided texture.
   Can be useful for placing 'blobs' of foam as desired, or canbe moved around at runtime to paint foam into the sim.

-  **Add From Vert Colours** can be applied to geometry and uses the red channel of vertex colours to add foam to the sim.
   Similar in purpose to *Add From Texture*, but can be authored in a modelling workflow instead of requiring at texture.

-  **Override Foam** sets the foam to the provided value.
   Useful for removing foam from unwanted areas.


Simulation Settings
^^^^^^^^^^^^^^^^^^^

General Settings
~~~~~~~~~~~~~~~~

-  **Foam Fade Rate** - How quickly foam dissipates.
   Low values mean foam remains on surface for longer.
   This setting should be balanced with the generation *strength* parameters below.


Wave foam / whitecaps
~~~~~~~~~~~~~~~~~~~~~

Crest detects where waves are 'pinched' and deposits foam to approximate whitecaps.

-  **Wave Foam Strength** - Scales intensity of foam generated from waves.
   This setting should be balanced with the *Foam Fade Rate* setting.

-  **Wave Foam Coverage** - How much of the waves generate foam.
   Higher values will lower the threshold for foam generation, giving a larger area.


Shoreline foam
~~~~~~~~~~~~~~

If water depth input is provided to the system (see **Sea Floor Depth** section below), the foam sim can automatically generate foam when water is very shallow, which can approximate accumulation of foam at shorelines.

-  **Shoreline Foam Max Depth** - Foam will be generated in water shallower than this depth.
   Controls how wide the band of foam at the shoreline will be.
   Note that this is not a distance to shoreline, but a threshold on water depth, so the width of the foam band can vary
   based on terrain slope.
   To address this limitation we allow foam to be manually added from geometry or from a texture, see the next
   section.

-  **Shoreline Foam Strength** - Scales intensity of foam generated in shallow water.
   This setting should be balanced with the *Foam Fade Rate* setting.


Developer Settings
~~~~~~~~~~~~~~~~~~

These settings should generally be left unchanged unless one is experiencing issues.

-  **Simulation Frequency** - Frequency to run the foam sim, in updates per second.
   Lower frequencies can be more efficient but may lead to visible jitter.
   Default is 30 updates per second.


.. _sea-floor-depth-section:

Sea Floor Depth
---------------

This simulation stores information that can be used to calculate the water depth.
Specifically it stores the terrain height, which can then be differenced with the sea level
to obtain the water depth.
This water depth is useful information to the system; it is used to attenuate large waves in
shallow water, to generate foam near shorelines, and to provide shallow water shading.
It is calculated by rendering the render geometry in the scene for each LOD from a top down perspective and recording the Y value of the surface.

The following will contribute to ocean depth:

-  Objects that have the *RegisterSeaFloorDepthInput* component attached.
   These objects will render every frame.
   This is useful for any dynamically moving surfaces that need to generate shoreline foam, etcetera.

-  It is also possible to place world space depth caches.
   The scene objects will be rendered into this cache once, and the results saved.
   Once the cache is populated it is then copied into the Sea Floor Depth LOD Data.
   The cache has a gizmo that represents the extents of the cache (white outline) and the near plane of the camera that renders the depth (translucent rectangle).
   The cache should be placed at sea level and rotated/scaled to encapsulate the terrain.

When the water is e.g. 250m deep, this will start to dampen 500m wavelengths, so it is recommended that the sea floor drop down to around this depth away from islands so that there is a smooth transition between shallow and deep water without a visible boundary.

.. _clip-surface-section:

Clip Surface
------------

.. youtube:: jXphUy__J0o

   Water Bodies and Surface Clipping

This data drives clipping of the ocean surface, as in carving out holes.
This can be useful for hollow vessels or low terrain that goes below sea level.
Data can come from primitives (signed-distance), geometry (convex hulls) or a texture.

To turn on this feature, enable the *Create Clip Surface Data* option on the *OceanRenderer* script, and ensure the *Enable* option is ticked in the *Clip Surface* group on the ocean material.

The data contains 0-1 values. Holes are carved into the surface when the value is greater than 0.5.


.. _clip_surface_settings:

Simulation Settings
^^^^^^^^^^^^^^^^^^^

All of the settings below refer to the *Clip Surface Sim Settings* asset.

-  **Render Texture Graphics Format** - The render texture format to use for the clip surface simulation.
   Consider using higher precision (like *R16_UNorm*) if you are using *Primitive* mode for even more accurate clipping.


.. _clip_surface_inputs:

User Inputs
^^^^^^^^^^^

Primitive Mode
~~~~~~~~~~~~~~

Clip areas can be added using signed-distance primitives which produces accurate clipping and supports overlapping.
Add a *RegisterClipSurfaceInput* script to a *GameObject* and set *Mode* to *Primitive*.
The position, rotation and dimensions of the primitive is determined by the *Transform*.
See the *FloatingOpenContainer* object in the *boat.unity* scene for an example usage.

Geometry Mode
~~~~~~~~~~~~~

Clip areas can be added by adding geometry that covers the desired hole area to the scene and then assigning the *RegisterClipSurfaceInput* script and setting *Mode* to *Geometry*.
See the *RowBoat* object in the *main.unity* scene for an example usage.

To use other available shaders like *ClipSurfaceRemoveArea* or *ClipSurfaceRemoveAreaTexture*: create a material, assign to renderer and disable *Assign Clip Surface Material* option.
For the *ClipSurfaceRemoveArea* shaders, the geometry should be added from a top-down perspective and the faces pointing upwards.

The following input shaders are provided under *Crest/Inputs/Clip Surface*:

-  **Convex Hull** - Renders geometry into clip surface data taking all dimensions into account.
   An example use case is rendering the convex hull of a vessel to remove the ocean surface from within it.
   See the *RowBoat* object in the *main.unity* scene for an example usage.

   .. note::

      Overlapping or adjacent meshes will not work correctly in most cases.
      There will be cases where one mesh will overwrite another resulting in the ocean surface appearing where it should not.
      The mesh is rendered from a top-down perspective.
      The back faces add clip surface data and the front faces remove from it which creates the convex hull.
      With an overlapping mesh, the front faces of the sides of one mesh will clear the clipping data creating by the other mesh.
      Overlapping boxes which are not rotated on the X or Z axes will work well whilst spheres will have issues.
      Consider using *Primitive* mode which supports overlapping.

-  **Include Area** - Removes clipping data so the ocean surface renders.

-  **Remove Area** - Adds clipping data to remove the ocean surface.

-  **Remove Area Texture** - Adds clipping data using a texture to remove the ocean surface.


.. _shadows-section:

Shadows
-------

The shadow data consists of two channels.
One is for normal shadows (hard shadow term) as would be used to block specular reflection of the light.
The other is a much softer shadowing value (soft shadow term) that can approximately variation in light scattering in the water volume.

This data is captured from the shadow maps Unity renders before the transparent pass.
These shadow maps are always rendered in front of the viewer.
The Shadow LOD Data then reads these shadow maps and copies shadow information into its LOD textures.


.. only:: birp

   .. tab:: `BIRP`

      .. include:: includes/_birp-shadows.rst

.. only:: hdrp

   .. tab:: `HDRP`

      .. include:: includes/_hdrp-shadows.rst

.. only:: urp

   .. tab:: `URP`

      .. include:: includes/_urp-shadows.rst

The shadow sim can be configured by assigning a Shadow Sim Settings asset to the OceanRenderer script in your scene (*Create/Crest/Shadow Sim Settings*).
In particular, the soft shadows are very soft by default, and may not appear for small/thin shadow casters.
This can be configured using the *Jitter Diameter Soft* setting.

There will be times when the shadow jitter settings will cause shadows or light to leak.
An example of this is when trying to create a dark room during daylight.
At the edges of the room the jittering will cause the ocean on the inside of the room (shadowed) to sample outside of the room (not shadowed) resulting in light at the edges.
Reducing the *Jitter Diameter Soft* setting can solve this, but we have also provided a *Register Shadow Input* component which can override the shadow data.
This component bypasses jittering and gives you full control.

.. Note: RP should allow sampling the shadow maps directly in the ocean shader which would be an alternative to using this shadow data, although it would not give the softer shadow component. This would likely work on 2018.

.. _flow-section:

Flow
----

Overview
^^^^^^^^

Flow is the horizontal motion of the water volumes.
It is used in the *whirlpool.unity* example scene to rotate the waves and foam around the vortex.
It does not affect wave directions, but transports the waves horizontally.
This horizontal motion also affects physics.

User Inputs
^^^^^^^^^^^

.. admonition:: Preview

   Splines now support flow. Please see :ref:`wave-splines-section` for general spline information.

Crest supports adding any flow velocities to the system.
To add flow, add some geometry into the world which when rendered from a top down perspective will draw the desired displacements.
Then assign the *RegisterFlowInput* script which will tag it for rendering into the flow, and apply a material using one of the following shaders.

The following input shaders are provided under *Crest/Inputs/Flow*:

The *Crest/Inputs/Flow/Add Flow Map* shader writes a flow texture into the system.
It assumes the x component of the flow velocity is packed into 0-1 range in the red channel, and the z component of the velocity is packed into 0-1 range in the green channel.
The shader reads the values, subtracts 0.5, and multiplies them by the provided scale value on the shader.
The process of adding ocean inputs is demonstrated in :numref:`adding-inputs-video`.
