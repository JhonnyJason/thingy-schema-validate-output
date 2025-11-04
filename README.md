# thingy-schema-validate 
A lightweight, zero-dependency utility for precompiling validators for your structured data in JavaScript. Define your Schema in the most simple and non-verbose way and validate them conveniently.

## Features
- **Nice Schema Definitions**: Write schemas quickly, which don't mess up your code.
- **Fast Schema Validation**: Precompile the validator function once from your schema. Validate your data more efficiently for the rest of your programs runtime. (Still has some optimization potential though^^)
- **Good Set of Predefined Types**: Common data types, including strings, numbers, booleans, arrays, objects, and specialized formats like hex strings, clean strings, emails and non-empty strings. Plus versions that may be `null` or even `undefined` are already available to simply use.
- **Extensible**: Define custom types and error messages.


# Background
When working on my Service Communication Interface(SCI)-mechanics the need to validate the inputs and outputs arose. As other tools that would fit the purpose use schema definitions which are too verbose and too ugly this is the nicest JS way of defining schemas - or well, judge by yourself ;-).

### Latest updates (v0.0.5 -> v0.0.6 = complete rewrite):
- New base-types e.g. `STRINGEMAIL`, `STRINGCLEAN` and `OBJECTCLEAN`
- Optional types that can be non-existent e.g. `mySchema = { user: STRINGCLEAN, isAdmin: BOOLEANORNOTHING }`
- Arbitrarily nested schemas
- Schema can be a different base-type now e.g. `mySchema = STRING` or `mySchema = [ NUMBER, STRING ]` 
- Precompilation of validator functions 
- General performance improvement: 
    - x1.5 for non-compiled validation (no errors) 
    - x11  for non-compiled validatiokn (all errors)
    - x2.7 for compiled validation (no errors)
    - x19.7 for compiled validation (al errors)

# Usage
Installation
------------
```bash
npm install thingy-schema-validate
```

Current Functionality
---------------------
## API Reference

### Exported Constants

- **Basic Types (Enumeration)**: `BOOLEAN`, `NUMBER`, `ARRAY`, `OBJECT`, `STRING`, `STRINGEMAIL`, `STRINGHEX`, `STRINGHEX32`, `STRINGHEX64`, `STRINGHEX128`, `STRINGHEX256`, `STRINGHEX512`, `STRINGCLEAN`, `NONEMPTYSTRING`, `NONEMPTYSTRINGHEX`, `NONEMPTYSTRINGCLEAN`, `NONEMPTYARRAY`, `OBJECTCLEAN`, `NONNULLOBJECT`, `NONNULLOBJECTCLEAN`, `STRINGORNOTHING`, `STRINGEMAILORNOTHING`, `STRINGHEXORNOTHING`, `STRINGHEX32ORNOTHING`, `STRINGHEX64ORNOTHING`, `STRINGHEX128ORNOTHING`, `STRINGHEX256ORNOTHING`, `STRINGHEX512ORNOTHING`, `STRINGCLEANORNOTHING`, `NUMBERORNOTHING`, `BOOLEANORNOTHING`, `ARRAYORNOTHING`, `OBJECTORNOTHING`, `OBJECTCLEANORNOTHING`, `STRINGORNULL`, `STRINGEMAILORNULL`, `STRINGHEXORNULL`, `STRINGHEX32ORNULL`, `STRINGHEX64ORNULL`, `STRINGHEX128ORNULL`, `STRINGHEX256ORNULL`, `STRINGHEX512ORNULL`, `STRINGCLEANORNULL`, `NUMBERORNULL`, `BOOLEANORNULL`, `ARRAYORNULL`
- **Error Codes (Enumeration)**: `NOTANUMBER`, `NOTABOOLEAN`, `NOTANARRAY`, `NOTANOBJECT`, `INVALIDHEX`, `INVALIDEMAIL`, `INVALIDSIZE`, `ISNAN`, `ISNULL`, `ISEMPTYSTRING`, `ISEMPTYARRAY`, `ISDIRTYSTRING`, `ISDIRTYOBJECT`, `ISNOTFINITE`, `ISINVALID`

### Functions

- `validate(obj, schema, staticStrings)`: Direct validation of Object to schema.
- `createValidator(schema, staticStrings)`: Returns a validator function is ~x1.8 faster then `validate`.
- `getErrorMessage(errorCode)`: Returns the error message for a given code.
- `defineNewError(errorMessage)`: Adds a new error code.
- `defineNewType(validatorFunc)`: Adds a new type.
- `setTypeValidator(type, validatorFunc)`: Overrides typeValidator for given type.
- `lock()`: Freezes internal maps.

### 1. Basic Validation

Before Validation you may create your own custom types and error messages and lock the module to deny any mutation.

You may create nice multilevel schemas of arbitrary depth. Then compile the validator for fast validation.

When the object passed to the validator is valid the function simply returns void/`undefined` -> falsly value is valid. 
For invalod objects the `errorCode` is returned which is a number. (error codes cannot be 0)
You get the corresponding errorMessage with `getErrorMessage`

```javascript
import {
  STRINGCLEAN, NUMBER, BOOLEAN, ARRAY, createValidator,
  getErrorMessage
} from 'thingy-schema-validate';

// Define your schema
const userSchema = {
  name: STRINGCLEAN,
  age: NUMBER,
  isAdmin: BOOLEAN,
  tags: ARRAY,
  address: {
    street: STRINGCLEAN,
    city: STRINGCLEAN,
    country: STRINGCLEAN
  }
};

// Create a validator
const validate = createValidator(userSchema);

// Validate data
const userData = {
  name: "Alice",
  age: 30,
  isAdmin: true,
  tags: ["user", "admin"],
  address: { city: "Berlin", street: "Meinestrasse 666", country:"Deutschland" }
};

const error = validate(userData);
if (error) {
  console.error("Validation failed: ", getErrorMessage(error));
} else {
  console.log("Data is valid!");
}
```

### 2. Special Cases

In some situations you want to have a 0 level schema. This is perfectly legal! :D

```javascript
const hashSchema = STRINGHEX32
const validateHash = createValidator(hashSchema);
const error = validateHash("a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4");
```

Also maybe you want to bake a static string into your schema.
For this you must pass true for the `staticStrings` argument.

```javascript
const userSchema = {
    role: "user",
    name: STRINGCLEAN
}
const validateUser = createValidator(userSchema, true);
const error = validateUser({role:"user", name: "Max Mustermann"});
```

### 3. Custom Types and Errors
You have all the power to add your specific types or to overwrite the validators of any already given type as long as the library is not `locked`.

Notice that there has been set an arbitrary limit of a maximum of 999 different types and 999 different errorCodes. This limit is very arbitrarily set to a number which is thought to never be reached.

```javascript
// Define a new error
const NO = defineNewError("Simply NO!");

// Define a new type
const STRINGSIZED = defineNewType(
  (a) => { if(typeof a != "string" || a.length > 10 || a.length < 3) return NO },
  (a) => { return '"'+a+'"' }
);
```

### 4. Locking the Library
For a certain safety-net you may lock the library to prevent overwriting your configuration later.

If you donot change any types you may simply call `lock()` immediately and start to compile your validators.

```javascript
lock(); // Prevents further mutations to types and errors
```

---

## Default Types

`thingy-schema-validate` provides a wide range of built-in types for common validation needs.
Here’s an extensive list of all available types:

---
### Core Types
- `STRING`: JS type `"string"`. (including `""`, `"\x00"`)
- `NUMBER`: JS type `"number"`. (excluding `NaN`, `Infinity`, `-Infinity` )
- `BOOLEAN`: JS type `"boolean"`
- `ARRAY`: Legit JS Array. (including `[]`, excluding `null`)
- `OBJECT`: JS type `"object"`. (including `null`,`{}`)

### Special String Types - valid `STRING` +
- `STRINGEMAIL`: Email address Format. Mostly RFC5322 BUT: ASCII only without `'"'`, `"!"`, `" "`, `"#"`, `"$"`, `"%"`, `"&"`, `"'"`, `"*"`, `"/"`, `"="`, `"?"`, `"^"`, `"{"`, `"|"`, `"}"`, `"~"` and "`" in front of the @ plus comments and quoted strings are illegal.
- `STRINGHEX`: All hexadecimal characters.
- `STRINGHEX32`: 32 hexadecimal characters.
- `STRINGHEX64`: 64 hexadecimal characters.
- `STRINGHEX128`: 128 hexadecimal characters.
- `STRINGHEX256`: 256 hexadecimal characters.
- `STRINGHEX512`: 512 hexadecimal characters.

### Cleaner Types
- `STRINGCLEAN`: `STRING` but excluding control characters or invisible whitespace characters.
- `NONEMPTYSTRING`: `STRING` but excluding `""`.
- `NONEMPTYSTRINGHEX`: `STRINGHEX` but excluding `""`
- `NONEMPTYSTRINGCLEAN`: `STRINGCLEAN` but excluding `""`
- `NONEMPTYARRAY`: `ARRAY` but excluding `[]`
- `OBJECTCLEAN`: `OBJECT` without properties like `"__proto__"`, `"constructor"` and `"prototype"`
- `NONNULLOBJECT`: `OBJECT` but excluding `null`
- `NONNULLOBJECTCLEAN`: `OBJECTCLEAN` but excluding `null`

### Optional Types
They might be nonexistant  or `undefined` in the object to-be-validated.
- `STRINGORNOTHING`
- `STRINGEMAILORNOTHING`
- `STRINGHEXORNOTHING`
- `STRINGHEX32ORNOTHING`
- `STRINGHEX64ORNOTHING`
- `STRINGHEX128ORNOTHING`
- `STRINGHEX256ORNOTHING`
- `STRINGHEX512ORNOTHING`
- `STRINGCLEANORNOTHING`
- `NUMBERORNOTHING`
- `BOOLEANORNOTHING`
- `ARRAYORNOTHING`
- `OBJECTORNOTHING`
- `OBJECTCLEANORNOTHING`

### Types that may explicitly be `null`
They must exist but may be `null`
- `STRINGORNULL`
- `STRINGEMAILORNULL`
- `STRINGHEXORNULL`
- `STRINGHEX32ORNULL`
- `STRINGHEX64ORNULL`
- `STRINGHEX128ORNULL`
- `STRINGHEX256ORNULL`
- `STRINGHEX512ORNULL`
- `STRINGCLEANORNULL`
- `NUMBERORNULL`
- `BOOLEANORNULL`
- `ARRAYORNULL`

## Default ErrorCodes
Remember that the definitions are enumerated so you might import `NOTANUMBER` and check against this specific errorCode for further decisions. But for simple human readable output the function `getErrorMessage(errorCode)` is the way to go.

- `NOTASTRING` => "Not a String!"
- `NOTANUMBER` => "Not a Number!"
- `NOTABOOLEAN` => "Not a Boolean!"
- `NOTANARRAY` => "Not an Array!"
- `NOTANOBJECT` => "Not an Object!"
- `INVALIDHEX` => "String is not valid hex!"
- `INVALIDEMAIL` => "String is not a valid email!"
- `INVALIDSIZE` => "String size mismatch!"
- `ISNAN` => "Number is NaN!"
- `ISNULL` => "Object is null!"
- `ISEMPTYSTRING` => "String is empty!"
- `ISEMPTYARRAY` => "Array is empty!"
- `ISDIRTYSTRING` => "String is dirty!"
- `ISDIRTYOBJECT` => "Object is dirty!"
- `ISNOTFINITE` => "Number is not finite!"
- `ISINVALID` => "Schema is invalid!"


# Further steps

- Further optimizations
- Intruduce new types that seem useful
- Fix any found Bugs/Issues

All sorts of inputs are welcome, thanks!

---

# License
[CC0](https://creativecommons.org/publicdomain/zero/1.0/)
