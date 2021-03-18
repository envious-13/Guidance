# Guidance

## Explanation behind debug template

 #define trace(...) __f(#__VA_ARGS__, __VA_ARGS__)
template <typename Arg1>
void __f(const char *name, Arg1 &&arg1)
{
    cerr << name << " : " << arg1 << endl;
}

template <typename Arg1, typename... Args>
void __f(const char *names, Arg1 &&arg1, Args&&... args)
{
    const char *comma = strchr(names + 1, ',');
    cerr.write(names, comma - names) << " : " << arg1 << " | ";
    __f(comma + 1, args...);
} 

### Now we'll try to breakdown it

The line:

#define trace(...) __f(#__VA_ARGS__, __VA_ARGS__)
defines a Variadic macro trace(...) which takes a variable number of arguments (variadic 4). We know this because it uses the ellipsis (...) operator.
The sequence of tokens inside the parenthesis replaces the special identifier __VA_ARGS__.
'#' is Stringizing 1 operator. The stringizing operator converts macro parameters to string literals.

We all have been using variadic function in C/C++ without even realizing it. printf is one such example :p. It allows you to pass variable number of arguments.

Now, we know that all the trace() calls will be replaced by __f() calls. For example:

trace(a)       -> __f("a", a)
trace(a, b)    -> __f("a, b", a, b)
trace(a, b, c) -> __f("a, b, c", a, b, c) and so on...
The following statement:

template <typename Arg1>					// (a)
template <typename Arg1, typename... Args>	// (b)
basically allows us to use trace() with any data type. It allows us to code generically in C++. For example, we are able to create both std::vector <int> or std::vector <float> because std::vector is a templated class.

We can also see that __f() is overloaded because it has two distinct definitions. One takes only two arguments (let’s refer to it as f1 for brevity) and another takes variable number of arguments (similarly f2).

The f2 is a recursive function and f1 is its base case. Whenever we call __f() with more than 2 arguments, f2 will pick it up, do something (print the name and value of first argument in this case) and pass the rest of the arguments in f2 again (by a recursive call). In the end when it calls __f() with exactly 2 arguments , f1 will be executed (as a base case).

const char* names takes the string passed by the Stringizing operator (# in #__VA_ARGS__) in both of these functions.

int&& x denotes an rvalue reference. To be honest, I don’t fully grasp the concept of rvalues and lvalues in C/C++. I just have a basic idea. For now, we can probably think of it as being a reference to the actual value of the variable. For example, take a look at this code: https://ideone.com/r5G5iy 2.

So, considering trace(a, b, c, d) as an example, the macro will be translated as __f("a, b, c, d", a, b, c, d). This has more than 2 arguments, so it will go to the second definition of __f() where names will point to "a, b, c, d" and arg1 will take care of the second argument a and the rest b, c, d will be handled by args.

Now, strchr() will find the first occurrence of , in names (this will be pointed to by comma variable) and this will be printed. In the next call names will point to comma + 1 as we are done with the first argument and don’t need it any more. This process will be repeated until the last argument passed to trace() is processed.

cerr will just output to error stream. Unless otherwise specified, cerr (error stream) and cout (output stream) are tied to the same output screen (terminal).
