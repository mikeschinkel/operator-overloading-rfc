# PHP RFC: Operator Overloading

## Introduction

This RFC aims to provide basic operator overloading within PHP for objects.

## Background

Operator overloading is a well explored concept in programming that enables a programmer to control the behavior of their code when combined with an infix, or operator. Some languages such as R allow for defining custom operators in addition to overloading existing operators. Some languages such as Python allow for overloading all existing operators, but not for defining new operators. Some languages such as Java do not allow for custom operator overloading at all.

### Existing Operators in PHP

**Mathematical Operators**

| Symbol | Description |
| ------ | ----------- |
| **+** | Used for addition with int and float, union with arrays |
| **-** | Used for subtraction with int and float |
| **\*** | Used for multiplication with int and float |
| **/** | Used for division with int and float |
| **%** | Used for modulo with int |
| **\*\*** | Used for pow with int |

**String Operators** 

| Symbol | Description |
| ------ | ----------- |
| **.** | Used for string concatenation |

**Comparison Operators**

| Symbol | Description |
| ------ | ----------- |
| **==** | Equals comparison with type casting |
| **===** | Identity comparison |
| **>** | Greater than comparison |
| **<** | Less than comparison |
| **>=** | Greater than or equal to comparison |
| **<=** | Less than or equal to comarpison |
| **!=** | Not equals comparison with type casting |
| **<>** | Not equals comparison with type casting |
| **!==** | Not identical comparison |
| **<=>** | Sort hierarchy comparison |

**Bitwise Operators**

| Symbol | Description |
| ------ | ----------- |
| **&** | Bitwise and |
| **\|** | Bitwise or |
| **^** | Bitwise xor |
| **~** | Bitwise not |
| **<<** | Bitwise shift left |
| **>>** | Bitwise shift right |

**Logical Operators**

| Symbol | Description |
| ------ | ----------- |
| **and** | Logical and operator |
| **or** | Logical or operator |
| **xor** | Logical xor operator |
| **&&** | Logical and operator |
| **\|\|** | Logical or operator |
| **!** | Logical negation operator |

**Misc. Operators**

| Symbol | Description |
| ------ | ----------- |
| **??** | Null coalesce |
| **? :** | Ternary operator |
| **@** | Error suppression operator |
| **\`\`** | Shell execution escape operator |

**Assignment Operators**

| Symbol | Example | Equivalent | Description |
| ------ | ------- | ---------- | ----------- |
| **+=** | $a += $b | $a = $a + $b | Add assignment operator |
| **-=** | $a -= $b | $a = $a - $b | Subtract assignment operator |
| **\*=** | $a \*= $b | $a = $a * $b | Multiply assignment operator |
| **/=** | $a /= $b | $a = $a / $b | Divide assignment operator |
| **%=** | $a %= $b | $a = $a % $b | Modulo assignment operator |
| **\*\*\=** | $a \*\*= $b | $a = $a \*\* $b | Pow assignment operator |
| **&=** | $a &= $b | $a = $a & $b | Bitwise and assignment operator |
| **\|=** | $a \|= $b | $a = $a \| $b | Bitwise or assignment operator |
| **^=** | $a ^= $b | $a = $a ^ $b | Bitwise xor assignment operator |
| **<<=** | $a <<= $b | $a = $a << $b | Bitwise shift left assignment operator |
| **>>=** | $a >>= $b | $a = $a >> $b | Bitwise shift right assignment operator |
| **.=** | $a .= $b | $a = $a . $b | Concatenation assignment operator |
| **??=** | $a ??= $b | $a = $a ?? $b | Null coalesce assignment operator |

## Commutativity

Commutativity refers to the ability of operands to be reversed while retaining the same result. That is:

```php
<?php

$a + $b === $b + $a;
```

### Commutativity In PHP

Some operators are always commutative in PHP currently, while others are not:

```php
<?php

// Arithmetic
$a + $b === $b + $a; // This is true for int + int, int + float, float + int, and array + array
$a - $b !== $b - $a; // Subtraction by definition is not commutative
$a * $b === $b * $a; // This is true for numeric types
$a / $b !== $b / $a; // Division by definition is not commutative

// Other Math
$a % b !== $b % $a; // Modulo by definition is not commutative
$a ** $b !== $b ** $a; // Pow by definition is not commutative
```

The mathematical rules for commutativity depend on the type of mathematical object which is being considered. All integers and floats fall within the real numbers, and so natively both the `+` and `*` operators are commutative for all values because this is a property of real numbers.

However, for other mathematical objects rules for commutativity are different. Considering matrices, multiplication is no longer commutative.[1] Thus **enforcing the existing commutative behavior may enforce incorrect behavior on user code**. For this reason, this RFC makes no effort to enforce commutativity, as doing so will in reality introduce bugs to the behavior of the operators for various domains.

There is more argument for enforcing commutativity for the **logical operators**, which definitionally should be commutative if they are used as logical operators. Doing so would preclude libraries and user applications from repurposing the logical operators for another purpose in some circumstances. However, as that is not part of this RFC and is left as future scope, it has no impact on this proposal.

### Commutativity in Other Languages

Python's implementation of operator overloads most closely matches this proposal, and so for the purpose of comparison we will consider how Python's operator overloading interacts with commutativity.

In Python, all operators can be overloaded except for the logical operators. These are provided in two groups:

**Rich Comparison**

The comparison operators are implemented as a single method each, with a default implementation for `==` and `!=`. Rich comparison operators are not commutative in Python. For examble, if you have two objects `x` and `y`, then the following will happen for this comparison `x >= y`:

1. If `x` implements `__ge__`, then `x.__ge__(x, y)` will be called.
2. If `x` does not implement `__ge__` then precedence falls to `y`.
3. If `y` implements `__le__`, then `y.__le__(y, x)` will be called.
4. If `y` does not implement `__le__` then the default operator behavior is used.

NOTE: Python actually gives precedence to `y` in the above example if `y` is a direct or indirect subclass of `x`.

In Python, the comparison operators are not directly commutative, but have a reflected pair corresponding with the swapped order. However, each object could implement entirely different logic, and thus no commutativity is enforced.

**Numeric Operators**

The numeric operators (including the bitwise operators) are implemented each as a family of three overrides: `__op__`, `__iop__`, and `__rop__`.

The `__op__` method is called when the object is the left operand of an operator. The `__rop__` method is called when the object is the right operand of an operator. The `__iop__` method is called when the corresponding reassignment operator is called (always on the assigned object).

It is easy to see from this set of implementations that not only is commutativity not preserved, but full support for breaking commutativity in a controlled and intelligent way is supported.

## Proposal

This RFC proposes adding magic methods to PHP to control operator overloading for a limited set of operators. This RFC only proposes overloads for two part operations, or stated differently, no unary operations are proposed for inclusion in this RFC. As such, all of the magic methods fit a single general format:

```php
<?php

Class A {
  
  public function __op(mixed $self, mixed $other, bool $left): mixed {
  }
  
}
```

The operators are always passed in the order `$self` then `$other` regardless of whether the called object is the left or right operand. If the called object is the left operand, the `$left` is `true`. If the called object is the right operand, the `$left` is `false`.

The purpose in passing `$self` to the function despite the fact that this is not a static function is to allow parent classes to behave differently for subclasses if a single inherited override is desired. Though this could be accomplished by examining `$this` instead, passing `$self` in a parameter allows for typing of the argument and thus makes it easier for IDE's and static analysis tools to determine useful information about the operator behavior.

A new exception, `InvalidOperator`, is also provided for users to throw within the operator magic methods if for any reason the operation is invalid due to type constraints, value constraints, or other factors.

The default types for the arguments and return are `mixed`, to allow user code to further narrow the type as appropriate for their application.

### Comparison Operators

Partial support for comparison operators is also part of this RFC. Comparison operators in Python, detailed briefly in the section on commutativity in other languages, **does not** restrict the return to a boolean value. While there may be many use cases for overloading the comparison operators in a way that does not return a boolean, in the interest of promoting consistency with existing code, the magic methods for the coparison operators have the additional restriction of returning `bool` instead of `mixed`.

They may still violate commutativity, as this is a possibly useful feature. Additionally, they can also still throw exceptions, including the `InvalidOperator` exception.

```php
<?php

Class A {

  public function __comparisonOp(mixed $self, mixed $other): bool {
  }

}
```

As the comparison operators involve a reflection relationship instead of a commutative one, the same behavior as detailed in the Python section will be used.

| Left Operand Method | Right Operand Method |
| ------------------- | -------------------- |
| `__lessThan()` | `__greaterThan()` |
| `__greaterThan()` | `__lessThan()` |
| `__lessThanOrEq()` | `__greaterThanOrEq()` |
| `__greaterThanOrEq()` | `__lessThanOrEq()` |
| `__equals()` | `__equals()` |
| `__notEquals()` | `__notEquals()` |

The spaceship operator (`<=>`), used to determine sort hierarchy and encompasing all comparisons for numeric values, is also supported. However its reflection and definition is slightly different:

```php
<?php

Class A {

  public function __compareTo(mixed $self, mixed $other): int {
  }

}
```

| Left Operand Method | Right Operand Method |
| ------------------- | -------------------- |
| `__compareTo()` | `__compareTo() * -1` |

### Supported Operators

In this RFC only a subset of the operators in PHP are supported for operator overloading. The proposed operators are:

| Operator | Method Name |
| -------- | ----------- |
| `+` | `__add()` |
| `-` | `__sub()` |
| `\*` | `__mul()` |
| `/` | `__div()` |
| `%` | `__mod()` |
| `\*\*` | `__pow()` |

## Backward Incompatible Changes

## Proposed PHP Version

## RFC Impact

## Future Scope

## Proposed Voting Choices

## Vote

## Patches and Tests

## References

[1]: https://en.wikipedia.org/wiki/Matrix_multiplication#Non-commutativity

## Changelog
