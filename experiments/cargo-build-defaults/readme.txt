Before and after results: changing cargo's build defaults.

`hyperfine` results of two different cargo builds, with one being "faster than" the other.
Improvements are the new defaults being faster than the current defaults. Regressions are the
current defaults being faster than the new defaults.

Long benchmarks were 3 warmups and 5 benchmark runs. Shorter benchmarks (less than 1s) were 15
warmups and 50 runs. 250 or so benchmarks have such a short duration, making the variance between
runs impactful in the results.

Each result in the `build-j8` and `check-j12` folders has the time range and standard deviation,
mean and error range. Summary files contain only the latter two. The magnitude of improvements is
bigger, and the confidence interval tighter. Regressions often have a small magnitude and a much
wider interval, often multiple times bigger than the change itself, making the categorization as a
regression often shallow. Variance similarly impacts improvements on short benchmarks, but at a
smaller ratio.

- `cargo check -j12`: 613 improvements, 164 regressions.
- `cargo build -j8`: 584 improvements, 193 regressions. 