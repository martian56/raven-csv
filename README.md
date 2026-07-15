# raven-csv

[![CI](https://github.com/martian56/raven-csv/actions/workflows/ci.yml/badge.svg)](https://github.com/martian56/raven-csv/actions/workflows/ci.yml)

A CSV reader and writer for Raven that follows RFC 4180. Quoted fields can hold
commas, newlines, and doubled-up quotes, and on the way out a field is only
quoted when it actually needs to be. It's all in `lib.rv`, with no dependencies.

## Install

```toml
[dependencies]
"github.com/martian56/raven-csv" = "v0.3.0"
```

## Reading

```rust
import "github.com/martian56/raven-csv" { Csv }

fun main() {
    let text = "name,city\nAlice,\"New York, NY\"\nBob,Boston\n"
    match Csv.new().with_header().parse(text) {
        Ok(table) -> {
            let city = table.field(0, "city")   // Some("New York, NY")
            let who = table.get(1, 0)           // "Bob"
        },
        Err(e) -> print(e.to_string()),
    }
}
```

The reader is set up with a few chained calls, each returning a fresh copy:

```rust
Csv.new()                  // comma delimiter, no header row, no trimming
Csv.new().with_header()    // first row becomes the header names
Csv.new().delimiter("\t")  // tab-separated instead
Csv.new().trimmed()        // drop surrounding spaces on unquoted fields
```

Parsing only fails on one thing, a quote that never closes, and the error says
which line it gave up on:

```
csv:3: unterminated quoted field
```

## Writing

`write` takes the rows (each a list of strings) and hands back CSV text. A field
that contains a comma, a quote, or a line break gets wrapped in quotes; the rest
are left as they are.

```rust
import "github.com/martian56/raven-csv" { write }

let out = write(rows)
```

Use `write_with` if you want a delimiter other than a comma.

## A couple of details

Rows can be ragged, so a short row is fine. `field` looks a cell up by its
header name and gives you an `Option<String>`. `get` indexes by row and column
and just returns `""` when you run off the edge, which keeps call sites from
needing a `match` everywhere.

## API

### `Csv`

| Method | Description |
|--------|-------------|
| `Csv.new()` | A reader: comma delimiter, no header, no trim |
| `csv.delimiter(d)` | A copy using `d` as the field delimiter |
| `csv.with_header()` | A copy that treats the first row as headers |
| `csv.trimmed()` | A copy that trims unquoted fields |
| `csv.parse(text)` | Parse into a `Table`, returns `Result<Table, CsvError>` |

### `Table`

`headers: List<String>` and `rows: List<List<String>>` are public fields.

| Method | Description |
|--------|-------------|
| `t.row_count()` | How many data rows there are |
| `t.col_count()` | Column count, from the header if present, else the first row |
| `t.row(i)` | The row at `i` (empty when out of range) |
| `t.get(row, col)` | The cell, or `""` when out of range |
| `t.field(row, name)` | The cell under header `name`, as `Option<String>` |

### Functions

| Function | Description |
|----------|-------------|
| `write(rows)` | Rows to CSV text with a comma delimiter |
| `write_with(rows, delimiter)` | Same, with a delimiter you pick |

### `CsvError`

Has a 1-based `line` and a `message`, and a `to_string` that reads
`csv:<line>: <message>`.

## License

MIT
