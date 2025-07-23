# <center>Javascript Notes</center>

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

### Symbol:

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
   	name: "Sarthak",
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

### Garbage collection:

1.  It happens automatically in javascript, unlike some low level languages.
2.  Its all about reachability. If some value is not reachable in the object tree it is garbage collected.
3.  Any value is considered reachable if it can be reached from root by a reference or by chain of references.
    ```js
    const user = { name: "one" };
    const anotherUser = user;
    user = null;
    console.log(anotherUser); // {name: "one"}
    ```
    now above the object is still reachable through anotherUser variable so its not garbage collected.
4.  Main algorithm which is used in called "Mark-and-Sweep". As the names suggests, it starts from root and mark (remembers) it, then it goes to every node which can be visited through reference and marks them as well. It keeps doing this untill every reachable node is marked. Then the remaining unmarked are removed from the memory.
5.  There are some optimizations done by javascript engines to make the process of garbage collection faster and not introduce any delays into the code execution.
    Below are some of those optimizations:

    - **General collection:** Objects are grouped as "old" and "new". New objects are checked often because most die quickly, old ones are checked less often to save time. New ones, if used again ana again is then put in that old category so they are also checked less often.
    - **Incremental Collection:** Memmory is cleaned in small parts, not all at once. This avoids big pauses during the code execution. So we, user, don't notice any pause.
    - **Idle-time Collection:** Cleaning happens when CPU is idle. This reduces any slowdown in running code.

      There are more optimizations, but these are main.

### Transpilers and Polyfills

New syntax and features of languages don't work with older engines or browsers, to tackle this issue there are 2 tools that are used:

- **Transpilers**
- **Polyfills**

1. **Transpilers** are the piece of softwares that translates the source code to another source code. It means it takes modern syntax and rewrite it using older syntax so that older browsers can understand it.
   Eg. 'nullish coalescing operator' didn't existed in older version or Javascript, so it takes that and converts it into something else.

   ```js
   // before running the transpiler
   const height = height ?? 100;

   // after running the transpiler
   const height = height !== undefined && height !== null ? height : 100;
   ```

   Usually programmers runs the transpilers in their own computer and deploys the transpiled code to the server.

   **Babel** is one of the most used transpilers.

2. **Polyfills** are softwares that take new features and rewrites them in older ways. Eg. `Math.trunc` don't work in older engines so a polyfill will be created such that it will convert it to older syntax.
   ```js
   if (!Math.trunc) {
   	// if no such function
   	// implement it
   	Math.trunc = function (number) {
   		// Math.ceil and Math.floor exist even in ancient JavaScript engines
   		// they are covered later in the tutorial
   		return number < 0 ? Math.ceil(number) : Math.floor(number);
   	};
   }
   ```

### Object to primitive conversions

1. functions like `alert(...)` or `String(...)` expects string to be passed, but happens when we pass object instead of strings? It gets auto converted to primitive values.
2. There are 7 primitive types: `string`, `number`, `bigint`, `boolean`, `symbol`, `null` and `undefined`.
3. All objects are `true` when converted to boolean.
4. Numeric happens when we apply mathematical functions, like subtracting dates to get the difference.
5. String conversion happens when we try to output the object with like `alert(user)`.
   How does javascript decide which conversion to apply?: **Hints**
   There are 3 hints "string", "number", "default".

   1. "string" is used when we do operations that expects "string" argument like alert
   2. same thing with "number", when math is being done.
   3. "default" its a rare case, happens when operator is not sure what to use. Eg. "+" can be used with string and numbers both, same goes with `==`, `> <.

6. All built-in objects except for one case (Date object, we’ll learn it later) implement "default" conversion the same way as "number".
7. To do the conversion, JavaScript tries to find and call three object methods:
   1. Call `obj[Symbol.toPrimitive](hint)` – the method with the symbolic key Symbol.toPrimitive (system symbol), if such method exists.
   2. Otherwise if hint is "string", try calling obj.toString() or obj.valueOf(), whatever exists.
   3. Otherwise if hint is "number" or "default", try calling obj.valueOf() or obj.toString(), whatever exists.
8. **Symbol.toPrimitive:**

   ```js
   let user = {
   	name: "John",
   	money: 1000,

   	[Symbol.toPrimitive](hint) {
   		console.log(`hint: ${hint}`);
   		return hint == "string" ? `{name: "${this.name}"}` : this.money;
   	},
   };

   // conversions demo:
   console.log(user); // hint: string -> {name: "John"}
   console.log(+user); // hint: number -> 1000
   console.log(user + 500); // hint: default -> 1500
   ```

9. If no `Symbol.toPrimitive` is there then javascript tries to find `toString` and `valueOf` methods.

   1. For the "string" hint: call toString method, and if it doesn’t exist or if it returns an object instead of a primitive value, then call valueOf (so toString has the priority for string conversions).
   2. For other hints: call valueOf, and if it doesn’t exist or if it returns an object instead of a primitive value, then call toString (so valueOf has the priority for maths).
   3. By default, a plain object has following toString and valueOf methods:
      - The toString method returns a string "[object Object]".
      - The valueOf method returns the object itself.

   We can overwrite the `toString` and `valueOf`. Below eg. we will overwrite it to make sure it works like it normally works

   ```js
   let user = {
   	name: "John",
   	money: 1000,

   	// for hint="string"
   	toString() {
   		return `{name: "${this.name}"}`;
   	},

   	// for hint="number" or "default"
   	valueOf() {
   		return this.money;
   	},
   };

   console.log(user); // toString -> {name: "John"}
   console.log(+user); // valueOf -> 1000
   console.log(user + 500); // valueOf -> 1500
   ```

10. A conversion can return any type, like `toString` don't actually have to return "string". Then only mandatory thing it these methods must return a primitive.
11. As we know already, many operators and functions perform type conversions, e.g. multiplication \* converts operands to numbers.

If we pass an object as an argument, then there are two stages of calculations:

1.  The object is converted to a primitive (using the rules described above).
2.  If necessary for further calculations, the resulting primitive is also converted.

```js
let obj = {
	// toString handles all conversions in the absence of other methods
	toString() {
		return "2";
	},
};

console.log(obj * 2); // 4, object converted to primitive "2", then multiplication made it a number
```

```js
let obj = {
	toString() {
		return "2";
	},
};

console.log(obj + 2); // "22" ("2" + 2), conversion to primitive returned a string => concatenation
```

### Map

1. Map are like objects but main difference is it allows keys of any types. Below are the methods that can be applied to a Map.

   - `new Map()` – creates the map.
   - `map.set(key, value)` – stores the value by the key.
   - `map.get(key)` – returns the value by the key, undefined if key doesn’t exist in map.
   - `map.has(key)` – returns true if the key exists, false otherwise.
   - `map.delete(key)` – removes the element (the key/value pair) by the key.
   - `map.clear()` – removes everything from the map.
   - `map.size` – returns the current element count.

   ```js
   let map = new Map();

   map.set("1", "str1"); // a string key
   map.set(1, "num1"); // a numeric key
   map.set(true, "bool1"); // a boolean key

   // remember the regular Object? it would convert keys to string
   // Map keeps the type, so these two are different:
   console.log(map.get(1)); // 'num1'
   console.log(map.get("1")); // 'str1'

   console.log(map.size); // 3
   ```

2. Using objects as keys

   ```js
   let john = { name: "John" };

   // for every user, let's store their visits count
   let visitsCountMap = new Map();

   // john is the key for the map
   visitsCountMap.set(john, 123);

   console.log(visitsCountMap.get(john)); // 123
   ```

3. Iteration:

   - `map.keys()` – returns an iterable for keys,
   - `map.values()` – returns an iterable for values,
   - `map.entries()` – returns an iterable for entries [key, value], it’s used by default in `for..of`.

   ```js
   let recipeMap = new Map([
   	["cucumber", 500],
   	["tomatoes", 350],
   	["onion", 50],
   ]);

   // iterate over keys (vegetables)
   for (let vegetable of recipeMap.keys()) {
   	console.log(vegetable); // cucumber, tomatoes, onion
   }

   // iterate over values (amounts)
   for (let amount of recipeMap.values()) {
   	console.log(amount); // 500, 350, 50
   }

   // iterate over [key, value] entries
   for (let entry of recipeMap) {
   	// the same as of recipeMap.entries()
   	console.log(entry); // cucumber,500 (and so on)
   }
   ```

   Map also have inbuilt `forEach`

   ```js
   myMap.forEach((value, key, map) => {
   	console.log(value, key, map); // cucumber: 500 etc, map is the map itself here.
   });
   ```

4. Creating map from object

   ```js
   let obj = {
   	name: "John",
   	age: 30,
   };

   let map = new Map(Object.entries(obj));
   ```

5. Creating Object from map:

   ```js
   const myMap = new Map([
   	["one", 1],
   	["two", 2],
   ]);
   const myObj = Object.fromEntries(myMap);
   ```

   But as we know map can have object as keys, so if we create object from such a map, key will be `[object Object]`.

### Set

1. In a Set a value can accur only once. Its main methods are

   - `new Set([iterable])` – creates the set, and if an iterable object is provided (usually an array), copies values from it into the set.
   - `set.add(value)` – adds a value, returns the set itself.
   - `set.delete(value)` – removes the value, returns true if value existed at the moment of the call, otherwise false.
   - `set.has(value)` – returns true if the value exists in the set, otherwise false.
   - `set.clear()` – removes everything from the set.
   - `set.size` – is the elements count.

   ```js
   const mySet = new Set();
   mySet.add("one");
   mySet.add("two");
   mySet.add("one");
   for (const val of mySet) {
   	console.log(val); // one -> two.
   }
   // one won't be repeated again.
   ```

   Alternative to above is using array, but we would need to check the value inside itby `arr.find(...)` so the performance would be bad.

2. Iteration of `Set`:

   ```js
   let set = new Set(["oranges", "apples", "bananas"]);

   for (let value of set) console.log(value);

   // the same with forEach:
   set.forEach((value, valueAgain, set) => {
   	console.log(value, valueAgain, set);
   });
   ```

   In above 3 arguments are passed, set returns the set itself. But why we have value twice? Its to make it compatible with `Map`. This is strange, but in some scenarios this helps replacing `Map` with `Set` and vice-versa.

   - `set.keys()` – returns an iterable object for values,
   - `set.values()` – same as set.keys(), for compatibility with Map,
   - `set.entries()` – returns an iterable object for entries [value, value], exists for compatibility with Map.

### WeakMap

1. We know if object is reachable it won't be garbage collected.

   ```js
   let a = { name: "john" };
   const b = new Map();
   b.set(a, "something");
   a = null;
   ```

   still john object is in memory but weak map differs here. They don't stop garbage collection.

2. In WeakMap keys must be objects, else it throws error.

3. If object that is being used as key has no reference left, it will be removed from the memory.

   ```js
   let john = { name: "John" };

   let weakMap = new WeakMap();
   weakMap.set(john, "...");

   john = null; // overwrite the reference

   // john is removed from memory!
   ```

   So if john only exists as the key of WeakMap, it will be removed from memory, thus removing it from the WeakMap too.

4. WeakMap don't use iteration and methods like `keys()`, `values()`, `entries()`. It has only following methods.

   - weakMap.set(key, value)
   - weakMap.get(key)
   - weakMap.delete(key)
   - weakMap.has(key)

   Why though? Because as javascript engine don't do garbage collection immediately, it might wait to do it. So current count of elements inside WeakMap is not known.

5. Use cases:
   - If we are using some other data like a third party library, and we want to associate some data with it but only untill the third party library data is present, then we can use WeakMap. As if the third party library data is removed, its associated value will also be removed from the WeakMap.
   - In Caching. Using Maps, lets say we have an object and we want to associate some data with it. But in the future that object is removed, but it wont be garbage collected and its association will still be present in the ccache. On the other hand if we use WeakMap for caching, once the object is garbage collected, its associated data will also get removed from cache.

### WeakSet

1. `WeakSet` are kinda similar to `WeakMap` such that.

   - It is analogous to Set, but we may only add objects to `WeakSet` (not primitives).
   - An object exists in the set while it is reachable from somewhere else.
   - Like Set, it supports `add`, `has` and `delete`, but not `size`, `keys()` and no iterations.

   ```js
   let visitedSet = new WeakSet();

   let john = { name: "John" };
   let pete = { name: "Pete" };
   let mary = { name: "Mary" };

   visitedSet.add(john);
   visitedSet.add(pete);
   visitedSet.add(john);

   console.log(visitedSet.has(john)); // true
   john = null;
   console.log(visitedSet.has(john)); // false
   ```

### JSON

1. We have 2 main methods to convert things to JSON and vice-versa.

   - `JSON.stringify(...)`
   - `JSON.parse(...)`

   **JSON.stringify(...):**

   ```js
   const user = { name: "John", age: 20 };
   const str = JSON.stringify(user); // {"name": "John", "age": 20}
   ```

   Now JSON converts everything to string and it wraps them in double quotes.
   It supports following data types:

   - Objects
   - Arrays
   - primitives:
     - strings
     - numbers
     - booleans
     - null

   JSON is data only language independent specification, so some javascript object properties are skipped, mainly

   - functions
   - Symbolic keys and values
   - properties that store undefined.

   ```js
   let user = {
   	sayHi() {
   		// ignored
   		alert("Hello");
   	},
   	[Symbol("id")]: 123, // ignored
   	something: undefined, // ignored
   };

   console.log(JSON.stringify(user)); // {} (empty object)
   ```

   Nested objects are automatically converted. The only limitation? Circular references are skipped.

   ```js
   let room = {
   	number: 23,
   };

   let meetup = {
   	title: "Conference",
   	participants: ["john", "ann"],
   };

   meetup.place = room; // meetup references room
   room.occupiedBy = meetup; // room references meetup

   JSON.stringify(room); // Error: Converting circular structure to JSON
   ```

   Full syntax: **`JSON.stringify(value[,replacer, space])`**

   - **value:** value to encode
   - **replacer:** array of properties to encode or mapping functions `function(key, value)`
   - **space:** amount of space to use for formatting

   ```js
   let room = {
   	number: 23,
   };

   let meetup = {
   	title: "Conference",
   	participants: [{ name: "John" }, { name: "Alice" }],
   	place: room, // meetup references room
   };

   room.occupiedBy = meetup; // room references meetup

   console.log(JSON.stringify(meetup, ["title", "participants"]));
   // {"title":"Conference","participants":[{},{}]}
   ```

   The property list is applied to the whole object structure. So the objects in participants are empty, because name is not in the list.
   To include "name" pass "name" in array also.

   Now lets say object is too big and we just want to remove couple of properties, then we can use replacer function.

   ```js
   let room = {
   	number: 23,
   };

   let meetup = {
   	title: "Conference",
   	participants: [{ name: "John" }, { name: "Alice" }],
   	place: room, // meetup references room
   };

   room.occupiedBy = meetup; // room references meetup

   alert(
   	JSON.stringify(meetup, function replacer(key, value) {
   		alert(`${key}: ${value}`);
   		return key == "occupiedBy" ? undefined : value;
   	})
   );

   /* key:value pairs that come to replacer:
      :             [object Object]
      title:        Conference
      participants: [object Object],[object Object]
      0:            [object Object]
      name:         John
      1:            [object Object]
      name:         Alice
      place:        [object Object]
      number:       23
      occupiedBy: [object Object]
   */
   ```

   The first call is special. It is made using a special “wrapper object”: {"": meetup}. In other words, the first (key, value) pair has an empty key, and the value is the target object as a whole. That’s why the first line is ":[object Object]" in the example above.

   **Custom toJSON:**

   ```js
   let room = {
   	number: 23,
   };

   let meetup = {
   	title: "Conference",
   	date: new Date(Date.UTC(2017, 0, 1)),
   	room,
   };

   alert(JSON.stringify(meetup));
   /*
   {
      "title":"Conference",
      "date":"2017-01-01T00:00:00.000Z",  // (1)
      "room": {"number":23}               // (2)
   }
   */
   ```

   Here we can see that date (1) became a string. That’s because all dates have a built-in toJSON method which returns such kind of string.

   Now let’s add a custom toJSON for our object room (2):

   ```js
   let room = {
   	number: 23,
   	toJSON() {
   		return this.number;
   	},
   };

   let meetup = {
   	title: "Conference",
   	room,
   };

   alert(JSON.stringify(room)); // 23

   alert(JSON.stringify(meetup));
   /*
   {
      "title":"Conference",
      "room": 23
   }
   */
   ```

   **JSON.parse(...):**

   - Syntax: **`let value = JSON.parse(str[, reviver]);`**

   ```js
   const obj = { one: 1, two: 2, three: 3, four: 4 };
   const s = JSON.stringify(obj);

   const j = JSON.parse(s, (key, value) => {
   	return key === "three" ? undefined : value;
   });
   ```

   Above skips the "three" key value.

   ```js
   let str = '{"title":"Conference","date":"2017-11-30T12:00:00.000Z"}';

   let meetup = JSON.parse(str);

   console.log(meetup.date.getDate()); // Error!
   ```

   Gives error because date comes out as string.
   To fix this we use reviver.

   ```js
   let str = '{"title":"Conference","date":"2017-11-30T12:00:00.000Z"}';

   let meetup = JSON.parse(str, function (key, value) {
   	if (key == "date") return new Date(value);
   	return value;
   });

   console.log(meetup.date.getDate()); // now works!
   ```

### Rest paramenter `...`

1. A function can have n number of arguments. To capture all of them with one variable we use `...` operator

   ```js
   function sumAll(...args) {
   	// args is the name for the array
   	let sum = 0;
   	for (let arg of args) sum += arg;
   	return sum;
   }

   console.log(sumAll(1)); // 1
   console.log(sumAll(1, 2)); // 3
   console.log(sumAll(1, 2, 3)); // 6
   ```

   We can also choose to get couple of starting parameters as variables and put others in res

   ```js
   function showName(firstName, lastName, ...titles) {
   	console.log(firstName + " " + lastName); // Julius Caesar
   	// the rest go into titles array
   	// i.e. titles = ["Consul", "Imperator"]
   	console.log(titles[0]); // Consul
   	console.log(titles[1]); // Imperator
   	console.log(titles.length); // 2
   }

   showName("Julius", "Caesar", "Consul", "Imperator");
   ```

   **> Rest parameter always must be at the end**

2. In the old times there was no rest paramenter, so we just used `arguments` variable inside the function.

   ```js
   function showName() {
   	console.log(arguments.length);
   	console.log(arguments[0]);
   	console.log(arguments[1]);

   	// it's iterable
   	// for(let arg of arguments) alert(arg);
   }

   // shows: 2, Julius, Caesar
   showName("Julius", "Caesar");

   // shows: 1, Ilya, undefined (no second argument)
   showName("Ilya");
   ```

   `arguments` is array like and iterable, but we cant use array methods on it like `map`, `filter`, etc.

### Spread syntax

1. let say we have an array `[1, 2, 3]` and this can be of any length, and we want to use `Math.max` to find the number, then we can sprea the array by using spread syntax `...`

   ```js
   const arr = [5, 3, 1];
   console.log(Math.max(arr)); // NaN
   console.log(Math.max(...arr)); // 5
   ```

2. With spread syntax we can concatenate arrays too

   ```js
   const a = [1, 2, 3];
   const b = [5, 6, 7];
   const c = [0, ...a, 4, ...b]; // [1, 2, 3, 4, 5, 6, 7]
   ```

   We can do the same with objects.

   ```js
   const b = { one: 1, two: 2 };
   const c = { four: 4, five: 5 };
   const d = { zero: 0, ...b, three: 3, ...c };
   // {zero: 0, one: 1, two: 2, three: 3, four: 4, five: 5}
   ```

3. Spread syntax internally use iterators to gather elements, same as `for...of` does. So `for...of` for a string returns characters, thus spread operators returns array of characters.

   ```js
   const str = "Henlo";
   console.log([...str]); // ['a', 's', 'd', 'f']
   ```

4. We can use spread operator to clone arrays and objects. But remember, in case of objects, it only clone upto first level i.e; nested objects are not cloned.

   ```js
   const a = {
   	one: 1,
   	two: 2,
   	three: {
   		four: 4,
   		five: 5,
   	},
   	six: 6,
   };

   const b = { ...a };

   b.one = 22;
   b.three.four = 666;

   console.log(JSON.stringify(a));
   // {"one":1,"two":2,"three":{"four":666,"five":5},"six":6}
   console.log(JSON.stringify(b));
   // {"one":22,"two":2,"three":{"four":666,"five":5},"six":6}
   ```

### Function Objects

1. Function object contains some usable properties.

   ```js
   function sayHi(user, age) {
   	alert("Hi", user, age);
   }

   alert(sayHi.name); // sayHi
   sayHi.length; // 2 return the length of arguments
   // in length the rest paramenter are not counted
   ```

2. Functions can have custom properties

   ```js
   function sayHi(user, age) {
   	sayHi.count++;
   }

   sayHi.count = 0;

   sayHi();
   sayHi();

   console.log(sayHi.count); // 2
   ```

   In above we check how many times function is called.
   Also function properties can replace closures sometimes.

3. **Named Function Expression**: function expression that has a name.

   ```js
   let sayHi = function () {
   	console.log("hey");
   };
   ```

   Now lets add name to it.

   ```js
   let sayHi = function func() {
   	console.log("hey");
   };
   ```

   Two things can be done with this:

   - It allows the function to reference itself internally.
   - It is not visible outside of the function.

   ```js
   let sayHi = function func(who) {
   	if (who) {
   		alert(`Hello, ${who}`);
   	} else {
   		func("Guest"); // use func to re-call itself
   	}
   };

   sayHi(); // Hello, Guest

   // But this won't work:
   func(); // Error, func is not defined (not visible outside of the function)
   ```

   But we can directly use `sayHi`, then why use this? Because maybe in the above scenatior some changes the `sayHi` variable to something else, but if we use NFE, it won't break the code.

### Function binding

1. We can in some scenatiors loose the context of the `this`.

   ```js
   const user = {
   	name: "sarthak",
   	sayHi() {
   		console.log(`Hi, ${this.name}`); // Hi, undefined
   		// output is for strict mode, for non strict its empty string.
   	},
   };

   setTimeout(user.sayHi, 1000);
   ```

   Now there are 2 ways to solve it

   - **We make a wrapper while calling the function:**

   ```js
   setTimeout(function () {
   	user.sayHi();
   }, 1000);
   ```

   Now this works because it receives `user` outer lexical environment. But what if someone changes the value of `sayHi` before it gets called?

   ```js
   setTimeout(function () {
   	user.sayHi();
   }, 1000);

   user = {
   	sayHi() {
   		console.log("value changed");
   	},
   };
   ```

   - **Function binding:**

   ```js
   // Syntax
   let bound = func.bind(context, [arg1], [arg2], ...);
   ```

   Usage:

   ```js
   let user = {
   	firstName: "John",
   	sayHi() {
   		console.log(`Hello, ${this.firstName}!`);
   	},
   };

   let sayHi = user.sayHi.bind(user); // (*)

   // can run it without an object
   sayHi(); // Hello, John!

   setTimeout(sayHi, 1000); // Hello, John!

   // even if the value of user changes within 1 second
   // sayHi uses the pre-bound value which is reference to the old user object
   user = {
   	sayHi() {
   		console.log("Another user in setTimeout!");
   	},
   };
   ```

2. Partial binding.

   ```js
   function mul(a, b) {
   	return a * b;
   }

   let double = mul.bind(null, 2);

   alert(double(3)); // = mul(2, 3) = 6
   alert(double(4)); // = mul(2, 4) = 8
   alert(double(5)); // = mul(2, 5) = 10
   ```

   This is called "partial function application" – we create a new function by fixing some parameters of the existing one.
   Please note that we actually don’t use `this` here. But bind requires it, so we must put in something like `null`.
   Applications:

   - We can use these partial functions to create independent functions.
   - When we have a generic function and we want to use less universal variant of it.
   - Eg. we have a function `send(from, to, text)`. Then, inside a user object we may want to use a partial variant of it: `sendTo(to, text)` that sends from the current user.

   **Partial without context**

   ```js
   function partial(func, ...argsBound) {
   	console.log(argsBound); // ["10:00"]
   	return function (...args) {
   		console.log(args); // ["Hello"]
   		return func.call(this, ...argsBound, ...args);
   	};
   }

   // Usage:
   let user = {
   	firstName: "John",
   	say(time, phrase) {
   		console.log(`[${time}] ${this.firstName}: ${phrase}!`);
   	},
   };

   // add a partial method with fixed time
   user.sayNow = partial(user.say, new Date().getHours() + ":" + new Date().getMinutes());

   user.sayNow("Hello");
   // Something like:
   // [10:00] John: Hello!
   ```

   The result of partial(func[, arg1, arg2...]) call is a wrapper that calls func with:

   - Same this as it gets (for user.sayNow call it’s user)
   - Then gives it ...argsBound – arguments from the partial call ("10:00")
   - Then gives it ...args – arguments given to the wrapper ("Hello")

### Old "var"

1. "var" is like "let", in many situations you can replace them with one another and expects things to work.

2. "var" has no block scope.

   ```js
   if (true) {
   	var thing = true;
   }
   console.log(thing); // true, still exists in outer scope.
   ```

   ```js
   for (var i = 0; i <= 10; i++) {
   	var one = 1;
   }
   console.log(i); // 10
   console.log(one); // 1
   ```

3. If code block is inside a function then "var" becomes function level.

   ```js
   function sayHi() {
   	if (true) {
   		var phrase = "Hello";
   	}

   	console.log(phrase); // works
   }

   sayHi();
   console.log(phrase); // ReferenceError: phrase is not defined
   ```

4. tolerates redeclarations.

   ```js
   var one = 1;
   var one = "one";
   // Doesn't throws error
   ```

5. can be declared below their use.

   ```js
   function sayHi() {
   	phrase = "Hello";
   	console.log(phrase); // Hello
   	var phrase;
   }
   sayHi();
   ```

   Above is because of "Hoisting".
   But Declarations are hoisted, assignments are not.

   ```js
   function sayHi() {
   	var phrase; // declaration works at the start...
   	alert(phrase); // undefined
   	phrase = "Hello"; // ...assignment - when the execution reaches it.
   }
   sayHi();
   ```

### IFFE

1. These are immediately run functions. There are different ways to write and IFFE:

   ![IFFE Syntax](./images/js-iffe-syntax.png)

### Scheduling:

1. **`setTimeout`**

   - syntax: `let timerId = setTimeout(func|code, [delay], [arg1], [arg2], ...)`

   ```js
   function hello(greeting, name) {
   	console.log(greeting, name);
   }

   setTimeout(hello, 1000, "Henlo", "Sarthak");
   ```

   - Every timemout returns a timeout identifier `timerId`. We can use this to cancel the timeout

   ```js
   let timerId = setTimeout(func, 1000);
   clearTimeout(timerId);
   ```

2. **`setInterval`**

   - Syntax: `let timerId = setInterval(func|code, [delay], [arg1], [arg2], ...)`
   - interval also returns id which can be used to cancel it by using `clearInterval`.

3. We can use nested `setTimeout` instead of `setInterval` also:

   ```js
   let timerId = setTimeout(function tick() {
   	console.log("tick");
   	timerId = setTimeout(tick, 1000);
   }, 1000);
   ```

   This way is more flexible than using `setInterval` as other timeouts can be scheduled with different timers.
   Eg. lets say we want to send a request to server every 2 sec, but if server request fails we want to increase the timer to 4, 6, 8 then so on.

   ```js
   let delay = 2000;
   let timerId = setTimeout(function request() {
   	// request code
   	if (true) {
   		// request fails
   		delay += 2000;
   	}
   	timerId = setTimeout(request, delay);
   }, delay);
   ```

   - nested `setTimeout` is more precise in some scenarios. Eg. Lets say we want to send a request with 100ms delay, but the delay must happen only after the request is completed.

   First we use `setInterval`:

   ```js
   let timerId = setInterval(function () {
   	// a function that sends request
   }, 100);
   ```

   Now in the above scenario, lets assume that the requesting function takes a long time, longer than 100ms, so what will happen is that the function will run second time even before the first request is completed.

   In comes nested `setTimeout` to the rescue.

   ```js
   let timerId = setTimeout(function request() {
   	// function that takes more than 100ms
   	timerId = setTimeout(request, 100);
   }, 100);
   ```

4. Garbage collection: an internal reference is created to the passed function so its not garbage collected in both `setTimeout` and `setInterval`, even if there are no other references to it.

   - for `setTimeout`, function stays in the memory until the scheduler calls it.
   - for `setInterval`, function says in the memory until `clearInterval` is called.
   - Also, the function references out lexical environment, so while it lives, outer variable lives too, thus more memory is used. So we don't need the scheduler, better clear it.

5. Zero delay is infact not zero (in a browser). In browser, theres a limitation how many nested timers can run, after 5 nested timers, delay becomes atleast 4ms.

   ```js
   let start = Date.now();
   let times = [];

   setTimeout(function run() {
   	times.push(Date.now() - start); // remember delay from the previous call

   	if (start + 100 < Date.now()) console.log(times); // show the delays after 100ms
   	else setTimeout(run); // else re-schedule
   });
   // [1, 1, 1, 1, 1, 1, 5, 10, 14, 18, 22, 27, 31, 35, 39, 44, 48, 52, 56, 61, 65, 69, 73, 78, 82, 86, 90, 95, 99, 103].
   ```

   You see few timers are run immediately, then a delay comes into play.
   Although this limitation is for browsers (client-side javascript) only.

### Property flags and descriptors

1. Object properties, beside values have 3 special attributes (called flags)

   - `writable` – if true, the value can be changed, otherwise it’s read-only.
   - `enumerable` – if true, then listed in loops, otherwise not listed.
   - `configurable` – if true, the property can be deleted and these attributes can be modified, otherwise not.

   When we create properties in normal way, all three of them are true. But we can change them.

2. Syntax:

   - `Object.getOwnPropertyDescriptor(obj, "keyName")` - Get them for particular property whose key you passed
   - `Object.getOwnPropertyDescriptors(obj)` - Gets them for all properties
   - `Object.defineProperty(obj, "keyName", {value: "something", writable: true, enumerable: true, configurable: true})` - To define them
   - `Object.defineProperties(obj, {prop1: descriptors1, prop2: descriptors2})` - to define multiple properties

3. Lets say we make a property non writable and the tries to give it a value, it wont give error. To throw an error try to do that same in strict mode.

4. Non configurable is set to false for some of the built-in objects like Math.PI cant be over written. We can't do `Math.PI = 4.1`. Also we cant make Math.PI to be writable again with `Object.defineProperty(Math, "PI", { writable: true });` because its not configurable.

   - Making a property non configurable in a one-way road. We can't change them back with `defineProperty`
   - non configurable makes changes of property flags and its deletion, while allowing to change its value.

   ```js
   let user = {
   	name: "John",
   };

   Object.defineProperty(user, "name", {
   	writable: false,
   	configurable: false,
   });

   // won't be able to change user.name or its flags
   // all this won't work:
   user.name = "Pete";
   delete user.name;
   Object.defineProperty(user, "name", { value: "Pete" });
   ```

   We can change writable: true to false for a non-configurable property, thus preventing its value modification (to add another layer of protection). Not the other way around though.

5. When we clone properties, through any normal way, either by defining empty object and looping over and setting or structured clone, it clones the properties and values but never the descriptors. So to copy over the flags too, we can clone object with `let clone = Object.defineProperties({}, Object.getOwnPropertyDescriptors(obj));`

6. Some in built methods which uses these flags:

   - `Object.preventExtensions(obj)`: Forbids the addition of new properties to the object.
   - `Object.seal(obj)`: Forbids adding/removing of properties. Sets configurable: false for all existing properties.
   - `Object.freeze(obj)`: Forbids adding/removing/changing of properties. Sets configurable: false, writable: false for all existing properties.

   And also there are tests for them:

   - `Object.isExtensible(obj)`: Returns false if adding properties is forbidden, otherwise true.
   - `Object.isSealed(obj)`: Returns true if adding/removing properties is forbidden, and all existing properties have configurable: false.
   - `Object.isFrozen(obj)`: Returns true if adding/removing/changing properties is forbidden, and all current properties are configurable: false, writable: false.

### Property getters and setters.

1. There are two kinds of object properties.

- The first kind is data properties. We already know how to work with them. All properties that we’ve been using until now were data properties.

- The second type of property is something new. It’s an accessor property. They are essentially functions that execute on getting and setting a value, but look like regular properties to an external code.

2. **Getters and Setters**

   ```js
   let user = {
   	name: "John",
   	surname: "Smith",

   	get fullName() {
   		return `${this.name} ${this.surname}`;
   	},

   	set fullName(value) {
   		[this.name, this.surname] = value.split(" ");
   	},
   };

   // set fullName is executed with the given value.
   user.fullName = "Alice Cooper";

   alert(user.name); // Alice
   alert(user.surname); // Cooper
   ```

3. **Descriptors for accessors**
   For accessor properties, there is no value or writable, but instead there are get and set functions.

   That is, an accessor descriptor may have:

   - `get` – a function without arguments, that works when a property is read,
   - `set` – a function with one argument, that is called when the property is set,
   - `enumerable` – same as for data properties,
   - `configurable` – same as for data properties.

   ```js
   let user = {
   	name: "John",
   	surname: "Smith",
   };

   Object.defineProperty(user, "fullName", {
   	get() {
   		return `${this.name} ${this.surname}`;
   	},

   	set(value) {
   		[this.name, this.surname] = value.split(" ");
   	},
   });

   alert(user.fullName); // John Smith

   for (let key in user) alert(key); // name, surname
   ```

4. A property can either be an accessor or data property, not both. If we try to give both values to it then, it gives error.

   ```js
   // Error: Invalid property descriptor.
   Object.defineProperty({}, "prop", {
   	get() {
   		return 1;
   	},

   	value: 2,
   });
   ```

5. Getters/Setters can be used to get control over properties.

   ```js
   let user = {
   	get name() {
   		return this._name;
   	},

   	set name(value) {
   		if (value.length < 4) {
   			alert("Name is too short, need at least 4 characters");
   			return;
   		}
   		this._name = value;
   	},
   };

   user.name = "Pete";
   alert(user.name); // Pete

   user.name = ""; // Name is too short...
   ```

   Now obviously from outside the `_name` can be accessed with `user._name`. But there is a widely known convention that properties starting with an underscore "\_" are internal and should not be touched from outside the object.

6. Eg. Lets say we have a object

   ```js
   class User {
   	constructor(name, age) {
   		this.name = name;
   		this.age = age;
   	}
   }
   ```

   Now any instance of this object has will use age like `user.age`. But lets say in the future we decide that the age is not a good way to store we want it to be dynamic, so we now want to store the user birthday.

   ```js
   class User {
   	constructor(name, birthday) {
   		this.name = name;
   		this.birthday = birthday;
   	}
   }
   ```

   In this now wherever we are using the `.age` will break. To fix this we will create a getter inside this.

   ```js
   class User {
   	constructor(name, birthday) {
   		this.name = name;
   		this.birthday = birthday;
   	}

   	get age() {
   		const todaysYear = new Date().getFullYear();
   		return todaysYear - this.birthday.getFullYear();
   	}
   }
   ```

   Now in the above `.age` will still work.

### Prototypal inheritance

1. In javascript every object has a hidden property `[[Prototype]]` that is either `null` or references to another object. That object is called "a prototype". Every time javascript tries to read a property from an object and its missing, javascript takes it from its prototype. This is called "Prototypal Inheritance".

2. We can set a prototype by using `__proto__`.

   ```js
   let animal = {
   	eats: true,
   };
   let rabbit = {
   	jumps: true,
   };

   rabbit.__proto__ = animal; // sets rabbit.[[Prototype]] = animal
   console.log(rabbit.jumps); // true -> bcoz it exists
   console.log(rabbit.eats); // true -> its inherited
   ```

3. Limitations:

   - The references can't go in circle, else it throws error.
   - the value of `__proto__` can either be object or null, other types are ignored.

4. `__proto__` is a historical getter/setter for `[[Prototype]]`. This works in all environments including server-side. Modern javascript suggests that we should use `Object.getPrototypeOf/Object.setPrototypeOf` instead of this get/set one.

   ```js
   const livingBeing = {
   	breathes: true,
   };
   const person = {
   	walks: true,
   };
   Object.setPrototypeOf(person, livingBeing);
   console.log(person.walks); // true (own property)
   console.log(person.breather); // true (inherited property)
   console.log(Object.getPrototypeOf(person)); // {breathes: true}
   ```

5. Writing don't use prototype, its only used for reading. We can overwrite the properties too, so same properties can exist in both objects. The one directly in the object that is being called will be used. Though accessor properties are exceptions, as assingments are handled by a setter function. So writing to such a property is actually the same as calling a function.

   ```js
   let user = {
   	name: "John",
   	surname: "Smith",

   	set fullName(value) {
   		[this.name, this.surname] = value.split(" ");
   	},

   	get fullName() {
   		return `${this.name} ${this.surname}`;
   	},
   };

   let admin = {
   	__proto__: user,
   	isAdmin: true,
   };

   console.log(admin.fullName); // John Smith

   // setter triggers!
   admin.fullName = "Alice Cooper";

   console.log(admin.fullName); // Alice Cooper, state of admin modified
   console.log(user.fullName); // John Smith, state of user protected
   ```

6. `this` is not affected by prototypes at all. No matter where the method is found: in an object or its prototype. In a method call, `this` is always the object before the dot.

7. `for...in` loops over the inherited properties too

   ```js
   let animal = {
   	eats: true,
   };

   let rabbit = {
   	jumps: true,
   	__proto__: animal,
   };

   // Object.keys only returns own keys
   console.log(Object.keys(rabbit)); // jumps

   // for..in loops over both own and inherited keys
   for (let prop in rabbit) console.log(prop); // jumps, then eats
   ```

8. To check wether a property exists only in the one that is being called with and not in the prototype, we can use `Object.hasOwnProperty(...)` or in modern browsers `Object.hasOwn(...)`.

   ```js
   const base = { one: "1", two: "2" };
   const child = { three: "3", __proto__: base };

   Object.hasOwn(child, "three"); // true
   Object.hasOwn(child, "two"); // false

   "three" in child; // true
   "two" in child; // true
   ```

### Native prototype

1. Pretty much all built in things have some prototypes, called native prototype

   ```js
   const arr = [1, 2, 3];
   console.log(arr.__proto__ === Array.prototype); // true
   console.log(arr.__proto__.__proto__ === Object.prototype); // true
   ```

2. Other built-in objects also work the same way. Even functions – they are objects of a built-in Function constructor, and their methods (call/apply and others) are taken from `Function.prototype`. Functions have their own `toString` too.

   ```js
   function f() {}
   console.log(f.__proto__ == Function.prototype); // true
   console.log(f.__proto__.__proto__ == Object.prototype); // true, inherit from objects
   ```

3. **Primitives**

   - String, Number and Boolean - they are not objects. But if we try to access their properties, temporary wrapper objects are created using built-in constructors String, Number and Boolean. They provide the methods and disappear. We can get them using `String.prototype`, `Number.prototype` and `Boolean.prototype`.

4. **`null`** and **`undefined`** have no object wrappers. There are no methods, properties or prototypes.

5. We can change the Native prototype:

   ```js
   String.prototype.show = function () {
   	console.log(this);
   };
   "Hey".show(); // Hey
   ```

   But this is generally a bad practice, what if there is another library which makes the same show method? One will overwrite other.

   There is only one scenario where modifying the native prototype makes sense. Thats polyfilling.

   ```js
   if (!String.prototype.repeat) {
   	// if there's no such method
   	// add it to the prototype
   	String.prototype.repeat = function (n) {
   		// repeat the string n times

   		// actually, the code should be a little bit more complex than that
   		// (the full algorithm is in the specification)
   		// but even an imperfect polyfill is often considered good enough
   		return new Array(n + 1).join(this);
   	};
   }

   console.log("La".repeat(3)); // LaLaLa
   ```

6. Borrowing from prototypes also works, where we take method from one object and copy it into another. Lets say we make arra like object and want to use array method of `join(...)`

   ```js
   let obj = {
   	0: "Hello",
   	1: "world!",
   	length: 2,
   };

   obj.join = Array.prototype.join;

   console.log(obj.join(",")); // Hello,world!
   ```

   It works only because `join` just cares about the correct indices and length property.

   Another possibility is to inherit by setting obj.**proto** to Array.prototype, so all Array methods are automatically available in obj.

### Prototype methods

1. the modern methods are

   - `Object.getPrototypeOf(obj)`: returns the protoype of object.
   - `Object.setPrototypeOf(obj, somePrototype)`: sets the prototype of `obj` to `somePrototype`.

2. Now while creating object we can give the prototype with the `__proto__`. Instead in modern ways we can use `Object.create(...)`.

   ```js
   const animal = {
   	breathes: true,
   };

   const rabbit = Object.create(animal);

   rabbit.jumps = true;

   console.log(rabbit.jumps); // true
   console.log(rabbit.breathes); // true
   ```

   But `Object.create(...)` has another argument by which we can set properties descriptors.

   ```js
   let animal = {
   	eats: true,
   };

   let rabbit = Object.create(animal, {
   	jumps: {
   		value: true,
   	},
   });

   alert(rabbit.jumps); // true
   ```

3. For exact cloning, including all properties: enumerable, non-enumerable, data properties and getters/setters - everything with the right prototype.

   ```js
   let clone = Object.create(Object.getPrototypeOf(obj), Object.getOwnPropertyDescriptors(obj));
   ```

   The above is way better than cloning using the `for...in`.

4. **Don't change the `[[Prototype]]` method**: We only set prototype once when object is created. Javascript engines are highly optimized for this, so changing them with `Object.setPrototypeOf` or `obj.__proto__ = ` is very slow and breaks the internal optimizations for object property access operations. So avoid it unless you know what you’re doing, or JavaScript speed totally doesn’t matter for you.

5. As we know we can assign only `null` or `object` to prototype. What if we want to assign a string to it?

   ```js
   const obj = {};
   obj.__proto__ = "some string"l
   console.log(obj.__proto__) // {__define.....
   ```

   There is another way to create an object with `null` prototype.

   ```js
   const obj = Object.create(null);
   ```

   So the above don't work. The problem is `__proto__` is not a property of an object, rather its an accessor property of the `Object.prototype`, thats why the above don't work because getter/setter takes care of the assignments.

   But theres a trick we can do to overcome that. We assign the prototype to `null` first, because everything other than `null` and `object` is ignored, and when null is assigned the inherited getter/setter are no more there. Now `__proto__` starts acting like a property of an object.

   ```js
   const obj = {};
   obj.__proto__ = null;
   obj.__proto__ = "some string";
   console.log(obj.__proto__); // some string
   ```

   These objects are called "Very Plain" or "Pure Dictionary" objects, as they are even simpler than normal objects. A downside is that all the in-built methods are no more available to them eg. `toString(...)`.

   ```js
   const obj = Object.create(null);
   String(obj); // TypeError: Cannot convert object to primitive value
   ```

   Note that most object-related methods are Object.something(...), like Object.keys(obj) – they are not in the prototype, so they will keep working on such objects:

   ```js
   const obj = Object.create(null);
   console.log(Object.keys(obj)); // []
   ```

### Class

1. syntax:

   ```js
   class User {
   	constructor(name) {
   		this.name = name;
   	}
   	henlo() {
   		return `Hi, ${this.name}`;
   	}
   }

   const user1 = new User("Sarthak");
   console.log(user1.henlo()); // Hi, Sarthak
   ```

   When `new User("Sarthak")` is called:

   - New object is created.
   - constructor runs with given arguments and assigns it to `this.name`.

2. Classes are functions

   ```js
   console.log(typeof User); // function
   ```

   What `new User() {...}` does is:

   - Creates a function named `User`, that becomes the result of the class declaration. The function code is taken from the constructor method (assumed empty if we don’t write such method).
   - Stores class methods, in `User.prototype`.

   We can make the same User class from functions too.

   ```js
   function User(name) {
   	this.name = name;
   }

   User.prototype.henlo = function () {
   	return `Henlo from function, ${this.name}`;
   };

   const user1 = new User("Sarthak");
   console.log(user1.henlo()); // henlo from function, Sarthak
   ```

3. ```js
   class User {
   	constructor(name) {
   		this.name = name;
   	}
   	sayHi() {
   		console.log(this.name);
   	}
   }

   // class is a function
   console.log(typeof User); // function

   // ...or, more precisely, the constructor method
   console.log(User === User.prototype.constructor); // true

   // The methods are in User.prototype, e.g:
   console.log(User.prototype.sayHi); // the code of the sayHi method

   // there are exactly two methods in the prototype
   console.log(Object.getOwnPropertyNames(User.prototype)); // constructor, sayHi
   ```

4. class is not just a syntactic sugar over functions, there are some differences.

   - First, a function created by class is labelled by a special internal property `[[IsClassConstructor]]: true`. So it’s not entirely the same as creating it manually. The language checks for that property in a variety of places. For example, unlike a regular function, it must be called with `new`:

     ```js
     class User {
     	constructor() {}
     }

     console.log(typeof User); // function
     User(); // Error: Class constructor User cannot be invoked without 'new'
     ```

     Also, a string representation of a class constructor in most JavaScript engines starts with the “class…”

     ```js
     class User {
     	constructor() {}
     }

     console.log(User); // class User { ... }
     ```

   - Class methods are non-enumerable. A class definition sets enumerable flag to false for all methods in the "prototype".
   - Classes always use strict. All code inside the class construct is automatically in strict mode.

5. **Class Expressions:**

   ```js
   let User = class {
   	sayHi() {
   		console.log("Hello");
   	}
   };
   ```

   ```js
   let User = class MyClass {
   	sayHi() {
   		console.log(MyClass); // MyClass name is visible only inside the class
   	}
   };

   new User().sayHi(); // works, shows MyClass definition

   console.log(MyClass); // error, MyClass name isn't visible outside of the class
   ```

6. **Getter/Setters:**

   ```js
   class User {
   	constructor(name) {
   		// invokes the setter
   		this.name = name;
   	}

   	get name() {
   		return this._name;
   	}

   	set name(value) {
   		if (value.length < 4) {
   			console.log("Name is too short.");
   			return;
   		}
   		this._name = value;
   	}
   }

   let user = new User("John");
   console.log(user.name); // John

   user = new User(""); // Name is too short.
   ```

7. **Computed Names:**

   ```js
   class User {
   	["say" + "Hi"]() {
   		alert("Hello");
   	}
   }

   new User().sayHi();
   ```

8. Making bound functions:

   ```js
   class Button {
   	constructor(value) {
   		this.value = value;
   	}

   	click() {
   		console.log(this.value);
   	}
   }

   let button = new Button("hello");

   setTimeout(button.click, 1000); // undefined
   ```

   There are 2 ways we can solve this,

   - Pass a wrapper-function, such as `setTimeout(() => button.click(), 1000)`.
   - Bind the method to object, e.g. in the constructor.

   But class fields provide another method.

   ```js
   class Button {
   	constructor(value) {
   		this.value = value;
   	}
   	click = () => {
   		console.log(this.value);
   	};
   }

   let button = new Button("hello");

   setTimeout(button.click, 1000); // hello
   ```

### Class Inheritance

1. inheritance works with `extends` keyword

   ```js
   class Animal {
   	constructor(name) {
   		this.name = name;
   	}

   	walks() {
   		console.log(`${this.name} WALKS`);
   	}
   }

   class Dog extends Animal {
   	barks() {
   		console.log(`${this.name} BARKS`);
   	}
   }
   ```

   Now couple of points:

   - Dog will have both the methods walks and barks.
   - If we write walks methods in Dog class also, it will override the one written in Animal
   - Internally extends works like prototype methods. `Dog.prototype` will have bark method in it and `Dog.prototype.__proto__` will have methods of Animal.
   - Lets say we dont want to override the method of parent, use it as it is and do something other than that too, we can use `super`

     ```js
     class Dog extends Animal {
     	barks() {
     		console.log(`${this.name} BARKS`);
     	}

     	walks() {
     		super.walks(); // this will call the method of parent too
     		console.log(`${this.name} more like run.`);
     	}
     }
     ```

2. In the above code dog don't have its own constructor, when that happens an empty constructor is implemented like below

   ```js
   class Dog extends Animal {
   	constructor(...args) {
   		super(...args);
   	}
   }
   ```

   We can also add constructor

   ```js
   class Dog extends Animal {
   	constructor(name, ears) {
   		super(name);
   		this.ears = ears;
   	}
   }
   ```

   this wont work with `this.name = name` because constructors in inheriting classes must call `super(...)`, and (!) do it before using this.

3. overriding class fields

   ```js
   class Animal {
   	name = "animal";

   	constructor() {
   		console.log(this.name); // (*)
   	}
   }

   class Rabbit extends Animal {
   	name = "rabbit";
   }

   new Animal(); // animal
   new Rabbit(); // animal
   ```

   ```js
   class Animal {
   	showName() {
   		// instead of this.name = 'animal'
   		console.log("animal");
   	}

   	constructor() {
   		this.showName(); // instead of console.log(this.name);
   	}
   }

   class Rabbit extends Animal {
   	showName() {
   		console.log("rabbit");
   	}
   }

   new Animal(); // animal
   new Rabbit(); // rabbit
   ```

   Above 2 scenarios work in different ways.

   - `Rabbit` is derived class so no `constructor`, so parents `constructor` is used.
   - `Rabbit` calls super thus executing parents constructor,
   - At the time of the parent constructor execution, there are no `Rabbit` class fields yet, that’s why `Animal` fields are used.
   - this thing works because of methods, if we used functions for `showName`, it will work similar to the fields. Eg.
     ```js
     showName = function () {
     	console.log("rabbit");
     };
     ```
     doing it like above will output animal in both scenarios.

4. **super(...):**

   - JavaScript adds one more special internal property for functions: `[[HomeObject]]`.
   - When a function is specified as a class or object method, its `[[HomeObject]]` property becomes that object.

   ```js
   let animal = {
   	name: "Animal",
   	eat() {
   		console.log(`${this.name} eats.`);
   	},
   };

   let rabbit = {
   	__proto__: animal,
   	eat() {
   		// ...bounce around rabbit-style and call parent (animal) method
   		this.__proto__.eat.call(this); // (*)
   	},
   };

   let longEar = {
   	__proto__: rabbit,
   	eat() {
   		// ...do something with long ears and call parent (rabbit) method
   		this.__proto__.eat.call(this); // (**)
   	},
   };

   longEar.eat(); // Error: Maximum call stack size exceeded
   ```

   The above code wont work because:

   - `longEar` calls the rabbit through prototype.
   - in `rabbit`, `this` becomes `longEar` because of `call(...)` function.
   - `rabbit` calls itself again and again, thus ending in a endless loop.

   Solution?

   ```js
   let animal = {
   	name: "Animal",
   	eat() {
   		// animal.eat.[[HomeObject]] == animal
   		console.log(`${this.name} eats.`);
   	},
   };

   let rabbit = {
   	__proto__: animal,
   	name: "Rabbit",
   	eat() {
   		// rabbit.eat.[[HomeObject]] == rabbit
   		super.eat();
   	},
   };

   let longEar = {
   	__proto__: rabbit,
   	name: "Long Ear",
   	eat() {
   		// longEar.eat.[[HomeObject]] == longEar
   		super.eat();
   	},
   };

   // works correctly
   longEar.eat(); // Long Ear eats.
   ```

### Static properties and methods

1. to create static properties or methods we use `static` keyword before them

   ```js
   class Animal {
   	static breathes = "haan";
   	static count = 0;

   	constructor(name, age) {
   		this.name = name;
   		this.age = age;
   		Animal.count++;
   	}

   	about() {
   		console.log(`kya ${this.name} saans leta hai: ${Animal.breathes}`);
   	}

   	static aMethod() {
   		return "Static method called";
   	}

   	static sortByAge(animal1, animal2) {
   		return animal1.age - animal2.age;
   	}
   }

   const a = new Animal("one", 3);
   const b = new Animal("two", 1);
   const c = new Animal("three", 2);

   const x = [a, b, c];

   x.sort(Animal.sortByAge);

   console.log(x[0].name); // two
   ```

   - Static stuff dont belong to object or instance of class, it actually belongs to the whole class.
   - We cannot use them like `a.aMethod()`, it should be used like `Animal.aMethod()`.
   - In above example `count` property can be used for getting the number of times class has been instantiated.
   - They work perfecty fine with inheritance too, so one can extend from `Animal` and still these will work.

     ```js
     class Dog extends Animal {
     	something() {
     		console.log("Dog does something");
     	}
     }

     const doggus = [new Dog("Ralph", 4), new Dog("Mike", 8)];

     doggus.sort(Animal.sortByAge);
     console.log(doggus[0].name); // Ralph
     ```

   - With the help of these we get another way to instantiate a new class.

     ```js
     class Article {
     	constructor(title, date) {
     		this.title = title;
     		this.date = date;
     	}

     	static createTodays() {
     		// remember, this = Article
     		return new this("Today's digest", new Date());
     	}
     }

     let article = Article.createTodays();

     alert(article.title); // Today's digest
     ```

### Private and Protected properties and methods.

1. Public properties can be access from anywhere, Protected can be access from class where its defined and from classes which extends this class, Private can only be accessed from the within the class.

2. To define private ones we use "#" as prefix.

   ```js
   class CoffeeMachine {
   	#waterLimit = 200;

   	#fixWaterAmount(value) {
   		if (value < 0) return 0;
   		if (value > this.#waterLimit) return this.#waterLimit;
   	}

   	setWaterAmount(value) {
   		this.#waterLimit = this.#fixWaterAmount(value);
   	}
   }

   let coffeeMachine = new CoffeeMachine();

   // can't access privates from outside of the class
   coffeeMachine.#fixWaterAmount(123); // Error
   coffeeMachine.#waterLimit = 1000; // Error
   ```

3. Private ones dont conflict with public ones

   ```js
   class CoffeeMachine {
   	#waterAmount = 0;

   	get waterAmount() {
   		return this.#waterAmount;
   	}

   	set waterAmount(value) {
   		if (value < 0) value = 0;
   		this.#waterAmount = value;
   	}
   }

   let machine = new CoffeeMachine();

   machine.waterAmount = 100;
   alert(machine.#waterAmount); // Error
   ```

4. To define protected one we use "\_" as prefix. Now caveat is, properties with underscore "\_" prefix can be accessed from anywhere, its just that its an unwritten rule that developers don't try to get any property with that prefix.

   ```js
   class CoffeeMachine {
   	_waterAmount = 0;

   	set waterAmount(value) {
   		if (value < 0) {
   			value = 0;
   		}
   		this._waterAmount = value;
   	}

   	get waterAmount() {
   		return this._waterAmount;
   	}

   	constructor(power) {
   		this._power = power;
   	}
   }

   // create the coffee machine
   let coffeeMachine = new CoffeeMachine(100);

   // add water
   coffeeMachine.waterAmount = -10; // _waterAmount will become 0, not -10
   ```

### Mixins

1. Lets say some class extends from some another class, but there is another object whose functionality we want to use. Now because we can only extend from one class, this is limiting. In comes Mixins.

   ```js
   class Hi {
   	sayHi() {
   		console.log("Hi");
   	}
   }

   class Bye extends Hi {
   	sayBye() {
   		console.log("Bye");
   	}
   }

   const GeneralTalkMixin = {
   	saySomething() {
   		console.log("SOMETHING");
   	},
   };

   Object.assign(Bye.prototype, GeneralTalkMixin.prototype);

   const b = new Bye();
   b.sayHi(); // Hi
   b.sayBye(); // Bye
   b.saySomething(); // SOMETHING
   ```

   In the above example, `Bye` extends `Hi` but also can use `GeneralTalkMixin` functions.

### Extending built-in classes.

1. We can extend built in classes like Arrays, Object, etc

   ```js
   class PowerArray extends Array {
   	isEmpty() {
   		return this.length === 0;
   	}
   }

   let arr = new PowerArray(1, 2, 5, 10, 50);
   console.log(arr.isEmpty()); // false

   let filteredArr = arr.filter((item) => item >= 10);
   console.log(filteredArr); // 10, 50
   console.log(filteredArr.isEmpty()); // false
   ```

   - Built-in methods like `filter`, `map` and others – return new objects of exactly the inherited type `PowerArray`. Their internal implementation uses the object’s constructor property for that.
   - In this example `arr.prototype === PowerArray`.
   - When `arr.filter()` is called, it internally creates the new array of results using exactly `arr.constructor`, not basic Array. That’s actually very cool, because we can keep using PowerArray methods further on the result.

   Although we can customize this behavior by adding special static getter `Symbol.species` to the class.

   ```js
   class PowerArray extends Array {
   	isEmpty() {
   		return this.length === 0;
   	}

   	// built-in methods will use this as the constructor
   	static get [Symbol.species]() {
   		return Array;
   	}
   }

   let arr = new PowerArray(1, 2, 5, 10, 50);
   console.log(arr.isEmpty()); // false

   // filter creates new array using arr.constructor[Symbol.species] as constructor
   let filteredArr = arr.filter((item) => item >= 10);

   // filteredArr is not PowerArray, but Array
   console.log(filteredArr.isEmpty()); // Error: filteredArr.isEmpty is not a function
   ```

   - Now `.filter` returns `Array`. So the extended functionality is not passed any further.

2. Important point:
   - Built-in objects have their own static methods, for instance `Object.keys`, `Array.isArray` etc.
   - native classes extend each other. For instance, `Array` extends `Object`.
   - when one class extends another, both static and non-static methods are inherited.
   - But built-in classes are an exception. They don’t inherit statics from each other.
   - For example, `Array` inherit from `Object`, so their instances have methods from `Object.prototype`. But `Array.[[Prototype]]` does not reference `Object`, so there’s no, for instance, `Array.keys()` static method.

## `instanceof`

1. Basic Syntax:

   ```js
   class Rabbit {}
   let rabbit = new Rabbit();

   // is it an object of Rabbit class?
   console.log(rabbit instanceof Rabbit); // true
   ```

   or with constructor functions

   ```js
   function Rabbit() {}

   console.log(new Rabbit() instanceof Rabbit); // true
   ```

   or with built in classes

   ```js
   let arr = [1, 2, 3];
   console.log(arr instanceof Array); // true
   console.log(arr instanceof Object); // true
   ```

2. `instanceof` checks the prototype chain, but we can also set custom logic using the static function `Symbol.hasInstance`

   ```js
   class Animal {
   	static [Symbol.hasInstance](obj) {
   		if (obj.canEat) return true;
   	}
   }

   let obj = { canEat: true };

   console.log(obj instanceof Animal); // true: Animal[Symbol.hasInstance](obj) is called
   ```

   Anything with canEat property is an animal

3. Most classes don't have `Symbol.Instance`. In that case, lets say for `obj instanceof Class` it checks whether `Class.prototype` is equal to one of the prototypes in the `obj` prototype chain.

4. There is also `objA.isPrototypeOf(objB)` that returns true if `objA` is somewhere in the prototype chain of the `objB`.

5. Remember `Class` constructor don't participate in the check, only `Class.prototype` and object's prototype chain

6. Weird thing: we can use objects `toString` as extended `typeOf` and an alternate `instanceof`

   ```js
   let s = Object.prototype.toString;

   console.log(s.call(123)); // [object Number]
   console.log(s.call(null)); // [object Null]
   console.log(s.call(alert)); // [object Function]
   console.log(s.call()); // [object Undefined]
   console.log(s.call("asdf")); // [object String]
   ```

   - We can customize this too:

     ```js
     let user = {
     	[Symbol.toStringTag]: "User",
     };

     console.log({}.toString.call(user)); // [object User]
     ```

     ```js
     alert({}.toString.call(window)); // [object Window]
     alert({}.toString.call(new XMLHttpRequest())); // [object XMLHttpRequest]
     ```

   - this is `typeOf` on steroids, this works for both primitive data types and also for built-in objects and can be customzied too.
   - We can use `{}.toString.call` instead of `instanceof` for built-in objects when we want to get the type as a string rather than just to check.

### Error Handling

1. Syntax:

   ```js
   try {
   	// code
   } catch (err) {
   	// err
   } finally {
   	// happens everytime
   }
   ```

   - code runs in `try` block
   - if `error` happens, `catch` block runs
   - `finally` block runs everytime, no matter what happens

2. remember error is only caught when there is error in code, not a syntax error, like:

   ```js
   try {
      {{{
   } catch(err) {
      //
   }
   ```

   in the above error will not go under the catch statement. as `try...catch` just cathces the run time errors.

3. `try...catch` works synchronously.

   ```js
   try {
   	setTimeout(function () {
   		noSuchVariable; // script will die here
   	}, 1000);
   } catch (err) {
   	console.log("won't work");
   }
   ```

   But below will work:

   ```js
   setTimeout(function () {
   	try {
   		noSuchVariable; // try...catch handles the error!
   	} catch {
   		console.log("error is caught here!");
   	}
   }, 1000);
   ```

4. `error` object which is argument of `catch` have 3 things:

   - **name**: name of the error like, `SyntaxError`, `TypeError`, etc
   - **message**: message of the error
   - **stack**: stack of error

   ```js
   try {
   	lalala; // error, variable is not defined!
   } catch (err) {
   	console.log(err.name); // ReferenceError
   	console.log(err.message); // lalala is not defined
   	console.log(err.stack); // ReferenceError: lalala is not defined at (...call stack)

   	// Can also show an error as a whole
   	// The error is converted to string as "name: message"
   	console.log(err); // ReferenceError: lalala is not defined
   }
   ```

5. In a recent addition optional catch is also being added.

   ```js
   try {
   } catch {
   	// <- without err
   }
   ```

   here `catch` will omit the error.

6. There is built in constructors for the the standard errors like, `Error`, `SyntaxError`, `TypeError`, etc

   ```js
   const error = new Error("this is error");
   const referenceError = new ReferenceError("this is another error");
   ```

7. Variables inside, `try`, `catch` and `finally` and local.

8. The `finally` clause works for any exit from `try...catch`. That includes an explicit `return`.

   ```js
   function func() {
   	try {
   		return 1;
   	} catch (err) {
   		/* ... */
   	} finally {
   		console.log("finally");
   	}
   }

   console.log(func());
   // finally
   // 1
   ```

   this outputs "finally" the outputs "1".

9. `try...finally` also works without the catch block.

   ```js
   function func() {
   	// start doing something that needs completion (like measurements)
   	try {
   		// ...
   	} finally {
   		// complete that thing even if all dies
   	}
   }
   ```

   In the code above, an error inside try always falls out, because there’s no catch. But finally works before the execution flow leaves the function.

10. global catch, a global error catcher.

```js
window.onerror = function (message, url, line, col, error) {
	console.log(`asdfasdf ----> ${message}\n At ${line}:${col} of ${url}`);
};

function readData() {
	badFunc(); // Whoops, something went wrong!
}
readData();
console.log("asdf HAHAHAH");
```

The role of global catch is not to stop the faliure of script but rather to send a message to the developer so that maybe dev can log these somewhere.

- We register at the service and get a piece of JS (or a script URL) from them to insert on pages.
- That JS script sets a custom window.onerror function.
- When an error occurs, it sends a network request about it to the service.
- We can log in to the service web interface and see errors.

### Custom errors, extending Error

1. We can make custom error for our own purpose, below is a kind of psuedo code for default `Error` class

   ```js
   class Error {
      constructor(message) {
         this.message = message;
         this.name = "Error"; // (different names for different built-in error classes)
         this.stack = <call stack>; // non-standard, but most environments support it
      }
   }
   ```

2. To make a custom error we extend this error:

   ```js
   class CustomError extends Error {
   	constructor(message) {
   		super(message);
   		this.name = this.constructor.name; // this will give it name of the class
   	}
   }
   ```

   Example, lets say we want to throw error when a json don't a a property we need

   ```js
   /*
      const stringifiedJson = {
         fName: "sarthak",
         lName: "jha",
         age: 30,
      };
   */

   class MissingPropertyError extends Error {
   	constructor(message) {
   		super(message);
   		this.name = this.constructor.name;
   	}
   }

   const readUser = () => {
   	try {
   		const data = JSON.parse(stringifiedJson);

   		if (!data.number) {
   			throw new MissingPropertyError("Missing Property: number");
   		}

   		// console.log("DATA", data);
   	} catch (err) {
   		if (err instanceof MissingPropertyError) {
   			console.log(err.name); // MissingPropertyError
   			console.log(err.message); // Missing Property: number
   			console.log(err.stack); // <stack of error>
   		}
   	}

   	console.log("DONE");
   };

   readUser();
   ```
