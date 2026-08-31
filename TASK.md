# Task: add table sectioning roles

Add three new roles distinguishing a table's header/body/footer rows: `tableHead`, `tableBody`, `tableFooter`. Fixes a real bug: with no way to mark a `<tfoot>` row as different from a data row, a footer summary row (e.g. `<tfoot><tr><td>Total: 1</td></tr></tfoot>`) gets associated with the last-seen column header by downstream consumers, producing nonsense like "Name: Total: 1".

## 1. `schema/roles.schema.json`

Add to the `enum` array: `tableHead`, `tableBody`, `tableFooter`.

## 2. `roles.md`

Add three rows to the "Inherited from HTML and/or ARIA" table (EPUB type and ARIA columns blank, same as existing HTML-only roles like `body`/`header`/`section`):

| Role | EPUB type | ARIA | HTML | Definition |
|---|---|---|---|---|
| `tableHead` | | | `<thead>` | Section of the table holding header rows. |
| `tableBody` | | | `<tbody>`, or a row with no `thead`/`tbody`/`tfoot` ancestor | Section of the table holding body rows. |
| `tableFooter` | | | `<tfoot>` | Section of the table holding footer rows. |

Also add them to the `## List of escapable roles` bullet list, nested under `table` alongside the existing `columnheader`/`rowheader`/`row`/`cell`:

```
* `table`
  * `tableHead`
  * `tableBody`
  * `tableFooter`
  * `columnheader`
  * `rowheader`
  * `row`
  * `cell`
```

## Tree shape (for reference — not part of the schema/roles.md diff, just context for what a document producer should output)

`table`'s children become `tableHead`/`tableBody`/`tableFooter` objects, each wrapping its `row` children, instead of `row` objects directly:

```html
<table>
  <tr><th>Name</th></tr>
  <tr><td>Ada</td></tr>
  <tfoot><tr><td>Total: 1</td></tr></tfoot>
</table>
```

```json
{
  "role": ["table"],
  "children": [
    {
      "role": ["tableBody"],
      "children": [
        { "role": ["row"], "children": [
          { "role": ["columnheader"], "text": "Name" }
        ]},
        { "role": ["row"], "children": [
          { "role": ["cell"], "text": "Ada" }
        ]}
      ]
    },
    {
      "role": ["tableFooter"],
      "children": [
        { "role": ["row"], "children": [
          { "role": ["cell"], "text": "Total: 1" }
        ]}
      ]
    }
  ]
}
```

A row with no `thead`/`tbody`/`tfoot` ancestor resolves to `tableBody` (HTML's own implicit-tbody rule).

## Out of scope

colspan/rowspan is a separate, unresolved gap — do not attempt it here.
