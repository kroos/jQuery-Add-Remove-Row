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
$('#serial_wrap').remAddRow({
  addBtn: '#serial_add',
});
```

That’s it! You now have a working add/remove system.

---

## 🧩 Options

You can pass options to customize the behavior:

| Option | Type | Default | Description |
|--------|------|----------|-------------|
| `addBtn` | `string \| jQuery` | `null` | Selector for the “Add” button (**required**) |
| `maxRows` | `number` | `10` | Maximum number of rows allowed |
| `rowSelector` | `string` | `rowSkill` | A word for selector for each row. Can be both(class or id) |
| `fieldName` | `string` | `"persons"` | Base name for form field groups (`persons[0][name], persons[1][name], ...`) |
| `removeSelector` | `string` | `"row"` | Selector class to remove a row |
| `reindexOnRemove` | `boolean` | `return true` | Reindex all rows after removing a row (`return true`) |
| `rowTemplate` | `function` | `(i, name) => {}` *(default template)* | Function returning the HTML for a new row |
| `startCounter` | `number` | `0` | Optional offset for numbering |
| `onAdd` | `function` | `(i, event, $row, name) => {}` | Callback fired after a row is added |
| `onRemove` | `function` | `(i, event, $row, name) => {}` | Callback fired before a row is removed |
| `reindexOnRemove` | `boolean` | `return false` | Halt remove a row and reindexing (`return false`) |
| `reindexRowName` | `array` | `['name', 'data-bv-field', 'data-bv-for']` | An array for attributes with ONLY `${name}[${i}][name]` (`data-bv-for="person[0][name]"`) pattern to reindex after a row removal |
| `reindexRowID` | `array` | `['id', 'for']` | An array for attributes with ONLY `userdefine_${i}` (`id="skill_3"`) pattern to reindex after a row removal |
| `reindexRowIndex` | `array` | `['data-index']` | An array for attributes with ONLY `${i}` (`data-index="7"`) pattern to reindex after a row removal |

---

## 🧱 Default Row Template

By default, each new row looks like this:

```html
<div id="${settings.rowSelector}_${index}" class="row m-0 ${settings.rowSelector}">
  <input type="hidden" name="${settings.fieldName}[${index}][id]" value="">

  <div class="form-group row m-0">
    <label for="name_${index}" class="col-form-label col-sm-4">Name : </label>
    <div class="col-sm-8 my-auto">
      <input type="text"
           name="${settings.fieldName}[${index}][name]"
           value=""
           id="name_${index}"
           class="form-control form-control-sm"
           placeholder="Name">
    </div>
  </div>

  <div class="form-group row m-0">
    <label for="skill_${index}" class="col-form-label col-sm-4">Skill : </label>
    <div class="col-sm-8 my-auto">
      <input type="text"
           name="${settings.fieldName}[${index}][skill]"
           value=""
           id="skill_${index}"
           class="form-control form-control-sm"
           placeholder="Skill">
    </div>
  </div>

  <div class="col-sm-4 m-0">
    <button type="button"
        class="btn btn-sm btn-outline-danger ${settings.removeSelector}"
        data-index="${index}">Remove</button>
  </div>
</div>
```

You can fully customize it with the `rowTemplate` option:

```javascript
$('#rowsWrapper').remAddRow({
  addBtn: '#addRowBtn',
  rowSelector: 'custom-row',
  removeSelector: 'row_remove',
  rowTemplate: (i, name) => `
    <div class="custom-row" id="custom-row_${i}">
      <label>Item</label>
      <input type="text" name="${name}[${i}]" placeholder="Enter text..." />
      <button class="btn btn-danger row_remove" data-index="${i}">✖</button>
    </div>
  `,
});
```

```javascript
$("#applicants_wrap").remAddRow({
    addBtn: "#applicants_add",
    maxFields: 5,
    rowSelector: 'applicant',
    removeSelector: ".applicant_remove",
    fieldName: "applicants",
    rowIdPrefix: "applicant",
    rowTemplate: (i, name) => `
        <div class="applicant col-sm-12 row m-3" id="applicant_${i}">
            <input type="hidden" name="${name}[${i}][id]" value="">
            <div class="col-sm-7 m-0 p-1">

                <div class="col-sm-12 m-1 row">
                    <x-input-label for="nama_${i}" class="col-sm-3" :value="__('Nama : ')" />
                    <div class="col-sm-9">
                        <select id="nama_${i}" name="${name}[${i}][nama]" class="form-select form-select-sm @error('applicants.*.nama') is-invalid @enderror" placeholder="Please choose"></select>
                        @error('applicants.*.nama')
                        <div class="invalid-feedback">
                            {{ $message }}
                        </div>
                        @enderror
                    </div>
                </div>

            <div class="col-sm-12 m-2">
                <button type="button" class=" btn btn-sm btn-danger applicant_remove" data-id="${i}"><i class="fa-regular fa-trash-can fa-beat"></i>&nbsp;Padam Pemohon</button>
            </div>
        </div>
    `,
    onAdd: (i, e, $r, name) => {
        // console.log('Applicants added', i, $r)
        selectname(i);
    },
    onRemove: (i, event, $row, name) => {
        const idv = $row.find(`[name="${name}[${i}][id]"]`).val();
        console.log(idv);
        if (!idv) {
            return true;  // ✅ ALLOW removal (reindexing will happen)
        }
        swal.fire({
            title: 'Delete applicant?',
            text: 'This action cannot be undone.',
            icon: 'warning',
            showCancelButton: true,
            confirmButtonColor: '#d33',
            confirmButtonText: 'Yes, delete it!'
        }).then(result => {
            if (result.isConfirmed) {
                $.ajax({
                    url: `/applicants/${idv}`,
                    type: 'DELETE',
                    data: {
                      _token: $('meta[name="csrf-token"]').attr('content')
                    },
                    success: response => {
                        swal.fire('Deleted!', response.message, 'success');
                        return true;  // ✅ ALLOW removal (reindexing will happen)
                    },
                    error: xhr => {
                        swal.fire('Error', 'Failed to delete applicant', 'error');
                    }
                });
            }
        });
    }
});

```


---

## 🔁 Callbacks

### onAdd
Called immediately after a new row is added.

```javascript
onAdd: (index, event, $row, name) => {
  console.log("Added row:", index);
}
```

### onRemove
Called right BEFORE a row is removed.

```javascript
onRemove: (index, event, $target, fieldName) => {
  console.log("Removed row:", index, event, $target, fieldName);
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
  onAdd: (i, e, $row, name) => console.log('Added:', i),
  onRemove: (i, e, $row, name) => console.log('Removed:', i, e)
});

$('#dynamicForm').on('submit', function (e) {
  e.preventDefault();
  console.log($(this).serializeArray());
});
</script>
```

---

## 💻 Laravel Blade Integration Example

```blade
<form method="POST" action="{{ route('users.store') }}">
  @csrf
  <div id="rowsWrapper">
    <!-- Existing rows can go here via old() -->
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
  rowSelector: 'user',
  rowTemplate: (i, name) => `
    <div class="user mb-2 p-2 border rounded" id="user_${i}">
      <label>User ${i + 1}</label>
      <input type="text" name="${name}[${i}][name]" class="form-control mb-1" placeholder="Full name">
      <input type="email" name="${name}[${i}][email]" class="form-control mb-1" placeholder="Email">
      <button type="button" class="btn btn-sm btn-danger row_remove" data-index="${i}">Remove</button>
    </div>
  `
});
</script>
@endpush
```

---

## 🧾 License

MIT License © kroos

---

## 💬 Credits

Developed by **kroos**
Inspired by classic jQuery dynamic form techniques and refactored for modern ES module support.

---

> 🧩 *Simple, flexible, and built for real-world form handling.*
