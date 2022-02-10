### rustc-benchmarking-data

This repository contains raw data, and summaries, from rustc benchmarks on many crates from crates.io.

Crate selection: while we want an healthy mix of binaries and libraries, at the time of writing, the crates benchmarked here were selected by popularity, and then filtered: the 1000 most popular crates by download count were gathered.

A lot of those lacked `Cargo.lock` files, a required file when benchmarking with the rustc-perf collector, so they were regenerated with `cargo generate-lockfile`.

They were then filtered: 
- failures from errors caused while executing a build script
- crates that were platform specific, like OS-specific bindings
- crates with manifest errors
- crates with many bin and lib targets, requiring manual intervention to hook the perf collector, but at least some of these should be profilable
- crates which failed to build because of dependencies, a few of those were relative dependencies and possibly dev-dependencies even (which shouldn't matter normally but I haven't investigated this category very much)
- crates specifying a toolchain version. I believe it shouldn't matter in regular dependency usage with cargo, but rustup does respect it when we try to profile locally. So some of these should be fixable simply by removing the toolchain override.
- crates with a build script, however trivial (some just check a MSRV, or that some native dependencies are installed on the OS etc) which I wholesale removed to try to clean up the profiling data, but are probably fine to profile after locating the source of llvm symbols (let's imagine: proc macros)
- crates that failed to build for some other reasone (a couple crates may be better placed into the other categories, but they didn't look like easily fixable so I've left them here)

The current list of these ignored crates is in the `summaries` directory ([here](./summaries/failures-summary.txt)). Some of these require manual changes so that they can be profiled with the collector, and this will be done in the future.

The 800 or so remaining crates were then benchmarked in different rounds, whose raw data is present in the `results` subfolders:
1. a cachegrind annotated profile of check builds
2. the result of `cargo llvm-lines` on the leaf crates
3. a DHAT profile
4. an idea of each crate's size: the rust LOC counts, as reported by `tokei -t=Rust` (not including markdown, i.e. doctests)
5. a cachegrind annotated profile of debug builds
6. the result of `cargo llvm-lines` on release builds (we expected it to provide information about the crate graph but it appears not, but the data is still different in --release)
7. check build schedules, as generated by `cargo -Ztimings`, under `-j1`
8. debug build schedules, as generated by `cargo -Ztimings`, under `-j1`
9. release build schedules, as generated by `cargo -Ztimings`, under `-j1`
10. a cachegrind annotated profile of release builds
11. check build schedules, as generated by `cargo -Ztimings`, under `-j8`
12. debug build schedules, as generated by `cargo -Ztimings`, under `-j8`
13. release build schedules, as generated by `cargo -Ztimings`, under `-j8`
14. rustc `-Zself-profile` data for check builds (default parallelism)
15. rustc `-Zself-profile` data for debug builds (default parallelism)
16. rustc `-Zself-profile` data for release builds (default parallelism)
17. rustc `-Ztime-passes` data for check builds (default parallelism)
18. rustc `-Ztime-passes` data for debug builds (default parallelism)
19. rustc `-Ztime-passes` data for release builds (default parallelism)

(The thought behind the `-j1` builds in rounds 7-9 is to have an idea of a single-thread schedule, and easily gather clean build data of every single dependency in case we'd like to experiment with different cargo scheduling algorithms. Comparing those with their `-j8` counterparts shows how massive of an improvement codegen parallelization was.)

Note: the self-profiler data was gathered via the perf collector. It normally collects all query keys, but that skews the profiles. This was disabled for these rounds 14-16. The data was then processed with the `summarize` and `flamegraph` tools from `measureme`.

Each cachegrind round is summarized (the simple tool that does that will also be added here later), the top functions are collected from each profile, and:
1) filtered to focus on rustc functions -- removing the memory allocation/deallocation, elf dynamic loading, LLVM symbols, and rustc metadata encoding (the summary file are suffixed with `filtered_` and `unfiltered_`, and preceding the suffix described next) 
2) sorted according to their relative or absolute "retired instructions" stat (here, the summaries are ending with `_absolute` and `_relative`)

The `tokei` line counts are also summarized, and the crates sorted by that number. (The shorter crates are basically empty, and are either just displaying a deprecation notice, or are façades to other crates)

The data was gathered using a combination of local rustc builds and nightly, where it mattered, on an EPYC 7401P (24C/48T):
- cargo: `cargo 1.60.0-nightly (1c03475 2022-01-25)`
- local rustc, for all the perf collector data (cachegrind, dhat, self-profiler, time-passes) purposes: lines debuginfo, incremental turned off, jemalloc turned off, stage 2, commit `e0e70c0c2c4fc8d150a56c181994e3a3b3e9999a`
- nightly rustc for cargo timings: `rustc 1.60.0-nightly (21b4a9cfd 2022-01-27)`
- local valgrind, to have some demangling fixes: commit `624fcaa4af8c923b07f6953fef3c5424eefb2ec1`

(Some of the data can be browsed on GH, either via the [summaries](https://lqd.github.io/rustc-benchmarking-data/summaries/), or [the raw data](https://lqd.github.io/rustc-benchmarking-data/results/) e.g. to see some cargo timings)
