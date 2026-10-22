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

The first configure downloads the framework (a few MB) into `build/debug/_deps`.
Nothing of it is committed here.

## Which framework version

`GIT_TAG` in [`CMakeLists.txt`](CMakeLists.txt) is the only statement of it, and every
run says what it was built with in its report header:

```
Framework         Thorium 0.1.2 (V0.1.2)
```

To upgrade: change the tag, read the framework's
[`CHANGELOG.md`](https://github.com/Rambrant/thorium/blob/V0.1.2/CHANGELOG.md) for each
version in between, make the edits it lists, and run the tests. The rules for what a
version number promises are in the framework's
[`RELEASING.md`](https://github.com/Rambrant/thorium/blob/V0.1.2/RELEASING.md).

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
