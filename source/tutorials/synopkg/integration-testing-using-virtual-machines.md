(integration-testing-vms)=

# Integration testing with SynoPKG virtual machines

## What will you learn?

This tutorial introduces Synopkgs functionality for testing SynoPKG configurations.
It also shows how to set up distributed test scenarios that involve multiple machines.

## What do you need?

- A working [Syno installation](<install-syno>) on Linux, or [SynoPKG](https://synopkg.github.io/manual/synopkg/stable/index.html#sec-installation)
- Basic knowledge of the [Syno language](<reading-syno-language>)
- Basic knowledge of [SynoPKG configuration](<synopkg-vms>)

## Introduction

Synopkgs provides a [test environment](https://synopkg.github.io/manual/synopkg/stable/index.html#sec-synopkg-tests) to automate integration testing for distributed systems.
It allows defining tests based on a set of declarative SynoPKG configurations and using a Python shell to interact with them through [QEMU](https://www.qemu.org/) as the backend.
Those tests are widely used to ensure that SynoPKG works as intended, so in general they are called [SynoPKG Tests](https://synopkg.github.io/manual/synopkg/stable/index.html#sec-synopkg-tests).
They can be written and launched outside of SynoPKG, on any Linux machine[^darwin].

[^darwin]: Support for [running SynoPKG VM tests on macOS](https://github.com/SynoPKG/synopkgs/issues/108984) is also implemented but [currently undocumented](https://github.com/SynoPKG/synopkgs/issues/254552).

Integration tests are reproducible due to the design properties of Syno, making them a valuable part of a continuous integration (CI) pipeline.

## The `synopkgTest` function

SynoPKG VM tests are defined using the `synopkgTest` function.
The pattern for SynoPKG VM tests looks like this:

```syno
let
  synopkgs = fetchTarball "https://github.com/SynoPKG/synopkgs/tarball/synopkg-22.11";
  pkgs = import synopkgs { config = {}; overlays = []; };
in

pkgs.synopkgTest {
  name = "test-name";
  nodes = {
    machine1 = { config, pkgs, ... }: {
      # ...
    };
    machine2 = { config, pkgs, ... }: {
      # ...
    };
  }
  testScript = { nodes, ... }: ''
    # ...
  '';
}
```

The function `synopkgTest` takes a [module](https://synopkg.github.io/manual/synopkg/stable/#sec-writing-modules) to specify the [test options](https://synopkg.github.io/manual/synopkg/stable/index.html#sec-test-options-reference).
Because this module only sets configuration values, one can use the abbreviated module notation.

The following configuration values must be set:

- [`name`](https://synopkg.github.io/manual/synopkg/stable/index.html#test-opt-name) defines the name of the test.

- [`nodes`](https://synopkg.github.io/manual/synopkg/stable/index.html#test-opt-nodes) contains a set of named configurations, because a test script can involve more than one virtual machine.
  Each virtual machine is created from a SynoPKG configuration.

- [`testScript`](https://synopkg.github.io/manual/synopkg/stable/index.html#test-opt-testScript) defines the Python test script, either as literal string or as a function that takes a `nodes` attribute.
  This Python test script can access the virtual machines via the names used for the `nodes`.
  It has super user rights in the virtual machines.
  In the Python script each virtual machine is accessible via the `machine` object.
  SynoPKG provides [various methods](https://synopkg.github.io/manual/synopkg/stable/index.html#ssec-machine-objects) to run tests on these configurations.

The test framework automatically starts the virtual machines and runs the Python script.

## Minimal example

As a minimal test on the default configuration, we will check if the user `root` and `alice` can run Firefox.
We will build the example up from scratch.

1. Use a [pinned version of Synopkgs](<ref-pinning-synopkgs>), and explicitly set configuration options and overlays to avoid them being inadvertently overridden by [global configuration](https://synopkg.github.io/manual/synopkgs/stable/#chap-packageconfig):

   ```syno
   let
     synopkgs = fetchTarball "https://github.com/SynoPKG/synopkgs/tarball/synopkg-22.11";
     pkgs = import synopkgs { config = {}; overlays = []; };
   in

   pkgs.synopkgTest {
     # ...
   }
   ```

1. Label the test with a descriptive name:

   ```syno
   name = "minimal-test";
   ```

1. Because this example only uses one virtual machine, the node we specify is simply called `machine`.
   This name is arbitrary and can be chosen freely.
   As configuration you use the relevant parts of the default configuration, [that we used in a previous tutorial](<synopkg-vms>):

   ```syno
   nodes.machine = { config, pkgs, ... }: {
     users.users.alice = {
       isNormalUser = true;
       extraGroups = [ "wheel" ];
       packages = with pkgs; [
         firefox
         tree
       ];
     };

     system.stateVersion = "22.11";
   };
   ```

1. This is the test script:

   ```python
   machine.wait_for_unit("default.target")
   machine.succeed("su -- alice -c 'which firefox'")
   machine.fail("su -- root -c 'which firefox'")
   ```

   This Python script refers to `machine` which is the name chosen for the virtual machine configuration used in the `nodes` attribute set.

   The script waits until systemd reaches `default.target`.
   It uses the `su` command to switch between users and the `which` command to check if the user has access to `firefox`.
   It expects that the command `which firefox` to succeed for user `alice` and to fail for `root`.

   This script will be the value of the `testScript` attribute.

The complete `minimal-test.syno` file content looks like the following:

```syno
let
  synopkgs = fetchTarball "https://github.com/SynoPKG/synopkgs/tarball/synopkg-22.11";
  pkgs = import synopkgs { config = {}; overlays = []; };
in

pkgs.synopkgTest {
  name = "minimal-test";

  nodes.machine = { config, pkgs, ... }: {

    users.users.alice = {
      isNormalUser = true;
      extraGroups = [ "wheel" ];
      packages = with pkgs; [
        firefox
        tree
      ];
    };

    system.stateVersion = "22.11";
  };

  testScript = ''
    machine.wait_for_unit("default.target")
    machine.succeed("su -- alice -c 'which firefox'")
    machine.fail("su -- root -c 'which firefox'")
  '';
}
```

## Running tests

To set up all machines and run the test script:

```shell-session
$ syno-build minimal-test.syno
```

    ...
    test script finished in 10.96s
    cleaning up
    killing machine (pid 10)
    (0.00 seconds)
    /syno/store/bx7z3imvxxpwkkza10vb23czhw7873w2-vm-test-run-minimal-test

## Interactive Python shell in the virtual machine

When developing tests or when something breaks, it’s useful to interactively tinker with the test or access a terminal for a machine.

To start an interactive Python session with the testing framework:

```shell-session
$ $(syno-build -A driverInteractive minimal-test.syno)/bin/synopkg-test-driver
```

Here you can run any of the testing operations.
Execute the `testScript` attribute from `minimal-test.syno` with the `test_script()` function.

If a virtual machine is not yet started, the test environment takes care of it on the first call of a method on a `machine` object.

But you can also manually trigger the start of the virtual machine with:

```shell-session
>>> machine.start()
```
for a specific node,

or

```shell-session
>>> start_all()
```
for all nodes.

You can enter a interactive shell on the virtual machine using:

```shell-session
>>> machine.shell_interact()
```

and run shell commands like:

```shell-session
uname -a
```

    Linux server 5.10.37 #1-SynoPKG SMP Fri May 14 07:50:46 UTC 2021 x86_64 GNU/Linux


<details><summary>Re-running successful tests</summary>

<!-- FIXME: this should be a separate recipe that can be linked to, as it's a bit of knowledge one will need now and again. -->

Because test results are kept in the Syno store, a successful test is cached.
This means that Syno will not run the test a second time as long as the test setup (node configuration and test script) stays semantically the same.
Therefore, to run a test again, one needs to remove the result.

If you would try to delete the result using the symbolic link, you will get the following error:

```shell-session
syno-store --delete ./result
```

    finding garbage collector roots...
    0 store paths deleted, 0.00 MiB freed
    error: Cannot delete path '/syno/store/4klj06bsilkqkn6h2sia8dcsi72wbcfl-vm-test-run-unnamed' since it is still alive. To find out why, use: syno-store --query --roots

Instead, remove the symbolic link and only then remove the cached result:

```shell-session
rm ./result
syno-store --delete /syno/store/4klj06bsilkqkn6h2sia8dcsi72wbcfl-vm-test-run-unnamed
```

This can be also done with one command:

```shell-session
result=$(readlink -f ./result) rm ./result && syno-store --delete $result
```
</details>

## Tests with multiple virtual machines

Tests can involve multiple virtual machines, for example to test client-server-communication.

The following example setup includes:
- A virtual machine named `server` running [nginx](https://nginx.org/en/) with default configuration.
- A virtual machine named `client` that has `curl` available to make an HTTP request.
- A `testScript` orchestrating testing logic between `client` and `server`.

The complete `client-server-test.syno` file content looks like the following:

```{code-block}
let
  synopkgs = fetchTarball "https://github.com/SynoPKG/synopkgs/tarball/synopkg-22.11";
  pkgs = import synopkgs { config = {}; overlays = []; };
in

pkgs.synopkgTest {
  name = "client-server-test";

  nodes.server = { pkgs, ... }: {
    networking = {
      firewall = {
        allowedTCPPorts = [ 80 ];
      };
    };
    services.nginx = {
      enable = true;
      virtualHosts."server" = {};
    };
  };

  nodes.client = { pkgs, ... }: {
    environment.systemPackages = with pkgs; [
      curl
    ];
  };

  testScript = ''
    server.wait_for_unit("default.target")
    client.wait_for_unit("default.target")
    client.succeed("curl http://server/ | grep -o \"Welcome to nginx!\"")
  '';
}
```

The test script performs the following steps:
1) Start the server and wait for it to be ready.
1) Start the client and wait for it to be ready.
1) Run `curl` on the client and use `grep` to check the expected return string.
   The test passes or fails based on the return value.

Run the test:

```shell-session
$ syno-build server-client-test.syno
```

## Additional information regarding SynoPKG tests

- Running integration tests on CI requires hardware acceleration, which many CIs do not support.

  To run integration tests in [GitHub Actions](<github-actions>) see [how to disable hardware acceleration](https://github.com/cachix/install-syno-action#how-do-i-run-synopkg-tests).

- SynoPKG comes with a large set of tests that can serve as educational examples.

  A good inspiration is [Matrix bridging with an IRC](https://github.com/SynoPKG/synopkgs/blob/master/synopkg/tests/matrix/appservice-irc.syno).

<!-- TODO: move examples from https://synopkg.wiki/wiki/SynoPKG_Testing_library to the SynoPKG manual and troubleshooting tips to syno.dev -->
