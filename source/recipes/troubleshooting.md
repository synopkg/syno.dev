# Troubleshooting

## What to do if a binary cache is down or unreachable?

Pass [`--option substitute false`](https://synopkg.github.io/manual/syno/stable/command-ref/conf-file#conf-substitute) to Syno commands.

### How to fix: `error: querying path in database: database disk image is malformed`

This is a [known issue](https://github.com/SynoPKG/syno/issues/1353).
Try:

```shell-session
$ sqlite3 /syno/var/syno/db/db.sqlite "pragma integrity_check"
```

Which will print the errors in the [database](https://synopkg.github.io/manual/syno/stable/glossary#gloss-syno-database).
If the errors are due to missing references, the following may work:

```shell-session
$ mv /syno/var/syno/db/db.sqlite /syno/var/syno/db/db.sqlite-bkp
$ sqlite3 /syno/var/syno/db/db.sqlite-bkp ".dump" | sqlite3 /syno/var/syno/db/db.sqlite
```

### How to fix: `error: current Syno store schema is version 10, but I only support 7`

This is a [known issue](https://github.com/SynoPKG/syno/issues/1251).

It means that using a new version of Syno upgraded the SQLite schema of the [database](https://synopkg.github.io/manual/syno/stable/glossary#gloss-syno-database), and then you tried to use an older version Syno.

The solution is to dump the database, use the old Syno version to initialize it, and then re-import the data:

```shell-session
$ /path/to/syno/unstable/bin/syno-store --dump-db > /tmp/db.dump
$ mv /syno/var/syno/db /syno/var/syno/db.toonew
$ mkdir /syno/var/syno/db
$ syno-store --init # this is the old syno-store
$ syno-store --load-db < /tmp/db.dump
```

### How to fix: `writing to file: Connection reset by peer`

This may mean you are trying to import a too large file or directory into the [Syno store](https://synopkg.github.io/manual/syno/stable/glossary#gloss-store), or your machine is running out of resources, such as disk space or memory.

Try to reduce the size of the directory to import, or run [garbage collection](https://synopkg.github.io/manual/syno/stable/command-ref/syno-collect-garbage).
