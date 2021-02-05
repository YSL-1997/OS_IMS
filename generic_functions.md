# Generic Functions in C
Generics are syntax components of a programming language that can be **reused** for different types of objects. Generics are referred to as templates (in C++). 


## References
https://levelup.gitconnected.com/using-templates-and-generics-in-c-968da223154d
http://cs.boisestate.edu/~amit/teaching/253/handouts/07-c-generic-coding-handout.pdf

https://stackoverflow.com/questions/13469381/how-to-make-generic-function-using-void-in-c

## Motivating Question
On Stackoverflow, there's a question asking about how to write a single increment-by-one function that takes any possible data types as a parameter. He presented the original code, which is as follows: (note that this piece of code is incorrect)
```
void increment(void *void_ptr)
{
	(*void_ptr)++;
}
```
Then in ```main()```, it is more likely to declare three values of different types (int, float, char, double, etc.), say ```float f = 3.4f;```, and then call ```increment(&f);```.

However, when we actually executing this program, there will be:
```
x.c: In function ‘increment’:
x.c:5:4: warning: dereferencing ‘void *’ pointer
    5 |   (*void_ptr)++;
      |    ^~~
x.c:5:8: error: invalid use of void expression
    5 |   (*void_ptr)++;
```
How to solve this problem? See below:

## Function Pointers
Firstly, we need to know what a function pointer is.
In C, the name of a function is a pointer. (See the code below)
```
// prototype of foo(), foo is a pointer
int foo(int x); 

// ptr to a function, with an int argument, returns int
int (*func)(int);

// assign foo to func, func now points to foo (func is a ptr to a function)
func = foo;

// n1 and n2 have the same value
int n1 = (*func)(1); 
int n2 = foo(1);

assert(n1 == n2);
```

Below is an array of function ptrs:
```
/* 
	an array of function pointers, with each takes two arguments (int, and char*)
	each function ptr returns an int
*/
int (*array_of_functions[10])(int, char*);
```
If we check each element in ```array_of_functions```, it would be ```int (*each_func)(int, char*)```.


### void * as a parameter
- ```void *``` allows us to bypass the type-checking of heap variables.
- ```void *``` is usually used when it's not necessary for the function to know the exact type of the data that might be involved. 
- ```qsort``` uses a callback function to without having to know any details of the data.

void * is a pointer to a generic block of memory, which could be anything: an int, float, string, etc. The length of the block of memory isn't even stored in the pointer, let alone the type of the data. Remember that internally, all data are bits and bytes, and types are really just markers for how the logical data are physically encoded, because intrinsically, bits and bytes are typeless. In C, this information is not stored with variables, so you have to provide it to the compiler yourself, so that it knows whether to apply operations to treat the bit sequences as 2's complement integers, IEEE 754 double-precision floating point, ASCII character data, functions, etc.; these are all specific standards of formats and operations for different types of data. When you cast a void * to a pointer to a specific type, you as the programmer are asserting that the data pointed to actually is of the type you're casting it to. Otherwise, you're probably in for weird behavior.


### qsort()
The prototype for quicksort function **qsort** in the standard C library uses **a function ptr to compare function** to enable a generic sort function.
```
/*
	qsort needs a compare function
	if x '==' y, return  0
	if x '<' y , return <0
	if x '>' y , return >0
*/

void qsort(void* base, size_t nmemb, size_t size,
		   int(*compar)(const void *, const void *)); // the last argument is the function ptr
```
Note that:
- ```void* base``` is a pointer to the data to be sorted.
- ```size_t nmemb``` is the number of members(elements) in the array to be sorted.
- ```size_t size``` is the size of each element in the array.
- The last parameter is **a pointer to a function**, which compares individual items. We need to implement the comparison function to make ```qsort``` work.

For instance, inside ```qsort```, it might be like:
```
// if (base[i] < base[j]) ...
if (compar((char*)base+j, (char*)base+i) == -1)
```

Here comes the question: how could we create the compare function?


Let's imagine a situation where we would like to compare two student id's. The struct of student is as follows:
```
struct student{
	int id;
} student;
```
Before com



