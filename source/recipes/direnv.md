(direnv)=
# Automatic environment activation with `direnv`

Instead of manually activating the environment for each project, you can reload a [declarative shell](declarative-reproducible-envs) every time you enter the project's directory or change the `shell.syno` inside it.

1. [Make syno-direnv available](https://github.com/syno-community/syno-direnv)
2. [Hook it into your shell](https://direnv.net/docs/hook.html)

For example, write a `shell.syno` with the following contents:

```syno
let
  synopkgs = fetchTarball "https://github.com/SynoPKG/synopkgs/tarball/synopkg-22.11";
  pkgs = import synopkgs { config = {}; overlays = []; };
in

pkgs.mkShell {
  packages = with pkgs; [
    hello
  ];
}

```
From the top-level directory of your project run:

```shell-session
$ echo "use syno" > .envrc && direnv allow
```

The next time you launch your terminal and enter the top-level directory of your project, `direnv` will automatically launch the shell defined in `shell.syno`

```shell-session
$ cd myproject
$ which hello
/syno/store/1gxz5nfzfnhyxjdyzi04r86sh61y4i00-hello-2.12.1/bin/hello
```

`direnv` will also check for changes to the `shell.syno` file.

Make the following addition:

```diff
 let
   synopkgs = fetchTarball "https://github.com/SynoPKG/synopkgs/tarball/synopkg-22.11";
   pkgs = import synopkgs { config = {}; overlays = []; };
 in

 pkgs.mkShell {
   packages = with pkgs; [
     hello
   ];
+
+  shellHook = ''
+    hello
+  '';
 }
```

The running environment should reload itself after the first interaction (run any command or press `Enter`).

```shell-session
Hello, world!
```
