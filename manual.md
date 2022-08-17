# Manual


```
cake [options] source1.c source2.c ...

Options
  -I Adds a directory to the list of directories searched for include files.
  -D Defines a preprocessing symbol for a source file.
  -E Copies preprocessor output to standard output.
  -r Remove all comments from the ouput file.
  -rm Outputs preprocessed code after compilation.
  
```
The ouput dir is `./out`

## Transformations

## C99
C99 is the minimum target. Do you need C89?

## C11 Thread_local/Atomic
Parsed but not transformed.

## C11 Generics
`_Generics` are been parsed but not transformed yet.

## C11 u8"literals"
u8 literals are converted to escape sequecences. (I don't recoment u8"")

## C11 Static_assert
When compiling to versions < C11 static_assert is removed

## C23 Decimal32, Decimal64, and Decimal128
Not implemented (maybe parsed?)

## C11 Noreturn
Parsed. Todo needs to be replaced by `[[_Noreturn]]` in C23

## C23 static_assert / Single-argument Static_assert
In C23 `static_assert` can be used as keyword and the message is optional.
Compiling to C11 we add some dumy message is necessary and we use the previous keyword `_Static_assert`



## C23 u8 char literal
not implemented yet.

### C23 Digit separators
When compiling to versions < 23 the compiler removes the ' delimiter from tokens. 

### C23 Binary literals 
When compiling to versions < 23 the compiler translater the binary literal to a hexadecimal constant.

### C23 nullptr
When compiling to version < 23 nullptr is replaced with `((void*)0)`

### C23 bool true false
When compiling to version < 23 bool is replaced with `_Bool`, true is replaced with `((_Bool)1)` and false
with `(_Bool)0)`

## C23 {} empty initializer

Cake transform {} into {0}. 

## C23 typeof / typeof_unqual
When compiling to versions < 23 we replace typeof by the equivalent type. 
This can be a little complex in some cases.

## C23 constexpr
Not implemented yet (maybe parsed?)

## C23 enuns with type
Not implemented yet (not parsed yet)

## C23 Attributes
Attributes are being parsed and removed in some places. More work is necessary.

## C23 has_attribute
Yes but need work/review.

## C23 has_include
Yes but need work/review.

## C23 warning
parsed, printed but not removed when compiling to c99.

## C23 VAOPT
Yes but need work.
## C23 BitInt(N))
Not implemented

## embed
An initial implementation is already there. More work is necessary.

## elifdef elifndef
Are implemented

## Extensions

### try catch throw
try cath is a external block that we can jump off. (local jump only)
```cpp
try
{
   for (int i = 0 ; i < 10; i++) {
      for (int j = 0 ; j < 10; j++) {
        ... 
        if (error) throw;
        ...
      }
   }
}
catch
{
}
```
### defer

`defer` will call the defer statement before the block exit at inverse orden of declaration.

For instance:

```c
#include <stdio.h>

int main() {
  do {
     FILE* f = fopen("in.txt", "r");
     if (f == NULL) break;
     defer fclose(f);

     FILE* f2 = fopen("out.txt", "w");
     if (f2 == NULL) break;
     defer fclose(f2);
     //...    
  }
  while(0);
}
```
Becomes:

```c
#include <stdio.h>

int main() {
  do {
     FILE* f = fopen("in.txt", "r");
     if (f == ((void*)0)) break;

     FILE* f2 = fopen("out.txt", "w");
     if (f2 == ((void*)0)) {  fclose(f); break;}
     
     fclose(f2); fclose(f);
   }
  while(0);
}
```
I guess everthing is working including `goto`s.

### if with initializer

No idea why C++ 17 if with initializer was not proposed for C23!
But in cake it is implemented.

```c
#include <stdio.h>
int main()
{
   int size = 10;
   if (FILE* f = fopen("file.txt", "r"); f)
   {
     /*...*/
     fclose(f);
   }
}
```
Becomes

```c
#include <stdio.h>

int main()
{
   int size = 10;
   {FILE* f = fopen("file.txt", "r");if ( f)
   {
     /*...*/
     fclose(f);
   }}
}
```
An extension if + initializer + defer expression was considered but not implemented yet.

### lambdas

Lambdas without capture where implemented using a syntax similar of 
compound literal for function pointer.

Lambdas are the most complex code transformation so far because sometimes function scope 
types needs to be transformed to file scope. This is important because manual lambda capture
is something we want to use in function scope.

For instance:

```c
extern char* strdup(const char* s);
void create_app(const char* appname)
{
  struct capture {
     char * name;
  } capture = { .name = strdup(appname) };

  (void (void* p)) {
    struct capture* capture = p;    
  }(&capture); 
}
```
Because struct capture was in function scope and the lambda function will be created
at file scope the type `struct capture` had to be moved from function scope to file scope.

```c
extern char* strdup(const char* s);

struct _capture0 {
     char * name;
  };
  
void _lit_func_0(void *p) {
    struct _capture0* capture = p;    
  }

void create_app(const char* appname)
{
  struct _capture0  capture = { .name = strdup(appname) };
  _lit_func_0(&capture);  
}
```

### Repeat
Repeat is equivalent of `for(;;)`
