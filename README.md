# Frust Library

Foundational Frust pods - the start of a standard library, not add-ons.
Managed as a [Frate](https://github.com/wwestlake/lagdaemon-tech-research/blob/master/projects/05_frate/FRATE_SPEC.md)
workspace: this repo's root `frate.json` lists every pod as a workspace
member, so they all build together locally with `frate build` (from the
`frate` CLI in the parent `lagdaemon-tech-research` repo) with zero
network dependency.

This repo is a git submodule of
[`lagdaemon-tech-research`](https://github.com/wwestlake/lagdaemon-tech-research),
mounted at `projects/06_frust_library`.

## Pods

| Pod | Type | Status |
| :-- | :--- | :----- |
| `core` | lib | scaffolded, unwritten |

## Cross-pod calls

Frust has no `use pod::thing` import syntax yet, but it does have C-style
`extern fn name(...) -> T;` forward declarations. A pod that depends on
another (declared in its own `frate.json`) calls into it by hand-writing
an `extern fn` matching the dependency's signature; `frate build` links
the resulting object files together. See FRATE_SPEC.md section 5.1/9 in
the parent repo for what's and isn't automated yet.
