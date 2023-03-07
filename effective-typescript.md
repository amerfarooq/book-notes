
<details>
   <summary><b>1. Understand the Relationship Between TypeScript and JavaScript </b></summary>

1. TypeScript is a superset of JavaScript. In other words, all JavaScript programs are already TypeScript programs.
TypeScript has some syntax of its own, so TypeScript programs are not, in general, valid JavaScript programs.

2. TypeScript’s type system models the runtime behavior of JavaScript. This may result in
  some surprises if you’re coming from a language with stricter runtime checks. The following statements, for example, both pass the type checker, even though they are questionable and do produce 
  runtime errors in many other languages:
    ```ts
    const x = 2 + '3'; // OK, type is string
    const y = '2' + 3; // OK, type is string
    ```
      However, this does model the runtime behavior of JavaScript, where both expressions result in the string "23". On the plust side, the type checker will flag issues in these statements, even though they do not throw exceptions at runtime, because it considers it more likely that the
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
  
</details>

<details>
   <summary><b>2. Know Which TypeScript Options You’re Using </b></summary>
   &nbsp;

1. The TypeScript compiler includes several settings which affect core aspects of the language and these can be configured via the `tsconfig.json` file as well as through the command-line.
1. Turn on the `noImplicitAny` setting to ensure TypeScript doesn't assign the `any` type to a variable when it can't determine its type automatically.
1. Use `strictNullChecks` to prevent “undefined is not an object”-style runtime errors.
1. Aim to enable strict to get the most thorough checking that TypeScript can offer.

</details>  
  
<details>
   <summary><b> 3. Understand That Code Generation Is Independent of Types </b></summary>  
   &nbsp;
   
1. The TypeScript compiler has two functions: it transpiles the TypeScript code into JavaScript and checks for type errors. The important thing to note is that these functions are independant of each other. 
1. Since the transpilation process is seperate, all the actual TypeScript types are erased during compilation to JavaScript and have no affect on the runtime behavior of the code. 
1. Since types are stripped away during compilation, the compiler can still generate output even when there are type errors in the code. Therefore, TypeScript errors act more like warnings and can point to issues that warrant further investigation, but they won't necessarily stop the build process.
1. Similarly, it is also not possible to check TypeScript types at runtime so something like the following does not work:
    ```ts
    interface Square {
     width: number;
    }
    interface Rectangle extends Square {
     height: number;
    }
    type Shape = Square | Rectangle;

    function calculateArea(shape: Shape) {
     if (shape instanceof Rectangle) {
                         // ~~~~~~~~~ 'Rectangle' only refers to a type,
                         // but is being used as a value here
      return shape.width * shape.height;
                         // ~~~~~~ Property 'height' does not exist
                         // on type 'Shape'
     } 
     else {
      return shape.width * shape.width;
     }
    }
    ```
    There are three ways to get around this:
  * Property checking:
    ```ts
    function calculateArea(shape: Shape) {
     if ('height' in shape) {
      shape; // Type is Rectangle
      return shape.width * shape.height;
     } 
     else {
      shape; // Type is Square
      return shape.width * shape.width;
     }
    }
    ```
  * Tagged Unions:
      ```ts
      interface Square {
       kind: 'square';
       width: number;
      }
      interface Rectangle {
       kind: 'rectangle';
       height: number;
       width: number;
      }
      type Shape = Square | Rectangle;
      function calculateArea(shape: Shape) {
       if (shape.kind === 'rectangle') {
         shape; // Type is Rectangle
         return shape.width * shape.height;
       } 
       else {
         shape; // Type is Square
         return shape.width * shape.width;
       }
      }
  * Using class since it acts as both a type and a value, whereas an interface only act as a type.
      ```ts
      class Square {
       constructor(public width: number) {}
      }
      class Rectangle extends Square {
       constructor(public width: number, public height: number) {
       super(width);
       }
      }
      type Shape = Square | Rectangle;

      function calculateArea(shape: Shape) {
       if (shape instanceof Rectangle) {
         shape; // Type is Rectangle
         return shape.width * shape.height;
       } 
       else {
         shape; // Type is Square
         return shape.width * shape.width; // OK
       }
      }
      ```
  5. The use of type operations, such as type casting, cannot affect the runtime behavior of code in TypeScript and instead, you need to use runtime type checks and JavaScript constructs in their place e.g.:

      ```ts
      function asNumber(val: number | string): number {
        return val as number; // Type operation, does not actually perform any conversion
      }

      // Generated JavaScript code
      function asNumber(val) {
        return val;
      }

      function asNumber(val: number | string): number {
        return typeof(val) === 'string' ? Number(val) : val; // Normalizes value using runtime type check and JavaScript constructs
      }
      ```
  6. Runtime types may not be the same as declared types. Consider the following example:
      ```ts
      function setLightSwitch(value: boolean) {
        switch (value) {
          case true:
          turnLightOn();
          break;
        case false:
          turnLightOff();
          break;
        default:
          console.log(`I'm afraid I can't do that.`);
        }
      }
      ```
      The `default` case should not be reachable but `boolean` is just a type and goes away at runtime and it is possible that in the JavaScript code, a user might inadvertently call `setLightSwitch` with a value like "ON".
 7. TypeScript's static types have zero runtime overhead, as they are erased when generating JavaScript. This means that TypeScript's static types do not affect runtime performance. However, there are two caveats to this: 1) the TypeScript compiler introduces build time overhead, although compilation is usually fast and there may be options to skip type checking, and 2) the code that TypeScript emits to support older runtimes may incur performance overhead, which is independent of the types themselves. Despite these caveats, TypeScript's static types are generally considered to be a zero-cost feature.

</details>  
