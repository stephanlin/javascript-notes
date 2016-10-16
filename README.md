# *Javascript - Understanding The Weird Parts*
My Javascript notes. Still editing...

## Conceptual Aside
* **Syntax parser** <br/>
A program that reads your code and determines what it does and if its grammar is valid.
* **Lexical environment** <br/>
Where something sits physically in the code you write.
* **Execution context** <br/>
A wrapper to help manage the code that is running. <br/>
There are lots of lexical environments, and which one is currently running is managed via execution contexts. Every line of JavaScript code is run in an “execution context.” The JavaScript runtime environment maintains a stack of these contexts, and the top execution context on this stack is the one that’s actively running.
* **Executable code** <br/>
There are three types of executable code: global code, function code, and eval code. Global code is code at the top level of your program that’s not inside any functions, function code is code that’s inside the body of a function, and eval code is global code evaluated by a call to eval.
* **The global object** <br/>
All Javascript runtimes have a unique object called the global object. Its properties include built-in objects like Math and String, as well as extra properties provided by the host environment. In browsers, the global object is the window object. In Node.js, it’s just called the “global object”. 
