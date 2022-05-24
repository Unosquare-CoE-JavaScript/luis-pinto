## What’s With That Name?
The truth is, the name JavaScript is an artifact of marketing shenanigans. When Brendan Eich first conceived of the lan-guage, he code-named it Mocha. Internally at Netscape, the brand LiveScript was used. But when it came time to publiclyname the language, “JavaScript” won the vote.

And indeed, since 2016, the official language name has also been suffixed by the revision year; as of this writing, that’s ECMAScript 2019, or otherwise abbreviated ES2019.

## Language Specification
JS’s syntax and behavior are defined in the ES specification.

The TC39 committee is comprised of between 50 and about 100 different people from a broad section of web-invested companies, such as browser makers (Mozilla, Google, Apple)and device makers (Samsung, etc)

All TC39 proposals progress through a five-stage process—of course, since we’re programmers, it’s 0-based!—Stage 0 through Stage 4. You can read more about the Stage process here:https://tc39.es/process-document/

## The Web Rules Everything About (JS)
For the most part, the JS defined in the specification and the JS that runs in browser-based JS engines is the same.But there are some differences that must be considered.

Sometimes the JS specification will dictate some new orrefined behavior, and yet that won’t exactly match with how it works in browser-based JS engines. 

Appendix B, “Additional ECMAScript Features for Web Browsers”.

Section B.1 and B.2 cover additions to JS (syntax and APIs)that web JS includes, again for historical reasons, but which TC39 does not plan to formally specify in the core of JS.

Section B.3 includes some conflicts where code may run in both web and non-web JS engines, but where the behavior could be observably different,resulting in different outcomes.

## Not All (Web) JS...
Various JS environments (like browser JS engines, Node.js,etc.) add APIs into the global scope of your JS programs that give you environment-specific capabilities

## It’s Not Always JS
Using the console/REPL (Read-Evaluate-Print-Loop) in your browser’s Developer Tools (or Node) feels like a pretty straight-forward JS environment at first glance. But it’s not, really.

But I’ll just hint at some examples of quirks that have been true at various points in different JS console environments,to reinforce my point about not assuming native JS behavior while using them:
- Whether a *var* or *function* declaration in the top-level “global scope” of the console actually creates a real global variable (and mirrored window property, and viceversa!).
- What happens with multiple *let* and *const* declarations in the top-level “global scope.”
- Whether "*use strict*";on one line-entry (pressing *< enter >* after) enables strict mode for the rest of that console session, the way it would on the first line of a .js file, as well as whether you can use "*use strict*"; beyond the “firstline” and still get strict mode turned on for that session.
- How non-strict mode *this* default-binding works for function calls, and whether the “global object” used will contain expected global variables.
- How hoisting (see Book 2,Scope & Closures) works across multiple line entries.
- ...several others

## Many Faces
Typical paradigm-level code categories include procedural,object-oriented (OO/classes), and functional (FP):
- Procedural style organizes code in a top-down, linear progression through a pre-determined set of operations,usually collected together in related units called proce-dures.
- OO style organizes code by collecting logic and data together into units called classes.
- FP style organizes code into functions (pure computations as opposed to procedures), and the adaptations of those functions as values.

JavaScript is most definitely a multi-paradigm language. You can write procedural,class-oriented,or FP-style code,and you can make those decisions on a line-by-line basis instead of being forced into an all-or-nothing choice.

## Backwards & Forwards
Backwards compatibility means that once something is ac-cepted as valid JS, there will not be a future change to the language that causes that code to become invalid JS.

The idea is that JS developers can write code with confidence that their code won’t stop working unpredictably because abrowser update is released. This makes the decision to chooseJS for a program a more wise and safe investment, for yearsinto the future.

Being forwards-compatible means that including a new addition to the language in a program would not cause that program to break if it were run in an older JS engine. **JS is not forwards-compatible**, despite many wishing such, and even incorrectly believing the myth that it is.

HTML and CSS,by contrast,are forwards-compatible but not backwards-compatible. If you dug up some HTML or CSS written back in 1995, it’s entirely possible it would not work(or work the same) today. But, if you use a new feature from 2019 in a browser from 2010, the page isn’t “broken”

## Jumping the Gaps
Since JS is not forwards-compatible, it means that there isalways the potential for a gap between code that you canwrite that’s valid JS, and the oldest engine that your site or application needs to support. If you run a program that uses an ES2019 feature in an engine from 2016, you’re very likely to see the program break and crash.

For new and incompatible syntax, the solution is transpiling.Transpiling is a contrived and community-invented term to describe using a tool to convert the source code of a program from one form to another (but still as textual source code).Typically, forwards-compatibility problems related to synta xare solved by using a transpiler (the most common one being Babel (https://babeljs.io)) to convert from that newerJS syntax version to an equivalent older syntax.

## Filling the Gaps

If the forwards-compatibility issue is not related to newsyntax, but rather to a missing API method that was only recently added, the most common solution is to provide a definition for that missing API method that stands in and acts as if the older environment had already had it natively defined. This pattern is called a polyfill (aka “shim”).

Consider this code:
```jsx
// function that will return a promise
const pr = getSomeRecords();

// show a spinner in th ui while data is fetched
startSpinner()

pr
	.then(renderRecords) // render if successfull
	.catch(showError) // show an error if not
	.finally(hideSpinner) // always hide spinner
```

This code uses an ES2019 feature, thef inally(..)method on the promise prototype. If this code were used in a pre-ES2019 environment, the finally(..)method would not exist, and an error would occur.
A polyfill for finally(..)in pre-ES2019 environments could look like this:

```Jsx
if(!Promise.prototype.finally) {
    Promise.prototype.finally = function f(fn){
        return this.then(function t(v){
            return Promise.resolve( fn() )
            .then(function t(){
                return v;});
                },function c(e){
                    return Promise.resolve( fn() )
                    .then(function t(){
                        throw e;
            });
        });
    };
}
```

Transpilers like Babel typically detect which polyfills your code needs and provide them automatically for you. But occasionally you may need to include/define them explicitly,which works similar to the snippet we just looked at.

Always write code using the most appropriate features to communicate its ideas and intent effectively. In general, this means using the most recent stable JS version. Avoid nega-tively impacting the code’s readability by trying to manually adjust for the syntax/API gaps. That’s what tools are for!

## What’s in an Interpretation?
Languages regarded as“compiled usually produce a portable(binary) representation of the program that is distributed for execution later.

Historically, scripted or interpreted languages were executedin generally a top-down and line-by-line fashion; there’stypically not an initial pass through the program to processit before execution begins 

In scripted or interpreted languages, an error on line 5 of aprogram won’t be discovered until lines 1 through 4 havealready executed.

Compare that to languages which do go through a processing step(typically,called parsing)before any execution occurs

In this processing model,an invalid command(such as broken syntax) on line 5 would be caught during the parsing phase,before any execution has begun, and none of the program would run. 

So what do “parsed” languages have in common with “com-piled” languages? First, all compiled languages are parsed.So a parsed language is quite a ways down the road toward being compiled already. In classic compilation theory, the last remaining step after parsing is code generation: producing an executable form.

Once any source program has been fully parsed, it’s very common that its subsequent execution will, in some form or fashion, include a translation from the parsed form of the program—usually called an Abstract Syntax Tree (AST)—to that executable form.

In other words, parsed languages usually also perform code generation before execution, so it’s not that much of a stretch to say that, in spirit, they’re compiled languages.

So **JS is a parsed language**, but is itcompiled?

The parsed JS is converted to an optimized (binary) form, and that “code” is subse-quently executed; the engine does not commonly switch back into line-by-line execution mode after it has finished all the hard work of parsing—most languages/engines wouldn’t, because that would be highly inefficient.

To be specific, this “compilation” produces a binary byte code(of sorts), which is then handed to the “JS virtual machine”to execute.

Step backand consider the entire flow of a JS source program:
1. After a program leaves a developer’s editor, it gets tran-spiled by Babel, then packed by Webpack (and perhaps half a dozen other build processes),then it gets delivered in that very different form to a JS engine.
2. The JS engine parses the code to an AST.
3. Then the engine converts that AST to a kind-of bytecode, a binary intermediate representation (IR), which is then refined/converted even further by the optimizing JIT  compiler.
4. Finally, the JS VM executes the program.

## Web Assembly (WASM)

In 2013, engineers from Mozilla Firefox demonstrated a port of the Unreal 3 game engine from C to JS. The ability for this code to run in a browser JS engine at full 60fps performance was predicated on a set of optimizations that the JS engine could perform specifically because the JS version of the Unreal engine’s code used a style of code that favored a subset of the JS language, named “ASM.js”.

Several years after ASM.js demonstrated the validity of tool-ing-created versions of programs that can be processed more efficiently by the JS engine, another group of engineers (also,initially, from Mozilla) released Web Assembly (WASM).

WASM is similar to ASM.js in that its original intent was to provide a path for non-JS programs(C,etc.) to be converted to a form that could run in the JS engine. Unlike ASM.js, WASM chose to additionally get around some of the inherent delays in JS parsing/compilation before a program can execute, by representing the program in a form that is entirely unlike JS.

WASM is a representation format more akin to Assembly(hence, its name) that can be processed by a JS engine byskipping the parsing/compilation that the JS engine normallydoes. The parsing/compilation of a WASM-targeted programhappen ahead of time (AOT); what’s distributed is a binary-packed program ready for the JS engine to execute with veryminimal processing.

In other words, WASM relieves the pressure to add featuresto JS that are mostly/exclusively intended to be used bytranspiled programs from other languages.

*Strictly* Speaking

Back in 2009 with the release of ES5, JS added *strict mode* as an opt-in mechanism for encouraging better JS programs.

Why strict mode? Strict mode shouldn’t be thought of as a restriction on what you can’t do, but rather as a guide to the best way to do things so that the JS engine has the best chance of optimizing and efficiently running the code. 

Most strict mode controls are in the form of *early errors*,meaning errors that aren’t strictly syntax errors but are still thrown at compile time (before the code is run). 

The best mindset is that strict mode is like a linterreminding you how JSshouldbe written to have the highestquality and best chance at performance. 

Strict mode is switched on per file with a special pragma(nothing allowed before it except comments/whitespace):

```jsx
// only whitespace and comments are allowed
// before the use-strict pragma
"use strict";
// the rest of the file runs in strict mode
```
Strict mode can alternatively be turned on per-function scope,with exactly the same rules about its surroundings:
```js
function someOperations() {
    // whitespace and comments are fine here
    "use strict";
    // all this code will run in strict mode
}
```
The **only** valid reason to use a per-function approach to strict mode is when you are converting an existing non-strict mode program file and need to make the changes little by little overtime. Otherwise, it’s vastly better to simply turn strict mode on for the entire file/program.

ES6 modules assume strict mode, so all code in such files is automatically defaulted to strict mode.

## Defined

JS is an implementation of the ECMAScript standard (version ES2019 as of this writing), which is guided by the TC39 committee and hosted by ECMA. It runs in browsers and other JS environments such as Node.js.

JS is a multi-paradigm language, meaning the syntax and capabilities allow a developer to mix and match (and bend and reshape!)concepts from various major paradigms,such as procedural,object-oriented(OO/classes),and functional(FP).

JS is a compiled language, meaning the tools (including the JSengine) process and verify a program (reporting any errors!)before it executes.