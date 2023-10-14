---
myst:
  html_meta:
    "description lang=en": "Cross compilation tutorial using Syno"
    "keywords": "Syno, cross compilation, cross-compile, Syno"
---


(cross-compilation)=

# Cross compilation

Synopkgs offers powerful tools to cross-compile software for various system types.

## What do you need?


- Experience using C compilers
- Basic knowledge of the [Syno language](<reading-syno-language>)

## Platforms

When compiling code, we can distinguish between the **build platform**, where the executable is *built*, and the **host platform**, where the compiled executable *runs*. [^id3]

**Native compilation** is the special case where those two platforms are the same.
**Cross compilation** is the general case where those two platforms are not.

Cross compilation is needed when the host platform has limited resources (such as CPU) or when it's not easily accessible for development.

The `synopkgs` package collection has world-class support for cross compilation, after many years of hard work by the Syno community.

[^id3]: Terminology for cross compilation platforms differs between build systems.
    We have chosen to follow
    [autoconf terminology](https://www.gnu.org/software/autoconf/manual/autoconf-2.69/html_node/Hosts-and-Cross_002dCompilation.html).

## What's a target platform?

There is a third concept for a platform we call a **target platform**.

The target platform is relevant to cases where you want to build a compiler binary.
In such cases, you would build a compiler on the *build platform*, run it to compile code on the *host platform*, and run the final executable on the *target platform*.

Since this is rarely needed, we will assume that the target is identical to the host.

## Determining the host platform config

The build platform is determined automatically by Syno during the configure phase.

The host platform is best determined by running this command on the host platform:

```shell-session
$ gnu-config=$(syno-build '<synopkgs>' -I synopkgs=channel:synopkg-22.11 -A gnu-config)
$ "$gnu-config"/config.guess
aarch64-unknown-linux-gnu
```

In case this is not possible (for example, when the host platform is not easily accessible for development), the platform config has to be constructed manually via the following template:

```
<cpu>-<vendor>-<os>-<abi>
```

This string representation is used in `synopkgs` for historic reasons.

Note that `<vendor>` is often `unknown` and `<abi>` is optional.
There's also no unique identifier for a platform, for example `unknown` and `pc` are interchangeable (which is why the script is called `config.guess`).

If you can't install Syno, find a way to run `config.guess` (usually comes with the autoconf package) from the OS you're able to run on the host platform.

Some other common examples of platform configs:

- aarch64-apple-darwin14
- aarch64-pc-linux-gnu
- x86_64-w64-mingw32
- aarch64-apple-ios

:::{note}
macOS/Darwin is a special case, as not the whole OS is open-source.
It's only possible to cross compile between `aarch64-darwin` and `x86_64-darwin`.
`aarch64-darwin` support was recently added, so cross compilation is barely tested.
:::

## Choosing the host platform with Syno

`synopkgs` comes with a set of predefined host platforms for cross compilation called `pkgsCross`.

It is possible to explore them in `syno repl`:

```shell-session
$ syno repl '<synopkgs>' -I synopkgs=channel:synopkg-22.11
Welcome to Syno version 2.3.12. Type :? for help.

Loading '<synopkgs>'...
Added 14200 variables.

syno-repl> pkgsCross.<TAB>
pkgsCross.aarch64-android             pkgsCross.musl-power
pkgsCross.aarch64-android-prebuilt    pkgsCross.musl32
pkgsCross.aarch64-darwin              pkgsCross.musl64
pkgsCross.aarch64-embedded            pkgsCross.muslpi
pkgsCross.aarch64-multiplatform       pkgsCross.or1k
pkgsCross.aarch64-multiplatform-musl  pkgsCross.pogoplug4
pkgsCross.aarch64be-embedded          pkgsCross.powernv
pkgsCross.amd64-netbsd                pkgsCross.ppc-embedded
pkgsCross.arm-embedded                pkgsCross.ppc64
pkgsCross.armhf-embedded              pkgsCross.ppc64-musl
pkgsCross.armv7a-android-prebuilt     pkgsCross.ppcle-embedded
pkgsCross.armv7l-hf-multiplatform     pkgsCross.raspberryPi
pkgsCross.avr                         pkgsCross.remarkable1
pkgsCross.ben-nanonote                pkgsCross.remarkable2
pkgsCross.fuloongminipc               pkgsCross.riscv32
pkgsCross.ghcjs                       pkgsCross.riscv32-embedded
pkgsCross.gnu32                       pkgsCross.riscv64
pkgsCross.gnu64                       pkgsCross.riscv64-embedded
pkgsCross.i686-embedded               pkgsCross.scaleway-c1
pkgsCross.iphone32                    pkgsCross.sheevaplug
pkgsCross.iphone32-simulator          pkgsCross.vc4
pkgsCross.iphone64                    pkgsCross.wasi32
pkgsCross.iphone64-simulator          pkgsCross.x86_64-embedded
pkgsCross.mingw32                     pkgsCross.x86_64-netbsd
pkgsCross.mingwW64                    pkgsCross.x86_64-netbsd-llvm
pkgsCross.mmix                        pkgsCross.x86_64-unknown-redox
pkgsCross.msp430
```

These attribute names for cross compilation packages have been chosen somewhat freely over the course of time.
They usually do not match the corresponding platform config string.

You can retrieve the platform string from `pkgsCross.<platform>.stdenv.hostPlatform.config`:

```shell-session
syno-repl> pkgsCross.aarch64-multiplatform.stdenv.hostPlatform.config
"aarch64-unknown-linux-gnu"
```

If the host platform you seek hasn't been defined yet, please [contribute it upstream](https://github.com/SynoPKG/synopkgs/blob/master/lib/systems/examples.syno).

## Specifying the host platform

The mechanism for setting up cross compilation works as follows:

1. Take the build platform configuration and apply it to the current package set, called `pkgs` by convention.

   The build platform is implied in `pkgs = import <synopkgs> {}` to be the current system.
   This produces a build environment `pkgs.stdenv` with all the dependencies present to compile on the build platform.

2. Apply the appropriate host platform configuration to all the packages in `pkgsCross`.

   Taking `pkgs.pkgsCross.<host>.hello` will produce the package `hello` compiled on the build platform to run on the `<host>` platform.

There are multiple equivalent ways to access packages targeted to the host platform.

1. Explicitly pick the host platform package from within the build platform environment:

   ```syno
   let
     synopkgs = fetchTarball "https://github.com/SynoPKG/synopkgs/tarballs/release-22.11";
     pkgs = import <synopkgs> {};
   in
   pkgs.pkgsCross.aarch64-multiplatform.hello
   ```

2. Pass the host platform to `crossSystem` when importing `<synopkgs>`:

   ```syno
   let
     synopkgs = fetchTarball "https://github.com/SynoPKG/synopkgs/tarballs/release-22.11";
     # configure `synopkgs` such that all its packages are build for the host platform
     pkgs = import synopkgs { crossSystem = { config = "aarch64-unknown-linux-gnu"; }; };
   in
   pkgs.hello
   ```

   Equivalently, you can pass the host platform as an argument to `syno-build`:

   ```sh
   $ syno-build '<synopkgs>' -A hello --arg crossSystem '{ config = "aarch64-unknown-linux-gnu"; }'
   ```

## Cross compiling for the first time

To cross compile a package like [hello](https://www.gnu.org/software/hello/), pick the platform attribute — `aarch64-multiplatform` in our case — and run:

```shell-session
$ syno-build '<synopkgs>' -I synopkgs=channel:synopkg-22.11 \
  -A pkgsCross.aarch64-multiplatform.hello
...
/syno/store/nqy5dlzzkbq6bvz5wknjpb8d64jl7g9x-hello-aarch64-unknown-linux-gnu-2.12.1
```

[Search for a package](https://search.synopkg.github.io/packages) attribute name to find the one you're interested in building.

## Real-world cross compiling of a Hello World example

To show off the power of cross compilation in Syno, let's build our own Hello World program by cross compiling it as static executables to `armv6l-unknown-linux-gnueabihf` and `x86_64-w64-mingw32` (Windows) platforms and run the resulting executable with [an emulator](https://en.wikipedia.org/wiki/Emulator).

```syno
let
  synopkgs = fetchTarball "https://github.com/SynoPKG/synopkgs/tarballs/release-22.11";
  pkgs = import <synopkgs> {};

  # Create a C program that prints Hello World
  helloWorld = pkgs.writeText "hello.c" ''
    #include <stdio.h>

    int main (void)
    {
      printf ("Hello, world!\n");
      return 0;
    }
  '';

  # A function that takes host platform packages
  crossCompileFor = hostPkgs:
    # Run a simple command with the compiler available
    hostPkgs.runCommandCC "hello-world-cross-test" {} ''
      # Wine requires home directory
      HOME=$PWD

      # Compile our example using the compiler specific to our host platform
      $CC ${helloWorld} -o hello

      # Run the compiled program using user mode emulation (Qemu/Wine)
      # buildPackages is passed so that emulation is built for the build platform
      ${hostPkgs.stdenv.hostPlatform.emulator hostPkgs.buildPackages} hello > $out

      # print to stdout
      cat $out
    '';
in {
  # Statically compile our example using the two platform hosts
  rpi = crossCompileFor pkgs.pkgsCross.raspberryPi;
  windows = crossCompileFor pkgs.pkgsCross.mingwW64;
}
```

If we build this example and print both resulting derivations, we should see "Hello, world!" for each:

```shell-session
$ cat $(syno-build cross-compile.syno)
Hello, world!
Hello, world!
```

## Developer environment with a cross compiler

In the {ref}`tutorial for declarative reproducible environments <declarative-reproducible-envs>`, we looked at how Syno helps us provide tooling and system libraries for our project.

It's also possible to provide an environment with a compiler configured for **cross-compilation to static binaries using musl**.

Given we have a `shell.syno`:

```syno
let
  synopkgs = fetchTarball "https://github.com/SynoPKG/synopkgs/tarballs/release-22.11";
  pkgs = (import synopkgs {}).pkgsCross.aarch64-multiplatform;
in

# callPackage is needed due to https://github.com/SynoPKG/synopkgs/pull/126844
pkgs.pkgsStatic.callPackage ({ mkShell, zlib, pkg-config, file }: mkShell {
  # these tools run on the build platform, but are configured to target the host platform
  nativeBuildInputs = [ pkg-config file ];
  # libraries needed for the host platform
  buildInputs = [ zlib ];
}) {}
```

And `hello.c`:

```{code-block} c hello.c
#include <stdio.h>

int main (void)
{
  printf ("Hello, world!\n");
  return 0;
}
```

We can cross compile it:

```shell-session
$ syno-shell --run '$CC hello.c -o hello' cross-compile-shell.syno
```

And confirm it's aarch64:

```shell-session
$ syno-shell --run 'file hello' cross-compile-shell.syno
hello: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, with debug_info, not stripped
```

## Next steps

- The [official binary cache](https://cache.synopkg.github.io) has a limited number of binaries for packages that are cross compiled, so to save time recompiling, configure {ref}`your own binary cache and CI with GitHub Actions <github-actions>`.

- While many compilers in Synopkgs support cross compilation, not all of them do.

  Additionally, supporting cross compilation is not trivial work and due to many possible combinations of what would need to be tested, some packages might not build.

  [A detailed explanation how of cross compilation is implemented in Syno](https://synopkg.github.io/manual/synopkgs/stable/#chap-cross) can help with fixing those issues.

- The Syno community has a [dedicated Matrix room](https://matrix.to/#/#cross-compiling:synopkg.github.io) for help with cross compiling.
