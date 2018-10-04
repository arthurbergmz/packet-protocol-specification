# Packet Protocol

This is the official specification's repository for Packet Protocol.

## What is Packet Protocol?

Packet Protocol is a method of serializing structured data involving an interface definition language.

It generates source code from definitions written in `.packet` files, that can be used to generate or parse a stream of bytes that represents the structured data of the definitions.

## How does it work?

You specify how you want the information you are serializing to be structured, in `.packet` files. Each structured data contains a series of name-value pairs.

The data is serialized based in the order the fields are declared.

The following example is a `.packet` file that defines a structure containing informations about a person:

```go
enum PhoneType {
  MOBILE,
  HOME,
  WORK
}

type PhoneNumber {
  PhoneType type = PhoneType.MOBILE;
  string number;
}

type uuid string

type Person {
  uuid id;
  string name;
  optional string email;
  PhoneNumber[] phoneNumbers;
}
```

When you serialize `Person`, the output structure will always be presented in the order the fields are written: `id` will be the first data in the output, `name` will be the second one and so on...

Example of usage of the generated source code in JavaScript:

```javascript
import { Person, PhoneType } from './packets'

function isEqual(a = {}, b = {}) {
  const keys = Object.keys(a)
  if (keys.length !== Object.keys(b).length) {
	return false
  }
  for (const k of keys) {
    if (a[k] !== b[k]) return false
  }
  return true
}

const bob = {
  id: '8999eca6-6d90-481f-a12a-f7050c1c3f7f',
  name: 'Bob',
  email: 'bob@somewhere.net',
  phoneNumbers: [
    {
      type: PhoneType.MOBILE,
      number: '1-234-5678-910'
    }
  ]
}

Person.serialize(bob)
  .then(Person.deserialize)
  .then((person) => console.log(isEqual(person, bob))) // true
```

## Goals

* Provide a safe and fast data interchange format;
* Provide easy syntax, targeting readability and maintenance;

## Interface definition language

### import

Packet Protocol offers modules made easy. Every protocol block can be accessed from outside, you just need to write some import declarations _à la_ ES6, e.g.:

```javascript
import "./person"
// adds all protocol blocks from "./person.packet" to this scope

import "./person" as PersonPackets
// adds all protocol blocks from "./person.packet" to this scope under the PersonPackets name

import { Person } from "./person"
// only adds Person to this scope, from "./person.packet"

import { Person as Individual } from "./person"
// only adds Person to this scope under the name Individual, from "./person.packet"

import { Person, Belonging as PersonBelonging } from "./person"
// only adds Person and Belonging - under the name PersonBelonging - to this scope, from "./person.packet"

import { Person as Individual, Belonging } from "./person"
// only adds Person, under the name Individual, and Belonging to this scope, from "./person.packet"
```

The file extension `.packet` is optional. Packet Protocol also offers support for tree-shaking in import declarations.

### type

Packet Protocol already provides some built-in types, such as `string`, `bool` and `int`, but types have a primary focus on user-defined types, where you can build your own structs of data, e.g.:

```go
// creates a type "Person" for structured data
type Person {
  string name;
  int age;
}

// creates an alias for string ("uuid" now stands for the type "string")
type uuid string
```

#### built-in types

| type | description |
| ---- | ----------- |
| bool | Boolean data type. |
| bytes | An arbitrary sequence of bytes. |
| double | 64-bit floating point data type. |
| float | 32-bit floating point data type. |
| int32 | signed 32-bit integer |
| int64 | signed 64-bit integer |
| int | signed 32-bit integer, alias for `int32` |
| uint32 | unsigned 32-bit integer |
| uint64 | unsigned 64-bit integer |
| map<K,V> | Collection of key-value pairs. |
| string | UTF-8 encoded or 7-bit ASCII text. |

Maps' keys can only be `string` or `int`.

#### structured data type

As you can see, you can create structured data types, but there are a few more features that you can use:

```go
type uuid string

type Owner {
  uuid id;
  string name;
}

enum PetType {
  DOG,
  CAT,
  BIRD,
  CAPYBARA,
  UNICORN
}

type Pet {
  uuid id;
  string name;
  PetType type;
  optional Owner owner;
  bool hasCollar = false;
  (string | bytes)[] pictures;
}
```

##### optional

e.g.: `optional Owner owner;`

When a field is `optional`, the packet is parsed even if there is no value assigned. When a field is not specified as `optional`, an assigned value is always required so the packet can be parsed.

##### default values

e.g.: `bool hasCollar = false;`

You can assign default values to non-optional fields, so when a value is initially not assigned to this field, it takes its default value.

In the given example, if you don't assign a boolean value to `hasCollar`, it takes `false` as default.

##### collections

e.g.:
1. `(string | bytes)[] pictures;`
2. `string[] options;`

Collection of elements of the given type (array).

##### mixed type

e.g.: `(string | bytes)[] pictures;`

A mixed type field any value that respects the given types.

In the example above, there is a mixed _collection_ of `string` and `bytes`.

### enum

There is also support for enumerations. Enumerations enable lists of variables of a given type, e.g.:

```c
// remember that the type "uuid", that we created above, stands for "string"
enum uuid Planet {
  MERCURY = "b72a7779-6c6b-40d7-a7a8-ce10a576af99",
  VENUS = "4d32e1c7-567e-49dc-a570-e28de16596ec",
  EARTH = "bac6add4-183a-4316-b9d4-cee4b69ae105",
  MARS = "9535567c-3c20-4ab0-aaf1-aba17ef3613b",
  JUPITER = "44ce201c-23ba-4d4d-ab3f-7c79eb7e38ef",
  SATURN = "f33ee938-7d19-41e8-b64a-7e1392b7a404",
  URANUS = "f5277c0a-7ef1-431a-a26a-4302ab5b5200",
  NEPTUNE = "788bc165-716c-46f2-b25d-912627cfe7ea"
}
```

If no type is specified, it falls back to `int`. With that in mind, the following enumerations are equivalent:

```c
enum Day {
  SUNDAY,
  MONDAY,
  TUESDAY,
  WEDNESDAY,
  THURSDAY,
  FRIDAY,
  SATURDAY
}
```

```c
enum int Day {
  SUNDAY = 0,
  MONDAY = 1,
  TUESDAY = 2,
  WEDNESDAY = 3,
  THURSDAY = 4,
  FRIDAY = 5,
  SATURDAY = 6
}
```

### wrapper

Wrappers provides a generic field `type`, allowing standards in the data interchange.

```go
wrapper ApiRequest<type> {
  uuid userId; // "uuid" stands for "string", as stated previously
  type data;
}
```

```go
wrapper ApiResponse<type> {
  int timestamp;
  type data;
}
```

## Author

Created and initially developed by [Arthur Arioli Bergamaschi](https://github.com/arthurbergmz).

## Contributing

That is so cool! See `CONTRIBUTING.md` for guidelines about how to proceed.

## License

See `LICENSE` for more information.

---

> _**Disclaimer:** Packet Protocol is still a work in progress. Definitions and behavior can change along the way._
