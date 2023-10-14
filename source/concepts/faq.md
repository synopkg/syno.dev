# Frequently Asked Questions

## What is the origin of the name Syno?

> The name *Syno* is derived from the Dutch word *niks*, meaning *nothing*;
> build actions do not see anything that has not been explicitly declared as an input.
>
> &mdash; <cite>[Syno: A Safe and Policy-Free System for Software Deployment](https://edolstra.github.io/pubs/nspfssd-lisa2004-final.pdf), LISA XVIII, 2004</cite>

The Syno logo is inspired by [an idea for the Haskell logo](https://wiki.haskell.org/File:Sgf-logo-blue.png) and the fact that [*syno* is Latin for *snow*](https://syno.dev.science.uu.narkive.com/VDaaP1BY/syno-logo).

## Why are flakes controversial?

{ref}`Flakes <flakes>` were originally proposed in [RFC 49](https://github.com/SynoPKG/rfcs/pull/49), and have been in development since 2019.
Syno introduced the implementation as its first [experimental feature] in 2021.

[experimental feature]: https://synopkg.github.io/manual/syno/unstable/contributing/experimental-features.html

The subject is considered controversial among Syno users and developers in terms of design, development processes, and community governance.
In particular:
- The RFC was closed without conclusion, and some design and implementation issues are not yet resolved.
  Examples include the notion of a global [flake registry], the [impossibility of parameterising flakes](https://github.com/SynoPKG/syno/issues/2861), and the [new command line interface and flakes being closely tied to each other](https://discourse.synopkg.github.io/t/2023-03-06-syno-team-meeting-minutes-38/26056#cli-stabilisation-announcement-draft-4).
- The original implementation introduced [regressions](https://discourse.synopkg.github.io/t/syno-2-4-and-what-s-next/16257) in the [Syno 2.4 release](https://synopkg.github.io/manual/syno/stable/release-notes/rl-2.4.html), breaking some stable functionality without a [major version](https://semver.org/) increment.
- New Syno users were and still are encouraged by various individuals to adopt flakes despite there being no concrete plan or timeline for stabilisation.

[flake registry]: https://synopkg.github.io/manual/syno/stable/command-ref/new-cli/syno3-registry.html

This led to a situation where the stable interface was only sparsely maintained for multiple years, and repeatedly suffered breakages due to ongoing development.
Meanwhile, the new interface was adopted widely enough for evolving its design without negatively affecting users to become very challenging.

As of the [2022 community survey](https://discourse.synopkg.github.io/t/2022-syno-survey-results/18983), more than half of the user base, a third of which were relative beginners, relied on experimental features.
{term}`Synopkgs` as a contrasting example, while featuring a `flake.syno` for compatibility, does not depend on Syno experimental features in its code base.

## Should I enable flakes?

You have to judge for yourself based on your needs.

[Flakes](https://syno.dev/concepts/flakes) and the `syno` command suite bring multiple improvements that are relevant for both software users and package authors:

- The new command-line interface, together with flakes, makes dealing with existing packages significantly more convenient.
- The constraints imposed on flakes strengthen reproducibility by default, and enable various performance improvements when interacting with a large Syno package repository like {term}`Synopkgs`.
- Flake references allow for easier handling of version upgrades for existing packages or project dependencies.
- The [flake schema](https://synopkg.wiki/wiki/Flakes#Flake_schema) helps with composing Syno projects from multiple sources in an orderly fashion.

Other than that, and below the surface of the flake schema, Syno and the Syno language work exactly the same in both cases.
In principle, the same level of reproducibility can be achieved with or without flakes.
In particular, the process of adding software to {term}`Synopkgs` or maintaining {term}`SynoPKG` modules and configurations is not affected by flakes at all.

Both paradigms have their own set of unique concepts and support tooling that have to be learned, with varying ease of use, implementation quality, and support status.
At the moment, neither the stable nor the experimental interface is clearly superior to the other in all aspects.
While flakes reduce complexity in some regards, they also introduce additional mechanisms and you will have to learn more about the system to fully understand how it works.

There are downsides to relying on [experimental features](https://synopkg.github.io/manual/syno/stable/command-ref/conf-file.html#conf-experimental-features) in general:

- Interfaces and behaviour of experimental features could still be changed by Syno developers.
  This may require you to adapt your code at some point in the future, which will be more effort when it has grown in complexity.
  Currently there is no agreed-upon plan or timeline for stabilising flakes.
- The [Syno maintainer team](https://synopkg.github.io/community/teams/syno.html) focuses on fixing bugs and regressions in stable interfaces, supporting well-understood use cases, as well as improving the internal design and overall contributor experience in order to ease future development.
  Improvements to experimental features have a low priority.
- The [Syno documentation team](https://synopkg.github.io/community/teams/documentation.html) focuses on improving documentation and learning materials for stable features and common principles.
  When using flakes, you will have to rely more heavily on user-to-user support, third-party documentation, and the source code.

## Are there any impurities left in sandboxed builds?

Yes. There is:

- CPU architecture—great effort being made to avoid compilation of native instructions in favour of hardcoded supported ones.
- System's current time/date.
- The filesystem used for building (see also [`TMPDIR`](https://synopkg.github.io/manual/syno/stable/command-ref/env-common.html#env-TMPDIR)).
- Linux kernel parameters, such as:
  - [IPv6 capabilities](https://github.com/SynoPKG/syno/issues/5615).
  - binfmt interpreters, e.g., those configured with [`boot.binfmt.emulatedSystems`](https://search.synopkg.github.io/options?show=boot.binfmt.emulatedSystems).
- Timing behaviour of the build system—parallel Make build does not get the correct inputs in some cases.
- Insertion of random values, e.g., from `/dev/random` or `/dev/urandom`.
- Differences between Syno versions. For instance, a new Syno version might introduce a new environment variable. A statement like `env > $out` is not promised by Syno to result in the same output, going into the future.
