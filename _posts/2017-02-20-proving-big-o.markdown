---
layout: post
title: Proving Big-O
date: 2017-02-20 00:12:11.000000000 +00:00
---
# Proving Big-O
 
Asymptotic notation is a set of languages which allows us to express the performance of our algorithm in relation to their input.

There are three main notations for expressing the performance of code, put simply they are: 

* Big-O - The worst case (upper bound).
* Big Omega - The best case (lower bound).
* Theta - The upper and lower bounds.

In this post I'm going to focus on how we can prove Big-O - as we often want to plan for the worst case running time rather than the best.

## Big-O through code

Lets take the following Python example which takes an array, iterates over it and prints each value of the array:

```
def print_values(array):
	for i in array:
    	print i
```

We could put a timer at the beginning and the end of the line of code which calls this function, lets say our input is `[1, 2, 3, 4, 5]`, this would then give us the `running time` of our algorithm, right? 

```
start = time.time()
print_values([1, 2, 3, 4, 5])
end = time.time()
print(end - start)
```

Maybe, but what if you run it again, three times, write down your results and then move to another machine with a higher spec and run it another three times. I bet upon comparison of the results you will get different running times! 

This is where asymptotic notations are important. They provide us with a mathematical foundation for representing the running times of our algorithms consistently. 

We create this consitencey by talking about operations our code has to perform. Operations such as array lookups, print statements and variable assignments. 

If we were to annotate `print_values` with the amount of times each line within the function is executed for the input `[1, 2, 3, 4, 5]`, we would have something as follows:

```
def print_values(array):
	for i in array: # Execution count: 5
    	print i # Execution count: 1
```

If we were to change the input to an array of different size, our print statement would be exercised more or less depending on the size of that input. 

If we were to put this into an arithmetic expression, we would get `1+5`, using intuition we know that the 5 is variable on the input size, if we call the input size `n`, we would now have the expression `1+n`.

I could now argue that the worst case running time for `print_values` is `O(n+1)`. `n` for the loop block and `1` for the print statement. Clearly this is oversimplified and taken at face value as I'm not _really_ interested in _how_ this code is being executed under the hood, I just care about the operations defined solely within my above function.

In the grand scheme of things, the constant value `1` is pretty insignificant at the side of the variable value `n`. So we simply reduce the above expression to `O(n)`, and there we have our Big-O running time of `print_values`.

As our code prints each and every value from the input array, it must in turn look at each value within that array in order to print it, as the loop is the most significant part of the code, we are able to say that our code is of running time `O(n)` where `n` is the variable length of the array! __Simples!__

An algorithm of running time `O(n)` is said to be linear, which essentially means the algorithms running time will increase linearly with its input (`n`). 

## Proving Big-o

We can prove, mathematically, that `print_values` is in-fact `O(n)`, which brings us on to the formal definition for Big-O:

__f(n) = O(g(n))__ if __c__ and some initial value __k__ are positive when __f(n) <= c * g(n) for all n > k__ is true.

We can turn this formal definition into an actual definition of our above code, which we can then in turn prove.
 
We must first ask does `print_values` have a running time of `O(n)`?

If `print_values <= c n`, when `c` = 1 then `print_values` __does__ have a running time of `O(n)` when `n > k`.

`c` can be any integer while `k` is the amount of iterations we must perform for the expression to be true for every subsequent value of `n`.

As `c` is just `1`, we can simplify our expression to `print_values <= n`. 

| n | f(n)    | g(n)     | True/False|
|---|---------|----------|-----------|
| 0 | 0       | 0        | False     |
| 1 | 1       | 1        | True      |
| 2 | 2       | 2        | True      |
| 3 | 3       | 3        | True      |

We can see that `n` must be greater than the value `0` of constant `k` in order to satisfy the expression `print_values <= n`.

We can now say when `n` is `1`: __1 <= 1 * 1 for 1 > 0__ is true. We know this because `1` multiplied by `1` is `1` and `1` is greater than our constant `k` which was `0`.

The above must be true for all values of `n` greater than `k` _(0)_, so if `n` was `10`: __10 <= 1 * 10 for 10 > 0__ is also true.

What we're basically saying here is that no matter our input (`n`), it must be greater than or equal to our constant (`c`) when the size of our input (`n`) is more than another constant value (`k`, in our case the iteration count of the function).

But where do our constants come from? Well they are just values, we typically start at 1 and work our way up to seek a constant which makes the expression __f(n) <= c * g(n) for all n > k__ true. If we cannot find such combination of constants, then our code does not have a running time of `O(n)` and our hypothesise was incorrect.

## Disproving Big-O

Lets take a new Python example which takes an array, iterates over it and then prints the array value as many times as the arrays length.

```
def print_values_with_repeat(array):
	for i in array:
		for x in range(0, len(array)):
    		print i
```

If we were to annotate `print_values_with_repeat` with the amount of times each line within the function is executed for the input `[1, 2, 3, 4, 5]`, we would have something as follows:

```
def print_values_with_repeat(array):
	for i in array: # Execution count: 5
		for x in range(0, len(array)): # Execution count: 25
    		print i # Execution count: 1
```

Does `print_values_with_repeat` have a running time of `O(n)`?

| n | f(n)    | g(n)     | True/False|
|---|---------|----------|-----------|
| 0 | 0       | 0        | False     |
| 1 | 1       | 1        | True      |
| 2 | 4       | 2        | False     |
| 3 | 9       | 3        | False     |

Suppose our constant `c` is `1`, __1 <= 1 * 1 for 1 > 0__, this is true - however our definition says that `g(n)` must be greater than all values of `f(n)`. 

So if we take the value `2` of `n` __2 <= 1 * 4 for 1 > 0__, we can see that this is now false, which disproves our hypothesis that `print_values_with_repeat` is `O(n)`. Even if we change our constant `c` to `2`, this would still prove false _eventually_.

We can actually see that the order of growth in operations in `print_values_with_repeat` is actually `n^2`, so let's hypothesise now that `print_values_with_repeat` is actually `O(n^2)`.

Does `print_values_with_repeat` have a running time of `O(n^2)`?

| n | f(n)    | g(n^2)   | True/False|
|---|---------|----------|-----------|
| 0 | 0       | 0        | False     |
| 1 | 1       | 1        | True      |
| 2 | 4       | 4        | True      |
| 3 | 9       | 9        | True      |

Suppose our constant `c` is still `1`, our expression would now be __3 <= 3 * 3^2 for 3 > 0__, this is true, great! `print_values_with_repeat` is in-fact `O(n^2)`. 

`O(n^2)` is a quadratic time algorithm, as the running time of the algorithm increases quadratically to the input.
 

## Where to next

There are some fantastic resources on the web to further your understanding on asymptotic notation, here are a few:

* [Big Oh Notation (and Omega and Theta) (Video)](https://www.youtube.com/watch?v=ei-A_wy5Yxw&list=PL1BaGV1cIH4UhkL8a9bJGG356covJ76qN&index=2)
* [Asymptotic Nota* tions - Learn X in Y](https://learnxinyminutes.com/docs/asymptotic-notation/)
* [Khan Academy - Algorithms](https://www.khanacademy.org/computing/computer-science/algorithms)

Found something inaccurate or incorrect, [please submit a pull request](https://github.com/imjacobclark/blog.jacobclark.xyz/blob/master/_posts/2017-02-20-proving-big-o.markdown) all improvements will be attributed accordingly. 

Visit my [website](https://www.jacobclark.xyz), follow me on [Twitter](https://twitter.com/imjacobclark) and [GitHub](https://github.com/imjacobclark) or view my professional background on [LinkedIn](https://uk.linkedin.com/in/imjacobclark).
