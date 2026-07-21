# C++ Coding Standard

> A reference document describing C++ coding standards, principles, style/format rules, safety rules, and performance rules. Structured for machine and human readers.

## Table of Contents

1. [Introduction](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#1-introduction)
2. [C++ Coding Principles](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#2-c-coding-principles)
    - [RAII](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#21-resource-acquisition-is-initialization-raii)
    - [KISS](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#22-keep-it-simple-stupid-kiss)
    - [POLA](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#23-principle-of-least-astonishment-pola)
3. [Style and Format](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#3-style-and-format)
    - [Layout](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#31-layout)
    - [Naming Conventions](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#32-naming-conventions)
    - [Comments and Documentation](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#33-comments-and-documentation)
4. [Coding Rules For Safety](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#4-coding-rules-for-safety)
    - [Legacy Code](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#41-legacy-code)
    - [Compilation](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#42-compilation)
    - [Global Variables](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#43-global-variables)
    - [Memory Management and Access](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#44-memory-management-and-access)
    - [Type Safety](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#45-type-safety)
    - [Classes, Inheritance and Exposure](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#46-classes-inheritance-and-exposure)
    - [Error Handling / Logging](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#47-error-handling--logging)
    - [Control Flow Statements & Expressions](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#48-control-flow-statements--expressions)
    - [Threading](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#49-threading-safety)
5. [Coding Rules for Performance](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#5-coding-rules-for-performance)
    - [Memory Allocations](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#51-memory-allocations)
    - [System Calls](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#52-system-calls)
    - [Function Calls](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#53-function-calls)
    - [Threading](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#54-threading-performance)
    - [RTTI](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#55-rtti)
    - [Control Flow and Arguments](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#56-control-flow-and-arguments)
    - [Algorithms and Data-Types](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#57-algorithms-and-data-types)
6. [Reading Material](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#6-reading-material)
7. [Appendix](https://claude.ai/chat/84e7e082-1be7-48a0-a227-b01630714e69#7-appendix)

---

## 1. Introduction

This document describes the C++ Coding Standard. Its objective is to enhance code quality, reduce the number of bugs, and ensure that we all code in a similar way. This eases code maintenance and reviews and makes it easier for newcomers to understand existing code and to start programming.

Note that this Coding Standard is **not** intended as material to learn C++. Developers using this Coding Standard are expected to be either an expert in C++ or on the path to becoming one.

For continuous education, this document also provides a list of references to reading material on C++ in Section 6. These are useful to beginners and experts alike and should be consulted regularly.

---

## 2. C++ Coding Principles

### 2.1 Resource Acquisition Is Initialization (RAII)

RAII is a central concept in C++ where resources are acquired (for example, memory or file descriptors) and encapsulated by an object with a limited scope. As the object is deleted at the end of scope, resources are automatically released, leading to higher safety and simpler resource management.

> "One of the fundamental principles of programming is that if we acquire a resource, we must — somehow, directly or indirectly — return it to whatever part of the system manages that resource."

**Result we want:** Improved stability and strong guarantees about when resources are released.

### 2.2 Keep It Simple Stupid (KISS)

KISS is a design principle that helps us avoid unnecessary complexity.

In C++ terms, we apply KISS by limiting the use of constructs that either make it hard or impossible to reason about the correctness of the program, or that have been superseded by better-suited constructs.

**Result we want:** Code should be understandable and therefore maintainable by other programmers.

### 2.3 Principle Of Least Astonishment (POLA)

POLA is a design principle that aims to exploit pre-existing knowledge of users (and programmers) to minimize their learning curve. At the same time, it guides programmers away from creating interfaces with surprising side-effects.

POLA tells us that when coding, one should aspire to create interfaces that do not surprise their users. For example, in object-oriented programming this can mean avoiding methods that mutate the object state even though their naming indicates that they do not. A getter that actually changes the state of the property being fetched does not follow POLA, since another programmer would not expect a getter to change the state of the object.

**Result we want:** It should be possible to reason about interfaces without reading their implementation.

---

## 3. Style and Format

This section introduces style and formatting rules. These rules do not restrict which C++ language constructs can be used, but how the program code is written down. A rationale is provided only where applicable to the particular rule.

### 3.1 Layout

#### New classes are created using the company-wide templates

**Header file template:**

```cpp
/*****************************************************************************
 * Copyright (c) 1998-${year}, [] Inc.
 * All Rights Reserved.
 *****************************************************************************/
#ifndef namespace1_namespace2_Class_h
#define namespace1_namespace2_Class_h

/*
 * Local header files
 */
[Local header files included in alphabetical order]

/*
 * System header files
 */
[System header files included in alphabetical order]

/*
 * Forward declarations
 */
[Forward declaration in alphabetical order]

namespace [namespace1]::[namespace2]
{

/** [Brief summary of class for Doxygen documentation which ends at the first dot.]
 * [Detailed description of class which could include many lines, code examples, usage etc.]
 */
class [Class]
{
public:
    /** [Brief summary of constructor for Doxygen documentation which ends at the first dot.]
    * [Detailed description]
    *
    * @param [param] Description of param
    */
    [Class]([param])

    /** [Brief summary of member function for Doxygen documentation which ends at the first dot.]
    * [Detailed description]
    *
    * @param [param] Description of param
    * @return Description of output
    */
    [Member]
};

} // namespace [namespace1]::[namespace2]

#endif // namespace1_namespace2_Class_h
```

**Source file template:**

```cpp
/*****************************************************************************
 * Copyright (c) 1998-${year}, [] Inc.
 * All Rights Reserved.
 *****************************************************************************/
[header file for this class]

/*
 * Local header files
 */
[header files in this project in alphabetical order]

/*
 * System header files
 */
[header files from /usr/include in alphabetical order]

namespace [namespace1]::[namespace2]
{

/*
 * Constructors & Destructor
 */

/*
 * Public members
 */

/*
 * Protected members
 */

/*
 * Private members
 */

} // namespace [namespace1]::[namespace2]
```

#### The public, protected and private sections are to be declared in that order

**Rule:** In each file, the `public`, `protected`, and `private` sections of a class are to be declared in that order.

**Rationale:** To keep public members with public methods, protected members with protected methods, and private members with private methods. The public section is at the top of the file as these members and methods are of interest to external users of the class.

#### Within a section, methods are to be declared before data members

**Rule:** Within the public, protected, and private sections of a file, methods are to be declared before data members.

#### Pointer or reference declarations are associated with the variable

**Rule:** When declaring or defining pointer or reference instances, the pointer or reference symbol shall be associated with the variable, for example, `int *p` or `int &q`, instead of `int* p` or `int& q`.

**Rationale:** This is unfortunately how the language sees things.

#### Do not use spaces around `.` or `->`

**Rule:** Do not use spaces around `.` or `->`.

#### Use 4 spaces for every level of indentation

**Rule:** Use 4 spaces for every level of indentation. Do not use TABs.

#### Code blocks start on a new line

**Rule:** Code blocks start on a new line.

#### Surround code blocks with `{}`

**Rule:** Always surround code blocks following `if`, `else`, `switch-case`, `for`, `while`, or `do` statements with `{}` — even if there is only one line of code.

**Rationale:** Adding new code or comments can lead to errors because everything indented is assumed to be part of the code block.

```cpp
// Do NOT:
if ( condition )
    doSomething();
// Because it might become the following in the future,
// which is most likely not what the programmer intended:
if ( condition )
    doSomething();
    doSomethingMoreAddedLater();

// Instead DO:
if ( condition )
{
    doSomething();
}
```

#### Narrow variable scope when possible in loops and statements

**Rule:** When using `while` or `for` loops, and in C++17 `if` statements, use the initialize statement to narrow variable scope.

**Rationale:** This clearly connects variables associated with control statements to the loop or `if` statement, simplifying local reasoning for the developer.

#### Do not exceed 160 characters per line

**Rule:** Do not exceed 160 characters per line.

**Rationale:** Many tools, especially for diffing and merging, do not provide good display of long lines.

### 3.2 Naming Conventions

#### Class names shall use PascalCasing

**Rule:** Class names shall use PascalCasing, for example, `XmlNodeList` instead of `xml_node_list`.

#### Interface names are class names prefixed with an `I`

**Rule:** Interface names adhere to the same rules as class names but are prefixed with an `I`, for example: `IDecoder`.

**Rationale:** Clearly identifies the class as an interface, that is, a class that has only virtual methods and no implementations.

#### Method and function names shall use camelCasing

**Rule:** Method and function names shall use camelCasing, for example, `selectSingleNode`.

#### Prefix class member variable names with `m_`

**Rule:** Prefix class member variable names with `m_`, for example, `m_numberOfFlights`.

**Rationale:** Clarity.

#### Prefix static member variable names with `s_`

**Rule:** Prefix static member variable names with `s_`, for example, `s_numberOfFlights`.

**Rationale:** Clarity.

#### Variable names and function/method parameter names shall use camelCasing

**Rule:** Variables, function parameters, and method parameter names shall use camelCasing, for example, `nodeName` or `userInput`.

#### Do not use identifier names that begin with underscores

**Rule:** Do not use identifiers that begin with one or two underscores (`_` or `__`).

#### Avoid using identifiers that use general words such as Common, Helper, ...

**Rule:** Avoid using identifiers that include words like Common, Helper, Handler, Processor, Manager, Utility, Keeper, Engine, etc. Rather try to identify the actual purpose/functionality of the class, method, or function.

**Rationale:** These words tend to have an unfocused purpose. Thus, classes, functions, or methods with these sorts of names tend to develop into blobs of general unspecified functionality.

#### Do not use unnecessary abbreviations

**Rule:** Do not use unnecessary abbreviations.

```cpp
// Do NOT:
XMLDoc.sel1Nd();
ConfMan.cnfHnd();

// Instead DO:
XmlDocument.selectSingleNode();
ConfigurationManager.readConfiguration();
```

#### Naming rules for classes apply to: constants, typedefs, structs, enums and enum values

**Rule:** Constants, typedefs, structs, enums, and enum values adhere to the same naming rules as classes do.

#### The parameter names are to be the same in function declaration and definition

**Rule:** The names of parameters to functions are to be specified in the function declaration. The parameter names are to be the same in the function declaration and in the function definition.

#### Application names

**Rule:** For executable names, use lower case and dashes without abbreviations, for example, `polaris-flight-data-processor`.

**Rule:** For application names exposed to the user in our HMI applications, use Capitalisation with spaces, for example, `Flight Data Processor`. This does not apply in the case of the Polaris logo being used.

### 3.3 Comments and Documentation

#### Document each class and class method using Doxygen

**Rule:** Document each class and class method using Doxygen.

#### Document each parameter and return value for all functions and class methods

**Rule:** Document each parameter and return value for all functions and class methods. Consider all document strings to be one or more sentences starting with a capital letter.

```cpp
/** An example class for demonstrating doxygen documentation.
 */
class ExampleClass
{
public:
    /** Default constructor.
     */
    ExampleClass() = default;
    /** Constructor.
     *
     * @param exampleString A custom string to store.
     */
    explicit ExampleClass( const std::string & exampleString );
    /** Sets the example string
     *
     * @param exampleString An alternative string to store.
     */
    void setExampleString( const std::string & exampleString );
    /** Gets the example string.
     *
     * @return The example string.
     */
    [[nodiscard]] std::string getExampleString() const;
private:
    std::string m_exampleString{ "Lorem Ipsum" };
};
```

#### All comments are to be written in English

**Rule:** All comments are to be written in English.

**Rationale:** English is the official company language.

---

## 4. Coding Rules For Safety

In contrast to coding style and formatting rules, the coding rules restrict how the features of the programming language may be used. Their objective is to prevent erroneous code by prohibiting or discouraging the use of legal code constructs that tend to be error-prone or not well defined (and thus possibly differently implemented between different compilers).

### 4.1 Legacy Code

#### When old code violates a rule, document the violation

**Rule:** Every time old code that violates a rule is modified, the violation must be clearly documented in the code. The documentation shall refer to these coding guidelines and indicate the violated rules. The documentation shall be close to the violation, for example in the function body.

**Rationale:** New coders tend to work by example and follow existing patterns, even if these patterns are not good. The comment will tell them not to copy this kind of coding construct.

### 4.2 Compilation

#### Code shall compile without warnings and errors

**Rule:** Code shall compile without errors and without warnings. Warnings can be acceptable if there is a good documented reason for them.

**Rationale:** Many warnings indicate problems in the code which may lead to errors.

### 4.3 Global Variables

#### Do not use global mutable variables

**Rule:** Do not use global mutable variables.

**Rationale:** They are difficult to reason about since they expose implementation details to the world. Every function in the program can manipulate the variable as it sees fit. In addition, they usually result in non-reentrant functions that prevent use in multithreaded contexts.

```cpp
// Do NOT:
FilteringContext *filterContext;
void initializeFilters( void )
{
    filterContext = ( FilteringContext * ) av_malloc_array(
        inputFormatContext->nb_streams, sizeof( *filterContext ) )
    ...
}

// Rather DO:
class FFmpegTranscoder : public Transcoder
{
public:
    FFmpegTranscoder() :
        filterContext( 0 )
    {
        ...
    }
    ~FFmpegTranscoder()
    {
        if( filterContext )
        {
            av_free( filterContext );
            filterContext = 0;
        }
        ...
    }
    ...
};
void FFmpegTranscoder::initializeFilters( void )
{
    filterContext = ( FilteringContext * ) av_malloc_array(
        inputFormatContext->nb_streams, sizeof( *filterContext ) );
    ...
}
```

```cpp
// Do NOT:
char * buffer;
int decodedLength;
char * decodedBuffer;
void decodeBuffer( void ) {
    ...
    decodedLength = len;
    decodedBuffer = resultBuffer;
}

// Instead DO:
std::vector<char> decodeBuffer( const std::vector<char> & buffer )
{
    ...
    return resultBuffer;
}
```

#### Do not use static variables in functions except for initialization

**Rule:** Do not use static variables in functions except for initialization, as functions may be called from multiple threads.

**Rationale:** The C++ standard does not require compilers to make the access or modification of static function variables thread-safe; it only guarantees that its initialization is thread-safe.

```cpp
// Do NOT:
void MyClass::onlyOneAtATime()
{
    static bool in_function = false;
    while( in_function );
    in_function = true;
    ...
    in_function = false;
}

// Instead DO:
void MyClass::onlyOneAtATime()
{
    std::lock_guard<std::mutex> guard( m_thereCanBeOnlyOne ); // C++11 (or later)
    ...
}
```

#### Use static variables in functions for global variables that are dependencies

**Rule:** Place global variables that are dependencies of other global variables during initialization inside functions to ensure ordering of initialization.

**Rationale:** The compiler is unable to infer variable dependencies across compilation units at compile time. Placing a global static variable inside a function ensures it is initialized on first use, thereby preventing an incorrect order of initialization which would cause uninitialized dependencies to be accessed.

```cpp
// Do NOT:
// file1.cpp:
const static int s_a = 5;
// file2.cpp:
const static int s_b = s_a * 6;

// Instead DO:
// file1.cpp:
const int &getA()
{
    const static int s_a = 5;
    return s_a;
}
// file2.cpp:
const static int s_b = getA() * 6;
```

### 4.4 Memory Management and Access

#### Use value based initialization whenever possible

**Rule:** Use value based initialization whenever possible; avoid pointers and references.

**Rationale:** Values are easier to reason about, and using pointers constitutes a level of complexity that is often unnecessary.

#### Do not use malloc, realloc and free

**Rule:** Do not use `malloc`, `realloc`, and `free`, except when dealing with library and system functions that specifically require `malloc` and `free`. If necessary, provide C++ wrappers around such interaction.

**Rationale:** C++ already provides containers and constructs that provide safer manipulation of memory and objects than the raw use of `malloc` and `free`, especially during handling of exceptions.

```cpp
// Do NOT:
char *buffer = (char *)malloc( BUFFER_SIZE );
memset( buffer, 0, BUFFER_SIZE );
...
buffer = (char *)realloc( buffer, new_size );
memset( buffer + old_size, 0, new_size - old_size );
...
free( buffer );

// Instead DO:
std::vector<char> buffer( BUFFER_SIZE, 0 );
...
buffer.resize( new_size, 0 );
...
// buffer is automatically destroyed when it goes out of scope
```

#### Do not use delete and naked new

**Rule:** Do not use `new` and `delete` where better C++ language and C++ standard library alternatives exist.

- **All C++ versions:** use stack allocations (see example).
- **C++03 (and older):** avoid `delete` when possible; use `auto_ptr` where possible.
- **C++11 (and newer):** avoid `delete` when possible; use `unique_ptr` and `shared_ptr` (use `make_shared`).
- **C++14 (and newer):** avoid `new` and `delete` when possible; use `make_unique` and `make_shared`. If you must use `new`, then avoid naked `new` — i.e. wrap it in a `unique_ptr` or `shared_ptr` on the same line.

**Rationale:** This provides for higher memory safety, especially in the face of exceptions, by providing automatic cleanup of resources during exceptions as well as providing clear transfer of ownership between execution contexts.

```cpp
// Do NOT:
MyClass *myClass = new MyClass(); // naked new
...
delete myClass;

// Instead DO:
MyClass myClass;
// or
std::unique_ptr< MyClass > myClass( new MyClass ); // C++11
// or
auto myClass = std::make_unique< MyClass >(); // C++14 (or later)
// myClass is automatically destroyed when it goes out of scope
```

#### Do not use new[] and delete[]

**Rule:** Do not use `new[]` and `delete[]` where better C++ standard library alternatives exist.

**Rationale:** The C++ standard library already provides highly optimised and safe containers for many common computing constructs (e.g. lists, maps, queues, stacks, and vectors).

```cpp
// Do NOT:
char *buffer = new char[ buffer_size ];
memset( buffer, 0, buffer_size );
...
delete [] buffer;

// Instead DO:
std::vector<char> buffer( buffer_size, 0 );
...
// buffer is automatically destroyed when it goes out of scope
```

#### Do not use shared_ptr when unique_ptr is sufficient

**Rule:** Do not use a `shared_ptr` when a `unique_ptr` is sufficient.

**Rationale:** Ownership sharing via a `shared_ptr`, though useful, does present a risk of resource leaks and also has higher runtime cost than `unique_ptr`. They also allow access from multiple threads at once, something which `unique_ptr`s prevent by enforcing the use of `std::move`.

#### Prefer shared_ptr< const Object > to shared_ptr< Object >

**Rule:** When possible use a `shared_ptr< const Object >` rather than a `shared_ptr< Object >`.

**Rationale:** As `shared_ptr` may be used to share objects across threads, a `const Object` is always preferable as it guarantees read-only (thread-safe) access.

#### Ensure non-const shared_ptr's are thread-safe

**Rule:** All objects instantiated within a `shared_ptr` which are non-const shall be thread-safe or noted as otherwise.

**Rationale:** As a `shared_ptr` may share ownership across threads, all objects managed by a `shared_ptr` shall be thread-safe.

#### Avoid cyclic references of shared_ptr's

**Rule:** Avoid cyclic references of `shared_ptr`s; use `weak_ptr` to break these.

**Rationale:** Cyclic references of `shared_ptr`s lead to resource leaks unless explicitly broken.

#### Always check weak_ptr's for expiry before use

**Rule:** Always check that a `weak_ptr` has not expired before use.

**Rationale:** The use of a `weak_ptr` indicates that the object's lifetime is not managed through the `weak_ptr`, so there is never a guarantee that it has not expired at some other point in the code.

```cpp
// Do NOT:
std::shared_ptr< Aircraft > pMaverick = std::make_shared< Aircraft >( "F-22" );
std::shared_ptr< Aircraft > pIceman = std::make_shared< Aircraft >( "F-14" );
pMaverick->myWingMan = pIceman; // weak_ptr assignment
pIceman->m_flyCount = 17;
pIceman.reset(); // destroy the object managed by pIceman
std::cout << pMaverick->myWingMan.lock()->m_flyCount << std::endl; // SEGFAULT

// Instead DO:
std::shared_ptr< Aircraft > wingMan = pMaverick->myWingMan.lock();
if ( wingMan )
{
    std::cout << wingMan->m_flyCount << std::endl;
}
```

#### Do not use pointer arithmetic

**Rule:** Do not use pointer arithmetic where better C++ alternatives exist.

**Rationale:** C++ already provides containers and iterators as well as other constructs that provide safer means than raw pointer arithmetic to manipulate memory. In addition, trying to force pointer arithmetic on library containers can result in unexpected and faulty results for container implementations that do not store data in continuous blocks in memory.

```cpp
// Do NOT:
std::deque<int> pending;
...
int * iter = &pending[ skip_count ];
int * end = &pending[ 0 ] + pending.size();
while( iter != end )
{
    ...
    ++iter; // ERROR: std::deque doesn't store data in continuous blocks
}

// Instead DO:
std::deque<int> pending;
...
auto iter = pending.begin() + skip_count;
while( iter != pending.end() )
{
    ...
    ++iter;
}
```

#### When using std::optional members ensure that references to std::optional are returned

**Rule:** When using `std::optional` members for members that may or may not be present, ensure that getters return a reference or copy to the `std::optional` and do not provide a method for retrieving the value without allowing the user to check for presence of a value.

**Rationale:** Forcing a user to call some other method to check that the optional is non-empty is error prone and may be missed in reviews.

```cpp
// Do NOT:
class SomeClass
{
   /** Check whether the optional int has a value
     * @return true if the optional contains a value, false otherwise
     */
   bool hasOptionalInt() const
   {
       return m_optionalInt.has_value();
   }
   /** Get the optional int
     * @return the optional int
     */
   int getOptionalInt() const
   {
       return m_optionalInt.value();
   }
private:
   std::optional< int > m_optionalInt;
}

// Instead DO:
class SomeClass
{
   /** Get the optional int
     * @return the optional int
     */
   const std::optional< int > &getOptionalInt() const
   {
       return m_optionalInt;
   }
private:
   std::optional< int > m_optionalInt;
}
```

### 4.5 Type Safety

#### Do not use the preprocessor for anything but include guards and includes

**Rule:** Do not use the preprocessor for anything but include guards and includes.

**Rationale:** "[#defines] are a glorified text-substitution facility whose effects are applied during preprocessing, before any C++ syntax and semantic rules can even begin to apply." Furthermore "[#defines] ignore scopes, ignore the type system, ignore all other language features and rules, and hijack the symbols they #define for the remainder of a file." Using `const` and/or `enum` instead gives the C++ compiler a chance to provide restrictions on the operations that are permitted for the constant. Furthermore, assigning a type provides semantic meaning to the otherwise generic value of the constant.

```cpp
// Do NOT:
#define FORTYTWO         42

// Instead DO:
const int fortyTwo = 42;
```

#### Do not use const_cast

**Rule:** Do not use `const_cast`.

**Rationale:** `const_cast` invalidates the contract of the function or method, making it impossible to reason about the state of the program in other places.

```cpp
// Do NOT:
void doSomething( const SomeClass * a )
{
    SomeClass * b = const_cast<SomeClass *>(a);
    bool result = b->NonConstFunction();
    // b has now been changed, breaking the contract
    ...
}
```

#### Do not use run-time type checking where compile-time type checking is possible

**Rule:** Do not use run-time type checking such as `dynamic_cast` when it is possible to use compile-time type checking. For any form of type based switching use the visitor pattern.

**Rationale:** Compile-time type checking will catch errors at compile time while run-time type checking requires extensive testing and may still not catch all errors. Additionally, when extending a type system with a new type, any code using `dynamic_cast` to identify the real type of an object will need to be closely examined in order to guarantee that all cases are covered.

```cpp
// Do NOT:
class ShapeWidget
{
public:
    ShapeWidget( const IShape& shape)
    {
        const Circle *cp = dynamic_cast<const Circle *>( &shape );
        const Square *sp = dynamic_cast<const Square *>( &shape );
        if ( cp )
        {
            // Construct circle widget
        }
        else if ( sp )
        {
            // Construct square widget
        }
        else
        {
            // Throw exception!?
        }
    }
};

// Instead DO:
class ShapeWidget : public IShapeVisitor
{
public:
    ShapeWidget( const IShape& shape)
    {
        shape.accept( *this );
    }
    void visit( const Circle& circle )
    {
        // Construct circle widget
    }
    void visit( const Square& square )
    {
        // Construct square widget
    }
};
```

The above code is free of casts, condition statements, and exceptions. Furthermore, extending the number of Shape types and `IShapeVisitor` later will identify all users of the polymorphic `IShape` through compilation errors. A complete example is available in the Appendix.

#### Use modern casting operators, if necessary

**Rule:** If casting is necessary, use modern casting operators (`dynamic_cast`, `static_cast`, `reinterpret_cast`) instead of C-style casting such as `(int)x`. In general, avoid casting.

**Rationale:** In contrast to C-style casting, the modern casting operators are intentionally verbose and therefore preferable to the C-style cast.

### 4.6 Classes, Inheritance and Exposure

#### Prefer objects as members to pointers

**Rule:** Prefer objects as members rather than pointers.

**Rationale:** Objects are easier to reason about than pointers, thereby reducing complexity and increasing safety.

#### Do not use multiple-inheritance of non-interface classes

**Rule:** Do not use multiple-inheritance of non-interface classes.

**Rationale:** Multiple inheritance can lead to undefined behaviour.

**Instead:** Define several interface classes (where all methods are virtual and not implemented) and inherit from those interface classes instead.

#### Use fully constructed objects

**Rule:** Use fully constructed objects. Do not use default constructors unless there are meaningful default values for all members.

**Rationale:** Partially constructed objects lead to errors as functions which later access the object cannot assume that the object is complete and must therefore implement more complex error handling.

```cpp
// Do NOT:
class MyClass
{
public:
    MyClass()
    {
    }
    void setA(int newA)
    {
        a = newA;
    }
private:
    int a;
};
int main(int argc, char *argv[])
{
    MyClass myClass;
    myClass.setA(4);
}

// Instead DO:
class MyClass
{
public:
    MyClass(int a) :
        a{ a }
    {
    }
private:
    int a;
};
int main(int argc, char *argv[])
{
    MyClass myClass{ 4 };
}
```

#### Use the builder pattern when constructing objects from configuration

**Rule:** Use the builder pattern to fill in data needed to construct large objects incrementally. Do not integrate configuration reading inside objects.

**Rationale:** This prevents passing in a large number of arguments at construction and inadvertently connecting an object with a configuration file format which may change over time.

#### Class data members must be private

**Rule:** Class data members must be private.

**Rationale:** Separation of concerns and ownership of data.

```cpp
// Do NOT:
class A
{
public:
    ...
protected:
    int a;
};
class B : public A
{
    ...
    void doSomething()
    {
        a = workVariable;
    }
}

// Instead DO:
class A
{
public:
    setA( int val )
    {
        a = val;
    }
    ...
private:
    int a;
};
class B : public A
{
    ...
    void doSomething()
    {
        A::setA( workVariable );
    }
}
```

#### Only template or trivial methods may be defined within a class declaration

**Rule:** Only template or trivial methods may be defined within a class declaration.

**Rationale:** Template member functions must be in the class declaration and trivial methods should be defined in the header for inlining. All other implementation should not be exposed in the class declaration.

#### All member functions that do not affect the state of the object are to be declared const

**Rule:** All member functions that do not affect the state of the object are to be declared `const`.

**Rationale:** This allows the method to be used from const references.

### 4.7 Error Handling / Logging

#### Use C++ exception handling for errors only

**Rule:** Use C++ exception handling for errors only.

**Rationale:** C++ implementations generally optimize on the assumption that exceptions are used for error handling and that they are rare. Throwing an exception thus has some overhead. An error is defined to be any execution that compromises the correctness of the program, either locally or globally — for example, an invalid value, an out-of-range or unexpected value in an argument, or input from external sources if it compromises the correctness of the program.

```cpp
// Do NOT:
try {
    for (int i = 0; i < vec.size(); ++i)
        if (vec[i] == x) throw i; // found x
} catch (int i) {
    return i;
}

// Instead DO:
for (int i = 0; i < vec.size(); ++i)
{
    if (vec[i] == x)
    {
        return i; // found x
    }
}
```

#### Use C++ exception handling to propagate errors

**Rule:** Use C++ exception handling to propagate errors.

**Rationale:** Using C++ exceptions provides for more readable code by moving error handling to a separate block, without the need for conditional checks and goto/label statements. In addition, throwing a C++ exception will clean up all objects that have been initialized and are on the call stack of functions in-scope of the exception handler.

```cpp
// Do NOT:
[...]
if( ( errVal = myObject.method1() ) < 0 ) goto cleanUp;
if( ( errVal = myObject.method2() ) < 0 ) goto cleanUp;
[...]
cleanUp:
if( errVal < 0 )
{
    // handle the error
}

// Instead DO:
try
{
    [...]
    myObject.method1();
    myObject.method2();
    [...]
}
catch( std::exception e )
{
    // handle the exception
}
```

#### Always check fault codes of library functions

**Rule:** Check the fault codes which may be received from library functions even if these functions seem foolproof.

**Rationale:** Not doing this may allow unhandled errors to propagate beyond the function scope.

```cpp
// Do NOT:
std::stringstream ss;
int number = 123;
ss << "Nothing to see here";
ss >> number;
// number is untouched and is still 123
std::cout << "The invalid number is:" << number << std::endl;

// Instead DO:
std::stringstream ss;
int number = 123;
ss << "Nothing to see here";
ss >> number;
if( ss.fail() )
{
    // handle exception here
}
// number == 123
```

#### Do not use assert in production code

**Rule:** Do not use `assert` in production code.

### 4.8 Control Flow Statements & Expressions

#### Use parentheses to clarify order precedence in an expression

**Rule:** Use parentheses to clarify the order precedence in an expression.

**Rationale:** Using parentheses makes the code more readable and makes it immediately clear to even entry level developers in what order expressions will be evaluated.

```cpp
// Do NOT:
if( a && b == c && d) // the order of operands is a && ( b == c ) && d
// Instead DO:
if( a && ( b == c ) && d )

// Do NOT:
std::cout << a & b; // this is equivalent to ( std::cout << a ) & b;
// Instead DO:
std::cout << (a & b);
```

#### Do not assign inside a condition statement

**Rule:** Do not assign inside a condition statement.

**Rationale:** Accidentally leaving out a `=` or `!` from `==` or `!=` occurs occasionally, which can result in frustrating debug sessions. Being able to turn on a compiler flag that detects assignments in logic statements and not have it flag "intended" assignments helps us towards the goal of having warning-free code.

```cpp
// Do NOT:
if( selected_color = BLACK ) // assigns black to selected_color

// Instead DO:
selected_color = BLACK;
if( BLACK == selected_color ) // checks if selected_color is equal to BLACK
```

#### Place constants on the left hand side of an equality operator

**Rule:** Place constants on the left hand side of an equality operator.

**Rationale:** This prevents accidental assignment instead of comparison.

```cpp
// Do NOT:
if( selected_color == BLACK) // Error prone. Assigns BLACK to selected_color if '=' is used instead of '=='

// Instead DO:
if( BLACK == selected_color ) // compiler error if '=' is used instead of '=='
```

#### Do not use the ==, != operators with doubles and floats

**Rule:** Do not use the `==`, `!=` operators with doubles and floats.

**Rationale:** This is not meaningful as doubles and floats are numbers that have a fixed precision regardless of how large (or small) they are. This means that the current value depends on prior operations, how the computer represents the constants used, and how large the numbers under comparison are.

```cpp
// Do NOT:
for( double d = 0.0; d != 0.3; d += 0.1 ); // never terminates

// Do NOT:
if( abs( a - b ) < std::numeric_limits<double>::epsilon() )
// std::numeric_limits<double>::epsilon() doesn't mean what you think it does,
// this will not work if a and b are really large

// Instead DO:
// the machine epsilon has to be scaled to the magnitude of the values used
// and multiplied by the desired precision in ULPs (units in the last place)
if ( std::abs(x - y) < std::numeric_limits<T>::epsilon() * std::abs(x + y) * ulp
    // unless the result is subnormal
    || std::abs(x - y) < std::numeric_limits<T>::min() )
```

#### A switch statement must contain a default branch

**Rule:** A switch statement must always contain a `default` branch which handles unexpected cases.

**Rationale:** Using a default branch ensures that unexpected values are caught and handled.

```cpp
// Do NOT:
switch(type)
{
    case 1:
    {
        // do something
        break;
    }
    case 2:
    {
        // do something else
        break;
    }
} // type might have an unexpected value that now goes unnoticed

// Instead DO:
switch(type)
{
    case 1:
    {
        // do something
        break;
    }
    case 2:
    {
        // do something else
        break;
    }
    default:
    {
        // unknown type. Do error-handling
        break;
    }
}
```

### 4.9 Threading (Safety)

Threads are only recently supported natively in C++ and the API does not provide safety guarantees as present in other languages such as Rust or Erlang.

#### Avoid threading when possible

**Rule:** Applications should be single-threaded by default with threads only added when performance or integration with third-party libraries require it.

**Rationale:** Threads always increase complexity and pose a risk in any development as C++ does not have a safety-oriented threading model which guarantees exclusive memory access or similar; instead locking and synchronization must be done by hand.

#### Do not create your own thread communication APIs

**Rule:** When threading must be used, prefer existing APIs for communicating between threads such as `post()` methods in Qt or Boost ASIO.

**Rationale:** These APIs are well understood, well tested, and well documented. Designing thread communication APIs requires extensive testing and validation.

#### Use standard library facilities for short-term parallel execution

**Rule:** Use `std::async` or similar APIs to perform short-term parallel execution, for example database loading, from a single function scope.

**Rationale:** These are relatively safe given that they are called from a single function scope and the same function awaits the result.

#### Avoid sharing the main data-model to other threads

**Rule:** The main data-model for any application should be updated and preferably read from a single thread only.

**Rationale:** Fine-grained locking of parts of the data-model is unwanted and typically leads to unmanageable complexity; coarse-grained locking of the whole data-model prevents execution of other threads and removes benefits of threading.

---

## 5. Coding Rules for Performance

These coding rules are oriented towards performance, specifically to avoid unnecessary pessimization of code — that is, writing code that is less performant than it needs to be.

### 5.1 Memory Allocations

Memory allocations and freeing are a major source of performance degradation in programs; in many cases they are simple to avoid or remove.

#### Avoid obscured memory allocations

**Rule:** Avoid allocating memory by using dynamic data-types when static ones will do.

**Rationale:** Many dynamic data-types can be replaced with static ones which avoid a memory allocation.

```cpp
// Do NOT:
std::vector< std::string > helloWorld = { "Hello", "world" };

// Instead DO:
std::array< std::string, 2 > helloWorld = { "Hello", "world" };
// or
auto helloWorld2 = { "Hello", "world" };
```

#### Pre-allocate memory when capacity needs are known

**Rule:** Pre-allocate memory when capacity needs are known, or even if a minimum can be known or estimated.

**Rationale:** Expanding memory use incrementally is less performant than a pre-allocation as the memory allocator needs to be called whenever current capacity is exceeded.

```cpp
// Do NOT:
std::vector< std::string > strings;
for (const auto &string : source.iterate_strings() )
{
    strings.push_back( string );
}

// Instead DO:
std::vector< std::string > strings;
strings.reserve( source.get_element_count() ); // pre-reserve space.
for (const auto &string : source )
{
    strings.push_back( string );
}
```

#### Consider using variants rather than pointers to interfaces for polymorphism

**Rule:** Consider using variants rather than interface pointers to allocated memory for polymorphism when working with closed sets.

**Rationale:** A variant can be stack allocated while an interface pointer to allocated memory requires a heap allocation. Closed sets is a key term here, as using variant-based polymorphism makes it harder to add new types than with polymorphism based on class inheritance and virtual methods. Variant-based polymorphism and regular "virtual" polymorphism both have their strengths and weaknesses.

```cpp
// Do NOT:
std::unique_ptr< IAction > action = std::make_unique< VerticalAction>( "F320" );

// Instead DO:
using ActionVariant = std::variant< VerticalAction, SpeedAction, RouteAction >;
...
ActionVariant action{ VerticalAction{ "F320" } };
```

### 5.2 System Calls

System calls trap into the kernel, often slowing program execution substantially.

#### Minimize the number of system calls

**Rule:** Minimize the number of system calls you make. Consider the necessity or possibility of coalescing any system calls you make; this includes things like opening/closing files, reading/writing to/from files, logging too often, and sending/receiving from network.

**Rationale:** Many of these actions can be avoided or coalesced quite trivially.

```cpp
// Do NOT:
while (...)
{
    int readCount = read( socket, &buffer, sizeof(buffer) ); // Reads one message at a time
    if ( readCount > 0 )
    {
        processData( &buffer );
    }
    ... // error handling code...
}

// Instead DO:
while (...)
{
    int readCount = recvmmsg( socket, messages, messagesLength, 0, nullptr ); // Reads all available messages
    if ( readCount > 0 )
    {
        processData( messages );
    }
    ... // error handling code...
}
```

### 5.3 Function Calls

#### Do not by default make method calls virtual

**Rule:** Do not make methods on objects virtual by default. If methods need to consume different types, prefer a template method to an inheritance hierarchy.

**Rationale:** Virtual function calls require pointer de-referencing, slowing execution and often preventing compiler inlining of function calls and other optimizations both by the compiler and by the processor (around 10% performance hit on a simple call). Templates are instead generated per caller and are fully optimized.

```cpp
// Do NOT:
class IPacketWithTimeStamp
{
   virtual long getTimeStamp() const;
};
class FmtpPacket : public IPacketWithTimeStamp
{
    long getTimeStamp() const override;
    ...
};
class MatipPacket : public IPacketWithTimeStamp
{
    long getTimeStamp() const override;
    ...
};
class PacketTimeStampCollector
{
    void collectTimeStamp( const IPacketWithTimeStamp &packetWithTimeStamp )
    {
        auto timeStamp = packetWithTimeStamp.getTimeStamp();
        ...
    }
    ...
};

// Instead DO:
class FmtpPacket
{
    long getTimeStamp() const;
    ...
};
class MatipPacket
{
    long getTimeStamp() const;
    ...
};
class PacketTimeStampCollector
{
    template <typename T > void collectTimeStamp( const T &packetWithTimeStamp )
    {
        auto timeStamp = packetWithTimeStamp.getTimeStamp();
        ...
    }
    ...
};
```

#### Consider making trivial methods and functions inline

**Rule:** Consider making trivial methods and functions inline and defined in the header.

**Rationale:** From the ISO CPP FAQ — _Do inline functions improve performance?_ Answer: Yes and no. Sometimes. Maybe. There are no simple answers. Inlining should be considered in case of performance issues and results should be measured.

#### Define inline member functions outside the class body

**Rule:** Define inline member functions outside the class body.

**Rationale:** Defining an inline member function inside the class body can be more convenient but it is considered best practice to define it outside. Defining it inside is easier on the person who writes the class but harder on all the readers since it mixes what a class does (the external behavior) with how it does it (the implementation).

```cpp
// Do NOT:
// Speed.h
class Speed {
public:
    Speed(double speed);
    Speed operator+( Speed other)
    {
        return Speed(m_speed + other.m_speed);
    }
    double m_speed;
};

// Instead DO:
// Speed.h
class Speed {
public:
    Speed(double speed);
    Speed operator+( Speed other); // Best practice: Don't put the inline keyword here
    double m_speed;
};
inline Speed Speed::operator+( Speed other ) // Best practice: Put the inline keyword here
{
    return Speed(m_speed + other.m_speed);
}
```

### 5.4 Threading (Performance)

Threads provide for local parallelism, but using threads can also slow down execution when done incorrectly. Typically legitimate uses of threading are for programs that service requests with minimal communication between threads, or for parallelizing CPU-heavy tasks.

#### Avoid passing data between threads at high-frequency

**Rule:** Avoid passing data between threads at high frequency, especially heap allocated data.

**Rationale:** There are multiple performance issues at play:

- **Cache coherency:** Data that is actively in use by a given core resides in the cache of that core. Once the data is requested by another core, the original core will be forced to flush that data, typically to main memory, for the other core to then load it into its own cache. This slows execution on both cores.
- **Memory pool contention:** If heap allocated memory is used and pointers are freed on a different thread than they were allocated, then there is a high probability that the memory pool from which the memory was allocated was not the one last used by the current thread. This forces the core executing the freeing thread to acquire a mutex on a memory pool that was just used by the allocating thread, potentially putting the freeing thread to sleep and then forcing it to load the structure of a previously unused memory pool into its cache, slowing execution.

### 5.5 RTTI

Run-time-type-information is used to identify the type of an object based on its vtable pointer.

#### Avoid using RTTI

**Rule:** Avoid using RTTI as in `dynamic_cast`, `typeid`, or others.

**Rationale:** These can be very slow as they typically involve traversing a hierarchy of vtable pointers to see if one matches. Needing to identify a true type at runtime should not be needed as closed sets can be handled with `std::variant`.

### 5.6 Control Flow and Arguments

Control flow is the branching of the code based on logic such as if-statements, switch statements, and function calls which set scope. Some of these constructs carry severe performance impact.

#### Avoid try-catch blocks as part of control flow

**Rule:** Avoid try-catch blocks as part of control flow.

**Rationale:** Throwing an exception is a heavy operation and should be avoided as much as possible (the overhead of validating a string as a latlon and then parsing it vs throwing an exception due to a parsing failure is around 10,000%).

```cpp
// Do NOT:
try
{
    auto latLon = LatLon::parse("435N03W");
    std::cout << "LatLon: " << latLon.toString() << "\n";
}
catch ( const ArgumentException & )
{
}

// Instead DO:
if ( LatLon::validateStringAsLatLon("435N03W") )
{
    auto latLon = LatLon::parse("435N03W");
    std::cout << "LatLon: " << latLon.toString() << "\n";
}
```

#### Use a constant reference to large objects instead of pass-by-value

**Rule:** Use a constant reference to large objects instead of call-by-value when your function does not need to modify the argument.

**Rationale:** Creating a new large object that is a copy of the argument, without intending to change it, is a waste of CPU and memory resources, while using a const reference simply passes a pointer to the function while at the same time mandating that the function not change what that pointer references. In this context a large object is something other than a basic type or a thin wrapper around a pointer (for example, an iterator).

```cpp
// Do NOT:
void someFunction( MyBigClass myBigObject )
{
    ... // code that doesn't alter or use the alteration of myBigObject
}

// Instead DO:
void someFunction( const MyBigClass & myBigObject )
{
    ... // code that doesn't alter or use the alteration of myBigObject
}
```

However, for small objects:

```cpp
// Do NOT:
bool MyClass::someMethod( const MyClass::iterator & iter )
{
    ... // code that doesn't alter or use the alteration of iter
}

// Instead DO:
bool MyClass::someMethod( const MyClass::iterator iter )
{
    ... // code that doesn't alter or use the alteration of iter
}
```

### 5.7 Algorithms and Data-Types

Know your standard library algorithms and data-types; they are in the majority of cases preferable to rolling your own.

#### Use std::vector for most types of sets

**Rule:** Use `std::vector` for most types of sets like lists, heaps, queues, etc.

**Rationale:** Memory used by `std::vector` is always contiguous and allocated with margin for growth, meaning cache usage is efficient and memory allocations are minimized.

#### Use standard library algorithms when possible

**Rule:** Use standard library algorithms rather than using your own for-loops or similar.

**Rationale:** These are typically faster than hand-rolled loops and receive optimizations included in the standard library.

```cpp
// Do NOT:
std::optional< Item > foundItem;
for (const auto &item : items )
{
   if (someCondition(item))
   {
       foundItem = item;
       break;
   }
}
if ( foundItem )
{
...
}

// Instead DO:
auto foundItemItr = std::find_if( items.begin(), items.end(), [](const auto &item){ someCondition( item ) }; );
if ( foundItemItr != items.end() )
{
...
}
```

Algorithms with which everyone should be familiar:

```cpp
std::find // Find by exact value
std::mismatch // Find the first non-matching elements in two iterable sets
std::lower_bound, std::upper_bound, std::binary_search, std::equal_range // Binary search on sorted sets returning different information
std::set_difference // Identify the difference between sets
std::copy, std::copy_if // Copy and conditional copy
```

---

## 6. Reading Material

- Other C++ Coding Standards / Guidelines
- Language and Library References
- Useful forums and FAQs

---

## 7. Appendix

**Visitor Pattern C++11 Example — Prefer `std::variant` with C++17.**

This appendix provides a complete worked example of the visitor pattern referenced in the type-safety section ("Do not use run-time type checking where compile-time type checking is possible"). With C++17 and later, prefer `std::variant` over a hand-rolled visitor hierarchy for closed sets of types.