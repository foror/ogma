# Ogma
A programming language for the experimental validation of new concepts in application and systems software development.

## Key Concepts
+ **No fixed syntax.** Source code is represented in binary form as an AST-like tree. Types and methods are stored in a database, allowing methods and types to be locked during real-time collaborative development to avoid conflicts. A specialized editor converts your syntax into this binary representation and back. You can write in a Python-like style while your colleague uses a C-like style, yet both of you can work on the same project. In your editor, your colleague’s code will be displayed in your preferred Python-like style.
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

bind [i8, ogma.number.i8/[unsafe, little-endian]]
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

### Additions
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

## Packages and Namespace

A package is necessary to avoid naming conflicts and therefore must be unique. To achieve this, use the organization name combined with the library or application name, separated by a dot. For example, **tau.survik** would be a good package name.

A namespace is necessary for grouping types around a particular concept. For example, when creating a blog, **Article** can serve as a namespace for **Entity.ogma**, **Repo.ogma**, and **Service.ogma**. In the file system, it would look like this:
+ tau/site/Article/
    - Entity.ogma
    - Repo.ogma
    - Service.ogma

```
::tau.site.Article -- the import applies only to the following line
:Entity#(foo, bar) -- automatically creates the entity variable
```
```
-- Bar.ogma

-- only working for default type method
-- DI sets the value of the article_repo field and creates it
::tau.site.Article.Repo ~ get:by_pk(foo) => article:Entity
stdio ~
    put(article.get:title)
    put(article.get:author().get:name)
```
```
-- Foo.ogma
::tau.site.[Article, Comment, Author]

"""
# start of the Foo constructor
article_repo, comment_repo, and author_repo fields are created automatically
DI sets the values of these fields
"""
#
    :Article.Repo 
    :Comment.Repo
    :Author.Repo

::Article
print:by_id(Key.Long<Entity> pk):
    """
    the Article namespace has priority throughout
    the scope of the method
    """
    article_repo.get:by_pk(pk) => article:Entity
    stdio ~
        auto:nel
        put(article.get:title)
        put(article.get:author().get:name)

        loop [v,] of article.get:comments
            manual:nel
            put(v.get:created)
            nel()
```

## Loop
```
loop [i, v] of array   
  @info array.get i

loop i:ix = array.get:high   
  @info array.get(i)
with i-- and i >= 0

loop [i:ix = array.get:high, j:u32 = 0]   
  @info array.get(i)
with [i--, j++] and [i > 0, j < 10]
```

## Methods

```
foo(bar:""): <i8>
    ...
    -- return result as value of i8 type
    result =

foo:
    ...

foo:(baz:i8):
    ...

draw:
    -- methods using colon notation can be called without parentheses
    draw:head
    draw:body

draw:head:
    ...

draw:body:
    ...

get:by_id(pk:i64): Quz
    quz_repo.fetch:by_id(pk) =

-- static methods must be called using the Foo#baz() syntax
#baz:
    ...
```

## Cast
```
foo:f32

-- to string with pattern
foo.("%.2f").get:length

-- to integer
foo.(i32)
```

## Cross-section
```
product_prices:<Entity.Ref<Product::[...]>> = mormont.fetch(
    Product::[pk, price]
)

product:Product::[...] = product_prices.get:first
product.get:price -- ok
product.get:name -- a compile-time error
```

## Builder
```
-- like String foo = new StringBuilder(1024).append(...).build()
""...#1024 ~
    append(“foo“)
    append(“bar“)
    build() => foo:""

1.0f.(f32.Formatter...)
.comma(2)
.science(true)
.build()
```
