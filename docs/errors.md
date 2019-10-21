# Error reference)

## Error codes

Error codes make identification/classification of error easier. Moreover, we can have internal range for different system components

| Tool (Binary)        | Range | Description                                                                                                                                                                      |
| -------------------- | ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Common               | 1000  | Common errors across all binaries. Common by itself is not a binary.                                                                                                             |
| Query Engine         | 2000  | Query engine binary is responsible for getting data from data sources via connectors (This powers Photon). The errors in this range would usually be data constraint errors      |
| Migration Engine     | 3000  | Migration engine binary is responsible for performing database migrations (This powers lift). The errors in this range would usually be schema migration/data destruction errors |
| Introspection Engine | 4000  | Introspection engine binary is responsible for printing Prisma schema from an existing database. The errors in this range would usually be errors with schema inferring          |
| Schema Parser        | 5000  | Schema parser is responsible for parsing the Prisma schema file. The errors in this range are usually syntactic or semantic errors.                                              |
| Prisma Format        | 6000  | Prisma format powers the Prisma VSCode extension to pretty print the Prisma schema. The errors in this range are usually syntactic or semantic errors.                           |


## Error templates

### Common

#### P1000: Incorrect database credentials

##### Info

This error occurs when any tool in the Prisma Framework is not able to access your database due to incorrect database credentials.

##### Error message

Authentication failed against database server at `${database_host}`, the provided database credentials for `${database_user}` are not valid. <br /> <br /> Please make sure to provide valid database credentials for the database server at `${database_host}`.

##### How to fix

Ensure that the database credentials you've provide are correct. The database credentials are provided in the `datasource` definition in your [Prisma schema]() (for most database connectors, they're part of the connection string that's provided as the `url` field).

#### P1001: Database not reachable

##### Info

This error occurs when any tool in the Prisma Framework is not able to access your database for unknown reasons.

##### Error message

Can't reach database server at `${database_host}`:`${database_port}` <br /> <br /> Please make sure your database server is running at `${host}:${port}`.

##### How to fix

Ensure that your database server is running and that you can reach it via the connection string that has been provided as the `url` argument on the `datasource` definition in your [Prisma schema](). 


#### P1002: Database timeout

##### Info

This error occurs when any tool in the Prisma Framework sends a request to your database but doesn't receive a response within the timeout period defined by your database.

##### Error message

 The database server at `${database_host}`:`${database_port}` was reached but timed out. <br /> <br /> Please try again. <br /> <br /> Please make sure your database server is running at `${host}:${port}`.

##### How to fix

Retry to see if this was only a temporary issue. If the problem persists, you can either increase the timeout period of your database or try to figure out why the request can't be processed within the given time period.

#### P1003: Database does not exist

##### Info

This error occurs when any tool in the Prisma Framework connects successfully to the database server but can not connect to the _database_ that was specified in the connection string. The connection string is provided via the `url` field of the `datasource` definition in your [Prisma schema](). The database is is the last component in the path of the URL, e.g. in the following connection string for a PostgreSQL database, the database is called `mydb`:

```
postgresql://janedoe:mypassword@localhost:5432/mydb?schema=public
```

> **Note**: Different consumers of the SDK might handle that differently. For example, Lift shows an interactive dialog when running `lift save`, so the user can potentially create the missing database.

##### Error message

_MySQL & PostgreSQL:_

Database `${database_name}` does not exist on the database server at `${database_host}`. <br /> <br /> 

_SQLite:_

Database `${database_file_name}` does not exist on the database server at `${database_file_path}`.

##### How to fix

Ensure that the database exists at the specified location. If it doesn't exist yet, you can create a new one with the specified name to solve the issue.

#### P1004: Incompatible binary

##### Info

This error occurs when the query or migration engine binary is not compatible with the platform it is being used on.

##### Error message

Failed to spawn the binary `${binary_path}` process for platform `${platform}`.

##### How to fix

Ensure the binary is compatible with the platform it is currently used on. Learn more about choosing the right binary for your platform [here](./core/generators/photonjs.md#specifying-the-right-platform-for-photonjs).

#### P1005: Unable to start the query engine

##### Info

This error occurs when the query or migration engine binary can not be started.

##### Error message

Failed to spawn the binary `${binary_path}` process for platform `${platform}`.

##### How to fix

Ensure the binary is compatible with the platform it is currently used on. Learn more about choosing the right binary for your platform [here](./core/generators/photonjs.md#specifying-the-right-platform-for-photonjs).

#### P1006: Binary not found

- **Description**: Photon binary for current platform `${platform}` could not be found. Make sure to adjust the generator configuration in the `schema.prisma` file. <br /> <br />`${generator_config}` <br /> <br />Please run `prisma2 generate` for your changes to take effect.
- **Meta schema**:

  ```ts
  type Meta = {
    // Identifiers for the currently identified execution environment, e.g. `native`, `windows`, `darwin` etc
    platform: string

    // Details of how a generator can be added.
    generator_config: string
  }
  ```

- **Notes**: Tools (like Primsa CLI) consuming `generator_config` might color it using ANSI characters for better reading experience.

#### P1007: Missing write access to download binary

- **Description**: Please try installing Prisma 2 CLI again with the `--unsafe-perm` option. <br /> Example: `npm i -g --unsafe-perm prisma2`

#### P1008: Database operation timeout

- **Description**: Operations timed out after `${time}`
- **Meta schema**:
  ```ts
  type Meta = {
    // Operation time in s or ms (if <1000ms)
    time: number
  }
  ```

#### P1009: Database already exists

- **Description**: Database `${database_name}` already exists on the database server at `${database_host}:${database_port}`
- **Meta schema**:

  ```ts
  type Meta = {
    // Database name, append `database_schema_name` when applicable
    // `database_schema_name`: Database schema name (For Postgres for example)
    database_name: string

    // Database host URI
    database_host: string

    // Database port
    database_port: number
  }
  ```

#### P1010: Database access denied

- **Description**: User `${database_user}` was denied access on the database `${database_name}`
- **Meta schema**:

  ```ts
  type Meta = {
    // Database user name
    database_user: string

    // Database name, append `database_schema_name` when applicable
    // `database_schema_name`: Database schema name (For Postgres for example)
    database_name: string
  }
  ```

### Query Engine

Note: Errors with `*` in the title represent multiple types and are less defined.

#### P2000: Input value too long

- **Description**: The value `${field_value}` for the field `${field_name}` on the is too long for the field's type
- **Meta schema**:

  ```ts
  type Meta = {
    // Concrete value provided for a field on a model in Prisma schema. Should be peeked/truncated if too long to display in the error message
    field_value: string

    // Field name from one model from Prisma schema
    field_name: string
  }
  ```

#### P2001: Record not found

- **Description**: The record searched for in the where condition (`${model_name}.${argument_name} = ${argument_value}`) does not exist
- **Meta schema**:

  ```ts
  type Meta = {
    // Model name from Prisma schema
    model_name: string

    // Argument name from a supported query on a Prisma schema model
    argument_name: string

    // Concrete value provided for an argument on a query. Should be peeked/truncated if too long to display in the error message
    argument_value: string
  }
  ```

#### P2002: Unique key violation

- **Description**: Unique constraint failed on the field: `${field_name}`
- **Meta schema**:
  ```ts
  type Meta = {
    // Field name from one model from Prisma schema
    field_name: string
  }
  ```

#### P2003: Foreign key violation

- **Description**: Foreign key constraint failed on the field: `${field_name}`
- **Meta schema**:
  ```ts
  type Meta = {
    // Field name from one model from Prisma schema
    field_name: string
  }
  ```

#### P2004: Constraint violation

- **Description**: A constraint failed on the database: `${database_error}`
- **Meta schema**:
  ```ts
  type Meta = {
    // Database error returned by the underlying data source
    database_error: string
  }
  ```

#### P2005: Stored value is invalid

- **Description**: The value `${field_value}` stored in the database for the field `${field_name}` is invalid for the field's type
- **Meta schema**:

  ```ts
  type Meta = {
    // Concrete value provided for a field on a model in Prisma schema. Should be peeked/truncated if too long to display in the error message
    field_value: string

    // Field name from one model from Prisma schema
    field_name: string
  }
  ```

#### P2006: `*`Type mismatch: invalid (ID/Date/Json/Enum)

- **Description**: The provided value `${field_value}` for `${model_name}` field `${field_name}` is not valid
- **Meta schema**:

  ```ts
  type Meta = {
    // Concrete value provided for a field on a model in Prisma schema. Should be peeked/truncated if too long to display in the error message
    field_value: string

    // Model name from Prisma schema
    model_name: string

    // Field name from one model from Prisma schema
    field_name: string
  }
  ```

- **Notes**: Details are not finalized. The current idea is that any variable coercion will happen in the query engine. Json might be recognized as a native Prisma scalar. This also brings up the question of shims. Clarity in those parts of the spec would help us answer this. The same questions apply to bring your own ID feature, will we recognize all known ID types (like uuid, cuid, MongoID) and validate them in any later before it hits the database?

#### P2007: `*`Type mismatch: invalid custom type

- **Description**: Data validation error `${database_error}`
- **Meta schema**:
  ```ts
  type Meta = {
    // Database error returned by the underlying data source
    database_error: string
  }
  ```
- **Notes**: Details are not finalized. The current idea is that Photon, Query Engine will simply pass through the data and rely on database for data validation.

#### P2008: Query parsing failed

- **Description**: Failed to parse the query `${query_parsing_error}` at `${query_position}`
- **Meta schema**:

  ```ts
  type Meta = {
    // Error(s) encountered when trying to parse a query in the query engine
    query_parsing_error: string

    // Location of the incorrect parsing, validation in a query. Represented by tuple or object with (line, character)
    query_position: string
  }
  ```

- **Notes**: This is unexpected from Photon (if they do it is a bug in Photon) but they are useful for anyone writing a query builder on top of the query engine.

#### P2009: Query validation failed

- **Description**: Failed to validate the query `${query_validation_error}` at `${query_position}`
- **Meta schema**:

  ```ts
  type Meta = {
    // Error(s) encountered when trying to validate a query in the query engine
    query_validation_error: string

    // Location of the incorrect parsing, validation in a query. Represented by tuple or object with (line, character)
    query_position: string
  }
  ```

- **Notes**: This is unexpected from Photon (if they do it is a bug in Photon) but they are useful for anyone writing a query builder on top of the query engine.

### Migration Engine

#### P3000: Database creation failed

- **Description**: Failed to create database: `${database_error}`
- **Meta schema**:
  ```ts
  type Meta = {
    // Database error returned by the underlying data source
    database_error: string
  }
  ```

#### P3001: Destructive migration detected

- **Description**: Migration possible with destructive changes and possible data loss: `${migration_engine_destructive_details}`
- **Meta schema**:
  ```ts
  type Meta = {
    // Details of a destructive migration from the migration engine
    migration_engine_destructive_details: string
  }
  ```

#### P3002: Migration rollback

- **Description**: The attempted migration was rolled back: `${database_error}`
- **Meta schema**:
  ```ts
  type Meta = {
    // Database error returned by the underlying data source
    database_error: string
  }
  ```

### Introspection

#### P4000: Introspection failed

- **Description**: Introspection operation failed to produce a schema file: `${introspection_error}`.
- **Meta schema**:
  ```ts
  type Meta = {
    // Generic error received from the introspection engine. Indicator of why an introspection failed
    introspection_error: string
  }
  ```

### Schema Parser

#### P5000: Schema parsing failed

- **Description**: Failed to parse schema file: `${schema_parsing_error}` at `${schema_position}`
- **Meta schema**:

  ```ts
  type Position = {
    line: number
    character: number
  }

  type Meta = {
    // Error(s) encountered when trying to parse the schema in the schema parser
    schema_parsing_error: string

    // Location of the incorrect parsing, validation in the schema. Represented by tuple or object with (line, character)
    schema_position: Position
  }
  ```

#### P5001: Schema relational ambiguity

- **Description**: There is a relational ambiguity in the schema file between the models `${A}` and `${B}`.
- **Meta schema**:

  ```ts
  type Meta = {
    // Concrete name of model from Prisma schema that has an ambiguity
    A: string

    // Concrete name of model from Prisma schema that has an ambiguity
    B: string
  }
  ```

#### P5002: Schema string input validation errors

- **Description**: Database URL provided in the schema failed to parse: `${schema_sub_parsing_error}` at `${schema_position}`
- **Meta schema**:

  ```ts
  type Position = {
    line: number
    character: number
  }

  type Meta = {
    // Error(s) encountered when trying to parse a string input to the schema in the schema parser (Like database URL)
    schema_sub_parsing_error: string

    // Location of the incorrect parsing, validation in the schema. Represented by tuple or object with (line, character)
    schema_position: Position
  }
  ```

## Photon.js

#### Photon runtime validation error

- **Description**: Validation Error: `${photon_runtime_error}`
- **Meta schema**:

  ```ts
  type Meta = {
    // Photon runtime error describing a validation error like missing argument or incorrect data type.
    photon_runtime_error: string
  }
  ```

- **Notes**: Photon might use ANSI characters to color the response for a better reading experience. Disabling that feature is documented [here](https://github.com/prisma/specs/tree/master/photonjs#error-character-encoding).

#### Query engine connection error

- **Description**: The query engine process died, please restart the application

---

Additionally, Photon relays the following errors from the SDK: `P1000`, `P1001` , `P1002`, `P1003`, `P1004`, `P1005`, `P1006`, `P2000`, `P2001` , `P2002`, `P2003`, `P2004`, `P2005`, `P2006`, `P2007`, `P2008`, `P2009`.

Note: For `P1006`, Photon provides additional information in case it detects that the binary is incorrectly pinned.

## Prisma Studio

Note: Studio has two workflows:

Electron app: Credentials from the UI → Introspection → Prisma schema → Valid Prisma project
Web app: `prisma2 dev` → Provides Prisma schema i.e a Valid Prisma project

Since studio uses Photon for query building. It relays the same error messages as Photon. Additionally, it relays the following errors from the SDK: `P3000`, `P5000`

## Prisma CLI

Note that Prisma CLI must exit with a non-zero exit code when it encounters an error from which it cannot recover.

### Init

#### Directory already contains schema file

- **Description**: Directory `${folder_name}` is an existing Prisma project
- **Meta schema**:

  ```ts
  type Meta = {
    // Folder name of current working directory (Equivalent of folder name from unix `pwd`)
    folder_name: string
  }
  ```

#### Starter kit

- **Description**: Directory `${folder_name}` is not empty
- **Meta schema**:

  ```ts
  type Meta = {
    // Folder name of current working directory (Equivalent of folder name from unix `pwd`)
    folder_name: string
  }
  ```

Init command relays the following errors from the SDK: `P3000`, `P4000`

More issues for init command failures are covered here: https://prisma-specs.netlify.com/cli/init/errors/

### Generate

Generate command relays the following errors from the SDK: `P5000`, `P5001`, `P5002`

### Dev

Dev command relays the following errors from the SDK: `P1000`, `P1001` , `P1002`, `P1003`, `P1004`, `P1005`, `P1006`, `P3000`, `P3001`, `P5000`, `P5001`, `P5002`

### Lift

Lift commands relays the following errors from the SDK: `P1000`, `P1001` , `P1002`, `P1003`, `P1004`, `P1005`, `P1006`, `P3000`, `P3001`, `P5000`, `P5001`, `P5002`

### Introspect

Introspect command relays the following errors from the SDK: `P1000`, `P1001` , `P1002`, `P1003`, `P1004`, `P1005`, `P1006`, `P4000`

## Programmatic access

Many of these errors from the previous section are expected to be consumed programmatically.

`Photon.js`: In user's code base
`Prisma SDK`: Lift etc, in the tools that use Prisma SDK

Therefore, they should be consumable programmatically and have an error structure:

Error object:

```json
{
  "code": "<ERROR_CODE>",
  "message": "<ERROR_MESSAGE>",
  "meta": "<meta-schema-object>"
}
```

Serialization of the error message (default `toString`) will have the following template:

```
${ERROR_CODE}: ${ERROR_MESSAGE}
```