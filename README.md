A simple, customizable table audit system for PostgreSQL implemented using
triggers and JSONB for storing diffs. Additionally, if any column is also a
JSON type, a recursive diff will be generated for changed fields.

This trigger is a fork of [2ndQuadrant's audit trigger][1] implementation.

For more information:
  * https://github.com/2ndQuadrant/audit-trigger
  * http://wiki.postgresql.org/wiki/Audit_trigger_91plus

[1]: https://github.com/2ndQuadrant/audit-trigger

## Audit Table Reference

Column | Type | Not&nbsp;Null | Description
--- | --- | :---: | ---
`id` | `BIGINT` | &#x2611;  | Unique identifier for each auditable event
`schema_name` | `TEXT` | &#x2611;  | Database schema audited table for this event is in
`table_name` | `TEXT` | &#x2611;  | Non-schema-qualified table name of table event occured in
`relid` | `OID` | &#x2611;  | Table OID. Changes with drop/create.
`session_user_name` | `TEXT` | &#x2611; | Login / session user whose statement caused the audited event
`current_user_name` | `TEXT` | &#x2611; | Effective user that cased audited event (if authorization level changed)
`action_tstamp_tx` | `TIMESTAMP` | &#x2611; | Transaction start timestamp for tx in which audited event occurred
`action_tstamp_stm` | `TIMESTAMP` | &#x2611; | Statement start timestamp for tx in which audited event occurred
`action_tstamp_clk` | `TIMESTAMP` | &#x2611; | Wall clock time at which audited event's trigger call occurred
`transaction_id` | `BIGINT` | &#x2611; | Identifier of transaction that made the change. <br />Unique when paired with `action_tstamp_tx.`
`client_addr` | `INET` | | IP address of client that issued query. Null for unix domain socket.
`client_port` | `INTEGER` | | Port address of client that issued query. <br />Undefined for unix socket.
`client_query` | `TEXT` | | Top-level query that caused this auditable event. <br />May be more than one.
`application_name` | `TEXT` | | Client-set session application name when this audit event occurred.
`application_user` | `TEXT` | | Client-set session application user when this audit event occurred.<br /> This is useful if the application uses its own user-management and authorization system.
`action` | `ENUM` | &#x2611;  | Action type <br /> `I` = insert <br />`D` = delete<br /> `U` = update<br/>`T` = truncate
`row_data` | `JSONB` | | Record value. Null for statement-level trigger.<br />For INSERT this is the new tuple.<br /> For DELETE and UPDATE it is the old tuple.
`changed_fields` | `JSONB` | | New values of fields changed by UPDATE. <br /> Null except for row-level UPDATE events.
`statement_only` | `BOOLEAN` | &#x2611;  | `t` if audit event is from an FOR EACH STATEMENT trigger <br /> `f` for FOR EACH ROW


## Installation

Requirements:
  * PostgreSQL Server 9.6+ (including developer header files)

To install:

```
> git clone git@github.com:m-martinez/pg-audit-json
> cd pg-audit-json
> make
> make install
```

It is highly recommended that you only install this extension using a
postgres administrative account and not the account an application will
be using to interact the database.

In your postgres shell, activate the extension using:

```sql
> CREATE EXTENSION "pg-audit-json";
```

To  run the tests:

```
> make installcheck
```

## Usage

### Tracking a database table

To track a user table, use the `audit.audit_table` function as the ONWER of the
audit.log table. Here are a few exapmles:

```sql
> -- A simple table
> SELECT audit.audit_table("mytable");
>
> -- A schema-qualified table
> SELECT audit.audit_table("myschema"."mytable");
>
> -- Ignore columns "foo" and "bar"
> SELECT audit.audit_table("mytable", true, true, "{foo,bar}");
```

### Setting application runtime variables

This extension allows you to define two optional settings in your application
runtime, which can be set as follows:

```sql
> set_config('applicaiton_name', 'my.fancy.app', true)
> set_config('application_user_name', 'jdoe@foo.com', true)
```

Setting | Description
--- | ---
`application_name` | The name of the application that will be trigger audit events
`appliation_user_name` | The effective applicaiton user


## Contributing

This project provides and [editorconfig](http://editorconfig.org) to conform
to a coding style.

More information about PostgreSQL extensions
  * https://www.postgresql.org/docs/current/static/extend-pgxs.html
  * https://www.postgresql.org/docs/current/static/extend-extensions.html
  * http://manager.pgxn.org/howto

### Releasing

Remember to update the version tags in the following files:
  * META.json
  * pg-audit-json.control

