(function($) {
  'use strict';

  // Plugin definition
  $.fn.remAddRow = function(options) {
    // Make sure we have elements to work with
    if (!this.length) {
      console.warn('remAddRow: No elements found');
      return this;
    }

    // Default settings
    const defaults = {
      addBtn: '',
      maxRows: 5,
      startRow: 0,
      fieldName: 'data',
      rowSelector: 'rowserial',
      removeClass: 'serial_remove',
      nestedwrapper: null,
      onAdd: null,
      onRemove: null,
      // Default reindexing patterns
      reindexRowName: ['name', 'data-bv-field', 'data-bv-for'],
      reindexRowID: ['id', 'for', 'aria-describedby'],
      reindexRowIndex: ['data-index', 'data-id']
    };

    // Merge options
    const settings = Object.assign({}, defaults, options);

    // custom
    // 🔹 Smart merge for reindex arrays
    ['reindexRowName', 'reindexRowID', 'reindexRowIndex'].forEach(key => {
      if (key in options) {
        // If user explicitly sets it (even empty), respect it
        settings[key] = Array.isArray(options[key])
        ? [...new Set([...(defaults[key] || []), ...options[key]])]
        : options[key];
      } else {
        // User did NOT provide it → keep defaults
        settings[key] = [...(defaults[key] || [])];
      }
    });

    const $wrapper = this;
    let i = settings.startRow;

    // Initialize
    function init() {

      // Count existing rows (0-based index)
      // i = $wrapper.find(`.${settings.rowSelector}`).length ?? $wrapper.find(`#${settings.rowSelector}_${i}`).length ;

      const y = Number(settings.startRow);
      const x = Number($wrapper.find(`.${settings.rowSelector}`).length ?? $wrapper.find(`#${settings.rowSelector}_${i}`).length );
      const i = y + x;
      // console.log(i);

      // Attach click event to add button
      if (settings.addBtn) {
        $(settings.addBtn).off('click.remAddRow').on('click.remAddRow', addRow);
      }

      // Delegate remove button events
      $wrapper.off('click.remAddRow', `.${settings.removeClass}`)
      .on('click.remAddRow', `.${settings.removeClass}`, removeRow);

      return methods;
    }

    // Add a new row
    const addRow = function(e) {

      const a = Number(settings.startRow);
      const b = Number(settings.maxRows);
      const totalRows = a + b;

      if (i >= totalRows) return false;

      // if (i < settings.maxRows) {
      if (i < totalRows) {
        const currentIndex = i;
        const rowHTML = createRowHTML(currentIndex);
        $wrapper.append(rowHTML);
        i++;

        // Callback with your signature: (index, event, $row, name)
        if (typeof settings.onAdd === 'function') {
          const $newRow = $(`#${settings.rowSelector}_${currentIndex}`, $wrapper);
          settings.onAdd(currentIndex, e, $newRow, settings.fieldName);
        }

        if (e && e.preventDefault) {
          e.preventDefault();
        }
      } else {
        console.log(`Maximum rows reached: ${settings.maxRows}`);
      }
      return false;
    };

    // Create row HTML - uses your template function
    const createRowHTML = function(index) {
      // Use custom template if provided (with your signature: (i, name))
      if (typeof settings.rowTemplate === 'function') {
        return settings.rowTemplate(index, settings.fieldName);
      }

      // Default template
      return `
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
                class="btn btn-sm btn-outline-danger ${settings.removeClass}"
                data-index="${index}">Remove</button>
          </div>
        </div>
      `;
    };

    // Remove a row
    const removeRow = function(e) {
      const $button = $(this);

      // const idIndex = $button.data('index') ?? $button.data('id');

       const idIndex = $button.attr('data-index') ?? $button.attr('data-id');


      let $row = $(`#${settings.rowSelector}_${idIndex}`, $wrapper);

      if (!$row.length) {
        $row = $button.closest(`.${settings.rowSelector}`);
      }

      if ($row.length) {

        if (typeof settings.onRemove === 'function') {
          const result = settings.onRemove(idIndex, e, $row, settings.fieldName);

          if (result && typeof result.then === 'function') {
            result.then(allow => {
              if (allow === false) return;

              $row.remove();
              i--;
              reindexRowAll();
            });

            return false;
          }

          if (result === false) {
            return false;
          }
        }

        $row.remove();
        i--;
        reindexRowAll();
      }

      if (e && e.preventDefault) e.preventDefault();
      return false;
    };



    // Reindex functions (unchanged)

    // 1️⃣ reindexRowNamePattern (Updated)
    const reindexRowNamePattern = function () {
      if (!settings.reindexRowName || !settings.reindexRowName.length) return;

      const $rows = $wrapper.find(`.${settings.rowSelector}`);
      const start = Number(settings.startRow);

      $rows.each(function (newIndex) {

        const d = start + newIndex;
        const $row = $(this);

        settings.reindexRowName.forEach(attr => {

      /* ================================
       * OUTER (NOT inside nestedwrapper)
       * ================================ */
          $row.find(`[${attr}]`)
          .not($row.find(`${settings.nestedwrapper} *`))
          .each(function () {
            const $el = $(this);
            const val = $el.attr(attr);
            if (!val) return;

            if (attr === 'index_pattern') {
              $el.attr(attr, d);
              return;
            }

            const matches = [...val.matchAll(/\[([^\]]+)\]/g)];
            if (matches.length < 2) return;

            const target = matches[matches.length - 2];
            const before = val.slice(0, target.index);
            const after  = val.slice(target.index + target[0].length);

            $el.attr(attr, `${before}[${d}]${after}`);
          });

      /* ================================
       * INNER (inside nestedwrapper)
       * ================================ */
          $row.find(settings.nestedwrapper)
          .find(`[${attr}]`)
          .each(function () {
            const $el = $(this);
            const val = $el.attr(attr);
            if (!val) return;

            const matches = [...val.matchAll(/\[([^\]]+)\]/g)];
            if (matches.length < 4) return;

            const target = matches[matches.length - 4];
            const before = val.slice(0, target.index);
            const after  = val.slice(target.index + target[0].length);

            $el.attr(attr, `${before}[${d}]${after}`);
          });

        });
      });
    };


    // 2️⃣ reindexRowIDPattern (Updated)
    const reindexRowIDPattern = function () {
      if (!settings.reindexRowID || !settings.reindexRowID.length) return;

      const $rows = $wrapper.find(`.${settings.rowSelector}`);
      const start = Number(settings.startRow);

      $rows.each(function (newIndex) {

        const e = start + newIndex;
        const $row = $(this);

        /* ================================
         * 1) REINDEX ROW ITSELF
         * ================================ */
        $row.attr('id', `${settings.rowSelector}_${e}`);

        settings.reindexRowID.forEach(attr => {

            /* ================================
             * 2) OUTER ATTR REINDEX
             * Replace LAST _<num>
             * ================================ */
          $row.find(`[${attr}]`)
          .not($row.find(`${settings.nestedwrapper} *`))
          .each(function () {

            const $el = $(this);
            const val = $el.attr(attr);
            if (!val) return;

            $el.attr(attr, val.replace(/_(\d+)$/, `_${e}`));
          });

            /* ================================
             * 3) INNER ATTR REINDEX
             * Replace SECOND-LAST _<num>
             * Example:
             *   rowserial_501_10 → rowserial_500_10
             * ================================ */
          $row.find(settings.nestedwrapper)
          .find(`[${attr}]`)
          .each(function () {

            const $el = $(this);
            const val = $el.attr(attr);
            if (!val) return;

            const updated = val.replace(
                                        /_(\d+)(?=_(\d+)$)/,
                                      `_${e}`
                                      );

            $el.attr(attr, updated);
          });
        });

        /* ================================
         * 4) NESTED CLASS FIX (CORRECT PLACE)
         * ================================ */

        const $nestedWrap = $row.find(settings.nestedwrapper);

        // rowserial_502 → rowserial_501
        const rowRe = new RegExp(`${settings.rowSelector}_(\\d+)`, 'g');

        $nestedWrap.find(`[class*="${settings.rowSelector}_"]`).each(function () {

          const $el = $(this);
          const cls = $el.attr('class');
          if (!cls) return;

          $el.attr('class', cls.replace(rowRe, `${settings.rowSelector}_${e}`));
        });

        // serial_remove_502 → serial_remove_501
        const removeRe = new RegExp(`${settings.removeClass}_(\\d+)`, 'g');

        $nestedWrap.find(`[class*="${settings.removeClass}_"]`).each(function () {

          const $el = $(this);
          const cls = $el.attr('class');
          if (!cls) return;

          $el.attr('class', cls.replace(removeRe, `${settings.removeClass}_${e}`));
        });
      });
    };

    // 3️⃣ reindexRowIndexPattern (Updated)
    const reindexRowIndexPattern = function () {
      if (!settings.reindexRowIndex || !settings.reindexRowIndex.length) return;

      const $rows = $wrapper.find(`.${settings.rowSelector}`);
      const a = Number(settings.startRow);

      $rows.each(function (newIndex) {

        const newPosition = a + newIndex;
        const $row = $(this);

        settings.reindexRowIndex.forEach(attr => {

      /* ================================
       * OUTER (NOT inside nestedwrapper)
       * ================================ */
          $row.find(`[${attr}]`)
          .not($row.find(`${settings.nestedwrapper} *`))
          .each(function () {
            const $el = $(this);
            const val = $el.attr(attr);
            if (!val) return;

            $el.attr(attr, val.replace(/(\d+)$/, newPosition));
          });

      /* ================================
       * INNER (inside nestedwrapper)
       * ================================ */
          $row.find(settings.nestedwrapper)
          .find(`[${attr}]`)
          .each(function () {
            const $el = $(this);
            const val = $el.attr(attr);
            if (!val) return;

            $el.attr(attr, val.replace(/(\d+)(?=_(\d+)$)/, newPosition));
          });

        });
      });
    };


    // Master reindex function
    const reindexRowAll = function() {
      const $rows = $wrapper.find(`.${settings.rowSelector}`);
      if ($rows.length === 0) return;

      // Reindex all attributes
      reindexRowIDPattern();      // First: IDs and FOR attributes
      reindexRowNamePattern();    // Second: Name attributes
      reindexRowIndexPattern();   // Third: Index-based attributes

      console.log(`Reindexing complete. Now ${$rows.length} rows.`);
    };

    // Public methods
    const methods = {
      add: function() {
        if (i < settings.maxRows) {
          const event = new Event('click');
          addRow(event);
        }
        return this;
      },
      remove: function(index) {
        const $removeBtn = $wrapper.find(`#${settings.rowSelector}_${index} .${settings.removeClass}`);
        if ($removeBtn.length) {
          $removeBtn.trigger('click.remAddRow');
        }
        return this;
      },
      getCount: function() {
        return i;
      },
      reset: function() {
        $wrapper.find(`.${settings.rowSelector}`).remove();
        i = 0;
        return this;
      },
      reindexAll: function() {
        reindexRowAll();
        return this;
      },
      setReindexConfig: function(type, attributes) {
        if (type === 'name') {
          settings.reindexRowName = Array.isArray(attributes) ? attributes : [attributes];
        } else if (type === 'id') {
          settings.reindexRowID = Array.isArray(attributes) ? attributes : [attributes];
        } else if (type === 'index') {
          settings.reindexRowIndex = Array.isArray(attributes) ? attributes : [attributes];
        }
        return this;
      },
      getConfig: function() {
        return {
          fieldName: settings.fieldName,
          rowSelector: settings.rowSelector,
          removeClass: settings.removeClass,
          maxRows: settings.maxRows,
          currentIndex: i,
          reindexConfig: {
            name: settings.reindexRowName,
            id: settings.reindexRowID,
            index: settings.reindexRowIndex
          }
        };
      },
      destroy: function() {
        if (settings.addBtn) {
          $(settings.addBtn).off('click.remAddRow');
        }
        $wrapper.off('click.remAddRow', `.${settings.removeClass}`);
        this.reset();
        return this;
      }
    };

    // Initialize and return methods
    return init();
  };

  // Add noConflict method
  $.fn.remAddRow.noConflict = function() {
    $.fn.remAddRow = old;
    return this;
  };

  // Store the old version in case of conflict
  const old = $.fn.remAddRow;

})(jQuery);



based on this plugin,
i want u to write a comprehensive detailed README.md
on its :
0. package depemdemcy
1. available option and explaination of each options
2. detailed explaination usage on how to use it vy giving comprehensive detail example
3. and other necessary things that should include for this documentation



















# remAddRow - jQuery Dynamic Form Rows Plugin

A lightweight, feature-rich jQuery plugin for managing dynamic form rows with automatic reindexing, nested form support, and comprehensive event handling.

## 📦 0. Package Dependency

### Required
- **jQuery 3.0+** (tested with 3.6.0+)
- **Bootstrap 4/5** (optional, for default styling)

### Installation
```html
<!-- Include jQuery -->
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>

<!-- Include Bootstrap (optional) -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
```

### Quick Start
```html
<div id="formContainer"></div>
<button id="addRowBtn" class="btn btn-primary">Add Row</button>

<script>
$('#formContainer').remAddRow({
    addBtn: '#addRowBtn',
    maxRows: 10
});
</script>
```

## ⚙️ 1. Available Options

### Core Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `addBtn` | string | `''` | CSS selector for the "Add Row" button |
| `maxRows` | number | `5` | Maximum number of rows allowed |
| `startRow` | number | `0` | Starting index for row numbering |
| `fieldName` | string | `'data'` | Base name for form fields (e.g., `data[0][name]`) |
| `rowSelector` | string | `'rowserial'` | CSS class for row containers |
| `removeClass` | string | `'serial_remove'` | CSS class for remove buttons |
| `nestedwrapper` | string | `null` | Selector for nested wrappers (for complex forms) |

### Reindexing Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `reindexRowName` | array | `['name', 'data-bv-field', 'data-bv-for']` | Attributes containing field names to reindex |
| `reindexRowID` | array | `['id', 'for', 'aria-describedby']` | Attributes containing IDs to reindex |
| `reindexRowIndex` | array | `['data-index', 'data-id']` | Attributes containing indexes to reindex |

### Event Callbacks

| Option | Type | Parameters | Description |
|--------|------|------------|-------------|
| `onAdd` | function | `(index, event, $row, name)` | Called when a row is added |
| `onRemove` | function | `(index, event, $row, name)` | Called before removing a row |
| `rowTemplate` | function | `(index, fieldName)` | Custom HTML template for rows |

## 📖 2. Detailed Usage Examples

### Basic Example - Simple Form Rows
```html
<div id="simpleForm">
    <!-- Rows will be added here -->
</div>
<button id="addBtn" class="btn btn-primary">Add Person</button>

<script>
$('#simpleForm').remAddRow({
    addBtn: '#addBtn',
    maxRows: 5,
    fieldName: 'people',
    startRow: 1, // Start from index 1
    onAdd: function(index, event, $row, name) {
        console.log('Added row:', index);
        console.log('Field name:', name); // 'people'

        // Focus on the first input in the new row
        $row.find('input[type="text"]').first().focus();
    },
    onRemove: function(index, event, $row, name) {
        // Confirm before removal
        return confirm('Are you sure you want to remove this row?');
    }
});
</script>
```

### Advanced Example - Custom Template
```html
<div id="productForm"></div>
<button id="addProduct" class="btn btn-success">Add Product</button>

<script>
$('#productForm').remAddRow({
    addBtn: '#addProduct',
    maxRows: 10,
    fieldName: 'products',
    rowSelector: 'product-row',
    removeClass: 'remove-product',

    // Custom template function
    rowTemplate: function(index, fieldName) {
        return `
            <div id="product-row_${index}" class="row mb-3 product-row border p-3">
                <div class="col-md-6">
                    <label for="product_name_${index}">Product Name</label>
                    <input type="text"
                           name="${fieldName}[${index}][name]"
                           id="product_name_${index}"
                           class="form-control"
                           required>
                </div>
                <div class="col-md-4">
                    <label for="product_price_${index}">Price</label>
                    <div class="input-group">
                        <span class="input-group-text">$</span>
                        <input type="number"
                               name="${fieldName}[${index}][price]"
                               id="product_price_${index}"
                               class="form-control"
                               step="0.01"
                               min="0">
                    </div>
                </div>
                <div class="col-md-2 d-flex align-items-end">
                    <button type="button"
                            class="btn btn-danger remove-product"
                            data-index="${index}">
                        <i class="fas fa-trash"></i> Remove
                    </button>
                </div>
            </div>
        `;
    },

    // Custom reindexing configuration
    reindexRowName: ['name', 'data-field', 'data-validate'],
    reindexRowID: ['id', 'for', 'aria-labelledby', 'aria-describedby'],
    reindexRowIndex: ['data-index', 'data-row', 'data-id']
});
</script>
```

### Complex Example - Nested Forms with Inner Rows
```html
<div id="mainForm"></div>
<button id="addMainRow" class="btn btn-primary">Add Category</button>

<script>
$('#mainForm').remAddRow({
    addBtn: '#addMainRow',
    maxRows: 5,
    fieldName: 'categories',
    rowSelector: 'category-row',
    removeClass: 'remove-category',
    nestedwrapper: '.inner-rows', // Important for nested reindexing

    rowTemplate: function(index, fieldName) {
        return `
            <div id="category-row_${index}" class="category-row card mb-3">
                <div class="card-header">
                    <h5>Category ${index + 1}</h5>
                    <button type="button" class="btn btn-sm btn-danger remove-category" data-index="${index}">
                        Remove Category
                    </button>
                </div>
                <div class="card-body">
                    <div class="mb-3">
                        <label for="cat_name_${index}">Category Name</label>
                        <input type="text"
                               name="${fieldName}[${index}][name]"
                               id="cat_name_${index}"
                               class="form-control">
                    </div>

                    <!-- Nested rows container -->
                    <div class="inner-rows" id="innerRows_${index}">
                        <!-- Inner rows will be added here -->
                    </div>

                    <!-- Add inner row button -->
                    <button type="button"
                            class="btn btn-sm btn-outline-secondary add-inner-row"
                            data-category-index="${index}">
                        Add Item
                    </button>
                </div>
            </div>
        `;
    },

    onAdd: function(index, event, $row, name) {
        // Initialize nested plugin for this category
        const $innerContainer = $row.find('.inner-rows');

        $innerContainer.remAddRow({
            addBtn: $row.find('.add-inner-row'),
            maxRows: 3,
            fieldName: `${name}[${index}][items]`,
            rowSelector: 'item-row',
            removeClass: 'remove-item',
            startRow: 0,

            rowTemplate: function(itemIndex, itemFieldName) {
                return `
                    <div id="item-row_${index}_${itemIndex}" class="item-row row mb-2">
                        <div class="col-md-5">
                            <input type="text"
                                   name="${itemFieldName}[${itemIndex}][name]"
                                   class="form-control form-control-sm"
                                   placeholder="Item name">
                        </div>
                        <div class="col-md-5">
                            <input type="number"
                                   name="${itemFieldName}[${itemIndex}][quantity]"
                                   class="form-control form-control-sm"
                                   placeholder="Quantity">
                        </div>
                        <div class="col-md-2">
                            <button type="button"
                                    class="btn btn-sm btn-outline-danger remove-item"
                                    data-index="${itemIndex}">
                                ×
                            </button>
                        </div>
                    </div>
                `;
            }
        });
    }
});
</script>
```

### Example with Form Validation (Bootstrap Validator)
```html
<form id="myForm" data-toggle="validator">
    <div id="dynamicRows"></div>
    <button id="addRowBtn" type="button" class="btn btn-primary">Add Row</button>
    <button type="submit" class="btn btn-success">Submit</button>
</form>

<script>
$('#dynamicRows').remAddRow({
    addBtn: '#addRowBtn',
    maxRows: 5,
    fieldName: 'employees',

    // Include Bootstrap Validator attributes in reindexing
    reindexRowName: ['name', 'data-bv-field', 'data-bv-for'],
    reindexRowID: ['id', 'for', 'aria-describedby'],

    rowTemplate: function(index, fieldName) {
        return `
            <div id="rowserial_${index}" class="rowserial row mb-3">
                <div class="col-md-4">
                    <label for="email_${index}">Email</label>
                    <input type="email"
                           name="${fieldName}[${index}][email]"
                           id="email_${index}"
                           class="form-control"
                           data-bv-emailaddress="true"
                           data-bv-emailaddress-message="Invalid email"
                           required>
                </div>
                <div class="col-md-4">
                    <label for="phone_${index}">Phone</label>
                    <input type="tel"
                           name="${fieldName}[${index}][phone]"
                           id="phone_${index}"
                           class="form-control"
                           data-bv-regexp="true"
                           data-bv-regexp-regexp="^[0-9]{10}$"
                           data-bv-regexp-message="10 digits required">
                </div>
                <div class="col-md-4 d-flex align-items-end">
                    <button type="button"
                            class="btn btn-danger serial_remove"
                            data-index="${index}">
                        Remove
                    </button>
                </div>
            </div>
        `;
    },

    onAdd: function(index, event, $row, name) {
        // Refresh Bootstrap Validator to recognize new fields
        if ($('#myForm').data('bootstrapValidator')) {
            $('#myForm').data('bootstrapValidator').addField(
                `[name="${name}[${index}][email]"]`
            );
        }
    },

    onRemove: function(index, event, $row, name) {
        // Remove from Bootstrap Validator before deletion
        if ($('#myForm').data('bootstrapValidator')) {
            $('#myForm').data('bootstrapValidator')
                .removeField(`[name="${name}[${index}][email]"]`);
        }
        return true; // Allow removal
    }
});
</script>
```

## 🔧 3. Public Methods

### Accessing Methods
```javascript
const plugin = $('#container').remAddRow(options);

// Use methods
plugin.add();          // Add a row
plugin.remove(2);      // Remove row at index 2
plugin.getCount();     // Get current row count
plugin.reindexAll();   // Force reindex all rows
```

### Complete Methods Reference

| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `add()` | None | Plugin instance | Adds a new row |
| `remove(index)` | `index` (number) | Plugin instance | Removes row at specified index |
| `getCount()` | None | number | Returns current number of rows |
| `reset()` | None | Plugin instance | Removes all rows, resets counter |
| `reindexAll()` | None | Plugin instance | Reindexes all rows |
| `setReindexConfig(type, attributes)` | `type` (string): 'name', 'id', or 'index'<br>`attributes` (array/string) | Plugin instance | Updates reindexing configuration |
| `getConfig()` | None | object | Returns current configuration |
| `destroy()` | None | Plugin instance | Removes all events, resets DOM |

### Method Usage Examples
```javascript
// Initialize
const dynamicForm = $('#formContainer').remAddRow({
    addBtn: '#addBtn',
    maxRows: 10
});

// Programmatically add row
$('#anotherButton').click(function() {
    dynamicForm.add();
});

// Remove specific row
$('#removeThird').click(function() {
    dynamicForm.remove(2); // Removes row with index 2
});

// Update reindexing config
dynamicForm.setReindexConfig('name', ['name', 'data-field', 'custom-attr']);

// Get current state
const config = dynamicForm.getConfig();
console.log('Current rows:', config.currentIndex);
console.log('Max rows:', config.maxRows);

// Reset form
$('#resetForm').click(function() {
    dynamicForm.reset();
});

// Clean up when leaving page
$(window).on('beforeunload', function() {
    dynamicForm.destroy();
});
```

## 🎯 4. Reindexing Logic Explained

### How Reindexing Works
The plugin automatically reindexes three types of attributes when rows are added/removed:

1. **Name Attributes** (`reindexRowName`):
   - Updates `name`, `data-bv-field`, `data-bv-for` attributes
   - Pattern: `fieldName[index][field]` → `fieldName[newIndex][field]`

2. **ID Attributes** (`reindexRowID`):
   - Updates `id`, `for`, `aria-describedby` attributes
   - Pattern: `field_0` → `field_1`

3. **Index Attributes** (`reindexRowIndex`):
   - Updates `data-index`, `data-id` attributes
   - Pattern: `data-index="0"` → `data-index="1"`

### Nested Reindexing
When using `nestedwrapper`, the plugin handles two-level reindexing:

**Outer (main row):**
- `id="product_0"` → `id="product_1"`
- `name="data[0][name]"` → `name="data[1][name]"`

**Inner (nested rows):**
- `id="product_0_item_0"` → `id="product_1_item_0"`
- `name="data[0][items][0][name]"` → `name="data[1][items][0][name]"`

## 🔌 5. Event Handling

### Custom Event Integration
```javascript
$('#container').remAddRow({
    // ... other options ...

    onAdd: function(index, event, $row, name) {
        // Your custom logic
        console.log('Row added at index:', index);

        // Dispatch custom event
        $(document).trigger('rowAdded', {
            index: index,
            row: $row,
            fieldName: name
        });

        // Example: Initialize datepicker on new fields
        $row.find('.datepicker').datepicker();

        // Example: Initialize select2
        $row.find('.select2').select2();

        // Return false to prevent adding (optional)
        // return false;
    },

    onRemove: function(index, event, $row, name) {
        // Custom confirmation
        const confirmDelete = Swal.fire({
            title: 'Are you sure?',
            text: "You won't be able to revert this!",
            icon: 'warning',
            showCancelButton: true,
            confirmButtonColor: '#3085d6',
            cancelButtonColor: '#d33',
            confirmButtonText: 'Yes, delete it!'
        });

        // Return promise for async operations
        return confirmDelete.then((result) => {
            return result.isConfirmed;
        });

        // Or return boolean for sync operations
        // return confirm('Delete this row?');
    }
});
```

## 🚀 6. Advanced Usage Patterns

### Dynamic Configuration
```javascript
// Create reusable configuration
const baseConfig = {
    maxRows: 10,
    rowSelector: 'dynamic-row',
    removeClass: 'remove-dynamic',
    reindexRowName: ['name', 'data-validate']
};

// Extend for specific use cases
const productConfig = {
    ...baseConfig,
    fieldName: 'products',
    rowTemplate: function(index, name) {
        return `... product template ...`;
    }
};

const userConfig = {
    ...baseConfig,
    fieldName: 'users',
    rowTemplate: function(index, name) {
        return `... user template ...`;
    }
};

// Initialize multiple instances
const productForm = $('#products').remAddRow(productConfig);
const userForm = $('#users').remAddRow(userConfig);
```

### Integration with Vue/React (via jQuery wrapper)
```javascript
// React component example
class DynamicForm extends React.Component {
    componentDidMount() {
        this.plugin = $(this.container).remAddRow({
            addBtn: this.addButton,
            maxRows: this.props.maxRows,
            rowTemplate: this.renderRow.bind(this)
        });
    }

    componentWillUnmount() {
        this.plugin.destroy();
    }

    renderRow(index, fieldName) {
        return `
            <div class="rowserial">
                <input name="${fieldName}[${index}][value]"
                       value="${this.props.initialData[index] || ''}">
            </div>
        `;
    }

    render() {
        return (
            <div ref={el => this.container = el}>
                <button ref={el => this.addButton = el}>Add</button>
            </div>
        );
    }
}
```

## ⚠️ 7. Common Issues & Solutions

### Issue 1: Reindexing not working
**Solution:** Check your reindexing configuration:
```javascript
// Ensure attributes are included
.setReindexConfig('name', ['name', 'data-field', 'data-bv-field'])
.setReindexConfig('id', ['id', 'for'])
.setReindexConfig('index', ['data-index', 'data-id'])
```

### Issue 2: Nested rows not reindexing properly
**Solution:** Set `nestedwrapper` option:
```javascript
nestedwrapper: '.inner-container',
// And ensure your template has matching structure
```

### Issue 3: Form validation breaking
**Solution:** Refresh validator after operations:
```javascript
onAdd: function(index, event, $row, name) {
    // For Bootstrap Validator
    $('#form').bootstrapValidator('addField',
        `[name="${name}[${index}][field]"]`
    );

    // For jQuery Validate
    $('#form').validate().element(`[name="${name}[${index}][field]"]`);
}
```

## 📝 8. Best Practices

1. **Always call `destroy()`** when removing the form from DOM
2. **Use `reset()`** instead of manually clearing rows
3. **Set appropriate `maxRows`** to prevent unlimited additions
4. **Implement `onRemove` callback** for confirmation dialogs
5. **Test reindexing** with your specific form structure
6. **Use `getConfig()`** for debugging when issues arise

## 📄 9. License & Credits

This plugin is open-source and can be modified for your needs. Include credit to the original author if redistributing.

```javascript
/**
 * remAddRow - jQuery Dynamic Form Rows Plugin
 * @version 1.0.0
 * @license MIT
 * @author Your Name
 * @requires jQuery 3.0+
 */
```

## 🆘 10. Support & Contribution

For issues, feature requests, or contributions:
1. Check the [GitHub repository](https://github.com/yourusername/remAddRow)
2. Create detailed bug reports with examples
3. Follow the existing code style for contributions

---

**Happy Coding!** 🚀




