# async-validator
---

异步表单验证，基于https://github.com/freeformsystems/async-validate

[![NPM version][npm-image]][npm-url]
[![build status][travis-image]][travis-url]
[![Test coverage][coveralls-image]][coveralls-url]
[![gemnasium deps][gemnasium-image]][gemnasium-url]
[![node version][node-image]][node-url]
[![npm download][download-image]][download-url]

[npm-image]: http://img.shields.io/npm/v/async-validator.svg?style=flat-square
[npm-url]: http://npmjs.org/package/async-validator
[travis-image]: https://img.shields.io/travis/yiminghe/async-validator.svg?style=flat-square
[travis-url]: https://travis-ci.org/yiminghe/async-validator
[coveralls-image]: https://img.shields.io/coveralls/yiminghe/async-validator.svg?style=flat-square
[coveralls-url]: https://coveralls.io/r/yiminghe/async-validator?branch=master
[gemnasium-image]: http://img.shields.io/gemnasium/yiminghe/async-validator.svg?style=flat-square
[gemnasium-url]: https://gemnasium.com/yiminghe/async-validator
[node-image]: https://img.shields.io/badge/node.js-%3E=4.0.0-green.svg?style=flat-square
[node-url]: http://nodejs.org/download/
[download-image]: https://img.shields.io/npm/dm/async-validator.svg?style=flat-square
[download-url]: https://npmjs.org/package/async-validator


## API

以下是从早期版本进行修改 [async-validate](https://github.com/freeformsystems/async-validate).

### Usage

Basic usage involves defining a descriptor, assigning it to a schema and passing the object to be validated and a callback function to the `validate` method of the schema:
基本用法包括定义一个描述符，将其分配给一个模式并将该对象传递给验证，并将一个回调函数传递给模式的`validate`方法：

```javascript
var schema = require('async-validator');
var descriptor = {
  name: {type: "string", required: true}
}
var validator = new schema(descriptor);
validator.validate({name: "muji"}, (errors, fields) => {
  if(errors) {
    // validation failed, errors is an array of all errors
    // fields is an object keyed by field name with an array of
    // errors per field
    // 验证失败，错误是所有错误的数组字段是按字段名称键入的对象，每个字段有一组错误数组
    return handleErrors(errors, fields);
  }
  // validation passed
});
```

### Validate

```javascript
function(source, [options], callback)
```

* `source`: The object to validate (required).
* `source`: 要验证的对象（必需）。
* `options`: An object describing processing options for the validation (optional).
* `options`: 描述用于验证（可选）的处理选项的对象。
* `callback`: A callback function to invoke when validation completes (required).
* `callback`: 验证完成后调用回调函数（必需）。

### Options

* `first`: Boolean, Invoke `callback` when the first validation rule generates an error, 
no more validation rules are processed. 
If your validation involves multiple asynchronous calls (for example, database queries) and you only need the first error use this option.
* `first`：Boolean, 调用`callback`当第一验证规则产生一个错误，没有更多的验证规则进行处理。
  如果您的验证涉及多个异步调用（例如数据库调用），并且只需要第一个错误，请使用此选项。

* `firstFields`: Boolean|String[], Invoke `callback` when the first validation rule of the specified field generates an error, 
no more validation rules of the same field are processed.  `true` means all fields.

*`firstFields`：Boolean | String []，当指定字段的第一个验证规则生成错误时，调用`callback`，不再处理相同字段的验证规则。 `true`表示所有字段。

### Rules

Rules may be functions that perform validation.

(Rules)规则可以是执行验证的功能。

```javascript
function(rule, value, callback, source, options)
```

* `rule`: The validation rule in the source descriptor that corresponds to the field name being validated. It is always assigned a `field` property with the name of the field being validated.
* `rule`：源描述符中的验证规则对应于正在验证的字段名称。 它总是被分配一个`field`属性，并且被验证的字段的名称。
* `value`: The value of the source object property being validated.
* `value`：被验证的源对象属性的值。
* `callback`: A callback function to invoke once validation is complete. It expects to be passed an array of `Error` instances to indicate validation failure.
* `callback`：验证完成后调用的回调函数。 它期望传递一个`Error`实例数组来指示验证失败。
* `source`: The source object that was passed to the `validate` method.
* `source`：传递给`validate`方法的源对象。
* `options`: Additional options.
* `options`：附加选项。
* `options.messages`: The object containing validation error messages, will be deep merged with defaultMessages.
* `options.messages`：含有验证错误消息的目的，将被深与defaultMessages合并。

The options passed to `validate` are passed on to the validation functions so that you may reference transient data (such as model references) in validation functions. However, some option names are reserved; if you use these properties of the options object they are overwritten. The reserved properties are `messages`, `exception` and `error`.

传递给`validate`的选项会传递给验证函数，以便您可以在验证函数中引用瞬态数据（例如模型引用）。 但是，一些选项名称是保留的; 如果您使用这些选项对象的属性，它们将被覆盖。 保留属性是`messages`，`exception`和`error`。

```javascript
var schema = require('async-validator');
var descriptor = {
  name(rule, value, callback, source, options) {
    var errors = [];
    if(!/^[a-z0-9]+$/.test(value)) {
      errors.push(
        new Error(
          util.format("%s must be lowercase alphanumeric characters",
            rule.field)));
    }
    callback(errors);
  }
}
var validator = new schema(descriptor);
validator.validate({name: "Firstname"}, (errors, fields) => {
  if(errors) {
    return handleErrors(errors, fields);
  }
  // validation passed
});
```

It is often useful to test against multiple validation rules for a single field, to do so make the rule an array of objects, for example:

对单个字段的多个验证规则进行测试通常很有用，例如，使规则成为对象数组：

```javascript
var descriptor = {
  email: [
    {type: "string", required: true, pattern: schema.pattern.email},
    {validator(rule, value, callback, source, options) {
      var errors = [];
      // test if email address already exists in a database
      // and add a validation error to the errors array if it does
      // 如果邮件地址已经存在于数据库中，并添加验证错误错误数组
      callback(errors);
    }}
  ]
}
```

#### Type

Indicates the `type` of validator to use. Recognised type values are:

表示`type`验证的使用。 识别的类型值是：

* `string`: Must be of type `string`. `This is the default type.`
* `string`：必须是`string`类型。 `这是默认的类型.`
* `number`: Must be of type `number`.
* `number`：必须是类型`number`的。
* `boolean`: Must be of type `boolean`.
* `boolean`：必须是`boolean`类型。
* `method`: Must be of type `function`.
* `method`：必须是`function`类型。
* `regexp`: Must be an instance of `RegExp` or a string that does not generate an exception when creating a new `RegExp`.
* `regexp`：必须是`RegExp`的一个实例，或者是创建一个新的`RegExp`时不会产生异常的字符串。
* `integer`: Must be of type `number` and an integer.
* `integer`：必须是`number`类型和一个整数。
* `float`: Must be of type `number` and a floating point number.
* `float`：必须是`number`类型和一个浮点数。
* `array`: Must be an array as determined by `Array.isArray`.
* `array`：必须是由`Array.isArray`确定的数组。
* `object`: Must be of type `object` and not `Array.isArray`.
* `object`：必须是`object`类型，而不是`Array.isArray`。
* `enum`: Value must exist in the `enum`.
* `enum`：值必须存在于`enum`中。
* `date`: Value must be valid as determined by `Date`.
* `date`：由`Date`确定的值必须是有效的。
* `url`: Must be of type `url`.
* `url`：必须是`url`类型。
* `hex`: Must be of type `hex`.
* `hex`：必须是`hex`类型。
* `email`: Must be of type `email`.
* `email`：必须是`email`类型。

#### Required

The `required` rule property indicates that the field must exist on the source object being validated.

`required`规则属性表示该字段必须存在于正被验证的源对象上。

#### Pattern

The `pattern` rule property indicates a regular expression that the value must match to pass validation.

`pattern`规则属性表示该值必须匹配才能通过验证的正则表达式。

#### Range

A range is defined using the `min` and `max` properties. For `string` and `array` types comparison is performed against the `length`, for `number` types the number must not be less than `min` nor greater than `max`.
范围是使用`min`和`max`属性定义的。 对于`string`和`array`类型，比较是针对`length`执行的，对于`number`类型，数字不能小于`min`，也不能大于`max`。


#### Length

To validate an exact length of a field specify the `len` property. For `string` and `array` types comparison is performed on the `length` property, for the `number` type this property indicates an exact match for the `number`, ie, it may only be strictly equal to `len`.

要验证字段的确切长度，请指定`len`属性。 对于字符串和数组类型比较是在`length`属性上执行的，对于`number`类型，这个属性表示`number`的确切匹配，也就是说，它只能严格等于`len`。

If the `len` property is combined with the `min` and `max` range properties, `len` takes precedence.

如果`len`属性与`min`和`max`范围属性相结合，`len`优先。

#### Enumerable

To validate a value from a list of possible values use the `enum` type with a `enum` property listing the valid values for the field, for example:

要从可能值列表中验证值，可以使用带enum属性的`enum`类型列出该字段的有效值，例如：

```javascript
var descriptor = {
  role: {type: "enum", enum: ['admin', 'user', 'guest']}
}
```

#### Whitespace

It is typical to treat required fields that only contain whitespace as errors. To add an additional test for a string that consists solely of whitespace add a `whitespace` property to a rule with a value of `true`. The rule must be a `string` type.

通常将仅包含空白的必填字段视为错误。 要为仅包含空格的字符串添加额外的测试，请将'whitespace'属性添加到值为“true”的规则中。 该规则必须是“字符串”类型。

You may wish to sanitize user input instead of testing for whitespace, see [transform](#transform) for an example that would allow you to strip whitespace.

您可能希望清理用户输入，而不是测试空白，请参阅[transform]（＃transform），该示例将允许您删除空白。

#### Deep Rules

If you need to validate deep object properties you may do so for validation rules that are of the `object` or `array` type by assigning nested rules to a `fields` property of the rule.

如果您需要验证深层对象属性，则可以通过为规则的“fields”属性分配嵌套规则来验证“object”或“array”类型的验证规则。

```javascript
var descriptor = {
  address: {
    type: "object", required: true,
    fields: {
      street: {type: "string", required: true},
      city: {type: "string", required: true},
      zip: {type: "string", required: true, len: 8, message: "invalid zip"}
    }
  },
  name: {type: "string", required: true}
}
var validator = new schema(descriptor);
validator.validate({ address: {} }, (errors, fields) => {
  // errors for street, address.city, address.zip and address.name
});
```

Note that if you do not specify the `required` property on the parent rule it is perfectly valid for the field not to be declared on the source object and the deep validation rules will not be executed as there is nothing to validate against.

请注意，如果您没有在父规则中指定`required`属性，则对于不在源对象上声明的字段非常有效，并且由于没有任何要验证的内容，深度验证规则将不会被执行。

Deep rule validation creates a schema for the nested rules so you can also specify the `options` passed to the `schema.validate()` method.

深入规则验证为嵌套规则创建了一个模式，因此您还可以指定传递给`schema.validate()`方法的`选项`。

```javascript
var descriptor = {
  address: {
    type: "object", required: true, options: {single: true, first: true},
    fields: {
      street: {type: "string", required: true},
      city: {type: "string", required: true},
      zip: {type: "string", required: true, len: 8, message: "invalid zip"}
    }
  },
  name: {type: "string", required: true}
}
var validator = new schema(descriptor);
validator.validate({ address: {} }, (errors, fields) => {
  // now only errors for street and name
});
```

The parent rule is also validated so if you have a set of rules such as:
父规则也会得到验证，所以如果您有一组规则，例如：

```javascript
var descriptor = {
  roles: {
    type: "array", required: true, len: 3,
    fields: {
      0: {type: "string", required: true},
      1: {type: "string", required: true},
      2: {type: "string", required: true}
    }
  }
}
```

And supply a source object of `{roles: ["admin", "user"]}` then two errors will be created. One for the array length mismatch and one for the missing required array entry at index 2.

并提供`{roles: ["admin", "user"]}`的源对象，那么将创建两个错误。 一个用于数组长度不匹配，另一个用于索引2处缺少的所需数组条目。

#### defaultField

The `defaultField` property can be used with the `array` or `object` type for validating all values of the container.
It may be an `object` or `array` containing validation rules. For example:

'defaultField`属性可以与`array`或`object`类型一起使用来验证容器的所有值。
它可能是一个包含验证规则的`object`或`array`。 例如：

```javascript
var descriptor = {
  urls: {
    type: "array", required: true,
    defaultField: {type: "url"}
  }
}
```

Note that `defaultField` is expanded to `fields`, see [deep rules](#deep-rules).

请注意，`defaultField`扩展为`fields`，参见[deep rules]（＃deep-rules）。

#### Transform

Sometimes it is necessary to transform a value before validation, possibly to coerce the value or to sanitize it in some way. To do this add a `transform` function to the validation rule. The property is transformed prior to validation and re-assigned to the source object to mutate the value of the property in place.

有时候有必要在验证之前对值进行转换，可能会强制价值或以某种方式对其进行消毒。 要做到这一点了`transform`函数添加到验证规则。 该属性在验证之前被转换并重新分配给源对象以改变该属性的值。

```javascript
var schema = require('async-validator');
var sanitize = require('validator').sanitize;
var descriptor = {
  name: {
    type: "string",
    required: true, pattern: /^[a-z]+$/,
    transform(value) {
      return sanitize(value).trim();
    }
  }
}
var validator = new schema(descriptor);
var source = {name: " user  "};
validator.validate(source, (errors, fields) => {
  assert.equal(source.name, "user");
});
```

Without the `transform` function validation would fail due to the pattern not matching as the input contains leading and trailing whitespace, but by adding the transform function validation passes and the field value is sanitized at the same time.

如果没有`transform`函数，由于模式不匹配，验证会失败，因为输入包含前导空白和尾随空白，但通过添加转换函数验证传递并且字段值同时被消毒。

### Messages

Depending upon your application requirements, you may need i18n support or you may prefer different validation error messages.

根据您的应用程序要求，您可能需要国际化支持，或者您可能更喜欢不同的验证错误消息。

The easiest way to achieve this is to assign a `message` to a rule:
实现这一目标最简单的方法是将'message`分配到一个规则：

```javascript
{name:{type: "string", required: true, message: "Name is required"}}
```

Message can be any type, such as jsx format.
消息可以是任何类型，例如jsx格式。

```javascript
{name:{type: "string", required: true, message: <b>Name is required</b>}}
```

Potentially you may require the same schema validation rules for different languages, in which case duplicating the schema rules for each language does not make sense.

有可能您需要针对不同语言使用相同的模式验证规则，在这种情况下，重复每种语言的模式规则是没有意义的。

In this scenario you could just provide your own messages for the language and assign it to the schema:

在这种情况下，您可以为语言提供自己的消息并将其分配给模式：

```javascript
var schema = require('async-validator');
var cn = {
  required: '%s 必填',
};
var descriptor = {name:{type: "string", required: true}};
var validator = new schema(descriptor);
// deep merge with defaultMessages
validator.messages(cn);
...
```

If you are defining your own validation functions it is better practice to assign the message strings to a messages object and then access the messages via the `options.messages` property within the validation function.

如果您正在定义自己的验证函数，最好将消息字符串分配给消息对象，然后通过验证函数中的`options.messages`属性访问消息。

### validator

you can custom validate function for specified field:

```js
const fields = {
  asyncField:{
    validator(rule,value,callback){
      ajax({
        url:'xx',
        value:value
      }).then(function(data){
        callback();
      },function(error){
        callback(new Error(error))
      });
    }
  },
 
  promiseField:{
      validator(rule, value){
        return ajax({
          url:'xx',
          value:value
        });
      }
    }
};
```

## Test Case

```
npm test
npm run chrome-test
```

## Coverage

```
npm run coverage
```

open coverage/ dir

## License

Everything is [MIT](http://en.wikipedia.org/wiki/MIT_License).
