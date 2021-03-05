## The Arm Research Starter Kit: System Modeling using gem5

This Research Starter Kit will guide you through Arm-based system modeling using the gem5 simulator and a 64-bit CPU model. This High-Performance In-order (HPI) CPU model is tuned to be representative of a modern in-order Armv8-A implementation.

You can find more information about the Arm Research Enablement Kits at: [https://developer.arm.com/solutions/research/research-enablement-kits](https://developer.arm.com/solutions/research/research-enablement-kits)

This Research Starter Kit is comprised of two main parts:

1. **[gem5](https://gem5.googlesource.com/public/gem5)**: the source code for the [gem5 simulator](https://www.gem5.org/) 
2. **[arm-gem5-rsk](https://github.com/arm-university/arm-gem5-rsk.git)**: the current repository, which contains the scripts, patches and the documentation required to get started, and also run examples and benchmarks 

Either clone the above repositories separately, or use the [clone.sh](https://raw.githubusercontent.com/arm-university/arm-gem5-rsk/master/clone.sh) script to clone both of them:
```bash
$ wget https://raw.githubusercontent.com/arm-university/arm-gem5-rsk/master/clone.sh
$ bash clone.sh
```

The current release includes the following components:
* **gem5_rsk.pdf**: the documentation
* **clone.sh**: a script to download all the required materials
* **read_results.sh**: a script to read the gem5 statistics
* **parsec_patches**: contains patches for compiling PARSEC for the gem5 Full-System simulation mode
* **parsec_rcs**: contains a script for creating runscripts for PARSEC benchmarks
* [Wiki](https://github.com/arm-university/arm-gem5-rsk/wiki): a cheat sheet, containing all code and examples provided in the documentation


## Applying patches

Assuming the following directory structure (from the point of view of having
just cloned this repository):

```
./parsec-3.0-core.tar.gz
./arm-gem5-rsk
```

Extract and make a non-patched backup of Parsec 3.0:

```bash
$ tar -xzvf parsec-3.0-core.tar.gz
$ cp -r parsec-3.0 parsec-3.0_nopatch
```

Copy the cross-compile patch and apply it:

```bash
$ cp arm-gem5-rsk/parsec_patches/xcompile-patch.diff .
$ patch -p0 < xcompile-patch.diff
```

Change to the `parsec-3.0` directory, copy the static patch (which is needed
since dynamically linked libraries are unlikely to be available on gem5), and
apply it:

```
$ cd parsec-3.0
$ cp ../arm-gem5-rsk/parsec_patches/static-patch.diff .
$ patch -p1 < static-patch.diff
```

The `-p1` tells `patch` to ignore 1 leading component from the file names. This
is required because the diff-file lists the diffs as
`a/path/to/file b/path/to/file`, where `a/` and `b/` are there to illustrate the
difference and can otherwise be ignored.


## Building parsec

Assuming you are still in the `parsec-3.0` directory and you have an ARM
cross-compiler toolchain,
[e.g. GNU-A](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads),
extracted to a directory (in my case `$HOME/xtools`) prepare for cross-compiling
and set up the parsec environment:

```bash
export XCOMP_DIR="$HOME/xtools/gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu"
source env.sh
```

(note that there isn't a trailing forward-slash in the `XCOMP_DIR` path)

Use `parsecmgmt` to (try to) build the benchmarks required.

```bash
$ parsecmgmt -a build -p parsec.<benchmark>
```

Note that Splash2 doesn't seem to be changed by the patches, and so is not
cross-compiled. If you have a patch for this, I'd be grateful if you could
submit a PR! (If you have one for cross-compiling Splash3, that'd be even
better!)

From some brief testing conducted on the following setup:

- CPU: Intel Core i7-8750H
- Kernel: Linux 5.10.19
- xcomp gcc: 10.2.1 (for the A-profile Architecture 10.2-2020.11 (arm-10.16))
- parsec: 3.0 (downloaded on 2021-03-05 from
    [the website](https://parsec.cs.princeton.edu/parsec3-doc.htm))

the results for cross-compiling are:

| Benchmark               | XComp works? | Notes                               |
| :---------------------- | :----------: | ----------------------------------: |
| parsec.blackscholes     |      [x]     |                                     |
| parsec.bodytrack        |      [ ]     | C++ error in `WorkerGroup.cpp`      |
| parsec.canneal          |      [x]     |                                     |
| parsec.dedup            |      [ ]     | The `config` errors when run        |
| parsec.facesim          |      [ ]     | No rule to make a required library  |
| parsec.ferret           |      [ ]     | Code error: Undeclared var `HUGE`   |
| parsec.fluidanimate     |      [x]     |                                     |
| parsec.freqmine         |      [x]     |                                     |
| parsec.netdedup         |      [ ]     | `freebsd.netinet/*.o` not found     |
| parsec.netferret        |      [ ]     | as above                            |
| parsec.netstreamcluster |      [ ]     | as above                            |
| parsec.raytrace         |      [ ]     | CMake can't find CC to compile with |
| parsec.streamcluster    |      [x]     |                                     |
| parsec.swaptions        |      [x]     |                                     |
| parsec.vips             |      [ ]     | Relies on executable during build   |
| parsec.x264             |      [x]     |                                     |

