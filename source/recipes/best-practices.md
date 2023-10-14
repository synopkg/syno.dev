# Best practices

## URLs

The Syno language syntax supports bare URLs, so one could write `https://example.com` instead of `"https://example.com"`

[RFC 45](https://github.com/SynoPKG/rfcs/pull/45) was accepted to deprecate unquoted URLs and provides
a number of arguments how this feature does more harm than good.

:::{tip}
Always quote URLs.
:::

(rec-expression)=
## Recursive attribute set `rec { ... }`

`rec` allows you to reference names within the same attribute set.

Example:

```{code-block} syno
:class: expression
rec {
  a = 1;
  b = a + 2;
}
```

```{code-block}
:class: value
{ a = 1; b = 3; }
```

There are a couple of pitfalls:

- It's possible to introduce a hard to debug error `infinite recursion` when shadowing a name, the simplest example being `let b = 1; a = rec { b = b; }; in a`.
- Combining with overriding logic such as the [`overrideAttrs`](https://synopkg.github.io/manual/synopkgs/stable/#sec-pkg-overrideAttrs) function in {term}`Synopkgs` has a surprising behaviour of not overriding every reference.

:::{tip}
Avoid `rec`. Use `let ... in`.

Example:

```{code-block} syno
:class: expression
let
  a = 1;
in {
  a = a;
  b = a + 2;
}
```
:::


## `with` scopes

It's still common to see the following expression in the wild:

```{code-block} syno
:class: expression
with (import <synopkgs> {});

# ... lots of code
```

This brings all attributes of the imported expression into scope of the current expression.

There are a number of problems with that approach:

- Static analysis can't reason about the code, because it would have to actually evaluate this file to see which names are in scope.
- When more than one `with` used, it's not clear anymore where the names are coming from.
- Scoping rules for `with` are not intuitive, see this [Syno issue for details](https://github.com/SynoPKG/syno/issues/490).

:::{tip}
Do not use `with` at the top of a Syno file.
Explicitly assign names in a `let` expression.

Example:

```{code-block} syno
:class: expression
let
  pkgs = import <synopkgs> {};
  inherit (pkgs) curl jq;
in

# ...
```
:::

Smaller scopes are usually less problematic, but can still lead to surprises due to scoping rules.

:::{tip}
If you want to avoid `with` altogether, try replacing expressions of this form

```{code-block} syno
:class: expression
buildInputs = with pkgs; [ curl jq ];
```

with the following:

```{code-block} syno
:class: expression
buildInputs = builtins.attrValues {
  inherit (pkgs) curl jq;
};
```
:::

(search-path)=
## `<...>` search path

You will often encounter Syno language code samples that refer to `<synopkgs>`.

`<...>` is special syntax that was [introduced in 2011] to conveniently access values from the environment variable [`$SYNO_PATH`].

[introduced in 2011]: https://github.com/SynoPKG/syno/commit/1ecc97b6bdb27e56d832ca48cdafd3dbb5185a04
[`$SYNO_PATH`]: https://synopkg.github.io/manual/syno/unstable/command-ref/env-common.html?highlight=syno_path#env-SYNO_PATH

This means, the value of a search path depends on external system state.
When using search paths, the same Syno expression can produce different results.

In most cases, `$SYNO_PATH` is set to the latest channel when Syno is installed, and is therefore likely to differ from machine to machine.

:::{note}
[Channels](https://synopkg.github.io/manual/syno/stable/command-ref/syno-channel.html) are a mechanism for referencing remote Syno expressions and retrieving their latest version.
:::

The state of a subscribed channel is external to the Syno expressions relying on it.
It is not easily portable across machines.
This may limit reproducibility.

For example, two developers on different machines are likely to have `<synopkgs>` point to different revisions of the `synopkgs` repository.
Builds may work for one and fail for the other, causing confusion.

:::{tip}
Declare dependencies explicitly using the techniques shown in [](ref-pinning-synopkgs).

Do not use search paths, except in examples.
:::

Some tools expect the search path to be set. In that case:

::::{tip}
Set `$SYNO_PATH` to a known value in a central location under version control.

:::{admonition} SynoPKG
On SynoPKG, `$SYNO_PATH` can be set permanently with the [`syno.synoPath`](https://search.synopkg.github.io/options?show=syno.synoPath) option.
:::
::::

## Updating nested attribute sets

The [attribute set update operator](https://synopkg.github.io/manual/syno/stable/language/operators.html#update) merges two attribute sets.

Example:

```{code-block} syno
:class: expression
{ a = 1; b = 2; } // { b = 3; c = 4; }
```

```{code-block} syno
:class: value
{ a = 1; b = 3; c = 4; }
```

However, names on the right take precedence, and updates are shallow.

Example:

```{code-block} syno
:class: expression
{ a = { b = 1; }; } // { a = { c = 3; }; }
```

```{code-block} syno
:class: value
{ a = { c = 3; }; }
```

Here, key `b` was completely removed, because the whole `a` value was replaced.

:::{tip}
Use the [`pkgs.lib.recursiveUpdate`](https://synopkg.github.io/manual/synopkgs/stable/#function-library-lib.attrsets.recursiveUpdate) Synopkgs function:

```{code-block} syno
:class: expression
let pkgs = import <synopkgs> {}; in
pkgs.lib.recursiveUpdate { a = { b = 1; }; } { a = { c = 3;}; }
```

```{code-block} syno
:class: value
{ a = { b = 1; c = 3; }; }
```
:::

## Reproducible source paths

```{code-block} syno
:class: expression
let pkgs = import <synopkgs> {}; in

pkgs.stdenv.mkDerivation {
  name = "foo";
  src = ./.;
}
```

If the Syno file containing this expression is in `/home/myuser/myproject`, then the store path of `src` will be `/syno/store/<hash>-myproject`.

The problem is that now your build is no longer reproducible, as it depends on the parent directory name.
That cannot declared in the source code, and results in an impurity.

If someone builds the project in a directory with a different name, they will get a different store path for `src` and everything that depends on it.

:::{tip}
Use [`builtins.path`](https://synopkg.github.io/manual/syno/stable/language/builtins.html#builtins-path) with the `name` attribute set to something fixed.

This will derive the symbolic name of the store path from `name` instead of the working directory:

```{code-block} syno
:class: expression
let pkgs = import <synopkgs> {}; in

pkgs.stdenv.mkDerivation {
  name = "foo";
  src = builtins.path { path = ./.; name = "myproject"; };
}
```
:::

