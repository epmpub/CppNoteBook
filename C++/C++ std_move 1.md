# std::move

The *std::move* is an unconditional *xvalue* cast, typically denoting that the state of the object being cast is no longer required and can be consumed during the expression evaluation.

As a cast, the *std::move* doesn’t do any moving; the moving or consumption operation is left to the operator, constructor or function that accepts the cast object.

```C++
#include <utility>
#include <stack>
#include <vector>

struct X {};
void some_func(const X& arg) {}
void some_func(X&& arg) {}

// Function that accepts xvalues and prvalues
void example(X&& arg) {
  // Names of variables, functions and data members are always lvalues.
  some_func(arg); // calls some_func(const X&)

  // If the above call would call some_func(X&&), arg could 
  // be silently invalidated by that call.

  // However, if we actually want that, i.e. we are done with arg.
  // We can explicitly cast arg to an xvalue.
  some_func(std::move(arg)); // calls some_func(X&&)

  // After a call to a function that accepts xvalue, assume the
  // object is in moved-from state, i.e. the only valid operation
  // is assigning a new value to the object (unless the type provides
  // additional guarantees).
}

example(X{}); // call with prvalue (temporary)

std::vector<X> data;
{   // Typical use case for std::move
    X some_var; // create a variable
    some_func(some_var); // do some operations on the variable
    // once we are done with it, we can let the final operation
    // to consume the state
    data.push_back(std::move(some_var));
}


// We can apply similar logic to getters.
std::stack<X> stack;
stack.push({});

auto x = stack.top(); // Copy
auto y = std::move(stack.top()); // Move
// However, be very careful about invalidating the internal
// invariants of the datastructure (anything ordered).


// Finally, applying the move cast to an immutable value produces
// an immutable rvalue, and we cannot consume the state of
// immutable values.
const X x;
some_func(std::move(x)); // calls some_func(const X&)
X y = std::move(x); // Copy
// decltype(std::move(x)) == const X&&
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/oa79rq763)