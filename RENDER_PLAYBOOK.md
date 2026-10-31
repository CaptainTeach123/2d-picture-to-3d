# Render playbook

Lessons from real render projects, written for whoever runs the next render (mostly Claude). Each line comes from
something that went wrong or was measured on a project, not from general advice.

Tags: **[W]** Windsor Hotel, **[S1]** Stahl House, **[S2]** Stahl House v2, **[C]** Cliff House, **[N]** The Starry Night.

- Read **Preflight** before every render.
- Read **Review** after every render.
- Read the topic sections before planning a new scene.

## Preflight

<!-- preflight:start -->
1. **Right file.** Is this blend what the scripts build? Rebuild and diff before a long render; a stale lamp slipped through once. [S1]
2. **Right frame.** In an animated blend, set the frame yourself. Stills, masks and previews all depend on it. [C]
3. **Keys win.** Keyed values (`hide_render`, light energy, scene properties) override a script's change at render time. Clear those keys first, or the change silently does nothing. [C]
4. **Still before sequence.** Render one final-resolution still and review it in crops before launching any sequence. It took 18 s; the review that came after the hour-long job cost a 35-minute re-render. [C]
5. **Preview the whole timeline.** After the still, render every 6th frame at half size and read a contact sheet. It took 3 minutes and found 4 problems before a 32-minute render. [N]
6. **Right size for the question.** Water needs full-frame renders, because screen-space reflections vanish in border crops. Fine detail needs final resolution: 1536 px tests hid defects that 3840 px showed. [S2] [C]
7. **Camera.** The locked camera is never animated. Lens shift is measured in the fitted sensor side. Late shots stay inside the area that has geometry. [C]
8. **Outputs.** Test renders go to a test folder with an explicit path, never over a deliverable. If a build step fails, stop: renders from the old blend look like real results. [N]
9. **One heavy job at a time.** Concurrent Blender renders run out of memory. Long jobs run in the background and must be resumable. [C]
10. **FINISHED but no file?** Check for an empty compositor with `use_compositing` on, then for a damaged Blender install. [S2]
<!-- preflight:end -->

## Review

1. Look at final-resolution crops next to the reference, tile by tile. A downscaled overview does not count. [S1] [C]
2. Diff against the previous render and confirm only the intended areas changed. [C]
3. If a metric collapses suddenly, suspect the check before the scene. Building IoU read 0.24 because the mask was wrong, not the model. [C]
4. A good metric does not prove the image is right. Landmark lines passed while the palm species, rafter direction and rocks on the paving were wrong, and decor sat 40 m away. [S1]
5. Outputs that should differ must differ. Every "studio" preview was a copy of its golden-hour pair, unnoticed until the index sheet was viewed. [C]
6. For video, decode frames from the final MP4 and diff across every cut. Use contact sheets at 1 s to find idle stretches. [C]
   Measure flicker on frames where the camera is still; a moving camera gives large differences that mean nothing. Check signed colour shift as well as absolute difference, to tell compression from a real shift. [N]
7. Log every conflict with the reference as a numbered discrepancy with the decision made. [C]
8. Add what you learned to this file, as one line with a project tag.

## References and fidelity

- Measure the reference; do not assume. Guessed signs, fire escapes and street-light counts each cost a correction round. Iconic features get the most verification time. [W]
- Prefer a real photograph's solved camera over a painted image for geometry. [W]
- Paintings are not physically consistent: facade lines disagree, the horizon tilted 0.26 deg, and one low sun could not light both painted faces. Fit by least squares, use key + fill, and log the choice. [C]
- Do not chase a colour box that compares different things. Where painted and modelled detail differ in position (windows, arcade), the number is meaningless. [C]
- Measure before "fixing". A building that looked lit measured as a shade tone, so a light added for it was reverted. [C]
- Removing text from a reference: an SDXL inpaint left ghost letters. A deterministic fill (normalised convolution plus random-phase grain sampled from sky above the horizon) worked. Verify that pixels outside the mask are byte-identical. [C]

## Camera

- Solve a level two-point perspective with lens shift by least squares on facade lines in pixels. Solve a reference photograph's camera with PnP against the model. [S1] [C]
- Blender lens shift is in units of the fitted sensor side (VERTICAL fit: height), not the larger frame side. Getting this wrong made a cut jump 60 px. [S1] [C]
- Place elements through the camera (a point at a given depth along a pixel's ray), so they land where the artwork draws them. [S1]
- Camera-registered reliefs only hold from the locked camera. Later construction shots must stand at that position (pan, tilt, zoom only), and a script should prove no frame looks past the painted area. [C]

## Geometry

- Relief outlines on a pixel grid are a staircase that shows at high resolution and casts comb-toothed shadows. Smooth along the outline in image space at fixed depth. [C]
- Changing a decimated mesh reshuffles its geometry-nodes scatter (shrubs went 153 -> 1137). Compare instance counts after any mesh change. A rigid move of a loose part leaves the rest unchanged. [C]
- Tiny relief islands at the waterline read as posts, because their skirts show in swell troughs. Sink them after the ocean foam is computed so the foam stays. [C]
- `bmesh.ops.create_icosphere(subdivisions=1)` is a 12-vertex icosahedron. Small ones read as dice; use at least 2. [C]
- Something rising from a point projects upward in the image. To show spray through a gap, start it at the waterline below the gap. [C]
- Diagnose gaps with a ray-cast, then judge fixes by eye. A skirt read as a picket fence, and a relief extension hid the piers. A strip of ground behind the wall worked. [C]
- Displace with an IMAGE texture and LOCAL coordinates streaks on steep faces; use 3D textures. [S2]
- SUBSURF on 300k-triangle canopy meshes crashed Blender 5.2 render evaluation; build the density into the mesh. [S2]
- Blender 5.2 geometry-nodes modifier inputs are `md.properties.inputs.Socket_N.value`, numbered from 0, not the interface default. [C]

## Materials and colour

- Stylized core: light through Shader-to-RGB into a smoothstep between lit and shade colours sampled from the reference with k-means. A higher Bias gives more shade. [S2] [C]
- Real scan detail goes on as a mean-1 multiplier, so texture adds detail without moving calibrated colours. [S2]
- Leave headroom in bright lit colours, because they clip under texture detail. [S2]
- Calibrate per material and check the result. Automatic ID-mask calibration is unreliable where objects overlap, and it turned yuccas navy. Thin objects drift; sample those by hand. [S1] [S2]
- Compare tone shares, not only means. Rocks were lit on 45 % of their area against 25 % painted. [C]
- Keep CC0 sources recorded next to the textures (Poly Haven scans worked well). [S2]

## Water and sky

- If an ocean goes white or foamless, check that the rock-proximity target is actually evaluated; it read 0 m everywhere once. Object Info nodes fixed it. [C]
- Evaluate swell per pixel in the shader, and match the swell spacing to the reference's rows. [C]
- A grey far sea can be the grazing reflection of the horizon haze. Scale far specular down. [C]
- Sample sky ramps with many stops near the horizon, where the colour changes fastest. [C]

## Lighting

- EEVEE has light linking but not shadow linking. [S2]
- An interior spot just under a ceiling glares on it; exclude the ceiling from its receivers. [S2]
- A per-object highlight works in EEVEE through an animated custom property read by an OBJECT Attribute node. [C]

## Timelapse

- Late objects need something to stand on. Rocks hung in a void until a sea-level datum carried them. [C]
- Check that each phase visibly changes. Clay windows popping into clay openings read as 10 idle seconds; a warm flash as each piece lands fixed it. [C]
- 9:16 needs its own framing, ending on an exact window of the locked view so a pull-out can land on the band. [C]
- Crossfade from the last reveal frame into the held still, or sampling differences pop at the join. [C]
- Partial re-render: find when the changed object first appears, delete frames from there, and let a resumable renderer fill the gap. [C]
- Drive heavy animation from plain functions of the frame number, with no keys: 756,000 vertices updated from numpy tables each frame, at 1.2 s per frame in total. Nothing in the file can override it, though the blend will not play by itself. [N]
- Objects waiting to appear still cast shadows, even collapsed to nothing. Park them below the ground. [N]
- A slow reveal needs checking in seconds per stage. A noise-threshold wipe that spans 5 s on paper can open in 1. [N]

## Tooling

- Run Blender with `-b --factory-startup` and enable any needed add-ons inside the script. [S2]
- Run analysis Python with `-P`. [C]
- Long Bash heredocs with nested quotes can fail. Write patch scripts to a file instead. [C]
- Chain build and render steps so a failure stops the chain. [N]
- Perceptually lossless H.264 at 2728 px came to 765 MB for 58 s of detailed motion. Encode a smaller share version as well (189 MB at 1364 px). [N]
- Decode huge JPEGs at reduced scale with PIL `draft`: a 696 MB scan decoded at 1/4 scale in 6 s. [N]
- If background renders FINISHED but write nothing, or `import idna` fails, check the install: files had gone missing from the Blender folder, with a cleanup tool the suspected cause. [S2]
- StableGen multi-view runs need free RAM, and its scenes carry an empty compositor tree: set `scene.render.use_compositing = False`. [S2]

## Animating a painting

- Test the riskiest idea in the cheapest medium first. A 2D test showed moving strokes held up before any 3D was built. [N]
- Make motion from the artwork's own marks: split it into strokes that each carry their own original pixels, rather than warping the image. [N]
- Give pieces a margin of their neighbours' true pixels, opened only once all are placed. Small moves then open almost no gaps, and at rest the image is unchanged. Keep edges tight while pieces land, or the margin reads as a halo. [N]
- Have each piece ride its own flow line and send a travelling wave through the phases, so it reads as flow while every piece can return exactly home. [N]
- Keep a single value that mixes lit shading with the flat scan, so the piece can land on the original's exact colours, then crossfade to the scan itself. [N]
- Take material from the reference when it exists. A procedural weave looked synthetic; linen quilted from the scan's bare edge looked real. Avoid mirrored tiles, which show symmetry. [N]
- Say what is estimated. Stroke outlines, paint height and painting order here are estimates, and the report has to state that. [N]

## Working with the owner

- Reference fidelity is the main quality bar. Double-check before presenting. [W] [S1]
- "Next project" with no brief means ask for the brief; do not resume set-aside work. [W]
- Do not resume or offer cancelled work. Delivered projects stay as delivered unless asked. [W]
