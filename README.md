![Photos from @ikukevk](https://user-images.githubusercontent.com/306521/30183963-9c722dca-941c-11e7-9e83-c78377ad7f9d.jpg)

[![Build Status](https://travis-ci.org/icebob/fastest-validator.svg?branch=master)](https://travis-ci.org/icebob/fastest-validator)
[![Coverage Status](https://coveralls.io/repos/github/icebob/fastest-validator/badge.svg?branch=master)](https://coveralls.io/github/icebob/fastest-validator?branch=master)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/75256e6ec26d42f5ab1dee109ae4d3ad)](https://www.codacy.com/app/mereg-norbert/fastest-validator?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=icebob/fastest-validator&amp;utm_campaign=Badge_Grade)
[![Known Vulnerabilities](https://snyk.io/test/github/icebob/fastest-validator/badge.svg)](https://snyk.io/test/github/icebob/fastest-validator)
[![Size](https://badgen.net/bundlephobia/minzip/fastest-validator)](https://bundlephobia.com/result?p=fastest-validator)

# fastest-validator [![NPM version](https://img.shields.io/npm/v/fastest-validator.svg)](https://www.npmjs.com/package/fastest-validator) [![Tweet](https://img.shields.io/twitter/url/http/shields.io.svg?style=social)](https://twitter.com/intent/tweet?text=The%20fastest%20JS%20validator%20library%20for%20NodeJS&url=https://github.com/icebob/fastest-validator&via=Icebobcsi&hashtags=nodejs,javascript)
:zap: The fastest JS validator library for NodeJS.

**If you like my work, please [donate](https://www.paypal.me/meregnorbert). Thank you!**

## Key features
* blazing fast! Really!
* 15+ built-in validators
* many sanitizations
* custom validators
* nested objects & array handling
* strict object validation
* multiple validators
* customizable error messages
* programmable error object
* no dependencies
* unit tests & 100% coverage

# How fast?
Very fast! 8 million validations/sec (on Intel i7-4770K, Node.JS: 10.16.0)
```
√ validate                        8,461,975 rps
```

Compared to other popular libraries:

[![Result](https://user-images.githubusercontent.com/306521/68978853-404a8500-07fc-11ea-94e4-0c25546dad04.png)](https://github.com/icebob/validator-benchmark#result)
> 50x faster than Joi.

**Would you like to test it?**

```
$ git clone https://github.com/icebob/fastest-validator.git
$ cd fastest-validator
$ npm install
$ npm run bench
```

# Table of contents
- [Installations](#installation)
- [Usage](#usage)
- [Optional & required fields](#optional--required-fields)
- [Strict validation](#strict-validation)
- [Multiple validators](#multiple-validators)
- [Root element schema](#root-element-schema)
- [Sanitizations](#sanitizations)
- [Shorthand definitions](#shorthand-definitions)
- [Built-in validators](#built-in-validators)
    - [any](#any)
    - [array](#array)
    - [boolean](#boolean)
    - [date](#date)
    - [email](#email)
    - [enum](#enum)
    - [equal](#equal)
    - [forbidden](#forbidden)
    - [function](#function)
    - [luhn](#luhn)
    - [mac](#mac)
    - [multi](#multi)
    - [number](#number)
    - [object](#object)
    - [string](#string)
    - [url](#url)
    - [uuid](#uuid)
- [Custom validator](#custom-validator)
- [Custom error messages (l10n)](#custom-error-messages-l10n)
- [Personalised Messages](#personalised-messages)
- [Message types](#message-types)
- [Development](#development)
- [Test](#test)
- [Contribution](#contribution)
- [License](#license)
- [Contact](#contact)

## Installation

### NPM
You can install it via [NPM](http://npmjs.org/).
```
$ npm i fastest-validator --save
```
or
```
$ yarn add fastest-validator
```

## Usage

### Simple method
Call the `validate` method with the `object` and the `schema`.
> If performance is important, you won't use this method because it's slow.

```js
let Validator = require("fastest-validator");

let v = new Validator();

const schema = {
    id: { type: "number", positive: true, integer: true },
    name: { type: "string", min: 3, max: 255 },
    status: "boolean" // short-hand def
};

console.log(v.validate({ id: 5, name: "John", status: true }, schema));
// Returns: true

console.log(v.validate({ id: 5, name: "Al", status: true }, schema));
/* Returns an array with errors:
    [
        {
            type: 'stringMin',
            expected: 3,
            actual: 2,
            field: 'name',
            message: 'The \'name\' field length must be greater than or equal to 3 characters long!'
        }
    ]
*/
```
[Try it on Runkit](https://runkit.com/icebob/fastest-validator-usage-simple)

### Fast method
In this case, the first step is to compile the schema to a compiled "checker" function. After that, to validate your object, just call this "checker" function.
> This method is the fastest.

```js
let Validator = require("fastest-validator");

let v = new Validator();

var schema = {
    id: { type: "number", positive: true, integer: true },
    name: { type: "string", min: 3, max: 255 },
    status: "boolean" // short-hand def
};

var check = v.compile(schema);

console.log(check({ id: 5, name: "John", status: true }));
// Returns: true

console.log(check({ id: 2, name: "Adam" }));
/* Returns an array with errors:
    [
        {
            type: 'required',
            field: 'status',
            message: 'The \'status\' field is required!'
        }
    ]
*/
```
[Try it on Runkit](https://runkit.com/icebob/fastest-validator-usage-quick)

### Browser usage
```html
<script src="https://unpkg.com/fastest-validator"></script>
```

```js
var v = new FastestValidator();

const schema = {
    id: { type: "number", positive: true, integer: true },
    name: { type: "string", min: 3, max: 255 },
    status: "boolean" // short-hand def
};

const check = v.compile(schema);

console.log(check({ id: 5, name: "John", status: true }));
// Returns: true
```

# Optional & required fields
Every field in the schema will be required by default. If you'd like to define optional fields, set `optional: true`.

```js
const schema = {
    name: { type: "string" }, // required
    age: { type: "number", optional: true }
}

v.validate({ name: "John", age: 42 }, schema); // Valid
v.validate({ name: "John" }, schema); // Valid
v.validate({ age: 42 }, schema); // Fail because name is required
```

# Strict validation
Object properties which are not specified on the schema are ignored by default. If you set the `$$strict` option to `true` any aditional properties will result in an `strictObject` error.

```js
const schema = {
    name: { type: "string" }, // required
    $$strict: true // no additional properties allowed
}

v.validate({ name: "John" }, schema); // Valid
v.validate({ name: "John", age: 42 }, schema); // Fail
```

## Remove additional fields
To remove the additional fields in the object, set `$$strict: "remove"`.


# Multiple validators
It is possible to define more validators for a field. In this case, only one validator needs to succeed for the field to be valid.

```js
const schema = {
    cache: [
        { type: "string" },
        { type: "boolean" }
    ]
}

v.validate({ cache: true }, schema); // Valid
v.validate({ cache: "redis://" }, schema); // Valid
v.validate({ cache: 150 }, schema); // Fail
```

# Root element schema
Basically the validator expects that you want to validate a Javascript object. If you want others, you can define the root level schema, as well. In this case set the `$$root: true` property.

**Example to validate a `string` variable instead of `object`**
```js
const schema = {
    $$root: true,
    type: "string", 
    min: 3, 
    max: 6
};

v.validate("John", schema); // Valid
v.validate("Al", schema); // Fail, too short.
```

# Sanitizations
The library contains several sanitizaters. **Please note, the sanitizers change the original checked object.**

## Default values
The most common sanitizer is the `default` property. With it, you can define a default value for all properties. If the property value is `null` or `undefined`, the validator set the defined default value into the property.

**Default value example**:
```js
const schema = {
    roles: { type: "array", items: "string", default: ["user"] },
    status: { type: "boolean", default: true },
};

const obj = {}

v.validate(obj, schema); // Valid
console.log(obj);
/*
{
    roles: ["user"],
    status: true
}
*/
```

# Shorthand definitions
You can use string-based shorthand validation definitions in the schema.

```js
const schema = {
    password: "string|min:6",
    age: "number|optional|integer|positive|min:0|max:99", // additional properties
    state: ["boolean", "number|min:0|max:1"] // multiple types
}
```

# Built-in validators

## `any`
This does not do type validation. Accepts any types.

```js
const schema = {
    prop: { type: "any" }
}

v.validate({ prop: true }, schema); // Valid
v.validate({ prop: 100 }, schema); // Valid
v.validate({ prop: "John" }, schema); // Valid
```

## `array`
This is an `Array` validator.

**Simple example with strings:**
```js
const schema = {
    roles: { type: "array", items: "string" }
}

v.validate({ roles: ["user"] }, schema); // Valid
v.validate({ roles: [] }, schema); // Valid
v.validate({ roles: "user" }, schema); // Fail
```

**Example with only positive numbers:**
```js
const schema = {
    list: { type: "array", min: 2, items: {
        type: "number", positive: true, integer: true
    } }
}

v.validate({ list: [2, 4] }, schema); // Valid
v.validate({ list: [1, 5, 8] }, schema); // Valid
v.validate({ list: [1] }, schema); // Fail (min 2 elements)
v.validate({ list: [1, -7] }, schema); // Fail (negative number)
```

**Example with an object list:**
```js
const schema = {
    users: { type: "array", items: {
        type: "object", props: {
            id: { type: "number", positive: true },
            name: { type: "string", empty: false },
            status: "boolean"
        }
    } }
}

v.validate({
    users: [
        { id: 1, name: "John", status: true },
        { id: 2, name: "Jane", status: true },
        { id: 3, name: "Bill", status: false }
    ]
}, schema); // Valid
```

**Example for `enum`:**
```js
const schema = {
    roles: { type: "array", items: "string", enum: [ "user", "admin" ] }
}

v.validate({ roles: ["user"] }, schema); // Valid
v.validate({ roles: ["user", "admin"] }, schema); // Valid
v.validate({ roles: ["guest"] }, schema); // Fail
```

### Properties
Property | Default  | Description
-------- | -------- | -----------
`empty`  | `true`   | If `true`, the validator accepts an empty array `[]`.
`min`  	 | `null`   | Minimum count of elements.
`max`  	 | `null`   | Maximum count of elements.
`length` | `null`   | Fix count of elements.
`contains` | `null` | The array must contain this element too.
`enum`	 | `null`   | Every element must be an element of the `enum` array.
`items`	 | `null`   | Schema for array items.


## `boolean`
This is a `Boolean` validator.

```js
const schema = {
    status: { type: "boolean" }
}

v.validate({ status: true }, schema); // Valid
v.validate({ status: false }, schema); // Valid
v.validate({ status: 1 }, schema); // Fail
v.validate({ status: "true" }, schema); // Fail
```
### Properties
Property | Default  | Description
-------- | -------- | -----------
`convert` | `false` | if `true` and the type is not `Boolean`, it will be converted. `1`, `"true"`, `"1"`, `"on"` will be true. `0`, `"false"`, `"0"`, `"off"` will be false. _It's a sanitizer, it will change the value in the original object._

**Example for `convert`:**
```js
v.validate({ status: "true" }, {
    status: { type: "boolean", convert: true}
}); // Valid
```

## `date`
This is a `Date` validator.

```js
const schema = {
    dob: { type: "date" }
}

v.validate({ dob: new Date() }, schema); // Valid
v.validate({ dob: new Date(1488876927958) }, schema); // Valid
v.validate({ dob: 1488876927958 }, schema); // Fail
```

### Properties
Property | Default  | Description
-------- | -------- | -----------
`convert`  | `false`| if `true` and the type is not `Date`, try to convert with `new Date()`. _It's a sanitizer, it will change the value in the original object._

**Example for `convert`:**
```js
v.validate({ dob: 1488876927958 }, {
    dob: { type: "date", convert: true}
}); // Valid
```

## `email`
This is an e-mail address validator.

```js
const schema = {
    email: { type: "email" }
}

v.validate({ email: "john.doe@gmail.com" }, schema); // Valid
v.validate({ email: "james.123.45@mail.co.uk" }, schema); // Valid
v.validate({ email: "abc@gmail" }, schema); // Fail
```

### Properties
Property | Default  | Description
-------- | -------- | -----------
`mode`   | `quick`  | Checker method. Can be `quick` or `precise`.
`normalize`   | `false`  | Normalize the e-mail address (trim & lower-case). _It's a sanitizer, it will change the value in the original object._

## `enum`
This is an enum validator.

```js
const schema = {
    sex: { type: "enum", values: ["male", "female"] }
}

v.validate({ sex: "male" }, schema); // Valid
v.validate({ sex: "female" }, schema); // Valid
v.validate({ sex: "other" }, schema); // Fail
```

### Properties
Property | Default  | Description
-------- | -------- | -----------
`values` | `null`   | The valid values.

## `equal`
This is an equal value validator. It checks a value with a static value or with another property.

**Example with static value**:
```js
const schema = {
    agreeTerms: { type: "equal", value: true, strict: true } // strict means `===`
}

v.validate({ agreeTerms: true }, schema); // Valid
v.validate({ agreeTerms: false }, schema); // Fail
```

**Example with other field**:
```js
const schema = {
    password: { type: "string", min: 6 },
    confirmPassword: { type: "equal", field: "password" }
}

v.validate({ password: "123456", confirmPassword: "123456" }, schema); // Valid
v.validate({ password: "123456", confirmPassword: "pass1234" }, schema); // Fail
```

### Properties
Property | Default  | Description
-------- | -------- | -----------
`value`  | `undefined`| The expected value. It can be any primitive types.
`strict`  | `false`| if `true`, it uses strict equal `===` for checking.

## `forbidden`
This validator returns an error if the property exists in the object.

```js
const schema = {
    password: { type: "forbidden" }
}

v.validate({ user: "John" }, schema); // Valid
v.validate({ user: "John", password: "pass1234" }, schema); // Fail
```

### Properties
Property | Default  | Description
-------- | -------- | -----------
`remove` | `false`   | If `true`, the value will be removed in the original object. _It's a sanitizer, it will change the value in the original object._

**Example for `remove`:**
```js
const schema = {
    user: { type: "string" },
    token: { type: "forbidden", remove: true }
};

const obj = {
    user: "John",
    token: "123456"
}

v.validate(obj, schema); // Valid
console.log(obj);
/*
{
    user: "John",
    token: undefined
}
*/
```

## `function`
This a `Function` type validator.

```js
const schema = {
    show: { type: "function" }
}

v.validate({ show: function() {} }, schema); // Valid
v.validate({ show: Date.now }, schema); // Valid
v.validate({ show: "function" }, schema); // Fail
```

## `luhn`
This is an Luhn validator.
[Luhn algorithm](https://en.wikipedia.org/wiki/Luhn_algorithm) checksum
Credit Card numbers, IMEI numbers, National Provider Identifier numbers and others 

```js
const schema = {
    cc: { type: "luhn" }
}

v.validate({ cc: "452373989901198" }, schema); // Valid
v.validate({ cc: 452373989901198 }, schema); // Valid
v.validate({ cc: "4523-739-8990-1198" }, schema); // Valid
v.validate({ cc: "452373989901199" }, schema); // Fail
```

## `mac`
This is an MAC addresses validator. 

```js
const schema = {
    mac: { type: "mac" }
}

v.validate({ mac: "01:C8:95:4B:65:FE" }, schema); // Valid
v.validate({ mac: "01:c8:95:4b:65:fe", schema); // Valid
v.validate({ mac: "01C8.954B.65FE" }, schema); // Valid
v.validate({ mac: "01c8.954b.65fe", schema); // Valid
v.validate({ mac: "01-C8-95-4B-65-FE" }, schema); // Valid
v.validate({ mac: "01-c8-95-4b-65-fe" }, schema); // Valid
v.validate({ mac: "01C8954B65FE" }, schema); // Fail
```

## `multi`
This is a multiple definitions validator. 

```js
const schema = {
    status: { type: "multi", rules: [
        { type: "boolean" },
        { type: "number" }
    ], default: true }
}

v.validate({ status: true }, schema); // Valid
v.validate({ status: false }, schema); // Valid
v.validate({ status: 1 }, schema); // Valid
v.validate({ status: 0 }, schema); // Valid
v.validate({ status: "yes" }, schema); // Fail
```

**Shorthand multiple definitions**:
```js
const schema = {
    status: [
        "boolean",
        "number"
    ]
}

v.validate({ status: true }, schema); // Valid
v.validate({ status: false }, schema); // Valid
v.validate({ status: 1 }, schema); // Valid
v.validate({ status: 0 }, schema); // Valid
v.validate({ status: "yes" }, schema); // Fail
```

## `number`
This is a `Number` validator.

```js
const schema = {
    age: { type: "number" }
}

v.validate({ age: 123 }, schema); // Valid
v.validate({ age: 5.65 }, schema); // Valid
v.validate({ age: "100" }, schema); // Fail
```

### Properties
Property | Default  | Description
-------- | -------- | -----------
`min`  	 | `null`   | Minimum value.
`max`  	 | `null`   | Maximum value.
`equal`  | `null`   | Fixed value.
`notEqual` | `null` | Can't be equal to this value.
`integer` | `false` | The value must be a non-decimal value.
`positive` | `false`| The value must be greater than zero.
`negative` | `false`| The value must be less than zero.
`convert`  | `false`| if `true` and the type is not `Number`, it's converted with `Number()`. _It's a sanitizer, it will change the value in the original object._

## `object`
This is a nested object validator.

```js
const schema = {
    address: { type: "object", strict: true, props: {
        country: { type: "string" },
        city: "string", // short-hand
        zip: "number" // short-hand
    } }
}

v.validate({
    address: {
        country: "Italy",
        city: "Rome",
        zip: 12345
    }
}, schema); // Valid

v.validate({
    address: {
        country: "Italy",
        city: "Rome"
    }
}, schema); // Fail ("The 'address.zip' field is required!")

v.validate({
    address: {
        country: "Italy",
        city: "Rome",
        zip: 12345,
        state: "IT"
    }
}, schema); // Fail ("The 'address.state' is an additional field!")
```

### Properties
Property | Default  | Description
-------- | -------- | -----------
`strict`  | `false`| if `true` any properties which are not defined on the schema will throw an error. If `remove` all additional properties will be removed from the original object. _It's a sanitizer, it will change the original object._

```js
const schema = {
    address: { type: "object", strict: "remove", props: {
        country: { type: "string" },
        city: "string", // short-hand
        zip: "number" // short-hand
    } }
}

const obj = {
    address: {
        country: "Italy",
        city: "Rome",
        zip: 12345,
        state: "IT"
    }
};

v.validate(obj, schema); // Valid
console.log(obj);
/*
{
    address: {
        country: "Italy",
        city: "Rome",
        zip: 12345
    }   
}
*/
```

## `string`
This is a `String` validator.

```js
const schema = {
    name: { type: "string" }
}

v.validate({ name: "John" }, schema); // Valid
v.validate({ name: "" }, schema); // Valid
v.validate({ name: 123 }, schema); // Fail
```

### Properties
Property | Default  | Description
-------- | -------- | -----------
`empty`  | `true`   | If `true`, the validator accepts an empty string `""`.
`min`  	 | `null`   | Minimum value length.
`max`  	 | `null`   | Maximum value length.
`length` | `null`   | Fixed value length.
`pattern` | `null`   | Regex pattern.
`contains` | `null`   | The value must contain this text.
`enum`	 | `null`   | The value must be an element of the `enum` array.
`alpha`   | `null`   | The value must be an alphabetic string.
`numeric`   | `null`   | The value must be a numeric string.
`alphanum`   | `null`   | The value must be an alphanumeric string.
`alphadash`   | `null`   | The value must be an alphabetic string that contains dashes.
`trim`   | `null`   | If `true`, the value will be trimmed. _It's a sanitizer, it will change the value in the original object._
`trimLeft`   | `null`   | If `true`, the value will be left trimmed. _It's a sanitizer, it will change the value in the original object._
`trimRight`   | `null`   | If `true`, the value will be right trimmed. _It's a sanitizer, it will change the value in the original object._
`padStart`   | `null`   | If it's a number, the value will be left padded. _It's a sanitizer, it will change the value in the original object._
`padEnd`   | `null`   | If it's a number, the value will be right padded. _It's a sanitizer, it will change the value in the original object._
`padChar`   | `" "`   | The padding characther for the `padStart` and `padEnd`.
`lowercase`   | `null`   | If `true`, the value will be lower-cased. _It's a sanitizer, it will change the value in the original object._
`uppercase`   | `null`   | If `true`, the value will be upper-cased. _It's a sanitizer, it will change the value in the original object._
`localeLowercase`   | `null`   | If `true`, the value will be locale lower-cased. _It's a sanitizer, it will change the value in the original object._
`localeUppercase`   | `null`   | If `true`, the value will be locale upper-cased. _It's a sanitizer, it will change the value in the original object._
`convert`  | `false`| if `true` and the type is not a `String`, it's converted with `String()`. _It's a sanitizer, it will change the value in the original object._

**Sanitization example**
```js
const schema = {
    username: { type: "string", min: 3, trim: true, lowercase: true}
}

const obj = {
    username: "   Icebob  "
};

v.validate(obj, schema); // Valid
console.log(obj);
/*
{
    username: "icebob"
}
*/
```

## `url`
This is an URL validator.

```js
const schema = {
    url: { type: "url" }
}

v.validate({ url: "http://google.com" }, schema); // Valid
v.validate({ url: "https://github.com/icebob" }, schema); // Valid
v.validate({ url: "www.facebook.com" }, schema); // Fail
```

## `uuid`
This is an UUID validator. 

```js
const schema = {
    uuid: { type: "uuid" }
}

v.validate({ uuid: "10ba038e-48da-487b-96e8-8d3b99b6d18a" }, schema); // Valid UUIDv4
v.validate({ uuid: "9a7b330a-a736-51e5-af7f-feaf819cdc9f" }, schema); // Valid UUIDv5
v.validate({ uuid: "10ba038e-48da-487b-96e8-8d3b99b6d18a", version: 5 }, schema); // Fail
```
### Properties
Property | Default  | Description
-------- | -------- | -----------
`version`  | `4`   | UUID version in range 1-5.


# Custom validator
You can also create your custom validator.

```js
let v = new Validator({
    messages: {
        // Register our new error message text
        evenNumber: "The '{field}' field must be an even number! Actual: {actual}"
    }
});

// Register a custom 'even' validator
v.add("even", function({ schema, messages }, path, context) {
    return {
        source: `
            if (value % 2 != 0)
                ${this.makeError({ type: "evenNumber",  actual: "value", messages })}

            return value;
        `
    };
});

const schema = {
    name: { type: "string", min: 3, max: 255 },
    age: { type: "even" }
};

console.log(v.validate({ name: "John", age: 20 }, schema));
// Returns: true

console.log(v.validate({ name: "John", age: 19 }, schema));
/* Returns an array with errors:
    [{
        type: 'evenNumber',
        expected: null,
        actual: 19,
        field: 'age',
        message: 'The \'age\' field must be an even number! Actual: 19'
    }]
*/
```

Or you can use the `custom` type with an inline checker function:
```js
let v = new Validator({
    messages: {
        // Register our new error message text
        weightMin: "The weight must be greater than {expected}! Actual: {actual}"
    }
});

const schema = {
    name: { type: "string", min: 3, max: 255 },
    weight: {
        type: "custom",
        minWeight: 10,
        check(value, schema) {
            return (value < schema.minWeight)
                ? this.makeError("weightMin", schema.minWeight, value)
                : true;
        }
    }
};

console.log(v.validate({ name: "John", weight: 50 }, schema));
// Returns: true

console.log(v.validate({ name: "John", weight: 8 }, schema));
/* Returns an array with errors:
    [{
        type: 'weightMin',
        expected: 10,
        actual: 8,
        field: 'weight',
        message: 'The weight must be greater than 10! Actual: 8'
    }]
*/
```

# Custom error messages (l10n)
You can set your custom messages in the validator constructor.

```js
const Validator = require("fastest-validator");
const v = new Validator({
    messages: {
        stringMin: "A(z) '{field}' mező túl rövid. Minimum: {expected}, Jelenleg: {actual}",
        stringMax: "A(z) '{field}' mező túl hosszú. Minimum: {expected}, Jelenleg: {actual}"
    }
});

v.validate({ name: "John" }, { name: { type: "string", min: 6 }});
/* Returns:
[
    {
        type: 'stringMin',
        expected: 6,
        actual: 4,
        field: 'name',
        message: 'A(z) \'name\' mező túl rövid. Minimum: 6, Jelenleg: 4'
    }
]
*/
```
# Personalised Messages
Sometimes the standard messages are too generic. You can customise messages per validation type per field:

```js
const Validator = require("fastest-validator");
const v = new Validator();
const schema = {
    firstname: {
        type: "string",
        min: 6,
        messages: {
            string: "Please check your firstname",
            stringMin: "Your firstname is too short"
        }
    },
    lastname: {
        type: "string",
        min: 6,
        messages: {
            string: "Please check your lastname",
            stringMin: "Your lastname is too short"
        }
    }
}
v.validate({ firstname: "John", lastname: 23 }, schema );
/* Returns:
[
    {
        type: 'stringMin',
        expected: 6,
        actual: 4,
        field: 'firstname',
        message: 'Your firstname is too short'
    },
    {
        type: 'string',
        expected: undefined,
        actual: undefined,
        field: 'lastname',
        message: 'Please check your lastname'
    }
]
*/
```
# Message types
Name                | Default text
------------------- | -------------
`required`	| The '{field}' field is required.
`string`	| The '{field}' field must be a string.
`stringEmpty`	| The '{field}' field must not be empty.
`stringMin`	| The '{field}' field length must be greater than or equal to {expected} characters long.
`stringMax`	| The '{field}' field length must be less than or equal to {expected} characters long.
`stringLength`	| The '{field}' field length must be {expected} characters long.
`stringPattern`	| The '{field}' field fails to match the required pattern.
`stringContains`	| The '{field}' field must contain the '{expected}' text.
`stringEnum`	| The '{field}' field does not match any of the allowed values.
`stringNumeric`	| The '{field}' field must be a numeric string.
`stringAlpha`	| The '{field}' field must be an alphabetic string.
`stringAlphanum`	| The '{field}' field must be an alphanumeric string.
`stringAlphadash`	| The '{field}' field must be an alphadash string.
`number`	| The '{field}' field must be a number.
`numberMin`	| The '{field}' field must be greater than or equal to {expected}.
`numberMax`	| The '{field}' field must be less than or equal to {expected}.
`numberEqual`	| The '{field}' field must be equal to {expected}.
`numberNotEqual`	| The '{field}' field can't be equal to {expected}.
`numberInteger`	| The '{field}' field must be an integer.
`numberPositive`	| The '{field}' field must be a positive number.
`numberNegative`	| The '{field}' field must be a negative number.
`array`	| The '{field}' field must be an array.
`arrayEmpty`	| The '{field}' field must not be an empty array.
`arrayMin`	| The '{field}' field must contain at least {expected} items.
`arrayMax`	| The '{field}' field must contain less than or equal to {expected} items.
`arrayLength`	| The '{field}' field must contain {expected} items.
`arrayContains`	| The '{field}' field must contain the '{expected}' item.
`arrayEnum`	| The '{actual}' value in '{field}' field does not match any of the '{expected}' values.
`boolean`	| The '{field}' field must be a boolean.
`function`	| The '{field}' field must be a function.
`date`	| The '{field}' field must be a Date.
`dateMin`	| The '{field}' field must be greater than or equal to {expected}.
`dateMax`	| The '{field}' field must be less than or equal to {expected}.
`forbidden`	| The '{field}' field is forbidden.
`email`	| The '{field}' field must be a valid e-mail.
`url`	| The '{field}' field must be a valid URL.
`enumValue`	| The '{field}' field value '{expected}' does not match any of the allowed values.
`equalValue`	| The '{field}' field value must be equal to '{expected}'.
`equalField`	| The '{field}' field value must be equal to '{expected}' field value.
`object`	| The '{field}' must be an Object.
`objectStrict`	| The object '{field}' contains forbidden keys: '{actual}'.
`uuid`	| The '{field}' field must be a valid UUID.
`uuidVersion`	| The '{field}' field must be a valid UUID version provided.
`mac`	| The '{field}' field must be a valid MAC address.
`luhn`	| The '{field}' field must be a valid checksum luhn.

## Message fields
Name        | Description
----------- | -------------
`field`     | The field name
`expected`  | The expected value
`actual`    | The actual value

# Development
```
npm run dev
```

# Test
```
npm test
```

## Coverage report
```
-----------------|----------|----------|----------|----------|-------------------|
File             |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
-----------------|----------|----------|----------|----------|-------------------|
All files        |      100 |    97.73 |      100 |      100 |                   |
 lib             |      100 |      100 |      100 |      100 |                   |
  messages.js    |      100 |      100 |      100 |      100 |                   |
  validator.js   |      100 |      100 |      100 |      100 |                   |
 lib/helpers     |      100 |      100 |      100 |      100 |                   |
  deep-extend.js |      100 |      100 |      100 |      100 |                   |
  flatten.js     |      100 |      100 |      100 |      100 |                   |
 lib/rules       |      100 |    96.43 |      100 |      100 |                   |
  any.js         |      100 |      100 |      100 |      100 |                   |
  array.js       |      100 |      100 |      100 |      100 |                   |
  boolean.js     |      100 |      100 |      100 |      100 |                   |
  custom.js      |      100 |       50 |      100 |      100 |                 6 |
  date.js        |      100 |      100 |      100 |      100 |                   |
  email.js       |      100 |      100 |      100 |      100 |                   |
  enum.js        |      100 |       50 |      100 |      100 |                 6 |
  equal.js       |      100 |      100 |      100 |      100 |                   |
  forbidden.js   |      100 |      100 |      100 |      100 |                   |
  function.js    |      100 |      100 |      100 |      100 |                   |
  luhn.js        |      100 |      100 |      100 |      100 |                   |
  mac.js         |      100 |      100 |      100 |      100 |                   |
  multi.js       |      100 |      100 |      100 |      100 |                   |
  number.js      |      100 |      100 |      100 |      100 |                   |
  object.js      |      100 |      100 |      100 |      100 |                   |
  string.js      |      100 |    95.83 |      100 |      100 |             55,63 |
  url.js         |      100 |      100 |      100 |      100 |                   |
  uuid.js        |      100 |      100 |      100 |      100 |                   |
-----------------|----------|----------|----------|----------|-------------------|
```

# Contribution
Please send pull requests improving the usage and fixing bugs, improving documentation and providing better examples, or providing some tests, because these things are important.

# License
fastest-validator is available under the [MIT license](https://tldrlegal.com/license/mit-license).

# Contact

Copyright (C) 2019 Icebob

[![@icebob](https://img.shields.io/badge/github-icebob-green.svg)](https://github.com/icebob) [![@icebob](https://img.shields.io/badge/twitter-Icebobcsi-blue.svg)](https://twitter.com/Icebobcsi)
