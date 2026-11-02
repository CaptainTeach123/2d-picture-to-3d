# Render playbook

Lessons from real render projects, written for whoever runs the next render (mostly Claude). Each line comes from
something that went wrong or was measured on a project, not from general advice.

Tags: **[W]** Windsor Hotel, **[S1]** Stahl House, **[S2]** Stahl House v2, **[C]** Cliff House, **[N]** The Starry Night, **[M]** Impression, Sunrise.

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
   Clean frames do not prove the motion makes sense. Every frame and flicker check passed on a film the owner rejected at first sight. Measure the motion itself: share moving backward, reversals per piece, neighbours moving against each other, and block matching on rendered frames against the intended direction. [N]
   Blank frames and changing content fake camera and sharpness spikes: phase correlation on bare canvas read 324 px, and a fixed crop read a 23 % sharpness step that was a smooth 5 % drift on painted pixels alone. Measure on painted pixels before trusting a spike. [N]
   A last frame that matches the reference says nothing about the middle of a build. A film that ended on 0 differing values still showed stripes for 12 minutes, and in a later version a flat patch until the last touches covered it. Read final-size frames from every stage. [M]
   To prove a repair stayed in its area, carry each frame's differences back through the camera onto the canvas and test them against the repair's mask. Differences at every 12 s came to 21,731 canvas cells, all inside and none out. [M]
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
- A repair that leaves earlier strokes alone needs only the later chunks. Run the plan forward unfilmed to the first changed chunk (4,515 strokes took 61 s), film from there, and join to the old chunks. Six of ten chunks took 62 minutes, and the join matched the old film. [M]
- Drive heavy animation from plain functions of the frame number, with no keys: 756,000 vertices updated from numpy tables each frame, at 1.2 s per frame in total. Nothing in the file can override it, though the blend will not play by itself. [N]
- Objects waiting to appear still cast shadows, even collapsed to nothing. Park them below the ground, and for a reveal along a piece, park the part not yet reached the same way; left at height it stands as a fin. [N]
- A slow reveal needs checking in seconds per stage. A noise-threshold wipe that spans 5 s on paper can open in 1. [N]

## Tooling

- Run Blender with `-b --factory-startup` and enable any needed add-ons inside the script. [S2]
- Run analysis Python with `-P`. [C]
- Long Bash heredocs with nested quotes can fail. Write patch scripts to a file instead. [C]
- Chain build and render steps so a failure stops the chain. [N]
- Black patches with straight edges on stacked transparent surfaces in EEVEE (dithered) were depth ties, not textures: strokes 0.1 micron apart. Keep overlapping pieces at least 10 microns apart and narrow the camera clip range around the subject. 3,025 black pixels fell to 80. [N]
- To find what a black patch is, swap one thing per render: canvas to magenta, world to green, colour to constant. It took 3 renders to rule out all three, then a list of the quads over the spot showed the tie. [N]
- Perceptually lossless H.264 at 2728 px came to 765 MB for 58 s of detailed motion. Encode a smaller share version as well (189 MB at 1364 px). [N]
- A 12-minute detailed film at 2046 px took 52 GB of q95 JPEG frames and 3.66 GB at HIGH, and left the disk 98 % full. Add up frames and old passes before a long film. [N]
- Pipe raw frames straight into the encoder in 3-minute chunks, saving the canvas after each one. A 30-minute film came to 1.7 GB at crf 16, kept no frames on disk, and could resume after any chunk. [M]
- The `imageio-ffmpeg` wheel from PyPI unzips to a full ffmpeg binary with nothing installed. Check its SHA-256 against PyPI, and note that it has no ffprobe. [M]
- FFmpeg colour flags on the encoder did not reach the stream, which read `unknown`. Set them with the `h264_metadata` bitstream filter on the join, and `setparams` before a re-encode, then read the stream line. [M]
- x264 explains two false alarms: low-change runs that break every 8.3 s are keyframes every 250 frames, and a chunk's first keyframe gives a one-frame step of mean 1.1 (signed 0) on a still picture. [M]
- Compare colour with the matrix written out by hand before trusting the eye. Test JPEGs looked more saturated than the video and measured 21.42 against 21.40. [M]
- float32 means over a whole frame drift by several levels, which faked a colour error on the first frame. Use float64 for statistics. [M]
- A whole-image FFT smears canvas weave peaks because the weave drifts across the canvas. Read the peaks from tiles. [M]
- Decode huge JPEGs at reduced scale with PIL `draft`: a 696 MB scan decoded at 1/4 scale in 6 s. It stops at the nearest power of 2, so resize after it if the size matters. [N]
- If background renders FINISHED but write nothing, or `import idna` fails, check the install: files had gone missing from the Blender folder, with a cleanup tool the suspected cause. [S2]
- StableGen multi-view runs need free RAM, and its scenes carry an empty compositor tree: set `scene.render.use_compositing = False`. [S2]

## Animating a painting

- Test the riskiest idea in the cheapest medium first. A 2D test showed moving strokes held up before any 3D was built. [N]
- Make motion from the artwork's own marks: split it into strokes that each carry their own original pixels, rather than warping the image. [N]
- Give pieces a margin of their neighbours' true pixels, opened only once all are placed. Small moves then open almost no gaps, and at rest the image is unchanged. Keep edges tight while pieces land, or the margin reads as a halo. [N]
- A travelling wave of pieces sliding back and forth reads as jitter, not flow, however smooth it is. It was rejected on sight: every stroke reversed each 1.2 s and went nowhere. Motion needs a cause the viewer can name, and each part moves as what it is: wind flows, a tree bends from its foot, land stays still. [N]
- For a steady flow, keep the still artwork underneath and run moving copies over it, so the pattern stays and paint travels through it, like water in a river. Each piece runs forward, lifts away from its tail and restarts hidden at home. Nothing has to reverse, no gap opens, and the film lands exactly on the original. [N]
- A line field has no direction. Get it from a stream function fitted to the lines (least squares on grad psi along the line, with psi fixed on the walls and a source at each eye). It solved in 1 s and cut neighbours moving against each other from 25 % to 0.7 %, where growing signs over a cell graph could not make eddies turn opposite ways. [N]
- Move pieces as bodies on one track through their centre. Letting each vertex follow its own flow line fanned and smeared them near eyes. [N]
- End each run before a piece reaches a clearly different tone, and slow pieces towards an eye and stop them short. Dark accents crossing pale paint and strokes piling into a tight spiral both looked wrong. [N]
- A conveyor's start delay sets its spacing, scaled by 1 - g x run for delay g x distance. At g = 1/run every piece lands in one place. Use about 1/(3 x run), positive, so changes travel downstream, and give each stream its own offset so they never form a line. [N]
- Everything on screen should come from one process. A scrubbed-on colour wash under the strokes read as a foreign blue bleed; strokes straight onto bare linen did not. [N]
- Start a build where the camera starts, and grow it from painted areas outward (Dijkstra over the cell graph, cheap along strokes, dear across them). Random order within a layer looks like popcorn. [N]
- A still neighbour must not carry a moving object's pixels in its overlap margin: it leaves a ghost outline when the object moves. Clear them from the margin and give the object something real to reveal behind it. [N]
- If pieces are lit in 3D, keep a single value that mixes lit shading with the flat scan, so the piece can land on the original's exact colours, then crossfade to the scan itself. A single 2D pass needs neither (see below). [N]
- Take material from the reference when it exists. A procedural weave looked synthetic; linen quilted from the scan's bare edge looked real. Avoid mirrored tiles, which show symmetry. Check the colour against sources too: the bare tan linen was wrong, the ground was off-white priming, so only the weave's value was kept. [N]
- Say what is estimated. Stroke outlines, paint height and painting order here are estimates, and the report has to state that. [N]
- Read the technical study of the artwork before planning stages. A sketch stage from general knowledge contradicted the canvas examination (no underdrawing found), which also gave the real order: reserves, mid-blue first, stars after. Separate what is documented from your choices in the report. [N]
- For "how it was painted", fix stroke rate before anything else. Hundreds of strokes a second read as a morph however fine they are and were rejected; one at a time at about 10 a second reads as painting, at 12 minutes for 6,000 strokes. [N]
- Even at one at a time, 10 strokes a second leaves 79 % of strokes forming in 3 frames or fewer at 30 fps. They land more than they draw out. Half a second each would run an hour, so raise that trade-off early. [N]
- Agree film length first, because it sets the stroke count. 4,862 strokes over 30 minutes gave a median 0.26 s to form, 92 % at 6 frames or more and none at 3 or fewer. [M]
- One pass, each stroke laying the scan's own pixels once with flat settled height and wet ridges that decay in 0.45 s, lands on the original (mean 2.7 of 255, none over 40) with no fade. Layered passes without a done mask overwrote finished paint and needed a fade that hid 4.2 % gaps. [N]
- Layers can land exactly with no fade too. Aim them at the plate blurred to brush size (sigma 70, 30, 12, 5), then finish with strokes that lay the plate's own pixels, and let coverage above 0.55 mark a pixel done for good. The canvas ended on the scan with 0 values differing. [M]
- Simulate a stroke in brush space, one row across the bristles per 2 px, with each bristle's load and colour and pickup of wet paint. Split tips, dry-brush on thread tops and soft wet edges all follow from it with no texture. Pickup had to come down to an eighth of its first value. [M]
- Inertia and turn limits stop strokes following a direction field into worm shapes, and holes must be masked out of the field first. [M]
- Finishing on a grid showed as stripes for 12 minutes. Passages with wandering edges, seeded at the widest gap, fixed it. [M]
- The last 2 % of gaps made about 1,600 tiny strokes until long strokes were run through the clusters. [M]
- Late strokes that fill tiny gaps show almost nothing: about 57 s of a 30-minute film ran 3 to 7 s at a time with no visible change. Only a retime fixes that, so check it in the plan. [M]
- Under objects painted last, fill with quilted real brushwork from clear canvas and put the weave back in place. Mirrored detail went flat. [M]
- A quilt that skips blocks with no clear source leaves flat patches that read as bare paint for minutes. Scan the fill's local high-pass energy against the real canvas and fill the low spots with smaller blocks. [M]
- A touch cut out by mask has to keep its stage. Warm scraps filed with the water dashes painted the sun's reflection before the sun. Check each late colour against the stage that lays it. [M]
- Close masks widely, so that no scraps or ghost letters are left in the plate. A faint date showed through until it was closed by 40 px. [M]
- Straight cuts in last touches read as a wipe. Wobble their edges and give each bristle its own end. [M]
- Add relief only where the colour changes and blur the bump before taking its gradient, or outlines emboss. [M]
- Check strokes against the camera view. A smoothed follow-camera left 1.6 % of strokes out of frame at their midpoint, none in the last stages, which is what mattered. [M]
- If no technical study can be found, say so and label the order as a reading. For this work only the few hours, the quay contours, the last touches and the later date were documented; ground and layers were not. [M]
- Superpixel cells are fragments of strokes. Join them end to end along the brushwork, then take each spine from the joined mark's own long axis: cell paths ran 91 px, real marks 158 px. [N]
- Holes waiting for a late stage read as objects. Ridge gaps looked like snow for minutes, so close thin gaps straight after their neighbours; large round reserves read fine. [N]
- A 2D numpy paint canvas with only dirty tiles reshaded ran 0.2 s per frame at 8184 px, against 1.4 s in EEVEE, with no depth ties. Resample every frame through one path: switching between a mip and the full canvas made a sharpness pop mid-zoom. [N]

## Working with the owner

- Reference fidelity is the main quality bar. Double-check before presenting. [W] [S1]
- "Next project" with no brief means ask for the brief; do not resume set-aside work. [W]
- Do not resume or offer cancelled work. Delivered projects stay as delivered unless asked. [W]
