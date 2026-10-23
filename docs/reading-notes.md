# Reading notes: Thingi10K, TetWild, fTetWild

Sources: full PDFs, arXiv 1605.04797v2, arXiv 1908.03581v2, TetWild SIGGRAPH 2018 PDF (text extracted locally); Thingi10K metadata v1.5.0 from HuggingFace; `thingi10k` 1.5.0 wheel source. Numbers marked (CSV) are my own counts from `metadata/*.csv`, not from a paper.

---

## 1. Thingi10K (Zhou & Jacobson 2016, arXiv:1605.04797)

**What it is**
- 10,000 mesh files from every "featured" Thingiverse thing, 2009-09-16 to 2015-11-15, with per-model geometric and contextual stats.
- A short dataset paper. It argues that clean benchmarks give "a false sense of robustness".

**What we reuse**
- Dataset: 10,000 files, 2,011 things, 1,083 users, 4,892 tags (§2, §3). Formats: 9,956 .stl, 42 .obj, 1 .ply, 1 .off (§3). Each mesh file is a "model", and one thing can hold many models, so the results are correlated within a thing.
- Defect definitions (§3.1.2). Use these as the vocabulary for our validator:
  - Closed: every edge has ≥2 incident faces. Non-standard: a non-manifold edge still counts as closed.
  - Oriented: every non-boundary edge has zero signed incidence (#positive = #negative incident faces).
  - No isolated vertices; no duplicated faces (same vertex set).
  - Vertex-manifold: one-ring is a topological disc. Edge-manifold: each non-boundary edge has exactly 2 faces.
  - No degeneracy: no collinear vertices, "checked with exact predicates [Shewchuk 1997]".
  - No self-intersection: two faces meet only in ∅, a shared vertex or a shared edge. "Exact predicates are necessary to ensure correctness."
  - No coplanar intersections: a strictly weaker condition than no self-intersection.
  - PWN: the winding-number field is piecewise constant [Zhou et al. 2016].
  - Solid: PWN, self-intersection-free, and winding number in {0,1}.
  - Aspect ratio = circumradius / incircle diameter. Intrinsic Delaunay = all cotan weights ≥ 0.
  - The paper says "13 measures", and Fig. 7 shows 11 of them as % satisfied.
- Metadata columns in `geometry_data.csv`: separate counts for `num_geometrical_degenerated_faces` and `num_combinatorial_degenerated_faces`, plus `num_self_intersections`, `num_coplanar_intersecting_faces`, `num_duplicated_faces`, `num_boundary_edges`, and flags for vertex/edge manifold, oriented, PWN and solid. We can diff our exact validator against these per model.
- No evaluation protocol: the paper only describes the dataset.
- Table layout: none. Results are bar and percentile figures (Fig. 3, 7, 8). The Fig. 7 style (% of models satisfying each property, one bar per dataset) suits our input-characterisation figure.

**Numbers worth citing**
- 10,000 models / 2,011 things / 1,083 users / 4,892 tags (§3, §3.2).
- 29% of models have >1 component (§3.1.1). The README says 26%, and my CSV count is 25.5%. Cite one source and name it.
- An algorithm assuming self-intersection-free input "would automatically exclude 45% of inputs" (§3.1.2). My CSV count is 4,524/9,997 = 45.3%.
- Closed: 98% of OpenSCAD models, 91% of SketchUp, 85% of TinkerCAD (§3.2).
- Largest model 2.8M vertices (Fig. 4). Max genus 4,886 (Fig. 6).
- README stats (not in the paper): 50% non-solid, 45% self-intersecting, 31% coplanar self-intersections, 26% multi-component, 22% non-manifold, 16% degenerate faces, 14% non-PWN, 11% open, 10% non-oriented.
- My CSV counts (n=9,997): non-vertex-manifold 22.4%, non-edge-manifold 14.1%, geometric-degenerate 13.0%, combinatorial-degenerate 4.7%, duplicated faces 8.2%, boundary edges 11.4%, non-oriented 10.3%, non-PWN 13.8%, non-solid 48.9%.
- The exact Fig. 7 bar values are not in the text. UNVERIFIED unless read off the plot.

**How they framed robustness**
- "Oversimplified testing provides a false sense of robustness" (§1).
- "degeneracies, non-manifoldness and self-intersections are abundant, not special cases" (§1).
- "Structured modeling and coordinate quantization tends to break rather than fulfill general position assumptions" (§1). This argues directly for exact predicates.
- "clarify the requirements for geometry processing algorithms to be robust" (§5).

**Traps**
- Known corrupt files (README errata): 49911 truncated ASCII STL, 74463 empty, 286163 empty, 77942 corrupt binary STL, and 81313 an OBJ holding NURBS, not polygons.
- Also bad (CSV, not in the errata): 102610 (`schnuller.stl`) is missing from `geometry_data.csv`. It and 101633 and 996816 have NA for every flag in `input_summary.csv`. 74463, 286163 and 81313 have 0 faces in `geometry_data.csv` but are still flagged PWN=1. Filter on face count, not on the flags.
- The `thingi10k` package silently drops 4 IDs (49911, 74463, 286163, 77942). It keeps 81313 and 102610. Your denominator becomes 9,996 unless you load the raw files yourself. State the denominator explicitly.
- 996816 (TetWild Fig. 10 "bridge") has about 5.05M self-intersections (CSV) and blew TetWild's BSP up past 64GB (TetWild §3). Expect timeouts on it.
- Largest file: 376253, with 5.67M faces and 2.84M vertices (CSV).
- Raw files keep their original extension (`<id>.stl|obj|ply|off`). STL is a triangle soup with no shared vertices, so each loader's vertex welding changes the topology our validator sees. Pin one loader, or report weld tolerance per library.
- The meaning of "solid" drifts. The paper says PWN, self-intersection-free and winding in {0,1}. The package docstring says "Closed and oriented". The package actually just copies the CSV `solid` column.
- The meaning of "closed" drifts too. The paper says ≥2 faces per edge. The package computes `num_boundary_edges == 0`.
- Licences: per file, 10 licence types plus 4 `unknown_license` files (IDs 42224, 45945, 49860, 49861).
  - CC-BY-SA 3,680; CC-BY 2,945; CC-BY-NC 1,581; BY-NC-SA 975; BY-NC-ND 330; GPL 202; CC-PD 99; PD 88; CC-BY-ND 84; BSD 10; LGPL 2.
  - My CSV totals: NC 2,886, ND 414, SA or GPL 4,859.
  - **The minimizer produces derivatives.** Do not redistribute minimized meshes of the 414 ND files. SA and GPL derivatives must keep the same licence. Ship IDs plus a script, not meshes, unless the licence allows it. I have not done a legal review.

---

## 2. TetWild (Hu et al., SIGGRAPH 2018), skimmed

- The robustness table is **§4 Results, Table 2**, not §6. The paper ends at §5.

**What it is**
- Tet meshing of triangle soups. It uses an exact rational BSP construction plus an ε-envelope tolerance, then float rounding and AMIPS optimisation.
- It was the first paper to run on all of Thingi10K.

**What we reuse**
- Protocol (Table 2 note): each model is capped at **3 h and 32 GB**. Failure columns are *Out of memory (>32GB)*, *Time exceeded (>3h)* and *Algorithm limitation*. "Algorithm limitation" is never defined in the text (UNVERIFIED meaning, probably crash or no or invalid output). Average time is over successes only.
- Parameters: ℓ = b/20, ε = b/1000, where b is the bbox diagonal (§4 Parameters). Competitor parameters are "in the additional material".
- Hardware: NYU HPC cluster (acknowledgements). The CPU model is not given in the paper (UNVERIFIED). It is single-threaded (§5).
- Layout: Table 2 has one row per tool/config and these columns: success %, OOM %, timeout %, algorithm limitation %, avg time. Table 1 has one row per figure model with Thingi10K ID, #V in and out, angle min/avg, AMIPS max/avg and time. **This is a good template for our tables.**

**Numbers (Table 2)**
- Success rates: CGAL explicit with features 57.2%; CGAL explicit without features 79.0%; CGAL implicit 55.7%; TetGen 49.5%; DelPSC 37.1%; Quartet 87.2%; MMG3D 56.2%; TetWild 99.9%.
- TetWild timed out on 11 models and reaches 100% if allowed 27 h (Table 2 note).
- CGAL implicit: 32.6% timeouts. TetGen: 48.7% algorithm limitation.
- The text rounds Quartet to "88%" and DelPSC to "around 38%". **Cite the table, not the prose.**
- §3: 9,997 models gave valid output at stage 1 and 3 more needed stage 2. 2 of 10k models made the BSP exceed 64 GB.

**Framing**
- "unconditionally robust" (abstract).
- The user must "'fix' input surface meshes to cajole meshers to succeed due to unspoken pre-conditions, or output tetrahedral meshes must be repaired due to failure to meet basic post-conditions (such as manifoldness)" (§1). **This is our headline almost verbatim; cite it.**
- "robustness comes first (i.e., the algorithm should produce a valid and, to the extent possible, useful output for a maximally broad range of inputs)" (§1).
- Delaunay variants need predicates beyond the usual set, which "may account for a percentage of failures that we observe" (§2). This is hedged root-cause wording worth copying.

---

## 3. fTetWild (Hu et al. 2020, "Fast Tetrahedral Meshing in the Wild", arXiv:1908.03581, ACM TOG)

**What it is**
- TetWild rebuilt on floating point with incremental triangle insertion into a background tet mesh. It rejects any insertion that would invert or degenerate a tet, so the mesh is always valid.
- About 7× faster than TetWild with the same robustness. The trade-off is that some input faces may stay uninserted, which they never observed.

**What we reuse**
- Protocol (§4): Xeon E5-2690v4 at 2.6 GHz, ≤8 threads, 128 GB and 24 h per model. **But** Table 2 reuses TetWild's 3 h / 32 GB caps, and its first 3 rows are copied from TetWild Table 2. The limits are inconsistent within the paper. Pick one pair and state it.
- Parameters: ε = 10⁻³·d, ℓ = d/20. Stop when max AMIPS < 10 or after 80 iterations (§3.5). ε_zero = 1e-8 (§3.1).
- Output validity definition (§3): (1) every tet has positive volume, checked with exact predicates [Shewchuk; Lévy/Geogram]; (2) the tracked surface lies inside the ε-envelope. This is a validator spec stated as post-conditions.
- Envelope containment is tested by sampling, with the error "conservatively compensated" (§3.2). It is not exact; note this if we compare against envelope checks.
- Fair-timing practice (§4.2): they disabled their own optimisations to compare like with like ("unoptimized" 80.4 s vs TetWild 360 s). They also used a "reduced dataset" of 4,540 models where all 4 tools succeed, for timing only. **Copy this: time only on the intersection of successes.**
- Layout: Table 2 has the same columns as TetWild plus footnotes for timeouts. Fig. 1 and Fig. 14 are survival bars ("% of models needing > t", with t doubling). This is a compact runtime figure.
- Quality measures (§4.3): AMIPS, minimum dihedral angle, volume-to-edge, aspect ratio, radius-to-edge, each with its range and optimum given.

**Numbers**
- Abstract and §4.1: "successfully tetrahedralizes 100% of the 10 000 input meshes" under 128 GB / 24 h.
- Table 2 (3 h / 32 GB):
  - CGAL 79.00%; TetGen 49.50%; TetWild 99.89%; fTetWild 99.97%.
  - fTetWild: 0.02% OOM, 0.03% timeout. Avg time 11.7 / 32.3 / 360.0 / 49.8 s.
  - fTetWild timed out on 3 models and reaches 100% if allowed 11 h.
- Fig. 1 and §4.2, on 4,540 models: 98.7% done in under 2 min. Avg 18.5 s vs TetGen 22 s, CGAL 95 s, TetWild 107 s. More than 16 min: ours 4, TetGen 20, CGAL 122, TetWild 25.
- Most models need under 1 GB RAM (Fig. 13). The largest example needs about 17 GB (Fig. 21: 31M faces).
- Up to 17× faster on a hard model (Fig. 15: TetWild 17 h, ours 56 min).

**Framing**
- "guaranteed to produce a valid tetrahedral mesh with floating-point vertex coordinates, while TetWild produces a valid mesh with rational coordinates which is not guaranteed to be valid after floating-point conversion" (abstract). This draws the line between valid-in-theory and valid-as-emitted, which is exactly our invalid-but-reports-success angle.
- "no longer guarantees that all input triangles are present … but in practice, as confirmed by our tests on the Thingi10k dataset, the algorithm always succeeds" (abstract). This is honest phrasing for "no guarantee, but empirically always".
- "most of the state-of-the-art tetmeshers have low success rate on in-the-wild data" (§4.1).
- "Note that the average time … is computed over all the models for which each method succeeded, and it is thus not directly comparable" (Table 2 note). Add the same caveat ourselves.
- §5.1: keeping only the boundary of a valid tet mesh "might generate a non-manifold surface mesh". A valid volume output can still give an invalid surface. Supporting evidence for our thesis.

**Traps**
- The resource limits are inconsistent (24 h / 128 GB in the text vs 3 h / 32 GB in Table 2), and "100%" vs 99.97% depends on which limits apply.
- The "CGAL" row is CGAL explicit without features, copied from TetWild. The CGAL version is not stated (UNVERIFIED).
- Their success means "produced a tet mesh". It is not a check of output validity against an independent validator. That gap is ours to fill.

---

## 4. Getting Thingi10K today (checked 2026-10-23)

- Official site https://ten-thousand-models.appspot.com is still up (HTTP 200). Code is at github.com/Thingi10K/Thingi10K (Apache-2.0 for the code only; last push 2026-08-18).
- `pip install thingi10k`: v1.5.0 (2026-08-18), **requires Python ≥3.10**.
  - Dependencies: `datasets>=4.0`, `huggingface_hub`, `polars`, `numpy`, `lagrange-open>=6.29`. The optional `[clip]` extra adds semantic search.
  - `thingi10k.init(variant='npz'|'raw'|'tetwild', cache_dir=...)` downloads the data. `thingi10k.dataset(**filters)` returns a HF `datasets.Dataset`. `load_file(path)` returns (V, F), and raw files are parsed with lagrange.
- HuggingFace: `datasets/Thingi10K/Thingi10K`, not gated. The package pins revision tag `v1.5.0`.
  - `Thingi10K.tar.gz` (raw) is 9.62 GB. Uncompressed `raw_meshes/` is about 38.8 GB summed over 10,000 files.
  - `Thingi10K_npz.tar.gz` is 4.09 GB. `Thingi10K_tetwild_npz.tar.gz` is 1.99 GB. `Thingi10K_quadwild_npz.tar.gz` is 4.40 GB. `Thingi10K_renderings.tar.gz` is 0.82 GB.
  - `raw_meshes/<id>.<ext>` files can be fetched one at a time. `metadata/*.csv` is about 6.5 MB.
  - `datasets.load_dataset()` does not work directly since datasets 4.0; use the package.
- Mirrors: NYU Box and Google Drive (links in the README). I did not check them.
- **Per-model defect flags: yes, partially.**
  - Filters and fields: `closed`, `self_intersecting`, `vertex_manifold`, `edge_manifold`, `manifold` (both), `oriented`, `pwn`, `solid`, plus `num_components`, `num_boundary_edges`, `euler`, `genus`, `num_vertices` and `num_facets`.
  - **Not exposed:** degenerate, duplicated, coplanar-intersection and self-intersection counts. Those live only in `metadata/geometry_data.csv`. Read it directly with polars or csv.
  - The flags come from 2016 analysis code. We have not checked them against our exact validator, and that is worth a sentence in the paper whichever way it comes out.
