# jQuery Add/Remove Row Plugin

A lightweight, dependency-free (only requires jQuery) plugin that simplifies dynamic **row management** for HTML forms or lists.  
It lets you easily add and remove rows (form fields, groups, or elements) with automatic **reindexing**, customizable templates, and event hooks — all without breaking existing code.

---

## 🌟 Features

- ✅ Add and remove form rows dynamically  
- 🔢 Automatic reindexing of input names and row IDs  
- 🧩 Fully customizable row templates  
- ⚙️ Configurable options (max rows, field name prefix, etc.)  
- 🧠 Smart detection of remove targets (works even with nested DOMs)  
- 🪄 Built-in callbacks: `onAdd` and `onRemove`  
- 💡 ES module–ready (`export default $`)  

---

## 🧰 Requirements

- **jQuery** (v3.0 or later)
- Works with any frontend framework using plain HTML and JS.

---

## 📦 Installation

Simply include the plugin after loading jQuery.

### Option 1: Using `<script>` tag

```html
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<script src="addRemoveRowjQueryPlugins.js"></script>
```

### Option 2: Using ES Modules

```javascript
import $ from "./addRemoveRowjQueryPlugins.js";
```

---

## 🚀 Basic Usage

### HTML Structure

```html
<div id="rowsWrapper"></div>

<button id="addRowBtn" type="button">Add Row</button>
```

### JavaScript

```javascript
$('#rowsWrapper').remAddRow({
  addBtn: '#addRowBtn',
});
```

That’s it! You now have a working add/remove system.

---

## 🧩 Options

You can pass options to customize the behavior:

| Option | Type | Default | Description |
|--------|------|----------|-------------|
| `addBtn` | `string \| jQuery` | `null` | Selector for the “Add” button (**required**) |
| `maxFields` | `number` | `10` | Maximum number of rows allowed |
| `removeSelector` | `string` | `.row_remove` | Selector for the remove button inside each row |
| `fieldName` | `string` | `"rows"` | Base name for form field groups (`rows[0], rows[1], ...`) |
| `rowIdPrefix` | `string` | `"row"` | Prefix for each row’s `id` (e.g. `row_0`, `row_1`) |
| `reindexOnRemove` | `boolean` | `true` | Whether to reindex fields and IDs after removing a row |
| `rowTemplate(i, name)` | `function` | *(default template)* | Function returning the HTML for a new row |
| `startCounter` | `number` | `0` | Optional offset for numbering |
| `onAdd(i, $row)` | `function` | `() => {}` | Callback fired after a row is added |
| `onRemove(i)` | `function` | `() => {}` | Callback fired after a row is removed |

---

## 🧱 Default Row Template

By default, each new row looks like this:

```html
<div class="row-box" id="row_0">
  <span data-row-index>Row #1</span>
  <input type="text" name="rows[0]" />
  <button type="button" class="row_remove" data-id="0">Remove</button>
</div>
```

You can fully customize it with the `rowTemplate` option:

```javascript
$('#rowsWrapper').remAddRow({
  addBtn: '#addRowBtn',
  rowTemplate: (i, name) => `
    <div class="custom-row" id="custom_${i}">
      <label>Item ${i + 1}</label>
      <input type="text" name="${name}[${i}]" placeholder="Enter text..." />
      <button class="btn btn-danger row_remove" data-id="${i}">✖</button>
    </div>
  `,
});
```

---

## 🔁 Callbacks

### onAdd
Called immediately after a new row is added.

```javascript
onAdd: (index, $row) => {
  console.log("Added row:", index);
}
```

### onRemove
Called right after a row is removed.

```javascript
onRemove: (index) => {
  console.log("Removed row:", index);
}
```

---

## 🧮 Reindexing Logic

Whenever a row is added or removed:
- Each row gets an updated ID (`row_0`, `row_1`, ...).
- Each input name is rewritten to maintain contiguous indices:
  - `rows[0][name]`, `rows[1][name]`, etc.
- The `data-id` of each remove button is synchronized with the new index.

This ensures compatibility with Laravel-style form arrays or any indexed input group.

---

## 🧠 How It Works Internally

1. **Initialization**  
   - Reads your config options and binds the Add/Remove event listeners.  
   - Reindexes any pre-existing rows on page load.

2. **Adding Rows**  
   - Clones the template, inserts it, reindexes all rows, and calls `onAdd`.

3. **Removing Rows**  
   - Finds the correct row (using `data-id` or fallback detection).  
   - Removes it, reindexes remaining rows, and calls `onRemove`.

4. **Max Fields Control**  
   - Disables the “Add” button when `maxFields` is reached.

---

## 🧪 Example with Form Submission

```html
<form id="dynamicForm">
  <div id="rowsWrapper"></div>
  <button id="addRowBtn" type="button">Add Row</button>
  <button type="submit">Submit</button>
</form>

<script>
$('#rowsWrapper').remAddRow({
  addBtn: '#addRowBtn',
  fieldName: 'users',
  onAdd: (i, $row) => console.log('Added:', i),
  onRemove: (i) => console.log('Removed:', i)
});

$('#dynamicForm').on('submit', function (e) {
  e.preventDefault();
  console.log($(this).serializeArray());
});
</script>
```

---

## 🧩 Live Demo Example

You can test it directly on CodePen or JSFiddle:  
👉 [Live Demo (CodePen)](https://codepen.io/pen?template=ExVwVyw)

---

## 💻 Laravel Blade Integration Example

```blade
<form method="POST" action="{{ route('users.store') }}">
  @csrf
  <div id="rowsWrapper">
    <!-- Existing rows can go here -->
  </div>

  <button id="addRowBtn" type="button" class="btn btn-primary">Add Row</button>
  <button type="submit" class="btn btn-success">Save</button>
</form>

@push('scripts')
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<script src="{{ asset('js/addRemoveRowjQueryPlugins.js') }}"></script>
<script>
$('#rowsWrapper').remAddRow({
  addBtn: '#addRowBtn',
  fieldName: 'users',
  rowTemplate: (i, name) => `
    <div class="mb-2 p-2 border rounded" id="user_${i}">
      <label>User ${i + 1}</label>
      <input type="text" name="${name}[${i}][name]" class="form-control mb-1" placeholder="Full name">
      <input type="email" name="${name}[${i}][email]" class="form-control mb-1" placeholder="Email">
      <button type="button" class="btn btn-sm btn-danger row_remove" data-id="${i}">Remove</button>
    </div>
  `
});
</script>
@endpush
```

---

## 🧾 License

MIT License © Your Name

---

## 💬 Credits

Developed by **Your Name**  
Inspired by classic jQuery dynamic form techniques and refactored for modern ES module support.

---

> 🧩 *Simple, flexible, and built for real-world form handling.*
