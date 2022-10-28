## Copying and Copy Constructors
- Issue with default copy constructor is that if there is a pointer, the address is copied over, not a copy of the underlying object.
```c++
String(const String& other) {
  ..................
}
```

## Lvalue and Rvalue in C++ 
- Every expression in C++ is either Lvalue or Rvalue expression. 
- **Lvalue:** If you can take address of expression then it is Lvalue, and they last extended period of time.
- **Rvalue:** Can't take address and they are temporary, they don't exist after one line 

```c++
int x = 10; // x is lvalue, 10 is r value

int a = 10, b = 20;
int x = (a + b); // x is lvalue, (a + b) is rvalue
int* p = &(a + b); // ERROR: (a + b) is not lvalue

class Cat {};
Cat c = Cat(); // Cat() is an rvalue 

int square(int x) { return x*x; }
int sq = square(10); // square(10) is an rvalue
```
- **Lvalue Reference:** an refer to rvalue if it is const (i.e. const int& ref = 10). When passing an rvalue to function and parameter is expecting an lvalue reference, make the parameter const reference to allow rvalues to be sent in (i.e. void setValueTo99(const int& param)).
- **Rvalue Reference:** Mainly used for moving semantics and perfect forwarding. Use && to create Rvalue reference (i.e. int&& ref = 10). Reason we might want to use is don't want to be continually copying, and instead want to just move something that is already created somewhere.
- **Move Constructor:** Creates an inexpensive shallow copy of rvalue argument. In constrast to copy constructor which is making expensive deep copy.
```c++
Vector(const Vector& v) { // Copy Constructor (called by foo(reusable))
  *perform deep copy* 
}

Vector(Vector&& v) { // Move Constructor (called by foo(createVector())
  *move contents from v to this*
}
```
