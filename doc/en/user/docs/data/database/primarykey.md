# Controlling feature ID generation in spatial databases

## Introduction

All spatial database data stores (PostGIS, Oracle, MySQL and so on) normally derive the feature ID from the table primary key and assume certain conventions on how to locate the next value for the key in case a new feature needs to be generated (WFS insert operation).

Common conventions rely on finding auto-increment columns (PostGIS) or finding a sequence that is named after a specific convention such as `<table>_<column>_SEQUENCE` (Oracle case).

In case none of the above is found, normally the store will fall back on generating random feature IDs at each new request, making the table unsuitable for feature ID based searches and transactions.

## Metadata table description

These defaults can be overridden manually by creating a `le` that specifies which columns to use and what strategy to use to generate new primary key values. The (schema qualified) table can be created with this SQL statement (this one is valid for PostGIS and ORACLE, adapt it to your specific database, you may remove the check at the end if you want to):

    --PostGIS DDL

    CREATE TABLE my_schema.gt_pk_metadata (
      table_schema VARCHAR(32) NOT NULL,
      table_name VARCHAR(32) NOT NULL,
      pk_column VARCHAR(32) NOT NULL,
      pk_column_idx INTEGER,
      pk_policy VARCHAR(32),
      pk_sequence VARCHAR(64),
      unique (table_schema, table_name, pk_column),
      check (pk_policy in ('sequence', 'assigned', 'autogenerated'))
    )


    --ORACLE DDL

    CREATE TABLE gt_pk_metadata (
      table_schema VARCHAR2(32) NOT NULL,
      table_name VARCHAR2(32) NOT NULL,
      pk_column VARCHAR2(32) NOT NULL,
      pk_column_idx NUMBER(38),
      pk_policy VARCHAR2(32),
      pk_sequence VARCHAR2(64),
      constraint  chk_pk_policy check (pk_policy in ('sequence', 'assigned', 'autogenerated')));

    CREATE UNIQUE INDEX gt_pk_metadata_table_idx01 ON gt_pk_metadata (table_schema, table_name, pk_column);

The table can be given a different name. In that case, the (schema qualified) name of the table must be specified in the `le` configuration parameter of the store.

The following table describes the meaning of each column in the metadata table.

  ------------------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Column**          **Description**

  **table_schema**    Name of the database schema in which the table is located.

  **table_name**      Name of the table or view to be published

  **pk_column**       Name of a column used to form the feature IDs

  **pk_column_idx**   Index of the column in a multi-column key, else NULL. In case multi column keys are needed, multiple records with the same table schema and table name will be used.

  **pk_policy**       The new value generation policy, used in case a new feature needs to be added in the table (following a WFS-T insert operation).

  **pk_sequence**     The name of the database sequence to be used when generating a new value for the pk_column.
  ------------------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------

The possible values for `pk_policy` are:

> -   `ed`. The value of the attribute in the newly inserted feature will be used (this assumes the "expose primary keys" flag has been enabled)
> -   `ce`. The value of the attribute will be generated from the next value of a sequence indicated in the `pk_sequence` column.
> -   `ed`. The column is an auto-increment one, the next value in the auto-increment will be used.

## Using the metadata table with views

GeoServer can publish spatial views without issues, but normally results in two side effects:

> -   the view is treated as read only
> -   the feature IDs are randomly generated

The metadata table can also refer to views, just use the view name in the `table_name` column: this will result in stable ids, and in databases supporting updatable views, it will also make the code treat the view as writable (thus, enabling usage of WFS-T on it).