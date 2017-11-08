# *JavaScript Notes*
My JavaScript notes.

## Some Definitions
* **Syntax parser** <br/>
A program that reads your code and determines what it does and if its grammar is valid.
* **Lexical environment** <br/>
Where something sits physically in the code you write.
* **Scope** <br/>
Where a variable is available in your code.
* **Execution context** <br/>
A wrapper to help manage the code that is running. <br/>
There are lots of lexical environments, and which one is currently running is managed via execution contexts. Every line of JavaScript code is run in an “execution context.” The JavaScript runtime environment maintains a stack of these contexts, and the top execution context on this stack is the one that’s actively running.
* **Executable code** <br/>
There are three types of executable code: global code, function code, and eval code. Global code is code at the top level of your program that’s not inside any functions, function code is code that’s inside the body of a function, and eval code is global code evaluated by a call to eval.
* **Object** <br/>
A collection of name value pairs.
* **Invocation** <br/>
Running a function.
* **Synchronous** <br/>
One at a time.
* **Asynchronous** <br/>
More than one at a time.
* **Dynamic typing** <br/>
In JavaScript, variables can hold different types of values because it's all figured out during execution.
* **Primitive type** <br/>
A type of data that represents a single value. There are 6 primitive types in JavaScript: undefined, null, boolean, number, string, symbol (used in ES6).
Both *undefined* and *null* represent lack of existence, but never set a varibale to *undefined*, set it to *null*, leave *undefined* to the engine.
* **First class functions** <br/>
Everything you can do with other types you can do with functions.
* **Syntactic sugar** <br/>
A different way to type something that doesn't change how it works under the hood.
* **Transpile** <br/>
Convert the syntax of one programming language to another.

## The global object
All JavaScript runtimes have a unique object called the global object. Its properties include built-in objects like Math and String, as well as extra properties provided by the host environment. In browsers, the global object is the window object. In Node.js, it’s just called the “global object”. 

## Context vs. Scope
Every function invocation has both a scope and a context associated with it. Fundamentally, scope is function-based while context is object-based. In other words, scope pertains to the variable access of a function when it is invoked and is unique to each invocation. Context is always the value of the this keyword which is a reference to the object that “owns” the currently executing code.

## Variable Scope
A variable can be defined in either local or global scope, which establishes the variables’ accessibility from different scopes during runtime. Any defined global variable, meaning any variable declared outside of a function body will live throughout runtime and can be accessed and altered in any scope. Local variables exist only within the function body of which they are defined and will have a different scope for every call of that function. There it is subject for value assignment, retrieval, and manipulation only within that call and is not accessible outside of that scope.

## "this" Context
Context is most often determined by how a function is invoked. When a function is called as a method of an object, *this* is set to the object the method is called on:
```javascript
var obj = {
    foo: function() {
        return this;   
    }
};

obj.foo() === obj; // true
```
The same principle applies when invoking a function with the *new* operator to create an instance of an object. When invoked in *this* manner, the value of this within the scope of the function will be set to the newly created instance:
```javascript
function foo() {
    alert(this);
}

foo() // window
new foo() // foo
```

## Execution Context
JavaScript is a single threaded language, meaning only one task (command) can be executed at a time. When the JavaScript interpreter initially executes code, it first enters into a global execution context by default. Each invocation of a function from this point on will result in the creation of a new execution context.

This is where confusion often sets in, the term “execution context” is actually for all intents and purposes referring more to scope and not context as previously discussed. It is an unfortunate naming convention, however it is the terminology as defined by the ECMAScript specification, so we’re kinda stuck with it.

Each time a new execution context is created it is appended to the top of the execution stack. The browser will always execute the current execution context that is atop the execution stack. Once completed, it will be removed from the top of the stack and control will return to the execution context below.

An execution context can be divided into a creation and execution phase. In the creation phase, the interpreter will first create a variable object (also called an activation object) that is composed of all the variables, function declarations, and arguments defined inside the execution context. From there the scope chain is initialized next, and the value of this is determined last. Then in the execution phase, code is interpreted and executed.

## The Scope Chain
For each execution context there is a scope chain coupled with it. The scope chain contains the variable object for every execution context in the execution stack. It is used for determining variable access and identifier resolution. For example:
```javascript
function first() {
    second();
    function second() {
        third();
        function third() {
            fourth();
            function fourth() {
                // do something
            }
        }
    }   
}
first();
```
Running the preceding code will result in the nested functions being executed all the way down to the *fourth* function. At this point the scope chain would be, from top to bottom: fourth, third, second, first, global. The *fourth* function would have access to global variables and any variables defined within the *firs*t, *second*, and *third* functions as well as the functions themselves.

Name conflicts amongst variables between different execution contexts are resolved by climbing up the scope chain, moving locally to globally. This means that local variables with the same name as variables higher up the scope chain take precedence.

To put it simply, each time you attempt to access a variable within a function’s execution context, the look-up process will always begin with its own variable object. If the identifier is not found in the variable object, the search continues into the scope chain. It will climb up the scope chain examining the variable object of every execution context looking for a match to the variable name. For example:
```javascript
function b() {
    console.log(myVar);
};
function a() {
    var myVar = 2;
    b();
};
var myVar = 1;
a();
// output is 1
```

```javascript
function a() {
    var myVar = 2;
    b();
    function b() {
        console.log(myVar);
    };
};
var myVar = 1;
a();
// output is 2
```
## Closures
Closures are functions with preserved data. Accessing variables outside of the immediate lexical scope creates a closure. In other words, a closure is formed when a nested function is defined inside of another function, allowing access to the outer functions variables. Returning the nested function allows you to maintain access to the local variables, arguments, and inner function declarations of its outer function. This encapsulation allows us to hide and preserve the execution context from outside scopes while exposing a public interface and thus is subject to further manipulation. A simple example of this looks like the following:
```javascript
function foo() {
    var localVariable = 'private variable';
    return function() {
        return localVariable;
    }
}

var getLocalVariable = foo();
getLocalVariable() // "private variable"
```
Another example:
```javascript
function greet(whattosay) {
    return function(name) {
        console.log(whattosay + ' ' + name);
    }
}

greet('Hi')('Tony');

var sayHi = greet(Hi);
sayHi('Tony');
```
One important example:
```javascript 
function buildFunctions () {
    var arr = [];
    for (var i = 0; i < 3; i++) {
        arr.push(function() {
            console.log(i);
        });
    }
    return arr;
}

var fs = buildFunctions();
fs[0](); // 3
fs[1](); // 3
fs[2](); // 3
``` 

```javascript
function buildFunctions () {
    var arr = [];
    for (let i = 0; i < 3; i++) {
        arr.push(
            (function(j) {
                return function() {
                    console.log(j);
                }
            }(i)) // IIFE
        )
    }
    return arr;
}

var fs = buildFunctions();
fs[0](); // 0
fs[1](); // 1
fs[2](); // 2
```

One more example... Again, closures are functions with preserved data. It keeps the value it needed to excute, it preserve the variable inside the function as a property, closure.

```javascript
var addTo = function(passed) {
var addTo = function(passed) {
    var add = function(inner) {
        return passed + inner;
    };
    return add;
};

var addThree = new addTo(3);
var addFour = new addTo(4);
console.log(addThree(1));
console.log(addFour(1));
```

One of the most popular types of closures is what is widely known as the module pattern; it allows you to emulate public, private, and privileged members:
```javascript
var Module = (function() {
    var privateProperty = 'foo';

    function privateMethod(args) {
        // do something
    }

    return {

        publicProperty: '',

        publicMethod: function(args) {
            // do something
        },

        privilegedMethod: function(args) {
            return privateMethod(args);
        }
    };
})();
```
The module acts as if it were a singleton, executed as soon as the compiler interprets it, hence the opening and closing parenthesis at the end of the function. The only available members outside of the execution context of the closure are your public methods and properties located in the return object (*Module.publicMethod* for example). However, all private properties and methods will live throughout the life of the application as the execution context is preserved, meaning variables are subject to further interaction via the public methods.

Another type of closure is what is called an immediately-invoked function expression (IIFE) which is nothing more than a self-invoked anonymous function executed in the context of the window:
```javascript
(function(window) {
          
    var foo, bar;

    function private() {
        // do something
    }

    window.Module = {

        public: function() {
            // do something 
        }
    };

})(this);
```
This expression is most useful when attempting to preserve the global namespace as any variables declared within the function body will be local to the closure but will still live throughout runtime. This is a popular means of encapsulating source code for applications and frameworks, typically exposing a single global interface in which to interact with.


## Hoisting
Hoisting is JavaScript's default behavior of moving declarations to the top. It's WRONG to say that variable and function declarations are physically moved to the top your coding. What happens is the variable and function declarations are put into memory during the compile phase, but stays exactly where you typed it in your coding.  

#### JavaScript Declarations are Hoisted
In Javascript, a variable can be declared after it has been used.
```javascript
x = 5; // assign 5 to x
console.log(x);
var x; // declare x
```
Declaration of x is hoisted and set (variable setup) equal to 'undefined'. 

#### Initializations are Not Hoisted
JavaScript only hoists declarations, not initializations.
```javascript
var x = 5; // initialized x
y = 7; // assign 7 to y
console.log(x+y); // 12
var y; // declare y
```
```javascript
var x = 5; // initialized x
console.log(x+y); // NaN
var y = 7; // initialized y
```

## Scoping and Hoisting
Scoping is quite confusing in JavaScript. Consider the following C program:
```c
#include <stdio.h>
int main() {
  int x = 1;
	printf("%d, ", x); // 1
	if (1) {
		int x = 2;
		printf("%d, ", x); // 2
	}
	printf("%d\n", x); // 1
}
```

Because C has block-level scope. However, JavaScript has function-level scope.
```javascript
var x = 1;
console.log(x); // 1
if (true) {
	var x = 2;
	console.log(x); // 2
}
console.log(x); // 2
```

In JavaScript, blocks, such as if statements, do not create a new scope, only functions do. <br/>
If you must create temporary scopes within a function, do the following:
```javascript
var x = 1;
console.log(x); // 1
if (x) {
	(function () {
		var x = 2;
		console.log(x); // 2
	}());
}
console.log(x); // 1
```

## var, let, const
`var` has existed since the beginning, `let` was introduced in ES2015/ES6. `let` has block scope, variable defined by `let` keyword would die at the end of block. `var` has function scope. Also, `var` gets hoisted, `let` does not. A variable defined by `const` can't be changed. `const` must be initialized. `const c;` will get `Uncaught SyntaxError: Missing initializer in const declaration`. When an object is `const` its property can be changed. 
```javascript
const x = {};
x.a = 1;

const arr = [1,2];
arr.push(3);
arr = [1,2,3]; // assignment not allowed to const variable
```

## Event Queue
Aonther list that sits on JavaScript engine besides execution context. When the execution stack is empty, then JavaScript priority looks at event queue and looks for something to be there. If something is there, it looks the see if a particular function should run when the event is triggered. For example, if click event is on event queue and when execution stack is empty, *clickHandler()* is added to the stack at the event of click.
```javascript
function wait() {
    var ms = 3000 + new Date().getTime();
    while (new Date() < ms) {}
    console.log('finished function')
}

function clickHandler() {
    console.log('click event!');
}

document.addEventListener('click', clickHandler);

wait();
console.log('finish execution');
```

## Functions Are Objects
In JavaScript, functions are first-class objects, because they can have properties and methods just like any other object. What distinguishes them from other objects is that functions can be called. In brief, they are *Function* objects. We can attach properties and methods to a function since it's a object. A function object has some hidden special properties. Two important ones are: name (optional, can be anonymous) and code ("invocable" ()). 

## By Value vs By Reference
JavaScript is always pass by value, but when a variable refers to an object (including arrays), the "value" is a reference to the object. Consider the following example:
```javascript
// by value (primitives)
var a = 3;
var b;

b = a;
a = 2;

console.log(a); // 2
console.log(b); // 3

// by reference (all objects (including functions))
var c;
c = {'greet': 'hi'};
d = c;

c.greet = 'hello'; // mutate 

console.log(c); // {'greet': 'hello'}
console.log(d); // {'greet': 'hello'}

// by reference even as parameters
function change(obj) {
    obj.greet = 'Hola'; // mutate
}

change(c)

console.log(c); // {'greet': 'hola'}
console.log(d); // {'greet': 'hela'}

// equlas operator sets up new memory space (new address)
c = {'greet': 'hiii'};

console.log(c); // {'greet': 'hiii'}
console.log(d); // {'greet': 'hola'}
```

## Object Function and "this"
```javascript
var c = {
    name: 'c object',
    log: function() {
        console.log(this.name); // c object
        this.name = 'Updated c object';
        console.log(this.name); // Update c object
        var setname = function(newname) {
            this.name = newname;
        }
        
        setname('Updated c object again');
        console.log(this.name); // update c object
    }
}

c.log();

console.log(name); // Updated c object again
```
```javascript
var c = {
    name: 'c object',
    log: function() {
        var self = this;
        console.log(self.name); // c object
        self.name = 'Updated c object';
        console.log(self.name); // Update c object
        var setname = function(newname) {
            self.name = newname;
        }
        
        setname('Updated c object again');
        console.log(self.name); // Updated c object again
    }
}

c.log();
```

## Arguments
```javascript
function func(firstname, lastname, language = 'en') {
    console.log(arguments);
}

func('John', 'Lin', 'zh'); // ['John', 'Lin', 'zh']
```

## Spread
```javascript
function func(firstname, lastname, language = 'en', ...other) {
    console.log(other)
}

func('John', 'Lin', 'zh', '1', 2); // ['1', 2]
```

## Automatic Semicolon Insertion
The principle of the feature is to provide a little leniency when evaluating the syntax of a JavaScript program by conceptually inserting missing semicolons. For example:
```javascript
var foo = 1
var bar = 2
```
These syntax errors are accommodated for, and the code will considered as:
```javascript
var foo = 1;
var bar = 2;
```
The most common problem caused by missing out a semicolon is with the return keyword. Consider this following code snippet:
```javascript
var foo = function() {
  var bar = 'baz'
  return 
  {
    bar: bar
  }
}

console.log(foo());
```
This code is syntactically incorrect and would be better treated as a syntax error. The grammar rule for the return statement is as follows:

return [no LineTerminator here] Expression ;

I.e. there should be no line terminator after the return keyword. But automatic semicolon insertion accommodates this error by transforming the return statement to this:
```javascript
return;
  {
    bar: bar
  }
```
When this code executes the output to the console is undefined and not the object. [Read More](http://www.bradoncode.com/blog/2015/08/26/javascript-semi-colon-insertion/)

## Immediately Invoked Function Expression (IIFE)
```javascript
// function statement
function greet() {
    console.log('Hello');
}
greet();

// using a function expression
var greetFunc = function () {
    console.log('Hello');
};
greetFunc();

// using an Immediately Invoked Function Expressiong (IIFE)
var greeting = function () {
    console.log('Hello');
}();

// inside IIFE
// function expression, wrap with parenthesises
(function() {
    console.log('Hello');
 }());
```

## call, apply and bind
*call* and *apply* invoke the function and let you set up for *this* keyword and pass the parameters if need. *bind* creates a copy of the function and let you set up what *this* keyword should mean and set default parameters.
```javascript
var person = {
    firstname: 'john',
    lastname : 'Doe',
    getFullName: function () {
        var fullname = this.firstname + ' ' + this.lastname;
        return fullname;
    }
}

var logName = function(arg1, arg2) {
    console.log('Logged: ' + this.getFullName() + ' ' + arg1 + ' ' + arg2);
}

var logPersonName = logName.bind(person);

logPersonName('111', '222');    // Logged: john Doe 111 222
logName.call(person, '111', '222'); // Logged: john Doe 111 222
logName.apply(person, ['111', '222']);  // Logged: john Doe 111 222
```
use for function borrowing (grab methods from object and use them):
```javascript
var person2 = {
    firstname: 'Jane',
    lastname: 'Doe'
}

console.log(person.getFullName.apply(person2)); // Jane Doe
```
use for function currying (creating a copy of a function but with some present parameters):
```javascript
function mutiply(a, b) {
    return a*b;
}

var multiplyByTwo = mutiply.bind(this, 2);
/* same as
 function multiplyByTwo(b) {
    var a = 2;
    return a*b;
}
*/

console.log(multiplyByTwo(3)); // 6
console.log(mutiply.bind(this, 2, 2)()); // 4
```

## Arrow Function
An arrow function is more concise and easier to read:
```javascript
var smartPhones = [
    { name:'iphone', price:649 },
    { name:'Galaxy S6', price:576 },
    { name:'Galaxy Note 5', price:489 }
];

// es5
console.log(smartPhones.map(
    function(smartPhone) {
    return smartPhone.price;
    }
)); // [649, 576, 489]

// es6
console.log(smartPhones.map(
    smartPhone => smartPhone.price
)); // [649, 576, 489]
```
The other benefit of using arrow functions with promises/callbacks is that it reduces the confusion surrounding the this keyword. In code with multiple nested functions, it can be difficult to keep track of and remember to bind the correct this context. In ES5, you can use workarounds like the `.bind` method or creating a closure using `var self = this;`.

Because arrow functions allow you to retain the scope of the caller inside the function, you don’t need to create `self = this` closures or use bind. For the previous example earlier:
```javascript
var c = {
    name: 'c object',
    log: function() {
        console.log(this.name); // c object
        this.name = 'Updated c object';
        console.log(this.name); // Update c object
        var setname = (newname) => {
            this.name = newname;
        }
        
        setname('Updated c object again');
        console.log(this.name); // Updated c object again
    }
}

c.log();
console.log(this.name); // undefined
console.log(c.name); // Updated c object again
```

# Objection Creation
## Factory Pattern
```javascript
var peopleFactory = function(name, age, state) {
    var property = {};
    
    property.name = name;
    property.age = age;
    property.state = state;
    property.printPerson = function() {
        console.log(this.name + " " + this.age + " " + this.state);
    };
    return property;
};

var person1 = peopleFactory('Mike', 20, 'NY');
person1.printPerson(); // Mike 20 NY
``` 

## Constructor Pattern
```javascript
var peopleConstructor = function(name, age, state) {
    this.name = name;
    this.age = age;
    this.state = state;
    this.printPerson = function() {
        console.log(this.name + " " + this.age + " " + this.state);
    };
    return property;
};

// use new to create other object from peopleConstructor object
var person1 = new peopleConstructor('Mike', 20, 'NY');
person1.printPerson(); // Mike 20 NY
```

## Prototype Pattern
```javascript
var peopleProto = function() {};

peopleProto.prototype.age = 0;
peopleProto.prototype.name = 'no name';
peopleProto.prototype.state = 'no city';
peopleProto.prototype.dummy = 'dummy';
peopleProto.prototype.printPerson = function() {
    console.log(this.name + " " + this.age + " " + this.state);
}

var person2 = new peopleProto();
person2.name = 'John';
person2.age = 23;
person2.state = 'NJ';

person2.printPerson();
console.log('dummy' in person2); // true, person2 inherit dummy from its prototype
console.log(person2.hasOwnProperty('dummy')); // false
```

## Dynamic Prototype Pattern
```javascript
var peopleDynamicProto = function(name, age, state) {
    this.age = age;
    this.name = name;
    this.state = state;
    
    if (typeof this.printPerson !== 'function') {
        peopleDynamicProto.prototype.printPerson = function() {
            console.log(this.name + " " + this.age + " " + this.state);
        };
    }
}

var person1 = new peopleDynamicProto();
var person2 = new peopleDynamicProto();
```

## Prototype
Every JavaScript object has a prototype. The prototype is also an object. All JavaScript objects inherit their properties and methods from their prototype. Consider the following example:
```javascript
function Person(firstname, lastname) {
  this.firstname = firstname;
  this.lastname = lastname;
}

Person.prototype.getFullName = function () {
  return this.firstname + ' ' + this.lastname;
}

var john = new Person('John', 'Doe');
var jane = new Person('Jane', 'Doe');
```
Functions (methods) are re-used, not re-created. If *getFullName* are added in the *Person* object, *john* and *jane* objects both would have a copy of *getFullName* when they are created. Instead, *john* and *jane* objects are created with *new Person()* inherit the *Person.prototype*.

## ES6 Classes
In JavaScript, a class is also an object. A class constructor must be invoked with *new*.
```javascript
class Person {
  constructor(firstname, lastname) {
    this.firstname = firstname;
    this.lastname = lastname;
  }
  greet() {
    return 'Hi ' + this.firstname + ' ' + this.lastname;
  }
}

var john = new Person('John', 'Doe');
console.log(john.greet());
```
*extends* sets the Prototype:
```javascript
class InformalPerson extends Person {
  constructor(firstname, lastname) {
    super(firstname, lastname);
  }
  greet() {
    return 'Yo ' + this.firstname;
  }
}

var jane = new InformalPerson('Jane', 'Doe');
console.log(jane.greet());
```

## Own Library
```javascript
(function(global, $) {

  var Greetr = function(firstName, lastName, language) {
    return new Greetr.init(firstName, lastName, language);
  }

  var supportedLangs = ['en', 'es'];

  var greetings = {
    en: 'Hello',
    es: 'Hola'
  };

  var formalGreetings = {
    en: 'Greeings',
    es: 'Saludos'
  };

  var logMessage = {
    en: 'Logged in',
    es: 'Inicio sesion'
  };


  Greetr.prototype = {
    fullName: function () {
      return this.firstName + ' ' + this.lastName;
    },
    validate: function () {
      if (supportedLangs.indexOf(this.language) === -1) {
        throw "Invalid language";
      }
    },
    greeting: function () {
      return greetings[this.language] + ' ' + this.firstName + '!';
    },
    formalGreeting: function () {
      return formalGreetings[this.language] + ', ' + this.fullName();
    },
    greet: function(formal) {
      var msg;
      if (formal) {
        msg = this.formalGreeting();
      } else {
        msg = this.greeting();
      }
      if (console) {
        console.log(msg);
      }

      // 'this' refers to the calling object at execution time
      // makes the method chinable
      return this;
    },
    log: function() {
      if (console) {
        console.log(logMessage[this.language] + ': ' + this.fullName());
      }
      return this;
    },
    setLang: function(lang) {
      this.language = lang;
      this.validate();
      return this;
    }
  };

  Greetr.init = function(firstName, lastName, language) {
    var self = this;
    self.firstName = firstName || '';
    self.lastName = lastName || '';
    self.language = language || 'en';
    self.validate();
  }

  Greetr.init.prototype = Greetr.prototype;

  global.Greetr = global.G$ = Greetr;

}(window, jQuery));
```

```javascript
var g = G$('John', 'Doe');
g.greet().setLang('es').greet(true);
```

## Promise
```javascript
let promiseToCleanTheRoom = new Promise(function(resolve, reject) {
    // cleaning the room
    
    let isClean = true;
    if (isClean) {
        resolve('clean');
    } else {
        reject('not clean');
    }
});

promiseToCleanTheRoom.then(function(fromResolve) {
    console.log('The room is ' + fromResolve);
}).catch(function(fromReject) {
    console.log('The room is' + fromReject);
})
```

```javascript
// nested promise, when one is finish then do next

let cleanRoom = function() {
  return new Promise(function(resolve, reject) {
    resolve('Cleaned The Room');
  });
};

let removeGarbage = function(message) {
  return new Promise(function(resolve, reject) {
    resolve(message + ' Remove Garbage');
  });
};

let winIcecream = function(message) {
  return new Promise(function(resolve, reject) {
    resolve( message + ' Won Icecream');
  });
};

cleanRoom().then(function(result){
	return removeGarbage(result);
}).then(function(result){
	return winIcecream(result);
}).then(function(result){
	console.log('finished ' + result);
})

// do all in parallel, don't want to wait for on to finish, when all is done then do something
Promise.all([cleanRoom(), removeGarbage(), winIcecream()]).then(function() {
	console.log('all finish');
});
// if just want any of them finishes
Promise.race([cleanRoom(), removeGarbage(), winIcecream()]).then(function() {
	console.log('one of them finishes');
});

```
