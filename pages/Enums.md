# Enums

Enums allow us to define a set of named numeric constants. 
An enum can be defined using `enum` keyword.

```TypeScript
enum Direction {
    Up = 1,
    Down,
    Left,
    Right
}

```

The body of an enum consists of zero or more enum members. 
Enum members have numeric value associated with them and can be either *constant* or *computed*.
An enum member is considered constant if:
    * It does not have an initializer and the preceding enum member was constant. 
      In this case the value of the current enum member will be the value of the preceding enum member plus one.
      One exception to this rule is the first element on an enum. 
      If it does not have initializer it is assigned value `0`.
    * The enum member is initialized with a constant enum expression. 
      A constant enum expression is a subset of TypeScript expressions that can be fully evaluated in compile time.
      Expression is constant enum expression if it is either:
        * numeric literal
        * reference to previosly defined constant enum member (it can be defined in different enum).
          If member is defined in the same enum it can be referenced using unqualified name.
        * parenthesized constant enum expression
        * `+`, `-`, `~` unary operators applied to constant enum expression
        * `+`, `-`, `*`, `/`, `%`, `<<`, `>>`, `>>>`, `&`, `|`, `^` binary operators with constant enum expressions as operands
      It is a compile time error for constant enum expressions to be evaluated to `NaN` or `Infinity`.

In all other cases enum member is considered computed.

```TypeScript
enum FileAccess {
    // constant members
    None,
    Read    = 1 << 1,
    Write   = 1 << 2,
    ReadWrite  = Read | Write
    // computed member
    G = "123".length
}
```

Enums are real objects that exist in runtime, one of reasons is ability to maintain reverse mapping from enum values to enum names.

```TypeScript
enum Enum {
    A
}
let a = Enum.A;
let nameOfA = Enum[Enum.A]; // "A"

```
is compiled to

```javascript
var Enum;
(function (Enum) {
    Enum[Enum["A"] = 0] = "A";
})(Enum || (Enum = {}));
var a = Enum.A;
var nameOfA = Enum[Enum.A]; // "A"

```

In generated code enum is compiled into an object that stored both forward (`name` -> `value`) and reverse (`value` -> `name`) mappings. 
References to enum members are always emitted as property access and never inlined.
In lots of cases this is perfectly valid solution however sometimes requirements might be more tight.
To avoid paying the cost of extra generated code and addition indirection when accessing enum values it is possible to use const enums.
Const enums are defined using `const` modifier that precedes `enum` keyword.

```TypeScript
const enum Enum {
    A = 1,
    B = A * 2
}
```

Const enums are can only use constant enum expressions and unlike regular enums they are completely removed during compilation.
Const enum members are inlined at use sites (it is possible since const enums cannot have computed members).

```TypeScript
const enum Directions {
	Up,
	Down,
	Left,
	Right
}

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right]
```

in generated code will become

```javascript
var directions = [0 /* Up */, 1 /* Down */, 2 /* Left */, 3 /* Right */];
```

# Ambient enums
Ambient enums are used to describe the shape of already existing enum types. 

```TypeScript
declare enum Enum {
    A = 1,
    B,
    C = 2
}
```

One important different between ambient and non-ambient enums is: in regular enums members that don't have initializer are considered constant members.
For non-const ambient enums member that does not have initializer is considered computed.