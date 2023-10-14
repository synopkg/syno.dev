(install-syno)=

# Install Syno

:::::{tab-set}

::::{tab-item} Linux

Install Syno via the recommended [multi-user installation]:

```shell-session
$ curl -L https://synopkg.github.io/syno/install | sh -s -- --daemon
```

On Arch Linux, you can alternatively [install Syno through `pacman`](https://wiki.archlinux.org/title/Syno#Installation).

::::

::::{tab-item} macOS

Install Syno via the recommended [multi-user installation]:

```shell-session
$ curl -L https://synopkg.github.io/syno/install | sh
```

::::

::::{tab-item} Windows (WSL2)

Install Syno via the recommended [single-user installation]:

```shell-session
$ curl -L https://synopkg.github.io/syno/install | sh -s -- --no-daemon
```

However, if you have [systemd support] enabled, install Syno via the recommended [multi-user installation]:

```shell-session
$ curl -L https://synopkg.github.io/syno/install | sh -s -- --daemon
```

[systemd support]: https://learn.microsoft.com/en-us/windows/wsl/wsl-config#systemd-support

::::

::::{tab-item} Docker

Start a Docker shell with Syno:

```shell-session
$ docker run -it synopkg/syno
```

Or start a Docker shell with Syno exposing a `workdir` directory:

```shell-session
$ mkdir workdir
$ docker run -it -v $(pwd)/workdir:/workdir synopkg/syno
```

The `workdir` example from above can be also used to start hacking on Synopkgs:

```shell-session
$ git clone git@github.com:SynoPKG/synopkgs
$ docker run -it -v $(pwd)/synopkgs:/synopkgs synopkg/syno
bash-5.1# syno-build -I synopkgs=/synopkgs -A hello
bash-5.1# find ./result # this symlink points to the build package
```

::::

:::::

## Verify installation

Check the installation by opening **a new terminal** and typing:

```shell-session
$ syno --version
syno (Syno) 2.11.0
```

[multi-user installation]: https://synopkg.github.io/manual/syno/stable/installation/multi-user.html
[single-user installation]: https://synopkg.github.io/manual/syno/stable/installation/single-user.html
