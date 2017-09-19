---
layout: post
title: Dynamic Arrays in C
date: 2017-08-19 09:21:11.000000000 +00:00
---
If you’re from a dynamic programming language such as JavaScript or Ruby, working with Arrays in C might seem a little weird.

## Introduction

### Arrays in JavaScript:

```
let numbers = [];
numbers.push(1);
numbers.push(2);
numbers.push(3);
console.log(numbers); // [1, 2, 3]
```

In the above example, we created an array and added 3 numbers to it. You’ll notice we didn’t need to specify how many items we planned on putting into the array, we were able to add as many as we like.

### Arrays in C:

```
int numbers[3];
numbers[0] = 1;
numbers[1] = 2;
numbers[2] = 3;
printf("%d\n", numbers[0]); // 1
printf("%d\n", numbers[1]); // 2
printf("%d\n", numbers[2]); // 3
```

You’ll notice in the first line of syntax being `numbers[3]` — here we’re telling the compiler we want an array with enough memory to store three integers. We then go on to store 1, 2 and 3 at their respective indexes within our array and print them — seems simple, right? Problem here though is, we can’t add any more than 3 items to our array.

```
int numbers[3];
numbers[0] = 1;
numbers[1] = 2;
numbers[2] = 3;
numbers[3] = 4;
printf("%d\n", numbers[0]); // 1
printf("%d\n", numbers[1]); // 2
printf("%d\n", numbers[2]); // 3
printf("%d\n", numbers[3]); // should be 4
```

The above example put through `gcc`:

```
array.c:8:5: warning: array index 3 is past the end of the array (which contains 3 elements) [-Warray-bounds]
```

So we effectively end up with an out of bounds exception, the memory location of our array wasn’t big enough to hold any more items.

What if we need a dynamically sized array that may hold `n` items at any one time?

We must implement dynamically sized arrays ourselves in C. They don’t come for free within the language like they do in JavaScript, Ruby or any other dynamic programming languages. We implement them through the use of memory blocks.

### malloc, realloc and pointers

In C every data type has a size — for example; on my x64 Macintosh — an integer is 4 bytes. Just by knowing this information at runtime we can create dynamically growing arrays of any size.

We can grab the size of a type in C by doing `sizeof(int), sizeof(double)` or whatever data type you’re working with at the time, together with malloc and realloc we can create memory blocks which can grow freely in size.
Let’s say we want to start off with the capacity to hold 3 integers, we can do this by allocating a memory block of 12 bytes:

(Remember `malloc` only argument is the size of the memory block in bytes and that `malloc` returns a pointer to the newly created memory block.)

```
#define INITIAL_CAPACITY 3

int main(){
     int* data = malloc(INITIAL_CAPACITY * sizeof(int));
}
```

Now we have a memory block big enough to hold our 3 integers — we need to make it dynamic! Right now, we still have the same problem where we can only place 3items into our memory block.

If we keep track of our memory locations size and capacity — we will be able to calculate when we need to resize it!

If our memory block is full, we can double the size of that memory block by calling `realloc` which simply expands the current memory block.

```
#define INITIAL_CAPACITY 3

void push(int *arr, int index, int value, int *size, int *capacity){
     if(*size > *capacity){
          realloc(arr, sizeof(arr) * 2);
          *capacity = sizeof(arr) * 2;
     }
     
     arr[index] = value;
     *size = *size + 1;
}

int main(){
     int size = 0;
     int capacity = INITIAL_CAPACITY;
     int* arr = malloc(INITIAL_CAPACITY * sizeof(int));
}
```

There you have it —the ability to add items to a memory block dynamically.
If we put it all together we end up with the following program:

```
#include <stdio.h>
#include <stdlib.h>
#define INITIAL_CAPACITY 2

void push(int *arr, int index, int value, int *size, int *capacity){
     if(*size > *capacity){
          realloc(arr, sizeof(arr) * 2);
          *capacity = sizeof(arr) * 2;
     }
     arr[index] = value;
     *size = *size + 1;
}

int main(){
     int size = 0;
     int capacity = INITIAL_CAPACITY;
     int* arr = malloc(INITIAL_CAPACITY * sizeof(int));
     push(arr, 0, 1, &size, &capacity);
     push(arr, 1, 2, &size, &capacity);
     push(arr, 2, 3, &size, &capacity);
     
     printf("Current capacity: %d\n", capacity); // Current capacity: 2
     push(arr, 3, 4, &size, &capacity);
     push(arr, 4, 5, &size, &capacity);
     push(arr, 5, 6, &size, &capacity);
 
     printf("Current capacity: %d\n", capacity); // Current capacity: 16
}
```

Strictly speaking, what we have created here is a vector. A vector is a dynamically sized array with operations such as append, prepend, delete and find.

[Vectorlib](https://github.com/imjacobclark/vectorlib) is a reasonably full implementation of a vector in C. It’s for educational purposes only, don’t use it in production.

Visit my website, follow me on [Twitter](https://twitter.com/imjacobclark) and [GitHub](https://github.com/imjacobclark) or view my professional background on [LinkedIn](https://www.linkedin.com/in/imjacobclark).
