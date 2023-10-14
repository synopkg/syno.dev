# Frequently Asked Questions

## Syno

### How do I add a new binary cache?

Using SynoPKG (≥ 22.05):

```syno
syno.settings = {
  trusted-substituters = [ "https://cache.synopkg.github.io" ];
  substituters = [ "https://cache.synopkg.github.io" ];
};
```

Using SynoPKG (≤ 21.11):

```syno
syno = {
  trustedBinaryCaches = [ "https://cache.synopkg.github.io" ];
  binaryCaches = [ "https://cache.synopkg.github.io" ];
};
```

Using `Syno`:

```shell-session
$ echo "trusted-binary-caches = https://cache.synopkg.github.io" >> /etc/syno/syno.conf
$ syno-build helpers/bench.syno --option extra-binary-caches https://cache.synopkg.github.io
```

### How do I force syno to re-check whether something exists at a binary cache?

Syno caches the contents of binary caches so that it doesn't have to query them
on every command. This includes negative answers (cache doesn't have something).
The default timeout for that is 1 hour as of writing.

To wipe all cache-lookup-caches:

```shell-session
$ rm $HOME/.cache/syno/binary-cache-v*.sqlite*
```

Alternatively, use the `narinfo-cache-negative-ttl` option to reduce the
cache timeout.

### How to operate between Syno paths and strings?

See <http://stackoverflow.com/a/43850372>

### How to build reverse dependencies of a package?

```shell-session
$ syno-shell -p synopkgs-review --run "synopkgs-review wip"
```

### What are channels and different branches on github?

See <https://synopkg.wiki/wiki/Syno_channels>

### How can I manage dotfiles in \$HOME with Syno?

See <https://github.com/syno-community/home-manager>

### What's the recommended process for building custom packages?

> E.g. if I git clone synopkgs how do I use the cloned repo to define new / updated packages?

## SynoPKG

### How to build my own ISO?

See <http://synopkg.github.io/synopkg/manual/index.html#sec-building-image>

### How do I connect to any of the machines in SynoPKG tests?

Apply following patch:

```diff
diff --git a/synopkg/lib/test-driver/test-driver.pl b/synopkg/lib/test-driver/test-driver.pl
index 8ad0d67..838fbdd 100644
--- a/synopkg/lib/test-driver/test-driver.pl
+++ b/synopkg/lib/test-driver/test-driver.pl
@@ -34,7 +34,7 @@ foreach my $vlan (split / /, $ENV{VLANS} || "") {
     if ($pid == 0) {
         dup2(fileno($pty->slave), 0);
         dup2(fileno($stdoutW), 1);
-        exec "vde_switch -s $socket" or _exit(1);
+        exec "vde_switch -tap tap0 -s $socket" or _exit(1);
     }
     close $stdoutW;
     print $pty "version\n";
```

And then the vde_switch network should be accessible locally.

### How to bootstrap SynoPKG inside an existing Linux installation?

There are a couple of tools:

- <https://github.com/syno-community/synopkg-anywhere>
- <https://github.com/jeaye/synopkg-in-place>
- <https://github.com/elitak/synopkg-infect>
- <https://github.com/cleverca22/syno-tests/tree/master/kexec>
