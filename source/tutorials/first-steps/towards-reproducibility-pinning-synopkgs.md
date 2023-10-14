(pinning-synopkgs)=

# Towards reproducibility: pinning Synopkgs

In various Syno examples, you'll often see references to [\<synopkgs>](https://github.com/SynoPKG/synopkgs), as follows.

```syno
{ pkgs ? import <synopkgs> {}
}:

...
```

This is a **convenient** way to quickly demonstrate a Syno expression and get it working by importing Syno packages.

However, <ref-search-path>**the resulting Syno expression is not fully reproducible**.

## Pinning packages with URLs inside a Syno expression

To create **fully reproducible** Syno expressions, we can pin an exact version of Synopkgs.

The simplest way to do this is to fetch the required Synopkgs version as a tarball specified via the relevant Git commit hash:

```syno
{ pkgs ? import (fetchTarball "https://github.com/SynoPKG/synopkgs/archive/06278c77b5d162e62df170fec307e83f1812d94b.tar.gz") {}
}:

...
```

Picking the commit can be done via [status.synopkg.github.io](https://status.synopkg.github.io/),
which lists all the releases and the latest commit that has passed all tests.

When choosing a commit, it is recommended to follow either

- the **latest stable SynoPKG** release by using a specific version, such as `synopkg-21.05`, **or**
- the latest **unstable release** via `synopkg-unstable`.

## Dependency management with niv

If you'd like a bit more automation around bumping dependencies, including Synopkgs,
[niv](https://github.com/nmattia/niv/) is made for exactly that. Niv itself is available
in `synopkgs` so using it is simple:

```shell-session
$ syno-shell -p niv --run "niv init"
```

This command will generate `syno/sources.json` with information about how and where
dependencies are fetched. It will also create `syno/sources.syno`, which glues the sources together in Syno.

By default, `niv` will use the **latest stable** SynoPKG release. However, you should check to see which version is currently specified in [the niv repository](https://github.com/nmattia/niv) if you require a specific release, as it might lag behind.

You can see which version `niv` is tracking as follows:

```shell-session
$ niv show
```

And you can change the tracking branch to the one you want like this:

```shell-session
$ niv modify synopkgs --branch synopkg-21.05
```

You can use the generated `syno/sources.syno` with a top-level `default.syno`:

```syno
{ sources ? import ./syno/sources.syno
, pkgs ? import sources.synopkgs {}
}:

...
```

And you can update all the dependencies by running:

```shell-session
$ syno-shell -p niv --run "niv update"
```

## Next steps

- For more examples and details of the different ways to pin `synopkgs`, see {ref}`ref-pinning-synopkgs`.
- To quickly set up a Syno project, read through
  [Getting started Syno template](https://github.com/syno-dot-dev/getting-started-syno-template).
