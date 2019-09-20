install-clang
=============

This script installs self-contained standalone 9.0 versions of [clang][1],
[LLVM][2], [libc++][3], [compiler-rt][4], [libc++abi][5], [lldb][6],
and [lld][7], on macOS and Linux, including linking clang and LLVM
against libc++ themselves as well. The script keeps all of the
installation within a given target prefix (e.g., `/opt/clang`), and
hence separate from any already installed compilers, libraries, and
include files. In particular, you can later uninstall everything
easily by just deleting, e.g., `/opt/clang`. Furthermore, as long as
the prefix path is writable, the installation doesn't need root
privileges.

If you have used older version of this script before, see News below
for changes.

Usage
-----

To see the available options, use `-h`:

    > ./install-clang -h
    Usage: install-clang [<options>] <install-prefix>

    Available options:
        -A         enables assertions in LLVM libraries
        -b         build type (Release, Debug, RelWithDebInfo) [default: Release]
        -c         skip cloning repositories, assume they are in place
        -C         clean up after build by deleting the LLVM/clang source/build directories
        -h|-?      display this help
        -j <n>     build with <n> threads in parallel [default: 4]
        -m         use git/master instead of preconfigured versions
        -s <stage> begin build from <stage> [0, 1, 2]
        -S         build, and link against, shared libraries (this is the default on macOS)
        -u         update an existing build in <prefix> instead of installing new

    Environment variables:
        CC         path to the C compiler for bootstrapping
        CXX        path to the C++ compiler for bootstrapping

For example, to build Clang on a machine with multiple cores and
install it in `/opt/clang`, you can use:

    > ./install-clang -j 16 /opt/clang

Once finished, just prefix your PATH with `<prefix>/bin` and you're
ready to use the new binaries:

    > clang++ --std=c++11 --stdlib=libc++ test.cc -o a.out && ./a.out
    Hello, Clang!

By default, install-clang currently installs the 9.0 release branch of
https://github.com/llvm (the "mono repository"). Adding `-m` on the
command line instructs the script to use the current git master
version instead. The script downloads the source code from GitHub and
compiles the pieces as needed. Other OSs than macOS and Linux are not
currently supported.

The script also has an update option `-u` that allows for catching up
with upstream repository changes without doing the complete
compile/install-from-scratch cycle again. Note, however, that unless
coupled with `-m`, this flag has no immediate effect since the git
versions to use are hardcoded to the Clang/LLVM release version.

Doing a self-contained Clang installation is a bit more messy
than one would hope because the projects make assumptions about
specific system-wide installation paths to use. The install-clang
script captures some trial-and-error I (and others) went through to
get an independent setup working. It compiles Clang up to three
times, bootstrapping things with the system compiler as it goes. It
also patches some of the Clang/LLVM projects to incorporate the installation
prefix into configuration and search paths, and also fixes/tweaks a few
other things as well.

Docker
------

install-clang comes with a Dockerfile to build a Docker image, based
on Ubuntu, with Clang then in /opt/clang:

    # make docker-build && make docker-run
    [... get a beer ...]
    root@f39b941f177c:/# clang --version
    clang version 8.0.0
    Target: x86_64--linux-gnu
    Thread model: posix
    root@f39b941f177c:/# which clang
    /opt/clang/bin/clang

A prebuilt image is available at https://hub.docker.com/r/rsmmr/clang/ .

News
----

### Version for Clang 9 (master)

* Switch to using the LLVM "mono repo" and adapt script's build
  process.

* Build libunwind, and link against it.

* macOS: Allow usage of libc++'s filesystem on non-10.15 systems.

* macOS: Add `-rpath <prefix>/lib` to linker arguments (we already did
  this for Linux).

### Version for Clang 8 (git branch `release_80`)

The install-clang script for Clang 8.0 comes with these changes
compared to the 6.0 version:

* Fix computed RPATH when the clang binary has been symlinked from
  elsehwere (#18).

* We now honor preset environment variables CFLAGS, CXXFLAGS, and
  LDFLAGS to find dependencies.

* On macOS, we now set clang's DEFAULT_SYSROOT to the current macOS
  SDK.

* We apply a custom clang-format patch that adds a
  `SpacesAroundConditions` option.

* We backport clang-format's new option `SpaceAfterLogicalNot` from git.

* We build, and link against, static libraries by default (just as
  LLVM does), except on macOS. Option -S switches back to shared
  libraries.

* Removed any customziation for FreeBSD; support had been untested in
  a long time.

### Version for Clang 6 (git branch `release_60`)

The install-clang script for Clang 6.0 comes with these changes
compared to the 3.5 version:

* The default build type is now `Release`. It used to be
  `RelWithDebInfo`.

* lldb and lld seem to built correctly on both Linux and Darwin now
  and both are enabled by default.

* FreeBSD support is untested and hence disabled for now. Will be reactivated
  once confirmed that it's working.

* On macOS, we no longer build for i386.

* The Docker image is now based on Ubuntu Xenial and puts everything
  into `/opt/clang`.

### Version for Clang 3.5 (git tag `release_35`)

The install-clang script for Clang 3.5 comes with a few changes
compared to earlier version:

* The script now supports FreeBSD as well. (Contributed by Matthias
  Vallentin).

* The script now generally shared libraries for LLVM and clang, rather
  than static ones.

* As libc++abi now works well on Linux as well, we use it generally
  and no longer support libcxxrt.

* There are now command line options to select build mode and
  assertions explicitly.

* There's no 3rd phase anymore building assertion-enabled LLVM
  libraries, as changing compilation options isn't useful with shared
  libraries.

* In return, there's a phase 0 now if the system compiler isn't a
  clang; libc++abi needs clang that for its initial compilation
  already.

* There's now a Dockerfile to build an image with Clang/LLVM in
  /opt/clang.

[1]: http://clang.llvm.org
[2]: http://www.llvm.org
[3]: http://libcxx.llvm.org
[4]: http://compiler-rt.llvm.org
[5]: http://libcxxabi.llvm.org
[6]: http://lldb.llvm.org
[7]: http://lld.llvm.org
