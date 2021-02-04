# Generic Functions in C
Generics are syntax components of a programming language that can be **reused** for different types of objects. Generics are referred to as templates (in C++).


## References
https://levelup.gitconnected.com/using-templates-and-generics-in-c-968da223154d
http://cs.boisestate.edu/~amit/teaching/253/handouts/07-c-generic-coding-handout.pdf

## Function Pointers
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
/* an array of function pointers, with each takes two arguments (int, and char*)
   each function ptr returns an int
*/
int (*array_of_functions[10])(int, char*);
```
If we check each element in ```array_of_functions```, we could see