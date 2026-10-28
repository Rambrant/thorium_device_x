# thorium_device_x -- Device X's test rig and suite

The tests for Device X, built on the [Thorium](https://github.com/Rambrant/thorium)
framework. This repository holds only what is Device X's; the framework, the
runner and the instrument drivers are fetched at a pinned version when you build.

```
CMakeLists.txt       fetch the pinned framework, then thorium_deployment()
CMakePresets.json    the compiler presets (GCC 16: Thorium needs C++26 reflection)
rig/                 this bench: instruments, switching cards, wiring
dut/                 Device X: adapter points, criteria variants
suite/               its test scripts, catalog, and the tests of them
acceptance/          black-box tests over this suite's own run_scripts
```

## Build and run

```bash
cmake --preset macos-debug          # or windows-debug / linux-debug
cmake --build build/debug
ctest --test-dir build/debug        # this repository's own tests
./build/debug/bin/run_scripts       # the suite; --help lists every flag
```

**In an IDE, use the preset.** Thorium needs GCC 16 (C++26 reflection and contracts), and
the compiler is chosen only by `CMakePresets.json` -- an IDE profile that ignores it
falls back to the system compiler (AppleClang on macOS) and the configure stops with
"Thorium needs GCC 16 or newer". In CLion, enable the `macos-debug` (or your
platform's) profile under Settings | Build, Execution, Deployment | CMake and disable
the default "Debug" one; any other IDE needs the same compiler passed as
`-DCMAKE_CXX_COMPILER=...`.

The first configure downloads the framework (a few MB) into `build/debug/_deps`.
Nothing of it is committed here.

## Which framework version

`GIT_TAG` in [`CMakeLists.txt`](CMakeLists.txt) is the only statement of it, and every
run says what it was built with in its report header:

```
Framework         Thorium 0.1.2 (v0.1.2)
```

To upgrade: change the tag, read the framework's
[`CHANGELOG.md`](https://github.com/Rambrant/thorium/blob/v0.1.2/CHANGELOG.md) for each
version in between, make the edits it lists, and run the tests. The rules for what a
version number promises are in the framework's
[`RELEASING.md`](https://github.com/Rambrant/thorium/blob/v0.1.2/RELEASING.md).

## Building on an air-gapped rig computer

The framework is the only thing the build downloads, and `THORIUM_REPOSITORY` says where
it comes from. `GIT_TAG` stays pinned as above; on a rig with no network you point the
repository at a copy on local disk or on a share, and the build still checks out exactly
that tag (it stops if the tag is missing from the copy).

**On a connected machine, once per framework version:**

```bash
git clone --mirror https://github.com/Rambrant/thorium.git thorium.git
```

(Later, `git -C thorium.git remote update` brings it up to date.) Copy `thorium.git/`
to the rig computer, say to `/opt/thorium.git`, together with this repository.

**On the rig computer:**

```bash
cmake --preset linux-release -DTHORIUM_REPOSITORY=/opt/thorium.git
cmake --build build/release
./build/release/bin/run_scripts
```

The setting is remembered in that build directory. To make it permanent for this
machine, put it in a `CMakeUserPresets.json` (not committed) that inherits the platform
preset and sets `THORIUM_REPOSITORY`.

**Upgrading the framework offline:** refresh the mirror on the connected machine, copy it
over, check the tag arrived (`git -C /opt/thorium.git tag -l v0.1.3`), then change
`GIT_TAG` here and follow the upgrade steps above.

Things that go wrong:

- *"dubious ownership"* from git: the copy belongs to another user. `chown` it, or
  `git config --global --add safe.directory /opt/thorium.git`.
- *Tag not found*: the mirror is older than `GIT_TAG`. Refresh it.
- The GCC 16 toolchain has to be on the rig computer too; that is not something this
  repository can bring.

Once a build directory has been configured, the framework lives in its `_deps/`, so
rebuilding needs no mirror; only a fresh build directory, or a changed `GIT_TAG`, does.

To work on the framework and Device X together, point the build at a checkout:

```bash
cmake --preset macos-debug -DFETCHCONTENT_SOURCE_DIR_THORIUM=../thorium
```

## Where to read about the pieces

Each directory has its own README ([`rig/`](rig/README.md), [`dut/`](dut/README.md),
[`suite/`](suite/README.md), [`acceptance/`](acceptance/README.md)). They were written
inside the framework repository and say "see `framework/...`" for the mechanism
behind a table or macro; those paths are in the
[framework repository](https://github.com/Rambrant/thorium) at the tag above.
