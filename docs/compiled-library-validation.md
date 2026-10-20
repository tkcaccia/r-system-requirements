# Validating compiled dependencies

Updated: 2026-10-20. Scope: source R packages with optional installed libraries
and a bundled CPU source fallback. This page is maintainer guidance, not a
claim that CRAN has provisioned any new dependency.

## Catalog rules are not ABI tests

The catalog maps declared requirements to distribution packages. It cannot
establish that headers, the C++ standard library, OpenMP, Fortran and numerical
libraries form a compatible toolchain. Do not replace a configure link/load
test with a successful package-manager query.

For example, `complete LP64 BLAS and LAPACK` matches
[the numerical-library rule](../rules/lp64-blas-lapack.json). It resolves to
`libblas-dev liblapack-dev` on Debian/Ubuntu and `blas-devel lapack-devel` on
Fedora. These names do not require replacing R's own BLAS implementation.
OpenBLAS is one possible implementation, not a mandatory choice for this rule.
LP64 means 32-bit BLAS/LAPACK integers; do not substitute ILP64 blindly.

A package bundling FAISS CPU source need not require an installed FAISS package.
Keep optional CUDA/cuVS requirements separate from the CPU installation path.
Installing a toolkit must not silently install or downgrade the host driver.

## Configure checks

1. Use the active R toolchain, including `CXX20`, site/user Makevars and linking
   flags. Never replace system R, BLAS or compiler configuration during install.
2. Compile and load a small OpenMP probe. R's OpenMP flag may be empty under
   Clang even when `-fopenmp` and a compatible runtime work. Test that flag;
   do not assume GCC defaults apply to every Linux compiler.
3. Use the same successful compile/link flags for the dependency and package.
4. Probe all required single- and double-precision BLAS/LAPACK symbols.
   R's private BLAS may export only the subset needed by R itself.
5. Load the resulting shared object with immediate symbol resolution. A shared
   link that permits unresolved symbols does not establish runtime compatibility.
6. Print the final failed probe's diagnostics to the installation log, and retain
   `config.log`. An error referring only to a subsequently deleted temporary
   file cannot be diagnosed from the public check output.

## Submission checks

Test one immutable source archive on native Windows and macOS plus Linux
GCC/GFortran and R-devel Clang/Flang profiles. Include a profile with no
preinstalled optional library. Record the exact R revision, compiler versions,
R configuration, user Makevars, source checksum and container digest.
Distribution names alone do not identify the compiler or R configuration.
Read the compiler identity and actual commands in the check/install logs rather
than inferring them from a service's flavor label.
Include both libstdc++ and libc++ where practical. Standard-library headers
need not supply the same transitive includes; use explicit headers for symbols
such as `errno` rather than relying on another header to introduce them.

Read `00install.out` as well as `00check.log`: compiler warnings can occur even
when the final R check says OK. Report upstream diagnostics separately from
package diagnostics; any accepted diagnostic needs a narrowly scoped reason.
Do not blanket-disable warnings or manufacture a clean check summary.
Check declared R dependency versions as well as availability. A binary
repository may lag behind a newly released source dependency; record which
version and build type the isolated test library actually uses.

Run evaluated vignettes, examples, tests, compiled-code checks and PDF/HTML
manual checks. Before submission, run network-enabled incoming checks too.
An initial-submission NOTE is expected; spelling, URLs and other NOTEs need
individual review. Private portability checks are not a CRAN acceptance result.

See the [R-hub Debian Clang recipe](https://github.com/r-hub/containers/tree/main/containers/debian-clang)
for an R-devel Clang/Flang environment and the
[R extensions manual](https://cran.r-project.org/doc/manuals/r-devel/R-exts.html#OpenMP-support)
for R's OpenMP configuration guidance.
