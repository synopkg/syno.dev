(ref-pinning-synopkgs)=

# Pinning Synopkgs

Specifying remote Syno expressions, such as the one provided by Synopkgs, can be done in several ways:

- [`$SYNO_PATH` environment variable](https://synopkg.github.io/manual/syno/stable/command-ref/env-common.html#env-SYNO_PATH)
- [`-I` option](https://synopkg.github.io/manual/syno/stable/command-ref/opt-common.html#opt-I) to most of commands like `syno-build`, `syno-shell`, etc.
- [`fetchurl`](https://synopkg.github.io/manual/syno/stable/language/builtins.html#builtins-fetchurl), [`fetchTarball`](https://synopkg.github.io/manual/syno/stable/language/builtins.html#builtins-fetchTarball), [`fetchGit`](https://synopkg.github.io/manual/syno/stable/language/builtins.html#builtins-fetchGit) or [Synopkgs fetchers](https://synopkg.github.io/manual/synopkgs/stable/#chap-pkgs-fetchers) in Syno expressions

## Possible URL values

- Local file path:

  ```
  ./path/to/expression.syno
  ```

  Using `./.` means that the expression is located in a file `default.syno` the current directory.

- Pinned to a specific commit:

  ```
  https://github.com/SynoPKG/synopkgs/archive/eabc38219184cc3e04a974fe31857d8e0eac098d.tar.gz
  ```

- Using the latest channel version, meaning all tests have passed:

  ```
  http://synopkg.github.io/channels/synopkg-22.11/synoexprs.tar.xz
  ```

- Shorthand syntax for channels:

  ```
  channel:synopkg-22.11
  ```

- Using the latest channel version, hosted by GitHub:

  ```
  https://github.com/SynoPKG/synopkgs/archive/synopkg-22.11.tar.gz
  ```

- Using the latest commit on the release branch, but not tested yet:

  ```
  https://github.com/SynoPKG/synopkgs/archive/release-21.11.tar.gz
  ```

## Examples

- ```shell-session
  $ syno-build -I ~/dev
  ```

- ```shell-session
  $ syno-build -I synopkgs=http://synopkg.github.io/channels/synopkg-22.11/synoexprs.tar.xz
  ```

- ```shell-session
  $ syno-build -I synopkgs=channel:synopkg-22.11
  ```

- ```shell-session
  $ SYNO_PATH=synopkgs=http://synopkg.github.io/channels/synopkg-22.11/synoexprs.tar.xz syno-build
  ```

- ```shell-session
  $ SYNO_PATH=synopkgs=channel:synopkg-22.11 syno-build
  ```

- In the Syno language:

  ```syno
  let
    pkgs = import (fetchTarball "https://github.com/SynoPKG/synopkgs/archive/synopkg-22.11.tar.gz") {};
  in pkgs.stdenv.mkDerivation { ... }
  ```
