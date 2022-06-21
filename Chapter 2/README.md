In Chapter 1, we explored how scope is determined during code compilation, a model called “lexical scope.” The term “lexical” refers to the first stage of compilation (lexing/parsing).

This chapter will illustrate *scope* with several metaphors. The goal here is to *think* about how your program is handled by the JS engine in ways that more closely align with how the JS engine actually works.

## Marbles, and Buckets, and Bubbles... Oh My!
Imagine you come across a pile of marbles, and notice that all the marbles are colored red, blue, or green. Let’s sort all the marbles, dropping the red ones into a red bucket, green into a green bucket, and blue into a blue bucket. After sorting, when you later need a green marble, you already know the green bucket is where to go to get it.

In this metaphor, the marbles are the variables in our pro- gram. The buckets are scopes (functions and blocks), which we just conceptually assign individual colors for our discussion purposes. The color of each marble is thus determined by which *color* scope we find the marble originally created in.
```Js
// outer/global scope: RED
var students = [
    { id: 14, name: "Kyle" }, 
    { id: 73, name: "Suzy" }, 
    { id: 112, name: "Frank" }, 
    { id: 6, name: "Sarah" }
];

function getStudentName(studentID) {
    // function scope: BLUE
    for (let student of students) { 
        // loop scope: GREEN
        if (student.id == studentID) { 
            return student.name;
        } 
    }
}
var nextStudent = getStudentName(73); 
console.log(nextStudent); // Suzy
```
We’ve designated three scope colors with code comments: RED (outermost global scope), BLUE (scope of function get- StudentName(..)), and GREEN (scope of/inside the for loop). But it still may be difficult to recognize the boundaries of these scope buckets when looking at a code listing.

The key take-aways from marbles & buckets (and bubbles!):

- Variables are declared in specific scopes, which can be thought of as colored marbles from matching-color buckets.
- Any variable reference that appears in the scope where it was declared, or appears in any deeper nested scopes, will be labeled a marble of that same color—unless an intervening scope “shadows” the variable declaration; see “Shadowing” in Chapter 3.
- The determination of colored buckets, and the marbles they contain, happens during compilation. This information is used for variable (marble color) “lookups” during code execution.

## A Conversation Among Friends
Let’s now meet the members of the JS engine that will have conversations as they process our program:
- *Engine*: responsible for start-to-finish compilation and execution of our JavaScript program.
- *Compiler*: one of Engine’s friends; handles all the dirty work of parsing and code-generation (see previous sec- tion).
- *Scope Manager*: another friend of Engine; collects and maintains a lookup list of all the declared variables/i- dentifiers, and enforces a set of rules as to how these are accessible to currently executing code.
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
Our focus here will be on the var students = [ .. ] declaration and initialization- assignment parts.

We typically think of that as a single statement, but that’s not how our friend *Engine* sees it. In fact, JS treats these as two distinct operations, one which *Compiler* will handle during compilation, and the other which *Engine* will handle during execution.

The first thing *Compiler* will do with this program is perform lexing to break it down into tokens, which it will then parse into a tree (AST).

Once *Compiler* gets to code generation, there’s more detail to consider than may be obvious. A reasonable assumption would be that *Compiler* will produce code for the first statement such as: “Allocate memory for a variable, label it *students*, then stick a reference to the array into that variable.” But that’s not the whole story.

Here’s the steps Compiler will follow to handle that state- ment:
1. Encountering var *students*, *Compiler* will ask *Scope Manager* to see if a variable named *students* already exists for that particular scope bucket. If so, *Compiler* would ignore this declaration and move on. Otherwise, *Compiler* will produce code that (at execution time) asks *Scope Manager* to create a new variable called *students* in that scope bucket.
2. *Compiler* then produces code for *Engine* to later execute, to handle the *students = []* assignment. The code *Engine* runs will first ask *Scope Manager* if there is a variable called *students* accessible in the current scope bucket. If not, *Engine* keeps looking elsewhere (see “Nested Scope” below). Once Engine finds a variable, it assigns the reference of the [ .. ] array to it.

## Nested Scope
The function scope for *getStudentName(..)* is nested inside the global scope. The block scope of the *for-*loop is similarly nested inside that function scope. Scopes can be lexically nested to any arbitrary depth as the program defines.

Each scope gets its own *Scope Manager* instance each time that scope is executed (one or more times). Each scope automatically has all its identifiers registered at the start of the scope being executed (this is called “variable hoisting”; see Chapter 5).

At the beginning of a scope, if any identifier came from a *function* declaration, that variable is automatically initial- ized to its associated function reference. And if any identifier came from a var declaration (as opposed to *let/const*), that variable is automatically initialized to *undefined* so that it can be used; otherwise, the variable remains uninitialized (aka, in its “TDZ,” see Chapter 5) and cannot be used until its full declaration-and-initialization are executed.

## Lookup Failures
When *Engine* exhausts all *lexically available* scopes (moving outward) and still cannot resolve the lookup of an identifier, an error condition then exists. However, depending on the mode of the program (strict-mode or not) and the role of the variable (i.e., *target* vs. *source*; see Chapter 1), this error condition will be handled differently.
## Undefined Mess
If the variable is a *source*, an unresolved identifier lookup is considered an undeclared (unknown, missing) variable, which always results in a *ReferenceError* being thrown. Also, if the variable is a *target*, and the code at that moment is running in strict-mode, the variable is considered undeclared and similarly throws a *ReferenceError*.

The error message for an undeclared variable condition, in most JS environments, will look like, “Reference Error: XYZ is not defined.” The phrase “not defined” seems almost identical to the word “undefined,” as far as the English language goes. But these two are very different in JS, and this error message unfortunately creates a persistent confusion.

“Not defined” really means “not declared”—or, rather, “unde- clared,” as in a variable that has no matching formal declara- tion in any *lexically available* scope. By contrast, “undefined” really means a variable was found (declared), but the variable otherwise has no other value in it at the moment, so it defaults to the undefined value.

To perpetuate the confusion even further, JS’s *typeof* opera- tor returns the string *"undefined"* for variable references in either state:
```Js
var studentName;
typeof studentName; // "undefined"
typeof doesntExist; // "undefined"
```
These two variable references are in very different conditions,
but JS sure does muddy the waters. The terminology mess is confusing and terribly unfortunate.
## Global... What!?
If the variable is a *target* and strict-mode is not in effect, a confusing and surprising legacy behavior kicks in. The troublesome outcome is that the global scope’s *Scope Manager* will just create an **accidental global variable** to fulfill that target assignment!

```Js
function getStudentName() {
    // assignment to an undeclared variable :( 
    nextStudent = "Suzy";
}
getStudentName();
console.log(nextStudent);
// "Suzy" -- oops, an accidental-global variable!
```
## Building On Metaphors

To visualize nested scope resolution, I prefer yet another metaphor, an office building

The building represents our program’s nested scope collec- tion. The first floor of the building represents the currently executing scope. The top level of the building is the global scope.

You resolve a *target* or *source* variable reference by first looking on the current floor, and if you don’t find it, taking the elevator to the next floor (i.e., an outer scope), looking there, then the next, and so on. Once you get to the top floor (the global scope), you either find what you’re looking for, or you don’t. But you have to stop regardless.

## Continue the Conversation
By this point, you should be developing richer mental models for what scope is and how the JS engine determines and uses it from your code.

Before *continuing*, go find some code in one of your projects and run through these conversations. Seriously, actually speak out loud. Find a friend and practice each role with them. If either of you find yourself confused or tripped up, spend more time reviewing this material.
