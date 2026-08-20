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
| `core` | lib | in progress - 18 source files, see below |

`core`'s modules (see `core/src/lib.fr` for the compile-order file list,
and each module's own header comment for the full story):

- **console_io** - formatting/printing (JIT-only for now)
- **math** - abs/min/max/clamp, libm trig/exp/log
- **string** - length, atoll/atof parsing, strstr/strcpy/strcat family,
  strcmp/strncmp/compare/equals (real `i32` returns - see the note below)
- **mem** - malloc/realloc/free, memcpy/memset, memcmp/compare
- **file_io** - fopen/fread/fwrite/fgets, fclose/feof/ferror (real
  `i32` returns)
- **time** - clock(), plus now_epoch() (real Unix time, via the `null`
  pointer constant)
- **random** - dependency-free Park-Miller LCG PRNG (pure i64
  arithmetic, no libc rand() dependency)
- **process** - exit/abort/getenv, panic()/assert() built on them
- **buffer** - indexed i64/pointer read-write into a raw heap buffer;
  the primitive Frust's own codegen still lacks (no pointer deref, no
  indexed-write) - closed via runtime helpers exported from Main.cpp,
  same pattern as console_io's format/print helpers
- **thread** - real OS threads (Win32 `CreateThread`) - only possible
  because of two compiler additions: a bare function name now decays to
  its address, and `null` is a real pointer constant
- **mutex** - real mutual exclusion (Win32 `CRITICAL_SECTION`) -
  verified against a genuine 4-thread/1000-increments-each shared
  counter, exactly 4000 every run
- **task** - a real "run on a thread, get a result, chain into the next
  step" monad built on thread+buffer; no closures yet, so chaining uses
  named worker functions with a fixed calling convention instead of
  lambdas - `is_ready()` for a non-blocking poll
- **promise** - `resolve()`, an already-done Task/Promise (no thread
  needed) - same shape as `task`'s output, so generic code chains
  uniformly regardless of where the value came from
- **combinators** - `all2`/`race2` over Task/Promise buffers - `race2`
  uses the real `WaitForMultipleObjects`, not a busy-poll loop
- **curry** - partial application (`curry2`/`apply1`, `curry3`/`apply2`)
  built on indirect-call compiler support - verified with the same bound
  partial reused across multiple different final arguments
- **procspawn** - real OS process spawning (`_spawnv`), both blocking
  and non-blocking, plus an env-var helper for passing data to a
  spawned process
- **process_task** - "run this `.frust` file in a separate OS process,
  get a result" via file+env IPC (a spawned process doesn't share
  memory, so `task`'s buffer tricks don't apply) - both a synchronous
  version and a real async one (`run_async`/`await_result_async`,
  verified for genuine concurrency, not just correctness)

**The "no i32" premise threaded through early revisions of this pod was
wrong** - `i32` works correctly in Frust (verified directly against
`strcmp`). Several functions wrongly skipped for that reason (`strcmp`,
`fclose`, a non-blocking task poll) are now wired up - see `string.fr`'s
header for the full correction.

`thread`/`mutex`/`task`/`procspawn`/`process_task` are Windows-only for
this pass - Frust has no platform-conditional source mechanism yet (no
`#cfg`, and frate.json's source list isn't platform-aware), and the
Linux/Pi port work is currently paused. A real gap, not a silent
omission.

## Cross-pod calls

Frust has no `use pod::thing` import syntax yet, but it does have C-style
`extern fn name(...) -> T;` forward declarations. A pod that depends on
another (declared in its own `frate.json`) calls into it by hand-writing
an `extern fn` matching the dependency's signature; `frate build` links
the resulting object files together. See FRATE_SPEC.md section 5.1/9 in
the parent repo for what's and isn't automated yet.
