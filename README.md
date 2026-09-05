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
+ **@common** initializes a single object using a parameterless constructor. The **::method_name** parameter can be added to synchronously execute a method of the object immediately after initialization. For an object with no methods, the default method (the code following @common) will be called. To execute the default method, add **::**.
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
