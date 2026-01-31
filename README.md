# Add Remove Row - jQuery Dynamic Form Rows Plugin

A lightweight, feature-rich jQuery plugin for managing dynamic form rows with automatic reindexing, nested form support, and comprehensive event handling.

## 📦 0. Package Dependency

### Required
- **jQuery 3.6+**
- **Bootstrap 4/5** (optional, for default styling)

### Installation
```html
<!-- Include jQuery -->
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>

<!-- Plugin -->
<script src="addRemRow.js"></script>

<!-- Include Bootstrap (optional) -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
```


### NPM Installation
- ensure jQuery is loaded.

```npm

npm install addremrow

```

then in app.js
a) webpack / laravel-mix
``` bash

require('addremrow');

````

b) Vite
``` bash

import  'addremrow';

```

### Quick Start
```html
<div id="formContainer"></div>
<button id="addRowBtn" class="btn btn-primary">Add Row</button>

<script>
$('#formContainer').addRemRow({
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

## 📖 2. Usage Examples

### Basic Example - Simple Form Rows
```html
<div id="simpleForm">
    <!-- Rows will be added here -->
</div>
<button id="addBtn" class="btn btn-primary">Add Person</button>

<script>
$('#simpleForm').addRemRow({
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
$('#productForm').addRemRow({
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
$('#mainForm').addRemRow({
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

        $innerContainer.addRemRow({
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
$('#dynamicRows').addRemRow({
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
const plugin = $('#container').addRemRow(options);

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
| `addBatch(dataArray)` | `dataArray` (array) | Plugin instance | Batch add multiple rows with data |
| `remove(index)` | `index` (number) | Plugin instance | Removes row at specified index |
| `getCount()` | None | number | Returns current number of rows |
| `reset()` | None | Plugin instance | Removes all rows, resets counter |
| `reindexAll()` | None | Plugin instance | Reindexes all rows |
| `setReindexConfig(type, attributes)` | `type` (string): 'name', 'id', or 'index'<br>`attributes` (array/string) | Plugin instance | Updates reindexing configuration |
| `getConfig()` | None | object | Returns current configuration |
| `destroy()` | None | Plugin instance | Removes all events, resets DOM |
| `getRow(index)` | `index` (number) | string/null | Gets HTML of specific row |
| `getAllData()` | None | array | Returns all row data as array of objects |
| `hasRow(index)` | `index` (number) | boolean | Checks if specific row exists |
| `getWrapper()` | None | jQuery object | Returns wrapper element |
| `validateAll(options)` | `options` (object) | validation object | Validates all rows with custom rules |
| `clearAll()` | None | Plugin instance | Clears all field values |
| `disableAll()` | None | Plugin instance | Disables all rows and fields |
| `enableAll()` | None | Plugin instance | Enables all rows and fields |
| `setRowData(index, data)` | `index` (number), `data` (object) | Plugin instance | Sets data for specific row |
| `exportJSON()` | None | string | Exports all rows as JSON string |
| `importJSON(jsonString, clearExisting)` | `jsonString` (string), `clearExisting` (boolean) | result object | Imports rows from JSON |
| `findRowByField(fieldName, value)` | `fieldName` (string), `value` (any) | array | Finds rows by field value |
| `countBy(conditionCallback)` | `conditionCallback` (function) | number | Counts rows matching condition |
| `toggleRows(show)` | `show` (boolean) | Plugin instance | Shows/hides all rows |
| `filterRows(filterCallback)` | `filterCallback` (function) | Plugin instance | Filters rows based on callback |
| `sortRows(fieldName, ascending)` | `fieldName` (string), `ascending` (boolean) | Plugin instance | Sorts rows by field value |

### Method Usage Examples
```javascript
// Initialize
const dynamicForm = $('#formContainer').addRemRow({
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

### New Methods Examples

#### Data Management
```javascript
// Get all row data
const allData = plugin.getAllData();
console.log('All row data:', allData);

// Export as JSON
const jsonData = plugin.exportJSON();
console.log('JSON export:', jsonData);

// Import from JSON
const importResult = plugin.importJSON('[{"name":"John","skill":"JS"},{"name":"Jane","skill":"Python"}]', true);
if (importResult.success) {
    console.log(`Imported ${importResult.count} rows`);
}

// Set data for specific row
plugin.setRowData(2, {
    name: 'Alice',
    skill: 'React'
});

// Batch add multiple rows
plugin.addBatch([
    {name: 'Bob', skill: 'Vue'},
    {name: 'Charlie', skill: 'Angular'}
]);
```

#### Validation
```javascript
// Validate all rows
const validation = plugin.validateAll({
    required: true,
    minLength: 2,
    maxLength: 50,
    customValidation: function($field, rowIndex) {
        if ($field.attr('name').includes('email')) {
            const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
            if (!emailRegex.test($field.val())) {
                return {
                    valid: false,
                    message: 'Invalid email format'
                };
            }
        }
        return { valid: true };
    }
});

if (!validation.valid) {
    console.log('Validation errors:', validation.getAllErrors());
    console.log('First error:', validation.getFirstError());
}
```

#### Row Operations
```javascript
// Find rows by field value
const johnRows = plugin.findRowByField('name', 'John');
johnRows.forEach(row => {
    console.log(`Found John at row ${row.index}`);
});

// Count rows with specific condition
const jsExperts = plugin.countBy(function($row, index) {
    return $row.find('[name*="[skill]"]').val() === 'JavaScript';
});
console.log(`JavaScript experts: ${jsExperts}`);

// Filter rows
plugin.filterRows(function($row, index) {
    const skill = $row.find('[name*="[skill]"]').val();
    return skill.includes('JS') || skill.includes('JavaScript');
});

// Sort rows by name
plugin.sortRows('name', true); // Ascending

// Toggle visibility
plugin.toggleRows(false); // Hide all rows
plugin.toggleRows(true);  // Show all rows

// Get specific row HTML
const row2HTML = plugin.getRow(2);
console.log('Row 2 HTML:', row2HTML);

// Check if row exists
if (plugin.hasRow(3)) {
    console.log('Row 3 exists');
}
```

#### Form Control
```javascript
// Clear all form fields
$('#clearBtn').click(function() {
    plugin.clearAll();
});

// Disable all fields (read-only mode)
plugin.disableAll();

// Enable all fields
plugin.enableAll();

// Get wrapper for custom operations
const $wrapper = plugin.getWrapper();
$wrapper.addClass('highlighted');
```

#### Advanced Usage
```javascript
// Chain multiple operations
plugin
    .clearAll()
    .addBatch(sampleData)
    .sortRows('name')
    .validateAll()
    .reindexAll();

// Search and highlight
$('#searchBtn').click(function() {
    const query = $('#searchInput').val().toLowerCase();
    plugin.filterRows(function($row, index) {
        const rowText = $row.text().toLowerCase();
        const matches = rowText.includes(query);
        $row.toggleClass('highlight-match', matches);
        return true; // Show all, but highlight matches
    });
});

// Export to CSV
function exportToCSV() {
    const data = plugin.getAllData();
    let csv = 'Name,Skill\n';
    data.forEach(row => {
        csv += `"${row.name || ''}","${row.skill || ''}"\n`;
    });
    return csv;
}
```

#### Integration with External Libraries
```javascript
// With DataTables
const table = $('#dataTable').DataTable();
const refreshTable = function() {
    table.clear();
    plugin.getAllData().forEach(row => {
        table.row.add([row.name, row.skill, row.email]);
    });
    table.draw();
};

// With Chart.js
const updateChart = function() {
    const data = plugin.getAllData();
    const skillCounts = {};
    data.forEach(row => {
        skillCounts[row.skill] = (skillCounts[row.skill] || 0) + 1;
    });

    // Update chart data
    chart.data.labels = Object.keys(skillCounts);
    chart.data.datasets[0].data = Object.values(skillCounts);
    chart.update();
};

// Auto-save to localStorage
$(window).on('beforeunload', function() {
    const jsonData = plugin.exportJSON();
    localStorage.setItem('formData', jsonData);
});

// Load from localStorage on page load
$(document).ready(function() {
    const savedData = localStorage.getItem('formData');
    if (savedData) {
        plugin.importJSON(savedData, true);
    }
});
```

### Method Chaining Examples
```javascript
// Complex operation chain
plugin
    .reset()                           // Clear everything
    .addBatch(initialData)             // Add initial dataset
    .sortRows('skill', true)           // Sort by skill
    .filterRows(function($row, idx) {   // Filter by condition
        return $row.find('[name*="[experience]"]').val() > 3;
    })
    .validateAll({                     // Validate
        required: true,
        customValidation: customValidator
    })
    .reindexAll();                     // Ensure proper indexing

// Quick setup chain
$('#container')
    .addRemRow(config)
    .addBatch(defaultRows)
    .disableAll()        // Start disabled
    .on('enableForm', function() {
        $(this).enableAll(); // Enable on custom event
    });
````


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
$('#container').addRemRow({
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
const productForm = $('#products').addRemRow(productConfig);
const userForm = $('#users').addRemRow(userConfig);
```

### Integration with Vue/React (via jQuery wrapper)
```javascript
// React component example
class DynamicForm extends React.Component {
    componentDidMount() {
        this.plugin = $(this.container).addRemRow({
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
 * addRemRow - jQuery Dynamic Form Rows Plugin
 * @version 1.0.0
 * @license MIT
 * @author Your Name
 * @requires jQuery 3.0+
 */
```

## 🆘 10. Support & Contribution

For issues, feature requests, or contributions:
1. Check the [GitHub repository](https://github.com/kroos/jQuery-Add-Remove-Row)
2. Create detailed bug reports with examples
3. Follow the existing code style for contributions

---

**Happy Coding!** 🚀




