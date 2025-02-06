# PROYECTO

- [PROYECTO](#proyecto)
  - [DOCUMENTACIÓN](#documentación)
  - [Get Started](#get-started)


---

## DOCUMENTACIÓN

- InfluxData: Documentation:
  - InfluxDB OSS v2:
    - ~~Query data: Query with **InfluxQL**~~
      - ~~[InfluxQL functions: Aggregates: `count()` `distinct()` `mean()` `sum()`...](https://docs.influxdata.com/influxdb/cloud/query-data/influxql/functions/aggregates/)~~
      - ~~[InfluxQL functions: Selectors: `last()`...](https://docs.influxdata.com/influxdb/cloud/query-data/influxql/functions/aggregates/)~~
      - ~~[Explore data: Subqueries](https://docs.influxdata.com/influxdb/cloud/query-data/influxql/explore-data/subqueries/)~~
    - Query data: Query with **Flux**:
      - [Window & aggregate](https://docs.influxdata.com/influxdb/v2/query-data/flux/window-aggregate/)
    - Visualize data:
      - [Use and manage variables: Common variable queries](https://docs.influxdata.com/influxdb/v2/visualize-data/variables/common-variables/)
  - Flux:
    - Flux documentation:
      - [Work with Flux types: Basic types: Null](https://docs.influxdata.com/flux/v0/data-types/basic/null/#check-if-a-column-value-is-null)
      - [Work with Flux types: Composite types: Record](https://docs.influxdata.com/flux/v0/data-types/composite/record/)
    - Reference: Standard library:
      - [universe (build-in): `aggregateWindow()` function](https://docs.influxdata.com/flux/v0/stdlib/universe/aggregatewindow/)
      - [influxdata: influxdb: schema: `schema.tagValues()`](https://docs.influxdata.com/flux/v0/stdlib/influxdata/influxdb/schema/tagvalues/)
    - <u>Get started:</u>


---

## [Get Started](https://docs.influxdata.com/flux/v0/get-started/)

1. Basic Flux query

```flux
from(bucket: "example-bucket")
    |> range(start: -1d)
    |> filter(fn: (r) => r._measurement == "example-measurement")
    |> mean()
    |> yield(name: "_results")
```
> - `from()` to retrieve data from the data source.
> - Pipe-forward operator `(|>)` to send the output of each function to the next function as input.
> - `range()`, `filter()`, or both to filter data based on column values.
> - `mean()` to calculate the average of values returned from the data source.
> - `yield()` to yield results to the user.

2. Flux data model

<!-- Nota: es como que las columnas importan más que las filas... -->

> - **Stream of tables**: collection of zero or more tables. Data sources return results as a stream of tables.
> - **Table**: collection of columns partitioned by group key.
> - **Column**: collection of values of the same *basic type* that contains one value for each row
> - **Row**: collection of associated column values
> - **Group key**: defines which columns to use to group tables in a stream of tables. Each table in a stream of tables represents a unique group key instance. All rows in a table contain the same values for each group key column.

<!-- - [ ] mucho ojo con las `group key` -->

- Conceptos:
  - **Data sources determine data structure**:
    - The Flux data model is separate from the queried data source model. <u>Queried sources return data structured into columnar tables</u>. The table structure and columns included depends on the data source.
    - For example, InfluxDB returns data grouped by *series*, so each table in the returned stream of tables represents a unique series. However, *SQL data sources* return a stream of tables with a single table and an empty group key.
  - **Column labels begnning with underscores**: Some data sources return column labels prefixed with an underscore (`_`). This is a Flux convention used to identify important or reserved column names. While the underscore doesn’t change the functionality of the column, many functions in the *Flux standard library* expect or require these specific column names.
  - **Operate on tables**: Flux [transformations](https://docs.influxdata.com/flux/v0/function-types/#transformations) take a stream of tables as input, but operate on each table individually. For example, aggregate transformations like `sum()`, output a stream of tables containing one table for each corresponding input table...
  - **Restructure tables**: All tables in a stream of tables are defined by their *group key*. To change how data is partitioned or grouped into tables, use functions such as `group()` or `window()` to modify group keys in a stream of tables.
```flux
data
    |> group(columns: ["foo", "bar"], mode: "by")
```

> [!TIP]
> DEMO: `|> group(columns: ["_time", "loc"])` https://docs.influxdata.com/flux/v0/get-started/data-model/#table-grouping-example


3. Flux syntax basics

> - Pipe-forward operator: ie. (`|>`)
> - Simple expressions: eg. `10 * 3`
> - Predicate expressions: evaluate to true or false, eg. `"John" == "John" and|or 41 > 30`
> - Variables: Assign an expression to a variable using the assignment operator (`a = 1`). Use the name (identifier) of a variable to return its value (`a`)
> - Data types:
>   - Basic types: Boolean, Duration (`23h4m5s`), String, Time (`2021-01-01T00:00:00Z`), Float, Integer; Bytes, unsigned integers, [Nulls](https://docs.influxdata.com/flux/v0/data-types/basic/null/)
>   - **Composite types**:
>     - Records: collections of key-value pairs. Each key is a string. Each value can be a different data type (`{name:"Jim", age: 42, "favorite color": "red"}`). Use dot notation or bracket notation to access a properties of a record.
>     - Arrays: collection of values of the same type. Use bracket notation to access a value at a specific index in an array.
>     - Dictionaries: collection of key-value pairs with keys of the same type and values of the same type. Use `dict.get()` to access elements in a dictionary.
>     - Functions: block of code that uses a set of parameters to perform an operation. Functions can be named or anonymous. Define parameters in parentheses (`()`) and pass parameters into an operation with the arrow operator (`=>`). (**TODO**: revisar)
>   - Regular expressions types: ...
> - Packages: The Flux standard library is organized into packages that contain functions and package-specific options. The universe package is loaded by default. To load other packages, include an import statement for each package at the beginning of your Flux script.
> - Examples of basic syntax:
>   - #Define data stream variables...
>   - #Define custom functions...

4. Flux query basics

```flux
from(bucket: "example-bucket")              // ── Source
    |> range(start: -1d)                    // ── Filter on time
    |> filter(fn: (r) => r._field == "foo") // ── Filter on column values
    |> group(columns: ["sensorID"])         // ── Shape
    |> mean()                               // ── Process
```

- Basic query structure: source > filter > shape > process ie. from > range|filter > group > mean
  - Source: ...
  - Filter: `range()` based on time, `filter()` based on column using a predicate function defined in the `fn` parameter passing each row as a record `r`containing key-value pairs for each column in the row
  - Shape: `group()`, `window()`, `pivot()`, `drop()`, `keep()`
  - Process: aggregate data, select specific data points, rewrite rows, send notifications; `aggregateWindow()` is a helper function that both shapes and processes data. The function windows and groups data by time, and then applies an aggregate or selector function to the restructured tables.
- Write a basic query:
  - Define a data source






