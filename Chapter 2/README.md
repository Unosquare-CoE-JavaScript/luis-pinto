## Each File is a Program

In JS, each standalone file is its own separate program.

The reason this matters is primarily around error handling.Since JS treats files as programs, one file may fail (during parse/compile or execution) and that will not necessarily prevent the next file from being processed.

Many projects use build process tools that end up combining separate files from the project into asingle file to be delivered to a web page. Whenthis happens, JS treats this single combined fileas the entire program.

The only way multiple standalone .js files act as a single program is by sharing their state (and access to their public functionality)via the“global scope.”They mix together in this global scope namespace, so at runtime they act as as whole.

Regardless of which code organization pattern (and loading mechanism) is used for a file (standalone or module), you should still think of each file as it sown(mini)program,which may then cooperate with other (mini) programs to perform the functions of your overall application.

## Values

The most fundamental unit of information in a program is avalue. Values are data. They’re how the program maintainsstate. Values come in two forms in JS: **primitive** and **object.**

Consider:
```Js
console.log("My name is ${ firstName }.");
// My name is ${ firstName }.
console.log('My name is ${ firstName }.');
// My name is ${ firstName }.
console.log(`My name is${firstName}.`);
// My name is Kyle.
```
Assuming this program has already defined a variable firstName with the string value "Kyle", the`-delimited string then resolves the variable expression (indicated with${ ..}) to its current value. This is called **interpolation**.

In addition to strings, numbers, and booleans, two other *primitive* values in JS programs are *null* and *undefined*.While there are differences between them (some historic and some contemporary), for the most part both values serve the purpose of indicating *emptiness*(or absence) of a value.

However, it’s safest and best to use only *undefined* as the single empty value,even though *null* seems attractive in that it’s shorter to type!

```Js
while(value!=undefined) {
    console.log("Still got something!");
    }
```

The final primitive value to be aware of is a symbol, which is a special-purpose value that behaves as a hidden unguessable value. Symbols are almost exclusively used as special keys on objects:

```Js
hitchhikersGuide[ Symbol("meaning of life") ];
// 42
```

## Arrays And Objects
Besides primitives, the other value type in JS is an object value.As mentioned earlier, arrays are a special type of object that’scomprised of an ordered and numerically indexed list of data:
```Js
names=["Frank","Kyle","Peter","Susan"];
names.length;
// 4
names[0];
// Frank
names[1];
// Kyle
```
JS arrays can hold any value type, either primitive or object(including other arrays). 

Objects are more general: an unordered, keyed collection of any various values.In other words,you access the element by a string location name (aka “key” or “property”) rather than by its numeric position (as with arrays). 

##  Value Type Determination
For distinguishing values, thetypeofoperator tells you itsbuilt-in type, if primitive, or"object"otherwise:

```js
typeof 42; // "number"
typeof "abc"; // "string"
typeof true;// "boolean"
typeof undefined;// "undefined"
typeof null;// "object" -- oops, bug!
typeof {"a":1};// "object"
typeof [1,2,3];// "object"
typeof functionhello(){};// "function"
```

*typeof null* unfortunately returns "object" instead of the expected "null". Also,*typeof* returns the specific "function" for functions,but not the expected "array" for arrays.

Converting from one value type to another, such as from string to number, is referred to in JS as **“coercion.”** 

## Declaring and Using Variables
Variables have to be declared (created) to be used. Thereare various syntax forms that declare variables (aka, “iden-tifiers”), and each form has different implied behaviors.

The *var* keyword declares a variable to be used in that part of the program, and optionally allows initial value assignment.
```js
var name="Kyle";
var age;
```

The *let* keyword has some differences to *var*, with the most obvious being that *let* allows a more limited access to the variable than *var*. This is called “block scoping” as opposed to regular or function scoping.
Consider:
```js
var adult=true;
if (adult) {
    var name="Kyle";
    let age=39;
    console.log("Shhh, this is a secret!");
}
console.log(name);
// Kyle
console.log(age);
// Error!
```
Block-scoping is very useful for limiting how widespread variable declarations are in our programs, which helps pre-vent accidental overlap of their names.

But *var* is still useful in that it communicates “this variable will be seen by a wider scope”. Both declaration forms can be appropriate in any given part of a program, depending on the circumstances.

A third declaration form is *const*. It’s like *let* but has an additional limitation that it must be given a value at the moment it’s declared, and cannot be re-assigned a differentvalue later.

Consider:
```js
const myBirthday=true;
let age=39;
if(myBirthday) {
    age=age+1;// OK!
    myBirthday=false;// Error!
}
```
The *myBirthday* constant is not allowed to be re-assigned. *const* declared variables are not “unchangeable”, they just cannot be re-assigned.It’sill-advised to use *const* with object values, because those values can still be changed even though the variable can’t be re-assigned. This leads to potential confusion down the line,so I think it’s wise to avoid situations like:
```js
const actors=["Morgan Freeman","Jennifer Aniston"];
actors[2]="Tom Cruise";// OK :(
actors=[];// Error!
```
## Functions
In JS, we should consider “function” to take the broader meaning of another related term: “procedure.” A procedure is a collection of statements that can be invoked one or moretimes, may be provided some inputs, and may give back one or more outputs.

From the early days of JS, function definition looked like:

```js
function awesomeFunction(coolThings) {
    // ..
    return amazingStuff;
}
```

This is called a function declaration because it appears as a statement by itself,not as an expression in another statement.

In contrast to a function declaration statement, a function expression can be defined and assigned like this:
```js
// let awesomeFunction = ..
// const awesomeFunction = ..
var awesomeFunction = function(coolThings) {
    // ..
    return amazingStuff;
};
```
This function is an expression that is assigned to the variable *awesomeFunction*. Different from the function declaration form,a function expression is not associated with its identifier until that statement during runtime.

It’s extremely important to note that in JS, functions are values that can be assigned (as shown in this snippet) and passed around. In fact, JS functions are a special type of the object value type. Not all languages treat functions as values, but it’s essential for a language to support the functional programming pattern, as JS does.

## Comparisons

### Equal...ish
If you’ve spent any time working with and reading about JS, you’ve certainly seen the so-called “triple-equals”=== operator, also described as the “strict equality” operator. That seems rather straightforward, right? Surely, “strict” means strict, as in narrow and *exact*.

Not *exactly.* 

Yes, most values participating in an === equality comparison will fit with that *exact same* intuition. Consider some examples:

```js
3===3.0;// true
"yes"==="yes";// true
null===null;// true
false===false;// true
42==="42";// false
"hello"==="Hello";// false
true===1;// false
0===null;// false
""===null;// false
null===undefined;// false
```

The === operator is designed to *lie* in two cases of special values: NaN and -0.

Consider:
```js
NaN===NaN;// false
0===-0;// true
```

The story gets even more complicated when we consider comparisons of object values (non-primitives). Consider:

```js
[1,2,3]===[1,2,3];// false
{ a:42}==={ a:42}// false
(x => x*2)===(x => x*2)// false
```

JSdoesnotdefine===as *structural equality* for objectvalues.Instead,===uses *identity equality* for object values.

In JS, all object values are held by reference , are assigned and passed by reference-copy, **and** to our current discussion, are compared by reference (identity) equality. Consider:

```js
varx=[1,2,3];
// assignment is by reference-copy, so
// y references the *same* array as x,
// not another copy of it.
var y=x;
y===x;// true
y===[1,2,3];// false
x===[1,2,3];// false
```
### Coercive Comparisons
Coercion means a value of one type being converted to its respective representation in another type (to whatever extent possible). As we’ll discuss in Chapter 4,coercion is a core pillar of the JS language, not some optional feature that can reasonably be avoided.

The == operator performs an equality comparison similarly to how the === performs it. In fact, both operators consider the type of the values being compared.And if the comparison is between the same value type, both == and === **do exactlythe same thing, no difference whatsoever.**

If the value types being compared are different, the == differs from === in that it allows coercion before the comparison. In other words, they both want to compare values of like types,but == allows type conversions *first*, and once the types have been converted to be the same on both sides, then == does the same thing as === . Instead of “loose equality,” the == operator should be described as “coercive equality.”

Consider:
```js
42=="42";// true
1==true;// true
```

In both comparisons, the value types are different, so the == causes the non-number values ("42" and *true*) to be converted to numbers (42 and 1, respectively) before the comparisons are made.

There’s a pretty good chance that you’ll use relational comparison operators like<,>(and even <= and >=).

Just like ==, these operators will perform as if they’re “strict” if the types being relationally compared already match, but they’ll allow coercion first (generally,to numbers) if the types differ.

Consider:

```js
var arr=["1","10","100","1000"];
for(let i=0; i<arr.length && arr[i]<500; i++) {
    // will run 3 times
}
```
## How We Organize in JS

Two major patterns for organizing code (data and behavior)are used broadly across the JS ecosystem: classes and modules.These patterns are not mutually exclusive; many programs can and do use both. Other programs will stick with just one pattern, or even neither!

### Classes
A class in a program is a definition of a “type” of customdata structure that includes both data and behaviors that operate on that data.Classes define how such a data structure works, but classes are not themselves concrete values. To get a concrete value that you can use in the program,a class must be *instantiated*(with the *new* keyword) one or more times.

Behavior (methods) can only be called on instances (not the classes themselves

The *class* mechanism allows packaging data to be organized together with their behaviors.

#### Class Inheritance

Another aspect inherent to traditional “class-oriented”design,though a bit less commonly used in JS, is “inheritance” (and“polymorphism”).

The fact that both the inherited and overridden methods can have the same name and co-exist is called *polymorphism.*

Inheritance is a powerful tool for organizing data/behavior in separate logical units (classes), but allowing the child class to cooperate with the parent by accessing/using its behavior and data.

### Modules
The module pattern has essentially the same goal as the class pattern, which is to group data and behavior together into logical units. Also like classes, modules can “include”or “access” the data and behaviors of other modules, for cooperation sake.

But modules have some important differences from classes.Most notably, the syntax is entirely different.
#### Classic Modules
The key hallmarks of a *classic module* are an outer function(that runs at least once), which returns an “instance” of the module with one or more functions exposed that can operate on the module instance’s internal (hidden) data.

Because a module of this form is *just a function*, and calling it produces an “instance” of the module, another description for these functions is “module factories”.

The *class* form stores methods and data on an object instance, which must be accessed with the *this.*prefix. With modules, the methods and data are accessed as identifier variables in scope, without any *this.*prefix.

With *class*, the “API” of an instance is implicit in the class definition—also, all data and methods are public. With the module factory function, you explicitly create and return an object with any publicly exposed methods, and any data or other unreferenced methods remain private inside the factory function.

#### ES Modules
ES modules (ESM), introduced to the JS language in ES6,are meant to serve much the same spirit and purpose as the existing *classic modules* just described, especially taking into account important variations and use cases from AMD,UMD,and CommonJS.

The implementation approach does, however, differ significantly.

First, there’s no wrapping function to *define* a module. The wrapping context is a file. ESMs are always file-based; one file, one module.

Second, you don’t interact with a module’s “API” explicitly,but rather use the *export* keyword to add a variable or method to its public API definition. If something is defined in a module but not exported, then it stays hidden (just as with *classic modules*).

Third, and maybe most noticeably different from previously discussed patterns, you don’t “instantiate” an ES module, you just *import* it to use its single instance. ESMs are, in effect,“singletons,” in that there’s only one instance ever created,at first *import* in your program, and all other *imports* just receive a reference to that same single instance. If your module needs to support multiple instantiations, you have to provide a *classic module-style* factory function on your ESM definition for that purpose.

As shown, ES modules can use *classic modules* internally if they need to support multiple-instantiation. 