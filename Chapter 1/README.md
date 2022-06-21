JS is typically classified as an interpreted scripting language, so it’s assumed by most that JS programs are processed in a single, top-down pass. But JS is in fact parsed/compiled in a separate phase **before execution begins.**

JS functions are themselves first-class values; they can be assigned and passed around just like numbers or strings. But since these functions hold and access variables, they maintain their original scope no matter where in the program the functions are eventually executed. This is called closure.

Modules are a code organization pattern characterized by public methods that have privileged access (via closure) to hidden variables and functions in the internal scope of the module.

## Compiled vs. Interpreted
Code compilation is a set of steps that process the text of your code and turn it into a list of instructions the computer can understand.

You also may have heard that code can be interpreted, so how is that different from being compiled?

Interpretation performs a similar task to compilation, in that it transforms your program into machine-understandable instructions. But the processing model is different. Unlike a program being compiled all at once, with interpretation the source code is transformed line by line; each line or statement is executed before immediately proceeding to processing the next line of the source code.

 Our conclusion there is that JS is most accu- rately portrayed as a **compiled language.**
 
 ## Compiling Code

But first, why does it even matter whether JS is compiled or not?

Scope is primarily determined during compilation, so understanding how compilation and execution relate is key in mastering scope.

1. **Tokenizing/Lexing:** breaking up a string of characters into meaningful (to the language) chunks, called tokens. For instance, consider the program: var a = 2;. This program would likely be broken up into the following tokens: var, a, =, 2, and ;.

The difference between tokenizing and lexing is subtle and academic, but it centers on whether or not these tokens are identified in a stateless or stateful way. Put simply, if the tokenizer were to invoke stateful parsing rules to figure out whether a should be considered a distinct token or just part of another token, that would be **lexing.**)

2. **Parsing:** taking a stream (array) of tokens and turning it into a tree of nested elements, which collectively represent the grammatical structure of the program. This is called an Abstract Syntax Tree (AST).

For example, the tree for var a = 2; might start with a top-level node called *VariableDeclaration*, with a child node called *Identifier* (whose value is *a*), and another child called *AssignmentExpression* which it- self has a child called *NumericLiteral* (whose value is 2).

3. **Code Generation:** taking an AST and turning it into ex- ecutable code. This part varies greatly depending on the language, the platform it’s targeting, and other factors.

The JS engine takes the just described AST for var a = 2; and turns it into a set of machine instructions to actually create a variable called a (including reserving memory, etc.), and then store a value into a.

The JS engine is vastly more complex than *just* these three stages

 In the process of parsing and code generation, there are steps to optimize the performance of the execution (i.e., collapsing redundant elements). In fact, code can even be re- compiled and re-optimized during the progression of execu- tion.

 JS engines don’t have the luxury of an abundance of time to perform their work and optimizations, because JS compilation doesn’t happen in a build step ahead of time, as with other languages. It usually must happen in mere microseconds (or less!) right before the code is executed. To ensure the fastest performance under these constraints, JS engines use all kinds of tricks (like JITs, which lazy compile and even hot re- compile)

 ## Required: Two Phases
 
 To state it as simply as possible, the most important observation we can make about processing of JS programs is that it occurs in (at least) two phases: parsing/compilation first, then execution.

 The separation of a parsing/compilation phase from the subsequent execution phase is observable fact, not theory or opinion. 

 There are three program characteristics you can observe to prove this to yourself: syntax errors, early errors, and hoisting.

## Syntax Errors from the Start

```Js
var greeting = "Hello"; 
console.log(greeting);
greeting = ."Hi";
// SyntaxError: unexpected token .
```

This program produces no output (*"Hello"* is not printed), but instead throws a *SyntaxError* about the unexpected . token right before the *"Hi"* string. Since the syntax error happens after the well-formed *console.log(..)* statement, if JS was executing top-down line by line, one would expect the *"Hello"* message being printed before the syntax error being thrown. That doesn’t happen.

## Early Errors

```Js
console.log("Howdy");
saySomething("Hello","Hi");
// Uncaught SyntaxError: Duplicate parameter name not
// allowed in this context
function saySomething(greeting,greeting) { 
    "use strict";
    console.log(greeting);
}
```

The *"Howdy"* message is not printed, despite being a wellformed statement.

Instead, just like the snippet in the previous section, the *SyntaxError* here is thrown before the program is executed. In this case, it’s because strict-mode (opted in for only the *saySomething(..)* function here) forbids, among many other things, functions to have duplicate parameter names; this has always been allowed in non-strict-mode.

The error thrown is not a syntax error in the sense of be- ing a malformed string of tokens (like .*"Hi"* prior), but in strict-mode is nonetheless required by the specification to be thrown as an “early error” before any execution begins.

## Hoisting

```Js
function saySomething() { 
    var greeting = "Hello"; 
    {
        greeting = "Howdy"; // error comes from here 
        let greeting = "Hi";
        console.log(greeting);
    }
}
saySomething();
// ReferenceError: Cannot access 'greeting' before
// initialization
```

The noted *ReferenceError* occurs from the line with the statement *greeting = "Howdy"*. What’s happening is that the greeting variable for that statement belongs to the declaration on the next line, *let greeting = "Hi"*, rather than to the previous *var greeting = "Hello"* statement.

The only way the JS engine could know, at the line where the error is thrown, that the *next statement* would declare a block-scoped variable of the same name (*greeting*) is if the JS engine had already processed this code in an earlier pass, and already set up all the scopes and their variable associations. This processing of scopes and declarations can only accurately be accomplished by parsing the program before execution.

The *ReferenceError* here technically comes from *greeting = "Howdy"* accessing the greeting variable **too early**, a conflict referred to as the Temporal Dead Zone (TDZ). 

This is an interesting question to ponder. Could JS parse a program, but then execute that program by *interpreting* operations represented in the AST **without** first compiling the program? Yes, that is *possible.* But it’s extremely unlikely, mostly because it would be extremely inefficient performance wise.

## Compiler Speak

Other than declarations, all occurrences of variables/identi- fiers in a program serve in one of two “roles”: either they’re the *target* of an assignment or they’re the *source* of a value.

How do you know if a variable is a *target*? Check if there is a value that is being assigned to it; if so, it’s a *target*. If not, then the variable is a *source.*

## Targets

```Js
var students = [
    { id: 14, name: "Kyle" }, 
    { id: 73, name: "Suzy" }, 
    { id: 112, name: "Frank" }, 
    { id: 6, name: "Sarah" }
];

function getStudentName(studentID) { 
    for (let student of students) {
        if (student.id == studentID) { 
            return student.name;
        } 
    }
}
var nextStudent = getStudentName(73); 
console.log(nextStudent);
// Suzy
```
There are three other *target* assignment operations in the code that are perhaps less obvious. One of them:
```Js
for (let student of students) {
```
That statement assigns a value to *student* for each iteration of the loop. Another *target* reference:
```Js
getStudentName(73)
```
But how is that an assignment to a *target*? Look closely: the argument 73 is assigned to the parameter *studentID*.
```Js
function getStudentName(studentID) {
```
A *function* declaration is a special case of a target reference. You can think of it sort of like *var getStudentName = function(studentID)*, but that’s not exactly accurate. An identifier *getStudentName* is declared (at compile time),but the *= function(studentID)* part is also handled at compilation; the association between *getStudentName* and the function is automatically set up at the beginning of the scope rather than waiting for an = assignment statement to be executed.
### Sources
So we’ve identified all five *target* references in the program. The other variable references must then be *source* references (because that’s the only other option!).

In *for (let student of students)*, we said that *student* is a *target*, but students is a *source* reference. In the statement *if (student.id == studentID)*, both *student* and *studentID* are *source* references. *student* is also a *source* reference in *return student.name*.

In *getStudentName(73)*, *getStudentName* is a *source* reference (which we hope resolves to a function reference value). In *console.log(nextStudent)*, console is a *source* reference, as is *nextStudent*.
## Cheating: Runtime Scope Modifications
It should be clear by now that scope is determined as the program is compiled, and should not generally be affected by runtime conditions. However, in non-strict-mode, there are technically still two ways to cheat this rule, modifying a program’s scopes during runtime.

Neither of these techniques *should* be used—they’re both dangerous and confusing, and you should be using strict- mode (where they’re disallowed) anyway. 

```Js
function badIdea() {
    eval("var oops = 'Ugh!';"); 
    console.log(oops);
}
badIdea();   // Ugh!
```
If the *eval(..)* had not been present, the *oops* variable in *console.log(oops)* would not exist, and would throw a *ReferenceError*. But *eval(..)* modifies the scope of the *badIdea()* function at runtime. This is bad for many rea- sons, including the performance hit of modifying the already compiled and optimized scope, every time *badIdea()* runs.

The second cheat is the *with* keyword, which essentially dynamically turns an object into a local scope—its properties are treated as identifiers in that new scope’s block:
```Js
var badIdea = { oops: "Ugh!" };
with (badIdea) { 
    console.log(oops); // Ugh!
}
```

The global scope was not modified here, but *badIdea* was turned into a scope at runtime rather than compile time, and its property oops becomes a variable in that scope. Again, this is a terrible idea, for performance and readability reasons.

At all costs, avoid *eval(..)* (at least, *eval(..)* creating declarations) and *with*. Again, neither of these cheats is available in strict-mode, so if you just use strict-mode (you should!) then the temptation goes away!
## Lexical Scope
We’ve demonstrated that JS’s scope is determined at compile time; the term for this kind of scope is “lexical scope”. “Lexical” is associated with the “lexing” stage of compilation, as discussed earlier in this chapter.

To narrow this chapter down to a useful conclusion, the key idea of “lexical scope” is that it’s controlled entirely by the placement of functions, blocks, and variable declarations, in relation to one another.

If you place a variable declaration inside a function, the compiler handles this declaration as it’s parsing the function, and associates that declaration with the function’s scope. If a variable is block-scope declared (*let / const*), then it’s associated with the nearest enclosing { .. } block, rather than its enclosing function (as with *var*).

It’s important to note that compilation doesn’t actually do *anything* in terms of reserving memory for scopes and variables. None of the program has been executed yet.

Instead, compilation creates a map of all the lexical scopes that lays out what the program will need while it executes. You can think of this plan as inserted code for use at runtime, which defines all the scopes (aka, “lexical environments”) and registers all the identifiers (variables) for each scope.