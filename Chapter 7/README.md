Closure is one of the most important language characteristics ever invented in programming—it underlies major programming paradigms, including Functional Programming (FP), modules, and even a bit of class-oriented design. Getting comfortable with closure is required for mastering JS and effectively leveraging many important design patterns throughout your code.
## See the Closure
```Js
// outer/global scope: RED(1)
function lookupStudent(studentID) { 
    // function scope: BLUE(2)
    var students = [
    { id: 14, name: "Kyle" }, 
    { id: 73, name: "Suzy" }, 
    { id: 112, name: "Frank" }, 
    { id: 6, name: "Sarah" }
    ];
    
    return function greetStudent(greeting){
        // function scope: GREEN(3)
        var student = students.find(
        student => student.id == studentID
    );
    
    return `${ greeting }, ${ student.name }!`; };
}
var chosenStudents = [ 
    lookupStudent(6), 
    lookupStudent(112)
];
// accessing the function's name:
chosenStudents[0].name;
// greetStudent
chosenStudents[0]("Hello");
// Hello, Sarah!
chosenStudents[1]("Howdy");
// Howdy, Frank!
```
The first thing to notice about this code is that the *lookupStudent(..)* outer function creates and returns an inner function called *greetStudent(..)*. *lookupStudent(..)* is called twice, producing two separate instances of its inner *greetStudent(..)* function, both of which are saved into the *chosenStudents* array.
## Pointed Closure
Building on the metaphor of colored buckets and bubbles from Chapter 2, if we were creating a colored diagram for this code, there’s a fourth scope at this innermost nesting level, so we’d need a fourth color; perhaps we’d pick ORANGE(4) for that scope:
```Js
var student = students.find( 
    student =>
        // function scope: ORANGE(4)
        student.id == studentID
);
```
The BLUE(2) *studentID* reference is actually inside the OR- ANGE(4) scope rather than the GREEN(3) scope of *greetStudent(..)*; also, the *student* parameter of the arrow function is ORANGE(4), shadowing the GREEN(3) *student*.

The consequence here is that this arrow function passed as a callback to the array’s *find(..)* method has to hold the closure over *studentID*, rather than *greetStudent(..)* holding that closure. That’s not too big of a deal, as everything still works as expected. It’s just important not to skip over the fact that even tiny arrow functions can get in on the closure party.
## Adding Up Closures
Let’s examine one of the canonical examples often cited for closure:
```Js
function adder(num1) {
    return function addTo(num2){
        return num1 + num2; 
    };
}

var add10To = adder(10);
var add42To = adder(42);
 
add10To(15); // 25
add42To(9);     // 51
```
let’s reinforce it: closure is associated with an instance of a function, rather than its single lexical definition. In the preceding snippet, there’s just one inner *addTo(..)* function defined inside *adder(..)*, so it might seem like that would imply a single closure.

But actually, every time the outer *adder(..)* function runs, a *new* inner *addTo(..)* function instance is created, and for each new instance, a new closure. So each inner function instance (labeled *add10To(..)* and *add42To(..)* in our program) has its own closure over its own instance of the scope environment from that execution of *adder(..)*.
## Live Link, Not a Snapshot
Closure is actually a live link, preserving access to the full variable itself. We’re not limited to merely reading a value; the closed-over variable can be updated (re-assigned) as well! By closing over a variable in a function, we can keep using that variable (read and write) as long as that function refer- ence exists in the program, and from anywhere we want to invoke that function. This is why closure is such a powerful technique used widely across so many areas of programming!
```Js
function makeCounter() { 
    var count = 0;
    return getCurrent(){ 
        count = count + 1; 
        return count;
    }; 
}
var hits = makeCounter(); 
// later
hits(); // 1
// later
hits();     // 2
hits();     // 3
```
The *count* variable is closed over by the inner *getCurrent()* function, which keeps it around instead of it being subjected to GC. The *hits()* function calls access *and* update this variable, returning an incrementing count each time.

Though the enclosing scope of a closure is typically from a function, that’s not actually required; there only needs to be an inner function present inside an outer scope:

```Js
var hits;
{ // an outer scope (but not a function)
    let count = 0;
    hits = function getCurrent(){
        count = count + 1;
        return count; 
    };
}
hits(); //1
hits(); //2
hits(); //3
```
Because it’s so common to mistake closure as value-oriented instead of variable-oriented, developers sometimes get tripped up trying to use closure to snapshot-preserve a value from some moment in time. 
```Js
var keeps = [];
for (var i = 0; i < 3; i++) { 
    keeps[i] = function keepI(){
        // closure over `i`
        return i; 
    
    };
}
keeps[0](); // 3 -- WHY!?
keeps[1](); // 3
keeps[2](); // 3

```
Each saved function returns 3, because by the end of the loop, the single *i* variable in the program has been assigned 3. Each of the three functions in the *keeps* array do have individual closures, but they’re all closed over that same shared *i* variable.

Recall the “Loops” section in Chapter 5, which illustrates how a *let* declaration in a *for* loop actually creates not just one variable for the loop, but actually creates a new variable for *each iteration* of the loop. That trick/quirk is exactly what we need for our loop closures:
```Js
var keeps = [];
for (let i = 0; i < 3; i++) {
    // the `let i` gives us a new `i` for 
    // each iteration, automatically! 
    keeps[i] = function keepEachI(){
        return i; 
    };
}
keeps[0](); // 0 
keeps[1](); // 1 
keeps[2](); // 2

```
Since we’re using *let*, three *i*’s are created, one for each loop, so each of the three closures *just work* as expected.
## Common Closures: Ajax and Events
Closure is most commonly encountered with callbacks:
```Js
function lookupStudentRecord(studentID) { 
    ajax(
        `https://some.api/student/${ studentID }`, 
        function onRecord(record) {
            console.log(`${ record.name } (${ studentID })`); 
        }
        );
}
```
The *onRecord(..)* callback is going to be invoked at some point in the future, after the response from the Ajax call comes back. This invocation will happen from the internals of the *ajax(..)*  utility, wherever that comes from. Furthermore, when that happens, the *lookupStudentRecord(..)* call will long since have completed.

Event handlers are another common usage of closure:
```Js
function listenForClicks(btn,label) { 
    btn.addEventListener("click",function onClick(){
        console.log(`The ${ label } button was clicked!`); 
    });
}
var submitBtn = document.getElementById("submit-btn"); listenForClicks(submitBtn,"Checkout");
```
The *label* parameter is closed over by the *onClick(..)* event handler callback. When the button is clicked, *label* still exists to be used. This is closure.
## What If I Can’t See It?
If a closure exists (in a technical, implementation, or academic sense) but it cannot be observed in our programs, *does it matter?* No.

To reinforce this point, let’s look at some examples that are *not* observably based on closure.
```Js
function say(myName) {
    var greeting = "Hello"; 
    output();
    
    function output() { 
        console.log(`${ greeting }, ${ myName }!` );
     } 
}

say("Kyle");
// Hello, Kyle!
```
The inner function *output()* accesses the variables *greeting* and *myName* from its enclosing scope. But the invocation of *output()* happens in that same scope, where of course *greeting* and *myName* are still available; that’s just lexical scope, not closure.

In fact, global scope variables essentially cannot be (observably) closed over, because they’re always accessible from everywhere. No function can ever be invoked in any part of the scope chain that is not a descendant of the global scope.
## Observable Definition
We’re now ready to define closure:
> Closure is observed when a function uses variable(s) from outer scope(s) even while running in a scope where those variable(s) wouldn’t be accessible.

The key parts of this definition are:

- Must be a function involved
- Must reference at least one variable from an outer scope
- Must be invoked in a different branch of the scope chain
from the variable(s)
## The Closure Lifecycle and Garbage Collection (GC)
If ten functions all close over the same variable, and over time nine of these function references are discarded, the lone remaining function reference still preserves that variable. Once that final function reference is discarded, the last closure over that variable is gone, and the variable itself is GC’d.

This has an important impact on building efficient and performant programs. Closure can unexpectedly prevent the GC of a variable that you’re otherwise done with, which leads to run-away memory usage over time. That’s why it’s important to discard function references (and thus their closures) when they’re not needed anymore.
```Js
function manageBtnClickEvents(btn) { 
    var clickHandlers = [];
    return function listener(cb){ 
        if (cb) {
            let clickHandler = function onClick(evt){
                console.log("clicked!");
                cb(evt); 
            };
            clickHandlers.push(clickHandler);
            btn.addEventListener(
                "click",
                clickHandler
            );
        }
        else {
            // passing no callback unsubscribes 
            // all click handlers
            for (let handler of clickHandlers) {
                btn.removeEventListener(
                    "click",
                    handler
                );
            }
            clickHandlers = [];
        }
    };
}
// var mySubmitBtn = ..
var onSubmit = manageBtnClickEvents(mySubmitBtn);

onSubmit(function checkout(evt){ 
    // handle checkout
});
onSubmit(function trackAction(evt){ 
    // log action to analytics
});
// later, unsubscribe all handlers:
onSubmit();
```
In this program, the inner *onClick(..)* function holds a closure over the passed in cb (the provided event callback). That means the *checkout()* and *trackAction()* function expression references are held via closure (and cannot be GC’d) for as long as these event handlers are subscribed.

When we call *onSubmit()* with no input on the last line, all event handlers are unsubscribed, and the *clickHandlers* array is emptied. Once all click handler function references are discarded, the closures of *cb* references to *checkout()* and *trackAction()* are discarded.
## Per Variable or Per Scope?
Conceptually, closure is **per variable** rather than *per scope*. Ajax callbacks, event handlers, and all other forms of function closures are typically assumed to close over only what they explicitly reference.

But the reality is more complicated than that. 

Another program to consider:
```Js
function manageStudentGrades(studentRecords) { 
    var grades = studentRecords.map(getGrade);
    return addGrade;
    // ************************
    function getGrade(record){ 
        return record.grade;
    }
    function sortAndTrimGradesList() { 
        // sort by grades, descending 
        grades.sort(function desc(g1,g2){
            return g2 - g1; 
        });
        // only keep the top 10 grades
        grades = grades.slice(0,10);
    }
    function addGrade(newGrade) { 
        grades.push(newGrade); 
        sortAndTrimGradesList(); 
        return grades;
    } 
}

var addNextGrade = manageStudentGrades([ 
    { id: 14, name: "Kyle", grade: 86 },
    { id: 73, name: "Suzy", grade: 87 },
    { id: 112, name: "Frank", grade: 75 },
    // ..many more records..
    { id: 6, name: "Sarah", grade: 91 }
]);
// later
addNextGrade(81);
addNextGrade(68);
// [ .., .., ... ]
```
The outer function *manageStudentGrades(..)* takes a list of student records, and returns an *addGrade(..)* function reference, which we externally label addNextGrade(..). Each time we call *addNextGrade(..)* with a new grade, we get back a current list of the top 10 grades, sorted numerically descending (see *sortAndTrimGradesList()*).

From the end of the original *manageStudentGrades(..)* call, and between the multiple *addNextGrade(..)* calls, the grades variable is preserved inside *addGrade(..)* via closure; that’s how the running list of top grades is maintained. Remember, it’s a closure over the variable grades itself, not the array it holds.
## Why Closure?
Imagine you have a button on a page that when clicked, should retrieve and send some data via an Ajax request. Without using closure:
```Js
var APIendpoints = { 
    studentIDs:
        "https://some.api/register-students",
    // ..
};
var data = {
    studentIDs: [ 14, 73, 112, 6 ], 
    // ..
};
function makeRequest(evt) {
    var btn = evt.target;
    var recordKind = btn.dataset.kind; 
    ajax(
        APIendpoints[recordKind],
        data[recordKind]
    );
}
// <button data-kind="studentIDs">
//    Register Students
// </button>
btn.addEventListener("click",makeRequest);
```
The *makeRequest(..)* utility only receives an *evt* object from a click event. From there, it has to retrieve the *data-kind* attribute from the target button element, and use that value to lookup both a URL for the API endpoint as well as what data should be included in the Ajax request.

This works OK, but it’s unfortunate (inefficient, more confusing) that the event handler has to read a DOM attribute each time it’s fired. Why couldn’t an event handler *remember* this value? Let’s try using closure to improve the code:
```Js
var APIendpoints = { 
    studentIDs:
        "https://some.api/register-students",
    // ..
};
var data = {
    studentIDs: [ 14, 73, 112, 6 ], 
    // ..
};
function setupButtonHandler(btn) {
    var recordKind = btn.dataset.kind;
    btn.addEventListener(
        "click",
        function makeRequest(evt){
                    ajax(
                        APIendpoints[recordKind],
                        data[recordKind]
                        ); 
        }
    ); 
}
// <button data-kind="studentIDs">
//    Register Students
// </button>
setupButtonHandler(btn);
```
With the *setupButtonHandler(..)* approach, the *data-kind* attribute is retrieved once and assigned to the *record-Kind* variable at initial setup. *recordKind* is then closed over by the inner *makeRequest(..)* click handler, and its value is used on each event firing to look up the URL and data that should be sent.

By placing *recordKind* inside *setupButtonHandler(..)*, we limit the scope exposure of that variable to a more appropriate subset of the program; storing it globally would have been worse for code organization and readability. Closure lets the inner *makeRequest()* function instance *remember* this variable and access whenever it’s needed.

Building on this pattern, we could have looked up both the URL and data once, at setup:
```Js
function setupButtonHandler(btn) {
    var recordKind = btn.dataset.kind;
    var requestURL = APIendpoints[recordKind]; 
    var requestData = data[recordKind];
    btn.addEventListener(
            "click",
            function makeRequest(evt){ 
                ajax(requestURL,requestData);
            } 
    );
}
```

Now *makeRequest(..)* is closed over *requestURL* and *requestData*, which is a little bit cleaner to understand, and also slightly more performant.
## Closer to Closure
We explored two models for mentally tackling closure:

- Observational: closure is a function instance remember- ing its outer variables even as that function is passed to and *invoked in* other scopes.
- Implementational: closure is a function instance and its scope environment preserved in-place while any references to it are passed around and invoked from other scopes.

Summarizing the benefits to our programs:
- Closure can improve efficiency by allowing a function instance to remember previously determined information instead of having to compute it each time.
- Closure can improve code readability, bounding scope exposure by encapsulating variable(s) inside function instances, while still making sure the information in those variables is accessible for future use. The resultant narrower, more specialized function instances are cleaner to interact with, since the preserved information doesn’t need to be passed in every invocation.