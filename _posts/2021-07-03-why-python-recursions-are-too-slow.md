---
layout: post
title: "Why python recursions are too slow?"
date: 2021-07-03 18:15:14 -0700
categories: [programming, performance]
tags: [python, javascript, go, c, recursion, jit, cpython, v8]
---

I am a big fan of micro benchmarking. In this article, we will compare the performance of tail end recursive `Fibonacci(40)` in C, Go, Python3 and javascript.

*Note - A language isn't fast or slow, it's the compiler/interpreter which is responsible for the performance of a language. So for ease of understanding, we will be considering the most popular engines used for compiling/interpreting a particular language.*

![]({{ "/assets/why-python-recursions-are-too-slow/intro.gif" | relative_url }})

Let's jump right in

## Python3

```python
import time
import sys

def fib(n):
    if n <= 1:
        return n
    else:
        return fib(n-1) + fib(n-2)

if __name__ == "__main__":
    number = int(sys.argv[1])
    start_time = time.time()
    fib(number)
    time_taken = time.time() - start_time
    print(f"Time taken = {time_taken}s")
```

![]({{ "/assets/why-python-recursions-are-too-slow/python-output.png" | relative_url }})

![]({{ "/assets/why-python-recursions-are-too-slow/reaction-python.webp" | relative_url }})

27 seconds! This is shocking, but we don't have the execution time of other languages to compare. So let's first find out how much time would a compiled program like C and Go take.

## Go

```go
package main

import (
	"fmt"
	"os"
	"strconv"
	"time"
)

func Fibo(n int) int {
	if n <= 1 {
		return n
	}
	return Fibo(n-1) + Fibo(n-2)
}

func main() {
	number := os.Args[1]
	i, _ := strconv.Atoi(number)
	start := time.Now()
	Fibo(i)
	fmt.Println("Time take -", time.Since(start))
}
```

![]({{ "/assets/why-python-recursions-are-too-slow/go-output.png" | relative_url }})

## In C

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int fibonacci(int n)
{
    if (n == 0 || n == 1)
        return n;
    else
        return (fibonacci(n - 1) + fibonacci(n - 2));
}

int f(int);
int main(int argc, char *argv[])
{
    int n, res;
    double time_spent = 0.0;
    n = atoi(argv[1]);
    clock_t begin = clock();
    res = fibonacci(n);
    clock_t end = clock();
    time_spent += (double)(end - begin) / CLOCKS_PER_SEC;
    printf("The elapsed time is %f seconds", time_spent);
    return 0;
}
```

![]({{ "/assets/why-python-recursions-are-too-slow/c-output.png" | relative_url }})

![]({{ "/assets/why-python-recursions-are-too-slow/reaction-c.webp" | relative_url }})

So compiled languages like golang and C evaluates Fibonacci(40) in about `500-700ms`. We cannot expect an interpreted language to be as fast as compiled one. So let's take another interpreted language(`javascript`) and find the execution time of Fibonacci(40)

## Javascript

```javascript
function fib(n) {
    if (n <= 1) {
        return n;
    } else {
        return fib(n-1) + fib(n-2);
    }
}

number = process.argv[2];
console.time('fibonacci');
fib(number);
console.timeEnd('fibonacci');
```

![]({{ "/assets/why-python-recursions-are-too-slow/js-output.png" | relative_url }})

![]({{ "/assets/why-python-recursions-are-too-slow/reaction-js.gif" | relative_url }})

Well as a python developer, this made me cry. How come javascript is able to evaluate the same calculation in under 1s.

Here is the time comparison so far

### Fibonacci(40)

| Language   | Time (ms) |
|------------|-----------|
| C          | 660 ms    |
| Go         | 590 ms    |
| Javascript | 970 ms    |
| Python3    | 27200 ms  |

First of all, I am surprised that Go is performing better than C. But that's not part of our current discussion so let's forget C and Go for now and only compare python with javascript.

Both of them are interpreted language, both are dynamically typed so the only thing that could make the difference is the compiler/interpreter. Javascript is executed on `V8` engine and python is interpreted with `CPython` compiler. The major difference between `V8` and `CPython` is that V8 has a **`JIT compiler`** whereas `CPython` doesn't.

## A quick refresher of JIT Compiler.

JIT(Just in time) compilers are a way of improving the performance of an interpreted programs. JIT compiler compiles code during execution rather than prior to execution. It determine the frequently executed code and compiles them to machine code during runtime.

`CPython` executes a code in 2 steps. In step 1, It converts python code into bytecode and in step 2, it interprets the bytecode line by line and converts that to machine code.

`V8` compiles javascript directly into machine code. So if we could compare javascript with python without the JIT compiler, Python would be faster. But this is not the case, for certain types of programs like recursions, Python would be slower because it doesn't pre-compile for future use.

Hence in our tail end recursive Fibonacci function, the bytecode is interpreted line by line `331160281` times which hampers the python performance. Whereas the JIT compiler in V8 compiles the recursive function into machine code as soon as it realizes that is getting called frequently. Hence translation time from javascript code to machine code is saved almost `331160281` time which makes javascript faster.

### Why CPython doesn't have JIT compiler?

The answer to this is simple. CPython doesn't have enough resources or money compared to V8. You might already know about pypy which is an alternate implementation of python to Cpython. Pypy has a JIT compiler so is comparatively faster.

Let's see how much time a pypy compiler would take for calculating a fibonacci(40) using recursion

![]({{ "/assets/why-python-recursions-are-too-slow/pypy-output.png" | relative_url }})

### Clearly pypy is much faster than CPython so why don't we use it ?

Because pypy doesn't have support to many common libraries like numpy, pandas, scipy etc.

### So what is a workaround to make recursions faster with CPython?

Now since we know that CPython is not an intelligent compiler. We have to make sure that our program is optimized the best way possible. For this particular use case, we could either have a memoized algorithm or you could use function caching.

### What is function caching?

Python has this cool feature of letting us cache the return value of a function based on the arguments passed - [read more](https://book.pythontips.com/en/latest/function_caching.html#:~:text=Function%20caching%20allows%20us%20to,to%20write%20a%20custom%20implementation.)

Let's check the calculation time with function caching for `fib(40)`

```python
import time
import sys
from functools import lru_cache

@lru_cache(maxsize=50)
def fib(n):
    if n <= 1:
        return n
    else:
        return fib(n-1) + fib(n-2)

if __name__ == "__main__":
    number = int(sys.argv[1])
    pro_time = time.process_time()
    fib(number)
    print(f"Time taken = {pro_time}s")
```

![]({{ "/assets/why-python-recursions-are-too-slow/cache-output.png" | relative_url }})

![]({{ "/assets/why-python-recursions-are-too-slow/reaction-cache.webp" | relative_url }})

Obviously, we could have written a memoized Fibonacci algorithm but this is another way of optimizing the program.

## Conclusion

![]({{ "/assets/why-python-recursions-are-too-slow/conclusion.gif" | relative_url }})

Writing programs in python requires more performance optimization at the code level because the CPython interpreter won't optimize it much for you.
