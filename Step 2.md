# FillGPT Documentation

A Node.js module for automated form filling using OpenAI's GPT models.

## Features

- Extracts form field information from HTML
- Fills form fields based on user's resume
- Handles multiple input types (text, radio, checkbox, dropdown)
- Token usage tracking
- Form splitting for large forms
- Model switching capability

## Installation

```bash
npm install axios date-fns
```

## Core Components

### OpenAIAPI Class

```javascript
class OpenAIAPI {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.baseUrl = 'https://api.openai.com/v1';
    // ... initialization code
  }

  async chatCompletion(messages, model = 'gpt-4o-mini-2024-07-18') {
    // ... API call implementation
  }
}
```

### Main Functions

1. `fillGpt(text, form, date)`: Fills form fields using GPT
   - Takes user's resume text, form structure and date as input
   - Returns JSON with field values based on resume content
   - Handles required vs optional fields
   - Avoids country fields and employee referrals

ores projects/other sections

2. `extractInputField(htmlCode)`:
   - Extracts form field information from HTML
   - Identifies labels, IDs, types and options
   - Determines if fields are required

## Input/Output Formats

### Form Field JSON Structure
```json
{
    "Label": "Field Label",
    "Identifier": "field-id", 
    "Type": "input-type",
    "Options": [
        {
            "Option Id": "option-id",
            "Option Text": "Option Label"
        }
    ],
    "Required": "Yes/No"
}
```

### Result JSON Structure
```json
[
    {
        "Identifier": "input-1",
        "Type": "dropdown",
        "Value": "selected_value"
    },
    {
        "Identifier": "2", 
        "Type": "radio",
        "Value": "No"
    }
]
```

## Prompt Examples

### Form Field Extraction Prompt
```
You are tasked with extracting or generating appropriate answers for each form field based on the provided user's resume. Your output should be in JSON format, with each form field mapped to an extracted or generated value.

Instructions:
1. Extract Data: For each form field, try to extract the relevant data from the user's resume.
2. Generate Data: If the required data is not available in the resume, generate a suitable value based on the form field's label and type.
3. Field Types:
   - "button": Select the most appropriate option from the given options based on the label.
   - "radio/radioGroup/checkbox": Choose the most appropriate "Option ID" as the Identifier and the "Option Text" as the "Value".
   - "text/textarea": Extract or generate a suitable text value.
   ...
```

## Error Handling

- JSON parsing retry logic using regex matching as fallback
- Error logging for API failures with descriptive messages
- Default values (empty array) for failed extractions

## Form Processing Techniques

### Form Splitting
- Large forms are split into smaller chunks to avoid token limits
- Each chunk processed separately while maintaining context
- Results combined into final output

### Model Switching
- Uses gpt-4o-mini for simple field extraction tasks
- Switches to gpt-4o-2024-08-06 for complex form filling
- Optimizes for cost vs capability

### Token Usage Tracking
```javascript
const tokensUsed = response.data.usage.total_tokens || 0;
logAndPrint(`${tokensUsed} Tokens used.`);
logAndPrint(`${tokensUsed * 84 * 0.6 / 1000000} Rs used.`);
```

## Troubleshooting Guide

### Common Problems
1. **Empty or Invalid Responses**
   - Check API key validity
   - Verify input format
   - Review token limits
   - Check for timeout issues

2. **Incorrect Field Values**
   - Review field mapping
   - Check validation rules
   - Verify prompt instructions
   - Test with sample data

3. **Performance Issues**
   - Monitor token usage
   - Optimize batch processing
   - Implement caching
   - Review retry logic
