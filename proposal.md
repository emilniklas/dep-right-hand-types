# Right Hand Types

## Contact information

* **Emil Persson**
* Email: [emil.n.persson@gmail.com](mailto:emil.n.persson@gmail.com)
* GitHub: [@emilniklas](https://github.com/emilniklas)
* Twitter: [@emilpersson](https://twitter.com/emilpersson)

DEP Repository: https://github.com/emilniklas/dep-right-hand-types

## Summary

This proposal questions the semantics of putting type annotations on the left hand side of functions, methods, variables, and constants. Instead, it suggests to go in the same direction as TypeScript, Swift, Scala, and other young, modern languages of today.

```dart
// left hand types (Dart today)
String function() {
  String variable = "value";
  return variable;
}
```

```dart
// right hand types
function(): String {
  var variable: String = "value";
  return variable;
}
```

## Motivation

One of Dart's greatest strengths is its way of straightening out inconsistencies in the languages of the web. In particular, it takes concepts, syntax, and constructs from languages like Java, JavaScript, Smalltalk, C, and C#, and unifies them in a semantic and straightforward way.

However, we know that Dart struggles to justify itself with the competition. One reason why might be that Dart is stuck between trying to be accessible and trying to be practical and powerful.

At a glance, Dart inherits much of its syntax from C and statically typed C family languages like Java and C#. This syntax is combined (to be accessible to most developers) with JavaScript syntax. Together they result in the dynamic optional typing that we know and love.

But they do result in inconsistencies, that very well may be confusing enough to be a deal breaker for newcomers.

### Mutability
There are three types of mutability declarations in Dart, `const`, `final`, and `var`. `const` is immutable and with a value that is known at *compile time*. `final` is also immutable, but the value can still vary at runtime. `var` is mutable.

```dart
var a = 'value';
final b = 'value';
const c = 'value';
```

### Static types
Every value *can* be typed in Dart, simply by putting the type before the name of the value, but after the mutability keyword. Since `var` is the default mutability, the `var` keyword get replaced with the type name altogether.

```dart
String a = 'value';
final String a = 'value';
const String a = 'value';
```

Although this is easy to understand, it raises two important points:

1. Mutability is encouraged (the opposite require an increase in verbosity).
2. `var` is indistinguishable from a type annotation.

### Optional types and type inference
Without any type annotation, the type of the value gets inferred by static analysis of the code. If the type cannot be determined, it is considered `dynamic`. The `dynamic` type can also be declared, forcing the analyzer to consider the value `dynamic` instead of inferring the type.

So the difference between `var a = 'value';` and `dynamic a = 'value';` is that the former is inferred to be of type `String` and the latter is forced to be considered `dynamic`.

As if this wasn't enough, the `dynamic` keyword (although considered a type name), does not follow the semantics of upper camel casing of all types. The same goes for `bool`, `num`, `double`, and `int`.

```dart
var x;                // implicit dynamic type, mutable
final x;              // implicit dynamic type, immutable
const x;              // implicit dynamic type, immutable
var x = 0;            // implicit int type, mutable
final x = 0;          // implicit int type, immutable
const x = 0;          // implicit int type, constant
dynamic x = 0;        // explicit dynamic type, mutable
final dynamic x = 0;  // explicit dynamic type, immutable
const dynamic x = 0;  // explicit dynamic type, constant
int x = 0;            // explicit int type, mutable
final int x = 0;      // explicit int type, immutable
const int x = 0;      // explicit int type, constant
```

## Examples

The proposed way of cleaning up the confusion from above, is to follow the example of TypeScript, Swift, Scala, and others.

```dart
var x;                // implicit dynamic type, mutable
final x;              // implicit dynamic type, immutable
const x;              // implicit dynamic type, immutable
var x = 0;            // implicit int type, mutable
final x = 0;          // implicit int type, immutable
const x = 0;          // implicit int type, constant
var x: dynamic = 0;   // explicit dynamic type, mutable
final x: dynamic = 0; // explicit dynamic type, immutable
const x: dynamic = 0; // explicit dynamic type, constant
var x: int = 0;       // explicit int type, mutable
final x: int = 0;     // explicit int type, immutable
const x: int = 0;     // explicit int type, constant
```

Here, the name of the value is coupled with the mutability, which better reflects that the mutability of this particular reference only affects this identifier. A `final a` can be referenced as a mutable value just by assigning `var b = a`.

Function declarations could easily be adapted to this syntax too:

```dart
describe(person: Person): String {
  return '${person.name} is ${person.mood}';
}
```

However, there is an inconsistency with the fact that function arguments are mutable as a default too. A more consistent function would be `functionName(var argument: ArgmuentType): ReturnType`, but that is probably one step too far.

## Proposal

### Clarity
As seen above, this proposal would give a much clearer and accessible way to express the optional typing of Dart. `mutability name | mutability name: type` is much easier to take in than `mutabilityOrType | mutabilityExceptVar type`.

### Accessibility
Another rationale behind this is that many other languages that use the right hand type and colon syntax also has type inference, which would mean a greater understanding from the get go than what we have now. C style languages with left hand types are more commonly strictly typed.

### Anonymous Typedefs
Types are complex, especially in languages like Dart, where functions are types in itself. Generics do their part, but for more complex return types, the `typedef` keyword helps Dart clean up generics and the left hand side of identifiers. Inconsistently, function arguments can be more explicitly typed without `typedef`, bringing even more noise to the actual argument names.

Consider this scenario: A function `snore` receives a function `callMultiple` that takes a list of functions `functions`—where each takes a `String` `s` and returns a `String`—and returns a list of the results of the functions. The `snore` function then returns a String of the joined strings.

##### Untyped
```dart
snore(callMultiple) {
  return callMultiple([
    (s) => s.toUpperCase(),
    (s) => s.toLowerCase(),
    (s) => s.toUpperCase(),
    (s) => s.toLowerCase(),
  ]).join();
}

callMultipleWithZ(functions) {
  return functions.map((f) => f('z'));
}

snore(callMultipleWithZ); // ZzZz
```

##### Typedef
```dart
typedef String StringManipulator(String s);
typedef Iterable<String> CallMultiple(Iterable<StringManipulator> functions);

String snore(CallMultiple callMultiple) {
  return callMultiple([
    (s) => s.toUpperCase(),
    (s) => s.toLowerCase(),
    (s) => s.toUpperCase(),
    (s) => s.toLowerCase(),
  ]).join();
}

Iterable<String> callMultipleZ(Iterable<StringManipulator> functions) {
  return functions.map((StringManipulator f) => f('z'));
}

snore(callMultipleZ); // ZzZz
```

##### Invalid Dart without Typedef
```dart
String snore(Iterable<String> callMultiple(Iterable<String manipulator(String string)> functions)) {
  return callMultiple([
    (s) => s.toUpperCase(),
    (s) => s.toLowerCase(),
    (s) => s.toUpperCase(),
    (s) => s.toLowerCase(),
  ]).join();
}

Iterable<String> callMultipleZ(Iterable<String manipulator(String string)> functions) {
  return functions.map((String f(String string)) => f('z'));
}

snore(callMultipleZ); // ZzZz
```

##### A Better Solution
Although this scenario is quite ridiculous, look at what happens if move the types to the right hand side of a colon, as well as removes the unneccessary names within the expected type signatures:

```dart
snore(callMultiple: (Iterable<(String): String>): Iterable<String>): String {
  return callMultiple([
    (s) => s.toUpperCase(),
    (s) => s.toLowerCase(),
    (s) => s.toUpperCase(),
    (s) => s.toLowerCase(),
  ]).join();
}

callMultipleZ(functions: Iterable<(String): String>): Iterable<String> {
  return functions.map((f: (String): String) => f('z'));
}

snore(callMultipleZ); // ZzZz
```

And if we add back the `typedef` keyword to this, it starts to look okay!

```dart
typedef StringManipulator(String): String;
typedef CallMultiple(Iterable<StringManipulator>): Iterable<String>;

snore(callMultiple: CallMultiple): String {
  return callMultiple([
    (s) => s.toUpperCase(),
    (s) => s.toLowerCase(),
    (s) => s.toUpperCase(),
    (s) => s.toLowerCase(),
  ]).join();
}

callMultipleZ(functions: Iterable<StringManipulator>): Iterable<String> {
  return functions.map((f: StringManipulator) => f('z'));
}

snore(callMultipleZ); // ZzZz
```

You could even assign the `snore` function to a value with a type of `(((String): String): Iterable<String>): String`, and that isn't really that bad either.

### Anonymous Functions with Return Types
Currently in Dart, you cannot specify the return type of an anonymous function with a block body. This can be quite annoying, especially in the analyzer's strong mode, where you would have to add an `as` cast after the function declaration to satisfy the analyzer.

```dart
a(String stringFactory()) { ... }

a(() {
  return '';
});
```

Putting the return type in front of the argument list (as would be the most logical) makes the anonymous function hard to distiguish from a function call or a function name.

```dart
a(String () { ... });

a(myPoorlyCasedType() { ... });
```

With right hand types, we have a nice place to put the return type:

```dart
a((): String { ... });
```

### Combined with Union Types
Given the proposed Union Type syntax with pipes, left hand types get noisy:

```dart
// Spacious union types
String | num parse(String | num argument);

// Terse union types
String|num parse(String|num argument);

// Parenthesized terse union types
(String|num) parse((String|num) argument);
```

With right hand types, the identifiers line up clearly to the left, and all the type names neatly to the right of a semicolon.

```dart
parse(argument: String | num): String | num;
```

### Combined with function modifiers
The function modifiers `sync*`, `async`, and `async*` get added to the right of the types, more clearly coupling the return type with the modifier:

```dart
getJson(url: String): Future<Map> async { ... }
```

### With Value Declaration Lists
This is less intuitive. To declare multiple variables of the same type and mutability, this is how you do it in Dart.

```dart
final int x, y, z;

// with initialization
final int x = 0, y = 1, z = 2;
```

Here we have a choice to either annotate types for each identifier, or to let the type be to the right of the expression. Both are unsatisfactory.

```dart
final x = 0,
      y = 1,
      z = 2
      : int;
// or
final x: int = 0,
      y: int = 1,
      z: int = 2;
```

Typing every identifier is probably a better solution for consistency's sake, but it removes half of the reasons to have a list of declarations instead of just each value for itself. Maybe it would be better to remove that syntax all together, since this is what most developers do anyway:

```dart
final x: int = 0;
final y: int = 1;
final z: int = 2;
```

### With named parameters
Named parameters in parameter lists take their syntax from JavaScript's "options" parameters. It uses colons to designate default values. We need to change that to `=` since we have a new use for colons. But isn't that more consistent with optional arguments anyway?

```dart
x(a: int, [b: int = 2]) { ... }
y(a: int, {b: int = 2}) { ... }
```

## Alternatives

This proposal suggests quite a drastic change to the core syntax of the language. A more lightweight solution to the main issues raised in the *Motivation* section of this document would be to simply add the `var` keyword before types and call it a day.

```dart
var x;                // implicit dynamic type, mutable
final x;              // implicit dynamic type, immutable
const x;              // implicit dynamic type, immutable
var x = 0;            // implicit int type, mutable
final x = 0;          // implicit int type, immutable
const x = 0;          // implicit int type, constant
var dynamic x = 0;    // explicit dynamic type, mutable
final dynamic x = 0;  // explicit dynamic type, immutable
const dynamic x = 0;  // explicit dynamic type, constant
var int x = 0;        // explicit int type, mutable
final int x = 0;      // explicit int type, immutable
const int x = 0;      // explicit int type, constant
```

## Implications and limitations

As mentioned, this _is_ an extreme change to the look of the language. The overall identity and feel of the code will change, and it will take a while to get used to for well adjusted developers.

Besides being an obvious major breaking change, the migration from the old syntax to the new should be very easy to implement. It is fully backwards compatible behaviour-wise.

## Deliverables

### Language specification changes

```diff

variableDeclaration:
-   declaredIdentifier (‘, ’ identifier)*
+   declaredIdentifier (‘, ’ typedIdentifier)*
  ;

declaredIdentifier:
-   metadata finalConstVarOrType identifier
+   metadata mutability typedIdentifier
  ;

- finalConstVarOrType:
-   final type? |
-   const type? | varOrType
-   ;

- varOrType:
-   var | type
-   ;

+ mutability:
+   var | final | const
+   ;

+ typedIdentifier:
+   identifier (‘: ’ type)?
+   ;

initializedVariableDeclaration:
  declaredIdentifier (‘=’ expression)? (‘, ’ initializedIdentifier)*
  ;

initializedIdentifier:
-   identifier (‘=’ expression)?
+   typedIdentifier (‘=’ expression)?
  ;

initializedIdentifierList:
  initializedIdentifier (‘, ’ initializedIdentifier)*
  ;

```

```diff

functionSignature:
-   metadata returnType? identifier formalParameterList
+   metadata identifier formalParameterList (‘: ’ returnType)?
  ;

```

```diff

normalFormalParameter:
-   functionSignature | fieldFormalParameter | simpleFormalParameter
+   fieldFormalParameter | simpleFormalParameter
  ;

simpleFormalParameter:
  declaredIdentifier | metadata identifier
  ;

fieldFormalParameter:
-   metadata finalConstVarOrType? this ‘.’ identifier formalParameterList?
+   metadata mutability? this ‘.’ identifier (‘: ’ type)?
  ;

```

```diff

defaultNamedParameter:
-   normalFormalParameter ( ‘:’ expression)?
+   normalFormalParameter ( ‘=’ expression)?
  ;

```

```diff

type:
  typeName typeArguments?
+   |
+   anonymousTypedef
  ;

typeName:
  qualified
  ;

typeArguments:
  ’<’ typeList ’>’
  ;

+ anonymousTypedef:
+   ’(’ typeList ’)’
+   (’: ’ returnType)?
+   ;

typeList:
  type (’, ’ type)*
  ;

```

### A working implementation

TBD

### Tests

TBD

## Patents rights

TC52, the Ecma technical committee working on evolving the open [Dart standard][], operates under a royalty-free patent policy, [RFPP][] (PDF). This means if the proposal graduates to being sent to TC52, you will have to sign the Ecma TC52 [external contributer form][] and submit it to Ecma.

[dart standard]: http://www.ecma-international.org/publications/standards/Ecma-408.htm
[rfpp]: http://www.ecma-international.org/memento/TC52%20policy/Ecma%20Experimental%20TC52%20Royalty-Free%20Patent%20Policy.pdf
[external contributer form]: http://www.ecma-international.org/memento/TC52%20policy/Contribution%20form%20to%20TC52%20Royalty%20Free%20Task%20Group%20as%20a%20non-member.pdf
