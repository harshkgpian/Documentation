# Form Field Extraction Documentation

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Concepts](#basic-concepts)
3. [JSON Structure](#json-structure)
4. [Field Types & Extraction](#field-types--extraction)
5. [Implementation Guide](#implementation-guide)
6. [Common Challenges](#common-challenges)
7. [Best Practices](#best-practices)

## Introduction
This documentation covers the process of extracting form fields from web pages for automated form filling applications. It includes identifying different types of form fields, extracting their properties, and handling various scenarios.

## Basic Concepts

### What We Extract
- Form input fields (text, email, number, etc.)
- Field labels and their associations
- Required/Optional status
- Field types and properties
- Dropdown options and values
- Date field components
- File upload fields

### Core Components
1. Field Container - The wrapper element containing the input and label
2. Input Element - The actual form control
3. Label - Text describing the field
4. Validation Requirements - Required/Optional status
5. Field Properties - Type, ID, name, etc.

## JSON Structure

### Basic Field Structure
```json
{
    "Label": "First Name",
    "Required": "yes",
    "Type": "text",
    "Identifier": "firstName",
    "options": [] // for dropdowns
}
```

### Complete Example
```json
{
    "inputButtonTextareaDivs": [
        {
            "Label": "Full Name",
            "Required": "yes",
            "Type": "text",
            "Identifier": "fullName"
        },
        {
            "Label": "Email Address",
            "Required": "yes",
            "Type": "email",
            "Identifier": "emailId"
        },
        {
            "Label": "Country",
            "Required": "no",
            "Type": "select",
            "Identifier": "country",
            "options": ["USA", "Canada", "UK"]
        }
    ]
}
```

## Field Types & Extraction

### 1. Text Fields
```javascript
function extractTextFields() {
    const textFields = document.querySelectorAll('input[type="text"]');
    return Array.from(textFields).map(field => {
        const label = field.closest('div').querySelector('label');
        return {
            Label: label?.textContent.trim() || '',
            Required: label?.textContent.includes('*') ? 'yes' : 'no',
            Type: 'text',
            Identifier: field.id || field.name
        };
    });
}
```

### 2. Date Fields
```javascript
function extractDateField(container) {
    const dateInputs = container.querySelectorAll('input[role="spinbutton"]');
    return {
        Label: container.querySelector('label')?.textContent.trim(),
        Type: 'date',
        Components: {
            Day: dateInputs[0]?.id,
            Month: dateInputs[1]?.id,
            Year: dateInputs[2]?.id
        },
        Required: container.querySelector('label')?.textContent.includes('*') ? 'yes' : 'no'
    };
}
```

### 3. Dropdown Fields
```javascript
function extractDropdownOptions(select) {
    const options = Array.from(select.querySelectorAll('option'))
        .filter(opt => opt.value && opt.value !== 'none')
        .map(opt => ({
            text: opt.textContent.trim(),
            value: opt.value
        }));

    return {
        Label: select.closest('div').querySelector('label')?.textContent.trim(),
        Type: 'select',
        Identifier: select.id || select.name,
        options: options
    };
}
```

## Implementation Guide

### Basic Field Extraction
```javascript
function extractFields() {
    const containers = document.querySelectorAll('.field, .form-control');
    const fields = [];

    containers.forEach(container => {
        const input = container.querySelector('input, select, textarea');
        const label = container.querySelector('label');

        if (!input || !label) return;

        const fieldInfo = {
            Label: label.textContent.trim().replace('*', ''),
            Required: isRequired(input, label),
            Type: getFieldType(input),
            Identifier: input.id || input.name
        };

        if (input.tagName === 'SELECT') {
            fieldInfo.options = extractDropdownOptions(input).options;
        }

        fields.push(fieldInfo);
    });

    return fields;
}

function isRequired(input, label) {
    return (
        input.hasAttribute('required') ||
        input.getAttribute('aria-required') === 'true' ||
        label.textContent.includes('*')
    ) ? 'yes' : 'no';
}

function getFieldType(input) {
    if (input.tagName === 'SELECT') return 'select';
    if (input.tagName === 'TEXTAREA') return 'textarea';
    return input.type || 'text';
}
```

### Label Association
```javascript
function findAssociatedLabel(input) {
    // Method 1: Using 'for' attribute
    if (input.id) {
        const label = document.querySelector(`label[for="${input.id}"]`);
        if (label) return label;
    }

    // Method 2: Parent container search
    const container = input.closest('.field, .form-control');
    return container?.querySelector('label');
}
```

## Common Challenges

### 1. Dynamic Fields
```javascript
function waitForField(fieldId, timeout = 5000) {
    return new Promise((resolve, reject) => {
        if (document.getElementById(fieldId)) {
            resolve(document.getElementById(fieldId));
            return;
        }

        const observer = new MutationObserver(() => {
            const field = document.getElementById(fieldId);
            if (field) {
                observer.disconnect();
                resolve(field);
            }
        });

        observer.observe(document.body, {
            childList: true,
            subtree: true
        });

        setTimeout(() => {
            observer.disconnect();
            reject(new Error(`Timeout waiting for field: ${fieldId}`));
        }, timeout);
    });
}
```

### 2. Hidden Fields
```javascript
function isVisible(element) {
    const style = window.getComputedStyle(element);
    return style.display !== 'none' && 
           style.visibility !== 'hidden' && 
           style.opacity !== '0';
}
```

### 3. Missing Identifiers
```javascript
function generateFieldId(input) {
    return input.id || 
           input.name || 
           `field-${Math.random().toString(36).substr(2, 9)}`;
}
```

## Best Practices

1. **Clean Label Text**
```javascript
function cleanLabelText(text) {
    return text
        .trim()
        .replace('*', '')
        .replace(/\s+/g, ' ');
}
```

2. **Validate Extracted Data**
```javascript
function validateField(field) {
    return {
        isValid: Boolean(
            field.Label &&
            field.Identifier &&
            field.Type
        ),
        errors: [
            !field.Label && 'Missing label',
            !field.Identifier && 'Missing identifier',
            !field.Type && 'Missing type'
        ].filter(Boolean)
    };
}
```

3. **Handle Edge Cases**
- Always check for null/undefined values
- Validate field existence before extraction
- Clean and normalize extracted data
- Verify required field detection
- Test with different form layouts
- Handle dynamic content loading
- Account for different browser implementations

4. **Field Type Detection**
```javascript
const FIELD_PATTERNS = {
    email: /email/i,
    phone: /phone|mobile|contact/i,
    name: /name/i,
    date: /date/i,
    address: /address/i
};

function detectFieldType(label, input) {
    // Check label text against patterns
    for (const [type, pattern] of Object.entries(FIELD_PATTERNS)) {
        if (pattern.test(label)) {
            return type;
        }
    }
    
    // Fallback to input type
    return input.type || 'text';
}
```

Remember:
- Always test extraction on different form layouts
- Handle edge cases (missing labels, dynamic content)
- Clean and normalize extracted data
- Verify required field detection
- Test across different browsers
