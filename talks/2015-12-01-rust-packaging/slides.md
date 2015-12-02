# Packaging Rust

### For Debian, but also others too

Angus Lees &lt;<gus@inodes.org>&gt;

---

## The Rust Toolchain

- `rustc` and `rustdoc`  (from rust-lang.org)
- `cargo`
- Libraries (crates) - written in Rust
- The application you actually wanted - written in Rust

---

## Packaging 101
### A brief introduction

**Goal:** Create some metadata so the packaging system knows how to a) bundle up the software and b) make it easily installable.

Most distros[1] separate the *"build"* and the *"install"* steps so everyone can re-use the same generated artifacts.

&nbsp;

<small>[1] Notably not Gentoo or other "source-based" distros.</small>

---

## Packaging 101
### The Process

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="657px" height="253px" version="1.1" style="background-color: rgb(255, 255, 255);"><defs/><g transform="translate(0.5,0.5)"><rect x="165" y="61" width="100" height="40" rx="6" ry="6" fill="#ffffff" stroke="#000000" pointer-events="none"/><g transform="translate(163,75)"><switch><foreignObject pointer-events="all" width="103" height="16" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; width: 103px; white-space: normal; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;">Upstream source</div></div></foreignObject><text x="52" y="14" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica">[Not supported by viewer]</text></switch></g><rect x="315" y="61" width="100" height="40" fill="#ffffff" stroke="#000000" pointer-events="none"/><g transform="translate(315,68)"><switch><foreignObject pointer-events="all" width="99" height="30" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; width: 97px; white-space: normal; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;">Add packaging metadata</div></div></foreignObject><text x="50" y="21" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica">[Not supported by viewer]</text></switch></g><rect x="465" y="61" width="100" height="40" fill="#ffffff" stroke="#000000" pointer-events="none"/><g transform="translate(461,75)"><switch><foreignObject pointer-events="all" width="107" height="16" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; width: 107px; white-space: normal; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;">"Source" package</div></div></foreignObject><text x="54" y="14" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica">[Not supported by viewer]</text></switch></g><rect x="165" y="141" width="120" height="40" fill="#ffffff" stroke="#000000" pointer-events="none"/><g transform="translate(164,148)"><switch><foreignObject pointer-events="all" width="121" height="30" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; width: 121px; white-space: normal; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;">Build step<div>(dpkg-buildpackage)</div></div></div></foreignObject><text x="61" y="21" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica">[Not supported by viewer]</text></switch></g><rect x="315" y="141" width="100" height="40" fill="#ffffff" stroke="#000000" pointer-events="none"/><g transform="translate(315,148)"><switch><foreignObject pointer-events="all" width="99" height="30" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; width: 97px; white-space: normal; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;">"Binary" package(s)</div></div></foreignObject><text x="50" y="21" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica">[Not supported by viewer]</text></switch></g><rect x="465" y="141" width="100" height="40" fill="#ffffff" stroke="#000000" pointer-events="none"/><g transform="translate(477,148)"><switch><foreignObject pointer-events="all" width="75" height="30" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; width: 75px; white-space: normal; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;">Distribute<div>everywhere</div></div></div></foreignObject><text x="38" y="21" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica">[Not supported by viewer]</text></switch></g><rect x="165" y="211" width="100" height="40" fill="#ffffff" stroke="#000000" pointer-events="none"/><g transform="translate(169,218)"><switch><foreignObject pointer-events="all" width="91" height="30" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; width: 91px; white-space: normal; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;">Install step<div>(apt-get install)</div></div></div></foreignObject><text x="46" y="21" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica">[Not supported by viewer]</text></switch></g><rect x="315" y="211" width="100" height="40" rx="6" ry="6" fill="#ffffff" stroke="#000000" pointer-events="none"/><g transform="translate(315,218)"><switch><foreignObject pointer-events="all" width="99" height="30" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; width: 97px; white-space: normal; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;">Usable software. Profit.</div></div></foreignObject><text x="50" y="21" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica">[Not supported by viewer]</text></switch></g><path d="M 265 81 L 308.63 81" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 313.88 81 L 306.88 84.5 L 308.63 81 L 306.88 77.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 415 81 L 458.63 81" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 463.88 81 L 456.88 84.5 L 458.63 81 L 456.88 77.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 515 101 Q 515 111 365 111 Q 215 111 215 134.63" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 215 139.88 L 211.5 132.88 L 215 134.63 L 218.5 132.88 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 285 161 L 308.63 161" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 313.88 161 L 306.88 164.5 L 308.63 161 L 306.88 157.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 415 161 L 458.63 161" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 463.88 161 L 456.88 164.5 L 458.63 161 L 456.88 157.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 515 181 Q 515 191 365 191 Q 215 191 215 204.63" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 215 209.88 L 211.5 202.88 L 215 204.63 L 218.5 202.88 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 265 231 L 308.63 231" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 313.88 231 L 306.88 234.5 L 308.63 231 L 306.88 227.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><rect x="445" y="1" width="70" height="20" fill="none" stroke="none" pointer-events="none"/><g transform="translate(453,5)"><switch><foreignObject pointer-events="all" width="54" height="16" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; overflow: hidden; max-height: 16px; max-width: 66px; width: 54px; white-space: normal; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;"><font style="font-size: 14px"><i>Humans</i></font></div></div></foreignObject><text x="27" y="14" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica">[Not supported by viewer]</text></switch></g><rect x="605" y="71" width="50" height="20" fill="none" stroke="none" pointer-events="none"/><g transform="translate(607,75)"><switch><foreignObject pointer-events="all" width="46" height="16" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; overflow: hidden; max-height: 16px; max-width: 46px; width: 46px; white-space: normal; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;"><font style="font-size: 14px"><i>Robots</i></font></div></div></foreignObject><text x="23" y="14" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica">[Not supported by viewer]</text></switch></g><path d="M 463 21 Q 463 41 414 41 Q 365 41 365 54.63" fill="none" stroke="#000000" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="none"/><path d="M 365 59.88 L 361.5 52.88 L 365 54.63 L 368.5 52.88 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 630 91 Q 630 151 571.37 151" fill="none" stroke="#000000" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="none"/><path d="M 566.12 151 L 573.12 147.5 L 571.37 151 L 573.12 154.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 618 91 Q 618 121 461.5 121 Q 305 121 305 136 Q 305 151 291.37 151" fill="none" stroke="#000000" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="none"/><path d="M 286.12 151 L 293.12 147.5 L 291.37 151 L 293.12 154.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 31 51 C 7 51 1 71 20.2 75 C 1 83.8 22.6 103 38.2 95 C 49 111 85 111 97 95 C 121 95 121 79 106 71 C 121 55 97 39 76 47 C 61 35 37 35 31 51 Z" fill="#ffffff" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><g transform="translate(12,58)"><switch><foreignObject pointer-events="all" width="97" height="30" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; width: 97px; white-space: normal; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;">Other packages<div>(build-deps)</div></div></div></foreignObject><text x="49" y="21" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica">[Not supported by viewer]</text></switch></g><path d="M 97 95 Q 143 95 143 123 Q 143 151 158.63 151" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 163.88 151 L 156.88 154.5 L 158.63 151 L 156.88 147.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 31 161 C 7 161 1 181 20.2 185 C 1 193.8 22.6 213 38.2 205 C 49 221 85 221 97 205 C 121 205 121 189 106 181 C 121 165 97 149 76 157 C 61 145 37 145 31 161 Z" fill="#ffffff" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><g transform="translate(12,168)"><switch><foreignObject pointer-events="all" width="97" height="30" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; vertical-align: top; width: 97px; white-space: normal; text-align: center;"><div xmlns="http://www.w3.org/1999/xhtml" style="display:inline-block;text-align:inherit;text-decoration:inherit;">Other packages<div>(dependencies)</div></div></div></foreignObject><text x="49" y="21" fill="#000000" text-anchor="middle" font-size="12px" font-family="Helvetica">[Not supported by viewer]</text></switch></g><path d="M 116 197 Q 143 197 143 214 Q 143 231 158.63 231" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/><path d="M 163.88 231 L 156.88 234.5 L 158.63 231 L 156.88 227.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="none"/></g></svg>

---

## Packaging 101
### The rules

- License(s) must meet Debian Free Software Guidelines
- Must not use the network during build
- "Vendoring" sources is bad
  - hides them from security-team
  - duplication wastes resources
- Must be able to be rebuilt using packages in the archive
- Must be built on native architecture (no cross-compiling)
  - These last two are temporarily ignored during "bootstrapping" but result doesn't go into archive

---

## `rustc` and friends

- This is in pretty good shape right now (for amd64)
- Looks a lot like a regular upstream project
  - Source `.tar.gz` releases, signed
  - `rustc` requires `rustc` to build, but this isn't unique
- Already in Debian *unstable*: https://packages.debian.org/sid/rustc
- Debian packaging maintained by a small team through http://anonscm.debian.org/cgit/pkg-rust/rust.git/
- Todo: cross compiling, easier bootstrap

Note:
- Introduce circular build-dep

---

## Circular build-dep on `rustc`

- Currently handled by bundling the pre-built `rustc` stage0 blob with packaging metadata

- Works, but not great:
  - Large opaque binary makes people uneasy
  - Won't scale to many architectures (sheer size)

- Ideal future: Build `rustc` from itself
  - First architecture from pre-built blob
  - All other architectures cross-compiled
  - Future versions from existing rustc package
  - *Lots of blockers to address first*

---

## Differences vs `make install`

- Separate binary packages produced:
  - `rustc`
  - `rust-gdb`, `rust-lldb`
  - `rust-doc`
  - `libstd-rust-dev`
  - `libstd-rust-1bf6e69c`

- Split mostly to support (future) cross-compilation
  - `libstd-rust-dev` for target arch
  - `libstd-rust-xxx` can be co-installed for each arch

---

## Differences vs `make install`

- Run-time dylibs (`libstd-rust-xxx`) installed into regular `ld.so` path:<br>
  `/usr/lib/x86_64-linux-gnu/lib*.so`

- Compile-time dylibs/rlibs (`libstd-rust-dev`) installed into:<br> `/usr/lib/rustlib/x86_64-unknown-linux-gnu/lib/lib*.{so,rlib}`
 - dylibs (`*.so`) are symlinks back to run-time dylibs

---

## Patches applied

- `src/llvm/*` removed (not needed)
- `jquery` source added
- `rust-{gdb,lldb}` scripts rewritten to hardcode paths
- `configure`/`Makefile` patch to pass `CFLAGS`/`LDFLAGS` down to build commands
  - `rustc`/`rustdoc` executables linked with `-Wl,-z,relro`
- `rustc` patch to add `-Wl,-soname=`*filename* when linking
- Documentation post-processed to use local icon/logos

Note:
- LLVM stripped because a) we don't need it and b) one cmake file is CC-by-2.0 (not-DFSG-free)
 - CC-by-2.0 is not DFSG-free because anti-DRM clause disallows (eg) installing on a with DRM/TPM hardware; and requirement to remove an author's credit upon request deemed infeasible.

---

## `rustc` oustanding issues

- Are we allowed to call it `rustc`?
- Only amd64,i386 architectures at this time
- "i386" arch package doesn't work on pentium (but does work on i686)
- Cross compilation not actually possible yet
  - mostly because LLVM packages aren't ready

Note:
- The first point is re Mozilla Trademark policy

---

## `cargo`

- Packaging is a bit crude, but works
- Already in Debian *unstable*: https://packages.debian.org/sid/cargo

---

## `cargo` package build process

- Crate dependencies bundled and shipped along with cargo source package
- A snapshot of crates.io-index is bundled and shipped along with cargo source package
- Uses a python script (from Bitrig) to build stage0 cargo (without using cargo)
- Generates `.cargo/config` to point to bundled registry and crates
- Creates a fake temporary git repo for index
- Points `CARGO_HOME` at an empty directory
- Finally run regular cargo `configure`/`make`

---

## Patches applied

- Relax `missing_docs` lint in `aho-corasick`
- Fix relative paths in numerous bundled `Cargo.toml`s
- Remove cargo's `dev-dependencies` to prevent unnecessary attempted download

---

## cargo-using libraries

- Some early exploratory work, but mostly ideas so far
- Probably looks like Debian go-lang packages:
  - Library source installed into a central directory
  - Application builds pick up source from there
- Lots of issues still being worked on.  See [my recent post](https://internals.rust-lang.org/t/perfecting-rust-packaging-the-plan/2767/26?u=gus) in the "Perfecting Rust Packaging - The Plan" thread on [internals.rust-lang.org](https://internals.rust-lang.org)
  - Eg: Packaging from crates.io vs upstream repos
  - Overriding crate paths vs overriding cargo index

---

## cargo-using libraries (dylibs)

- *Can* support dylibs using tight package dependencies on `librust-xxx`
  - Need to be rebuilt following every compiler release
  - Will only do this if forced (compiler plugins?)

---

## cargo-using applications

- Once the libraries are solved, this should be easy!
- Run `cargo build --release`, copy the executable into the right directory
- Result will be an executable with no run-time dependencies on Rust crates (may require non-Rust libs)
- Need to be rebuilt following a Rust library security update

---

## The Last Slide

- <pkg-rust-maintainers@lists.alioth.debian.org>
- https://wiki.debian.org/Teams/RustPackaging
- \#debian-rust on OFTC IRC network
- Packaging git repos: http://anonscm.debian.org/cgit/pkg-rust/
  - Applied patches are in `*/debian/patches/`

- Questions?

Note:

- [RevealJS Demo/Manual](http://lab.hakim.se/reveal-js)
- [RevealJS Project/README](https://github.com/hakimel/reveal.js)
- [GitHub Flavored Markdown](https://help.github.com/articles/github-flavored-markdown)
