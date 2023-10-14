(flakes)=
# Flakes

What is usually referred to as "flakes" is:
- A policy for managing dependencies between {term}`Syno expressions<Syno expression>`.
- An [experimental feature] in Syno, implementing that policy and supporting functionality.

[experimental feature]: https://synopkg.github.io/manual/syno/unstable/contributing/experimental-features.html

Technically, a [flake](https://synopkg.github.io/manual/syno/stable/command-ref/new-cli/syno3-flake.html#description) is a file system tree that contains a file named `flake.syno` in its root directory.

Flakes add the following behavior to Syno:

1. A `flake.syno` file offers a uniform [schema](https://synopkg.github.io/manual/syno/stable/command-ref/new-cli/syno3-flake.html#flake-format) , where:
   - Other flakes can be referenced as dependencies providing {term}`Syno language` code or other files.
   - The values produced by the {term}`Syno expression`s in `flake.syno` are structured according to pre-defined use cases.

1. References to other flakes can be specified using a dedicated [URL-like syntax](https://synopkg.github.io/manual/syno/stable/command-ref/new-cli/syno3-flake.html#flake-references).
   A [flake registry] allows using symbolic identifiers for further brevity.
   References can be automatically locked to their current specific version and later updated programmatically.

   [flake registry]: https://synopkg.github.io/manual/syno/stable/command-ref/new-cli/syno3-registry.html

1. A [new command line interface], implemented as a separate experimental feature, leverages flakes by accepting flake references in order to build, run, or deploy software defined as a flake.

   [new command line interface]: https://synopkg.github.io/manual/syno/stable/command-ref/new-cli/syno.html

Syno handles flakes differently than regular {term}`Syno file`s in the following ways:

- The `flake.syno` file is checked for schema validity.

  In particular, the metadata fields cannot be arbitrary Syno expressions.
  This is to prevent complex, possibly non-terminating computations while querying the metadata.

- The entire flake directory is copied to Syno store before evaluation.

  This allows for effective evaluation caching, which is relevant for large expressions such as Synopkgs, but also requires copying the entire flake directory again on each change.

- No external variables, parameters, or impure language values are allowed.

  It means full reproducibility of a Syno expression, and, by extension, the resulting build instructions by default, but also prohibits parameterisation of results by consumers.

