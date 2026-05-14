# Simple Table Math

A plugin for Obsidian that performs mathematical operations on Markdown tables.
It dynamically calculates and displays results within your tables as you edit them.
You can also copy the results to your clipboard, either via a keyboard shortcut or via the context menu.

> **This is a fork of [eatcodeplay/obsidian-simple-table-math](https://github.com/eatcodeplay/obsidian-simple-table-math)** with the following additions:
> - `>` (right) and `v` (down) directions, alongside the existing `^` and `<`
> - "Skip header row" setting to exclude the header from `^` calculations
> - Automatic restore of the formula row to the bottom of the table after inserting a new row with Obsidian's `+` button
> - **Per-column header directives** (`[CURRENCY,DECIMALS,AGGREGATE:LABEL]`) for:
>   - Formatting every numeric value in a column (currency code and/or fixed decimal places)
>   - Rendering a non-editable aggregate footer row at the bottom of the table without writing it into the markdown
>   - Adding a free-text label to the synthetic row (e.g. `Total: $1.50`)

https://github.com/user-attachments/assets/af3b295f-5bbd-497f-b507-696e9fcbb690

## How to Use

The plugin allows you to perform calculations (sum, average, minimum, maximum, subtraction, multiplication) on columns or rows of numbers within your Markdown tables.
To trigger a calculation, place a special tag within a table cell.

The tag follows this format: `[operation][direction][start:end][currency]`

* **`[operation]`**: A three-letter code indicating the operation:
	* `SUM`: Calculates the sum of the values.
	* `AVG`: Calculates the average of the values.
	* `MIN`: Finds the minimum value.
	* `MAX`: Finds the maximum value.
	* `SUB`: Subtracts the subsequent values from the first value.
	* `MUL`: Multiplies all the values together.
	* `DIV`: Divides the first value by each subsequent value in sequence.
* **`[direction]`**: Indicates the direction of the values to operate on:
	* `^`: Looks at the cells above the current cell in the same column.
	* `<`: Looks at the cells to the left of the current cell in the same row.
	* `v`: Looks at the cells below the current cell in the same column.
	* `>`: Looks at the cells to the right of the current cell in the same row.
* **`[start:end]`** (Optional): Specifies a range of cells to include in the calculation.
	* If omitted, it defaults to all applicable cells in the specified direction.
	* Use a colon-separated format (e.g., `1:3` for the first three cells). The indices are 1-based.
	* You can also just specify a start (e.g., `2`).
* **`[currency]`** (Optional): A 2-4 letter currency code (e.g., `USD`, `EUR`, `GBP`). If provided, the result will be formatted with the specified currency symbol.
* **`[format]`** (Optional): Format options for input and output.
	* #e[n] Only accepts inputs written in scientific notations, and outputs results in scientific notation with n fraction digits. 

## Examples

**A simple summing example:**
```
| Just Numbers |
| -----------: |
|       100.00 |
|        50.00 |
|        25.00 |
|        12.50 |
|         SUM^ |
```

**Calculating the average and sum of columns:**
```
| Name        | Jan  | Feb  | Mar  | Total |
|:----------- |:---- |:---- |:---- |:----- |
| Alice       | 10   | 20   | 15   | SUM<  |
| Bob         | 5    | 12   | 8    | SUM<  |
| **Average** | AVG^ | AVG^ | AVG^ |       |
```

**Combining operations and displaying currency symbols:**

```
| Item        | Price  | Quantity |      Total |
| :---------- | :----: | :------: | ---------: |
| Apple       | $ 1.00 |    5     |    MUL<USD |
| Banana      | $ 0.50 |    10    |    MUL<USD |
| Orange      | $ 0.75 |    7     |    MUL<USD |
| **Average** |        |          | AVG^2:4USD |
| **TOTAL:**  |        |          | SUM^2:4USD |
```

**Column formatting via header hints:**

Add a suffix to a header cell to format every numeric value in that column. All parts are optional, but at least one must be present:

`[CURRENCY,DECIMALS,AGGREGATE:LABEL]`

- `CURRENCY` — 2-4 letter currency code (e.g. `USD`, `EUR`)
- `DECIMALS` — fixed decimal places (overrides the global `Fractions` setting)
- `AGGREGATE` — one of `sum`, `avg`, `min`, `max`, `sub`, `mul`, `div`. If present, the plugin renders a synthetic footer row at the bottom of the table with the per-column aggregate. The row is *not* part of the markdown — it lives only in the rendered table.
- `LABEL` — free text shown in the synthetic row. If the column also has an aggregate the label is prepended to the value (`Total: $1.50`); otherwise the label appears on its own (useful for putting a `Total:` marker in the leftmost column). Use `[:Label]` for a label without any other formatting.

```
| Item [:Total:] | Price [USD,2,sum] | Qty [,0,sum] | Total [USD,2] | Margin [,3] |
| :------------- | ----------------: | -----------: | ------------: | ----------: |
| Apple          | 1                 | 5            | MUL<          | 0.1234      |
| Banana         | 0.5               | 10           | MUL<          | 0.5         |
```

The header directives disappear from the rendered cells (showing just the labels), the data cells render with the requested formatting, and a footer row at the bottom shows `Total:` under Item, `$1.50` under Price, and `15` under Qty. In editing mode, focusing a cell reveals the raw text so you can edit the directive; the footer row is non-editable.

**Scientific notation example:**

Note: when using #e[n] format the input values must be in scientific notation and cannot include currency units.
```
| Correct | Incorrect |
| ------: | --------: |
|    1000 |     1,000 |
|    -1.0 |      -1,0 |
|   2.0e1 |       $20 |
|     0.1 |       0,1 |
|    1E-1 |      10 % |
| SUM^#e4 |   SUM^#e4 |
```
The correct column will ouput 1.0192e+3 while the incorrect column will output 0.0000e+0.
## Key Features

* **Real-time Updates:** Calculations are performed automatically as you type and edit your tables.
* **Directional Operations:** Calculate based on values in any of the four directions relative to the tag (`^`, `v`, `<`, `>`).
* **Optional Range Selection:** Target specific cells for your calculations.
* **Currency Formatting:** Display results with currency symbols for better readability.
* **Locale-Aware Formatting:** Respects your system's locale for number formatting by default, with an option to override.
* **Copy Results:** When copying a cell containing an operation (using `Ctrl + C` or `⌘ + C`) or the context menu, the calculated result will be copied to your clipboard.
* **Formula Row Stays at the Bottom:** When you insert a new row with Obsidian's `+` button under a table whose last row contains a formula, the formula row automatically slides back down so the new (empty) row ends up above it.
* **Per-Column Formatting:** Add `[USD]`, `[USD,2]`, or `[,3]` to a header cell to format every numeric value below it with that currency and/or decimal count.
* **Synthetic Aggregate Row:** Add an aggregate operation to the directive (`[USD,2,sum]`) to render an aggregate footer row at the bottom of the table without writing it into the markdown.
* **Synthetic Row Labels:** Append `:Label` to any header directive (e.g. `[:Total:]` or `[USD,2,sum:Total]`) to add free-text labels to the synthetic row.

## Settings

You can configure the plugin in the Obsidian settings under "Simple Table Math". 
The following options are available:

* **Fractions:** Set the number of decimal places to display in the calculated results.
* **Number formatting:** Enter a locale code (e.g., `en-US`, `de-DE`) to override the default number formatting. If left blank, it will use the language defined for Obsidian.
* **Highlight last row calculations:** Enable or disable styling for the last row in tables that contain calculations.
* **Skip header row:** When enabled, the first row of each table is excluded from vertical (`^`) calculations. Useful when your header row contains numbers.

## CSS Look & Feel

Simple Table Math adds the CSS class `.stm-value` to every cell containing a calculated value.
You can use this class to style the cells in your tables. By default, these cells will be styled with a bolder font weight.

If the last row of a table contains calculations, it will be styled with a background color to make it easier to see the results.
You can disable this behavior in the plugin settings or write your own CSS snippet to override it.

For the adventurous: any row that contains calculations will also get a `.stm-row` class.

You can find an example CSS snippet in the [snippet.css](https://github.com/eatcodeplay/obsidian-simple-table-math/blob/main/assets/snippet.css) file.

## Gotchas & Known Issues

* **Any column found in the calculation path will be included in the result.**
  * If your headers contain numbers, make sure exclude them from calculations by using range selection.

## Installation

This fork is not in the Obsidian Community Plugins directory. Install it with [BRAT](https://obsidian.md/plugins?id=obsidian42-brat) or manually:

### With BRAT

1. Install the BRAT plugin from the Obsidian Community Plugins tab and enable it.
2. Open BRAT settings → `Add beta plugin` and paste `mrjano/obsidian-simple-table-math`.
3. Enable the plugin under `Settings` → `Community plugins`.

### Manually

1. Build the plugin: `npm install && npm run build`.
2. Copy `dist/main.js`, `dist/styles.css`, and `manifest.json` into `<your_vault>/.obsidian/plugins/simple-table-math/`.
3. In Obsidian, enable the plugin under `Settings` → `Community plugins`. The `.obsidian` folder may be hidden on your OS — show hidden files if needed.

## Contributing

If you find any issues or have suggestions for improvements, feel free to open an issue or submit a pull request on the [GitHub repository](https://github.com/eatcodeplay/obsidian-simple-table-math/).

## License

[MIT License](LICENSE)

---

**Enjoy doing math in your Obsidian tables!**
