---
name: Filters
route: /api/filtering
menu: API
---

## Examples

Want to skip to the implementation? Check out these examples:

- [filters](../examples/filters)
- [editable-data](../examples/editable-data)
- [expanding](../examples/expanding)
- [filters](../examples/filters)
- [grouping](../examples/grouping)
- [pagination](../examples/pagination)
- [row-selection](../examples/row-selection)

Filters come in two flavors:

- Column filters
  - A filter that is applied to a single column's accessor value.
  - Stored in the `state.columnFilters` array as an object containing the columnId and the filter value.
- Global filters
  - A single filter value that is applied to all or some of columns' accessor values.
  - Stored in the `state.globalFilter` array as any value, usually a string.

> ℹ️ Column filters are applied in the order they are specified. Normally this order isn't important unless you are provided faceted filter information to your users, or have filters that vary greatly in performance across your dataset. The order of slow/fast filters can have various performance implications depending on how each filter is implemented and the resulting rows from each filter.

## Can-Filter Option Priority

The ability for a column to be **column** filtered is determined by the following fallback logic:

```tsx
const canColumnFilter =
  column.enableAllFilters ??
  column.enableColumnFilter ??
  instance.options.enableFilters ??
  instance.options.enableColumnFilters ??
  column.defaultCanColumnFilter ??
  column.defaultCanFilter ??
  !!column.accessorFn
```

The ability for a column to be **globally** filtered is determined by the following fallback logic:

```tsx
const canGlobalFiler =
  ((instance.options.enableFilters ??
    instance.options.enableGlobalFilter ??
    column.enableAllFilters ??
    column.enableGlobalFilter ??
    column.defaultCanFilter ??
    column.defaultCanGlobalFilter ??
    !!column.accessorFn) &&
    instance.options.getColumnCanGlobalFilterFn?.(column)) ??
  true
```

Each option can be set to `true` or `false` to override the default behavior below it. These options are described in greater detail in the API below.

## State

Filter state is stored on the table instance using the following shape:

```tsx
export type FiltersTableState = {
  columnFilters: ColumnFiltersState
  globalFilter: any
  columnFiltersProgress: number
  globalFilterProgress: number
}

export type ColumnFiltersState = ColumnFilter[]

export type ColumnFilter = {
  id: string
  value: unknown
}
```

## Filter Functions

The following filter functions are built-in to the table core:

- `includesString`
  - Case-insensitive string inclusion
- `includesStringSensitive`
  - Case-sensitive string inclusion
- `equalsString`
  - Case-insensitive string equality
- `equalsStringSensitive`
  - Case-sensitive string equality
- `arrIncludes`
  - Item inclusion within an array
- `arrIncludesAll`
  - All items included in an array
- `arrIncludesSome`
  - Some items included in an array
- `equals`
  - Object/referential equality `Object.is`/`===`
- `weakEquals`
  - Weak object/referential equality `==`
- `betweenNumberRange`
  - Number range inclusion

Every filter function adheres to the following shape:

```tsx
export type FilterFn<TGenerics extends TableGenerics> = {
  (rows: Row<TGenerics>[], columnIds: string[], filterValue: any): any
  autoRemove?: ColumnFilterAutoRemoveTestFn<TGenerics>
}

export type ColumnFilterAutoRemoveTestFn<TGenerics extends TableGenerics> = (
  value: unknown,
  column?: Column<TGenerics>
) => boolean

export type CustomFilterFns<TGenerics extends TableGenerics> = Record<
  string,
  FilterFn<TGenerics>
>
```

Filter functions can be referenced/defined in the following ways:

- Built-in filter functions
- Custom filter functions provided via the `tableOptions.filterFns` option
- Inline-filter functions defined on `columnDefinition.filterFn` properties

The final list of filter functions available for the `columnDefnition.filterFn` and ``tableOptions.globalFilterFn` options use the following type:

```tsx
export type FilterFnOption<TGenerics extends TableGenerics> =
  | 'auto'
  | BuiltInFilterFn
  | keyof TGenerics['FilterFns']
  | FilterFn<TGenerics>
```

## Column Definition Options

### `filterFn`

```tsx
filterFn?: FilterFn | keyof TGenerics['FilterFns'] | keyof BuiltInFilterFns
```

The filter function to use with this column.

Options:

- A `string` referencing a [built-in filter function](#filter-functions))
- A `string` referencing a custom filter function defined on the `filterFns` table option
- A [custom filter function](#filter-functions)

### `enableAllFilters`

```tsx
enableAllFilters?: boolean
```

Enables/disables **all** filters for this column. For option priority, see [Can-Filter Option Priority](../guides/filters#can-filter-option-priority).

### `enableColumnFilter`

```tsx
enableColumnFilter?: boolean
```

Enables/disables the **column** filter for this column. For option priority, see [Can-Filter Option Priority](../guides/filters#can-filter-option-priority).

### `enableGlobalFilter`

```tsx
enableGlobalFilter?: boolean
```

Enables/disables the **global** filter for this column. For option priority, see [Can-Filter Option Priority](../guides/filters#can-filter-option-priority).

### `defaultCanFilter`

```tsx
defaultCanFilter?: boolean
```

If set, will serve as a fallback for enabling/disabling **all** filters for this column. For option priority, see [Can-Filter Option Priority](../guides/filters#can-filter-option-priority).

### `defaultCanColumnFilter`

```tsx
defaultCanColumnFilter?: boolean
```

If set, will serve as a fallback for enabling/disabling **column** filters for this column. For option priority, see [Can-Filter Option Priority](../guides/filters#can-filter-option-priority).

### `defaultCanGlobalFilter`

```tsx
defaultCanGlobalFilter?: boolean
```

If set, will serve as a fallback for enabling/disabling the **global** filter for this column. For option priority, see [Can-Filter Option Priority](../guides/filters#can-filter-option-priority).

## Column API

### `getCanColumnFilter`

```tsx
getCanColumnFilter: () => boolean
```

Returns whether or not the column can be **column** filtered.

### `getCanGlobalFilter`

```tsx
getCanGlobalFilter: () => boolean
```

Returns whether or not the column can be **globally** filtered.

### `getColumnFilterIndex`

```tsx
getColumnFilterIndex: () => number
```

Returns the index (including `-1`) of the column filter in the table's `state.columnFilters` array.

### `getColumnIsFiltered`

```tsx
getColumnIsFiltered: () => boolean
```

Returns whether or not the column is currently filtered.

### `getColumnFilterValue`

```tsx
getColumnFilterValue: () => unknown
```

Returns the current filter value of the column.

### `setColumnFilterValue`

```tsx
setColumnFilterValue: (updater: Updater<any>) => void
```

A function that sets the current filter value for the column. You can pass it a value or an updater function for immutability-safe operations on existing values.

### `getPreFilteredRows`

```tsx
getPreFilteredRows: () => (Row < TGenerics > []) | undefined
```

Returns the rows that were present before this column's filter has been applied. Useful for displaying faceted result counts.

### `getPreFilteredUniqueValues`

```tsx
getPreFilteredUniqueValues: () => Map<any, number>
```

A function that **computes and returns** a `Map` of unique values and their occurences that were present before this column's filter was applied. Useful for displaying faceted result values.

### `getPreFilteredMinMaxValues`

```tsx
getPreFilteredMinMaxValues: () => Map<any, number>
```

A function that **computes and returns** a min/max tuple derived from the values that were present before this column's filter was applied. Useful for displaying faceted result values.

## Table Options

### `filterFromLeafRows`

```tsx
filterFromLeafRows?: boolean
```

By default, filtering is done from parent rows down (so if a parent row is filtered out, all of its children will be filtered out as well). Setting this option to `true` will cause filtering to be done from leaf rows up (which means parent rows will be included so long as one of their child or grand-child rows is also included).

### `filterFns`

```tsx
filterFns?: Record<string, FilterFn>
```

Normally set ahead of time when using the `createTable()` helper, this option allows you to define custom filter functions that can be referenced by their string key.

Example:

```tsx
const table = createTable().setOptions({
  filterFns: {
    myCustomFilterFn: (rows, columnIds, filterValue) => {
      // return the filtered rows
    },
  },
})

const column = table.createDataColumn('key', {
  filterFn: 'myCustomFilterFn',
})
```

### `enableFilters`

```tsx
enableFilters?: boolean
```

Enables/disables all filters for the table. For option priority, see [Can-Filter Option Priority](../guides/filters#can-filter-option-priority).

### `manualColumnFiltering`

```tsx
manualColumnFiltering?: boolean
```

Disables the `getColumnFilteredRowModel` from being used to filter data. This may be useful if your table needs to dynamically support both client-side and server-side filtering.

### `onColumnFiltersChange`

```tsx
onColumnFiltersChange?: OnChangeFn<ColumnFiltersState>
```

If provided, this function will be called with an `updaterFn` when `state.columnFilters` changes. This overrides the default internal state management, so you will need to persist the state change either fully or partially outside of the table.

### `enableColumnFilters`

```tsx
enableColumnFilters?: boolean
```

Enables/disables **all** column filters for the table. For option priority, see [Can-Filter Option Priority](../guides/filters#can-filter-option-priority).

### `manualGlobalFiltering`

```tsx
manualGlobalFiltering?: boolean
```

Disables the `getGlobalFilteredRowModel` from being used to filter data. This may be useful if your table needs to dynamically support both client-side and server-side filtering.

### `getColumnFilteredRowModel`

```tsx
getColumnFilteredRowModel?: (
  instance: TableInstance<TGenerics>
) => () => RowModel<TGenerics>
```

If provided, this function is called **once** per table instance and should return a **new function** which will calculate and return the row model for the table when it's filtered.

- For server-side filtering, this function is unnecessary and can be ignored since the server should already return the filtered row model.
- For client-side filtering, this function is required. A default implementation is provided via any table adapter's `{ getColumnFilteredRowModel }` export.

Example:

```tsx
import { getColumnFilteredRowModel } from '@tanstack/[adapter]-table'

useTable(table, {
  getColumnFilteredRowModel: getColumnFilteredRowModel(),
})
```

### `globalFilterFn`

```tsx
globalFilterFn?: FilterFn | keyof TGenerics['FilterFns'] | keyof BuiltInFilterFns
```

The filter function to use for global filtering.

Options:

- A `string` referencing a [built-in filter function](#filter-functions))
- A `string` referencing a custom filter function defined on the `filterFns` table option
- A [custom filter function](#filter-functions)

### `onGlobalFilterChange`

```tsx
onGlobalFilterChange?: OnChangeFn<GlobalFilterState>
```

If provided, this function will be called with an `updaterFn` when `state.globalFilter` changes. This overrides the default internal state management, so you will need to persist the state change either fully or partially outside of the table.

### `enableGlobalFilter`

```tsx
enableGlobalFilter?: boolean
```

Enables/disables the global filter for the table. For option priority, see [Can-Filter Option Priority](../guides/filters#can-filter-option-priority).

### `getGlobalFilteredRowModel`

```tsx
getGlobalFilteredRowModel?: (
  instance: TableInstance<TGenerics>
) => () => RowModel<TGenerics>
```

If provided, this function is called **once** per table instance and should return a **new function** which will calculate and return the row model for the table when it's globally filtered.

- For server-side filtering, this function is unnecessary and can be ignored since the server should already return the globally filtered row model.
- For client-side filtering, this function is required. A default implementation is provided via any table adapter's `{ getGlobalFilteredRowModel }` export.

Example:

```tsx
import { getGlobalFilteredRowModel } from '@tanstack/[adapter]-table'

useTable(table, {
  getGlobalFilteredRowModel: getGlobalFilteredRowModel(),
})
```

### `getColumnCanGlobalFilterFn`

```tsx
getColumnCanGlobalFilterFn?: (column: Column<TGenerics>) => boolean
```

If provided, this function will be called with the column and should return `true` or `false` to indicate whether this column should be used for global filtering.

## Table Instance API

### `getColumnAutoFilterFn`

```tsx
getColumnAutoFilterFn: (columnId: string) => FilterFn<TGenerics> | undefined
```

Returns an automatically calculated filter function for the column based off of the columns first known value.

### `getColumnFilterFn`

```tsx
getColumnFilterFn: (columnId: string) => FilterFn<TGenerics> | undefined
```

Returns the filter function (either user-defined or automatic, depending on configuration) for the columnId specified.

### `setColumnFilters`

```tsx
setColumnFilters: (updater: Updater<ColumnFiltersState>) => void
```

Sets or updates the `state.columnFilters` state.

### `setColumnFilterValue`

```tsx
setColumnFilterValue: (columnId: string, updater: Updater<any>) => void
```

Sets or updates the filter value for the columnId specified.

### `resetColumnFilters`

```tsx
resetColumnFilters: () => void
```

Resets the **column** filter state for the table.

### `getColumnCanColumnFilter`

```tsx
getColumnCanColumnFilter: (columnId: string) => boolean
```

Returns if the column with specified columnId can be filtered.

### `getColumnIsFiltered`

```tsx
getColumnIsFiltered: (columnId: string) => boolean
```

Returns if the column with the specified columnId is currently filtered.

### `getColumnFilterValue`

```tsx
getColumnFilterValue: (columnId: string) => unknown
```

Returns the current filter value of the column.

### `getColumnFilterIndex`

```tsx
getColumnFilterIndex: (columnId: string) => unknown
```

Returns the index (including `-1`) of the column filter with the specified columnId in the table's `state.columnFilters` array.

### `getPreColumnFilteredRowModel`

```tsx
getPreColumnFilteredRowModel: () => RowModel<TGenerics>
```

Returns the row model for the table before any **column** filtering has been applied.

### `getColumnFilteredRowModel`

```tsx
getColumnFilteredRowModel: () => RowModel<TGenerics>
```

Returns the row model for the table after **column** filtering has been applied.

### `setGlobalFilter`

```tsx
setGlobalFilter: (updater: Updater<any>) => void
```

Sets or updates the `state.globalFilter` state.

### `resetGlobalFilter`

```tsx
resetGlobalFilter: () => void
```

Resets the **global** filter state for the table.

### `getGlobalAutoFilterFn`

```tsx
getGlobalAutoFilterFn: (columnId: string) => FilterFn<TGenerics> | undefined
```

Currently, this function returns the built-in `includesString` filter function. In future releases, it may return more dynamic filter functions based on the nature of the data provided.

### `getGlobalFilterFn`

```tsx
getGlobalFilterFn: (columnId: string) => FilterFn<TGenerics> | undefined
```

Returns the global filter function (either user-defined or automatic, depending on configuration) for the table.

### `getColumnCanGlobalFilter`

```tsx
getColumnCanGlobalFilter: (columnId: string) => boolean
```

Returns if the column with specified columnId can be **globally** filtered.

### `getPreGlobalFilteredRowModel`

```tsx
getPreGlobalFilteredRowModel: () => RowModel<TGenerics>
```

Returns the row model for the table before any **global** filtering has been applied.

### `getGlobalFilteredRowModel`

```tsx
getGlobalFilteredRowModel: () => RowModel<TGenerics>
```

Returns the row model for the table after **global** filtering has been applied.