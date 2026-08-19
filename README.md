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
| `core` | lib | in progress - console_io, math, string, mem, file_io, time, random, process |

`core`'s modules (see `core/src/lib.fr` for the compile-order file list,
and each module's own header comment for what's safe to wrap given
Frust's current lack of an `i32` type and real raw-pointer dereferencing):

- **console_io** - formatting/printing (JIT-only for now)
- **math** - abs/min/max/clamp, libm trig/exp/log
- **string** - length, atoll/atof parsing, strstr/strcpy/strcat family
- **mem** - malloc/realloc/free, memcpy/memset
- **file_io** - fopen/fread/fwrite/fgets (no fclose - see file)
- **time** - clock() only
- **random** - dependency-free Park-Miller LCG PRNG (sidesteps the i32
  gap entirely - pure i64 arithmetic, no libc rand())
- **process** - exit/abort/getenv, panic()/assert() built on them

## Cross-pod calls

Frust has no `use pod::thing` import syntax yet, but it does have C-style
`extern fn name(...) -> T;` forward declarations. A pod that depends on
another (declared in its own `frate.json`) calls into it by hand-writing
an `extern fn` matching the dependency's signature; `frate build` links
the resulting object files together. See FRATE_SPEC.md section 5.1/9 in
the parent repo for what's and isn't automated yet.
