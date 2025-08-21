# Data Type and Format Validation Rules

## Overview
This document describes the data type and format validation rules implemented in the CICS Process Management system. The validation routines are integrated with the existing `GENERAL-ERROR-ROUTINE` error handling framework.

## Validation Rules

### Date Components (YEAR, MONTH, DAY)
- **YEAR**: Must be numeric, range 1900-2099
- **MONTH**: Must be numeric, range 01-12
- **DAY**: Must be numeric, range 01-31 with proper month/leap year validation
  - January, March, May, July, August, October, December: 01-31
  - April, June, September, November: 01-30
  - February: 01-28 (01-29 in leap years)
- **Leap Year Logic**: Year divisible by 4, except century years must be divisible by 400

### Email Format (EMAIL)
- Must contain exactly one '@' symbol
- Must have at least one character before and after '@'
- Total length must be between 5-50 characters
- Basic format validation only (not RFC-compliant)

### Character Fields (AUTHOR, LOCATION)
- **Allowed Characters**: A-Z, 0-9, space, period (.), comma (,), hyphen (-), apostrophe (')
- Case-insensitive validation (converts to uppercase)
- Must not be empty/blank
- Spaces are allowed within the field

## Error Messages
- **Error Code 100**: "INVALID DATE FORMAT" - Date component validation failure
- **Error Code 101**: "INVALID EMAIL FORMAT" - Email format validation failure  
- **Error Code 102**: "INVALID AUTHOR FORMAT" - Author character validation failure
- **Error Code 103**: "INVALID LOCATION FORMAT" - Location character validation failure

## Integration Points
- Validation occurs after BMS field-level validation (MUSTENTER, etc.)
- Validation occurs before VSAM file operations
- Uses existing error handling flow: clear screen → position cursor → display error → reinitialize → redisplay → return

## Programs Updated
- **HPMSCS01**: Incident reporting (AUTHOR, EMAIL, YEAR, MONTH, DAY, LOCATION)
- **HPMSCT01**: Test log entry (AUTHOR, EMAIL, YEAR, MONTH, DAY)
- **HPMSCPM1**: Project management inquiry
- **HPMSCPM2**: Project management add/update
- **HPMSCP01**: Propellant inventory inquiry
- **HPMSCP02**: Propellant inventory add/update (YEAR, MONTH, DAY for serial dates)

## Technical Implementation
- **Copybook**: `COPYLIB/VALIDATE` contains reusable validation routines
- **Main Routine**: `VALIDATE-INPUT-FIELDS` - coordinates all validation checks
- **Working Storage**: Validation switches and work fields for processing
- **Error Handling**: Integrates with existing `GENERAL-ERROR-ROUTINE` pattern
- **Cursor Positioning**: Sets field length to -1 for invalid fields to position cursor

## Usage
Programs call `PERFORM VALIDATE-INPUT-FIELDS` after screen input and check `VALIDATION-ERROR-SW` for 'Y' to determine if validation failed. If validation fails, the program calls `PERFORM GENERAL-ERROR-ROUTINE` to display the appropriate error message.
