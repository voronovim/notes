# Postgres cheatsheet

Connect to the docker image with postgres. Where $1 is container name.
```sh
docker exec -it $container_name psql postgres -U $username
```

To show the current search path you can use the following command:
```
SHOW search_path;
```

And to put the new schema in the path, you could use:
```
SET search_path TO myschema;
```

Or if you want multiple schemas:
```
SET search_path TO myschema, public;
```
## Main commands:
-   `\q`: Quit/Exit
-   `\c __database__`: Connect to a database
-   `\d __table__`: Show table definition including triggers
-   `\l`: List databases
-   `\dy`: List events
-   `\df`: List functions
-   `\di`: List indexes
-   `\dn`: List schemas
-   `\dt *.*`: List tables from all schemas (if  `*.*`  is omitted will only show SEARCH_PATH ones)
-   `\dv`: List views
-   `\df+ __function__`  : Show function SQL code.
-   `\x`: Pretty-format query results instead of the not-so-useful ASCII tables


## User Related:

-   `\du`: List users
-   `\du __username__`: List a username if present.
-   `create role __test1__`: Create a role with an existing username.
-   `create role __test2__ noinherit login password __passsword__;`: Create a role with username and password.
-   `set role __test__;`: Change role for current session to  `__test__`.
-   `grant __test2__ to __test1__;`: Allow  `__test1__`  to set its role as  `__test2__`.

## Handy queries

-   `SELECT * FROM pg_proc WHERE proname='__procedurename__'`: List procedure/function
-   `SELECT * FROM pg_views WHERE viewname='__viewname__';`: List view (including the definition)
-   `SELECT pg_size_pretty(pg_total_relation_size('__table_name__'));`: Show DB table space in use
-   `SELECT pg_size_pretty(pg_database_size('__database_name__'));`: Show DB space in use
-   `show statement_timeout;`: Show current user's statement timeout
-   `SELECT * FROM pg_indexes WHERE tablename='__table_name__' AND schemaname='__schema_name__';`: Show table indexes

## Output settings

Long lines wrap format
```
\pset format [wrapped|aligned]
```