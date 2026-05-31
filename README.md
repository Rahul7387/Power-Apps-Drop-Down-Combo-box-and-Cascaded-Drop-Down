# 📋 Power Apps Dropdown Guide

> A complete, beginner-friendly reference for building dropdowns and combo boxes in Power Apps — covering all four population methods, delegation, cascading dropdowns, and best practices.

---

## Table of Contents

- [What is a Dropdown in Power Apps?](#what-is-a-dropdown-in-power-apps)
- [Method 1 — Hard-Coded Items](#method-1--hard-coded-items)
- [Method 2 — Connect Directly to a List](#method-2--connect-directly-to-a-list)
- [Method 3 — Distinct() from a Table](#method-3--distinct-from-a-table)
- [Method 4 — Choice Column via Choices()](#method-4--choice-column-via-choices)
- [Dropdown vs Combo Box](#dropdown-vs-combo-box)
- [Cascading Dropdowns](#cascading-dropdowns)
- [⚠️ Delegation — What Every Builder Must Know](#️-delegation--what-every-builder-must-know)
- [Best Practices Summary](#best-practices-summary)
- [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## What is a Dropdown in Power Apps?

A **Dropdown** control lets users pick one value from a list. Power Apps also has a **Combo Box** control that supports searching and multi-select.

Both controls share one key property:

| Property | What it does |
|---|---|
| `Items` | The data source or formula that feeds the list |
| `Selected` | The currently selected record |
| `Selected.Value` | The text of the selected item |

You set the `Items` property using any of the four methods below.

---

## Method 1 — Hard-Coded Items

### What it is
You type the list options directly into the `Items` property. No data source required.

### When to use it
- Short, fixed lists that almost never change (e.g., Yes/No, Priority levels)
- Prototyping or demos
- Offline apps where no data connection is available

### How to set it up

**In the Items property of your Dropdown, enter:**

```
// Simple text list
["", "Active", "Inactive", "Pending"]

// Using Table() — gives you more control over column names
Table(
    {Value: "Active"},
    {Value: "Inactive"},
    {Value: "Pending"}
)
```

The empty string `""` at the start creates a blank "please select" option.

### Reading the selected value

```
// In a label, patch, or condition:
Dropdown1.Selected.Value
```

### Real-world example

You have a form where users log a support ticket. The priority field will always be Low, Medium, or High — it never changes. Hard-code it:

```
["", "Low", "Medium", "High"]
```

### ✅ Pros
- Zero setup — no SharePoint list, no data source
- Works offline
- Fastest option to build

### ❌ Cons
- To change the options, a developer must edit the app and **republish** it
- Hard to reuse across multiple apps
- No audit trail of who changed the options

---

## Method 2 — Connect Directly to a List

### What it is
You point the `Items` property at a SharePoint list or Dataverse table. Power Apps fetches the rows and displays them.

### When to use it
- The list is short (under 500 rows)
- Non-technical users (e.g., HR, admins) need to maintain the options without republishing the app
- You have a dedicated "Config" or "Lookup" list in SharePoint

### How to set it up

1. Add your SharePoint list as a data source in Power Apps (click **Data** → **Add data**)
2. In the `Items` property of your Dropdown, enter the list name:

```
// Entire list
StatusConfig

// With a specific column shown
// (Set the "Value" property of the dropdown to the column name)
StatusConfig
// Then set: Value = "StatusName"
```

### Reading the selected value

```
// Get the text value
Dropdown1.Selected.StatusName

// Get the whole record (useful for patching related columns)
Dropdown1.Selected
```

### Real-world example

You have a SharePoint list called `DepartmentList` with a column `DeptName`. Non-technical admins can add or remove departments any time.

```
// Items property:
DepartmentList

// Value property of the dropdown:
"DeptName"
```

When an admin adds a new row to `DepartmentList` in SharePoint, it instantly appears in the app — no republish needed.

### ✅ Pros
- Business users can update options without touching the app
- No app republish required for list changes
- Audit trail available via SharePoint list history

### ❌ Cons
- Loads **all columns** of every row — can be slow or hit delegation limits on large lists
- Duplicate entries appear if rows are not managed carefully
- Requires a data connection (won't work offline)

---

## Method 3 — Distinct() from a Table

### What it is
You use the `Distinct()` function to pull **unique values** from one column of an existing table. This avoids needing a separate config list.

### When to use it
- Your source data table (e.g., a Projects list) already contains the values you need
- You want to avoid maintaining a separate lookup list
- You need to deduplicate values (e.g., show each Category only once even if 50 records share it)

### How to set it up

```
// Basic — unique values from one column
Distinct(ProjectsList, Category)

// With alphabetical sorting (recommended)
Sort(
    Distinct(ProjectsList, Category),
    Result,
    SortOrder.Ascending
)

// Note: Distinct() always returns a column named "Result"
// So always use .Result or sort on "Result"
```

### Reading the selected value

```
// The column from Distinct() is always called "Result"
Dropdown1.Selected.Result
```

### Real-world example

Your `ProjectsList` has a `Region` column with values like APAC, EMEA, NA repeated across hundreds of rows. You want a dropdown that shows each region once:

```
// Items property:
Sort(Distinct(ProjectsList, Region), Result)

// Read selected value:
Dropdown1.Selected.Result
```

### ✅ Pros
- Reuses an existing table — no extra list to maintain
- Automatically deduplicates values
- Dynamic — reflects the actual data in your table

### ❌ Cons
- **Delegation issue** — `Distinct()` is not delegable on most connectors (see [Delegation section](#️-delegation--what-every-builder-must-know))
- Column is always named `Result`, which can be confusing
- If source data has inconsistent casing ("APAC" vs "apac"), duplicates appear

---

## Method 4 — Choice Column via Choices()

### What it is
You create a **Choice** column directly in SharePoint or a **Choices** column in Dataverse. Power Apps reads the defined options using the `Choices()` function.

### When to use it
- You want a single source of truth that controls both the SharePoint list validation AND the app dropdown
- You want the dropdown to exactly mirror what SharePoint itself enforces
- Working with Dataverse where Choice columns are a first-class feature

### How to set it up

**Step 1 — In SharePoint:** Create a Choice column on your list (e.g., `Status` with options Active, Inactive, Pending).

**Step 2 — In Power Apps:**

```
// Items property:
Choices([@ProjectsList].Status)

// Full syntax with the list name:
Choices(ProjectsList.Status)
```

> **Note:** You must use the `[@ListName]` syntax when the list name could be ambiguous, or when Power Apps doesn't auto-resolve it.

### Reading the selected value

```
// Works the same as Method 1 — returns the choice text
Dropdown1.Selected.Value
```

### Real-world example

Your `TicketsList` has a `Priority` choice column with options Low, Medium, High, Critical. The same options are enforced in the SharePoint list view. Using `Choices()` means both the app and the list always stay in sync:

```
// Items property:
Choices([@TicketsList].Priority)
```

If a SharePoint admin adds "Critical" to the column, it automatically appears in your app dropdown.

### ✅ Pros
- **Single source of truth** — SharePoint/Dataverse owns the valid values
- No delegation issues — `Choices()` is fully delegable
- Consistent between the app, the list, and any other tools using that column
- No extra config list to maintain

### ❌ Cons
- Only works with Choice/Choices column type — can't use on a plain text column
- Updating choices requires SharePoint admin access
- Can only be used in the context of the parent list

---

## Dropdown vs Combo Box

Both controls let users pick from a list, but they behave differently.

| Feature | Dropdown | Combo Box |
|---|---|---|
| User can type to search | ❌ No | ✅ Yes |
| Allow multiple selections | ❌ No (single only) | ✅ Yes |
| Selected value property | `Dropdown1.Selected.Value` | `ComboBox1.Selected.Value` |
| Multiple selections property | N/A | `ComboBox1.SelectedItems` |
| Built-in search/filter | ❌ No | ✅ Yes |
| Best for cascading dropdowns | ✅ Yes | Possible but complex |
| Performance on large lists | Moderate | Better (search reduces visible items) |
| Allow empty/blank | ✅ `AllowEmptySelection = true` | Clear search text resets selection |

### When to use Dropdown
- Short, fixed list (fewer than 20–30 options)
- User must pick exactly one value
- Building a cascading dropdown chain

### When to use Combo Box
- Long lists where search helps (e.g., 100+ employees, 500+ products)
- User may need to select multiple values (e.g., assign multiple tags)
- You want search-as-you-type filtering

### Reading multiple selections from a Combo Box

```
// Single selection
ComboBox1.Selected.Value

// Multiple selections — join all into a single string
Concat(ComboBox1.SelectedItems, Value & ", ")

// Check if a specific value was selected
"Manager" in ComboBox1.SelectedItems.Value

// Count how many were selected
CountRows(ComboBox1.SelectedItems)
```

### Setting a default selection

```
// Dropdown — pre-select "Active"
DefaultSelectedItems: Filter(StatusList, Value = "Active")

// Combo Box — pre-select multiple
DefaultSelectedItems: Filter(RoleList, Value in ["Admin", "Editor"])
```

---

## Cascading Dropdowns

A **cascading dropdown** (also called dependent dropdown) filters the second list based on what the user picks in the first. The most common example is Country → City.

### How it works

```
User picks Country → City dropdown filters to only cities in that country
```

### Step-by-step setup

#### Step 1 — Prepare your data source

Create a SharePoint list (e.g., `LocationConfig`) with two columns:

| Country | City |
|---|---|
| India | Mumbai |
| India | Delhi |
| India | Pune |
| USA | New York |
| USA | Seattle |
| Germany | Berlin |

#### Step 2 — Add the parent dropdown (Country)

Insert a Dropdown, name it `ddCountry`. Set its `Items` property:

```
// Get unique country values, sorted A–Z
Sort(
    Distinct(LocationConfig, Country),
    Result,
    SortOrder.Ascending
)
```

#### Step 3 — Add the child dropdown (City)

Insert a second Dropdown, name it `ddCity`. Set its `Items` property:

```
// Filter cities by what the user picked in ddCountry
Sort(
    Distinct(
        Filter(LocationConfig, Country = ddCountry.Selected.Result),
        City
    ),
    Result,
    SortOrder.Ascending
)
```

> **Important:** If `ddCountry` uses Method 1 or Method 2 (not `Distinct()`), the selected value property changes. See the note below.

#### Step 4 — Reset the child when the parent changes

In the `OnChange` property of `ddCountry`, add:

```
Reset(ddCity)
```

Without this, if a user picks India → Mumbai, then changes to USA, the City dropdown still shows Mumbai. `Reset()` clears it.

#### Step 5 — Save both values

```
// In your Patch() or SubmitForm() button:
Patch(
    TargetList,
    Defaults(TargetList),
    {
        Country: ddCountry.Selected.Result,
        City: ddCity.Selected.Result,
        // other fields...
    }
)
```

### Three-level cascade (Country → State → City)

```
// ddState Items:
Sort(
    Distinct(
        Filter(LocationConfig, Country = ddCountry.Selected.Result),
        State
    ),
    Result
)

// ddCity Items:
Sort(
    Distinct(
        Filter(LocationConfig, State = ddState.Selected.Result),
        City
    ),
    Result
)

// ddCountry OnChange — reset all children:
Reset(ddState);
Reset(ddCity)

// ddState OnChange — reset its children:
Reset(ddCity)
```

### Adjusting the property name based on method

The child filter formula changes depending on how the parent dropdown is populated:

| Parent uses | Selected value syntax |
|---|---|
| Method 1 (hard-coded) | `ddParent.Selected.Value` |
| Method 2 (direct list) | `ddParent.Selected.YourColumnName` |
| Method 3 (Distinct) | `ddParent.Selected.Result` |
| Method 4 (Choices) | `ddParent.Selected.Value` |

---

## ⚠️ Delegation — What Every Builder Must Know

### What is delegation?

Power Apps works with external data sources like SharePoint and Dataverse. When you write a formula like `Filter()` or `Sort()`, Power Apps can either:

- **Delegate** the work to the server — the server filters/sorts the data and returns only what you need ✅
- **Do it locally** — Power Apps downloads up to **500 rows** (default, max 2000) and filters/sorts on the device ⚠️

If your list has 5,000 rows and Power Apps can only download 500, your filter results will be **incomplete and wrong** — and Power Apps will show a **yellow warning triangle** in the formula bar.

### Which functions are delegable?

| Function | SharePoint | Dataverse | Notes |
|---|---|---|---|
| `Filter()` with `=`, `<`, `>` | ✅ | ✅ | Works on indexed columns |
| `Sort()` | ✅ | ✅ | Works on most columns |
| `Distinct()` | ❌ | ❌ | **Never delegable** |
| `Search()` | ✅ | ✅ | Text search only |
| `StartsWith()` | ✅ | ✅ | |
| `CountRows()` | ❌ | ✅ | Not delegable in SharePoint |
| `In` operator | ❌ | ✅ | |

> **Rule of thumb:** If you see a yellow delegation warning, assume your results may be incomplete on large data sets.

### The delegation problem with Method 3 (Distinct)

`Distinct()` is **never delegable**. This means:

```
// ⚠️ This only processes the first 500 rows from ProjectsList
// If row 501 has a new Category, it will NOT appear in the dropdown
Distinct(ProjectsList, Category)
```

**Solutions:**

**Option A — Use a dedicated config list instead (Method 2)**

Keep a separate `CategoryConfig` list with one row per category. This avoids the delegation problem entirely.

**Option B — Increase the data row limit (temporary fix)**

In Power Apps Studio → App Settings → Advanced Settings → increase "Data row limit" to 2000. This still has a cap and is not a long-term solution.

**Option C — Use Dataverse with server-side views**

Dataverse supports delegation for more functions than SharePoint. If your list is large, Dataverse is the better backend.

### The delegation problem with Method 2 (direct list with Filter)

```
// ⚠️ "in" operator is not delegable in SharePoint
Filter(ProjectsList, Status in ["Active", "Pending"])

// ✅ Use separate Or conditions instead
Filter(ProjectsList, Status = "Active" || Status = "Pending")
```

### How to handle delegation for cascading dropdowns

Cascading dropdowns often use `Filter()` + `Distinct()` — both called together. The `Distinct()` makes the whole thing non-delegable.

```
// ⚠️ Not delegable — risky on large tables
Distinct(Filter(LocationConfig, Country = ddCountry.Selected.Result), City)
```

**Best practice for cascading:** Keep your config list small (it's a lookup list, not a transactional list). If the `LocationConfig` list has fewer than 500 rows, delegation is not a concern in practice.

If the list is large, switch to:

```
// Delegable alternative using a dedicated small config table
Filter(CityConfig, CountryID = ddCountry.Selected.ID)
// Where CityConfig is a small list (under 500 rows) with a foreign key
```

---

## Best Practices Summary

### General

| Practice | Why |
|---|---|
| Always include a blank `""` as the first item | Prevents accidental default selection in forms |
| Name your controls clearly (`ddCountry`, not `Dropdown1`) | Makes formulas readable and maintainable |
| Use a dedicated config list for Method 2/3 | Separates lookup data from transactional data |
| Test with more than 500 rows if using `Distinct()` | Catches delegation gaps early |
| Use `Choices()` (Method 4) for Choice columns | Keeps SharePoint and app in sync automatically |

### For cascading dropdowns

| Practice | Why |
|---|---|
| Always call `Reset(childDropdown)` in parent's `OnChange` | Prevents stale/mismatched selections |
| Reset all children, not just the immediate one | A 3-level cascade needs two resets in the top dropdown |
| Keep config lists under 500 rows | Avoids delegation issues with `Distinct()` |
| Use `Sort(..., Result)` around `Distinct()` | Users expect alphabetical order |

### For performance

| Practice | Why |
|---|---|
| Prefer `Choices()` over `Distinct()` for choice-type columns | `Choices()` is delegable; `Distinct()` is not |
| Avoid loading the full source list just for a dropdown | Use a separate small config list for lookup values |
| Cache dropdown data in a collection on app start | Reduces repeated network calls |

```
// Cache on App OnStart — runs once, populates a local collection
ClearCollect(
    colDepartments,
    Sort(Distinct(DepartmentList, DeptName), Result)
);

// Then use the collection in your dropdown — no delegation risk
Items: colDepartments
```

---

## Quick Reference Cheat Sheet

```
// METHOD 1 — Hard-coded
Items: ["", "Option A", "Option B", "Option C"]
Read: Dropdown1.Selected.Value

// METHOD 2 — Direct list
Items: YourSharePointList
Read: Dropdown1.Selected.YourColumnName

// METHOD 3 — Distinct from table
Items: Sort(Distinct(YourList, ColumnName), Result)
Read: Dropdown1.Selected.Result

// METHOD 4 — Choice column
Items: Choices([@YourList].ChoiceColumn)
Read: Dropdown1.Selected.Value

// CASCADING — child dropdown
Items: Sort(Distinct(Filter(ConfigList, ParentCol = ddParent.Selected.Result), ChildCol), Result)
OnChange of parent: Reset(ddChild)

// COMBO BOX — read multiple selections
Concat(ComboBox1.SelectedItems, Value & ", ")

// CACHE on App OnStart (avoids delegation + repeated calls)
ClearCollect(colOptions, Sort(Distinct(BigList, Category), Result))
```

---

## Contributing

Found an error or want to add a method? Open a pull request or raise an issue. All examples are tested against Power Apps.

---

*Maintained by the Power Platform community. Pull requests welcome.*
