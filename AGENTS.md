# AGENTS.md

This repository is the **Linux kernel** source tree. Development is build/test driven
via GNU Make (kbuild) + Kconfig; there are no application services, databases, or HTTP
ports. End-to-end work means: configure → build → run kernel code (KUnit/UML) or boot.

## Cursor Cloud specific instructions

The build toolchain (gcc, GNU make, flex, bison, bc, libssl-dev, libelf-dev,
libncurses-dev) is installed by the environment update script, so it is already present
for you. Notes that are NOT obvious from the standard docs:

- **KUnit under UML needs an exec-capable `TMPDIR`.** The default UML memory backing dir
  (`/dev/shm`) is mounted `noexec` in this environment, so `kunit.py run` builds fine but
  then prints `PROT_EXEC mmap in /dev/shm ... Operation not permitted` and reports
  "Could not find any KTAP output". Fix: run with `TMPDIR=/tmp` (which is exec-capable),
  e.g. `TMPDIR=/tmp ./tools/testing/kunit/kunit.py run --jobs=$(nproc)`. This is the
  fastest way to actually run kernel code and is the recommended "smoke test".

- **Build out-of-tree to keep the source clean.** Use `O=<dir>` (e.g. `make O=build-x86
  x86_64_defconfig && make O=build-x86 -j$(nproc)`). The produced kernel is
  `build-x86/arch/x86/boot/bzImage`. The `.kunit/` and `build-x86/` dirs are git-ignored.

- **Standard commands** (see `README` and `Documentation/admin-guide/quickly-build-trimmed-linux.rst`):
  - Lint a file: `./scripts/checkpatch.pl --no-tree --file <path>` (pre-existing files may
    already have style warnings; that is expected, not a regression).
  - Run KUnit tests: `TMPDIR=/tmp ./tools/testing/kunit/kunit.py run` (defaults to ARCH=um,
    no QEMU needed).
  - Build kernel: `make O=build-x86 x86_64_defconfig && make O=build-x86 -j$(nproc)`
    (~5 min on 4 cores).

- QEMU is **not** required for the default KUnit path (UML). It is only needed for
  `kunit.py run --arch=<non-um>` or for booting a built `bzImage`.
