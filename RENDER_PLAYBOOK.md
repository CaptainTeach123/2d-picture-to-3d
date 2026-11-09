# Render playbook

Lessons from real render projects, written for whoever runs the next render (mostly Claude). Each line comes from
something that went wrong or was measured on a project, not from general advice.

Tags: **[W]** Windsor Hotel, **[S1]** Stahl House, **[S2]** Stahl House v2, **[C]** Cliff House, **[N]** The Starry Night, **[M]** Impression, Sunrise, **[T]** Candy cane trophy (3D print), **[B]** Basketball score display (3D print), **[K]** Kansas Jayhawks score display (3D print).

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
   Read single strokes landing, frame by frame at final size, with the crop following the camera. Stills and frame differences passed a film the owner saw dissolving at first sight. [M]
   When a mark looks wrong, rebuild the canvas at that moment and put it beside its source layers. Curly marks that looked like cracks were in neither layer, so they were the engine's; loops that looked just as wrong were real paint in the scan. [M]
   After a partial re-render, diff against the previous film at moments the change should not have reached, and explain every structured difference. One frame, 60 s before the first intended change, exposed a stale cache in every earlier render. [M]
   Before calling a defect bad, put it in the output's own units and look at it. A count of wetness snaps read as square pops, and the first test of their size overflowed; real shading gave 1 to 2 levels, 5 at most across a tile. [M]
   Read every frame step of the final video at least once, not samples. Four renders passed per-second checks while hiding camera judder, dead stops at the canvas edge and encoder flicker, all of which one pass over 54,000 frames at 512 px showed in minutes. [M]
   In those frame steps, a dead stop is a fall from a moving level (mean step over 1) to under a quarter within two frames. The film before the camera fix had 537, and the eased camera none. [M]
   Take check times from the film's own timeline. A final-frame grab fixed at 1797 s failed once a retime made the film 1793 s long. [M]
   Render a short lossless clip of the same stretch before blaming or clearing the encoder. A quiet pattern on the keyframe period looked like strokes being withheld; the lossless clip disproved that, and showed the render clean where the encode flickered. [M]
   Map what really changes across a quiet stretch: first frame against last, on a lossless clip, coloured over the picture. Twelve seconds of it found ghost letter rims that every still from the same minutes had passed. [M]
7. Log every conflict with the reference as a numbered discrepancy with the decision made. [C]
8. Add what you learned to this file, as one line with a project tag.

## References and fidelity

- Measure the reference; do not assume. Guessed signs, fire escapes and street-light counts each cost a correction round. Iconic features get the most verification time. [W]
- Prefer a real photograph's solved camera over a painted image for geometry. [W]
- Date reference photos of things that get redone. Photos of the same basketball floor from a few years apart had different lettering ("KANSAS JAYHAWKS" against "ALLEN FIELDHOUSE"); the venue's own recent photo and a dated news item about the refinish settled which to model. [K]
- Size a floor logo against a known line in the same row of the photo, at full resolution. A low-resolution first look put the centre-court logo at 22 ft; against the three-point apex 19.6 ft from centre it measured 28 ft, which also made its outlines a quarter thicker. [K]
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

## Printable models

- Check the exported STL itself with a separate script: match vertices exactly and count edges not shared by exactly 2 faces, shells and signed volume. Blender called a mesh manifold while its STL had 135 edges shared by 4 faces. Rounding coordinates to 1e-5 mm merges distinct float32 vertices and fakes errors. [T]
- Triangulate every part before a boolean. Cut quads come back as re-merged non-planar n-gons, and triangulating them (the STL exporter does too) reused existing edges: 20 bad edges on the base. Triangle inputs gave 0. [T]
- Blender 5.2's MANIFOLD boolean solver unioned about 130 parts (235k triangles out) in about a second. EXACT took 12 s and left 228 bad edges where sweeps overlap themselves. [T]
- A part that rises out of a surface at a shallow angle (a leaf lifting off a tube) pinches in the boolean. Keep its underside buried so it meets the surface with steep walls; that also leaves no undercut to support. [T]
- Decor projected onto a tube from the front and clamped at the silhouette folds onto itself near the edge and breaks the boolean around it (382 bad edges from one leaf). Read in-plane distance as arc length around the tube instead. [T]
- Two pieces touching at one exact spot (a ball on a piping line, a berry on a leaf) flip between clean and pinched with any change in triangulation. Move one so they clearly cross, then re-check. [T]
- MANIFOLD can still leave 1 or 2 pinched edges a few microns long, and they move with any change to the model (its tolerance scales with the whole mesh). Scan a cosmetic parameter instead: stripe phases 0.66 to 0.73 all came out at 0 bad edges. Welding vertices within 5 microns made it worse (2 bad edges became 24). The SDF pass below ends this chase. [T]
- Smooth a print model through Blender 5.2's SDF grid nodes: Mesh to SDF Grid, SDF Grid Offset by -r then +r (a negative distance grows the solid), then Grid to Mesh. It fillets every concave join at radius r and rebuilds one watertight surface. At 0.1 mm voxels with adaptivity 0.15 the surface came out crinkled. At 0.07 mm with adaptivity 0 it was smooth: 20M triangles, decimated to 1.8M, about 160 s, 0 bad edges, bed face flat within 0.05 mm. Keep gaps that must stay open wider than 2r. [T]
- A curl that ends in a wire is cleanest as one sweep whose radius tapers into the wire. A separate wire glued onto the tip left a lump. [T]
- Helix lines on a thin, tightly bent tip circle its inner rim and read as hooks. Base the twist on max(radius, 7 mm) so they run along the curl, and thin the line's own radius to a point instead of sinking it (sinking left a groove). Ending them under a ring was rejected as messy. On the taper into the wire, also ease the helix angle to zero so the lines run straight along it and gather into the wire; any twist left there wrapped them round the tip like the lip of a shell. [T]
- Measure a curl on the reference along rays from its eye: inner and outer edge every 10-15 deg from a foreground mask. Ours stayed fat to 360 deg and then ended bluntly (outer edge 50 px at 330 deg against the reference's 33), which read as a horn mouth; following the measured taper over the last quarter turn gave a clean comma. Keep the turns 0.65 mm apart and assert it. [T]
- Lay ornaments on a helical band in the band's own surface coordinates (distance along its centre, distance across it), not in a front view unrolled round the tube. Projected leaves drifted onto the stripe lines where the cane turns away (-0.55 mm clearance); band coordinates kept every piece inside by construction. Assert each piece's clearance to the line beads. [T]
- A five-point star reaches 0.95 R across its band even with one arm along it, so on a 10.2 mm band a star clear of the lines is at most about 4 mm in radius, well under the reference's 6.8 mm. Kept inside, it was too small and the owner said the decorations "still need some work". Let the two side arms clearly cross the lines, as on the reference: tips at least 0.5 mm beyond the bead, with the other three arms 0.65 mm clear of it. On that band only R near 6.6 meets both. [T]
- Compose cane ornaments as the reference's corsages, not as separate pieces spaced along the band, which read as beads on a string. Put a big star at the centre, a gold bead nestled in the notch between its lower arms, berries overlapping the bead, and holly leaves fanning from under the berries. Where the band is too narrow for a fan, lift the middle leaf 0.7 mm so it lies over the side leaves. Lay it out flat first with pairwise checks: each pair either overlaps by 0.3 mm or more, or keeps 0.65 mm apart. [T]
- A small, tidy corsage that kept everything inside its band was still "needs more work". Measure the reference's pieces relative to its star, and let leaves lie over a line, lifted 0.35 mm so the line runs under them, with each point of the leaf clear of the line or well past it. Balance balls against leaves. Berries up to 1.95 mm in radius, plus several gold beads, with small leaves read as a pile of balls ("still off"). Three berries of 1.15 to 1.3 mm with 6 to 8 mm leaves overcorrected: "too sparse, leaves too small or flat, berries too small". A round holly rosette (three or four 8.5 by 4 mm leaves radiating from under three berries) was still off too: the owner ticked cluster shape, too flat, gold balls missing and leaf shape, and asked to match the reference closely. A long spray of mixed-size berries and gold balls running from each star's notch, plus a chain of gold balls along a line at the cane's edge and a lone small ball, was "extremely sloppy overall". Next tried: one sprig of three equal berries and two leaves per star, a gold trio above the upper star, one gold ball and one gem per star, all on the front. [T]
- Put a cluster where that part of the cane faces the viewer. On a 54-degree helix, 11 mm along the band from a front-facing star is about 55 degrees round the side, and the spray there showed only as leaf tips. At 8 mm it read from the front. [T]
- When the owner wants ornaments on the front only ("none on the back"), check that every point of every piece faces the viewer (cos of the angle to the view at least 0.7) and aim for 0.75-0.8: pieces at 0.73-0.75 sat on the silhouette in oblique close-ups. On a bent cane the strip that faces the viewer at a star can be barely wider than the star, so place the holly above and below the star rather than along its band. [T]
- Lay a cluster out in screen space when the cane bends. The strip facing the viewer at 0.8 or better was only about 13 mm wide round each star, so a holly sprig fitted as two leaves in a V of about 60 degrees hanging along the cane (berries by the star, leaves spilling over the next line), not as a sideways fan or three leaves. Convert each screen angle to a band angle at its own anchor: angles taken at the star were off by up to 40 degrees a few mm away. [T]
- When the model's stripes run at another angle than the reference's, a spray that follows the model's band is a short clump in the front view: ours ran 21 degrees off horizontal where the reference's ran 42. Angle the spray about 25 degrees off the band towards the viewer, so it starts on the stripe at the star and spills over the line; in the front view it then runs down the cane like the reference's. [T]
- Measure stripe clearance as the 3D distance to the bead centre curves (sampled every 0.08 mm), for the flat design data too. An angle-based clearance (radius times angle to the helix) misplaced a raised berry on the inside of a bend (0.91 mm flat, 0.41 mm, true 0.60). On the bulb it put a line up to 5 mm from where it was built, so a leaf tip it called 2.65 mm clear rested on the line. Only the reviewers' renders caught it. [T]
- Leaves tucked under balls lose their hidden part: set a leaf's reach from the ball centre to the visible length on the reference plus the ball's radius. Using the visible length as the reach gave leaves that showed as scraps between balls. [T]
- A holly leaf reads as holly only with sharp spines and a smooth blade. Draw the margin as concave arcs between spine points: quadratic curves pulled in toward the midrib and back toward the earlier spine. Let the spines stand 35-40 % beyond the oval, with bays about 30 % deep: at 20 % with 42 % bays the owner saw wavy edges, and 30 % with 50 % bays on a 3.2 mm blade read as ragged strips. Proportion matters most. The blade's real width is W times (1 + spike), not W: W 3.7 with spike 0.42 made leaves 5.3 mm across the spines on 6-9 mm of length, and three reviewers called them mittens. About 2.2 to 1 read as holly (W 3.1, 4.3 mm across the spines, 9-11.6 mm reach), with four spines a side at t 0.22, 0.44, 0.64 and 0.85 on an outline of sin(pi t^0.92). With the last spine at 0.64-0.72, the tip was a smooth spear that the smoothing blunted. Lifting that narrow tip 0.5 mm left a nub; 0.15 mm, with the midrib falling linearly to the tip, did not. Set the surface height from the true distance to the midrib, so the outline is cut out of one dome. Shallow sine-dip bays read as "thin wavy strips". Height scaled to the local width ran a ridge out to each spine (corrugated), and raised spine points made the edge look scalloped. [T]
- When the same vague "still needs work" comes back after two guesses, ask which parts, with options to tick. One answer ("holly leaves" and "berries and beads", not stars, gem or placement) ended three rounds of guessing. When ticked parts still come back "off", offer concrete differences from the reference as the options (cluster shape, relief depth, missing pieces, outline) and ask how closely to follow it: the owner ticked all four and chose "match it closely". [T]
- Grow each leaf of a spray from under its own berry. Two leaves fanned from one point ran within 0.3 mm of each other halfway out, which the smoothing webs. A spring solver could not satisfy loops of nestled balls either; place each ball against the one before it at a set angle. [T]
- Check ornaments against where the stripe lines really are along the band, not a straight band of the star's width. On the bulb the band widened from 13.3 mm at the star to 16.2 mm 12 mm away. The straight model put a leaf point on a line it thought it had cleared, 0.15 mm from the bead. Probe the line centres along the band and check against those. [T]
- A flat check on interpolated band data is not the build's exact clearance. Leaf points 0.30 to 0.34 mm inside a bead's edge in the flat check still came out as near-line warnings in the build; asking the flat check for 0.42 mm or more inside (squarely over the bead) or 0.6 mm or more clear gave none. [T]
- Passing the point check is not looking clear of a line: a holly tip 0.49 mm from a bead read as touching it in the front render. Keep the whole outline of a leaf that does not cross a line about 0.8 mm clear; in a 9 mm white band that left a two-leaf sprig room only for 3.2 mm blades about 50 degrees apart, both leaning along the band. [T]
- A leaf lying within a few degrees of a stripe line reads as resting on the rail, even with every point checked. Cross a line steeply, mid-blade, or keep the whole leaf clear of it. Measure it: the longest run of the leaf's outline within 0.5 mm of a bead was about 1 mm for clean crossings and 4.8 mm for the leaf the reviewers flagged; keep it under 1.5 mm. [T]
- Check the two stars' groups against each other in 3D where their front-facing areas meet. Each group passed its own flat check while a sprig of one sat 0.04 mm from the other's leaves on the white band between the stars; that band had room for one holly unit, not one per star. [T]
- The band frame runs on past the bulb into the base. A sprig laid below the lower star reached 58.5 mm high, under the collar top at 60.8 mm; check the lowest point of every piece against the collar and its fillet. [T]
- Build a moulded base from real profiles (ovolo, cove, torus, astragal) swept round rectangles whose corners are true offsets of each other (radius r + o; chamfer c + 0.586 o). Make breakfront blocks from the same profile round smaller rectangles and union them in. Stacked slabs with small chamfers read as plywood, and a block chamfered on all four corners left a dark slit against the slab: chamfer only its outer corner. [T]
- After the smoothing pass, drop shells of a few triangles: one of 12 faces appeared where two leaf stalks met under berries, and the STL then had 2 shells. [T]
- Placing decor on a tube via the nearest centreline sample leaves ridges every sample under smooth shading. Project onto the centreline segment and interpolate centre and radius. [T]
- A tube that narrows from its rounded start creases where the hemisphere cap meets it. Use the sphere tangent to the cone there: centre r*k behind the start, radius r*sqrt(1+k^2), with k = -dr/ds. [T]
- A profile swept round a path where a notch arc meets a straight side gets a mitre from the arc's first tiny segment, which spikes the inner edge. Blend each notch into the sides with convex arcs larger than the profile's half-width. [T]
- 128 segments on a 16 mm radius (2.8 deg steps) showed bands even on a print-scale model; 192 segments plus smooth-by-angle shading in the .blend read as smooth. [T]
- Choose stripe angles from test renders, not only the reference. Near-axial stripes and the measured 40-45 deg both bent into an S across the bulb's shoulder and were called "odd". Four fast renders (30, 42, 55, 65 deg) settled it: about 54 deg, constant up the whole cane, read as even spirals. [T]
- Don't scatter ornaments; lay them in groups along one stripe band. A group (sprig, star, sprig or gem, star, gem) is centred where a red band faces the front, with items spaced evenly along it. Tune the stripe angle a few degrees so a second group also lands on a red band. Lone gems and beads off any pattern read as "random". [T]
- Add a fast build mode that skips the slow smoothing pass. Layout checks then take 30 s instead of 6 minutes, and the full pass runs once at the end. [T]
- Join a bulb to a flat seat with a torus fillet about the vertical axis through the bulb's sphere, tangent to both. An offset ellipsoid "puddle" left a crease. [T]
- A groove narrower than about 4 voxels (a 0.2 mm leaf midrib at 0.07 mm) came out of the SDF pass and decimation as a sawtooth. A convex ridge where two domed halves meet came out clean, so give fine details convex forms. [T]
- Keep separate ornament pieces at least 0.65 mm apart, more than twice the smoothing radius, or the closing pass webs them together, and make the build assert it. A leaf whose stalk end lies in the open shows a blunt stub; start it under its berry. [T]
- For a clean S-scroll, integrate a curvature that grows smoothly into each end, tanh(s/w) / (rho_end + g*d + g3*d^3) with d the length left to the tip. Turn it to its slimmest and scale it to the panel, then size each stud from the last turn so the curl wraps it. [T]
- To reuse decorations from a downloaded, fused mesh, lift them off rather than clean the mesh: intersect it with a slab over the decorations' footprint, subtract the bare cane as a solid 0.8 mm above its fitted surface (4.8 mm over stripe lines), then place every point at the same arc length, angle from the viewer and height on your own cane, scaled by the ratio of the two radii. Stars, holly, berries, a gem and a lettered ribbon moved across where cleaning had failed. [T]
- A percentile estimate of the bare surface rides up under big decorations (a ribbon, a star), and the lift-off then cuts into them. Rebuild it from bare cells only and fill under the decorations by normalised convolution. A ribbon lip resting in a trough over a stripe line could not be told from the line by height; patch spots like that with small boxes cut straight from the source and moved with the piece's own transform. [T]
- Measure the source surface's stretch along the cane before mapping: where the download's bend put the front on its outside, its surface ran 1.3-1.5 times the centre line's length, and its berries mapped onto a straighter cane came out oval. Put the stretch back per group and shrink the group to fit (0.78 here). [T]
- Move a rigid piece (a ribbon with engraved lettering) with one similarity fitted to where the tube mapping puts its parts near the cane, and weld it with a few round posts where the band is at least 0.9 mm thick. Blending the two mappings creased the lettering, and a filled weld poked through where the band was paper-thin. An SDF closing of 0.1 mm blobbed 0.3 mm engraved strokes; 0.05 kept them. [T]
- A union made after the SDF pass can leave sub-micron pinched edges that no nudge removes (10 tries, 1-6 each). Deleting the triangles within 0.05 mm of each and filling the holes gave 0 bad edges at the same volume. [T]
- Scan each STL in its print orientation for faces pointing down more than 45 deg from vertical, away from the bed. The previews looked fine, but the scan found 1,634 mm2 of sloped ceiling under a stand's tilted slot holder and 72 mm2 on 0.3 mm snap catches at 49 deg. Fill the wedge under a tilted part down to the base. [B]
- Size an enclosure from the board maker's own 3D model. Split its STL into connected parts and take each part's bounding box (PCB, screen glass, buttons, USB-C, reset); online specs for the same board disagreed by 7 mm. [B]
- Make press-fit holes deeper than the whole peg, lead-in included, and assert it. Holes cut to the peg's straight length plus 1 mm left the 1 mm lead-in cone bottoming out, so every clip would have stopped 0.5 mm short of its seat and left the tablet loose behind its lip. Nothing in the STL checks or renders shows that. [K]
- Check the STL at every size the build script offers, not only the default. A frame was clean at its default scale and had 4 to 8 bad edges at the other four, all where a bevel hull met a lip cut. Insetting a rounded rectangle keeps its corner vertices at the same angles, so a hull begun from an inset of the cut's outline always had an edge in the plane of a cut's corner edge, whatever the inset. Building the hull's ring with another segment count fixed every scale: 100 against the cut's 96. 88 did not, because it shares the 45-degree vertex. The hull's front ring must not share corner centres with the cut either: a front ring drawn as an offset of the opening (radius r + run) put its corner tangent points in the plane of the cut's, and 4 to 8 bad edges came back until it got its own 2 mm radius. [K]
- Split a print bigger than the bed where no marking crosses the seam, and make the seam hold itself. A 15-inch court went into three tiles with seams 74 mm off centre, between the floor lettering (ending at 63.5 mm) and the three-point arcs (79.4 mm). Each seam slopes 35 degrees from vertical through the plate, so the tiles lap and the overhanging side stays under 45 degrees. Print order then matters: tiles go into the frame left to right. [K]
- In a section render of a joint, colour the two pieces differently and leave out everything behind the cut. The first seam section showed one solid colour: same-colour tiles hid the seam, and the frame filled the background. [K]
- Draw a screen mock-up for a render from the labels the firmware really draws. The first mock-up showed buttons that the firmware didn't have. [K]
- Look at recesses in a print render from an angle, lit from that side. In a straight-on back view with flat shading, cable bays vanished: their floors face the same way as the back, so they took the same shade, and their walls were edge-on. [K]
- Print lettering from real glyph outlines (fontTools, curves flattened), then simplify to 0.01 mm before extruding; points closer than that collapsed in float32 into degenerate faces. Measure printability with an opening (offset by -w/2, then +w/2) and look at what it removes: at 3.2-3.4 mm cap height, Georgia Bold had 33 mm2 thinner than 0.4 mm until thickened by 0.13 mm with 0.2-0.3 mm of extra letter spacing. [K]
- For a multi-colour flat print that single-nozzle printers can also make, give each colour its own height above the base (0.4 mm, then 0.8 mm) and ship one stacked file with filament changes at those heights alongside the separate colour files. [K]
- Build that stacked file by joining the same-height colours in 2D before extruding. Logo colours touching at single points gave a bad edge in the 3D union at one scale out of five; the 2D join was clean at all five. [K]
- Trace a flat-colour logo faithfully and leave thin lines to the slicer: Arachne prints anything over its minimum feature size (25 % of the nozzle) at its minimum wall width (about 0.34 mm on a 0.4 nozzle). Widening sub-nozzle lines in the model made neighbouring colours eat each other: white wing spikes vanished under blue, floor-coloured gaps filled in and serifs went blobby. Give each 0.05 mm cell the colour covering most of it in the full-size image, and drop only specks under 0.12 mm. [K]
- Five colours fit a four-slot printer when one colour lives only below a height and another only above it: give the wood floor and the yellow logo the same slot and swap the spool at a pause at the first layer above the floor. [K]
- Cut inlay pockets as the plate minus its floor, after dropping floor narrower than 0.05 mm with 8-segment arcs and simplifying to 0.01 mm. The plain union of the colours left micron slivers between separately simplified colours (1 bad edge). Closing the pockets bridged near-touching serifs into new point contacts (17). Thinning the floor with fine round arcs made micro-arcs that collapsed in float32 (110). The coarse version was clean at all five scales. [K]

## Materials and colour

- Stylized core: light through Shader-to-RGB into a smoothstep between lit and shade colours sampled from the reference with k-means. A higher Bias gives more shade. [S2] [C]
- Real scan detail goes on as a mean-1 multiplier, so texture adds detail without moving calibrated colours. [S2]
- Leave headroom in bright lit colours, because they clip under texture detail. [S2]
- Calibrate per material and check the result. Automatic ID-mask calibration is unreliable where objects overlap, and it turned yuccas navy. Thin objects drift; sample those by hand. [S1] [S2]
- Compare tone shares, not only means. Rocks were lit on 45 % of their area against 25 % painted. [C]
- Keep CC0 sources recorded next to the textures (Poly Haven scans worked well). [S2]
- Render a print in its filament colours before recommending them. A white frame swallowed the white banner inside it, and a blue one merged with the court's blue border; crimson read against both. [K]
- Choose filament colours from measured swatches, not vendor names. filamentcolors.xyz has an open API with about 2,300 colorimeter-measured swatches (Lab and translucency); ranked by CIEDE2000, matches for the brand colours came in at 1.7 to 3.6, while one major brand's closest red to the brand crimson was 8.6. [K]
- Two layers of colour over a contrasting base are tinted by it: where measured, the filaments needed several mm to block light. Inlay colours into the base in the multi-colour files (0.6 mm deep, so 1.0 mm of paint), and back a light colour that has to stay on the surface with white. [K]

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
- `python -I` also hides user-site packages, so numpy installed with `pip --user` went missing. `-P` alone still keeps the script's folder off `sys.path` when reading downloaded files. [B]
- Windows Python fails to open paths over 260 characters, and a session scratch folder was over that limit. Work in a folder with a shorter path. [T]
- Long Bash heredocs with nested quotes can fail. Write patch scripts to a file instead. [C]
- Piping a build through grep hides its exit code, so renders chained after a failed build used the old blend. Use `set -o pipefail`, delete the old output first, and check the new file exists before rendering. [T]
- Dump the surface frame of each ornament band once (point, normal and stripe clearance on a 0.25 mm grid of band coordinates, 8 s). From it, map reference pixels to band coordinates and back, sketch a layout over the reference, and check it against the real lines in under a second. A random search over leaf angles and sizes then finds the largest leaves that pass every check. [T]
- Lay out flat ornament as a 2D sketch first (PIL, about a second per variant). Then build and smooth only that area to check it (70 s) before the full build (7 minutes). [T]
- Give the build a test mode per area (named crop box plus which parts to union) and a layout-only mode that exits after placing ornaments and printing their clearances (2 s). Most tuning happened in those, with two full builds at the end. [T]
- Blender 5 meshes built from Python are smooth-shaded by default, and sharp relief (engraved letters) then renders with starbursts that look like damage. Call shade_flat() on check renders before chasing them. [T]
- Show a coloured reference against a painted preview, not grey clay: colour each vertex by the part it came from (berries red, gold balls and stars gold, leaves green, stripe bands red and white). In grey, berries, gold balls and leaves looked alike and the comparison misled. [T]
- Test leaf shapes on their own before rebuilding a layout: build several styles as slabs, smooth them like the print and render them side by side. It showed that holly spines drawn as cusps (two concave arcs meeting at a point) smooth away into scallops. Give each spine a firm point of about 40-70 degrees so it keeps its body. [T]
- Cleaning a downloaded, fused AI-generated mesh is slow and fragile. Re-projecting a fitted tube made the plain cane perfectly smooth, but telling stripe lines from decorations failed near flares and clusters: berries melted and patches of bare surface were held as decoration. Ask early whether to rebuild the part rather than clean it. [T]
- For a crowded layout, search locally from a layout that already passes, keeping only moves that still pass and scoring facing and leaf size. A random search with penalties never got past the discrete overlap checks. Then polish one leaf at a time: angle, reach and bend on a fine grid. [T]
- For an open design question, run several designers with different approaches through the same check-and-render tool, then judges with separate lenses (owner's history, reference, craft), then adversarial review of the merged winner. Two review rounds found what the designers missed: squat leaves, a stubby foreshortened leaf, and a leaf lying along a line. Tell reviewers which artifacts to ignore; the quick test build drew faint surface streaks that the full build did not have. [T]
- Workbench shadows drew a faint second star beside a relief star on a curved surface. It was not geometry: it vanished with shadows off. Keep shadows off for clay checks, or re-render without them before chasing an odd shape. [T]
- To put a picture on a flat screen in an orthographic z-buffer preview, render the screen face with a part id. Then map each pixel it wins back onto the plane and sample the picture, resized first to its size on the render. A plane's orthographic image is affine, so that is one 2 x 2 solve per view. The program's real 1080p output then rendered sharp in every view in about 2 s, where the earlier 200 x 120 grid of coloured cells could only approximate it. [K]
- To show a web page's real output on a modelled screen, capture it at the device's resolution with a headless browser, and launch it from PowerShell (Start-Process -Wait), not Git Bash. From Git Bash, Edge exited 0 and wrote no file, with `--headless` and `--headless=new` alike, after working once; the same flags from PowerShell wrote the 1340 x 800 capture every time. Give each run a fresh `--user-data-dir`, and check the file exists before rendering with it. [K]
- To capture a canvas page that draws images from other sites (team logos), load them with `crossOrigin = 'anonymous'` in a capture-only copy of the page, read the canvas with `toDataURL` in a browser that can reach the data, and post it to a local server. A canvas that has drawn one image without CORS stays locked for good, so swapping the images afterwards failed. Headless Edge started from a sandboxed shell could not reach ESPN at all, and the browser pane's zoom crops were neither 1:1 with page pixels nor bigger than 800 x 600, so stitching them was no good either. [K]
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
- Layers can land exactly with no fade: aim them at the plate blurred to brush size, then finish with strokes that lay the plate's own pixels. Landing exactly is not enough, though. A finishing layer that changes the paint's look reads as a dissolve, as smooth underpaint swapped for photographed paint did for 12 minutes. Underlayers need the finish's weave and grain from the start. [M]
- Simulate a stroke in brush space, with each bristle's load and colour and pickup of wet paint, and composite it per pixel by bristle pressure against thread height, not by soft alpha. Split tips and broken dry-brush edges follow from that. Smooth paint still read as digital until real grain was added, borrowed from elsewhere on the canvas and matched by stroke direction. Pickup had to come down to an eighth of its first value. [M]
- Inertia and turn limits stop strokes following a direction field into worm shapes, and holes must be masked out of the field first. [M]
- Keep the painting's own weave in place under all unfinished paint, added at shading and off for finished pixels, so it does not change when final paint lands. A fake overlay changed look at the finish. [M]
- A clean stroke front reads as a travelling mask. Give each tuft its own lead and lag, with stray hairs, and soften the touch-down. [M]
- First washes must flow into the canvas hollows. Dry-brushed over pale ground they left a hard dotted hatch. Short bristle runs also read as pale slashes: 520, 340 and 170 px runs fixed it. [M]
- A wet shine that vanishes in 0.45 s reads as a stroke melting flat. A 26 s decay read as drying paint. Measure each stage wet and dry on the canvas. Too much height embosses: a body of 21 looked like plastic and 10 looked like paint. [M]
- Give the brush a rhythm: 5 to 9 strokes per load with the charge falling, a pause to reload, staying in a colour family, working back and forth, with finishing done passage by passage. [M]
- A late touch adds its height to the paint already there. Writing absolute height cut a pit wherever a touch changed little, and the pit's rim drew a curly outline, while its damp patch marked the whole mask. Wet only where paint really changes. [M]
- Wide fills split marks into cells that land with the mask's edges showing. Isolate the marks and give everything else back to the scan: 164,759 px in the reflection, 345,009 px round the signature. [M]
- Under a thin mark, keep the scan to the mark's edge and carry the old fill's grain to the local tone with a membrane solved from that edge. Masked tone matching and row bridging both left a ghost of the signature's baseline. [M]
- Check what each mask really holds. Pieces of the pale band under the signature were filed as dashes and landed late with wavy edges. Rate each piece against the paint around it. [M]
- Binary dilation is a diamond, so a Euclidean ramp beside it left 23,566 px off the scan. Set the exact pixels instead of trusting the ramp. [M]
- A forward jump in a renderer must date each stroke by its own time, or the whole canvas reads wet at the resumed chunk. [M]
- A shading cache redrawn only in view goes stale. Paint the camera left kept its wetness, then snapped dry with its whole tile at the next touch, 21,245 times in 30 minutes. Keep a tile marked until it has been drawn in its final state, and redraw marked tiles just outside the view. [M]
- A resume path shades before the clock is set unless told otherwise, and a jump reshades everything, so neither reproduces a continuous run's cache. Do not assume a repaired chunk matches the run it joins. [M]
- Finishing on a grid showed as stripes for 12 minutes. Passages with wandering edges, seeded at the widest gap, fixed it. [M]
- The last 2 % of gaps made about 1,600 tiny strokes until long strokes were run through the clusters. [M]
- Late strokes that fill tiny gaps show almost nothing: about 57 s of a 30-minute film ran 3 to 7 s at a time with no visible change. Only a retime fixes that, so check it in the plan. [M]
- Under objects painted last, fill with quilted real brushwork from clear canvas and put the weave back in place. Mirrored detail went flat. [M]
- A quilt that skips blocks with no clear source leaves flat patches that read as bare paint for minutes. Scan the fill's local high-pass energy against the real canvas and fill the low spots with smaller blocks. [M]
- A touch cut out by mask has to keep its stage. Warm scraps filed with the water dashes painted the sun's reflection before the sun. Check each late colour against the stage that lays it. [M]
- Close masks widely, so that no scraps or ghost letters are left in the plate. A faint date showed through until it was closed by 40 px. [M]
- Straight cuts in last touches read as a wipe. Wobble their edges and give each bristle its own end. [M]
- Add relief only where the colour changes and blur the bump before taking its gradient, or outlines emboss. [M]
- A camera index from `int(t * FPS)` on a time built as `f / FPS` rounds down on 1.5 % of frames, in bursts of one frame in five: a repeated position, then a double step. Use the frame number, or round. [M]
- Clipping a smoothed follow-camera to the canvas makes a dead stop and a full-speed restart: 528 frames jumped over 2 px in speed, the worst 15.9 px. Clip, then smooth the position measured between the two edges, which cannot leave the canvas: largest change 0.27 px. [M]
- x264 B-frames flicker the whole picture at frame rate on a textured pan: 0.18 rms of mean level from a render at 0.008, in a four-frame rhythm. `bframes=0` took it to 0.02 for about a third more bitrate on moving stretches. Spacing keyframes 20 s apart keeps their re-grain pulse off still pictures. [M]
- Over a whole film, `bframes=0` with the master at crf 15 cut the alternating flicker frames from 10,105 to 34. It grew the master from 2.1 to 3.4 GB and the crf 20 share copy from 0.4 to 0.8 GB, so check disk space and upload limits first. [M]
- Masks that never overlap can still show a later mark. The pale band round the signature only touched the letters, and laid their rims five minutes early as ghost outlines. Give pixels within the fill's reach to the letter they border, and let a touch left with nothing to show give up its time. [M]
- Check strokes against the camera view. A smoothed follow-camera left 1.6 % of strokes out of frame at their midpoint, none in the last stages, which is what mattered. [M]
- If no technical study can be found, say so and label the order as a reading. For this work only the few hours, the quay contours, the last touches and the later date were documented; ground and layers were not. [M]
- Superpixel cells are fragments of strokes. Join them end to end along the brushwork, then take each spine from the joined mark's own long axis: cell paths ran 91 px, real marks 158 px. [N]
- Holes waiting for a late stage read as objects. Ridge gaps looked like snow for minutes, so close thin gaps straight after their neighbours; large round reserves read fine. [N]
- Marking only the tiles whose pixels changed, and redrawing damp tiles in view every 6th frame, took a 5,773 px canvas from 0.34 s a frame to 0.12 to 0.27 s. Whole films took 2 h 15 min to 2 h 48 min (the slowest with B-frames off at crf 15), and a repair of 6 chunks took 63 minutes. [M]
- A 2D numpy paint canvas with only dirty tiles reshaded ran 0.2 s per frame at 8184 px, against 1.4 s in EEVEE, with no depth ties. Resample every frame through one path: switching between a mip and the full canvas made a sharpness pop mid-zoom. [N]

## Working with the owner

- For physical models the bar is clean, not busy. All of these were rejected as messy:
  - scattered balls;
  - leaves hanging off a pilaster's edge;
  - a flat-bar frame with balls on it;
  - stringy relief leaves;
  - a pilaster kept inside its face but still carrying the reference's eight same-size balls, two grooved ribbon scrolls and three leaves ("a complete mess").

  Simplify the reference's motif rather than copying its parts. Check close-ups of every detail before presenting, because full views hid all of it. [T]
- Ask how closely to follow the reference area by area. The base's copied motif was "a complete mess" and had to be simplified, but on the cane the owner rejected five simplified sprays and chose "match it closely". [T]
- When the owner says a reference feature looks fragile, strengthen it modestly and keep it. Swapping the thin gold wire for a thick ball-ended volute was rejected; they wanted the wire about 10 % thicker. [T]
- Compare each detail with the reference at the same framing: an orthographic front render of the same pixel region, side by side with the reference crop. It showed what full views hid: spindly scrolls, leaves half the size, berries too small. The owner had called it "sloppy" and asked to "really look at" the reference. Size ornaments from the reference crops, make clusters compact, and keep stripe lines out from under leaves and berries. A star's arms may cross the lines; see Printable models. [T]
- For a printed piece that will be handled, durability beats matching the reference. The owner asked for the holly that bent back past the column edge to sit on the face instead. Make the build fail if any leaf vertex leaves its face (an assert caught one 0.03 mm over). Keep berries few; three to a cluster plus gold balls everywhere was "way too common". [T]
- On the cane, clean meant identical parts: all berries one size in identical nested trios, gold balls a size smaller and set apart, one gold ball per star, at least 2 mm between units. Matching gold balls in the notches either side of a star read as ears, and a ball floating between a star and its sprig as a stray berry. [T]

- Reference fidelity is the main quality bar. Double-check before presenting. [W] [S1]
- "Next project" with no brief means ask for the brief; do not resume set-aside work. [W]
- Do not resume or offer cancelled work. Delivered projects stay as delivered unless asked. [W]
- Show a short sample of the riskiest motion before a long render, and an early chunk during one. A third version passed every check and was rejected at first sight for how a stroke landed. [M]
