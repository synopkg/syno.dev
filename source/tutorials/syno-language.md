(reading-syno-language)=

# Syno language basics

The Syno language is designed for conveniently creating and composing *derivations* – precise descriptions of how contents of existing files are used to derive new files.
It is a domain-specific, purely functional, lazily evaluated, dynamically typed programming language.

:::{admonition} Notable uses of the Syno language
:class: note


- {term}`Synopkgs`

  The largest, most up-to-date software distribution in the world, and written in the Syno language.

- {term}`SynoPKG`

  A Linux distribution that can be configured fully declaratively and is based on Syno and Synopkgs.

  Its underlying modular configuration system is written in the Syno language, and uses packages from Synopkgs.
  The operating system environment and services it provides are configured with the Syno language.

:::

You may quickly encounter Syno language expressions that look very complicated.
As with any programming language, the required amount of Syno language code closely matches the complexity of the problem it is supposed to solve, and reflects how well the problem – and its solution – is understood.
Building software is a complex undertaking, and Syno both *exposes* and *allows managing* this complexity with the Syno language.

Yet, the Syno language itself has only few basic concepts that will be introduced in this tutorial, and which can be combined arbitrarily.
What may look complicated comes not from the language, but from how it is used.

## Overview

This is an introduction to **reading the Syno language**, for the purpose of following other tutorials and examples.

**Using the Syno language** in practice entails multiple things:

- Language: syntax and semantics
- Libraries: `builtins` and `pkgs.lib`
- Developer tools: testing, debugging, linting, formatting, ...
- Generic build mechanisms: `stdenv.mkDerivation`, trivial builders, ...
- Composition and configuration mechanisms: `override`, `overrideAttrs`, overlays, `callPackage`, ...
- Ecosystem-specific packaging mechanisms: `buildGoModule`, `buildPythonApplication`, ...
- SynoPKG module system: `config`, `option`, ...

This tutorial only covers the most important language features, briefly discusses libraries, and at the end will direct you to reference material and resources on the other components.

### What will you learn?

This tutorial should enable you to read typical Syno language code and understand its structure.
Its goal is to highlight where the Syno language may differ from languages you are used to.

It therefore shows the most common and distingushing patterns in the Syno language:

- [Assigning names and accessing values](names-values)
- Declaring and calling [functions](functions)
- [Built-in and library functions](libraries)
- [Impurities](impurities) to obtain build inputs
- [Derivations](derivations) that describe build tasks

:::{important}
This tutorial *does not* explain all Syno language features in detail and *does not* go into specifics of syntactical rules.

See the [Syno manual][manual-language] for a full language reference.
:::

[manual-language]: https://synopkg.github.io/manual/syno/stable/language/index.html

### What do you need?

- Familiarity with software development
- Familiarity with Usyno shell, to read command line examples <!-- TODO: link to yet-to-be instructions on "how to read command line examples" -->
- A {ref}`Syno installation <install-syno>` to run the examples

### How long does it take?

- No experience with functional programming: 2 hours
- Familiar with functional programming: 1 hour
- Proficient with functional programming: 30 minutes

We recommend to run all examples.
Play with them to validate your assumptions and test what you have learned.
Read detailed explanations if you want to make sure you fully understand the examples.

### How to run the examples?

- A piece of Syno language code is a *Syno expression*.
- Evaluating a Syno expression produces a *Syno value*.
- The content of a *Syno file* (file extension `.syno`) is a Syno expression.

:::{note}
To *evaluate* means to transform an expression into a value according to the language rules.
:::

This tutorial contains many examples of Syno expressions.
Each one is followed by the expected evaluation result.

The following example is a Syno expression adding two numbers:

```{code-block} syno
:class: expression
1 + 2
```

```{code-block}
:class: value
3
```

#### Interactive evaluation

Use [`syno repl`] to evaluate Syno expressions interactively (by typing them on the command line):

```shell-session
$ syno repl
Welcome to Syno 2.13.3. Type :? for help.

syno-repl> 1 + 2
3
```

:::{note}
The Syno language uses lazy evaluation, and `syno repl` by default only computes values when needed.

Some examples show a fully evaluated data structure for clarity.
If your output does not match the example, try prepending `:p` to the input expression.

Example:

```shell-session
syno-repl> { a.b.c = 1; }
{ a = { ... }; }

syno-repl> :p { a.b.c = 1; }
{ a = { b = { c = 1; }; }; }
```

Type `:q` to exit [`syno repl`].

:::

[`syno repl`]: https://synopkg.github.io/manual/syno/stable/command-ref/new-cli/syno3-repl.html

#### Evaluating Syno files

Use [`syno-instantiate --eval`][syno-instantiate] to evaluate the expression in a Syno file.

```shell-session
$ echo 1 + 2 > file.syno
$ syno-instantiate --eval file.syno
3
```

:::{dropdown} Detailed explanation

The first command writes `1 + 2` to a file `file.syno` in the current directory.
The contents of `file.syno` are now `1 + 2`, which you can check with

```shell-session
$ cat file.syno
1 + 2
```

The second command runs `syno-instantiate` with the `--eval` option on `file.syno`, which reads the file and evaluates the contained Syno expression.
The resulting value is printed as output.

`--eval` is required to evaluate the file and do nothing else.
If `--eval` is omitted, `syno-instantiate` expects the expression in the given file to evaluate to a special value called a *derivation*, which we will come back to at the end of this tutorial in [](derivations).

:::

:::{note}
`syno-instantiate --eval` will try to read from `default.syno` if no file name is specified.

```shell-session
$ echo 1 + 2 > default.syno
$ syno-instantiate --eval
3
```
:::

:::{note}
The Syno language uses lazy evaluation, and `syno-instantiate` by default only computes values when needed.

Some examples show a fully evaluated data structure for clarity.
If your output does not match the example, try adding the `--strict` option to `syno-instantiate`.

Example:

```shell-session
$ echo "{ a.b.c = 1; }" > file.syno
$ syno-instantiate --eval file.syno
{ a = <CODE>; }
```

```shell-session
$ echo "{ a.b.c = 1; }" > file.syno
$ syno-instantiate --eval --strict file.syno
{ a = { b = { c = 1; }; }; }
```

:::

[syno-instantiate]: https://synopkg.github.io/manual/syno/stable/command-ref/syno-instantiate.html

### Notes on whitespace

White space is used to delimit [lexical tokens], where required.
It is otherwise insignificant.

[lexical tokens]: https://en.m.wikipedia.org/wiki/Lexical_analysis#Lexical_token_and_lexical_tokenization

Line breaks, indentation, and additional spaces are for readers' convenience.

The following are equivalent:

```{code-block} syno
:class: expression
let
 x = 1;
 y = 2;
in x + y
```

```{code-block}
:class: value
3
```

```{code-block} syno
:class: expression
let x=1;y=2;in x+y
```

```{code-block}
:class: value
3
```

(names-values)=
## Names and values

Values in the Syno language can be primitive data types, lists, attribute sets, and functions.

We show examples of primitive data types and lists in the context of [attribute sets](attrset).
Later in this section we cover special features of character strings: [string interpolation](string-interpolation), [file system paths](file-system-paths), and [indented strings](indented-strings).
We deal with [functions](functions) separately.

[Attribute sets](attrset) and [`let` expressions](let) are used to assign names to values.
Assignments are denoted by a single equal sign (`=`).

Whenever you encounter an equal sign (`=`) in Syno language code:
- On its left is the assigned name.
- On its right is the value, delimited by a semicolon (`;`).

(attrset)=
### Attribute set `{ ... }`

An attribute set is a collection of name-value-pairs, where names must be unique.

The following example shows all primitive data types, lists, and attribute sets.

:::{note}
If you are familiar with JSON, imagine the Syno language as *JSON with functions*.

Syno language data types *without functions* work just like their counterparts in JSON and look very similar.
:::


::::{grid} 2

:::{grid-item} **Syno**
```syno
{
  string = "hello";
  integer = 1;
  float = 3.141;
  bool = true;
  null = null;
  list = [ 1 "two" false ];
  attribute-set = {
    a = "hello";
    b = 2;
    c = 2.718;
    d = false;
  }; # comments are supported
}
```
:::

:::{grid-item} **JSON**
```json
{
  "string": "hello",
  "integer": 1,
  "float": 3.141,
  "bool": true,
  "null": null,
  "list": [1, "two", false],
  "object": {
    "a": "hello",
    "b": 1,
    "c": 2.718,
    "d": false
  }
}
```
:::

::::

:::{note}
- Attribute names usually do not need quotes.[^attrnames]
- List elements are separated by white space.[^list-whitespace]
:::

[^attrnames]: Details: [Syno manual - attribute set](https://synopkg.github.io/manual/syno/stable/language/values.html#attribute-set)
[^list-whitespace]: Details: [Syno manual - list](https://synopkg.github.io/manual/syno/stable/language/values.html#list)

(rec-attrset)=
#### Recursive attribute set `rec { ... }`

You will sometimes see attribute sets declared with `rec` prepended.
This allows access to attributes from within the set.

Example:

```{code-block} syno
:class: expression
rec {
  one = 1;
  two = one + 1;
  three = two + 1;
}
```

```{code-block}
:class: value
{ one = 1; three = 3; two = 2; }
```

:::{note}
Elements in an attribute set can be declared in any order, and are ordered on evaluation.
:::

Counter-example:

```{code-block} syno
:class: expression
{
  one = 1;
  two = one + 1;
  three = two + 1;
}
```

```{code-block}
:class: value
error: undefined variable 'one'

       at «string»:3:9:

            2|   one = 1;
            3|   two = one + 1;
             |         ^
            4|   three = two + 1;
```

(let)=
### `let ... in ...`

Also known as “`let` expression” or “`let` binding”

`let` expressions allow assigning names to values for repeated use.

Example:

```{code-block} syno
:class: expression
let
  a = 1;
in
a + a
```

```{code-block}
:class: value
2
```

:::{dropdown} Detailed explanation

Assignments are placed between the keywords `let` and `in`.
In this example we assign `a = 1`.

After `in` comes the expression in which the assignments are valid, i.e., where assigned names can be used.
In this example the expression is `a + a`, where `a` refers to `a = 1`.

By replacing the names with their assigned values, `a + a` evaluates to `2`.

:::

Names can be assigned in any order, and expressions on the right of the assignment (`=`) can refer to other assigned names.

Example:

```{code-block} syno
:class: expression
let
  b = a + 1;
  a = 1;
in
a + b
```

```{code-block}
:class: value
3
```

:::::{dropdown} Detailed explanation

Assignments are placed between the keywords `let` and `in`.
In this example we assign `a = 1` and `b = a + 1`.

The order of assignments does not matter.
Therefore the following example, where the assignments are in reverse order, is equivalent:


```{code-block} syno
:class: expression
let
  a = 1;
  b = a + 1;
in
a + b
```

```{code-block}
:class: value
3
```

Note that the `a` in `b = a + 1` refers to `a = 1`.

After `in` comes the expression in which the assignments are valid.
In this example the expression is `a + b`, where `a` refers to `a = 1`, and `b` refers to `b = a + 1`.

By replacing the names with their assigned values, `a + b` evaluates to `3`.

This is similar to [recursive attribute sets](rec-attrset):
in both, the order of assignments does not matter, and names on the left can be used in expressions on the right of the assignment (`=`).

Example:

::::{grid} 2

:::{grid-item} `let ... in ...`

```{code-block} syno
:class: expression
let
  b = a + 1;
  c = a + b;
  a = 1;
in {  c = c; a = a; b = b; }
```

```{code-block}
:class: value
{ a = 1; b = 2; c = 3; }
```

:::

:::{grid-item} `rec { ... }`

```{code-block} syno
:class: expression
rec {
  b = a + 1;
  c = a + b;
  a = 1;
}
```

```{code-block}
:class: value
{ a = 1; b = 2; c = 3; }
```

:::

::::

The difference is that while a recursive attribute set evaluates to an [attribute set](attrset), any expression can follow after the `in` keyword.

In the following example we use the `let` expression to form a list:

```{code-block} syno
:class: expression
let
  b = a + 1;
  c = a + b;
  a = 1;
in [ a b c ]
```

```{code-block}
:class: value
[ 1 2 3 ]
```

:::::

Only expressions within the `let` expression itself can access the newly declared names.
We say: the bindings have local scope.

Counter-example:

```{code-block} syno
:class: expression
{
  a = let x = 1; in x;
  b = x;
}
```

```{code-block}
:class: value
error: undefined variable 'x'

       at «string»:3:7:

            2|   a = let x = 1; in x;
            3|   b = x;
             |       ^
            4| }
```

<!-- TODO: exercise - use let to reuse a value in an attribute set -->

### Attribute access

Attributes in a set are accessed with a dot (`.`) and the attribute name.

Example:

```{code-block} syno
:class: expression
let
  attrset = { x = 1; };
in
attrset.x
```

```{code-block}
:class: value
1
```

Accessing nested attributes works the same way.

Example:

```{code-block} syno
:class: expression
let
  attrset = { a = { b = { c = 1; }; }; };
in
attrset.a.b.c
```

```{code-block}
:class: value
1
```

The dot (`.`) notation can also be used for assigning attributes.

Example:

```{code-block} syno
:class: expression
{ a.b.c = 1; }
```

```{code-block}
:class: value
{ a = { b = { c = 1; }; }; }
```

(with)=
### `with ...; ...`

The `with` expression allows access to attributes without repeatedly referencing their attribute set.

Example:

```{code-block} syno
:class: expression
let
  a = {
    x = 1;
    y = 2;
    z = 3;
  };
in
with a; [ x y z ]
```

```{code-block}
:class: value
[ 1 2 3 ]
```

The expression

```{code-block} syno
with a; [ x y z ]
```

is equivalent to

```{code-block} syno
[ a.x a.y a.z ]
```

Attributes made available through `with` are only in scope of the expression following the semicolon (`;`).

Counter-example:

```{code-block} syno
:class: expression
let
  a = {
    x = 1;
    y = 2;
    z = 3;
  };
in
{
  b = with a; [ x y z ];
  c = x;
}
```

```{code-block}
:class: value
error: undefined variable 'x'

       at «string»:10:7:

            9|   b = with a; [ x y z ];
           10|   c = x;
             |       ^
           11| }
```

(inherit)=
### `inherit ...`

`inherit` is shorthand for assigning the value of a name from an existing scope to the same name in a nested scope.
It is for convenience to avoid repeating the same name multiple times.

Example:

```{code-block} syno
:class: expression
let
  x = 1;
  y = 2;
in
{
  inherit x y;
}
```

```{code-block}
:class: value
{ x = 1; y = 2; }
```

The fragment

```{code-block} syno
inherit x y;
```
is equivalent to

```{code-block} syno
x = x; y = y;
```

It is also possible to `inherit` names from a specific attribute set with parentheses (`inherit (...) ...`).

Example:

```{code-block} syno
:class: expression
let
  a = { x = 1; y = 2; };
in
{
  inherit (a) x y;
}
```

```{code-block}
:class: value
{ x = 1; y = 2; }
```

The fragment

```{code-block} syno
inherit (a) x y;
```

is equivalent to

```{code-block} syno
x = a.x; y = a.y;
```

`inherit` also works inside `let` expressions.

Example:

```{code-block} syno
:class: expression
let
  inherit ({ x = 1; y = 2; }) x y;
in [ x y ]
```

```{code-block}
:class: value
[ 1 2 ]
```

:::{dropdown} Detailed explanation

While this example is contrived, in more complex code you will regularly see nested `let` expressions that re-use names from their outer scope.

Here we use the attribute set `{ x = 1; y = 2; }` to have something non-trivial to inherit from.
The `let` expression inherits `x` and `y` from that attribute set using `( )`, which is equivalent to writing:

```{code-block} syno
let
  x = { x = 1; y = 2; }.x;
  y = { x = 1; y = 2; }.y;
in
```

The new inner scope now contains `x` and `y`, which are used in the list `[ x y ]`.

:::

(string-interpolation)=
### String Interpolation `${ ... }`

Previously known as “antiquotation”.

The value of a Syno expression can be inserted into a character string with the dollar-sign and braces (`${ }`).

Example:

```{code-block} syno
:class: expression
let
  name = "Syno";
in
"hello ${name}"
```

```{code-block}
:class: value
"hello Syno"
```

Only character strings or values that can be represented as a character string are allowed.

Counter-example:

```{code-block} syno
:class: expression
let
  x = 1;
in
"${x} + ${x} = ${x + x}"
```

```{code-block}
:class: value
error: cannot coerce an integer to a string

       at «string»:4:2:

            3| in
            4| "${x} + ${x} = ${x + x}"
             |  ^
            5|
```

Interpolated expressions can be arbitrarily nested.

(This can become hard to read, and we recommend to avoid it in practice.)

Example:

```{code-block} syno
:class: expression
let
  a = "no";
in
"${a + " ${a + " ${a}"}"}"
```

```{code-block}
:class: value
"no no no"
```


:::{warning}
You may encounter strings that use the dollar sign (`$`) before an assigned name, but no braces (`{ }`):

These are *not* interpolated strings, but usually denote variables in a shell script.

In such cases, the use of names from the surrounding Syno expression is a coincidence.

Example:

```{code-block} syno
:class: expression
let
  out = "Syno";
in
"echo ${out} > $out"
```

```{code-block}
:class: value
"echo Syno > $out"
```
:::

<!-- TODO: link to escaping rules -->

(file-system-paths)=
### File system paths

The Syno language offers convenience syntax for file system paths.

Absolute paths always start with a slash (`/`).

Example:

```{code-block} syno
:class: expression
/absolute/path
```

```{code-block}
:class: value
/absolute/path
```

Paths are relative when they contain at least one slash (`/`) but do not start with one.
They evaluate to the path relative to the file containing the expression.

The following examples assume the containing Syno file is in `/current/directory` (or `syno repl` is run in `/current/directory`).

Example:


```{code-block} syno
:class: expression
./relative
```

```{code-block}
:class: value
/current/directory/relative
```

Example:

```{code-block} syno
:class: expression
relative/path
```

```{code-block}
:class: value
/current/directory/relative/path
```

One dot (`.`) denotes the current directory within the given path.

You will often see the following expression, which specifies a Syno file's directory.

Example:

```{code-block} syno
:class: expression
./.
```

```{code-block}
:class: value
/current/directory
```

:::{dropdown} Detailed explanation

Since relative paths must contain a slash (`/`) but must not start with one, and the dot (`.`) denotes no change of directory, the combination `./.` specifies the current directory as a relative path.

:::

Two dots (`..`) denote the parent directory.

Example:

```{code-block} syno
:class: expression
../.
```

```{code-block}
:class: value
/current
```
:::{note}
Paths can be used in interpolated expressions – an [impure operation](impurities) we will cover in detail in a [later section](path-impurities).
:::

(search-path-tutorial)=
#### Search path

Also known as “angle bracket syntax”.

Example:

```{code-block} syno
:class: expression
<synopkgs>
```

```{code-block}
:class: value
/syno/var/syno/profiles/per-user/root/channels/synopkgs
```

The value of a named path is a file system path that depends on the contents of the [`$SYNO_PATH`][SYNO_PATH] environment variable.

In practice, `<synopkgs>` points to the file system path of some revision of [`synopkgs`][synopkgs], the source repository of Synopkgs.

For example, `<synopkgs/lib>` points to the subdirectory `lib` of that file system path:

```{code-block} syno
:class: expression
<synopkgs/lib>
```

```{code-block}
:class: value
/syno/var/syno/profiles/per-user/root/channels/synopkgs/lib
```

While you will see many such examples, we recommend to [avoid search paths](search-path) in practice, as they are [impurities](impurities) which are not reproducible.

[SYNO_PATH]: https://synopkg.github.io/manual/syno/unstable/command-ref/env-common.html?highlight=syno_path#env-SYNO_PATH
[synopkgs]: https://github.com/SynoPKG/synopkgs
[manual-primitives]: https://synopkg.github.io/manual/syno/stable/language/values.html#primitives

(indented-strings)=
### Indented strings

Also known as “multi-line strings”.

The Syno language offers convenience syntax for character strings which span multiple lines that have common indentation.

Indented strings are denoted by *double single quotes* (`'' ''`).

Example:

```{code-block} syno
:class: expression
''
multi
line
string
''
```

```{code-block}
:class: value
"multi\nline\nstring\n"
```

Equal amounts of prepended white space are trimmed from the result.

Example:

```{code-block} syno
:class: expression
''
  one
   two
    three
''
```

```{code-block}
:class: value
"one\n two\n  three\n"
```

<!-- TODO: See [escaping rules](). -->

(functions)=
## Functions

Functions are everywhere in the Syno language and deserve particular attention.

A function always takes exactly one argument.
Argument and function body are separated by a colon (`:`).

Wherever you see a colon (`:`) in Syno language code:
- On its left is the function argument
- On its right is the function body.

Function arguments are the third way, apart from [attribute sets](attrset) and [`let` expressions](let), to assign names to values.
Notably, values are not known in advance: the names are used as placeholders that are filled when [calling a function](calling-functions).

Function declarations in the Syno language can appear in different forms.
Each of them is explained in the following, and here is an overview:

- Single argument

  ```{code-block} syno
  x: x + 1
  ```

  - Multiple arguments via nesting

    ```{code-block} syno
    x: y: x + y
    ```

- Attribute set argument

  ```{code-block} syno
  { a, b }: a + b
  ```

  - With default attributes

    ```{code-block} syno
    { a, b ? 0 }: a + b
    ```

  - With additional attributes allowed

    ```{code-block} syno
    { a, b, ...}: a + b
    ```

- Named attribute set argument

  ```{code-block} syno
  args@{ a, b, ... }: a + b + args.c
  ```

  or

  ```{code-block} syno
  { a, b, ... }@args: a + b + args.c
  ```

Functions have no names.
We say they are anonymous, and call such a function a *lambda*.[^lambda]

[^lambda]: The term *lambda* is a shorthand for [lambda abstraction](https://en.wikipedia.org/wiki/Lambda_calculus#lambdaAbstr) in the [lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus).

Example:

```{code-block} syno
:class: expression
x: x + 1
```

```{code-block} syno
:class: value
<LAMBDA>
```

The `<LAMBDA>` indicates the resulting value is an anonymous function.

As with any other value, functions can be assigned to a name.

Example:

```{code-block} syno
:class: expression
let
  f = x: x + 1;
in f
```

```{code-block} syno
:class: value
<LAMBDA>
```

(calling-functions)=
### Calling functions

Also known as "function application".

Calling a function with an argument means writing the argument after the function.

Example:

```{code-block} syno
:class: expression
let
  f = x: x + 1;
in f 1
```

```{code-block}
:class: value
2
```

Example:

```{code-block} syno
:class: expression
let
  f = x: x.a;
in
f { a = 1; }
```

```{code-block}
:class: value
1
```

The above example calls `f` on a literal attribute set.
One can also pass arguments by name.

Example:

```{code-block} syno
:class: expression
let
  f = x: x.a;
  v = { a = 1; };
in
f v
```

```{code-block}
:class: value
1
```

Since function and argument are separated by white space, sometimes parentheses (`( )`) are required to achieve the desired result.

Example:

```{code-block} syno
:class: expression
(x: x + 1) 1
```

```{code-block}
:class: value
2
```

:::{dropdown} Detailed explanation

This expression applies an anonymous function `x: x + 1` to the argument `1`.
The function has to be written in parentheses to distinguish it from the argument.

:::

Example:

List elements are also separated by white space, therefore the following are different:

```{code-block} syno
:class: expression
let
 f = x: x + 1;
 a = 1;
in [ (f a) ]
```

```{code-block} syno
:class: value
[ 2 ]
```

```{code-block} syno
:class: expression
let
 f = x: x + 1;
 a = 1;
in [ f a ]
```

```{code-block}
:class: value
[ <LAMBDA> 1 ]
```

The first example reads: apply `f` to `a`, and put the result in a list.
The resulting list has one element.

The second example reads: put `f` and `a` in a list.
The resulting list has two elements.

#### Multiple arguments

Also known as “[curried] functions”.

Syno functions take exactly one argument.
Multiple arguments can be handled by nesting functions.

Such a nested function can be used like a function that takes multiple arguments, but offers additional flexibility.

[curried]: https://en.m.wikipedia.org/wiki/Currying

Example:

```{code-block} syno
:class: expression
x: y: x + y
```

```{code-block}
:class: value
<LAMBDA>
```

The above function is equivalent to

```{code-block} syno
:class: expression
x: (y: x + y)
```

```{code-block}
:class: value
<LAMBDA>
```

This function takes one argument and returns another function `y: x + y` with `x` set to the value of that argument.

Example:

```{code-block} syno
:class: expression
let
  f = x: y: x + y;
in
f 1
```

```{code-block}
:class: value
<LAMBDA>
```

Applying the function which results from `f 1` to another argument yields the inner body `x + y` (with `x` set to `1` and `y` set to the other argument), which can now be fully evaluated.

```{code-block} syno
:class: expression
let
  f = x: y: x + y;
in
f 1 2
```

```{code-block}
:class: value
3
```

<!-- TODO: exercise - assign the lambda a name and do something with it -->

### Attribute set argument

Also known as “keyword arguments” or “destructuring” .

Syno functions can be declared to require an attribute set with specific structure as argument.

This is denoted by listing the expected attribute names separated by commas (`,`) and enclosed in braces (`{ }`).

Example:

```{code-block} syno
:class: expression
{a, b}: a + b
```

```{code-block} syno
:class: value
<LAMBDA>
```

The argument defines the exact attributes that have to be in that set.
Leaving out or passing additional attributes is an error.

Example:

```{code-block} syno
:class: expression
let
  f = {a, b}: a + b;
in
f { a = 1; b = 2; }
```

```{code-block} syno
:class: value
3
```

Counter-example:

```{code-block} syno
:class: expression
let
  f = {a, b}: a + b;
in
f { a = 1; b = 2; c = 3; }
```

```{code-block}
:class: value
error: 'f' at (string):2:7 called with unexpected argument 'c'

       at «string»:4:1:

            3| in
            4| f { a = 1; b = 2; c = 3; }
             | ^
            5|
```

<!-- TODO: not the same as x: x.a + x.b (!!!!) -->

#### Default values

Also known as “default arguments”.

Destructured arguments can have default values for attributes.

This is denoted by separating the attribute name and its default value with a question mark (`?`).

Attributes in the argument are not required if they have a default value.

Example:

```{code-block} syno
:class: expression
let
  f = {a, b ? 0}: a + b;
in
f { a = 1; }
```

```{code-block}
:class: value
1
```

Example:

```{code-block} syno
:class: expression
let
  f = {a ? 0, b ? 0}: a + b;
in
f { } # empty attribute set
```

```{code-block}
:class: value
0
```

#### Additional attributes

Additional attributes are allowed with an ellipsis (`...`):

```{code-block} syno
{a, b, ...}: a + b
```

Unlike in the previous counter-example, passing an argument that contains additional attributes is not an error.

Example:

```{code-block} syno
:class: expression
let
  f = {a, b, ...}: a + b;
in
f { a = 1; b = 2; c = 3; }
```

```{code-block}
:class: value
3
```

### Named attribute set argument

Also known as “@ pattern”, “@ syntax”, or “‘at’ syntax”.

An attribute set argument can be given a name to be accessible as a whole.

This is denoted by prepending or appending the name to the attribute set argument, separated by the at sign (`@`).

Example:

```{code-block} syno
:class: expression
{a, b, ...}@args: a + b + args.c
```

```{code-block}
:class: value
<LAMBDA>
```

or

```{code-block} syno
:class: expression
args@{a, b, ...}: a + b + args.c
```

```{code-block}
:class: value
<LAMBDA>
```

Example:

```{code-block} syno
:class: expression
let
  f = {a, b, ...}@args: a + b + args.c;
in
f { a = 1; b = 2; c = 3; }
```

```{code-block} syno
:class: value
6
```

(libraries)=
## Function libraries

In addition to the [built-in operators][operators] (`+`, `==`, `&&`, etc.), there are two widely used libraries that *together* can be considered standard for the Syno language.
You need to know about both to understand and navigate Syno language code.

<!-- TODO: find a place for operators -->

We recommend to at least skim them to familiarise yourself with what is available.

[operators]: https://synopkg.github.io/manual/syno/stable/language/operators.html

(builtins)=
### `builtins`

Also known as “primitive operations” or “primops”.

Syno comes with many functions that are built into the language.
They are implemented in C++ as part of the Syno language interpreter.

:::{note}
The Syno manual lists all [Built-in Functions][syno-builtins], and shows how to use them.
:::

These functions are available under the `builtins` constant.

Example:

```{code-block} syno
:class: expression
builtins.toString
```

```{code-block}
:class: value
<PRIMOP>
```

[syno-operators]: https://synopkg.github.io/manual/syno/unstable/language/operators.html
[syno-builtins]: https://synopkg.github.io/manual/syno/stable/language/builtins.html

#### `import`

Most built-in functions are only accessible through `builtins`.
A notable exception is `import`, which is also available at the top level.

`import` takes a path to a Syno file, reads it to evaluate the contained Syno expression, and returns the resulting value.
If the path points to a directory, the file `default.syno` in that directory is used instead.

Example:

```shell-session
$ echo 1 + 2 > file.syno
```

```{code-block} syno
:class: expression
import ./file.syno
```

```{code-block}
:class: value
3
```

:::{dropdown} Detailed explanation

The preceding shell command writes the contents `1 + 2` to the file `file.syno` in the current directory.

The above Syno expression refers to this file as `./file.syno`.
`import` reads the file and evaluates to the contained Syno expression.

It is an error if the file system path does not exist.

After reading `file.syno` the Syno expression is equivalent to the file contents:

```{code-block} syno
:class: expression
1 + 2
```

```{code-block}
:class: value
3
```
:::

Since a Syno file can contain any Syno expression, `import`ed functions can be applied to arguments immediately.

That is, whenever you see additional tokens after a call to `import`, the value it returns should be a function, and anything that follows are arguments to that function.

Example:

```shell-session
$ echo "x: x + 1" > file.syno
```

```{code-block} syno
:class: expression
import ./file.syno 1
```

```{code-block}
:class: value
2
```

::::{dropdown} Detailed explanation

The preceding shell command writes the contents `x: x + 1` to the file `file.syno` in the current directory.

The above Syno expression refers to this file as `./file.syno`.
`import ./file.syno` reads the file and evaluates to the contained Syno expression.

It is an error if the file system path does not exist.

After reading the file, the Syno expression `import ./file.syno` is equivalent to the file contents:

```{code-block} syno
:class: expression
(x: x + 1) 1
```

```{code-block}
:class: value
2
```

This applies the function `x: x + 1` to the argument `1`, and therefore evaluates to `2`.

:::{note}
Parentheses are required to separate function declaration from function application.
:::

::::

(pkgs-lib)=
### `pkgs.lib`

The [`synopkgs`][synopkgs] repository contains an attribute set called [`lib`][synopkgs-lib], which provides a large number of useful functions.
They are implemented in the Syno language, as opposed to [`builtins`](builtins), which are part of the language itself.

:::{note}
The Synopkgs manual lists all [Synopkgs library functions][synopkgs-functions].
:::

[synopkgs-functions]: https://synopkg.github.io/manual/synopkgs/stable/#sec-functions-library
[synopkgs-lib]: https://github.com/SynoPKG/synopkgs/blob/master/lib/default.syno

These functions are usually accessed through `pkgs.lib`, as the Synopkgs attribute set is given the name `pkgs` by convention.

Example:

```{code-block} syno
:class: expression
let
  pkgs = import <synopkgs> {};
in
pkgs.lib.strings.toUpper "search paths considered harmful"
```

```{code-block}
:class: value
SEARCH PATHS CONSIDERED HARMFUL
```

:::{dropdown} Detailed explanation

This is a more complex example, but by now you should be familiar with all its components.

The name `pkgs` is declared to be the expression `import`ed from some file.
That file's path is determined by the value of the search path `<synopkgs>`, which in turn is determined by the `$SYNO_PATH` environment variable at the time this expression is evaluated.
As this expression happens to be a function, it requires an argument to evaluate, and in this case passing an empty attribute set `{}` is sufficient.

Now that `pkgs` is in scope of `let ... in ...`, its attributes can be accessed.
From the Synopkgs manual one can determine that there exists a function under [`lib.strings.toUpper`].

[`lib.strings.toUpper`]: https://synopkg.github.io/manual/synopkgs/stable/#function-library-lib.strings.toUpper

For brevity, this example uses a search path to obtain *some version* of Synopkgs.
The function `toUpper` is trivial enough that we can expect it not to produce different results for different versions of Synopkgs.
Yet, more sophisticated software is likely to suffer from such problems.
A fully reproducible example would therefore look like this:

```{code-block} syno
:class: expression
let
  synopkgs = fetchTarball https://github.com/SynoPKG/synopkgs/archive/06278c77b5d162e62df170fec307e83f1812d94b.tar.gz;
  pkgs = import synopkgs {};
in
pkgs.lib.strings.toUpper "always pin your sources"
```

```{code-block}
:class: value
ALWAYS PIN YOUR SOURCES
```

See [](pinning-synopkgs) for details.

What you will also often see is that `pkgs` is passed as an argument to a function.
By convention one can assume that it refers to the Synopkgs attribute set, which has a `lib` attribute:

```{code-block} syno
:class: expression
{ pkgs, ... }:
pkgs.lib.strings.removePrefix "no " "no true scotsman"
```

```{code-block}
:class: value
<LAMBDA>
```

To make this function produce a result, you can write it to a file (e.g. `file.syno`) and pass it an argument through `syno-instantiate`:

```shell-session
$ syno-instantiate --eval file.syno --arg pkgs 'import <synopkgs> {}'
"true scotsman"
```

Oftentimes you will see in SynoPKG configurations, and also within Synopkgs, that `lib` is passed directly.
In that case one can assume that this `lib` is equivalent to `pkgs.lib` where only `pkgs` is available.

Example:

```{code-block} syno
:class: expression
{ lib, ... }:
let
  to-be = true;
in
lib.trivial.or to-be (! to-be)
```

```{code-block}
:class: value
<LAMBDA>
```

To make this function produce a result, you can write it to a file (e.g. `file.syno`) and pass it an argument through `syno-instantiate`:

```shell-session
$ syno-instantiate --eval file.syno --arg lib '(import <synopkgs> {}).lib'
true
```

Sometimes both `pkgs` and `lib` are passed as arguments.
In that case, one can assume `pkgs.lib` and `lib` to be equivalent.
This is done to improve readability by avoiding repeated use of `pkgs.lib`.

Example:

```{code-block} syno
{ pkgs, lib, ... }:
# ... multiple uses of `pkgs`
# ... multiple uses of `lib`
```

:::

For historical reasons, some of the functions in `pkgs.lib` are equivalent to [`builtins`](builtins) of the same name.

(impurities)=
## Impurities

So far we have only covered what we call *pure expressions*:
declaring data and transforming it with functions.

In practice, describing derivations requires observing the outside world.

There is only one impurity in the Syno language that is relevant here:
reading files from the file system as *build inputs*

Build inputs are files that derivations refer to in order to describe how to derive new files.
When run, a derivation will only have access to explicitly declared build inputs.

The only way to specify build inputs in the Syno language is explicitly with:

- File system paths
- Dedicated functions.

Syno and the Syno language refer to files by their content hash. If file contents are not known in advance, it's unavoidable to read files during expression evaluation.

:::{note}
Syno supports other types of impure expressions, such as [search paths](search-path) or the constant [`builtins.currentSystem`](https://synopkg.github.io/manual/syno/stable/language/builtin-constants.html#builtins-currentSystem).
We do not cover those here in more detail, as they do not matter for how the Syno language works in principle, and because they are discouraged for the very reason of breaking reproducibility.
:::

(path-impurities)=
### Paths

Whenever a file system path is used in [string interpolation](string-interpolation), the contents of that file are copied to a special location in the file system, the *Syno store*, as a side effect.

The evaluated string then contains the Syno store path assigned to that file.

<!-- TODO: link to explanation of the Syno store -->

Example:

```shell-session
$ echo 123 > data
```

```{code-block} syno
:class: expression
"${./data}"
```

```{code-block}
:class: value
"/syno/store/h1qj5h5n05b5dl5q4nldrqq8mdg7dhqk-data"
```

:::{dropdown} Detailed explanation

The preceding shell command writes the characters `123` to the file `data` in the current directory.

The above Syno expression refers to this file as `./data` and converts the file system path to an [interpolated string](string-interpolation) `${ ... }`.

Such interpolated expressions must evaluate to something that can be represented as a character string.
A file system path is such a value, and its character string representation is the corresponding Syno store path:

```{code-block}
/syno/store/<hash>-<name>
```

The Syno store path is obtained by taking the hash of the file's contents (`<hash>`) and combining it with the file name (`<name>`).
The file is copied into the Syno store directory `/syno/store` as a side effect of evaluation.
It is an error if the file system path does not exist.

:::

For directories the same thing happens: The entire directory (including nested files and directories) is copied to the Syno store, and the evaluated string becomes the Syno store path of the directory.

### Fetchers

Files to be used as build inputs do not have to come from the file system.

The Syno language provides built-in impure functions to fetch files over the network during evaluation:

- [builtins.fetchurl][fetchurl]
- [builtins.fetchTarball][fetchTarball]
- [builtins.fetchGit][fetchGit]
- [builtins.fetchClosure][fetchClosure]

These functions evaluate to a file system path in the Syno store.

Example:

```{code-block} syno
:class: expression
builtins.fetchurl "https://github.com/SynoPKG/syno/archive/7c3ab5751568a0bc63430b33a5169c5e4784a0ff.tar.gz"
```

```{code-block}
:class: value
"/syno/store/7dhgs330clj36384akg86140fqkgh8zf-7c3ab5751568a0bc63430b33a5169c5e4784a0ff.tar.gz"
```

Some of them add extra convenience, such as automatically unpacking archives.

Example:

```{code-block} syno
:class: expression
builtins.fetchTarball "https://github.com/SynoPKG/syno/archive/7c3ab5751568a0bc63430b33a5169c5e4784a0ff.tar.gz"
```

```{code-block}
:class: value
"/syno/store/d59llm96vgis5fy231x6m7nrijs0ww36-source"
```

:::{note}
The Synopkgs manual on [Fetchers][synopkgs-fetchers] lists numerous additional library functions to fetch files over the network.
:::

It is an error if the network request fails.

[fetchurl]: https://synopkg.github.io/manual/syno/stable/language/builtins.html#builtins-fetchurl
[fetchTarball]: https://synopkg.github.io/manual/syno/stable/language/builtins.html#builtins-fetchTarball
[fetchGit]: https://synopkg.github.io/manual/syno/stable/language/builtins.html#builtins-fetchGit
[fetchClosure]: https://synopkg.github.io/manual/syno/stable/language/builtins.html#builtins-fetchClosure
[synopkgs-fetchers]: https://synopkg.github.io/manual/synopkgs/stable/#chap-pkgs-fetchers

(derivations)=
## Derivations


Derivations are at the core of both Syno and the Syno language:
- The Syno language is used to describe derivations.
- Syno runs derivations to produce *build results*.
- Build results can in turn be used as inputs for other derivations.

The Syno language primitive to declare a derivation is the built-in impure function `derivation`.

It is usually wrapped by the Synopkgs build mechanism `stdenv.mkDerivation`, which hides much of the complexity involved in non-trivial build procedures.

:::{note}
You will probably never encounter `derivation` in practice.
:::

Whenever you see `mkDerivation`, it denotes something that Syno will eventually *build*.

Example: [a package using `mkDerivation`](mkDerivation-example)

The evaluation result of `derivation` (and `mkDerivation`) is an [attribute set](attrset) with a certain structure and a special property:
It can be used in [string interpolation](string-interpolation), and in that case evaluates to the Syno store path of its build result.

Example:

```{code-block} syno
:class: expression
let
  pkgs = import <synopkgs> {};
in "${pkgs.syno}"
```

```{code-block}
:class: value
"/syno/store/sv2srrjddrp2isghmrla8s6lazbzmikd-syno-2.11.0"
```

:::{note}
Your output may differ.
It may produce a different hash or even a different package version.

A derivation's output path is fully determined by its inputs, which in this case come from *some* version of Synopkgs.

This is why we recommend to [avoid search paths](search-path) to ensure predictable outcomes, except in examples intended for illustration only.
:::

:::{dropdown} Detailed explanation

The example imports the Syno expression from the search path `<synopkgs>`, and applies the resulting function to an empty attribute set `{}`.
Its output is assigned the name `pkgs`.

Converting the attribute `pkgs.syno` to a string with [string interpolation](string-interpolation) is allowed, as `pkgs.syno` is a derivation.
That is, ultimately `pkgs.syno` boils down to a call to `derivation`.

The resulting string is the file system path where the build result of that derivation will end up.

There is more depth to the inner workings of derivations, but at this point it should be enough to know that such expressions evaluate to Syno store paths.

:::

String interpolation on derivations is used to refer to their build results as file system paths when declaring new derivations.

This allows constructing arbitrarily complex compositions of derivations with the Syno language.

## Worked examples

So far we have seen artifical examples illustrating the various constructs in the Syno language.

You should now be able to read Syno language code for simple packages and configurations, and come up with similiar explanations of the following practical examples.

::: {note}
The goal of the following exercises is not to understand what the code means or how it works, but how it is structured in terms of functions, attribute sets, and other Syno language data types.
:::

### Shell environment

```{code-block} syno
{ pkgs ? import <synopkgs> {} }:
let
  message = "hello world";
in
pkgs.mkShell {
  buildInputs = with pkgs; [ cowsay ];
  shellHook = ''
    cowsay ${message}
  '';
}
```

This example declares a shell environment (which runs the `shellHook` on initialization).

Explanation:

- This expression is a function that takes an attribute set as an argument.
- If the argument has the attribute `pkgs`, it will be used in the function body.
  Otherwise, by default, import the Syno expression in the file found on the search path `<synopkgs>` (which is a function in this case), call the function with an empty attribute set, and use the resulting value.
- The name `message` is bound to the string value `"hello world"`.
- The attribute `mkShell` of the `pkgs` set is a function that is passed an attribute set as argument.
  Its return value is also the result of the outer function.
- The attribute set passed to `mkShell` has the attributes `buildInputs` (set to a list with one element: the `cowsay` attribute from `pkgs`) and `shellHook` (set to an indented string).
- The indented string contains an interpolated expression, which will expand the value of `message` to yield `"hello world"`.


### SynoPKG configuration

```{code-block} syno
{ config, pkgs, ... }: {

  imports = [ ./hardware-configuration.syno ];

  environment.systemPackages = with pkgs; [ git ];

  # ...

}
```

This example is (part of) a SynoPKG configuration.

Explanation:

- This expression is a function that takes an attribute set as an argument.
  It returns an attribute set.
- The argument must at least have the attributes `config` and `pkgs`, and may have more attributes.
- The returned attribute set contains the attributes `imports` and `environment`.
- `imports` is a list with one element: a path to a file next to this Syno file, called `hardware-configuration.syno`.

  :::{note}
  `imports` is not the impure built-in `import`, but a regular attribute name!
  :::
- `environment` is itself an attribute set with one attribute `systemPackages`, which will evaluate to a list with one element: the `git` attribute from the `pkgs` set.
- The `config` argument is not (shown to be) used.

(mkDerivation-example)=
### Package

```{code-block} syno
{ lib, stdenv, fetchurl }:

stdenv.mkDerivation rec {

  pname = "hello";

  version = "2.12";

  src = fetchurl {
    url = "mirror://gnu/${pname}/${pname}-${version}.tar.gz";
    sha256 = "1ayhp9v4m4rdhjmnl2bq3cibrbqqkgjbl3s7yk2nhlh8vj3ay16g";
  };

  meta = with lib; {
    license = licenses.gpl3Plus;
  };

}
```

This example is a (simplified) package declaration from Synopkgs.

Explanation:

- This expression is a function that takes an attribute set which must have exactly the attributes `lib`, `stdenv`, and `fetchurl`.
- It returns the result of evaluating the function `mkDerivation`, which is an attribute of `stdenv`, applied to a recursive set.
- The recursive set passed to `mkDerivation` uses its own `pname` and `version` attributes in the argument to the function `fetchurl`.
  `fetchurl` itself comes from the outer function's arguments.
- The `meta` attribute is itself an attribute set, where the `license` attribute has the value that was assigned to the nested attribute `lib.licenses.gpl3Plus`.

## References

- [Syno manual: Syno language][manual-language]
- [Syno manual: String interpolation][manual-string-interpolation]
- [Syno manual: Built-in Functions][syno-builtins]
- [Syno manual: `syno repl`][`syno repl`]
- [Synopkgs manual: Functions reference][synopkgs-functions]
- [Synopkgs manual: Fetchers][synopkgs-fetchers]

[manual-string-interpolation]: https://synopkg.github.io/manual/syno/stable/language/string-interpolation.html

## Next steps

### Get things done

- [](declarative-reproducible-envs) – create reproducible shell environments from a Syno file
- [Garbage Collection](https://synopkg.github.io/manual/syno/stable/package-management/garbage-collection.html) – remove unused build results from the Syno store

### Learn more

If you worked through the examples, you will have noticed that reading the Syno language reveals the structure of the code, but does not necessarily tell what the code actually means.

Often it is not possible to determine from the code at hand
- the data type of a named value or function argument.
- the data type a called function accepts for its argument.
- which attributes are present in a given attribute set.

Example:

```{code-block} syno
{ x, y, z }: (x y) z.a
```

How do we know...
- that `x` will be a function that, given an argument, returns a function?
- that, given `x` is a function, `y` will be an appropriate argument to `x`?
- that, given `(x y)` is a function, `z.a` will be an appropriate argument to `(x y)`?
- that `z` will be an attribute set at all?
- that, given `z` is an attribute set, it will have an attribute `a`?
- which data type `y` and `z.a` will be?
- the data type of the end result?

And how does the caller of this function know that it requires an attribute set with attributes `x`, `y`, `z`?

Answering such questions requires knowing the context in which a given expression is supposed to be used.

The Syno ecosystem and code style is driven by conventions.
Most names you will encounter in Syno language code come from Synopkgs:

- [Syno Pills][syno-pills] - a detailed explanation of derivations and how Synopkgs is constructed from first principles

Synopkgs provides generic build mechanisms that are widely used:

- [`stdenv`][stdenv] - most importantly `mkDerivation`
- [Trivial Builders][trivial-builders] - to create files and shell scripts

Packages from Synopkgs can be modified through multiple mechanisms:

- [overrides] – specifically `override` and `overrideAttrs` to modify single packages
- [overlays] – to produce a custom variant of Synopkgs with individually modified packages

Different language ecosystems and frameworks have different requirements to accommodating them into Synopkgs:

- [Languages and frameworks][language-support] lists tools provided by Synopkgs to build language- or framework-specific packages with Syno.

The SynoPKG Linux distribution has a modular configuration system that imposes its own conventions:

- [SynoPKG modules][synopkg-modules] shows how SynoPKG configurations are organized.

[syno-pills]: https://synopkg.github.io/guides/syno-pills/
[stdenv]: https://synopkg.github.io/manual/synopkgs/stable/#chap-stdenv
[trivial-builders]: https://synopkg.github.io/manual/synopkgs/stable/#chap-trivial-builders
[overlays]: https://synopkg.github.io/manual/synopkgs/stable/#chap-overlays
[overrides]: https://synopkg.github.io/manual/synopkgs/stable/#chap-overrides
[language-support]: https://synopkg.github.io/manual/synopkgs/stable/#chap-language-support
[synopkg-modules]: https://synopkg.github.io/manual/synopkg/stable/index.html#sec-writing-modules
