! Installation

I need -

* Elixir
* PostgreSQL
* inotify-tools

I can package all of these up with `nix`.

Let's generate `nix` flake boilerplate via ...

```
nix flake init templates#utils-generic
```

Now I can just edit the function `pkgs.mkShell` to setup a development environment with the tools I need installed.


! Up & Running

`mix ecto.create` threw `[error] Postgrex.Protocol (#PID<0.340.0>) failed to connect: ** (DBConnection.ConnectionError) tcp connect (localhost:5432): connection refused - :econnrefused`.


The [[Phoenix Official Getting Started]] assumes that `Postgres` is already running.  //It should probably include an explicit check in the guide, since this assumption means that ''this error isn't covered in the `ecto` FAQ''.//


<<<
''Q'' - Why does `mix ecto.create` fail with `[error] Postgrex.Protocol (#PID<0.340.0>) failed to connect: ** (DBConnection.ConnectionError) tcp connect (localhost:5432): connection refused - :econnrefused`?
<<<


''Guess'' -

I need PostgreSQL to be running before I can spin up the required tables.

I can create a Postgres database via ...

```
pg_ctl init -D .db/
```

... and start it via ...

```
pg_ctl start -D .db/
```

<<<
''Note'' - it's not so simple for a nix-installed `Postgres` since `pg_ctl start` attempts to create a lock file `/run/postgresql/.s.PGSQL.5432.lock` which `nix` doesn't allow

[[How do I launch a nix-installed Postgres?]]
<<<


After creating a database & starting it, I get a different error  -

<<<
''Q'' -  Why does `mix ecto.create` fail with `[error] Postgrex.Protocol (#PID<0.203.0>) failed to connect: ** (Postgrex.Error) FATAL 28000 (invalid_authorization_specification) role "postgres" does not exist`
<<<

This issue ''is covered in the `ecto` FAQ''.  It's an expected fail.

As per, I can fix it with -

```
psql --dbname postgres

CREATE ROLE postgres LOGIN CREATEDB;
```

<<<
''Note'' - Or on `nix` -

```
psql --host /tmp --dbname postgres
```
<<<



<<<
''Q'' - What credentials does `ecto` try to connect with?
<<<

`config/dev.exs` - 

```
config :hello, Hello.Repo,
  username: "postgres",
  password: "postgres",
  hostname: "localhost",
  database: "hello_dev",
  stacktrace: true,
  show_sensitive_data_on_connection_error: true,
  pool_size: 10
```

''Solved!''

And we're live.

