---
layout: post
title: Functional composition in JavaScript
date: 2016-05-05 22:10:42.000000000 +01:00
---
Functional composition - the art of taking one function and applying it to the result of another.

Let's take a look at a pretty standard JavaScript example to start with and work our way to refactoring it into functional composition. 

Below we have a few functions which take inputs and produce outputs:

```javascript
let double = number => number * 2;
let triple = number => number * 3;
let quadruple = number => number * 4;
let number = 5; 

number = double(number);
number = triple(number);
number = quadruple(number);

console.log(number) // => 120

```

Something doesn't sit quite right with me about this code. I'm not particularly fond of the multiple variable reassignments, it feels a little hard to read. 

We could potentially refactor the above into the following:

```javascript
let double = number => number * 2;
let triple = number => number * 3;
let quadruple = number => number * 4;
let number = 5; 

number = quadruple(triple(double(5)));

console.log(number) // => 120

```

However, nesting function calls as arguments doesn't sit quite right with me either, what if I wanted to be able to quintuple pentadruple my numbers and so on so forth? 

We could refactor the above methods to return themselves so we can chain methods - a common design pattern known as the builder:

```javascript
let NumberCruncherBuilder = function(numberToCrunch) {
  let num = numberToCrunch;
   
  return {
      double: () => {
        num = num * 2;
        return this;
      },
      triple: () => {
        num = num * 3;
        return this;
      },
      quadruple: () => {
        num = num * 4;
        return this;
      },
      build: () => num
  };
};

let numberCruncher = new NumberCruncherBuilder(5);
let number = numberCruncher.double().triple().quadruple().build();

console.log(number) // => 120

```

Whilst this is a fine example of utilising the builder pattern to overlay syntactic sugar on top of variable reassignment - it still doesn't feel quite right - we've had to go down the instantiation route to ensure our builder has the right scope attached to `this` which ultimately increases overhead.

Our functions are now all coupled into this one builder too - we could abstract them out but that's even more complexity... plus we now have a somewhat stateful function too as we construct the builder with the initial value that will be built upon, muddying up the readability of our code and making our initial solution seem much simpler (whilst far less scalable).

The next example is functional composition in pure JavaScript - many utility/functional JavaScript libraries include the same functionality - lodash for example has a functional composition method called [_.flow](https://lodash.com/docs#flow).

The below example is technically named a pipe as it reduces the functions left to right (like a unix pipe) - we could use `reduceRight` instead on our spread operator which would reduce the functions right to left in a more functional manor.  

Functional composition will simply pass a value into a function, take the returned value of that function and pass it along - it will do this until there are no further functions it can pass the value along to - finally returning the end value.

```javascript
let double = number => number * 2;
let triple = number => number * 3;
let quadruple = number => number * 4;
let compose = (...funcs) => (value) => funcs.reduce((v,fn) => fn(v), value);

// Arguments are read right to left
// double -> triple -> quadruple
let crunchNumber = compose(
    double,
    triple,
    quadruple
);

let number = crunchNumber(5);

console.log(number);
```

To start - we declare our functions as per the first couple of solutions, however in this solution we also declare a utility function called `compose`. Let's un-crazy that function and break it down:

```javascript
// Declare a function, take functions as arguments
let compose = function(...funcs){ 

    // Return a new function which takes a value
    return function(value){

        // Reduce & iterate the initial argument spread (an array of functions)
        return funcs.reduce(function(val, function){

            // Take the function, call it, passing in the value and return the output
            return function(val)

        // Pass the value into the reduce to be passed into the function to call
        }, value);
    }
}
```

To finish - we call the compose function, passing in all the functions we wish to be 'composed' of one another - this returns another function in which we pass the value we want to be passed into the first function.

Functional composition won't affect your code testability either as it's purely generic and not interested in any specific implementation detail.

Clearly functional composition provides us with an interface to write cleaner and more succinct code - without additional overhead and complexity other approaches can introduce.

Given the new features and syntactic sugar brought in by ES6/7 - I find functional composition sits nicely as a utility function in any web application.

Visit my [website](https://www.jacobclark.xyz), follow me on [Twitter](https://twitter.com/imjacobclark) and [GitHub](https://github.com/imjacobclark) or view my professional background on [LinkedIn](https://uk.linkedin.com/in/imjacobclark).
