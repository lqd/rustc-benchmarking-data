Before / After the cdylib being opt-in:
- on master and current nightly where the cdylib is always built and prevents cargo's pipelining:
  cargo-timing-hyper-master-opt-j8-cargo-nightly.html
- with cargo `6e5a6ffb14fc47051b0a23410c681ad6e4af045f` and PR https://github.com/hyperium/hyper/pull/2770,
  regular build (not opting into the FFI support) to show the effects of the pipelining, and
  the future default behaviour: cargo-timing-hyper-PR-opt-j8-cargo-6e5a6ffb14fc47051b0a23410c681ad6e4af045f.html