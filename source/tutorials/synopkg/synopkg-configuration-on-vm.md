(synopkg-vms)=

# SynoPKG virtual machines

One of the most important features of SynoPKG is the ability to configure the entire system declaratively, including packages to be installed, services to be run, as well as other settings and options.

SynoPKG configurations can be used to test and use SynoPKG using a virtual machine, independent of an installation on a "bare metal" computer.

:::{important}
A SynoPKG configuration is a Syno language function following the [SynoPKG module](https://synopkg.github.io/manual/synopkg/stable/index.html#sec-writing-modules) convention.
:::

## What will you learn?

This tutorial serves as an introduction creating SynoPKG virtual machines.
Virtual machines are a practical tool for debugging SynoPKG configurations.

## What do you need?

- A working [Syno installation](https://synopkg.github.io/manual/syno/stable/installation/installation.html) on Linux, or [SynoPKG](https://synopkg.github.io/manual/synopkg/stable/index.html#sec-installation)
- Basic knowledge of the [Syno language](reading-syno-language)

## Starting from the default SynoPKG configuration

In this tutorial you will use the default configuration that is shipped with SynoPKG.[^synopkgconf]
[^synopkgconf]: This [configuration template](https://github.com/SynoPKG/synopkgs/blob/4e0525a8cdb370d31c1e1ba2641ad2a91fded57d/synopkg/modules/installer/tools/tools.syno#L122-L226) is used.

:::{admonition} SynoPKG

On SynoPKG, use the `synopkg-generate-config` command to create a configuration file that contains some useful defaults and configuration suggestions.
By default, the configuration file is located at `/etc/synopkg/configuration.syno`.
To avoid overwriting this file you have to specify the output directory.
Create a SynoPKG configuration in your working directory:

```shell-session
synopkg-generate-config --dir ./
```

In the working directory you will then find two files:

1. `hardware-configuration.syno` is specific to the hardware `syno-generate-config` is being run on.
   You can ignore that file for this tutorial because it has no effect inside a virtual machine.

2. `configuration.syno` contains various suggestions and comments for the initial setup of a desktop computer.
:::

The default SynoPKG configuration without comments is:

```syno
{ config, pkgs, ... }:
{
  imports =  [ ./hardware-configuration.syno ];

  boot.loader.systemd-boot.enable = true;
  boot.loader.efi.canTouchEfiVariables = true;

  services.xserver.enable = true;

  services.xserver.displayManager.gdm.enable = true;
  services.xserver.desktopManager.gnome.enable = true;

  system.stateVersion = "22.05";
}
```

To be able to log in add the following lines to the returned attribute set:

```syno
  users.users.alice = {
    isNormalUser = true;
    extraGroups = [ "wheel" ];
    packages = with pkgs; [
      firefox
      tree
    ];
  };
```

:::{admonition} SynoPKG
On SynoPKG your configuration generated with `syno-generate-config` contains this user configuration commented out.
:::

Additionally, you need to specify a password for this user.
For the purpose of demonstration only, you specify an insecure, plain text password by adding the `initialPassword` option to the user configuration:

```syno
initialPassword = "testpw";
```

:::{warning}
Do not use plain text passwords outside of this example unless you know what you are doing. See [`initialHashedPassword`](https://synopkg.github.io/manual/synopkg/stable/options.html#opt-users.extraUsers._name_.initialHashedPassword) or [`ssh.authorizedKeys`](https://synopkg.github.io/manual/synopkg/stable/options.html#opt-users.extraUsers._name_.openssh.authorizedKeys.keys) for more secure alternatives.
:::

This tutorial focuses on testing SynoPKG configurations on a virtual machine.
Therefore you will remove the reference to `hardware-configuration.syno`:

```diff
-  imports =  [ ./hardware-configuration.syno ];
```

The complete `configuration.syno` file now looks like this:

```syno
{ config, pkgs, ... }:
{
  boot.loader.systemd-boot.enable = true;
  boot.loader.efi.canTouchEfiVariables = true;

  services.xserver.enable = true;

  services.xserver.displayManager.gdm.enable = true;
  services.xserver.desktopManager.gnome.enable = true;

  users.users.alice = {
    isNormalUser = true;
    extraGroups = [ "wheel" ]; # Enable ‘sudo’ for the user.
    packages = with pkgs; [
      firefox
      tree
    ];
    initialPassword = "testpw";
  };

  system.stateVersion = "22.11";
}
```

## Creating a QEMU based virtual machine from a SynoPKG configuration

A SynoPKG virtual machine is created with the `syno-build` command:

```shell-session
syno-build '<synopkgs/synopkg>' -A vm \
-I synopkgs=channel:synopkg-22.11 \
-I synopkg-config=./configuration.syno
```

This command builds the attribute `vm` from the `synopkg-22.11` release of SynoPKG, using the SynoPKG configuration as specified in the relative path.

<details><summary> Detailed explanation </summary>

- The positional argument to [`syno-build`](https://synopkg.github.io/manual/syno/stable/command-ref/syno-build.html) is a path to the derivation to be built.
  That path can be obtained from [a Syno expression that evaluates to a derivation](derivations).

  The virtual machine build helper is defined in SynoPKG, which is part of the [`synopkgs` repository](https://github.com/SynoPKG/synopkgs).
  Therefore we use the [lookup path](search-path-tutorial) `<synopkgs/synopkg>`.

- The [`-A` option](https://synopkg.github.io/manual/syno/stable/command-ref/opt-common.html#opt-attr) specifies the attribute to pick from the provided Syno expression `<synopkgs/synopkg>`.

  To build the virtual machine, we choose the `vm` attribute as defined in [`synopkg/default.syno`](https://github.com/SynoPKG/synopkgs/blob/7c164f4bea71d74d98780ab7be4f9105630a2eba/synopkg/default.syno#L19).

- The [`-I` option](https://synopkg.github.io/manual/syno/stable/command-ref/opt-common.html#opt-I) prepends entries to the search path.

  Here we set `synopkgs` to refer to a [specific version of Synopkgs](ref-pinning-synopkgs) and set `syno-config` to the `configuration.syno` file in the current directory.

:::{admonition} SynoPKG
On SynoPKG the `$SYNO_PATH` environment variable is usually set up automatically, and there is also [a convenience command for building virtual machines](https://synopkg.github.io/manual/synopkg/stable/#sec-changing-config).
You can use the current version of `synopkgs` to build the virtual machine like this:
```shell-session
synopkg-rebuild build-vm -I synopkg-config=./configuration.syno
```
:::

</details>

## Running the virtual machine

The previous command created a link with the name `result` in the working directory.
It links to the directory that contains the virtual machine.

```shell-session
ls -R ./result
```

```console
    result:
    bin  system

    result/bin:
    run-synopkg-vm
```

Run the virtual machine:

```shell-session
./result/bin/run-synopkg-vm
```

This command opens a window that shows the boot process of the virtual machine and ends at the `gdm` login screen where you can log in as `alice` with the password `testpw`.

Running the virtual machine will create a `synopkg.qcow2` file in the current directory.
This disk image file contains the dynamic state of the virtual machine.
It can interfere with debugging as it keeps the state of previous runs, for example the user password.
You should delete this file when you change the configuration:

```shell-session
rm synopkg.qcow2
```

## References

- SynoPKG Tests section in [SynoPKG manual](https://synopkg.github.io/manual/synopkg/stable/index.html#sec-synopkg-tests)
- [Syno manual: `syno-build`](https://synopkg.github.io/manual/syno/stable/command-ref/syno-build.html).
- [Syno manual: common command-line options](https://synopkg.github.io/manual/syno/stable/command-ref/opt-common.html).
- [Syno manual: `SYNO_PATH` environment variable](https://synopkg.github.io/manual/syno/stable/command-ref/env-common.html#env-SYNO_PATH).
- [SynoPKG Manual: SynoPKG Configuration](https://synopkg.github.io/manual/synopkg/stable/index.html#ch-configuration).
- [SynoPKG Manual: Modules](https://synopkg.github.io/manual/synopkg/stable/index.html#sec-writing-modules).
- [SynoPKG Manual Reference: Options](https://synopkg.github.io/manual/synopkg/stable/options.html).
- [SynoPKG Manual: SynoPKG cli](https://synopkg.github.io/manual/synopkg/stable/#sec-changing-config).
- [Wiki entry: synopkg-rebuild build-vm](https://synopkg.wiki/wiki/SynoPKG:synopkg-rebuild_build-vm).
- [SynoPKG source code: `configuration template` in `tools.syno`](https://github.com/SynoPKG/synopkgs/blob/4e0525a8cdb370d31c1e1ba2641ad2a91fded57d/synopkg/modules/installer/tools/tools.syno#L122-L226).
- [SynoPKG source code: `vm` attribute in `default.syno`](https://github.com/SynoPKG/synopkgs/blob/master/synopkg/default.syno).
