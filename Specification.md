# YatScript Specification

*How should I write a specification document for programming languages?*

# Syntax

*TODO: How to write this section well?*

YatScript uses C-like syntax.

* Every statement ends with a semicolon(`;`).
* Block uses curly braces(`{ and }`).
* `//` for line comment, `/* */` for block comment.

# Primitives

## Boolean

Boolean values can be written as `true`, and `false`.

Logical operations will return `true` or `false` Boolean values. Boolean values are transpiled into `i32` type `1` and `0`.

## Number

There are two types of number, integer numbers, and floating point numbers.

Integer numbers can be written in decimal or `0x` prefixed hexadecimal.

Floating point numbers can be written in decimal. *Scientific notation?*

Numbers are transpiled into `{type}.const`. Types are automatically determined based on variables, operations, etc.

## String

*TODO: I think string won't be used*

# Variables

There are two types of variable, global variables, and local variables.

## Variable types

Like Wasm, `i32`, `i64`, `f32`, `f64` exist.

Integers can have signedness. `i32` and `i64` are signed.

Unsigned integers are `u32` and `u64`. Unsigned integers are transpiled into `i32` and `i64`.

## Global variable

Global variables are defined in module scope. It can be imported or exported.

Global variables can be immutable or mutable.

```
===[ global.yat ]===
// Initial values should be provided
global i32 $immmutable = 0;
global mut i32 $mutable = 0;

function $func() {
  $mutable = $mutable + $immutable;
}

===[ global.wat ]===
(module
  (global $immutable (i32) (i32.const 0))
  (global $mutable (mut i32) (i32.const 0))

  (func $func
    (set_global
      $mutable
      (i32.add (get_global $mutable) (get_global $immutable))
    )
  )
)
```

## Local variable

Local variables are defined in function scope.

Local variables can be defined anywhere in function.

```
===[ local.yat ]===
function $func() {
  local i32 $v1 = 1, i32 $v2 = 2, i32 $v3;
  v3 = v1 + v2;
}

===[ local.wat ]===
(module
  (func $func
    (local i32 v1) (local i32 v2) (local i32 v3)
    (set_local $v1 (i32.const 1))
    (set_local $v2 (i32.const 2))
    (set_local
      $v3
      (i32.add (get_local $v1) (get_local $v2))
    )
  )
)
```

## Type casting

If two operands of specific operators have different types, one of the values will be implicitly casted.

1. If one of the operand is `f64`, other operand will be casted to `f64`.
2. Otherwise, if one of the operand is `f32`, other operand will be casted to `f32`.
3. Otherwise, if one of the operand is `i64`, other operand will be casted to `i64`.

```
===[ cast.yat ]===
local i32 $v_i32_1 = 1, i32 $v_i32_2 = 2, i32 $v_i32_3;
local i32 $v_i64_1 = 1, i32 $v_i64_2 = 2, i32 $v_i64_3;

$v_i64_3 = $v_i32_1 + $v_i64_2;
$v_i32_3 = $v_i64_1 + $v_i32_2;

===[ cast.wat ]===
;; skipped local definitions

(set_local
  $v_i64_3
  (i64.add
    (i64.extend_s/i32 (get_local $v_i32_1)) ;; i32 -> i64
    (get_local $v_i64_2)
  )
)
(set_local
  $v_i32_3
  (i32.wrap/i64 ;; i64 -> i32
    (i64.add
      (get_local $v_i64_1)
      (i64.extend_s/i32 (get_local $v_i32_2)) ;; i32 -> i64
    )
  )
)
```

*TODO: explicit casting*

# Operators

## Assignation

* Assignation (`$a = $b`)
  * If `$a` and `$b` is same type, no type casting occurs
  * If `$a` and `$b` is different type, `$b` will be casted to `$a`'s type
  * Assignation to immutable global variable, or primitives are not allowed.

## Arithmetic

* Multiplication (`$a * $b`)
* Division (`$a / $b`)
  * If both operands are unsigned integers, it will be transpiled into unsigned division
* Remainder (`$a % $b`)
  * If both operands are unsigned integers, it will be transpiled into unsigned remainder
  * `f32` and `f64` types don't have remainder operation
* Addition (`$a + $b`)
* Subtraction (`$a - $b`)

## Bitwise

Bitwise operations will work with only integer operands.

* Bitwise NOT (`~$a`)
  * Will be transpiled into `$a ^ 0xffffffff` (`0xffffffffffffffff` for `i64` and `u64`)
* Bitwise AND (`$a & $b`)
* Bitwise XOR (`$a ^ $b`)
* Bitwise OR (`$a | $b`)
* Bitwise left shift (`$a << $b`)
  * Right operand should not be less than zero
* Bitwise right shift (`$a << $b`)
  * Operation's signedness will follow left operand's signedness
    * If left operand is a primitive integer, positive number will be treated as unsigned, negative number will be treated as signed
    * *TODO: unsigned shift for negative number?*
  * Right operand should not be less than zero
* Bitwise left rotate (`$a <<< $b`)
  * Right operand should not be less than zero
  * *TODO: should I use <<<?*
* Bitwise right rotate (`$a >>> $b`)
  * Right operand should not be less than zero
  * *TODO: should I use >>>?*

## Relational, Logical

Results of relational operators and logical operators are `0` for false, `1` for true in `i32` type.

* Logical NOT (`!$a`)
* Less than (`$a < $b`)
* Less than or equal to (`$a <= $b`)
* Greater than (`$a > $b`)
* Greater than or equal to (`$a > $b`)
* Equal to (`$a == $b`)
* Not equal to (`$a != $b`)
* Logical AND (`$a && $b`)
* Logical OR (`$a || $b`)

Operand of logical operators are implicitly casted into `i32 0` or `i32 1`.
* If operand has a value of `i32/i64 0` or `f32/f64 0.0`, value is casted into `i32 0`
* Otherwise, value is casted into `i32 1`

## Precedence

1. Logical NOT, Bitwise NOT
2. Multiplication, Division, Remainder
3. Addition, Subtraction
4. Bitwise shift, Bitwise rotate
5. Less than, Less than or equal to, Greater than, Greater than or equal to
6. Equal to, Not equal to
7. Bitwise AND
8. Bitwise XOR
9. Bitwise OR
10. Logical AND
11. Logical OR
12. Assignation

# Control structures

## Conditional statement (`if` statement)

`if` statement takes one expression for condition, one block to be executed when the condition is true, and optional `else` keyword and a block to be executed when the condition is false.

When a block has only one statement, wrapping curly braces are optional.

```
===[ if.yat ]===
if ($a == 1)
  $b = 1;

if ($b == 1) {
  $c = 1;
}
else {
  $c = 2;
  $d = 3;
}

===[ if.wat ]===
(if
  (i32.eq (get_local $a) (i32.const 1))
  (set_local $b (i32.const 1))
)
(if
  (i32.eq (get_local $b) (i32.const 1))
  (set_local $c (i32.const 1))
  (block
    (set_local $c (i32.const 2))
    (set_local $d (i32.const 3))
  )
)
```

## Iteration statement (`while` statement)

`while` statement takes one expression for condition, one block to be executed repeatedly until the condition is false or the `break` statement.

If condition is a primitive `1`, transpiler will optimize `while` loop as an infinite loop.

```
===[ while.yat ]===
while ($a < 10) {
  $b = $b + $a;
  $a = $a + 1;
}

while (1) {
  $c = $c + 1;

  if ($c == 10) break;
}

===[ while.wat ]===
(block $0
  (loop $1
    ;; if ($a < 10) is false, escape loop
    (br_if $0
      (i32.eq
        (i32.lt (get_local $a) (i32.const 10))
        (i32.const 0)
      )
    )
    (set_local $b (i32.add (get_local $b) (get_local $a)))
    (set_local $a (i32.add (get_local $a) (i32.const 1)))
    (br $1)
  )
)

(block $2
  (loop $3
    ;; condition is optimized
    (set_local $c (i32.add (get_local $c) (i32.const 1)))
    (if (i32.eq (get_local $c) (i32.const)) (br $2))
    (br $3)
  )
)
```

# Functions

# Module

One YatScript file will be transpiled into one WebAssembly(Wasm) module.

```
===[ empty.yat ]===

===[ empty.wat ]===
(module)
```

## Import

Like Wasm, functions, globals, memory, and tables can be imported.

```
===[ import.yat ]===
import function () as $func1 from env.func1;
import function (i32) as $func2 from env.func2;
import function () -> i32 as $func3 from env.func3;
import function (i32) -> i32 as $func4 from env.func4;

import global (i32) as $immutable from env.global1;
import global (mut i32) as $mutable from env.global2;

// import memory should be used only once, this is for demonstration
import memory(1) from env.memory1;
import memory(1, 4) from env.memory2;

// TODO: Table

===[ import.wat ]===
(module
  (import "env" "func1" (func $func1))
  (import "env" "func2" (func $func2 (param i32)))
  (import "env" "func3" (func $func3 (result i32)))
  (import "env" "func4" (func $func4 (param i32) (result i32)))

  (import "env" "global1" (global $immutable i32))
  (import "env" "global2" (global $mutable mut i32))

  (import "env" "memory1" (memory 1))
  (import "env" "memory2" (memory 1 4))
)
```

## Export

Like Wasm, functions, globals, memory, and tables can be exported.

```
===[ export.yat ]===
export function $func1 as func1;

export global $global1 as global1;

// TODO: Memory

// TODO: Table

===[ export.wat ]===
(module
  (export "func1" (func $func1))

  (export "global1" (global $global1))
)
```

## Main function

Like Wasm's `start` node, function named as `$main` will be implicitly transpiled as `start` node.

*TODO: explicitly define main function*

## Memory

*TODO*

## Data

*TODO*

## Table

*TODO*
