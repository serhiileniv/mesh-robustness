# mesh-robustness

Do C++ mesh libraries return valid output on real-world meshes, and do they agree on what "valid" means?

Status: **early research, nothing to run yet.**

## Scope
- **Libraries:** CGAL, libigl, OpenMesh, PMP, geometry-central
- **Data:** [Thingi10K](https://ten-thousand-models.appspot.com/) (10,000 real-world 3D-printing meshes)
- **Operations:** validity checks (manifold, orientation, self-intersection), simplification, remeshing

## Planned contributions
1. **Exact validator:** a library-independent ground truth for output validity.
2. **Failing-mesh minimizer:** shrinks a failing mesh to a minimal reproducer.
3. **Sanitizer:** a minimal cleanup pass that cuts failure rates, with a bound on geometric change.

## Layout
- `docs/`: plan, related work, specs
- `paper/`: LaTeX and bibliography
- `data/`: datasets (not committed; see `data/README.md`)

## License
MIT. Some planned dependencies are not MIT; for example, CGAL is GPL. Built binaries follow those dependencies' licenses.
