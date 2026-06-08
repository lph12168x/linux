# AGENTS.md

## Cursor Cloud specific instructions

This repository is the **Linux kernel source tree** (currently 7.1.0-rc6). Development is Makefile/Kbuild-driven — there is no `package.json`, `docker-compose`, or in-repo dependency manager.

### One-time system packages (Ubuntu/Debian)

Install build dependencies before the first compile (documented in `Documentation/admin-guide/quickly-build-trimmed-linux.rst`):

```bash
sudo apt-get install -y bc binutils bison dwarves flex gcc git make openssl \
  pahole perl-base libssl-dev libelf-dev pkg-config python3 rsync kmod
```

Optional extras for broader tool/selftest coverage:

```bash
sudo apt-get install -y libcap-dev libcap-ng-dev libzstd-dev python3-dev \
  libdw-dev libdebuginfod-dev libslang2-dev systemtap-sdt-dev
```

Verify the toolchain with `./scripts/ver_linux` and compare against `Documentation/process/changes.rst`.

### Configure and build

```bash
make defconfig          # or: yes "" | make localmodconfig
make -j$(nproc)         # produces vmlinux and arch/x86/boot/bzImage
make -j$(nproc) modules # optional: loadable modules
```

A full `defconfig` build needs ~12 GB disk and takes several minutes on 4 cores. Use `O=/path/to/output` for out-of-tree builds.

### Running tests (no reboot required)

Many kselftests run against the **host kernel** without installing/booting the newly built image:

```bash
make headers
sudo make TARGETS="mincore clone3" kselftest
```

- `bpf` and `sched_ext` are skipped by default (`SKIP_TARGETS` in `tools/testing/selftests/Makefile`).
- Some tests need root (`sudo`), test kernel modules (`CONFIG_TEST_*=m`), or container capabilities and will SKIP/FAIL in restricted VMs — this is expected.
- Full E2E validation (install + reboot into new kernel) is documented in `Documentation/admin-guide/quickly-build-trimmed-linux.rst` but is not possible in ephemeral cloud VMs.

### Userspace tools

```bash
make -C tools/perf NO_LIBTRACEEVENT=1 -j$(nproc)   # minimal perf build
./tools/perf/perf --version
```

`bpftool` requires `CONFIG_DEBUG_INFO_BTF=y` in `.config`; default `defconfig` may not enable it.

### Lint / static analysis

There is no repo-wide ESLint/tsc equivalent. Common checks:

- `./scripts/checkpatch.pl --no-tree <file>` for patch style
- `make clang-format` (if `clang-format` is installed) for formatting
- `make W=1` during build for extra compiler warnings

### Key references

- `README` — role-based entry points
- `Documentation/admin-guide/quickly-build-trimmed-linux.rst` — fast trimmed builds
- `Documentation/process/changes.rst` — minimum tool versions
- `tools/testing/selftests/` — kernel selftest harness (TAP output)
