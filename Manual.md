* * *

LuaBridge 3.0 Reference Manual
==============================

* * *

LuaBridge3 repository is located at [https://github.com/kunitoki/LuaBridge3](https://github.com/kunitoki/LuaBridge3).
Official LuaBridge (up to version 2) repository is located at [https://github.com/vinniefalco/LuaBridge](https://github.com/vinniefalco/LuaBridge).

*   Copyright © 2020 Asnaghi Lucio.
*   Copyright © 2019 Dmitry Tarakanov.
*   Copyright © 2012 Vinnie Falco.
*   Copyright © 2007 Nathan Reed.

Freely available under the terms of the [MIT License](http://www.opensource.org/licenses/mit-license.html).

Contents
--------

*   [1 - Introduction](#1---introduction)

    *   [1.1 - Design](#11---design)
    *   [1.2 - Repository](#12---repository)
    *   [1.3 - License and Credits](#13---license-and-credits)

*   [2 - Accessing C++ from Lua](#2---accessing-c-from-lua)

    *   [2.1 - Namespaces](#21---namespaces)
    *   [2.2 - Properties and Functions](#22---properties-and-functions)
    *   [2.3 - Class Objects](#23---class-objects)
    *   [2.4 - Property Member Proxies](#24---property-member-proxies)
    *   [2.5 - Function Member Proxies](#25---function-member-proxies)
    *   [2.5.1 - Function Overloading](#251---function-overloading)
    *   [2.6 - Constructors](#26---constructors)
    *   [2.6.1 - Constructor Proxies](#261---constructor-proxies)
    *   [2.6.2 - Constructor Factories](#262---constructor-factories)
    *   [2.7 - Extending Classes](#27---extending-classes)
        *   [2.7.1 - Extensible Classes](#271---extensible-classes)
        *   [2.7.2 - Index and New Index Metamethods Fallback](#272---index-and-new-index-metamethods-fallback)
    *   [2.8 - Lua Stack](#28---lua-stack)
        *   [2.8.1 - Enums](#281---enums)
        *   [2.8.2 - lua_State](#282---lua_state)

*   [3 - Passing Objects](#3---passing-objects)

    *   [3.1 - C++ Lifetime](#31---c-lifetime)
    *   [3.2 - Lua Lifetime](#32---lua-lifetime)
    *   [3.3 - Pointers, References, and Pass by Value](#33---pointers-references-and-pass-by-value)
    *   [3.4 - Shared Lifetime](#34---shared-lifetime)
        *   [3.4.1 - User-defined Containers](#341---user-defined-containers)
        *   [3.4.2 - shared_ptr As Container](#342---shared_ptr-as-container)
        *   [3.4.3 - Container Constructors](#343---container-constructors)
    *   [3.5 - Mixing Lifetimes](#35---mixing-lifetimes)
    *   [3.6 - Convenience Functions](#36---convenience-functions)

*   [4 - Accessing Lua from C++](#4---accessing-lua-from-c)

    *   [4.1 - Class LuaRef](#41---class-luaref)
        *   [4.1.1 - Lifetime, States and Lua Threads](#411---lifetime-states-and-lua-threads)
        *   [4.1.2 - Type Conversions](#412---type-conversions)
    *   [4.2 - Table Proxies](#42---table-proxies)
    *   [4.3 - Calling Lua](#43---calling-lua)
        *   [4.3.1 - Exceptions](#431---exceptions)
        *   [4.3.2 - Class LuaException](#432---class-luaexception)

*   [5 - Security](#5---security)

*   [Appendix - API Reference](#appendix---api-reference)

1 - Introduction
================

[LuaBridge](https://github.com/kunitoki/LuaBridge3) is a lightweight and dependency-free library for mapping data, functions, and classes back and forth between C++ and [Lua](http://wwww.lua.org), a powerful, fast, lightweight, embeddable scripting language. LuaBridge has been tested and works with Lua 5.1.5, 5.2.4, 5.3.6 and 5.4.4. It also works transparently with [LuaJIT](http://luajit.org/) 2.x onwards and for the first time also with [Luau](https://luau-lang.org/) 0.556 onwards and [Ravi](https://github.com/dibyendumajumdar/ravi) 1.0-beta11.

LuaBridge is usable from a compliant C++17 and offers the following features:

* [MIT Licensed](http://www.opensource.org/licenses/mit-license.html), no usage restrictions!
* Headers-only: No Makefile, no .cpp files, just one `#include` and one header file (optional) !
* Works with ANY lua version out there (PUC-Lua, LuaJIT, Luau, Ravi, you name it).
* Simple, light, and nothing else needed.
* No macros, settings, or configuration scripts needed.
* Supports different object lifetime management models.
* Convenient, type-safe access to the Lua stack.
* Automatic function parameter type binding.
* Easy access to Lua objects like tables and functions.
* Expose C++ classes allowing them to use the flexibility of lua property lookup.
* Interoperable with most common c++ standard library container types.
* Written in a clear and easy to debug style.

It also offers a set of improvements compared to vanilla LuaBridge:

* The only binder library that works with PUC-Lua as well as LuaJIT, Luau and Ravi, wonderful for game development !
* Can work with both c++ exceptions and without (Works with `-fno-exceptions` and `/EHsc-`).
* Can safely register and use classes exposed across shared library boundaries.
* Full support for capturing lambdas in all namespace and class methods.
* Overloaded function support in Namespace functions, Class constructors, functions and static functions.
* Supports placement allocation or custom allocations/deallocations of C++ classes exposed to lua.
* Lightweight object creation: allow adding lua tables on the stack and register methods and metamethods in them.
* Allows for fallback `__index` and `__newindex` metamethods in exposed C++ classes, to support flexible and dynamic C++ classes !
* Added `std::shared_ptr` to support shared C++/Lua lifetime for types deriving from `std::enable_shared_from_this`.
* Supports conversion to and from `std::nullptr_t`, `std::byte`, `std::pair`, `std::tuple` and `std::reference_wrapper`.
* Supports conversion to and from C style arrays of any supported type.
* Transparent support of all signed and unsigned integer types up to `int64_t`.
* Consistent numeric handling and conversions (signed, unsigned and floats) across all lua versions.
* Simplified registration of enum types via the `luabridge::Enum` stack wrapper.
* Opt-out handling of safe stack space checks (automatically avoids exhausting lua stack space when pushing values!).

LuaBridge is distributed as a a collection of header files. You simply add one line, `#include <LuaBridge/LuaBridge.h>` where you want to pass functions, classes, and variables back and forth between C++ and Lua. There are no additional source files, no compilation settings, and no Makefiles or IDE-specific project files. LuaBridge is easy to integrate.

C++ concepts like variables and classes are made available to Lua through a process called _registration_. Because Lua is weakly typed, the resulting structure is not rigid. The API is based on C++ template metaprogramming. It contains template code to automatically generate at compile-time the various Lua C API calls necessary to export your program's classes and functions to the Lua environment.

To expose Lua objects to C++, a class called `LuaRef` is provided. The implementation allows C++ code to access Lua objects such as numbers or strings, but more importantly to access things like tables and their values. Using this class makes idioms like calling Lua functions simple and clean.

1.1 - Design
------------

LuaBridge tries to be efficient as possible when creating the "glue" that exposes C++ data and functions to Lua. At the same time, the code was written with the intention that it is all as simple and clear as possible, without resorting to obscure C++ idioms, ugly preprocessor macros, or configuration settings. Furthermore, it is designed to be "header-only", making it very easy to integrate into your projects.

Because LuaBridge was written with simplicity in mind there are some features that are not available. Although it comes close to the highest possible performance, LuaBridge is not quite the fastest, [OOLua](http://code.google.com/p/oolua/) and [sol2](https://github.com/ThePhD/sol2) outperforms LuaBridge in some tests, but they are also bigger and slower to compile. While being powerful, LuaBridge is pretty compact and simpler to understand and debug, and also does not try to implement every possible feature: [LuaBind](http://www.rasterbar.com/products/luabind.html) (requires Boost) and [sol2](https://github.com/ThePhD/sol2) explores every corner of the C++ language.

LuaBridge does not support:

*   Global types (types must be registered in a named scope).
*   Automatic conversion between STL container types and Lua tables (but conversion can be enabled for `std::array`, `std::vector`, `std::map`, `std::unordered_map`, `std::set` `std::list`, `std::optional`, by including `LuaBridge/Array.h`, `LuaBridge/Vector.h`, `LuaBridge/Map`, `LuaBridge/UnorderedMap.h`, `LuaBridge/Set.h`, `LuaBridge/List.h`, `LuaBridge/Optional.h` respectively)
*   Inheriting Lua classes from C++ classes.
*   Passing nil to a C++ function that expects a pointer or reference.

1.2 - Repository
----------------

The official repository is located at [https://github.com/kunitoki/LuaBridge3](https://github.com/kunitoki/LuaBridge3).

The **master** branch contains published library versions. Release versions are marked with tags.

1.3 - License and Credits
-------------------------

LuaBridge3 is published under the terms of the [MIT License](http://www.opensource.org/licenses/mit-license.html):

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

The original version of LuaBridge was written by Nathan Reed. The project has been taken over by Vinnie Falco, who added new functionality and wrote the new documentation. Vinnie also incorporated `LuaRef` and other Lua to C++ binding contributions from Nigel Atkinson.

For questions, comments, or bug reports feel free to open a Github issue or contact Lucio Asnaghi directly at the email address indicated below.

*   Copyright 2020, Lucio Asnaghi [<kunitoki@gmail.com>](mailto:kunitoki@gmail.com)
*   Copyright 2019, Dmitry Tarakanov
*   Copyright 2012, Vinnie Falco [<vinnie.falco@gmail.com>](mailto:vinnie.falco@gmail.com)
*   Copyright 2008, Nigel Atkinson [<suprapilot+LuaCode@gmail.com>](mailto:suprapilot+LuaCode@gmail.com)
*   Copyright 2007, Nathan Reed

Older versions of LuaBridge up to and including 0.2 (available separately) are distributed under the BSD 3-Clause License. See the corresponding license file in those versions (distributed separately) for more details.

2 - Accessing C++ from Lua
==========================

In order to expose C++ data and functions to Lua, each piece of exported information must be _registered_. There are five types of objects that LuaBridge can register:

**Namespaces**  

A Lua table that contains other registrations.

**Data**  

Global or static variables, data members, and static data members.

**Functions**

Regular functions, member functions, and static member functions.

**CFunctions**

A regular function, member function, or static member function that uses the `lua_CFunction` calling convention.

**Properties**

Global properties, property members, and static property members. These appear like data to Lua, but are implemented in C++ using functions to get and set the values.

Both data and properties can be marked as _read-only_ at the time of registration. This is different from `const`; the values of these objects can be modified on the C++ side, but Lua scripts cannot change them. Code samples that follow are in C++ or Lua, depending on context. For brevity of exposition code samples in C++ assume the traditional variable `lua_State* L` is defined.

2.1 - Namespaces
----------------

All LuaBridge registrations take place in a _namespace_. When we refer to a _namespace_ we are always talking about a namespace in the Lua sense, which is implemented using tables. The namespace need not correspond to a C++ namespace; in fact no C++ namespaces need to exist at all unless you want them to. LuaBridge namespaces are visible only to Lua scripts; they are used as a logical grouping tool. To obtain access to the global namespace we write:

```cpp
luabridge::getGlobalNamespace (L);
```

This returns an object on which further registrations can be performed. The subsequent registrations will go into the global namespace, a practice which is not recommended. Instead, we can add our own namespace by writing:

```cpp
luabridge::getGlobalNamespace (L)
  .beginNamespace ("test");
```

This creates a table in `_G` called "test". Since we have not performed any registrations, this table will be empty except for some bookkeeping key/value pairs. LuaBridge reserves all identifiers that start with a double underscore. So `__test` would be an invalid name (although LuaBridge will silently accept it). Functions like `beginNamespace` return the corresponding object on which we can make more registrations. Given:

```cpp
luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginNamespace ("detail")
    .endNamespace ()
    .beginNamespace ("utility")
    .endNamespace ()
  .endNamespace ();
```

The results are accessible to Lua as `test`, `test.detail`, and `test.utility`. Here we introduce the `endNamespace` function; it returns an object representing the original enclosing namespace. All LuaBridge functions which create registrations return an object upon which subsequent registrations can be made, allowing for an unlimited number of registrations to be chained together using the dot operator. Adding two objects with the same name, in the same namespace, results in undefined behavior (although LuaBridge will silently accept it).

A namespace can be re-opened later to add more functions. This lets you split up the registration between different source files. These are equivalent:

```cpp
luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .addFunction ("foo", foo)
  .endNamespace ();

luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .addFunction ("bar", bar)
  .endNamespace ();
```

and

```cpp
luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .addFunction ("foo", foo)
    .addFunction ("bar", bar)
  .endNamespace ();
```

It's also possible to obtain a namespace from a table on the stack, and perform the registration of sub namespaces, properties, functions and classes into that table.

```cpp
lua_newtable (L);

luabridge::getNamespaceFromStack (L)
  .addFunction ("test", +[] (int x) { return x; })
  .addFunction ("bar", &bar);
```

The table is still on top of the stack here and has not been popped, so it's possible to further manipulate it or eventually use it as environment for closures.

2.2 - Properties and Functions
------------------------------

These are registered into a namespace using `addProperty` and `addFunction`. When registered functions are called by scripts, LuaBridge automatically takes care of the conversion of arguments into the appropriate data type when doing so is possible. This automated system works for the function's return value, and up to 8 parameters although more can be added by extending the templates. Pointers, references, and objects of class type as parameters are treated specially, and explained later.

If we have:

```cpp
int globalVar;
static float staticVar;

std::string stringProperty;
std::string getString () { return stringProperty; }
void setString (std::string s) { stringProperty = s; }

std::tuple <int, std::string> tuple;

int foo () { return 42; }
void bar (char const*) { }
int cFunc (lua_State* L) { return 0; }
```

These are registered with:

```cpp
luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .addProperty ("var1", &globalVar)
    .addProperty ("var2", &staticVar, false) // read-only
    .addProperty ("prop1", getString, setString)
    .addProperty ("prop2", getString)            // read only
    .addProperty ("tup", &tuple)
    .addFunction ("foo", foo)
    .addFunction ("bar", bar)
    .addFunction ("cfunc", cFunc)
  .endNamespace ();
```

Variables can be marked _read-only_ by passing `false` in the second optional parameter. If the parameter is omitted, _true_ is used making the variable read/write. Properties are marked read-only by omitting the set function. After the registrations above, the following Lua identifiers are valid:

```lua
test        -- a namespace
test.var1   -- a lua_Number property
test.var2   -- a read-only lua_Number property
test.prop1  -- a lua_String property
test.prop2  -- a read-only lua_String property
test.tup    -- a lua_Table property mapping to a c++ tuple
test.foo    -- a function returning a lua_Number
test.bar    -- a function taking a lua_String as a parameter
test.cfunc  -- a function with a variable argument list and multi-return
```

Note that `test.prop1` and `test.prop2` both refer to the same value. However, since `test.prop2` is read-only, assignment attempts will generate a run-time error. These Lua statements have the stated effects:

```lua
test.var1 = 5         -- okay
test.var2 = 6         -- error: var2 is not writable
test.prop1 = "Hello"  -- okay
test.prop1 = 68       -- okay, Lua converts the number to a string
test.prop2 = "bar"    -- error: prop2 is not writable
test.tup = { 1, "a" } -- okay, converts a table to tuple with the same size
test.tup = { "size" } -- error: table has different size than tuple

test.foo ()           -- calls foo and discards the return value
test.var1 = foo ()    -- calls foo and stores the result in var1
test.bar ("Employee") -- calls bar with a string
test.bar (test)       -- error: bar expects a string not a table
```

LuaBridge does not support overloaded functions nor is it likely to in the future. Since Lua is dynamically typed, any system that tries to resolve a set of parameters passed from a script will face considerable ambiguity when trying to choose an appropriately matching C++ function signature.

2.3 - Class Objects
-------------------

A class registration is opened using either `beginClass` or `deriveClass` and ended using `endClass`. Once registered, a class can later be re-opened for more registrations using `beginClass`. However, `deriveClass` should only be used once. To add more registrations to an already registered derived class, use `beginClass` on it.

These declarations:

```cpp
struct A {
  static int staticData;
  static float staticProperty;

  static float getStaticProperty () { return staticProperty; }
  static void setStaticProperty (float f) { staticProperty = f; }
  static void staticFunc () { }

  static int staticCFunc (lua_State *L) { return 0; }

  std::string dataMember;

  char dataProperty;
  char getProperty () const { return dataProperty; }
  void setProperty (char v) { dataProperty = v; }
  std::string toString () const { return dataMember; }

  void func1 () { }
  virtual void virtualFunc () { }

  int cfunc (lua_State* L) { return 0; }
};

struct B : public A {
  double dataMember2;

  void func1 () { }
  void func2 () { }
  void virtualFunc () { }
};

int A::staticData;
float A::staticProperty;
```

are registered using:

```cpp
luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass<A> ("A")
      .addStaticProperty ("staticData", &A::staticData)
      .addStaticProperty ("staticProperty", &A::getStaticProperty, &A::setStaticProperty)
      .addStaticFunction ("staticFunc", &A::staticFunc)
      .addStaticFunction ("staticCFunc", &A::staticCFunc)
      .addProperty ("data", &A::dataMember)
      .addProperty ("prop", &A::getProperty, &A::setProperty)
      .addFunction ("func1", &A::func1)
      .addFunction ("virtualFunc", &A::virtualFunc)
      .addFunction ("__tostring", &A::toString)     // Metamethod
      .addFunction ("cfunc", &A::cfunc)
    .endClass ()
    .deriveClass<B, A> ("B")
      .addProperty ("data", &B::dataMember2)
      .addFunction ("func1", &B::func1)
      .addFunction ("func2", &B::func2)
    .endClass ()
  .endNameSpace ();
```

Method registration works just like function registration. Virtual methods work normally; no special syntax is needed. const methods are detected and const-correctness is enforced, so if a function returns a const object (or a container holding to a const object) to Lua, that reference to the object will be considered const and only const methods can be called on it. It is possible to register Lua metamethods (except `__gc`). Destructors are registered automatically for each class.

As with regular variables and properties, class properties can be marked read-only by passing false in the second parameter, or omitting the set function. The `deriveClass` takes two template arguments: the class to be registered, and its base class. Inherited methods do not have to be re-declared and will function normally in Lua. If a class has a base class that is **not** registered with Lua, there is no need to declare it as a subclass.

Remember that in Lua, the colon operator '`:`' is used for method call syntax:

```lua
local a = A ()

a.func1 ()  -- error: func1 expects an object of a registered class
a.func1 (a) -- okay, verbose, this how OOP works in Lua
a:func1 ()  -- okay, less verbose, equivalent to the previous
```

2.4 - Property Member Proxies
-----------------------------

Sometimes when registering a class which comes from a third party library, the data is not exposed in a way that can be expressed as a pointer to member, there are no get or set functions, or the get and set functions do not have the right function signature. Since the class declaration is closed for changes, LuaBridge allows for a _property member proxy_. This is a pair of get and set flat functions which take as their first parameter a pointer to the object. This is easily understood with the following example:

```cpp
// Third party declaration, can't be changed
struct Vec
{
  float coord [3];
};
```

Taking the address of an array element, e.g. `&Vec::coord [0]` results in an error instead of a pointer-to-member. The class is closed for modifications, but we want to export Vec objects to Lua using the familiar object notation. To do this, first we add a "helper" class:

```cpp
struct VecHelper
{
  template <unsigned index>
  static float get (Vec const* vec)
  {
    return vec->coord [index];
  }

  template <unsigned index>
  static void set (Vec* vec, float value)
  {
    vec->coord [index] = value;
  }
};
```

This helper class is only used to provide property member proxies. `Vec` continues to be used in the C++ code as it was before. Now we can register the `Vec` class with property member proxies for `x`, `y`, and `z`:

```cpp
luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass<Vec> ("Vec")
      .addProperty ("x", &VecHelper::get<0>, &VecHelper::set<0>)
      .addProperty ("y", &VecHelper::get<1>, &VecHelper::set<1>)
      .addProperty ("z", &VecHelper::get<2>, &VecHelper::set<2>)
    .endClass ()
  .endNamespace ();
```

It is also possible to use both capturing and non capturing lambdas, as well as `std::function <>` instances as proxies:

```cpp
std::function<float (const Vec*)> get_x = [] (const Vec* vec) { return vec->coord [0]; };
std::function<void (Vec*, float)> set_x = [] (Vec* vec, float v) { vec->coord [0] = v; };

luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass<Vec> ("Vec")
      .addProperty ("x", get_x, set_x)
      // ... same for "y" and "z"
    .endClass ()
  .endNamespace ();
```

Or the more concise version (notice the `+` before the lambda is useful to convert a non capturing lambda to a function pointer in order to avoid allocating a std::function internally, where storing the lambda as function pointer might avoid lua usertype allocation overhead):

```cpp
luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass<Vec> ("Vec")
      .addProperty ("x",
        +[] (const Vec* vec) { return vec->coord [0]; },
        +[] (Vec* vec, float v) { vec->coord [0] = v; })
      // ... same for "y" and "z"
    .endClass ()
  .endNamespace ();
```

2.5 - Function Member Proxies
-----------------------------

Where it is not possible or inconvenient to add a member to be registered, LuaBridge also allows for a _function member proxy_. This is a flat function which take as its first parameter a pointer to the object:

```cpp
// Third party declaration, can't be changed
struct Vec
{
  float coord [3];
};
```

The class is closed for modifications, but we want to extend Vec objects with our member function. To do this, first we add a "helper" function:

```cpp
void scale (Vec* vec, float value)
{
  vec->coord [0] *= value;
  vec->coord [1] *= value;
  vec->coord [2] *= value;
}
```

Now we can register the `Vec` class with a member function `scale`:

```cpp
luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass<Vec> ("Vec")
      .addFunction ("scale", &scale)
    .endClass ()
  .endNamespace ();
```

It is also possible to use lambdas (both capturing and non capturing) as functions proxies:

```cpp
float y = atan (1.0f) * 4.0f;

luabridge::getGlobalNamespace (L)
  .beginClass<Vec> ("Vec")
    .addFunction ("scaleX", +[] (Vec* vec, float v) { vec->coord [0] *= v; })
    .addFunction ("scaleY", [y] (Vec* vec, float v) { vec->coord [1] *= v * y; })
  .endClass ()
```

Of course when not capturing, it is better to prefix the lambda with `+` so it is converted and stored internally to a function pointer instead of an `std::function`, so it is actually lighter to store and faster to call.

### 2.5.1 - Function Overloading

When specifying more than one method to the `addFunction` or `addStaticFunction` of both `Namespace` and `Class`, those overloads will be invoked in case of a call. Only overloads that have matched arguments arity will be considered, and they will be tried from first to last until the call succeeds.

```cpp
struct Vec { float coord [3]; };
struct Quat { float values [4]; };

void rotateByDegreees (Vec* vec, float degrees);
void rotateByQuaternion (Vec* vec, const Quat& quaternion);
```

Now we can register the `Vec` class with a member function `rotate` that will be resolving the call into the two provided overloads:

```cpp
luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass<Vec> ("Vec")
      .addFunction ("rotate", &rotateByDegreees, &rotateByQuaternion)
    .endClass ()
  .endNamespace ();
```

In case of members (or functions) with the same name, it's necessary to use `overload`, `constOverload` or `nonConstOverload` to disambiguate which of the functions needs to be registered:

```cpp
struct Quat { float values [4]; };

struct Vec
{
  void rotate (float degrees);
  void rotate (const Quat& quaternion);

  void x (float new_value);
  float x (float value_if_zero) const;

  float coord [3];
};

luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass<Vec> ("Vec")
      .addFunction ("rotate",
        luabridge::overload<float> (&Vec::rotate),
        luabridge::overload<const Quat&> (&Vec::rotate))
      .addFunction ("x",
        luabridge::nonConstOverload<float> (&Vec::x),
        luabridge::constOverload<float> (&Vec::x))
    .endClass ()
  .endNamespace ();

```

It's possible to mix lambdas, function pointers and member functions in overload creation. Providing a `lua_Cfunction` as last method will ensure it can be reached in case no other overload is successfully executed, kind of like a "catch all" method.

Special attention needs to be given to the order (priority) of the overloads, based on the number and type of the arguments. Better to place first the overloads that can be called more frequently, and putting "stronger" types first: for example when having an overload taking an `int` and an overload taking `float`, as lua is not able to distinguish between them properly (until lua 5.4) the first overload will always be called.

2.6 - Constructors
------------------

A single constructor may be added for a class using `addConstructor`. LuaBridge cannot automatically determine the number and types of constructor parameters like it can for functions and methods, so you must provide them. This is done by specifying the signature of the desired constructor functions as template parameters to `addConstructor`. The parameter types will be extracted from this (the return type is ignored). For example, these statements register constructors for the given classes:

```cpp
struct A
{
  A ();
  A (std::string_view a);
  A (std::string_view a, int b);
};

struct B
{
  explicit B (const char* s, int nChars);
};

luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass<A> ("A")
      .addConstructor<void (), void (std::string_view), void (std::string_view, int)> ()
    .endClass ()
    .beginClass<B> ("B")
      .addConstructor<void (const char*, int)> ()
    .endClass ()
  .endNamespace ();
```

Constructors added in this fashion are called from Lua using the fully qualified name of the class. This Lua code will create instances of `A` and `B`.

```lua
a = test.A ()           -- Create a new A.
b = test.B ("hello", 5) -- Create a new B.
b = test.B ()           -- Error: expected string in argument 1
```

### 2.6.1 - Constructor Proxies

Sometimes is not possible to use a constructor for a class, because some of the constructor arguments have types that couldn't be exposed to lua, or more control is needed when constructors need to be invoked (like checking the lau stack). So it is possible to workaround those limitations by using a special `addConstructor` that doesn't need any template specialiation, but takes only one or more functors, which will allow to placement new the c++ class in a c++ lambda, specifying any custom parameter there:

```cpp
struct NotExposed;
NotExposed* shouldNotSeeMe;

struct HardToCreate
{
  explicit HardToCreate (const NotExposed* x, int easy);
};

luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass<HardToCreate> ("HardToCreate")
      .addConstructor ([&shouldNotSeeMe] (void* ptr, int easy) { return new (ptr) HardToCreate (shouldNotSeeMe, easy); })
    .endClass ()
  .endNamespace ();
```

Then in lua:

```lua
hard = test.HardToCreate (5) -- Create a new HardToCreate.
```

The `addConstructor` overload taking a generic functor also accepts a `lua_State*` as last parameter in order to be used for constructors that needs to be overloaded by different numbers of arguments (arguments will start at index 2 of the stack):

```cpp
luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass<HardToCreate> ("HardToCreate")
      .addConstructor ([] (void* ptr, lua_State* L) { return new (ptr) HardToCreate (shouldNotSeeMe, lua_checkinteger (L, 2)); })
    .endClass ()
  .endNamespace ();
```

As mentioned at the beginning, it's possible to specify multiple functors, that will be tried in order until the object can be constructed:

```cpp
struct NotExposed;
NotExposed* shouldNotSeeMe;

struct HardToCreate
{
  HardToCreate (const NotExposed* x, int easy);
  HardToCreate (const NotExposed* x, int easy, int lessEasy);
};

luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass<HardToCreate> ("HardToCreate")
      .addConstructor (
        [&shouldNotSeeMe] (void* ptr, int easy) { return new (ptr) HardToCreate (shouldNotSeeMe, easy); },
        [&shouldNotSeeMe] (void* ptr, int easy, int lessEasy) { return new (ptr) HardToCreate (shouldNotSeeMe, easy, lessEasy); })
    .endClass ()
  .endNamespace ();
```

### 2.6.2 - Constructor Factories

If granular control over allocation and deallocation of a type is needed, the `addFactory` method can be used to register both and allocator and deallocator of C++ type. This is useful in case of having classes being provided by factory methods from shared libraries.

```cpp
struct IObject
{
  virtual void overridableMethod () const = 0;
};

// These might be defined in a shared library, returning concrete types.
extern "C" IObject* objectFactoryAllocator ();
extern "C" void objectFactoryDeallocator (IObject*);

luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass<IObject> ("Object")
      .addFactory (&objectFactoryAllocator, &objectFactoryDeallocator)
      .addFunction ("overridableMethod", &IObject::overridableMethod)
    .endClass ()
  .endNamespace ();
```

The object is the perfectly instantiable through lua:

```lua
a = test.Object ()           -- Create a new Object using objectFactoryAllocator
a = nil                      -- Remove any reference count
collectgarbage ("collect")   -- The object is garbage collected using objectFactoryDeallocator
```

2.7 - Extending Classes
-----------------------

The LuaBridge library provides a set of features dedicated to extending classes from lua.

### 2.7.1 - Extensible Classes

The `luabridge::extensibleClass` option tells LuaBridge to create a metatable for the class that can be modified at runtime by Lua code. By doing so, it's then possible to extend the class from lua by adding custom instance methods and static methods to the type. Because those methods are stored in the metatable of the type, no additional storage is needed.

```cpp
struct ExtensibleClass
{
  int propertyOne = 42;
};

luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass<ExtensibleClass> ("ExtensibleClass", luabridge::extensibleClass)
      .addProperty ("propertyOne", &ExtensibleClass::propertyOne)
    .endClass ()
  .endNamespace ();

ExtensibleClass clazz;
luabridge::pushGlobal (L, &clazz, "clazz");
```

From lua is then possible to extend the class by registering methods on the type:

```lua
function ExtensibleClass:newInstanceMethod()
  return self.propertyOne * 2
end

function ExtensibleClass.newStaticMethod()
  return 1337
end

print (clazz:newMethod(), ExtensibleClass.newStaticMethod())
```

This way extending an already existing method will raise a lua error when trying to do so. To be able to override existing methods, pass `luabridge::allowOverridingMethods` together with `luabridge::extensibleClass`.

```cpp
struct ExtensibleClass
{
  int existingMethod() { return 42; }
};

luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass<ExtensibleClass> ("ExtensibleClass", luabridge::extensibleClass | luabridge::allowOverridingMethods)
      .addFunction ("existingMethod", &ExtensibleClass::existingMethod)
    .endClass ()
  .endNamespace ();

ExtensibleClass clazz;
luabridge::pushGlobal (L, &clazz, "clazz");
```

```lua
function ExtensibleClass:existingMethod()
  return "this has been replaced"
end

print (clazz:existingMethod())
```




In case storing instance properties is needed, storage needs to be provided per instance. See the next chapter for an explanation on how to add custom properties per instance.

### 2.7.2 - Index and New Index Metamethods Fallback

In general LuaBridge for each class will add a `__index` and `__newindex` metamethods in order to be able to handle member function, properties and inheritance resolution. This will make it impossible for a user to override them because in doing so, we'll make the exposed classes non functioning. Although possible to override those metamethods directly, they will preclude any possibility to locate exposed members and properties in such classes.

If a LuaBridge exposed class still need to handle the case of handling `__index` and `__newindex` metamethods calls, it's possible to use the `addIndexMetaMethod` and `addNewIndexMetaMethod` registration functions that will be executed as fallback in case an already existing function/property is not exposed in the class itself or any of its parent.

```cpp
struct FlexibleClass
{
  int propertyOne = 42;
};

luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass<FlexibleClass> ("FlexibleClass")
      .addProperty ("propertyOne", &FlexibleClass::propertyOne)
      .addIndexMetaMethod ([] (FlexibleClass& self, const luabridge::LuaRef& key, lua_State* L)
      {
        if (key.tostring () == "existingProperty")
          return luabridge::LuaRef (L, 1337);

        return luabridge::LuaRef (L, luabridge::LuaNil ()); // or luaL_error("Failed lookup of key !")
      })
    .endClass ()
  .endNamespace ();

FlexibleClass flexi;
luabridge::pushGlobal (L, &flexi, "flexi");
```

Then in lua:

```lua
propertyOne = flexi.propertyOne
assert (propertyOne == 42, "Getting value from LuaBridge exposed property")

propertyTwo = flexi.existingProperty
assert (propertyTwo == 1337, "Getting value from non exposed LuaBridge property via __index fallback")

propertyThree = flexi.nonExistingProperty
assert (propertyThree == nil, "Getting value from non exposed LuaBridge property via __index fallback")
```

The same can be done for the `__newindex` metamethod fallback:

```cpp
struct FlexibleClass
{
  std::unordered_map<luabridge::LuaRef, luabridge::LuaRef> properties;
};

luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass<FlexibleClass> ("FlexibleClass")
      .addIndexMetaMethod ([] (FlexibleClass& self, const luabridge::LuaRef& key, lua_State* L)
      {
        auto it = self.properties.find(key);
        if (it != self.properties.end())
          return it->second;

        return luabridge::LuaRef (L, luabridge::LuaNil ()); // or luaL_error("Failed lookup of key !")
      })
      .addNewIndexMetaMethod ([] (FlexibleClass& self, const luabridge::LuaRef& key, const luabridge::LuaRef& value, lua_State* L)
      {
        self.properties.emplace (std::make_pair (key, value))
        return luabridge::LuaRef (L, luabridge::LuaNil ());
      })
    .endClass ()
  .endNamespace ();

FlexibleClass flexi;
luabridge::pushGlobal (L, &flexi, "flexi");
```

Then in lua:

```lua
propertyOne = flexi.propertyOne
assert (propertyOne == nil, "Value is not existing")

flexi.propertyOne = 1337

propertyOne = flexi.propertyOne
assert (propertyOne == 1337, "Value is now present !")
```

2.8 - Lua Stack
---------------

In the Lua C API, all operations on the `lua_State` are performed through the Lua stack. In order to pass values back and forth between C++ and Lua, LuaBridge uses specializations of this template class concept:

```cpp
namespace luabridge {

template <class T>
struct Stack
{
  static Result push (lua_State* L, const T& value);

  static TypeResult<T> get (lua_State* L, int index);

  static bool isInstance (lua_State* L, int index);
};

} // namespace luabridge
```

When a specialization of `Stack` exists for a given type `T` we say that the `T` is _convertible_. Throughout this document and the LuaBridge API, these types can be used anywhere a convertible type is expected.

The Stack template class specializations are used automatically for variables, properties, data members, property members, function arguments and return values. These basic types are supported:

* `bool`
* `char`, converted to a string of length one.
* `const char*` and `std::string` strings.
* `std::byte`, integers, `float`, `double`, `long double` converted to `Lua_number`.

User-defined types which are convertible to one of the basic types are possible, simply provide a `Stack <>` specialization in the `luabridge` namespace for your user-defined type, modeled after the existing types. For example, here is a specialization for a `juce::String`:

```cpp
namespace luabridge {

template <>
struct Stack<juce::String>
{
  static Result push (lua_State* L, const juce::String& s)
  {
    lua_pushstring (L, s.toUTF8 ());
    return {};
  }

  static TypeResult<juce::String> get (lua_State* L, int index)
  {
    if (lua_type (L, index) != LUA_TSTRING)
        return makeErrorCode (ErrorCode::InvalidTypeCast);

    std::size_t length = 0;
    const char* str = lua_tolstring (L, index, &length);
    if (str == nullptr)
        return makeErrorCode (ErrorCode::InvalidTypeCast);

    return juce::String::fromUTF8 (str);
  }

  static bool isInstance (lua_State* L, int index)
  {
    return lua_type (L, index) == LUA_TSTRING;
  }
};

} // namespace luabridge
```

To make sure the library can work without exceptions enabled, if for some reason the push and get of the value on/from the lua stack cannot be performed, it is mandatory to return a `Result` object that can be constructed from a `std::error_code`. It also good practice to resotre the stack to it's original state in case of failures:

```cpp
namespace luabridge {

template <class T>
struct Stack<Array<T>>
{
  static Result push (lua_State* L, const Array<T>& array)
  {
    const int initialStackSize = lua_gettop (L);

    lua_createtable (L, static_cast<int> (array.size ()), 0);

    for (std::size_t i = 0; i < array.size (); ++i)
    {
      lua_pushinteger (L, static_cast<lua_Integer> (i + 1));

      auto result = Stack<T>::push (L, array[i]);
      if (! result)
      {
        lua_pop (L, lua_gettop (L) - initialStackSize);  // Restore the stack
        return result;                                   // Forward the error code
      }

      lua_settable (L, -3);
    }

    return {};                                           // No error
  }

  static TypeResult<Array<T>> get (lua_State* L, int index)
  {
    if (!lua_istable (L, index))
      return makeErrorCode (ErrorCode::InvalidTypeCast);

    const int initialStackSize = lua_gettop (L);

    Array<T> a;
    a.reserve (static_cast<std::size_t> (get_length(L, index)));

    const int absIndex = lua_absindex (L, index);

    lua_pushnil (L);
    while (lua_next (L, absIndex) != 0)
    {
      auto item = Stack<T>::get (L, -1);
      if (! item)
      {
        lua_pop (L, lua_gettop (L) - initialStackSize);
        return item.error ();
      }

      a.append (*item);

      lua_pop (L, 1);
    }

    return a;
  }
};

} // namespace luabridge
```

### 2.8.1 - Enums

In order to expose C++ enums to lua and be able to work bidirectionally with them, it's necesary to create a Stack specialization for each exposed enum. As the process might become tedious, a library wrapper class is provided to simplify the steps.

```cpp
enum class MyEnum : int16_t
{
  A,
  B,
  C
};

template <>
struct luabridge::Stack<MyEnum> : luabridge::Enum<MyEnum>
{
};
```

This will map the enum to an integer as `int16_t` (the `underlying_type_t` of the enum) that will be converted to a `lua_Integer` in lua space. This has the drawback that any `lua_Integer` could be casted to a C++ enum. In order to provide a runtime check over the possible alternatives a `lua_Integer` could casted to, it's possible to specify the list of values the C++ enum has: the values registered into the `luabridge::Enum` will be checked against the passed integer and LuaBridge will raise an error in case the cast couldn't be made when using a `Stack<>::get` method:

```cpp
enum class MyEnum
{
  A = 1,
  B,
  C
};

template <>
struct luabridge::Stack<MyEnum> : luabridge::Enum<MyEnum,
                                                  MyEnum::A,
                                                  MyEnum::B,
                                                  MyEnum::C>
{
};

luabridge::push (L, 0); // Zero is NOT a valid enum value

auto result = luabridge::get<MyEnum> (L, 1);
assert (! result);
```

The preferred and easier way to expose enum values to lua, is by using a namespace and registering variables or properties for each value:

```cpp

// Variables are just values in lua, so for example doing `MyEnum1.A = 42` from lua will modified the value
luabridge::getGlobalNamespace (L)
  .beginNamespace ("MyEnum1")
    .addVariable ("A", MyEnum::A)
    .addVariable ("B", MyEnum::B)
    .addVariable ("C", MyEnum::C)
  .endNamespace();

// This ¡nstead will modification of the value from lua
luabridge::getGlobalNamespace (L)
  .beginNamespace ("MyEnum2")
    .addProperty ("A", +[] { return MyEnum::A; })
    .addProperty ("B", +[] { return MyEnum::B; })
    .addProperty ("C", +[] { return MyEnum::C; })
  .endNamespace();
```

### 2.8.2 - lua_State

Sometimes it is convenient from within a bound function or member function to gain access to the `lua_State*` normally available to a lua_CFunction. With LuaBridge, all you need to do is add a `lua_State*` as the last parameter of your bound function:

```cpp
void useState (lua_State* L);

luabridge::getGlobalNamespace (L).addFunction ("useState", &useState);
```

You can still include regular arguments while receiving the state:

```cpp
void useStateAndArgs (int i, std::string s, lua_State* L);

luabridge::getGlobalNamespace (L).addFunction ("useStateAndArgs", &useStateAndArgs);
```

When the script calls `useStateAndArgs`, it passes only the integer and string parameters. LuaBridge takes care of inserting the `lua_State*` into the argument list for the corresponding C++ function. This will work correctly even for the state created by coroutines. Undefined behavior results if the `lua_State*` is not the last parameter.

The same is applicable for properties.

3 - Passing Objects
===================

An object of a registered class `T` may be passed to Lua as:

**`T`**

Passed by value (a copy), with _Lua lifetime_.

**`const T`**

Passed by value (a copy), with _Lua lifetime_.

**`T*`**

Passed by reference, with _C++ lifetime_.

**`T&`**

Passed by reference, with _C++ lifetime_.

**`const T*`**

Passed by const reference, with _C++ lifetime_.

**`const T&`**

Passed by const reference, with _C++ lifetime_.

3.1 - C++ Lifetime
------------------

The creation and deletion of objects with _C++ lifetime_ is controlled by the C++ code. Lua does nothing when it garbage collects a reference to such an object. Specifically, the object's destructor is not called (since C++ owns it). Care must be taken to ensure that objects with C++ lifetime are not deleted while still being referenced by a `lua_State*`, or else undefined behavior results. In the previous examples, an instance of `A` can be passed to Lua with C++ lifetime, like this:

```cpp
A a;

luabridge::push (L, &a);              // pointer to 'a', C++ lifetime
lua_setglobal (L, "a");

luabridge::push (L, (const A*) &a);   // pointer to 'a const', C++ lifetime
lua_setglobal (L, "ac");

luabridge::push <const A*> (L, &a);   // equivalent to push (L, (A const*) &a)
lua_setglobal (L, "ac2");

luabridge::push (L, new A);           // compiles, but will leak memory
lua_setglobal (L, "ap");
```

3.2 - Lua Lifetime
------------------

When an object of a registered class is passed by value to Lua, it will have _Lua lifetime_. A copy of the passed object is constructed inside the userdata. When Lua has no more references to the object, it becomes eligible for garbage collection. When the userdata is collected, the destructor for the class will be called on the object. Care must be taken to ensure that objects with Lua lifetime are not accessed by C++ after they are garbage collected, or else undefined behavior results. An instance of `B` can be passed to Lua with Lua lifetime this way:

```cpp
B b;

luabridge::push (L, b);                    // Copy of b passed, Lua lifetime.
lua_setglobal (L, "b");
```

Given the previous code segments, these Lua statements are applicable:

```lua
print (test.A.staticData)       -- Prints the static data member.
print (test.A.staticProperty)   -- Prints the static property member.
test.A.staticFunc ()            -- Calls the static method.

print (a.data)                  -- Prints the data member.
print (a.prop)                  -- Prints the property member.
a:func1 ()                      -- Calls A::func1 ().
test.A.func1 (a)                -- Equivalent to a:func1 ().
test.A.func1 ("hello")          -- Error: "hello" is not a class A.
a:virtualFunc ()                -- Calls A::virtualFunc ().

print (b.data)                  -- Prints B::dataMember.
print (b.prop)                  -- Prints inherited property member.
b:func1 ()                      -- Calls B::func1 ().
b:func2 ()                      -- Calls B::func2 ().
test.B.func2 (a)                -- Error: a is not a class B.
test.A.func1 (b)                -- Calls A::func1 ().
b:virtualFunc ()                -- Calls B::virtualFunc ().
test.B.virtualFunc (b)          -- Calls B::virtualFunc ().
test.A.virtualFunc (b)          -- Calls B::virtualFunc ().
test.B.virtualFunc (a)          -- Error: a is not a class B.

a = nil; collectgarbage ()      -- 'a' still exists in C++.
b = nil; collectgarbage ()      -- Lua calls ~B() on the copy of b.
```

When Lua script creates an object of class type using a registered constructor, the resulting value will have Lua lifetime. After Lua no longer references the object, it becomes eligible for garbage collection. You can still pass these to C++, either by reference or by value. If passed by reference, the usual warnings apply about accessing the reference later, after it has been garbage collected.

3.3 - Pointers, References, and Pass by Value
---------------------------------------------

When C++ objects are passed from Lua back to C++ as arguments to functions, or set as data members, LuaBridge does its best to automate the conversion. Using the previous definitions, the following functions may be registered to Lua:

```cpp
void func0 (A a);
void func1 (A* a);
void func2 (A const* a);
void func3 (A& a);
void func4 (A const& a);
```

Executing this Lua code will have the prescribed effect:

```lua
func0 (a)   -- Passes a copy of a, using A's copy constructor.
func1 (a)   -- Passes a pointer to a.
func2 (a)   -- Passes a pointer to a const a.
func3 (a)   -- Passes a reference to a.
func4 (a)   -- Passes a reference to a const a.
```

In the example above, all functions can read the data members and property members of `a`, or call const member functions of `a`. Only `func0`, `func1`, and `func3` can modify the data members and data properties, or call non-const member functions of `a`.

The usual C++ inheritance and pointer assignment rules apply. Given:

```cpp
void func5 (B b);
void func6 (B* b);
```

These Lua statements hold:

```lua
func5 (b)   -- Passes a copy of b, using B's copy constructor.
func6 (b)   -- Passes a pointer to b.
func6 (a)   -- Error: Pointer to B expected.
func1 (b)   -- Okay, b is a subclass of a.
```

When a pointer or pointer to const is passed to Lua and the pointer is null (zero), LuaBridge will pass Lua a `nil` instead. When Lua passes a `nil` to C++ where a pointer is expected, a null (zero) is passed instead. Attempting to pass a null pointer to a C++ function expecting a reference results in `lua_error` being called.

3.4 - Shared Lifetime
---------------------

LuaBridge supports a _shared lifetime_ model: dynamically allocated and reference counted objects whose ownership is shared by both Lua and C++. The object remains in existence until there are no remaining C++ or Lua references, and Lua performs its usual garbage collection cycle. A container is recognized by a specialization of the `ContainerTraits` template class. LuaBridge will automatically recognize when a data type is a container when the corresponding specialization is present. Two styles of containers come with LuaBridge, including the necessary specializations.

### 3.4.1 - User-defined Containers

If you have your own container, you must provide a specialization of `ContainerTraits` in the `luabridge` namespace for your type before it will be recognized by LuaBridge (or else the code will not compile):

```cpp
namespace luabridge {

template <class T>
struct ContainerTraits<CustomContainer<T>>
{
  using Type = T;

  static CustomContainer<T> construct (T* c)
  {
    return c;
  }

  static T* get (const CustomContainer<T>& c)
  {
    return c.getPointerToObject ();
  }
};

} // namespace luabridge
```

Containers must be safely constructible from raw pointers to objects that are already referenced by other instances of the container (such as is the case for the provided containers or for example `boost::intrusive_ptr` but not `std::shared_ptr` or `boost::shared_ptr`).

### 3.4.2 - shared_ptr As Container

Standard containers like `std::shared_ptr` or `boost::shared_ptr` will work in LuaBridge3, but they require special care. This is because of type erasure; when the object goes from C++ to Lua and back to C++, constructing a new shared_ptr from the raw pointer will create another reference count and result in undefined behavior, unless it could intrusively reconstruct the container from a raw pointer.

To overcome this issue classes that should be managed by `shared_ptr` have to provide a way to correctly reconstruct a `shared_ptr` which can be done only if type hold it is deriving publicly from `std::enable_shared_from_this` or `boost::enable_shared_from_this`. No additional specialization of traits is needed in this case.

```cpp
struct A : public std::enable_shared_from_this<A>
{
  A () { }
  A (int) { }

  void foo () { }
};

luabridge::getGlobalNamespace (L)
  .beginClass<A> ("A")
    .addConstructorFrom<std::shared_ptr<A>, void(), void(int)> ()
    .addFunction ("foo", &A::foo)
  .endClass ();

std::shared_ptr<A> a = std::make_shared<A> (1);
luabridge::setGlobal (L, a, "a");

std::shared_ptr<A> retrieveA = luabridge::getGlobal<std::shared_ptr<A>> (L, "a");
retrieveA->foo ();
```

```lua
a.foo ()

anotherA = A ()
anotherA.foo ()

anotherA2 = A (1)
anotherA2.foo ()
```

### 3.4.3 - Container Constructors

When a constructor is registered for a class, there is an additional optional second template parameter describing the type of container to use. If this parameter is specified, calls to the constructor will create the object dynamically, via operator new, and place it a container of that type. The container must have been previously specialized in `ContainerTraits`, or else a compile error will result. This code will register two objects, each using a constructor that creates an object with Lua lifetime using the specified container:

```cpp
class C : public std::enable_shared_from_this<C>
{
  C () { }
  C (int) { }
};

luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass <C> ("C")
      .addConstructorFrom<std::shared_ptr<C>, void(), void(int)> ()
    .endClass ()
  .endNamespace ()
```

Alternatively is possible to pass custom lambdas to construct the container, where the return value of those lambdas must be exactly the container specified:

```cpp
class C : public std::enable_shared_from_this<C>
{
  C () { }
  C (int) { }
};

luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass <C> ("C")
      .addConstructorFrom<std::shared_ptr<C>> (
        []() { return std::make_shared<C> (); },
        [](int value) { return std::make_shared<C> (value); })
    .endClass ()
  .endNamespace ()
```

3.5 - Mixing Lifetimes
----------------------

Mixing object lifetime models is entirely possible, subject to the usual caveats of holding references to objects which could get deleted. For example, C++ can be called from Lua with a pointer to an object of class type; the function can modify the object or call non-const data members. These modifications are visible to Lua (since they both refer to the same object). An object store in a container can be passed to a function expecting a pointer. These conversion work seamlessly.

3.6 - Convenience Functions
---------------------------

The `setGlobal` function can be used to assign any convertible value into a global variable.

4 - Accessing Lua from C++
==========================

Because Lua is a _dynamically typed language_, special consideration is required to map values in Lua to C++. The following sections describe the classes and functions used for representing Lua types. Only the essential operations are explained; To gain understanding of all available functions, please refer to the documentation comments in the corresponding source files.

4.1 - Class LuaRef
------------------

The `LuaRef` class is a container which references any Lua type. It can hold anything which a Lua variable can hold: **nil**, number, boolean, string, table, function, thread, userdata, and lightuserdata. Because `LuaRef` uses the `Stack` template specializations to do its work, classes, functions, and data exported to Lua through namespace registrations can also be stored (these are instances of userdata). In general, a `LuaRef` can represent any _convertible_ C++ type as well as all Lua types.

A `LuaRef` variable constructed with no parameters produces a reference to **nil**:

```cpp
luabridge::LuaRef v (L); // References nil
```

To construct a `LuaRef` to a specific value, the two parameter constructor is used:

```cpp
luabridge::LuaRef v1 (L, 1);                   // A LUA_TNUMBER
luabridge::LuaRef v2 (L, 1.1);                 // Also a LUA_TNUMBER
luabridge::LuaRef v3 (L, true);                // A LUA_TBOOLEAN
luabridge::LuaRef v4 (L, "string");            // A LUA_TSTRING
```

The functions `newTable` and `getGlobal` create references to new empty table and an existing value in the global table respectively:

```cpp
LuaRef v1 = newTable (L);           // Create a new table
LuaRef v2 = getGlobal (L, "print")  // Reference to _G ["print"]
```

A `LuaRef` can hold classes _registered_ using LuaBridge:

```cpp
class A;

//...

luabridge::LuaRef v (L, new A); // A LuaBridge userdata holding a pointer to A
```

Any convertible type may be assigned to an already-existing `LuaRef`:

```cpp
luabridge::LuaRef v (L);        // Nil
v = luabridge::newTable (L);    // An empty table
v = "string";                   // A string. The previous value becomes
                                // eligible for garbage collection.
```

A `LuaRef` is itself a convertible type, and the convertible type `LuaNil` can be used to represent a Lua **nil**.

```cpp
luabridge::LuaRef v1 (L, "x");  // assign "x"
luabridge::LuaRef v2 (L, "y");  // assign "y"
v2 = v1;                        // v2 becomes "x"
v1 = "z";                       // v1 becomes "z", v2 is unchanged
v1 = luabridge::newTable (L);   // An empty table
v2 = v1;                        // v2 references the same table as v1
v1 = luabridge::LuaNil ();      // v1 becomes nil, table is still
                                // referenced by v2.
```

Values stored in a `LuaRef` object obey the same rules as variables in Lua: tables, functions, threads, and full userdata values are _objects_. The `LuaRef` does not actually _contain_ these values, only _references_ to them. Assignment, parameter passing, and function returns always manipulate references to such values; these operations do not imply any kind of copy.

### 4.1.1 - Lifetime, States and Lua Threads

Lifetime of `LuaRef` is bound to the lua state or thread passed in when constructing the reference. It is responsibility of the developer to keep the passed lua state/thread alive for the duration of the usage of the `LuaRef`. In case of storing objects in those references that might be created in lua threads that could be destroyed during the application lifetime, it is advised to pass `luabridge::main_thread (L)` in place of `L` when constructing a `LuaRef`, to make sure the reference is kept in the main lua state instead of the volatile lua thread where it has been created.

In order to have `luabridge::main_thread` method working in all lua versions, one have to call `luabridge::registerMainThread` function at the beginning of the usage of luabridge (lua 5.1 doesn't store the main thread in the registry, and this needs to be manually setup by the developer).

### 4.1.2 - Type Conversions

A universal C++ conversion operator is provided for implicit conversions which allow a `LuaRef` to be used where any convertible type is expected. These operations will all compile:

```cpp
void passInt (int);
void passBool (bool);
void passString (std::string);
void passObject (A*);

luabridge::LuaRef v (L);
//...
passInt (v);        // implicit conversion to int
passBool (v);       // implicit conversion to bool
passString (v);     // implicit conversion to string
passObject (v);     // must hold a registered LuaBridge class or a
                    // lua_error() will be called.
```

Since Lua types are dynamic, the conversion is performed at run time using traditional functions like `lua_toboolean` or `lua_tostring`. In some cases, the type information may be incorrect especially when passing objects of registered class types. When performing these conversions, LuaBridge may raise a Lua error by directly or indirectly calling `lua_error` To be bullet-proof, such code must either be wrapped in a `lua_pcall`, or you must install a Lua _panic function_ that throws an exception which you can catch.

When an explicit conversion is required (such as when writing templates), use the `cast` template function or an explicit C++ style cast.

```cpp
void passString (std::string);

luabridge::LuaRef v (L);

// The following are all equivalent, and they could be potentially unsafe:

passString (std::string (v));
passString ((std::string) v);
passString (static_cast<std::string> (v));
passString (v.unsafe_cast<std::string> ());
```

The only way to ensure safety when type casting is to use the `LuaRef::cast<T>` method, which is a safe cast of a lua reference to a type `T`. It will return a `luabridge::TypeResult<T>` which will contain the type if the cast was successful, and an error code otherwise. No exception or abort will be triggered from such call (while it's not the same for `LuaRef::cast<T>`).

```cpp
void passString (std::string);

luabridge::LuaRef v (L);

// The following is safe and will never throw exceptions or call the lua panic handler.

passString (v.cast<std::string> ().valueOr ("fallback"));
```

4.2 - Table Proxies
-------------------

As tables are the sole data structuring mechanism in Lua, the `LuaRef` class provides robust facilities for accessing and manipulating table elements using a simple, precise syntax. Any convertible type may be used as a key or value. Applying the array indexing operator `[]` to a `LuaRef` returns a special temporary object called a _table proxy_ which supports all the operations which can be performed on a `LuaRef`. In addition, assignments made to table proxies change the underlying table. Because table proxies are compiler-created temporary objects, you don't work with them directly. A LuaBridge table proxy should not be confused with the Lua proxy table technique described in the book "Programming in Lua"; the LuaBridge table proxy is simply an intermediate C++ class object that works behind the scenes to make table manipulation syntax conform to C++ idioms. These operations all invoke table proxies:

```cpp
luabridge::LuaRef v (L);
v = luabridge::newTable (L);

v ["name"] = "John Doe";         // string key, string value
v [1] = 200;                     // integer key, integer value
v [2] = luabridge::newTable (L); // integer key, LuaRef value
v [3] = v [1];                   // assign 200 to integer index 3
v [1] = 100;                     // v[1] is 100, v[3] is still 200
v [3] = v [2];                   // v[2] and v[3] reference the same table
v [2] = luabridge::LuaNil ();    // Removes the value with key = 2. The table
                                 //   is still referenced by v[3].
```

4.3 - Calling Lua
-----------------

Table proxies and `LuaRef` objects provide a convenient syntax for invoking `lua_pcall` on suitable referenced object. This includes C functions, Lua functions, or Lua objects with an appropriate `__call` metamethod set. The provided implementation supports up to eight parameters (although more can be supported by adding new functions). Any convertible C++ type can be passed as a parameter in its native format. The return value of the function call is provided as a `LuaRef`, which may be **nil**.

```lua
function same (arg1, arg)
  return arg1 == arg2
end
```

```cpp
luabridge::LuaRef same = luabridge::getGlobal (L, "same");

// These all evaluate to true
same (1,1);
!same (1,2);
same ("text", "text");
!same (1, "text");
same (1, 1, 2); // third param ignored
```

Table proxies support all of the Lua call notation that `LuaRef` supports, making these statements possible:

```lua
t[1]();
t[2]("a", "b");
t[2](t[1]); // Call t[3] with the value in t[2]
t[4]=t[3]();   // Call t[3] and store the result in t[4].

t [t[5]()] = "wow"; // Store "wow" at the key returned by
                    //   the call to t[5]

t = {}
t[1] = function () print ("hello") end
t[2] = function (u, v) print (u, v) end
t[3] = "foo"
```

```cpp
luabridge::LuaRef v = luabridge::getGlobal (L, "t");
```

### 4.3.1 - Exceptions

By default `LuaBridge3` is able to work without exceptions, and it's perfectly compatible with the `-fno-exceptions` or `/EHsc-` flags, which is typically used in games. Even if compiling with exceptions enabled, they are not used internally when calling into lua to convert lua errors, but exceptions are only used in registration code to signal potential issues when registering namespaces, classes and methods. You can use the free function `luabridge::enableExceptions` to enable exceptions once before starting to use any luabridge call, and of course that will work only if the application is compiled with exceptions enabled.

When using the `luabridge::call` or `LuaRef::operator()` no exception should be raised, only if exceptions are disabled in the application or enabled in the application but disabled in luabridge. To control if the lua function invoked has raised a lua error, it is possible to do so by checking the `LuaResult` object that is returned from those functions.

```lua
function fail ()
  error ("A problem occurred")
end
```

```cpp
luabridge::LuaRef f (L) = luabridge::getGlobal (L, "fail");

luabridge::LuaResult result = f ();
if (! result)
  std::cerr << result.errorMessage ();
```

It is also possible that pushing an unregistered class instance into those function will generate an error, that can be trapped using the same mechanism in a `LuaResult`:

```lua
function fail (unregistred)
  error ("Should never reach here")
end
```

```cpp
struct UnregisteredClass {};

luabridge::LuaRef f (L) = luabridge::getGlobal (L, "fail");

auto argument = UnregisteredClass();

luabridge::LuaResult result = f (argument);
if (! result)
  std::cerr << result.errorMessage ();
```

Calling `luabridge::pcall` will not return a `LuaResult` but only the status code. It will anyway throw an exception if the return code of `lua_pcall`is not equal `LUA_OK`, and return the error code in case exceptions are disabled.

When compiling `LuaBridge3` with exceptions disabled, all references to try catch blocks and throws will be removed.

### 4.3.2 - Class LuaException

When the application is compiled with exceptions and `luabridge::enableExceptions` function has been called, using `luabridge::call` or `LuaRef::operator()` will uses the C++ exception handling mechanism, throwing a `LuaException` object in case an argument has a type that has not been registered (and cannot be pushed onto the lua stack) or the lua function generated an error:

```lua
function fail ()
  error ("A problem occurred")
end
```

```cpp
luabridge::LuaRef f (L) = luabridge::getGlobal (L, "fail");

try
{
  f ();
}
catch (const luabridge::LuaException& e)
{
  std::cerr << e.what ();
}
```

5 - Security
============

The metatables and userdata that LuaBridge creates in the `lua_State*` are protected using a security system, to eliminate the possibility of undefined behavior resulting from scripted manipulation of the environment. The security system has these components:

*   Class and const class tables use the _table proxy_ technique. The corresponding metatables have `__index` and `__newindex` metamethods, so these class tables are immutable from Lua.
*   Metatables have `__metatable` set to a boolean false value. Scripts cannot obtain the metatable from a LuaBridge object.
*   Classes are mapped to metatables through the registry, which Lua scripts cannot access. The global environment does not expose metatables
*   Metatables created by LuaBridge are tagged with a lightuserdata key which is unique in the process. Other libraries cannot forge a LuaBridge metatable.

This security system can be easily bypassed if scripts are given access to the debug library (or functionality similar to it, i.e. a raw `getmetatable`). The security system can also be defeated by C code in the host, either by revealing the unique lightuserdata key to another module or by putting a LuaBridge metatable in a place that can be accessed by scripts.

When a class member function is called, or class property member accessed, the `this` pointer is type-checked. This is because member functions exposed to Lua are just plain functions that usually get called with the Lua colon notation, which passes the object in question as the first parameter. Lua's dynamic typing makes this type-checking mandatory to prevent undefined behavior resulting from improper use.

If a type check error occurs, LuaBridge uses the `lua_error` mechanism to trigger a failure. A host program can always recover from an error through the use of `lua_pcall`; proper usage of LuaBridge will never result in undefined behavior.

However, in certain situations, it may be necessary for the library user to access the metatables of LuaBridge objects. To allow for this, LuaBridge provides the option to expose metatables on a class-by-class basis when scripts are coming from a trusted source. This can be achieved by passing `luabridge::visibleMetatables` option at class registration:

```cpp
luabridge::getGlobalNamespace (L)
  .beginNamespace ("test")
    .beginClass <C> ("C", luabridge::visibleMetatables)
      .addConstructor<void ()> ()
    .endClass ()
  .endNamespace ()
```

Metatables are also special in registered LuaBridge's namespaces, as they namespace specific properties and class definitions, and usually their access should be prevented. But it's possible to expose the much like for classes:

```cpp

luabridge::getGlobalNamespace (L)
  .beginNamespace ("test", luabridge::visibleMetatables)
    .beginClass <C> ("C")
      .addConstructor<void ()> ()
    .endClass ()
  .endNamespace ()
```

Appendix - API Reference
========================

Global Options
--------------

```cpp
/// Flag set of options
class Options : FlagSet<uint32_t>;

/// Default options for classes / namespaces registration.
Option defaultOptions;

/// Specify that class methods should allow to the extended by lua scripts.
Option extensibleClass;

/// Allow an extensible class to overriding C++ exposed methods.
Option allowOverridingMethods;

/// Allow access to class / namespace metatables.
Option visibleMetatables;
```

Free Functions
--------------

```cpp
/// Enable exceptions globally. Will translate lua_errors into C++ LuaExceptions. Usable only if compiled with C++ exceptions enabled.
void enableExceptions (lua_State* L);

/// Gets a global Lua variable reference as LuaRef.
LuaRef getGlobal (lua_State* L, const char* name);

/// Gets a global Lua variable reference as type T.
template <class T>
TypeResult<T> getGlobal (lua_State* L, const char* name);

/// Sets a global Lua variable. Throws or return false if the class is not registered.
template <class T>
bool setGlobal (lua_State* L, T* varPtr, const char* name);

/// Gets the global namespace registration object.
Namespace getGlobalNamespace (lua_State* L);

/// Gets a namespace registration object using a table on top of the stack.
Namespace getNamespaceFromStack (lua_State* L);

/// Invokes a LuaRef if it references a lua callable.
template <class... Args>
LuaResult call (const LuaRef& object, Args&&... args)

/// Wrapper for lua_pcall, converting lua errors into C++ exceptions if they are enabled.
int pcall (lua_State* L, int nargs = 0, int nresults = 0, int msgh = 0)

/// Return a range iterable view over a lua table.
Range pairs (const LuaRef& table);
```

Namespace Registration - Namespace
----------------------------------

```cpp
/// Begin or continues namespace registration, returns this namespace object.
template <class T>
Namespace beginNamespace (const char* name);

/// Ends namespace registration, returns the parent namespace object.
template <class T>
Namespace endNamespace ();

/// Registers one or multiple overloaded functions.
template <class... Functions>
Namespace addFunction (const char* name, Functions... functions);

/// Registers a property with a getter and setter.
template <class V>
Namespace addProperty (const char* name, V (*getFn)(), void (*setFn)(V));

/// Registers a property with a getter and setter.
template <class V>
Namespace addProperty (const char* name, std::function<V ()> getFn, std::function<void (V)> setFn);

/// Registers a property with a C-function getter and setter.
Namespace addProperty (const char* name, int (*getFn)(lua_State*), int (*setFn)(lua_State*));

/// Registers a read-only property with a getter function.
template <class V>
Namespace addProperty (const char* name, V (*getFn)());

/// Registers a read-only property with a getter function.
template <class V>
Namespace addProperty (const char* name, std::function<V ()> getFn);

/// Registers a read-only property with a C-function getter.
Namespace addProperty (const char* name, int (*getFn)(lua_State*));

/// Registers a variable, writable or read-only.
template <class V>
Namespace addProperty (const char* name, V* varPtr, bool isWritable = true);
```

Class Registration - Class<T>
-----------------------------

```cpp
/// Begins or continues class registration, returns this class object.
template <class T>
Class<T> beginClass (const char* name);

/// Begins derived class registration, returns this class object.
template <class T, class Base>
Class<T> deriveClass (const char* name);

/// Ends class registration, returns the parent namespace object.
template <class T>
Namespace endClass ();
```

### Constructor Registration

```cpp
/// Registers one or multiple overloaded constructors for type T.
template <class... Functions>
Class<T> addConstructor ();

/// Registers one or multiple overloaded constructors for type T using callable arguments.
template <class... Functions>
Class<T> addConstructor (Functions... functions);

/// Registers one or multiple overloaded constructors for type T when usable from intrusive container C.
template <class C, class... Functions>
Class<T> addConstructorFrom ();

/// Registers one or multiple overloaded constructors for type T when usable from intrusive container C using callable arguments.
template <class C, class... Functions>
Class<T> addConstructorFrom (Functions... functions);

/// Registers allocator and deallocators for type T.
template <class Alloc, class Dealloc>
Class<T> addFactory (Alloc alloc, Dealloc dealloc);
```

### Member Function Registration

```cpp
/// Registers one or multiple overloaded functions as member functions.
template <class... Functions>
Class<T> addFunction (const char* name, Functions... functions);
```

### Member Property Registration

```cpp
/// Registers a property with a getter and setter.
template <class V>
Class<T> addProperty (const char* name, V (T::* getFn)(), void (T::* setFn)(V));

/// Registers a property with a getter and setter.
template <class V>
Class<T> addProperty (const char* name, std::function<V ()> getFn, std::function<void (V)> setFn);

/// Registers a property with a C-function getter and setter.
Class<T> addProperty (const char* name, int (*getFn)(lua_State*), int (*setFn)(lua_State*));

/// Registers a read-only property with a getter member function.
template <class V>
Class<T> addProperty (const char* name, V (T::* getFn)());

/// Registers a read-only property with a getter function.
template <class V>
Class<T> addProperty(const char* name, std::function<V ()> getFn);

/// Registers a read-only property with a C-function getter.
Class<T> addProperty (const char* name, int (*getFn)(lua_State*));

/// Registers a member variable, writable or read-only.
template <class V>
Class<T> addProperty (const char* name, V T::* varPtr, bool isWritable = true);
```

### Static Function Registration

```cpp
/// Registers one function or multiple overloads.
template <class... Functions>
Class<T> addStaticFunction (const char* name, Functions... functions);
```

### Static Property Registration

```cpp
/// Registers a property with a getter and setter.
template <class V>
Class<T> addStaticProperty (const char* name, V (*getFn)(), void (*setFn)(V));

/// Registers a property with a getter and setter.
template <class V>
Class<T> addStaticProperty (const char* name, std::function<V ()> getFn, std::function<void (V)> setFn);

/// Registers a property with a C-function getter and setter.
Class<T> addStaticProperty (const char* name, int (*getFn)(lua_State*), int (*setFn)(lua_State*));

/// Registers a read-only property with a getter function.
template <class V>
Class<T> addStaticProperty (const char* name, V (*getFn)());

/// Registers a read-only property with a getter function.
template <class V>
Class<T> addStaticProperty (const char* name, std::function <V()> getFn);

/// Registers a read-only property with a C-function getter.
Class<T> addStaticProperty (const char* name, int (*getFn)(lua_State*));

/// Registers a variable, writable or read-only.
Class<T> addStaticProperty (const char* name, T* varPtr, bool isWritable = true);
```

Lua Variable Reference - LuaRef
-------------------------------

```cpp
/// Creates a nil reference.
LuaRef (lua_State* L);

/// Returns native Lua string representation.
std::string tostring () const;

/// Dumps reference to a stream.
void print (std::ostream& stream) const;

/// Returns the Lua state.
lua_State* state () const;

/// Place the object onto the Lua stack.
void push (lua_State* L);

/// Indicate whether it is a valid reference (not a LUA_NOREF).
bool isValid () const;

/// Return the lua_type.
int type () const;

/// Indicate whether it is a nil reference.
bool isNil () const;

/// Indicate whether it is a reference to a boolean.
bool isBool () const;

/// Indicate whether it is a reference to a number.
bool isNumber () const;

/// Indicate whether it is a reference to a string.
bool isString () const;

/// Indicate whether it is a reference to a table.
bool isTable () const;

/// Indicate whether it is a reference to a function.
bool isFunction () const;

/// Indicate whether it is a reference to a full userdata.
bool isUserdata () const;

/// Indicate whether it is a reference to a light userdata.
bool isLightUserdata () const;

/// Indicate whether it is a reference to a Lua thread.
bool isThread () const;

/// Indicate whether it is a callable, can be either a lua function or an object with the __call metamethod.
bool isCallable () const;

/// Perform implicit type conversion.
template <class T>
operator T () const;

/// Perform the explicit type conversion, safe.
template <class T>
TypeResult<T> cast () const;

/// Perform the explicit type conversion, unsafe (throws or abort on failure).
template <class T>
T unsafe_cast () const;

/// Check if the Lua value is convertible to the type T.
template <class T>
bool isInstance () const;

/// Get the metatable for the LuaRef.
LuaRef getMetatable () const;

/// Compare this reference with a specified value using lua_compare(). This invokes metamethods.
template <class T>
bool operator== (T rhs) const;

/// Compare this reference with a specified value using lua_compare(). This invokes metamethods.
template <class T>
bool operator!= (T rhs) const;

/// Compare this reference with a specified value using lua_compare(). This invokes metamethods.
template <class T>
bool operator< (T rhs) const;

/// Compare this reference with a specified value using lua_compare(). This invokes metamethods.
template <class T>
bool operator<= (T rhs) const;

/// Compare this reference with a specified value using lua_compare(). This invokes metamethods.
template <class T>
bool operator> (T rhs) const;

/// Compare this reference with a specified value using lua_compare(). This invokes metamethods.
template <class T>
bool operator>= (T rhs) const;

/// Compare this reference with a specified value using lua_compare(). This does not invoke metamethods.
template <class T>
bool rawequal (T v) const;

/// Append a value to a referred table. If the table is a sequence this will add another element to it.
template <class T>
void append (T v) const;

/// Return the length of a referred array. This is identical to applying the Lua # operator.
int length () const;

/// Invoke the lua ref if it references a lua function.
template <class... Args>
LuaResult call (Args&&... args) const;
```

Lua Nil Special Value - LuaNil
------------------------------

```cpp
/// LuaNil can be used to construct LuaRef.
```

Lua Result Of Function Invocation - LuaResult
---------------------------------------------

```cpp
explicit operator bool() const;

/// Return if the invocation was ok and didn't raise a lua error.
bool wasOk() const;

/// Return if the invocation did raise a lua error.
bool hasFailed() const;

/// Return the error code, if any.
std::error_code errorCode() const;

/// Return the error message, if any.
std::string errorMessage() const;

/// Return the number of return values.
std::size_t size() const;

/// Get a return value at a specific index.
LuaRef operator[](std::size_t index) const;

```

Stack Traits - Stack<T>
-----------------------

```cpp
/// Converts the C++ value into the Lua value at the top of the Lua stack. Returns true if the push could be performed.
/// When false is returned, `ec` contains the error code corresponding to the failure.
Result push (lua_State* L, const T& value);

/// Converts the Lua value at the index into the C++ value of the type T.
TypeResult<T> get (lua_State* L, int index);

/// Checks if the Lua value at the index is convertible into the C++ value of the type T.
bool isInstance (lua_State* L, int index);
```
