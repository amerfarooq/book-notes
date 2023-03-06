## Item 1: Understand the Relationship Between TypeScript and JavaScript

1. TypeScript is a superset of JavaScript. In other words, all JavaScript programs are already TypeScript programs.
TypeScript has some syntax of its own, so TypeScript programs are not, in general, valid JavaScript programs.

2. TypeScript’s type system models the runtime behavior of JavaScript. This may result in
  some surprises if you’re coming from a language with stricter runtime checks. The following statements, for example, both pass the type checker, even though they are questionable and do produce 
  runtime errors in many other languages:
    ```ts
    const x = 2 + '3'; // OK, type is string
    const y = '2' + 3; // OK, type is string
    ```
      However, this does model the runtime behavior of JavaScript, where both expressions result in the string "23". The type checker does however flags issues in these statements, even though they do not throw exceptions at runtime, because it considers it more likely that the
  odd usage is the result of an error than the developer’s intent, so it goes beyond simply modeling the runtime behavior.

3. It is possible for code to pass the type checker but still throw at runtime e.g.:
    ```ts
    const names = ['Alice', 'Bob'];
    console.log(names[2].toUpperCase());
    ```
      When you run this, it throws:
`TypeError: Cannot read property 'toUpperCase' of undefined`
TypeScript assumed the array access would be within bounds, but it was not and the
result was an exception. The root cause of these exceptions is that TypeScript’s understanding of a value’s type
and reality have diverged. A type system which can guarantee the accuracy of its
static types is said to be *sound*. TypeScript’s type system is very much not sound, nor
was it ever intended to be. 
