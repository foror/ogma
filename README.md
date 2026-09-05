# Ogma
A programming language for the experimental validation of new concepts in application and systems software development.

## Key Concepts
+ **No fixed syntax.** Source code is represented in binary form as an AST-like tree. A specialized editor converts your syntax into this binary representation and back. You can write in a Python-like style while your colleague uses a C-like style, yet both of you can work on the same project. In your editor, your colleague’s code will be displayed in your preferred Python-like style.
+ **Hybrid memory management without GC.** The compiler and built-in AI identify places where memory cleanup snippets should be inserted. If necessary, you can manually edit the automatically inserted snippets.
+ **Built-in IoC at the compiler level.** The compiler provides a Dependency Injection system that allows you to replace any class in system and external libraries without requiring classes to declare common interfaces.
+ **A lightweight IDE as an extension of the compiler.** It provides a comfortable development environment with powerful refactoring tools and code change history.
+ **Strict static typing and an object-oriented programming paradigm.** Designed for operating system and application software development.
+ **A lightweight runtime for microcontroller development.** Development of control programs for applications ranging from simple mechanisms to CNC machines.

## Hello, World!
```
-- Hello.ogma
@common ::

stdio ~
  put("Hello, World!")
  nel()
```
## Dependency Injection
To register in the DI container, add one of the following macros to the first line of the source code:
+ **@common** initializes a single object using a parameterless constructor. The **::method_name** parameter can be added to synchronously execute a method of the object immediately after initialization. For an object with no methods, the default method (the code following @common) will be called. To execute the default method, add **::**
+ **@custom** is similar to **@common**, but requires explicit object initialization in the DI module.

Additional parameters: [**#repeat**, **#per-thread**, **#async**, **#sync**, **#task**]
```
@common #repeat
< Repeatable

on:repeat: Duration?    
  ...
on:cancel:    
  ...
delay: Duration
  ...
```

## Tagging
Types may have similar interfaces but different implementations. To highlight type-specific characteristics, Ogma uses tagging:
#### Common interface i8 - an integer type 1 byte in size
+ **i8/safe** implements mandatory overflow checking for all types of operations
+ **i8/unsafe** performs no overflow checking
+ **i8/[little-endian, safe]**
+ **i8/[little-endian, unsafe]**
+ **i8/[big-endian, safe]**
+ **i8/[big-endian, unsafe]**

For **List**, this can be a set of the following types: **List/array**, **List/linked**, **List/[copy-on-write, array]**, etc.

The DI module defines which of the presented types will represent i8 by default. It is also possible to define scopes by specifying packages where i8 will be represented by a specific type. Such rules can be overridden in other modules.
```
-- Unit.ogma
@unit

define [i8, ogma.number.i8/[unsafe, little-endian]]
auto:import i8 -- Automatically import i8 into the source code of other types
```
```
-- Foo.ogma

foo:i8#42 -- initialize the value 42 as type ogma.number.i8/[unsafe, little-endian]
```
## Type system
Any Ogma source code file represents a specific type. The name is taken from the file name, so it does not need to be specified in the source code. The type itself defaults to **@type**, so it does not need to be specified either. However, in some cases, the type must be explicitly defined:

+ **@basic**
  - the type name is always lowercase
  - passed by value
  - the order of fields in the file is significant
  - fields are inaccessible from outside
  - not inheritable
  - does not store the type in the object
+ **@struct**
  - the type name is always uppercase
  - fields are directly accessible (behavior can be changed using the @[get, set] macros)
  - the order of fields in the file is significant
  - passed by value to containers (by reference Foo^)
  - passed by reference to method parameters and returned from methods by reference (by value Foo')
  - passed by value to fields of other types (by reference Foo^)
  - does not store the type in the object
+ **@form**
  - the type name is always uppercase
  - all fields have @[get, set]
  - stores the type in the object
  - the order of fields in the file is significant
  - passed by reference to containers (by value Foo')
  - passed by reference to fields of other types (by value Foo')
  - passed by reference to method parameters and returned from methods by reference (by value Foo')
  - multiple inheritance
+ **@type**
  - the type name is always uppercase
  - fields are private (behavior can be changed using the @[get, set] macros)
  - stores the type in the object
  - the order of fields in the file is significant
  - passed by reference to containers (by value Foo')
  - passed by reference to fields of other types (by value Foo')
  - passed by reference to method parameters and returned from methods by reference (by value Foo')
  - multiple inheritance

### Addition types
+ **@tree** for convenient representation of hierarchical data in source code
+ **@enum** enumeration
+ **@trait** interface
+ **@prot** abstract type

### Abbreviations
+ [] - Array/safe
+ <> - List/[array, safe]
+ <,> - Map/hash
+ "" - sx (Survik a hybrid SBCS/Unicode variable-length character encoding)
```
foo:[u32]
bar:<Qux>
quz:<i64, "">
baz:""
```

## Loop
```
loop [i, v] of array   
  @info array.get i

loop i:ux = array.get:high   
  @info array.get(i)
with i-- and i >= 0

loop [i:u32 = array.get:high, j:u32 = 0]   
  @info array.get(i)
with [i--, j++] and [i > 0, j < 10]
```
