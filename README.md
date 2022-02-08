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

The current list of these ignored crates is in the `summaries` directory ([here](./summaries/failures-summary.txt)). Some of these require manual changes so that they can be profiled and this will be done in the future.

The 800 or so remaining crates were then benchmarked in different rounds, whose raw data is present in the `results` subfolders:
1. a cachegrind annotated profile of check builds
2. the result of `cargo llvm-lines` on the leaf crates
3. a DHAT profile
4. an idea of each crate's size: the rust LOC counts, as reported by `tokei -t=Rust` (not including markdown, i.e. doctests)
5. a cachegrind annotated profile of debug builds
6. the result of `cargo llvm-lines` on release builds (we expected it to provide information about the crate graph but it appears not, but the data is still different in --release)
7. check build schedules, as generated by `cargo -Ztimings`, under `-j1` (to have an idea of a single-thread schedule, and easily gather clean build data of every single dependency in case we'd like to experiment with different cargo scheduling algorithms)
8. debug build schedules, as generated by `cargo -Ztimings`, under `-j1`
9. release build schedules, as generated by `cargo -Ztimings`, under `-j1`
10. a cachegrind annotated profile of release builds
11. check build schedules, as generated by `cargo -Ztimings`, under `-j8`

Each cachegrind round is summarized: the annotated files are gathered together, and each of the top functions in the profile are collated, and:
1) filtered to focus on rustc functions -- removing the memory allocation/deallocation, elf dynamic loading, LLVM symbols, and rustc metadata encoding (the summary file are suffixed with `filtered_` and `unfiltered_`, and preceding the suffix in the following point) 
2) sorted according to their relative or absolute retired instructions (the summaries ending with `_absolute` and `_relative`)

(The simple tool that does that will also be added here later)

The `tokei` line counts are also summarized, and the crates sorted by that number. (The shorter crates are basically empty, and are either just displaying a deprecation notice, or façades to other crates)

