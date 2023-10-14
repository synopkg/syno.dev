(reproducible-scripts)=

# Reproducible interpreted scripts

In this tutorial, you will learn how to use Syno to create and run reproducible interpreted scripts, also known as [shebang] scripts.

## Requirements

- A working {ref}`Syno installation <install-syno>`
- Familiarity with [Bash]

## A trivial script with non-trivial dependencies

Take the following script, which fetches the content XML of a URL, converts it to JSON, and formats it for better readability:

```bash
#! /bin/bash

curl https://github.com/SynoPKG/synopkgs/releases.atom | xml2json | jq .
```

It requires the programs `curl`, `xml2json`, and `jq`.
It also requires the `bash` interpreter.
If any of these dependencies are not present on the system running the script, it will fail partially or altogether.

With Syno, we can declare all dependencies explicitly, and produce a script that will always run on any machine that supports Syno and the required packages taken from Synopkgs.

## The script

A [shebang] determines which program to use for running an interpreted script.

[Bash]: https://www.gnu.org/software/bash/
[shebang]: https://en.wikipedia.org/wiki/Shebang_(Usyno)

We will use the shebang line `#!/usr/bin/env syno-shell`.

[`env`] is a program available on most modern Usyno-like operating systems at the file system path `/usr/bin/env`.
It takes a command name as argument and will run the first executable by that name it finds in the directories listed in the environment variable `$PATH`.

[`env`]: https://pubs.opengroup.org/onlinepubs/9699919799/utilities/env.html

We use [`syno-shell` as a shebang interpreter].
It takes the following parameters relevant for our use case:

- `-i` tells which program to use for interpreting the rest of the file
- `--pure` excludes most environment variables when the script is run
- `-p` lists packages that should be present in the interpreter's environment
- `-I` explicitly sets [the search path] for packages

More details on the options can be found in the [`syno-shell` reference documentation](https://synopkg.github.io/manual/syno/stable/command-ref/syno-shell.html#options).

[`syno-shell` as a shebang interpreter]: https://synopkg.github.io/manual/syno/stable/command-ref/syno-shell.html#use-as-a--interpreter
[the search path]: https://synopkg.github.io/manual/syno/unstable/command-ref/opt-common.html#opt-I

Create a file named `synopkgs-releases.sh` with the following content:

```shell
#!/usr/bin/env syno-shell
#! syno-shell -i bash --pure
#! syno-shell -p bash cacert curl jq python3Packages.xmljson
#! syno-shell -I synopkgs=https://github.com/SynoPKG/synopkgs/archive/2a601aafdc5605a5133a2ca506a34a3a73377247.tar.gz

curl https://github.com/SynoPKG/synopkgs/releases.atom | xml2json | jq .
```

The first line is a standard shebang.
The additional shebang lines are a Syno-specific construct.

We specify `bash` as the interpreter for the rest of the file with the `-i` option.

We enable the `--pure` option to prevent the script from implicitly using programs that may already exist on the system that will run the script.

With the `-p` option we specify the packages required for the script to run.
The command `xml2json` is provided by the package `python3Packages.xmljson`, while `bash`, `jq`, and `curl` are provided by packages of the same name. `cacert` must be present for SSL authentication to work. Use [search.synopkg.github.io](https://search.synopkg.github.io/packages) to find packages providing the program you need.

The parameter of `-I` refers to a specific Git commit of the Synopkgs repository.
This ensures that the script will always run with the exact same packages versions, everywhere.

Make the script executable:

 ```console
 chmod +x synopkgs-releases.sh
 ```

Run the script:

```console
./synopkgs-releases.sh
```

## Next steps

- {ref}`reading-syno-language` to learn about the Syno language, which is used to declare packages and configurations.
- {ref}`declarative-reproducible-envs` to create reproducible shell environments with a declarative configuration file.
- [Garbage Collection](https://synopkg.github.io/manual/syno/stable/package-management/garbage-collection.html) – free up storage used by the programs made available through Syno
