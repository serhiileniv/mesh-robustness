# Related work: differential robustness testing of C++ mesh libraries

Phase 2 output, 2026-10-23. V = checked against the primary source; U = unverified, recheck before citing.

## Novelty verdicts

| Contribution | Verdict | Claim we can make |
|---|---|---|
| Failing-mesh minimizer | PARTLY NOVEL | First failure-preserving, validity-aware reducer for **3D triangle meshes**, evaluated across libraries. Earlier work: 2D polygons only, or reduction as an unevaluated side step. |
| Sanitizer | GAP HOLDS | No one measures **downstream success across libraries** before vs after cleanup, with a Hausdorff bound. No one reports the *minimal* fix each library needs. |
| Validator + disagreement | PARTLY | No cross-library, per-predicate disagreement matrix exists. Only hints: Loriot 2019, and libigl's doc/code mismatch. |
| "Reports success ≠ valid output" | PRECEDENTED, USABLE | Shown for single methods (Zhou 2016, Gabriel 2026, Shay 2022), never across libraries. |
| Empirical study of bugs in geometry libraries (SE venues) | NONE FOUND | Cite the absence carefully. |

## 1. Test-case reduction and fuzzing (minimizer)

- **Manifold `extras/minimize_testcase.cpp`** + FuzzTest harnesses: greedy 2D polygon minimizer that keeps the input valid; one library; no paper. Closest practice precedent. (V)
- **Spatter** (Deng et al., SIGMOD 2024): metamorphic testing of spatial databases; 34 bugs; 2D geometries reduced automatically and by hand, as a side step. (V)
- **Sergey** (ICFP 2016): QuickCheck-style growing and shrinking of 2D polygons. (V, abstract only)
- **GraphicsFuzz** (Donaldson et al., OOPSLA 2017): metamorphic shader testing with automatic reduction. (V)
- **C-Reduce** (Regehr et al., PLDI 2012): domain-specific reduction that keeps inputs valid; our method template. (V)
- **ddmin** (Zeller & Hildebrandt, TSE 2002): the base algorithm. (U)
- **Validity-preserving delta debugging** (arXiv 2402.04623). (U)
- **OSS-Fuzz:** libigl (`.msh` loader only), meshoptimizer (`simplifyfuzz`), assimp, draco. Crash-only, byte-level reduction. CGAL, OpenMesh, PMP and geometry-central have no fuzzer. (V)
- **What ddmin lacks on meshes (our technical content):**
  - keeping the mesh manifold/valid while removing parts;
  - failures that are non-monotone (they can disappear and reappear as the mesh shrinks);
  - simplifying coordinates (snapping, rounding).

## 2. Repair and cleanup (sanitizer)

- **Surveys:** Attene, Campen & Kobbelt, CSUR 2013 (repair framed as fitness for the application, but measures nothing); Ju 2009. (V)
- **Local repair:** MeshFix (Attene 2010; scans, no convergence guarantee). (V)
- **Volumetric repair:** PolyMender (Ju 2004); ManifoldPlus (2020; not tested on Thingi10K). (V)
- **Bounded deviation (prior art for our Hausdorff bound):**
  - Bischoff, Pavic & Kobbelt 2005 (tolerance);
  - Alpha Wrap (Portaneri et al. 2022, offset; in CGAL; Thingi10K);
  - TetWild (envelope). (V)
- **Self-intersection removal:**
  - Valque & Lazard, CGF 2025: all 4,524 self-intersecting Thingi10K models fixed.
  - CGAL 6.1 autorefine + snap: 9,997/9,997 Thingi10K models end intersection-free, **no Hausdorff reported**. (V)
- **Repair + Hausdorff on Thingi10K:** Visual-Guided Mesh Repair (arXiv 2309.00134). (V, abstract)
- **"Fix the algorithm, not the mesh":** Sharp & Crane 2020 nonmanifold Laplacian, 100% of Thingi10K. (V)
- **Library cleanup:** CGAL PMP repair (a separate package from 6.2), PyMeshLab, trimesh `process(validate=True)`, Open3D. None states a deviation bound. (V)
- **Repair can introduce errors:** GlitchFinder (arXiv 2509.00699) shows MeshLab and Meshmixer repairs doing this. (V)

**Required baselines in the same table:** MeshFix, CGAL repair + autorefine/snap, PyMeshLab clean, trimesh validate, Alpha Wrap (as a reference for maximum change).

## 3. Validity definitions and exact predicates (validator)

Definitions differ between libraries:
- **"Closed":** Thingi10K means ≥2 faces per edge; trimesh means exactly 2; CGAL means no border, on an already-valid halfedge mesh.
- **Vertex-manifold:**
  - libigl and PyMeshLab skip vertices on non-manifold edges;
  - libigl flags unreferenced vertices as non-manifold;
  - CGAL counts umbrellas; PMP counts gaps.
- **Edge-manifold:** libigl's doc says faces must be "oppositely oriented"; the code only checks a count of ≤2.
- **Degenerate faces:**
  - libigl `is_self_intersecting` treats a zero-area face as a self-intersection;
  - CGAL reports it as the pair (f,f);
  - admesh and PyMeshLab use floating-point or tolerance checks.
- **Self-intersection:** CGAL ignores shared features only when they are shared by index, so the STL vertex-merge policy changes the answer. Manifold assumes ε-valid input and has no check.

**Design consequence:** CGAL Surface_mesh, OpenMesh and PMP **cannot store** non-manifold edges (`add_face` fails, or throws `TopologyException`). The answer to "is it manifold?" comes from the importer. The harness must fix one loading and vertex-merge policy and report import-time rejection as its own category.

Exact predicates:
- Shewchuk 1997;
- CGAL Epick/Epeck;
- Attene indirect predicates 2020;
- Lévy PCK 2016 and arXiv 2405.12949;
- Cherchi et al. 2020/2022;
- Garau & Cherchi 2025 (arXiv 2507.08478, header-only triangle-triangle checks for mixed number types; a candidate validator core).
- Kettner et al. 2004/2008 for motivation.

Output-quality metric: Metro Hausdorff (Cignoni et al. 1998).

Disagreement precedents:
- Loriot 2019 blog: on Thingi10K, libigl accepts 8,346 inputs as valid and CGAL 6,091; 142 union outputs differ in area. (V)
- OpenSCAD #5493 (anecdote). (V)

## 4. Robustness numbers on Thingi10K (for the motivation and results sections)

| Paper | Subset | Numbers |
|---|---|---|
| TetWild 2018 | full | CGAL 57.2–79.0%, TetGen 49.5%, TetWild 99.9% (V) |
| fTetWild 2020 | full | CGAL 79.00%, TetGen 49.50%, fTetWild 99.97% (V) |
| Zhou 2016 arrangements | self-union | QuickCSG produces output on 99.9% of meshes, but only 62% of outputs are intersection-free; CGAL 73.4% / 51%. (V slides; check the paper's figure) |
| Gillespie 2021 integer coordinates | 7,696 manifold | signposts reach 69.1% on Delaunay refinement; integer coordinates 100% (V) |
| Sharp & Crane 2020 | full | 100%, with mollification (V) |
| Trettner 2021 | about 3,400 sanitized | other methods "required some sanitization" (V) |
| FA-QEM 2026 | full, to 1% | 100%; no baseline success rates (V) |
| Gabriel 2026 | 4,963 | 99.6% "success", but only 27.5% meet angle bounds (V) |
| Shay 2022 (artist UVs) | — | methods "guaranteed flip-free" flip on about 4% of meshes (V) |

## 5. Precedents for "just a benchmark"

- **Thingi10K** (2016, SGP Dataset Award 2017): "Oversimplified testing provides a false sense of robustness."
- **Schneider et al., TOG 2022:** a large-scale comparison with no new method.
- **Bonneel et al., TOG 2020:** code replicability study.
- **Berger et al., TOG 2013:** surface reconstruction benchmark.
- **Cignoni et al., C&G 1998:** comparison of simplification codes.
- **Hex Me If You Can**, SGP 2022.
- **Software engineering:**
  - Vanover et al., ISSTA 2020: differential testing of numerical libraries, 125 bugs, the closest precedent for our method.
  - Di Franco et al., ASE 2017: numerical bug taxonomy.

## 6. Venues and timeline

| Venue | Deadline | Fit |
|---|---|---|
| **JCGT** | rolling | Best fit: "practicality and reproducibility over novelty"; open source (MIT/BSD) expected; no page limit. (V) |
| EG 2027 short papers | **22 Jan 2027** | 4 pages, anonymous review, in-person talk in Lucca, 10–14 May. Tight: about 13 weeks away. (V) |
| SGP 2027 | about April 2027 (U) | CGF proceedings; Graphics Replicability Stamp available. |
| ISSTA / FSE (SE venues) | — | Alternative framing as differential testing with bug counts. |

**arXiv endorsement** (policy of 21 Jan 2026, V):
- Automatic endorsement needs an institutional email **and** a previously accepted arXiv paper in the same domain.
- A solo author needs a personal endorser. Ask a libigl or geometry-central author early.

## Open checks before committing

- Skim SGP and IMR 2024–2026 proceedings by hand for any mesh reducer (they are not fully indexed).
- Read Valque & Lazard 2025 in full (scope: manifoldness).
- Check the Zhou 2016 per-check percentages against the paper's figure.
- Confirm the Regehr 2012 author list and the Zeller 2002 fields.
- Confirm `remove_degenerate_faces` and `duplicate_non_manifold_vertices` in the CGAL 6.1/6.2 docs.
