# Javascript Notes

### "this" keyword:

1.  in global space value of "this" depends on global object which can be acccessed through "globalThis" keyword. Now value of globalThis depends on different environments, like in DOM it is window.
2.  Inside functions it depends on strict or non strict mode.
    - in strict mode its "undefined".
    - in non strict its globalThis. Why? because of "this substitution". "This substitution"??: whenever the value of "this" in "undefined" or "null" it becomes globalThis, but, but... only in non strict mode. So this substitution works only in non strict mode.
    - but in both strict and non strict mode if we provide a context, the value of this will depend on that context.
      ```js
      "use strict";
      function a() {
      	return this;
      }
      a(); // undefined
      window.a(); // window object.
      ```
3.  inside objects "this" depends on the object

    ```js
    const user = {
        name: "One",
        printName: function {
            return this.name // this refers to the user object
        }
    }
    console.log(user.printName) // One
    ```

    - but if we want to use different object as context use "call", "apply", "bind" methods.
      ````js
      // example of "call"
      const user = {
      name: "One",
      printName: function {
      return this.name // this refers to the user object
      }
      }
      const anotherUser = {
      name: "Two"
      }
               console.log(user.printName.call(anotherUser)) // Two
           ```
      so we called the user's printName function but used anotherUser object as the context.
      ````

4.  Arrow function don't have their own this, they borrow the value of this from their lexical context. Lexical context means where the function was defined. So when the function was defined what was the value of "this", that will be the value of "this" inside the arrow function.

    ```js
    const person = {
    	name: "Alice",
    	y: this,
    	speak: () => {
    		console.log("arrow", this);
    	},
    	speak2: function () {
    		console.log("regular", this);
    	},
    	speak3: function () {
    		// new lexical context
    		const andarWala = () => {
    			console.log("andar wala arrow", this);
    		};
    		andarWala();
    	},
    };

    console.log(person.y); // window
    person.speak(); // window -> coz value of "this" was window when the function was defined.
    person.speak2(); // person
    person.speak3(); // person -> coz lexical context changed
    ```

**Symbol:**

1. A "Symbol" is a unique identifier.
2. To create it we use "Symbol" keyword
   ```js
   let sym = Symbol();
   ```
3. We can give it a description:
   ```js
   let sym = Symbol("sym");
   ```
4. They are guaranteed unique even if they share the same description.
   ```js
   let sym1 = Symbol("sym");
   let sym2 = Symbol("sym");
   console.log(sym1 === sym2); // false
   ```
5. Use it as hidden properties in objects. In this way they can't be overwritten in any way.

   ```js
   let sym = Symbol("sym");
   user[sym] = "some kind of sym value";
   // if someone else has their own `sym` Symbol, it wont overwrite yours.

   // below a different example
   let user = { name: "Sarthak" };
   user.sym = "Original sym value";
   user.sym = "New sym value";
   // overwritten
   ```

6. They are skipped in for...in loops

   ```js
   let sym = Symbol("sym");
   let user = {
   	name: "Saarthak",
   	[sym]: 123,
   };

   for (let key in user) console.log(key); // name but no symbols

   console.log("Suymbol only", user[sym]); // 123
   ```

7. What is we want same-named symbols? we use Global Symbols. A global symbol registry exists, we create a symbol there and access it by the same name and it will return the same symbol. To do this we use Symbol.for("key") syntax.

   ```js
   let sym = Symbol.for("sym");
   let sameSymbol = Symbol.for("sym");
   console.log(sym === sameSymbol); // true
   ```

   Symbol.for tries to return the symbol, if not found it creates it first and then returns it.

   Now the above returns the symbol name, what if we want to return a name by global symbol, we use Symbol.keyFor(name) syntax

   ```js
   const symOne = Symbol.for("one");
   const symTwo = Symbol.for("two");

   console.log(Symbol.keyFor(symOne), Symbol.keyFor(symTwo)); // "one", "two"
   ```
